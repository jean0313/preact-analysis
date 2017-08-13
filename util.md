
**util**提供一些公用方法

```javascript
/** Copy own-properties from `props` onto `obj`.
 *  把props的属性添加到obj上,浅拷贝,类似于ES6的Object.assign
 *	@returns obj
 *	@private
 */
export function extend(obj, props) {
	for (let i in props) obj[i] = props[i];
	return obj;
}

/** Call a function asynchronously, as soon as possible.
 *  异步调用，使用Promise.prototype.then或者如果没有Promimse则使用setTimeout
 *	@param {Function} callback
 */
export const defer = typeof Promise=='function' ? Promise.resolve().then.bind(Promise.resolve()) : setTimeout;
```