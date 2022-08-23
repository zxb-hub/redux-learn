# React Hooks - 理解组件重新渲染

当我开始研究 React Hooks 时，官方文档对核心概念的解释非常好 - [hooks 背后的动机](https://reactjs.org/docs/hooks-intro.html#motivation)、[详细的 api 参考](https://reactjs.org/docs/hooks-reference.html)、[hook 的规则](https://reactjs.org/docs/hooks-rules.html)、[回答几个常见问题](https://reactjs.org/docs/hooks-faq.html)。但是一旦我开始使用它们，我就感觉到我对组件生命周期的理解存在差距。有时我无法解释为什么我的组件会被重新渲染（[似乎是一个常见问题](https://www.google.com/search?q=react+hooks+rerender+site:stackoverflow.com)），什么变化导致了无限的重新渲染循环（[另一个常见的循环](https://www.google.com/search?q=react+hooks+infinite+re-render+site%3Astackoverflow.com)），有时我不确定我是否使用了 useCallback/useMemo 实际上是在提高性能。

使用 hook 的功能组件将经历一个生命周期，但没有显式的生命周期方法，这与[基于类](https://twitter.com/dan_abramov/status/981712092611989509)的对应物不同。虽然这种非显式性质允许使用更少的样板代码将可重用行为提取到自定义 hook 中，但我发现自己缺乏对 hook 如何影响执行流程的清晰理解。

这篇文章是我探索每个内置钩子如何影响组件重新渲染/生命周期的结果。 我将通过详细的示例分享我的经验，并强调我遇到的一些陷阱。 这是一篇很长的文章，涵盖了大多数内置钩子，并附有大量截图和示例。 如果你只对特定的钩子感兴趣，请直接从下面的链接访问。

## 目录

- [术语](#术语)

- [useState](#useState)

- [useEffect](#useEffect)

- [useLayoutEffect](#useLayoutEffect)

- [useContext](#useContext)

- [useCallback](#usecallback)

- [useMemo](#useMemo)

- [useReducer](#useReducer)

### 术语

在高层次上，React 在转换组件树并将结果刷新到渲染环境时会经历三个阶段：

（来自 Kent C. Dodds 的这篇文章）：

- “渲染”阶段：创建 React 元素 _`React.createElement`_（[了解更多](https://kentcdodds.com/blog/what-is-jsx)）
- “ reconciliation ”阶段：将以前的元素与新元素进行比较（[了解更多](https://reactjs.org/docs/reconciliation.html)）
- “commit” 阶段：更新 DOM（如果需要）。

现在，是什么促使 React 触发渲染阶段？ 这篇 [StackOverflow](https://stackoverflow.com/questions/55106951/react-with-hooks-when-do-re-renders-happen) 的帖子提供了一个简洁的答案。

- 组件接收新的 props。
- state 已更新。
- 更新上下文值（如果组件使用 useContext 监听上下文变化）。
- 由于上述任何原因，父组件会重新渲染。

这篇文章中每次提到“重新渲染”都是关于上面提到的“渲染阶段”。

#### 关于这篇文章中的例子：

- 我们将实现一个简单的股票报价器组件并围绕该组件添加功能以了解各种内置挂钩的行为。该组件显示股票代码及其价格。下拉菜单允许用户选择不同的代码。
  ![例子-1](./img/ReactHooks-1-%E4%BE%8B%E5%AD%90.png)
- 所有示例都在组件旁边显示执行日志。每当 React 重新渲染组件时，组件周围会出现一个虚线边框（作为视觉线索）。
  ![例子-2](./img/ReactHooks-1-%E4%BE%8B%E5%AD%902.png)
- 代码示例故意保持简单，主要关注执行流程（没有类型；没有后端调用；股票报价是模拟数据）。
- 当 React [发布并发模式](https://reactjs.org/docs/concurrent-mode-intro.html)时，某些示例中的行为可能会发生变化。

### useState

[useState](https://reactjs.org/docs/hooks-reference.html#usestate) hook 是主要的构建块，它使功能组件能够在重新渲染之间保持状态。

让我们通过一个例子来了解 useState 的工作原理。 我们将按照上面的草图实现 Ticker 组件。UI 上显示的活动代码使用 useState 挂钩存储在本地状态中。 这是组件的相关部分（[此处为完整代码](https://codesandbox.io/s/01a-usestate-single-61yn2)）。

```js
const [ticker, setTicker] = useState("AAPL");
...
const onChange = event => {
   setTicker(event.target.value);
}
...
```

让我们详细了解一下这段代码：

- useState 语句返回一个状态值（'ticker'）和一个 setter 函数来更新相应的状态值（'setTicker'）。
- 当组件第一次呈现时，“ticker” 值将是参数中指定的默认值（即“AAPL”）。
- 当从下拉列表中选择新代码时，将调用 onChange 函数。此函数使用 setter 函数更新 state。**此 state 更改会触发重新渲染**——调用 TickerComponent 函数再次执行。但这一次“useState(‘AAPL’)”返回了先前由 setter 函数设置的代码值。这几乎就像 React 将 state 存储在与我们组件实例相关的隐藏数据存储中。最新的状态值被保存并在重新渲染时返回。如果组件被卸载并重新安装，那么一切都会重新开始。

所有上述步骤都在下面的屏幕截图中可视化。安装后，Ticker 组件将使用默认代码（“AAPL”）呈现。下拉列表中的新代码选择会更新触发重新渲染的代码状态。请注意在代码选择之后突出显示的边框。执行日志将此步骤显示为“开始渲染”。
![useState](./img/ReactHooks-2-useState.gif)
这是日志的快照：
![日志快照](./img/ReactHooks-2-useState1.png)
该动画从代码的上下文中说明了组件的行为（步骤 1-12）。
![组件行为](./img/ReactHooks-2-useState2.gif)

#### 父组件中状态更新的影响

我们已经看到由于功能组件内的本地状态更改而导致组件重新渲染周期。会发生什么——当父组件和子组件拥有自己的本地状态（通过 useState）并且父组件的状态发生变化时？

让我们扩展 TickerComponent 以支持“主题” props。它的父组件（“App”）将主题值保存在一个状态中，并通过 props 将其传递给子组件。
![props传值](./img/ReactHooks-2-useState3.png)
（注意：在实际应用中，useContext 钩子更适合支持主题等功能，而不是通过组件树作为 props 向下传递值）

App 组件在其状态中包含“主题”变量，并有两个 Ticker 组件作为子组件。主题作为 props 传递给 TickerComponents。TickerComponent 使用 props.theme 并将值呈现为 css 类（这是对上一个示例中 TickerComponent 的唯一更改）。
![代码](./img/ReactHooks-2-useState4.png)

![运行](./img/ReactHooks-2-useState5.png)

让我们解析执行日志 --：

- 第 1 组：当 App 组件被挂载时，它被初始化为默认的“dark”主题，ticker 组件被初始化为默认的“AAPL”ticker。
- 第 2/3 组：在 Ticker 组件中都选择了新的代码，并且状态更改触发了本地重新渲染（如前面的示例）
- 第 4 组：现在在父组件中更改了主题值，父组件（App）和它的子组件（ticker 组件）都被重新渲染。

当 App 组件重新渲染时，其子组件将重新渲染，而不管它们是否使用主题值（作为 prop ）。查看这个[更新的示例](https://codesandbox.io/s/01c-usetstate-child-no-theme-dependency-3ig0z)，其中第二个代码组件不使用主题状态，而是在主题更改时重新呈现。这是 React 的默认行为，可以通过使用 [useMemo](https://medium.com/@guptagaruda/react-hooks-understanding-component-re-renders-9708ddee9928#204b) 钩子包装子组件来改变它，我们稍后会研究它。

#### 补充说明：

- 当使用某个 prop 值作为默认值（下面的片段）初始化状态变量时，prop 值仅在第一次创建状态时使用。对 prop 的任何进一步更新都不会自动“同步”组件的本地状态。这样的假设将导致错误的行为，请查看 Dan 关于该主题的出色文章，[以及 React 文档中的这篇文章](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)，以详细解释这种反模式和可能的解决方案。

```js
const [localState, setLocalState] = useState(props.theme);
```

- 当 setState 处理程序被多次调用时，React 会批处理这些调用，并且当调用代码位于基于 React 的事件处理程序中时，只会触发一次重新渲染。如果这些调用是从非基于 React 的处理程序（如 setTimeout）发出的，则每次调用都会触发重新渲染。 （请参阅有关此主题的[这篇文章](https://github.com/facebook/react/issues/14259#issuecomment-439632622)）

### useEffect

_[来自文档](https://reactjs.org/docs/hooks-reference.html#useeffect) - 函数组件的主体内不允许突变、订阅、计时器、日志记录和其他副作用（称为 React 的渲染阶段）。这样做会导致 UI 中出现令人困惑的错误和不一致。_

_相反，使用 `useEffect`。传递给 `useEffect` 的函数将在渲染提交到屏幕后运行。将效果视为从 React 的纯函数式世界到命令式世界的逃生口。_

_如果你熟悉 React 类的生命周期方法，你可以将 useEffect Hook 看作是 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 的组合。_

让我们用几个例子来理解所有这些陈述。

在第一个示例中——我们将使用一些日志语句简单地将 useEffect 处理程序添加到 TickerComponent。 目标是了解此函数何时在组件生命周期中执行。

```js
useEffect(() => {
  // logs 'useEffect inside handlers
  // logs the active ticker currently shown (by querying the DOM)
  return () => {
    // logs 'useEffect cleanup handler'
  };
});
```

下面是简化的代码（[完整版在这里](https://codesandbox.io/s/02a-useeffect-basic-tjy8o)）。执行日志是三个操作的结果：

- 挂载组件
- 将代码更改为“MSFT”
- 卸载组件
  ![组件](./img/ReactHooks-3-useEffect.png)

让我们仔细查看执行日志以了解发生了什么 -

#1, 2, 3：在安装时，组件使用默认代码 (‘AAPL’) 呈现。
#4: useEffect 处理程序在挂载后第一次运行 — **DOM 元素已经有了默认状态（‘AAPL’）。这意味着当 useEffect 处理程序运行时，React 完成了将组件状态同步到 DOM。**注意：清理处理程序尚未运行（因为之前没有运行 useEffect 处理程序调用）。
#5：从下拉列表中选择新代码（到“MSFT”）。
#6、7、8：状态改变。 使用新状态 (‘MSFT’) 重新渲染的组件。
#9：**从之前的 useEffect 调用返回的清理处理程序现在运行（注意：这里的代码值是“AAPL”，因为这个闭包是从之前的 useEffect 运行中创建并返回的）。**
#10：useEffect 处理程序再次运行，就像 DOM 已经有了新的代码状态（“MSFT”）之前一样。
#11, 12: 最后，当组件被卸载时，React 确保运行与最后一个效果相关的清理处理程序（对于代码‘MSFT’）。

我们从这个例子中学到了两件事：

- React 在将当前组件状态完全同步到 DOM 后运行 useEffect 处理程序。这使它成为启动副作用（后端调用、注册订阅、日志记录、计时器等）的理想场所。
- useEffect 处理程序（清理处理程序）返回的函数在 useEffect 处理程序下一次运行之前（或卸载之后）运行。这是进行与副作用相关的清理操作（取消订阅/分离事件处理程序等）的理想场所。这对于避免内存泄漏也很重要。由于“清理”处理程序是一个闭包，它会捕获函数创建和返回时的状态，即使该函数在下一次重新渲染中执行，事情也会自然而然地工作（从日志中检查步骤 #9 和 #12 — 清理处理程序中的状态值来自较早的迭代）。

让我们将所有这些概念与一个具体的例子联系起来。 我们希望更新代码组件，使其不断刷新活动代码的最新报价。我们将使用（模拟）流式股票报价服务。 该 API 允许为每几秒调用一次当前报价的代码注册回调。它返回一个清理处理程序，当调用 API 时停止执行代码的回调。

下面是精简的代码片段（[完整版](https://codesandbox.io/s/02c-useeffect-service-lvgnb)在这里）。每个价格变化通知都会在价格上执行一个 setState，从而触发重新渲染。当活动代码更改时，您可以看到清理处理程序针对前一个代码运行。

![]() ![]()

让我简要回顾一下 useEffect 调用中的依赖数组（第二个参数）。当省略此参数时，React 将在每次重新渲染后执行 useEffect 处理程序（如 useEffect 中的第一个示例）。大多数时候这将是低效的。指定依赖项后，React 将仅在该列表中的任何依赖项发生更改时运行 useEffect 处理程序。

大多数情况下，无限重新渲染是未正确配置依赖项列表的结果。 例如 如果您将函数引用添加为依赖项，并且如果每次重新渲染时引用都会发生变化（重新创建函数），那么 useEffect 会在每次重新渲染时运行，并且此处理程序中的状态更改会导致重新渲染并且循环重复 导致无限渲染循环。

### useLayoutEffect

### useContext

### useCallback

### useMemo

### useReducer
