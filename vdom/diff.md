```javascript
import { ATTR_KEY } from '../constants';
import { isSameNodeType, isNamedNode } from './index';
import { buildComponentFromVNode } from './component';
import { createNode, setAccessor } from '../dom/index';
import { unmountComponent } from './component';
import options from '../options';
import { removeNode } from '../dom';

/** Queue of components that have been mounted and are awaiting componentDidMount */
// 已经mount的组件队列，收集等待被调用componentDidMount回调的组件
export const mounts = [];

/** Diff recursion count, used to track the end of the diff cycle. */
// diff递归次数总计，用来跟踪diff cycle
export let diffLevel = 0;

/** Global flag indicating if the diff is currently within an SVG */
// 判断当前的DOM树是否为SVG
let isSvgMode = false;

/** Global flag indicating if the diff is performing hydration */
// 判断当前元素是否缓存了之前的虚拟dom
let hydrating = false;

/** Invoke queued componentDidMount lifecycle methods */
// 触发componentDidMount与afterMount
export function flushMounts() {
	let c;
	while ((c=mounts.pop())) {
		if (options.afterMount) options.afterMount(c);
		if (c.componentDidMount) c.componentDidMount();
	}
}


/** Apply differences in a given vnode (and it's deep children) to a real DOM Node.
 *	@param {Element} [dom=null]		A DOM node to mutate into the shape of the `vnode`
 *	@param {VNode} vnode			A VNode (with descendants forming a tree) representing the desired DOM structure
 *	@returns {Element} dom			The created/mutated element
 *	@private
 */
export function diff(dom, vnode, context, mountAll, parent, componentRoot) {
	// diffLevel having been 0 here indicates initial entry into the diff (not a subdiff)
	// 初始入口的操作
	if (!diffLevel++) {
		// when first starting the diff, check if we're diffing an SVG or within an SVG
		// 当刚开始进入diff时，检查dom树的类型是否为SVG
		isSvgMode = parent!=null && parent.ownerSVGElement!==undefined;

		// hydration is indicated by the existing element to be diffed not having a prop cache
		// 检查是否缓存了数据
		hydrating = dom!=null && !(ATTR_KEY in dom);
	}
	// 返回新的dom
	let ret = idiff(dom, vnode, context, mountAll, componentRoot);

	// append the element if its a new parent
	// 添加到新的父节点上
	if (parent && ret.parentNode!==parent) parent.appendChild(ret);

	// diffLevel being reduced to 0 means we're exiting the diff
	// 退出diff
	if (!--diffLevel) {
		hydrating = false;
		// invoke queued componentDidMount lifecycle methods
		// 执行所有hook方法
		if (!componentRoot) flushMounts();
	}

	return ret;
}


/** Internals of `diff()`, separated to allow bypassing diffLevel / mount flushing. */
/**
 * 1. 首先判断vnode是否为空值，如果是将vnode设定为空字符串
 * 2. 再次判断vnode是否为字符串或者数字，内部会判断是否为文本节点，最后进行更新或者替换工作
 * 3. 如果vnode.nodeName是一个component则进行组件的渲染
 */
function idiff(dom, vnode, context, mountAll, componentRoot) {
	let out = dom,
		prevSvgMode = isSvgMode;

	// empty values (null, undefined, booleans) render as empty Text nodes
	// 将null, undefined, boolean转换为空字符
	if (vnode==null || typeof vnode==='boolean') vnode = '';


	// Fast case: Strings & Numbers create/update Text nodes.
	// 将字符串和数字转化为文本节点
	if (typeof vnode==='string' || typeof vnode==='number') {

		// update if it's already a Text node:
		// 如果dom是文本节点
		if (dom && dom.splitText!==undefined && dom.parentNode && (!dom._component || componentRoot)) {
			/* istanbul ignore if */ /* Browser quirk that can't be covered: https://github.com/developit/preact/commit/fd4f21f5c45dfd75151bd27b4c217d8003aa5eb9 */
			if (dom.nodeValue!=vnode) {
				dom.nodeValue = vnode;
			}
		}
		else {
			// it wasn't a Text node: replace it with one and recycle the old Element
			// 如果不是文本节点，则创建新的虚拟dom
			out = document.createTextNode(vnode);
			if (dom) {
				if (dom.parentNode) dom.parentNode.replaceChild(out, dom);
				// 回收旧的dom
				recollectNodeTree(dom, true);
			}
		}

		out[ATTR_KEY] = true; // 暂时看应该是标明dom为preact使用的dom？

		return out;
	}


	// If the VNode represents a Component, perform a component diff:
	// 如果vnodeName 是一个组件，则buildComponentFromVNode
	let vnodeName = vnode.nodeName;
	if (typeof vnodeName==='function') {
		return buildComponentFromVNode(dom, vnode, context, mountAll);
	}


	// Tracks entering and exiting SVG namespace when descending through the tree.
	// 更新isSvgMode flag
	isSvgMode = vnodeName==='svg' ? true : vnodeName==='foreignObject' ? false : isSvgMode;


	// If there's no existing element or it's the wrong type, create a new one:
	// 如果dom不存在或者不是相同的节点，则创建新的
	vnodeName = String(vnodeName);
	if (!dom || !isNamedNode(dom, vnodeName)) {
		out = createNode(vnodeName, isSvgMode);

		if (dom) {
			// move children into the replacement node
			// 将真实的dom，转移到虚拟dom中
			while (dom.firstChild) out.appendChild(dom.firstChild);

			// if the previous Element was mounted into the DOM, replace it inline
			// 如果父元素存在，则插入到父节点
			if (dom.parentNode) dom.parentNode.replaceChild(out, dom);
		
			// recycle the old element (skips non-Element node types)
			// 回收旧的dom
			recollectNodeTree(dom, true);
		}
	}


	let fc = out.firstChild,
		props = out[ATTR_KEY],
		vchildren = vnode.children;

	if (props==null) {
		props = out[ATTR_KEY] = {};
		for (let a=out.attributes, i=a.length; i--; ) props[a[i].name] = a[i].value;
	}

	// Optimization: fast-path for elements containing a single TextNode:
	// 当只有一个文本节点时则直接替换
	if (!hydrating && vchildren && vchildren.length===1 && typeof vchildren[0]==='string' && fc!=null && fc.splitText!==undefined && fc.nextSibling==null) {
		if (fc.nodeValue!=vchildren[0]) {
			fc.nodeValue = vchildren[0];
		}
	}
	// otherwise, if there are existing or new children, diff them:
	// 否则，如果存在chidren,则进行diff
	else if (vchildren && vchildren.length || fc!=null) {
		innerDiffNode(out, vchildren, context, mountAll, hydrating || props.dangerouslySetInnerHTML!=null);
	}


	// Apply attributes/props from VNode to the DOM Element:
	diffAttributes(out, vnode.attributes, props);


	// restore previous SVG mode: (in case we're exiting an SVG namespace)
	isSvgMode = prevSvgMode;

	return out;
}


/** Apply child and attribute changes between a VNode and a DOM Node to the DOM.
 *  内部diff，比较子节点和属性的变化
 *	@param {Element} dom			Element whose children should be compared & mutated
 *	@param {Array} vchildren		Array of VNodes to compare to `dom.childNodes`
 *	@param {Object} context			Implicitly descendant context object (from most recent `getChildContext()`)
 *	@param {Boolean} mountAll
 *	@param {Boolean} isHydrating	If `true`, consumes externally created elements similar to hydration
 */
function innerDiffNode(dom, vchildren, context, mountAll, isHydrating) {
	let originalChildren = dom.childNodes,
		children = [],
		keyed = {},
		keyedLen = 0,
		min = 0,
		len = originalChildren.length,
		childrenLen = 0,
		vlen = vchildren ? vchildren.length : 0,
		j, c, f, vchild, child;

	// Build up a map of keyed children and an Array of unkeyed children:
	// 所有具有key值的child进入keyed，没有key值进入children
	// 此处检查的不是虚拟DOM
	if (len!==0) {
		for (let i=0; i<len; i++) {
			let child = originalChildren[i],
				props = child[ATTR_KEY],
				// 查找key，_component指定了就用_component.__key。没有就用props.key
				key = vlen && props ? child._component ? child._component.__key : props.key : null;
			if (key!=null) {
				keyedLen++;
				keyed[key] = child;
			}
			// 此处比较绕弯，先判断是否有属性存在，没有再判断是否为Text，是的话看它先前是否是存在的，最后的 : isHydrating 就是不存在的情况
			else if (props || (child.splitText!==undefined ? (isHydrating ? child.nodeValue.trim() : true) : isHydrating)) {
				children[childrenLen++] = child;
			}
		}
	}

	if (vlen!==0) {
		for (let i=0; i<vlen; i++) {
			vchild = vchildren[i];
			child = null;

			// attempt to find a node based on key matching
			// 匹配已经在keyed里面先前的node
			let key = vchild.key;
			if (key!=null) {
				if (keyedLen && keyed[key]!==undefined) {
					child = keyed[key];
					// 清除掉keyed里面的匹配到的value
					keyed[key] = undefined;
					keyedLen--;
				}
			}
			// attempt to pluck a node of the same type from the existing children
			// 匹配相同type的先前存在的child
			else if (!child && min<childrenLen) {
				for (j=min; j<childrenLen; j++) {
					if (children[j]!==undefined && isSameNodeType(c = children[j], vchild, isHydrating)) {
						child = c;
						children[j] = undefined;
						if (j===childrenLen-1) childrenLen--;
						if (j===min) min++;
						break;
					}
				}
			}

			// morph the matched/found/created DOM child to match vchild (deep)
			// 拿出匹配到的child再次进行diff算法比较，一层一层往深层diff
			child = idiff(child, vchild, context, mountAll);

			f = originalChildren[i];
			// child存在并且child不等于之前dom并且child不等于本次循环内的先前原始子节点
			if (child && child!==dom && child!==f) {
				if (f==null) {
					// 子节点为空，直接append
					dom.appendChild(child);
				}
				// 此处为了提升性能，每次
				else if (child===f.nextSibling) {
					// 表明child是新的dom，那么删除旧的节点
					removeNode(f);
				}
				else {
					// insertBefore，将child插到f前面是为了提升性能,以便下一次判断兄弟节点进行删除操作。
					dom.insertBefore(child, f);
				}
			}
		}
	}


	// 进行回收
	// remove unused keyed children:
	if (keyedLen) {
		for (let i in keyed) if (keyed[i]!==undefined) recollectNodeTree(keyed[i], false);
	}

	// remove orphaned unkeyed children:
	while (min<=childrenLen) {
		if ((child = children[childrenLen--])!==undefined) recollectNodeTree(child, false);
	}
}



/** Recursively recycle (or just unmount) a node and its descendants.
 *	@param {Node} node						DOM node to start unmount/removal from
 *	@param {Boolean} [unmountOnly=false]	If `true`, only triggers unmount lifecycle, skips removal
 */
 /**
  * 重新回收♻️整个节点树
	* unmountOnly表明只触发unmount生命周期，不进行节点的移除
	*/
export function recollectNodeTree(node, unmountOnly) {
	let component = node._component;
	if (component) {
		// if node is owned by a Component, unmount that component (ends up recursing back here)
		unmountComponent(component);
	}
	else {
		// If the node's VNode had a ref function, invoke it with null here.
		// (this is part of the React spec, and smart for unsetting references)
		// 如果存在ref，则设置为null
		if (node[ATTR_KEY]!=null && node[ATTR_KEY].ref) node[ATTR_KEY].ref(null);

		if (unmountOnly===false || node[ATTR_KEY]==null) {
			removeNode(node);
		}

		removeChildren(node);
	}
}


/** Recollect/unmount all children.
 *	- we use .lastChild here because it causes less reflow than .firstChild
 *	- it's also cheaper than accessing the .childNodes Live NodeList
 */
// 回收♻️所有的子孙
export function removeChildren(node) {
	node = node.lastChild;
	while (node) {
		let next = node.previousSibling;
		recollectNodeTree(node, true);
		node = next;
	}
}


/** Apply differences in attributes from a VNode to the given DOM Element.
 *	@param {Element} dom		Element with attributes to diff `attrs` against
 *	@param {Object} attrs		The desired end-state key-value attribute pairs
 *	@param {Object} old			Current/previous attributes (from previous VNode or element's prop cache)
 */
function diffAttributes(dom, attrs, old) {
	let name;

	// remove attributes no longer present on the vnode by setting them to undefined
	// 移除属性因为现在的name为空了
	for (name in old) {
		if (!(attrs && attrs[name]!=null) && old[name]!=null) {
			setAccessor(dom, name, old[name], old[name] = undefined, isSvgMode);
		}
	}

	// add new & update changed attributes
	// 添加或更新改变的属性
	for (name in attrs) {
		if (name!=='children' && name!=='innerHTML' && (!(name in old) || attrs[name]!==(name==='value' || name==='checked' ? dom[name] : old[name]))) {
			setAccessor(dom, name, old[name], old[name] = attrs[name], isSvgMode);
		}
	}
}
```