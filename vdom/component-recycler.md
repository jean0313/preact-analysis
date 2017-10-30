```javascript
import { Component } from '../component';

/** Retains a pool of Components for re-use, keyed on component name.
 *	Note: since component names are not unique or even necessarily available, these are primarily a form of sharding.
 *	@private
 */
// 主要作用是为了能重用组件渲染的内容而设置的共享池
const components = {};


/** Reclaim a component for later re-use by the recycler. */
// 回收一个组件以供以后重复利用，通过组件名分类将可重用的组件缓存在缓存池中
export function collectComponent(component) {
	let name = component.constructor.name;
	(components[name] || (components[name] = [])).push(component);
}


/** Create a component. Normalizes differences between PFC's and classful Components. */

/** 是创建组件实例
 *	@param {Object} props	与react一致
 *	@param {Object} context 与react一致
 *	@param {object}  Ctor   Ctor组件是需要创建的组件类型（函数或者是类）
 */
export function createComponent(Ctor, props, context) {
	let list = components[Ctor.name],
		inst;
	// 如果是class元素
	if (Ctor.prototype && Ctor.prototype.render) {
		inst = new Ctor(props, context);	// 创建Ctor实例
		Component.call(inst, props, context);  // 给inst中props,context进行初始化赋值
	}
	// 如果是纯函数组件（PFC）
	else {
		inst = new Component(props, context); // 直接调用函数Component创建实例，实例的constructcor属性设置为传入的函数
		inst.constructor = Ctor; 
		inst.render = doRender;  //将doRender函数作为实例的render属性，doRender函数会将Ctor的返回的虚拟dom作为结果返回。
	}

	// 其目的就是为了能基于此DOM元素进行渲染，以更少的代价进行相关的渲染。
	if (list) {
		for (let i=list.length; i--; ) {
			// 从组件回收的共享池中那拿到同类型组件的实例，从其中取出该实例之前渲染的实例(nextBase)，然后将其赋值到我们的新创建组件实例的nextBase属性上
			if (list[i].constructor===Ctor) {
				inst.nextBase = list[i].nextBase;
				list.splice(i, 1);
				break;
			}
		}
	}
	return inst;
}


/** The `.render()` method for a PFC backing instance. */
function doRender(props, state, context) {
	return this.constructor(props, context);
}
```