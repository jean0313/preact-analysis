# <img src="preact-icon.png" width="32" height="32" /> Preact 源码解析

[https://sinkmind.github.io/preact-analysis/](https://sinkmind.github.io/preact-analysis/)

此时[Preact](https://github.com/developit/preact)的版本为8.2.7。

如何使用Preact请参考[官网指导](https://preactjs.com/guide/getting-started)。

如有发现存在分析错误或不严谨之处或者其他补充，请前往[issue](https://github.com/sinkmind/preact-analysis/issues)提出，感谢大家。

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
constants.js
h.js
options.js
preact.js
render-queue.js
render.js
util.js
vnode.js
```

## Reference

在分析中借鉴了如下，在此表示感谢：
* [从Preact了解一个类React的框架是怎么实现的(一): 元素创建](https://juejin.im/post/59b69b6e5188257e6b6d7bfc)
* [从Preact了解一个类React的框架是怎么实现的(二): 元素diff](https://juejin.im/post/59c76e515188254f584132af)

## Contributors

- [g1eny0ung](https://github.com/g1eny0ung)
- [PerkinJ](https://github.com/PerkinJ)
