1. `store` 和 `state` 定义和应用场景

   store 一般用于全局状态管理，用于存储 state，action，reducers，其职责如下：

   - 在内部保存当前应用程序 state
   - 通过 `store.getState()` 访问当前 state;
   - 通过 `store.dispatch(action)` 更新状态;
   - 通过 `store.subscribe(listener)` 注册监听器回调;
   - 通过 `store.subscribe(listener)` 返回的 `unsubscribe` 函数注销监听器。

   Redux 应用程序中只有一个 store

   state 一般用于组件内部状态管理

2. redux 三大原则

   - 单一数据源
   - State 只读
   - 使用纯函数修改

3. useEffect 使用场景及注意事项
   注册监听时要注意清除监听，其他如定时器等消耗性能的也要在 return 中进行清除

4. 组件抽离，尽量避免组件状态和外部状态联系

5. memo 和 useMemo

   默认情况下，memo 只会浅浅地比较 props 对象中的复杂对象。如果要控制比较，还可以提供自定义比较函数作为第二个参数。

   useMemo 只需要把依赖项作为第二参数。

   当组件内部没使用其他 hooks 时，可以直接使用 memo，不用使用 useMemo，效果相同，区别为多走了一次 useMemo

6. useCallback 第二个参数是确保回调中使用数据为最新数据，数据改变不会触发回调函数；需要监听值变化跟新时，使用 useEffect

7. useEffect 和 useLayoutEffect 区别：

- useEffrct 可以理解为异步执行，在触发时会分出一个子线程进行执行回调函数，可能会在本次渲染之前完成，也可能在渲染之后完成，但是最终还是会重新渲染一次

- useLayoutEffect 为同步执行，只有执行完之后才会触发渲染。

8. useCallback 和 useMemo 区别和共通点

- 共通点：都是通过避免重复渲染进行提升性能，只有在依赖项发生改变时才会触发重新渲染/计算

- 区别：

  useCallback 可在回调函数中返回 jsx 模版，当依赖数组改变时才会重新渲染此模版

  useMemo 回调返回数值，可用于复杂项计算，避免重复计算浪费性能
