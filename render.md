**render**用来渲染JSX到指定element中

```javascript
import { diff } from './vdom/diff';

/** Render JSX into a `parent` Element.
 *	@param {VNode} vnode		A (JSX) VNode to render
 *	@param {Element} parent		DOM element to render into
 *	@param {Element} [merge]	Attempt to re-use an existing DOM tree rooted at `merge`
 *	@public
 *  
 *  // 下面是例子
 *	@example
 *	// render a div into <body>:
 *	render(<div id="hello">hello!</div>, document.body);
 *
 *	@example
 *	// render a "Thing" component into #foo:
 *	const Thing = ({ name }) => <span>{ name }</span>;
 *	render(<Thing name="one" />, document.querySelector('#foo'));
 */
 // 调用diff方法，这里vnode是虚拟DOM,parent是容器的节点,merge可选，是另外一个已经存在的dom树
 // react第三个参数是一个回调函数
export function render(vnode, parent, merge) {
	return diff(merge, vnode, {}, false, parent, false);
}
```
