**h**函数接受了两个主要参数以及多个其他的参数作为子节点，返回一个虚拟DOM元素。
相当于react中的react.createElement,来创建虚拟dom,也就是一个js的对象
结构如下:
```javascript
{
  nodeName:'div',
  attributes:{'id':'preact'},
  children:['preactChild']
}
```

```javascript
import { VNode } from './vnode'; // 引入VNode
import options from './options'; // 引入全局选项


const stack = []; // 用来存放除nodeName和attributes之外的参数

const EMPTY_CHILDREN = []; // 用来初始化children，在下方的h函数内的第一行

/** JSX/hyperscript reviver
 *	Benchmarks: https://esbench.com/bench/57ee8f8e330ab09900a1a1a0
 *	@see http://jasonformat.com/wtf-is-jsx
 *	@public
 */
export function h(nodeName, attributes) {
   // 其实这里这要的操作就是把除了nodeName, attributes后面的child做成数组children,将nodeName,attributes以及children,key放到vnode中，返回一个虚拟对象
  let children=EMPTY_CHILDREN, lastSimple, child, simple, i;
  for (i=arguments.length; i-- > 2; ) {
    stack.push(arguments[i]); // 先把除了nodeName和attributes之外所有的参数扔到stack里面
  }
  /** 
   * 如果attributes定义了children属性并且当前stack为空时，stack插入children
   * 随后删除children属性
   * 这个步骤是针对这样的方式
   *
   * h('div', {
   *   class: 'foo',
   *   children: h('div', {
   *     class: 'bar'
   *   }, 'content: bar')
   * })
   */ 
   if (attributes && attributes.children!=null) {
     if (!stack.length) stack.push(attributes.children);
     delete attributes.children;
   }
   /**
    * 以下部分主要对stack进行处理，对所有子节点进行遍历
    */
   while (stack.length) {
     // 首先判断该元素是不是数组类型，这里通过是否含有函数pop去判别是否是个数组，如果子元素是数组，将其全部压入栈中
     if ((child = stack.pop()) && child.pop!==undefined) {
       // 这里考虑的是如果child为数组的情况，则依次推入stack。注意：每次循环都会进行child = stack.pop()操作
       for (i=child.length; i--; ) stack.push(child[i]);
     }
     else {
       // 因为子元素是不支持布尔类型的，如果child是布尔类型，则将child置为null
       if (typeof child==='boolean') child = null;
       /* 这里判断nodeName是否为function,并赋值给simple,若
        * 不为function,则认为simple为true，我们不妨设为简单类型和非简单类型。
        */
       if ((simple = typeof nodeName!=='function')) {
         if (child==null) child = '';
         else if (typeof child==='number') child = String(child); // 将数字转换为string
         else if (typeof child!=='string') simple = false; // 如果child非字符串,simple则为false，即为非简单类型
       }

       if (simple && lastSimple) {
         // 如果本次跟上一次循环都是简单类型，则将字符串拼接
         children[children.length-1] += child;
       }
       else if (children===EMPTY_CHILDREN) {
         children = [child]; // 首次赋值
       }
       else {
         children.push(child); //最后将处理的子节点传入数组children中，现在传入children中的节点有三种类型：纯字符串、代表dom节点的字符串以及代表组件的函数
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
   if (options.vnode!==undefined) options.vnode(p);

   return p;
}
```
