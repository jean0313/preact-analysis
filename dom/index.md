# <img src="preact-icon.png" width="32" height="32" /> index

一些 dom 方法。

```javascript
import { IS_NON_DIMENSIONAL } from '../constants'
import options from '../options'

/** Create an element with the given nodeName.
 *  @param {String} nodeName
 *  @param {Boolean} [isSvg=false]  If `true`, creates an element within the SVG namespace.
 *  @returns {Element} node
 */
export function createNode(nodeName, isSvg) {
  // 这里我们需要注意区分普通元素与 SVG 元素
  let node = isSvg
    ? document.createElementNS('http://www.w3.org/2000/svg', nodeName)
    : document.createElement(nodeName)
  node.normalizedNodeName = nodeName
  return node
}

/** Remove a child node from its parent if attached.
 *  @param {Element} node       The node to remove
 */
// 从父节点删除该节点
export function removeNode(node) {
  let parentNode = node.parentNode
  if (parentNode) parentNode.removeChild(node)
}
```

> 设置存取器，对 node 中的属性一一处理。

```javascript
/** Set a named attribute on the given Node, with special behavior for some names and event handlers.
 *  If `value` is `null`, the attribute/handler will be removed.
 *  @param {Element} node   An element to mutate
 *  @param {string} name    The name/key to set, such as an event or attribute name
 *  @param {any} old    The last value that was set for this name/node pair
 *  @param {any} value  An attribute value, such as a function to be used as an event handler
 *  @param {Boolean} isSvg  Are we currently diffing inside an svg?
 *  @private
 */
export function setAccessor(node, name, old, value, isSvg) {
  if (name === 'className') name = 'class'

  if (name === 'key') {
    // 如果是key属性，忽略
  } else if (name === 'ref') {
    // 如果是ref函数被改变了，以null去执行之前的ref函数，并以node节点去执行新的ref函数
    if (old) old(null)
    if (value) value(node)
  } else if (name === 'class' && !isSvg) {
    node.className = value || ''
  } else if (name === 'style') {
    // 对style属性进行处理
    if (!value || typeof value === 'string' || typeof old === 'string') {
      node.style.cssText = value || ''
    }
    if (value && typeof value === 'object') {
      if (typeof old !== 'string') {
        // 从dom的style中剔除已经被删除的属性
        for (let i in old) if (!(i in value)) node.style[i] = ''
      }
      for (let i in value) {
        // 一些数值的换算
        node.style[i] =
          typeof value[i] === 'number' && IS_NON_DIMENSIONAL.test(i) === false
            ? value[i] + 'px'
            : value[i]
      }
    }
  } else if (name === 'dangerouslySetInnerHTML') {
    // 关于dangerouslySetInnerHTML https://facebook.github.io/react/docs/introducing-jsx.html#jsx-prevents-injection-attacks
    // __html: 背后的目的是表明它会被当成 "type/taint" 类型处理。这种包裹对象，可以通过方法调用返回净化后的数据，随后这种标记过的数据可以被传递给 dangerouslySetInnerHTML
    if (value) node.innerHTML = value.__html || ''
  } else if (name[0] == 'o' && name[1] == 'n') {
    // 事件处理
    // 如果属性是以on开头，说明要绑定的是事件，因为Preact不同于React，并没有采用事件代理的机制，所有的事件都会被注册到真实的dom中。
    // 而且另一点与React不相同的是，如果你的事件名后添加Capture，例如onClickCapture，那么该事件将在dom的捕获阶段响应，默认会在冒泡事件响应。
    // 如果事件的名称是以Capture为结尾的，则去掉，并在捕获阶段节点监听事件
    let useCapture = name !== (name = name.replace(/Capture$/, '')) //事件捕获boolean
    name = name.toLowerCase().substring(2)
    // 如果value有值，则进行事件监听，否则取消监听
    if (value) {
      if (!old) node.addEventListener(name, eventProxy, useCapture)
    } else {
      node.removeEventListener(name, eventProxy, useCapture)
    }
    ;(node._listeners || (node._listeners = {}))[name] = value
  } else if (name !== 'list' && name !== 'type' && !isSvg && name in node) {
    // 尝试去设置属性
    setProperty(node, name, value == null ? '' : value)
    if (value == null || value === false) node.removeAttribute(name)
  } else {
    // 对svg的命名空间设置
    let ns = isSvg && name !== (name = name.replace(/^xlink\:?/, ''))
    if (value == null || value === false) {
      if (ns)
        node.removeAttributeNS(
          'http://www.w3.org/1999/xlink',
          name.toLowerCase()
        )
      else node.removeAttribute(name)
    } else if (typeof value !== 'function') {
      if (ns)
        node.setAttributeNS(
          'http://www.w3.org/1999/xlink',
          name.toLowerCase(),
          value
        )
      else node.setAttribute(name, value)
    }
  }
}
```

> 两个辅助函数。

```javascript
/** Attempt to set a DOM property to the given value.
 *  IE & FF throw for certain property-value combinations.
 */
// 在节点上设置属性
function setProperty(node, name, value) {
  try {
    node[name] = value
  } catch (e) {}
}

/** Proxy an event to hooked event handlers
 *  @private
 */
// 设置事件代理，这里与react的事件代理不同。
function eventProxy(e) {
  return this._listeners[e.type]((options.event && options.event(e)) || e)
}
```
