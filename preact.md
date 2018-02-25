# <img src="preact-icon.png" width="32" height="32" /> preact

## 导出所有的方法，其中包括：

* `h` => **用来把 JSX 转换成 virtual DOM 元素**
* `createElement` => **h 的别称**
* `cloneElement` => **克隆函数**
* `Component` => **基本组件类**
* `render` => **渲染函数**
* `rerender` => **将队列里的组件重新渲染**
* `options` => **用来设置全局选项，指定一些可选方法来让用户做出插件化使用**

```javascript
import { h, h as createElement } from './h'
import { cloneElement } from './clone-element'
import { Component } from './component'
import { render } from './render'
import { rerender } from './render-queue'
import options from './options'

export default {
  h,
  createElement,
  cloneElement,
  Component,
  render,
  rerender,
  options
}

export { h, createElement, cloneElement, Component, render, rerender, options }
```
