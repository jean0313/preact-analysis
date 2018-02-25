# <img src="preact-icon.png" width="32" height="32" /> index

```javascript
import { extend } from '../util'
```

> 检测节点类型是否等同。

```javascript
/**
 * Check if two nodes are equivalent.
 *
 * @param {Node} node			DOM Node to compare
 * @param {VNode} vnode			Virtual DOM node to compare
 * @param {boolean} [hydrating=false]	If true, ignores component constructors when comparing.
 * @private
 */
export function isSameNodeType(node, vnode, hydrating) {
  if (typeof vnode === 'string' || typeof vnode === 'number') {
    return node.splitText !== undefined
  }
  if (typeof vnode.nodeName === 'string') {
    return !node._componentConstructor && isNamedNode(node, vnode.nodeName)
  }
  return hydrating || node._componentConstructor === vnode.nodeName
}
```

> 判断节点 dom 的名称是否与给定的 nodeName 相同。

```javascript
/**
 * Check if an Element has a given nodeName, case-insensitively.
 *
 * @param {Element} node	A DOM Element to inspect the name of.
 * @param {String} nodeName	Unnormalized name to compare against.
 */
export function isNamedNode(node, nodeName) {
  return (
    node.normalizedNodeName === nodeName ||
    node.nodeName.toLowerCase() === nodeName.toLowerCase()
  )
}
```

> 提取出 vnode 的 props。

```javascript
/**
 * Reconstruct Component-style `props` from a VNode.
 * Ensures default/fallback values from `defaultProps`:
 * Own-properties of `defaultProps` not present in `vnode.attributes` are added.
 *
 * @param {VNode} vnode
 * @returns {Object} props
 */
export function getNodeProps(vnode) {
  let props = extend({}, vnode.attributes)
  props.children = vnode.children

  let defaultProps = vnode.nodeName.defaultProps
  if (defaultProps !== undefined) {
    for (let i in defaultProps) {
      if (props[i] === undefined) {
        props[i] = defaultProps[i]
      }
    }
  }

  return props
}
```
