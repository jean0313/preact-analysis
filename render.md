# <img src="preact-icon.png" width="32" height="32" /> render

**render**用来渲染JSX到指定element中，其实就是渲染成一个真实的dom。

调用[diff](vdom/diff)方法。

这里vnode是虚拟DOM， parent是容器的DOM。

merge可选，是另外一个已经存在的dom树，如果指定merge则会将虚拟DOM生成的DOM树替换到merge上。如果不指定的话，将会把生成的DOM树依次添加到parent里面。

而React的第三个参数是一个回调函数，在渲染时触发。

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
export function render(vnode, parent, merge) {
	return diff(merge, vnode, {}, false, parent, false);
}
```
