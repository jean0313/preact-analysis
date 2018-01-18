# <img src="preact-icon.png" width="32" height="32" /> util

**util**提供一些公用方法。

```javascript
/** 
 *  Copy all properties from `props` onto `obj`.
 *  把props的属性添加到obj上,浅拷贝,类似于ES6的Object.assign
 *
 *  @param {Object} obj		Object onto which properties should be copied.
 *  @param {Object} props	Object from which to copy properties.
 *  @returns obj
 *  @private
 */
export function extend(obj, props) {
	for (let i in props) obj[i] = props[i];
	return obj;
}

/**
 * Call a function asynchronously, as soon as possible. Makes
 * use of HTML Promise to schedule the callback if available,
 * otherwise falling back to `setTimeout` (mainly for IE<11).
 * 异步调用一个函数。利用了Promise，接受一个callback
 *
 * @param {Function} callback
 */
export const defer = typeof Promise=='function' ? Promise.resolve().then.bind(Promise.resolve()) : setTimeout;
```
