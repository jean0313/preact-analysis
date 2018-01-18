# <img src="preact-icon.png" width="32" height="32" /> render-queue

**render-queue**处理需要render的组件的队列。

```javascript
import options from './options';
import { defer } from './util';
import { renderComponent } from './vdom/component';

/** Managed queue of dirty components to be re-rendered */

let items = [];

// 异步渲染组件队列
export function enqueueRender(component) {
    // 若component的_dirty为false，则将其赋值为true
    if (!component._dirty && (component._dirty = true) && items.push(component)==1) {
        // 如果定义了debounceRendering（防抖render？），或者使用defer来异步重新渲染
        (options.debounceRendering || defer)(rerender);
    }
}

// rerender函数就是将items中待更新的组件，逐个取出，并对其执行renderComponent。
export function rerender() {
    let p, list = items;
    items = [];
    while ( (p = list.pop()) ) {
        if (p._dirty) renderComponent(p);
    }
}
```
