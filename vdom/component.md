```javascript
import { SYNC_RENDER, NO_RENDER, FORCE_RENDER, ASYNC_RENDER, ATTR_KEY } from '../constants';
import options from '../options';
import { extend } from '../util';
import { enqueueRender } from '../render-queue';
import { getNodeProps } from './index';
import { diff, mounts, diffLevel, flushMounts, recollectNodeTree, removeChildren } from './diff';
import { createComponent, collectComponent } from './component-recycler';
import { removeNode } from '../dom';

/** 为组件实例设置属性(props).
 *	@param {Object} props
 *	@param {Object} [opts]
 *	@param {boolean} [opts.renderSync=false]	If `true` and {@link options.syncComponentUpdates} is `true`, triggers synchronous rendering.
 *	@param {boolean} [opts.render=true]			If `false`, no render will be triggered.
 */
export function setComponentProps(component, props, opts, context, mountAll) {
  // 防止其他的修改，其目的相当于一个锁，保证修改过程的原子性
	if (component._disable) return;
	component._disable = true;

  // 删除props在赋值到component内之后
	if ((component.__ref = props.ref)) delete props.ref;
	if ((component.__key = props.key)) delete props.key;

  /**
   * 下面是一些生命周期方法包括props，context的设置
   */
	// 初次渲染，或者强制渲染
	if (!component.base || mountAll) {
		if (component.componentWillMount) component.componentWillMount();
	}
	// 不是初次渲染，则进行componentWillReceiveProps
	else if (component.componentWillReceiveProps) {
		component.componentWillReceiveProps(props, context);
	}

	if (context && context!==component.context) {
		if (!component.prevContext) component.prevContext = component.context;
		component.context = context;
	}

	if (!component.prevProps) component.prevProps = component.props;
	component.props = props;

  // 修改完毕
	component._disable = false;

  // 渲染策略，同步渲染与异步渲染分别操作
	if (opts!==NO_RENDER) {
		// 如果是同步渲染或者首次渲染，则执行renderComponent
		if (opts===SYNC_RENDER || options.syncComponentUpdates!==false || !component.base) {
			renderComponent(component, SYNC_RENDER, mountAll);
		}
		else {
			enqueueRender(component);
		}
	}
	// 如果存在ref函数，则将组件实例作为参数调用ref函数。preact不支持字符串的ref
	if (component.__ref) component.__ref(component);
}



/** Render a Component, triggering necessary lifecycle events and taking High-Order Components into account.
 *  渲染组建，触发必要的生命周期方法
 *	@param {Component} component
 *	@param {Object} [opts]
 *	@param {boolean} [opts.build=false]		If `true`, component will build and store a DOM node if not already associated with one.
 *	@private
 */
export function renderComponent(component, opts, mountAll, isChild) {
	if (component._disable) return;

	let props = component.props,
		state = component.state,
		context = component.context,
		previousProps = component.prevProps || props,
		previousState = component.prevState || state,
		previousContext = component.prevContext || context,
		isUpdate = component.base,  // 通过组件实例是否对应存在真实DOM节点来判断
		nextBase = component.nextBase, // 基于此DOM元素进行修改
		initialBase = isUpdate || nextBase,
		initialChildComponent = component._component, // 表示组件的子组件，即表示是高阶组件的时候
		skip = false,  // 表示是否需要跳过更新的过程
		rendered, inst, cbase;

	// if updating
  	// 如果触发了更新
	if (isUpdate) {
		// 将props,state,context替换成之前的状态，因为在shouldComponentUpdate,componentWillUpdate中this.props,this.state,this.context都是更新前的状态
		component.props = previousProps;
		component.state = previousState;
		component.context = previousContext;
    // 判断是否需要跳过更新
		if (opts!==FORCE_RENDER
			&& component.shouldComponentUpdate
			&& component.shouldComponentUpdate(props, state, context) === false) {
			skip = true;
		}
		else if (component.componentWillUpdate) {
			component.componentWillUpdate(props, state, context);
		}
		// 
		component.props = props;
		component.state = state;
		component.context = context; // 将props,state,context替换成最新的状态
	}

	component.prevProps = component.prevState = component.prevContext = component.nextBase = null; 
	component._dirty = false;

	if (!skip) {
		// 如果不跳过更新，则进行render
		rendered = component.render(props, state, context);

		// context to pass to the child, can be updated via (grand-)parent component
		// 通过context传递全局状态
		// 当前组件实例getChildContext返回的context中的属性会覆盖父组件的context中的相同属性。
		if (component.getChildContext) {
			context = extend(extend({}, context), component.getChildContext());
		}

		let childComponent = rendered && rendered.nodeName, // 组件实例render函数返回的虚拟dom的类型
			toUnmount, base;

		if (typeof childComponent==='function') {
			// 表示为高阶函数
			// set up high order component link
			// 通过getNodeProps获得虚拟dom中子组件的属性
			let childProps = getNodeProps(rendered);
			inst = initialChildComponent;

			if (inst && inst.constructor===childComponent && childProps.key==inst.__key) {
				// 同步渲染的方式，递归调用setComponentProps来更新子组件的属性props
				setComponentProps(inst, childProps, SYNC_RENDER, context, false);
			}
			else {
				toUnmount = inst;

				component._component = inst = createComponent(childComponent, childProps, context);
				inst.nextBase = inst.nextBase || nextBase;
				inst._parentComponent = component;
				setComponentProps(inst, childProps, NO_RENDER, context, false);
				renderComponent(inst, SYNC_RENDER, mountAll, true);
			}

			base = inst.base;
		}
		else {
			// 非组件类型
			cbase = initialBase;

			// destroy high order component link
			toUnmount = initialChildComponent;
			if (toUnmount) {
				cbase = component._component = null;
			}

			if (initialBase || opts===SYNC_RENDER) {
				if (cbase) cbase._component = null;
        		// 开始diff阶段,base存储的是本次组件渲染的真实DOM元素
				base = diff(cbase, rendered, context, mountAll || !isUpdate, initialBase && initialBase.parentNode, true);
			}
		}
		// 如果组件前后返回的虚拟dom节点对应渲染后的真实dom节点，或者前后返回的虚拟DOM节点对应的前后组件实例不一致
		if (initialBase && base!==initialBase && inst!==initialChildComponent) {
			let baseParent = initialBase.parentNode;
			if (baseParent && base!==baseParent) {
        		// 进行替换
				baseParent.replaceChild(base, initialBase);
				// 如果没有需要卸载的实例
				if (!toUnmount) {
					initialBase._component = null;
					recollectNodeTree(initialBase, false);
				}
			}
		}

		if (toUnmount) {
			unmountComponent(toUnmount);
		}
		// 将当前组件渲染的dom元素存储在组件实例的base属性中
		component.base = base;
		if (base && !isChild) {
			let componentRef = component,
				t = component;
			while ((t=t._parentComponent)) {
				(componentRef = t).base = base;
			}
			base._component = componentRef;
			base._componentConstructor = componentRef.constructor;
		}
	}

	if (!isUpdate || mountAll) {
		mounts.unshift(component);
	}
	else if (!skip) {
		// Ensure that pending componentDidMount() hooks of child components
		// are called before the componentDidUpdate() hook in the parent.
		// Note: disabled as it causes duplicate hooks, see https://github.com/developit/preact/issues/750
		// flushMounts();

		if (component.componentDidUpdate) {
			component.componentDidUpdate(previousProps, previousState, previousContext);
		}
		if (options.afterUpdate) options.afterUpdate(component);
	}
	// 如果存在_renderCallbacks属性，这里相当于至于setState的回调函数
	if (component._renderCallbacks!=null) {
		while (component._renderCallbacks.length) component._renderCallbacks.pop().call(component);
	}

	if (!diffLevel && !isChild) flushMounts();
}



/** Apply the Component referenced by a VNode to the DOM.
 *	@param {Element} dom	The DOM node to mutate
 *	@param {VNode} vnode	A Component-referencing VNode
 *	@returns {Element} dom	The created/mutated element
 *	@private
 */
// 将组件的虚拟dom(VNode)转化成真实dom
export function buildComponentFromVNode(dom, vnode, context, mountAll) {
	let c = dom && dom._component,  // dom是组件对应的真实dom节点（如果未渲染，则为undefined）,dom._component属性是组件实例的缓存
		originalComponent = c,
		oldDom = dom,
		isDirectOwner = c && dom._componentConstructor===vnode.nodeName,// 标识原DOM节点对应的组件类型是否与当前虚拟DOM的组件类型相同
		isOwner = isDirectOwner,
		props = getNodeProps(vnode); //获取Vnode节点的属性值

	// 主要找到最开始渲染该DOM的高阶组件(防止某些情况下dom对应的_component属性指代的实例被修改)，然后再判断该高阶组件是否与当前的vnode类型一致。
	while (c && !isOwner && (c=c._parentComponent)) {
		isOwner = c.constructor===vnode.nodeName;
	}
	// 如果存在当前虚拟dom对应的组件实例存在，则直接调用函数setComponentProps
	if (c && isOwner && (!mountAll || c._component)) {
		setComponentProps(c, props, ASYNC_RENDER, context, mountAll);
		dom = c.base;
	}
	else {
		//首先如果之前的dom节点对应存在组件，并且虚拟dom对应的组件类型与其不相同时，则卸载之前的组件
		if (originalComponent && !isDirectOwner) {
			unmountComponent(originalComponent);
			dom = oldDom = null;
		}
		// 然后创建组件实例
		c = createComponent(vnode.nodeName, props, context);
		if (dom && !c.nextBase) {
			c.nextBase = dom;
			// passing dom/oldDom as nextBase will recycle it if unused, so bypass recycling on L229:
			oldDom = null;
		}
		// 创建组件实例dom节点
		setComponentProps(c, props, SYNC_RENDER, context, mountAll);
		dom = c.base;
		// 如果创建的当前的DOM与之前的DOM元素不相同，则将之前的DOM回收
		if (oldDom && dom!==oldDom) {
			oldDom._component = null;
			recollectNodeTree(oldDom, false);
		}
	}

	return dom;
}


// 从DOM中移除组件
/** Remove a component from the DOM and recycle it.
 *	@param {Component} component	The Component instance to unmount
 *	@private
 */
export function unmountComponent(component) {
	// 执行beforeUnmount的钩子函数
	if (options.beforeUnmount) options.beforeUnmount(component);
	// 
	let base = component.base;

	component._disable = true;  // 表示组件禁用

	if (component.componentWillUnmount) component.componentWillUnmount();

	component.base = null;

	// recursively tear down & recollect high-order component children:
	let inner = component._component;
	if (inner) {
		unmountComponent(inner);
	}
	else if (base) {
		// 
		if (base[ATTR_KEY] && base[ATTR_KEY].ref) base[ATTR_KEY].ref(null);

		component.nextBase = base;
		// 将base节点的父节点脱离出来
		removeNode(base);
		collectComponent(component);
		// 用递归遍历所有的子DOM元素，回收节点
		removeChildren(base);
	}
	// 如果最外层节点存在ref函数，则以参数null执行
	if (component.__ref) component.__ref(null);
}
```