指定不同的渲染模式
```
// render modes

export const NO_RENDER = 0;     // 表示没有渲染
export const SYNC_RENDER = 1;   // 同步渲染
export const FORCE_RENDER = 2;  // 强制更新
export const ASYNC_RENDER = 3;  // 组件异步更新


export const ATTR_KEY = '__preactattr_';  // 要添加的属性

// DOM properties that should NOT have "px" added when numeric
export const IS_NON_DIMENSIONAL = /acit|ex(?:s|g|n|p|$)|rph|ows|mnc|ntw|ine[ch]|zoo|^ord/i;
```