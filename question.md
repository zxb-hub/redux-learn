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

6.
