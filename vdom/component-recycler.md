# <img src="preact-icon.png" width="32" height="32" /> component-recycler

组件复用。

```javascript
import { Component } from '../component'
```

> 为了能重用组件，保留一个缓存池。

```javascript
/** Retains a pool of Components for re-use, keyed on component name.
 *	Note: since component names are not unique or even necessarily available, these are primarily a form of sharding.
 *	@private
 */
const components = {}
```

> 收集一个组件以供以后重复利用，通过组件名分类将可重用的组件缓存在缓存池中。

```javascript
/** Reclaim a component for later re-use by the recycler. */
export function collectComponent(component) {
  let name = component.constructor.name
  ;(components[name] || (components[name] = [])).push(component)
}
```

> 创建一个组件。分成两种形式，一种是 PFC（纯函），另一种是继承 Component 的。

```javascript
/** Create a component. Normalizes differences between PFC's and classful Components. */
/**
 *	@param {object} Ctor    Ctor是需要渲染的组件
 *	@param {Object} props   属性，与react一致
 *	@param {Object} context 上下文，与react一致
 */
export function createComponent(Ctor, props, context) {
  let list = components[Ctor.name],
    inst

  // 如果是class元素
  if (Ctor.prototype && Ctor.prototype.render) {
    inst = new Ctor(props, context) // 创建组件实例
    Component.call(inst, props, context) // 给inst中props,context进行初始化操作
  } else {
    // 如果是纯函数组件（PFC）
    inst = new Component(props, context) // 直接调用函数Component创建实例，实例的constructcor属性设置为传入的函数
    inst.constructor = Ctor
    inst.render = doRender //将doRender函数作为实例的render属性，doRender函数会将Ctor的返回的虚拟dom作为结果返回。
  }

  // 其目的就是为了能基于此DOM元素进行渲染，以更少的代价进行相关的渲染。
  if (list) {
    for (let i = list.length; i--; ) {
      // 从组件回收的共享池中那拿到同类型组件的实例，从其中取出该实例之前渲染的实例(nextBase)，
      // 然后将其赋值到我们的新创建组件实例的nextBase属性上
      if (list[i].constructor === Ctor) {
        inst.nextBase = list[i].nextBase
        list.splice(i, 1)
        break
      }
    }
  }
  return inst
}

/** The `.render()` method for a PFC backing instance. */
function doRender(props, state, context) {
  return this.constructor(props, context)
}
```
