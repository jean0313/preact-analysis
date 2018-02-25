# <img src="preact-icon.png" width="32" height="32" /> render

**render**用来渲染 JSX 到指定 element 中，其实就是渲染成一个真实的 dom。

调用[diff](vdom/diff)方法。

这里 vnode 是虚拟 DOM， parent 是容器的 DOM。

merge 可选，是另外一个已经存在的 dom 树，如果指定 merge 则会将虚拟 DOM 生成的 DOM 树替换到 merge 上。如果不指定的话，将会把生成的 DOM 树添加到 parent 里面。

而 React 的第三个参数是一个回调函数，在渲染时触发。

```javascript
import { diff } from './vdom/diff'

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
export function render(vnode, parent, merge) {
  return diff(merge, vnode, {}, false, parent, false)
}
```
