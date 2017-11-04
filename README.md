# preact 源码解析

此时[preact](https://github.com/developit/preact)的版本为8.2.1。

如何使用preact请参考[官网指导](https://preactjs.com/guide/getting-started)。

如有发现存在分析错误或不严谨之处，请前往**issue**提出。

## Source Code Tree

```
dom/
  - index.js
vdom/
  - component-recycler.js
  - component.js
  - diff.js
  - index.js
clone-element.js
component.js
h.js
options.js
preact.js
render-queue.js
render.js
util.js
vnode.js
```

## Reference

在分析中借鉴了如下，再次表示感谢：
* [从Preact了解一个类React的框架是怎么实现的(一): 元素创建](https://juejin.im/post/59b69b6e5188257e6b6d7bfc)
* [从Preact了解一个类React的框架是怎么实现的(二): 元素diff](https://juejin.im/post/59c76e515188254f584132af)

## Contributors

- [g1eny0ung](https://github.com/g1eny0ung)
- [PerkinJ](https://github.com/PerkinJ)
