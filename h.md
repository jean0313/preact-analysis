# <img src="preact-icon.png" width="32" height="32" /> h

**h**函数接受了两个主要参数以及多个其他的参数作为子节点，返回一个虚拟 DOM 元素。它相当于 React 中的[createElement](https://reactjs.org/docs/react-api.html#createelement)。返回一个 VNode 的实例，结构如下：

```javascript
{
  nodeName, attributes, children
}
// 也可能会携带key
```

它的作用是为 JSX 语法提供转换，例如添加`/** @jsx h */`的注释来  让编译器将 JSX 语法转换为 h 函数的调用。

## 解析

```javascript
import { VNode } from './vnode' // 引入VNode
import options from './options' // 引入全局选项

const stack = [] // 用来存放除nodeName和attributes之外的参数

const EMPTY_CHILDREN = [] // 用来初始化children，在下方的h函数内的第一行
```

```javascript
/**
 * JSX/hyperscript reviver.
 * @see http://jasonformat.com/wtf-is-jsx
 * Benchmarks: https://esbench.com/bench/57ee8f8e330ab09900a1a1a0
 *
 * Note: this is exported as both `h()` and `createElement()` for compatibility reasons.
 *
 * Creates a VNode (virtual DOM element). A tree of VNodes can be used as a lightweight representation
 * of the structure of a DOM tree. This structure can be realized by recursively comparing it against
 * the current _actual_ DOM structure, and applying only the differences.
```

> 创建一个 VNode（虚拟 DOM 元素）。多个 VNode 组成的树可以被当作一个轻量级的 DOM 树的表示。通过与当前它的真实的 DOM 结构递归比较，这个结构得以被识别出来，并且仅仅比较不同的地方。

```javascript
 * `h()`/`createElement()` accepts an element name, a list of attributes/props,
 * and optionally children to append to the element.
```

> `h()`/`createElement()` 接受一个元素名，一个 attributes 或者 props 的列表，还可以接受可选的子元素。

```javascript
 * @example The following DOM tree
 *
 * `<div id="foo" name="bar">Hello!</div>`
 *
 * can be constructed using this function as:
 *
 * `h('div', { id: 'foo', name : 'bar' }, 'Hello!');`
 *
 * @param {string} nodeName	An element name. Ex: `div`, `a`, `span`, etc.
 * @param {Object} attributes	Any attributes/props to set on the created element.
 * @param rest		 Additional arguments are taken to be children to append. Can be infinitely nested Arrays.
 *
 * @public
 */
export function h(nodeName, attributes) {

  // 其实整个函数主要的操作就是把除了nodeName, attributes后面的child做成数组children，
  // 将nodeName，attributes以及children，key放到vnode中，返回一个虚拟对象

  let children=EMPTY_CHILDREN, lastSimple, child, simple, i;
  for (i=arguments.length; i-- > 2; ) {
    stack.push(arguments[i]); // 先把除了nodeName和attributes之外所有的参数扔到stack里面, 倒着放
  }
```

```javascript
if (attributes && attributes.children != null) {
  if (!stack.length) stack.push(attributes.children)
  delete attributes.children
}
```

> 如果 attributes 定义了 children 属性并且当前 stack 为空时，stack 插入 children, 随后删除 children 属性，这个步骤是针对这样的方式：

```javascript
h('div', {
  class: 'foo',
  children: h(
    'div',
    {
      class: 'bar'
    },
    'content: bar'
  )
})
```

> 以下部分主要对 stack 进行处理，对所有子节点进行遍历

```javascript
   while (stack.length) {
     // 首先判断该元素是不是数组类型，这里通过是否实现pop方法去判别是否为数组，如果子元素是数组，将其全部压入栈中
     // 注意：每次循环都会进行child = stack.pop()操作
     if ((child = stack.pop()) && child.pop!==undefined) {
       for (i=child.length; i--; ) stack.push(child[i]);
     }
     else {
       // 因为子元素是不支持布尔类型的，如果child是布尔类型，则将child置为null
       if (typeof child==='boolean') child = null;
       /**
        * 这里判断nodeName是否为function,并赋值给simple,若
        * 不为function,则认为simple为true，我们不妨设为简单类型和非简单类型。
        */
       if ((simple = typeof nodeName!=='function')) {
         if (child==null) child = ''; // 如果为null，就设置为空字符
         else if (typeof child==='number') child = String(child); // 将数字转换为string
         else if (typeof child!=='string') simple = false; // 如果child非字符串，simple则为false，即为非简单类型
       }

       if (simple && lastSimple) {
         // 如果本次跟上一次循环都是简单类型，则将字符串拼接
         children[children.length-1] += child;
       }
       else if (children===EMPTY_CHILDREN) {
         children = [child]; // 首次赋值
       }
       else {
         // 最后将处理的子节点传入数组children中，现在传入children中的节点有三种类型：纯字符串、代表dom节点的字符串以及代表组件的函数
         children.push(child);
       }
       lastSimple = simple;
     }
   }

   let p = new VNode();  // 请查看vnode.md文件
   p.nodeName = nodeName; // 虚拟DOM的节点名称
   p.children = children; // 虚拟DOM所有的孩子们
   p.attributes = attributes==null ? undefined : attributes; // 所有的属性
   p.key = attributes==null ? undefined : attributes.key; // 指定key

   // if a "vnode hook" is defined, pass every created VNode to it
   // 如果vnode的钩子函数存在，每次创建VNode时会调用
   if (options.vnode!==undefined) options.vnode(p);

   return p;
}
```
