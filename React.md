## 基础知识

### 组件及传值

- props 和 State

  - props 为上级组件传入参数，State 为自身管理的状态

  - props 不可修改，只读属性；State 需要使用`this.setState()`方法进行更新组件 State

    ```js
    // wrong
    this.setState({
      counter: this.state.counter + this.props.increment,
    });

    // correct
    this.setState((state, props) => {
      counter: state.counter + props.increment;
    });
    ```

  - state 的更新可能是异步，由于性能考虑，可能会合并多个 setState 为一个调用，所以不要依赖他们的值来更新下一个状态。

  - 避免将 props 的值复制给 state，只有在你刻意忽略 prop 更新的情况下使用，参考[派生 state 出现的问题](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

- 组件间传值：

  > 不要尝试修改 props，react 组件必须保护他们的 props 不被更改

  ```js
  function c() {
    console.log("点击了子组件");
  }
  function a() {
    const string = "需要传递的值";
    return <b name={string} bClick={c} />;
  }

  function b(props) {
    return <div onClick={props.onclick}>{props.name}</div>;
  }
  ```

  子组件有专有的属性名（children），可以直接在标签之间使用{props.children}进行渲染，类似于 vue 中的插槽。

- jsx 注入攻击

  jsx 中可以安全的插入用户输入内容，React DOM 在渲染输入内容前，默认会进行转义，会将内容转化为字符串，可以有效地防止 XSS 攻击

- 生命周期及相关钩子

  只存在于 class 类创建的组件才会有生命周期，因为 class 组件会创建对应的实例，而函数组件不会。

  一次状态改变导致页面视图改变，会经历两个阶段： render 阶段和 commit 阶段。

  render 阶段进行 DOM 节点的 diff，找出改变的 DOM 操作；

  在 commit 阶段将对应的 DOM 操作提交到视图中；

  - Mount：constructor、render、componentDidMount（Commit 阶段）
  - Update：New props、setState()、forceUpdate()、render、componentDidUpdate(Commit 阶段)
  - Unmount：componentWillUnmount(commit 阶段)

  当组件实例被创建并插入 DOM 中时，生命周期调用顺序：
  constructor -> static getDerivedStateFromProps() -> render() -> componentDidMount()

  `forceUodate(callback)`: 默认情况下，当组件的 state 或 props 发生变化时，组件将重新渲染。如果 render() 方法依赖于其他数据，**_则可以调用 forceUpdate() 强制让组件重新渲染_**。调用 forceUpdate() 将致使组件调用 render() 方法，此操作会跳过该组件的 shouldComponentUpdate()。但其子组件会触发正常的生命周期方法，包括 shouldComponentUpdate() 方法。如果标记发生变化，React 仍将只更新 DOM。通常你应该避免使用 forceUpdate()，尽量在 render() 中使用 this.props 和 this.state。

  [生命周期函数](https://zh-hans.reactjs.org/docs/react-component.html#displayname)

- 事件处理
  在 React 中另一个不同点是你不能通过返回 false 的方式阻止默认行为。你必须显式地使用 preventDefault。例如，传统的 HTML 中阻止表单的默认提交行为：

  ```html
  <form onsubmit="console.log('You clicked submit.'); return false">
    <button type="submit">Submit</button>
  </form>
  ```

  在 react 中，可能是这样的：

  ```jsx
  function Form() {
    function handleSubmit(e) {
      e.preventDefault();
      console.log("You clicked submit.");
    }

    return (
      <form onSubmit={handleSubmit}>
        <button type="submit">Submit</button>
      </form>
    );
  }
  ```

  [相关合成事件查询](https://zh-hans.reactjs.org/docs/events.html)

- 组件开发

  - 在组件的 render 中返回 null 并不会影响组件的生命周期

  - 可以利用数组循环遍历生成标签，使用{}直接插入，注意需要有唯一标识 key，以帮助 React 识别哪些元素改变了

  - jsx 允许在大括号中嵌入任何表达式，可以在大括号中使用 map 等循环，如果循环嵌套过多就需要拆分组件了

  - 子组件不仅可以使用 children，也可自定义
    ```jsx
    function SplitPane(props) {
      return (
        <div className="SplitPane">
          <div className="SplitPane-left">{props.left}</div>
          <div className="SplitPane-right">{props.right}</div>
        </div>
      );
    }
    function App() {
      return <SplitPane left={<Contacts />} right={<Chat />} />;
    }
    ```
    `<Contacts />` 和 `<Chat />` 之类的 React 元素本质就是对象（object），所以你可以把它们当作 props，像其他数据一样传递

- Hooks

  - `useState`:

    返回一个 state 以及更新 state 的函数，setState 函数用于更新 state，并会将组件进行重新渲染

  - `useRef`:

    返回一个全局唯一属性，也可以定义 dom 元素

  - `useEffect`:

    可以在此处执行副作用操作，数据获取，设置订阅以及手动更改 React 组件中的 DOM 都属于副作用，在 React 更新 DOM 之后运行一些额外的代码。比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。当需要清除副作用时，将清除副作用的操作作为返回值 return

    传递两个参数，callback 和依赖，当依赖改变时会出发 callback 回调，在初始化渲染时也会执行一遍，如果依赖为空，则可以理解为挂载和卸载时触发;可以有返回值，返回函数在本次执行完到下次执行函数体之前触发；

  - `memo`：

    将此组件变为记忆组件，会跳过渲染组件的操作并直接服用最近一次渲染结果

  - `useMemo`:

    把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算

  - `useCallback`:

    把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。在嵌套组件中，可以使用这个 hook 进行规避渲染，

    ```js
    <a>
      <b data={data1} />
      <c data={data2} />
    </a>
    ```

    当 data2 改变时，会使`a`标签重新渲染，当使用 useCallback 包裹`b`组件时，data2 改变时，`b`不会重新渲染

    > useCallback(fn, deps) 相当于 useMemo(() => fn, deps)。

    > 依赖项数组不会作为参数传给回调函数。虽然从概念上来说它表现为：所有回调函数中引用的值都应该出现在依赖项数组中

  - `useContext`:

    使用`createContext`进行创建上下文，`useContext`进行使用

    `createContext`创建一个 Context 对象，当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 Provider 中读取到当前的 context 值。

    只有当组件所处的树中没有匹配到 Provider 时，其 defaultValue 参数才会生效

    xxx.Provider：Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据

    使用`useState`方法定义上下文，在修改时可以触发重新渲染

## Hook 相关知识
