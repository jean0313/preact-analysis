# <img src="preact-icon.png" width="32" height="32" /> clone-element

拷贝一个 VNode 也就是虚拟 DOM 元素。

主要接收两个参数，第一个参数是被 clone 的 vnode,第二个参数是增加的属性，剩下的参数则是替换的 children。

```javascript
import { extend } from './util'
import { h } from './h'

/**
 * Clones the given VNode, optionally adding attributes/props and replacing its children.
 * @param {VNode} vnode		The virtual DOM element to clone
 * @param {Object} props	Attributes/props to add when cloning
 * @param {VNode} rest		Any additional arguments will be used as replacement children.
 */
export function cloneElement(vnode, props) {
  return h(
    vnode.nodeName,
    extend(extend({}, vnode.attributes), props), // 在原本vnode属性的基础上增加新的属性
    arguments.length > 2 ? [].slice.call(arguments, 2) : vnode.children
  )
}
```
