# <img src="preact-icon.png" width="32" height="32" /> component

基本组件class。

```javascript
import { FORCE_RENDER } from './constants';
import { extend } from './util';
import { renderComponent } from './vdom/component';
import { enqueueRender } from './render-queue';

/** Base Component class.
 *  Provides `setState()` and `forceUpdate()`, which trigger rendering.
 *  @public
 *
 *  @example
 *  class MyFoo extends Component {
 *      render(props, state) {
 *          return <div />;
 *      }
 *  }
 */
export function Component(props, context) {
    this._dirty = true;  // 表示存在脏数据，即数据与现有的对应不一致

    /** @public
     *  @type {object}
     */
    this.context = context;

    /** @public
     *  @type {object}
     */
    this.props = props;

    /** @public
     *  @type {object}
     */
    this.state = this.state || {};
}
```

```javascript
extend(Component.prototype, {
```

> shouldComponentUpdate的表示

```javascript
    /** Returns a `boolean` indicating if the component should re-render when receiving the given `props` and `state`.
     *  @param {object} nextProps
     *  @param {object} nextState
     *  @param {object} nextContext
     *  @returns {Boolean} should the component re-render
     *  @name shouldComponentUpdate
     *  @function
     */
```

> setState的表示, 最后的重新渲染是异步的

```javascript
    /** Update component state by copying properties from `state` to `this.state`.
     *  @param {object} state       A hash of state properties to update with new values
     *  @param {function} callback  A function to be called once component state is updated
     */
    setState(state, callback) {
        let s = this.state;
        if (!this.prevState) this.prevState = extend({}, s);
        // 判断state时function时的情况
        extend(s, typeof state==='function' ? state(s, this.props) : state);
        // 将callback函数加入队列中去
        if (callback) (this._renderCallbacks = (this._renderCallbacks || [])).push(callback);
        enqueueRender(this); // 将组件放入渲染队列中
    },
```

> 强制更新

```javascript
    /** Immediately perform a synchronous re-render of the component.
     *  @param {function} callback      A function to be called after component is re-rendered.
     *  @private
     */
    forceUpdate(callback) {
        if (callback) (this._renderCallbacks = (this._renderCallbacks || [])).push(callback);
        renderComponent(this, FORCE_RENDER);
    },
```

> render的表示

```
    /** Accepts `props` and `state`, and returns a new Virtual DOM tree to build.
     *  Virtual DOM is generally constructed via [JSX](http://jasonformat.com/wtf-is-jsx).
     *  @param {object} props       Props (eg: JSX attributes) received from parent element/component
     *  @param {object} state       The component's current state
     *  @param {object} context     Context object (if a parent component has provided context)
     *  @returns VNode
     */
    render() {}

});
```
