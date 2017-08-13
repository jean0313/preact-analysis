处理render的队列
```
import options from './options';
import { defer } from './util';
import { renderComponent } from './vdom/component';

/** Managed queue of dirty components to be re-rendered */

let items = [];

// 主要用于延迟渲染当前组件
export function enqueueRender(component) {
    // 若component的_dirty为false，则将其赋值为true
    if (!component._dirty && (component._dirty = true) && items.push(component)==1) {
        (options.debounceRendering || defer)(rerender);
    }
}

export function rerender() {
    let p, list = items;
    items = [];
    while ( (p = list.pop()) ) {
        if (p._dirty) renderComponent(p);
    }
}
```