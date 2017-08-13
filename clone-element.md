拷贝一个vnode也就是虚拟DOM元素

```javascript
import { extend } from './util';
import { h } from './h';

export function cloneElement(vnode, props) {
	return h(
		vnode.nodeName,
		extend(extend({}, vnode.attributes), props),
		arguments.length>2 ? [].slice.call(arguments, 2) : vnode.children
	);
}
```