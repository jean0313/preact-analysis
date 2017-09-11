拷贝一个vnode也就是虚拟DOM元素

```javascript
import { extend } from './util';
import { h } from './h';
// 主要接收两个参数，第一个参数是被clone的vnode,第二个参数是增加的属性
export function cloneElement(vnode, props) {
	return h(
		vnode.nodeName,
		extend(extend({}, vnode.attributes), props),   // 在原本vnode属性的基础上增加新的属性
		arguments.length>2 ? [].slice.call(arguments, 2) : vnode.children
	);
}
```