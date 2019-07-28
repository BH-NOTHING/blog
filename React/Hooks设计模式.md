### why Hooks?

一: `多个组件间逻辑复用`: 在 Class 中使用 React 不能将带有 state 的逻辑给单独抽离成 function, 其只能通过嵌套组件的方式来解决多个组件间逻辑复用的问题, 基于嵌套组件的思想存在 [HOC](https://github.com/MuYunyun/blog/blob/master/React/从0到1实现React/8.HOC探索.md) 与 `render props` 两种设计模式。但是这两种设计模式是否存在缺陷呢?

* 嵌套地狱, 当嵌套层级过多后, 数据源的追溯会变得十分困难, 导致定位 bug 不容易; (hoc、render props)
* 性能, 需要额外的组件实例存在额外的开销; (hoc、render props)
* 命名重复性, 在一个组件中同时使用多个 hoc, 不排除这些 hoc 里的方法存在命名冲突的问题; (hoc)

二: `单个组件中的逻辑复用`: Class 中的生命周期 `componentDidMount`、`componentDidUpdate` 甚至 `componentWillUnMount` 中的大多数逻辑基本是类似的, 必须拆散在不同生命周期中维护相同的逻辑对使用者是不友好的, 这样也造成了组件的代码量变多。

三: Class 的其它一些问题: 在 React 使用 Class 需要书写大量样板, 用户通常会对 Class 中 Constructor 的 bind 以及 this 的使用感到困惑, 同时 React Team 表示 Class 在机器编译优化方面也不是很理想。

#### React Logo 与 Hooks 的小彩蛋

![](http://with.muyunyun.cn/ddbdcec2fc39ba350fc74647f4fad6f5.jpg-300)

React 的 logo 是一个原子图案, 原子组成了物质的表现, React 组成了页面的表现; 而 Hooks 就如夸克, 其一直都在, 但是直到 4 年后的今天才被设计出来。 —— by Dan in React Conf(2018)

### 为什么 useState 返回一个数组而非一个对象?

因为数组比对象更加方便, 可以观察如下

数组:

```js
[name, setName] = useState('鸣人')
[age, setAge] = useState(13)
```

对象:

```js
{value: name, setValue: setName} = useState('鸣人')
{value: name, setValue: setName} = useState(13)
```

### Hooks 传递的设计

Hooks 是否可以设计成在组件中通过函数传递, 比如像下面这样使用:

```js
const SomeContext = require('./SomeContext)

function Example({ someProp }, hooks) {
  const contextValue = hooks.useContext(SomeContext)
  return <div>{someProp}{contextValue}</div>
}
```

使用传递的劣势是在有时会出现冗余的传递。(待补充)

### Hooks 与 Class 中关于 setState 不同的表现差异

Hooks 中的 setState 与 Class 中最大区别在于 Hooks 不会对多次 setState 进行合并操作。如果要执行合并操作, 可像如下操作:

```js
setState(prevState => {
  return { ...prevState, ...updateValues }
})
```

此外关于 `setState` 的异步表现, 见如下表:

| | Class | Hooks |
|:---:|:---:|:---:|
| setState(async) | 多次输出 | 多次输出, 但是输出次数会小于等于 Class 的 |
| setState(sync) | 多次输出 | 单次输出 |

![](http://with.muyunyun.cn/314d5035e996809ab463e33e5029777f.jpg)

- [ ] [一些异步](https://codesandbox.io/s/funny-mclean-6lru4)

### Hooks 状态管理

useRedux:

```js
import * as actions from './actions'

function Counter() {
  const [count, {increment, decrement}] = useRedux(state => state.count, actions);

  return (
    <>
      Count: {count}
      <button onClick={() => increment()}>+</button>
      <button onClick={() => decrement()}>-</button>
    </>
  );
}
```

useReducer:

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter({initialState}) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
```

使用 hooks 实现自定义版本的 redux

### Hooks 中如何获取之前的 props 以及 state

React 官方在未来很可能会提供一个 `usePrevious` 的 hooks 来获取之前的 props 以及 state。

`usePrevious` 的核心思想是用 ref 来存储先前的值。

```js
function usePrevous(value) {
  const ref = useRef()
  useEffect(() => {
    ref.current = value
  })
  return ref.current
}
```

### Hooks 中如何调用实例上的方法

Hooks 中 something.current(a ref value) 的含义等价于在 Class 中使用 this.something。

```js
/* in a function */
const X = useRef()
X.current // can read or write

/* in a Class */
this.X // can read or write
```

> [twitter](https://twitter.com/dan_abramov/status/1125223181701263360)
> [Is there something like instance variables](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables)

### Hooks 中 getDerivedStateFromProps 的替代方案

在 [React 暗器百解](./React暗器百解.md) 中提到了 getDerivedStateFromProps 是一种反模式, 但是极少数情况还是用得到该钩子, 在 Hooks 中如何达到 getDerivedStateFromProps 的效果呢?

```js
function ScrollView({row}) {
  const [isScrollingDown, setISScrollingDown] = setState(false)
  const [prevRow, setPrevRow] = setState(null)

  // 核心是创建一个 `prevRow` state 与父组件传进来的 `row` 进行比较
  if (row !== prevRow) {
    setISScrollingDown(prevRow !== null && row > prevRow)
    setPrevRow(row)
  }

  return `Scrolling down ${isScrollingDown}`
}
```

### Hooks 中 forceUpdate 的替代方案

可以使用 `useReducer` 来 hack `forceUpdate`, 但是尽量避免 forceUpdate 的使用。

```js
const [ignored, forceUpdate] = useReduce(x => x + 1, 0)

function handleClick() {
  forceUpdate()
}
```

### Hooks 中 shouldComponentUpdate 的替代方案

在 Hooks 中可以使用 `useMemo` 来作为 `shouldComponentUpdate` 的替代方案, 但 `useMemo` 只对 props 进行浅比较。

```js
React.useMemo((props) => {
  // your component
})
```

#### useMemo 与 useCallback 的区别

```js
// 缓存 value
useMemo(() => value) <==> useCallback(value)
```

* useCallback: 一般用于缓存函数
* useMemo: 一般用于缓存组件

#### 依赖列表中移除函数是否是安全的?

通常来说, 结论是不安全的。可以观察 demo,

```js
const { useState, useEffect } = React

function Example({ someProp }) {
  function doSomething() {
    console.log(someProp) // 这里只输出 1, 点击按钮的 2 并没有输出。
  }

  useEffect(
    () => {
      doSomething()
    },
    [] // 🔴 This is not safe (it calls `doSomething` which uses `someProp`)
  )

  return <div>example</div>
}

export default function() {
  const [value, setValue] = useState(1)
  return (
    <>
      <Example someProp={value} />
      <Button onClick={() => setValue(2)}>button</Button>
    </>
  )
}
```

在该 demo 中, 点击 button 按钮, 并没有打印出 2。解决上述问题有两种方法。

方法一: 将函数放入 `useEffect` 中, 同时将相关属性放入依赖项中。因为在依赖中改变的相关属性一目了然, 所以这也是首推的做法。

```js
function Example({ someProp }) {
  useEffect(
    () => {
      function doSomething() {
        console.log(someProp)
      }
      doSomething()
    },
    [someProps] // 相关属性改变一目了然
  )

  return <div>example</div>
}
```

方法二: 把函数加入依赖列表中

```js
function Example({ someProp }) {
  function doSomething() {
    console.log(someProp)
  }

  useEffect(
    () => {
      doSomething()
    },
    [doSomething]
  )

  return <div>example</div>
}
```

方案二基本上不会单独使用, 它一般结合 `useCallback` 一起使用来处理某些函数计算量较大的函数。

```js
function Example({ someProp }) {
  const doSomething = useCallback(() => {
    console.log(someProp)
  }, [someProp])

  useEffect(
    doSomething(),
    [doSomething]
  )

  return <div>example</div>
}
```

#### 如何避免重复创建昂贵的对象

* 使用 `useState` 的 `lazy-initial`

使用 `const [value, setValue] = useState(() => createExpensiveObj)`, 见 [lazy-initial-state](https://reactjs.org/docs/hooks-reference.html#lazy-initial-state);

* 使用自定义 useRef 函数

```js
function Image(props) {
  const ref = useRef(null)

  function getExpensiveObj() {
    if (ref.current === null) {
      ref.current = ExpensiveObj
    }

    return ref.current
  }

  // if need ExpensiveObj, call getExpensiveObj()
}
```

### 相关资料

* [Hooks RFCS](https://github.com/reactjs/rfcs/pull/68#issuecomment-439314884)
* [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html)
* [Making Sense of React Hooks](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)
* [Vue Function-based API RFC](https://zhuanlan.zhihu.com/p/68477600)
