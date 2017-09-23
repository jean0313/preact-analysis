此处指定义了一个空函数来代表虚拟DOM元素。

使用时`new VNode()`来构造一个VNode对象。

VNode具体属性参考[h.md](./h.md)。

```javascript
/** Virtual DOM Node */
export function VNode() {}

// 结构如下所示
//function VNode(nodeName, attributes, children) {
//  this.nodeName =  nodeName
 // this.attributes = attributes
 // this.children = children
//}
```