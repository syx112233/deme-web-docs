# React Hooks 使用

## <b>前言</b>

以下规范皆为顶层约束，具体使用情况根据场景需要决定，如 useCallBack useMemo 等性能相关，避免滥用造成性能不升反降。

### 1.基本准则

#### <b>只在顶层调用 Hook</b>

<font color="red">不要在循环，条件或嵌套函数中调用 Hook </font>。 相反，总是在 React 函数的顶层使用 Hooks。 通过遵循此规则，您可以确保每次组件渲染时都以相同的顺序调用 Hook 。 这就是允许 React 在多个 useState 和 useEffect 调用之间能正确保留 Hook 状态的原因。

#### <b>只在 React Functions 调用 Hooks</b>

不要在常规 Javascript 函数中调用 Hook。相反，你可以：<br>

- 在 React 函数式组件中调用 Hooks
- 从自定义 Hooks 调用 Hooks

### 2.组件定义

Function Component 采用 const + 箭头函数方式定义：

```js
const App: React.FC<{ title: string }> = ({ title }) => {
  return React.useMemo(() => <div>{title}</div>, [title]);
};

App.defaultProps = {
  title: 'Function Component',
};
```

上面的例子包含了：

1. 用 React.FC 申明 Function Component 组件类型与定义 Props 参数类型。
2. 用 React.useMemo 优化渲染性能。
3. 用 App.defaultProps 定义 Props 的默认值。

### FAQ

> 为什么不用 React.memo?

推荐使用 <code>React.useMemo</code> 而不是 <code>React.memo</code>，因为在组件通信时存在 <code>React.useContext</code> 的用法，这种用法会使所有用到的组件重渲染，只有 <code>React.useMemo</code> 能处理这种场景的按需渲染。

> 没有性能问题的组件也要使用 useMemo 吗？

要，考虑未来维护这个组件的时候，随时可能会通过 `useContext` 等注入一些数据，这时候谁会想起来添加 `useMemo` 呢？

> 为什么不用解构方式代替 defaultProps?

虽然解构方式书写 `defaultProps` 更优雅，但存在一个硬伤：对于对象类型每次 `Rerender` 时引用都会变化，这会带来性能问题，因此不要这么做。

### 3.局部状态

局部状态有三种，根据常用程度依次排列： `useState` `useRef` `useReducer` 。

### <b>useState</b>

```js
const [hide, setHide] = React.useState(false);
const [name, setName] = React.useState('BI');
```

状态函数名要表意，尽量聚集在一起申明，方便查阅。

### <b>useRef</b>

```js
const dom = React.useRef(null);
```

`useRef` 尽量少用，大量 Mutable 的数据会影响代码的可维护性。

但对于不需重复初始化的对象推荐使用 useRef 存储，比如 new G2() 。

### <b>useReducer</b>

局部状态不推荐使用 `useReducer` ，会导致函数内部状态过于复杂，难以阅读。 `useReducer` 建议在多组件间通信时，结合 `useContext` 一起使用。

### FAQ

> 可以在函数内直接申明普通常量或普通函数吗？

不可以，Function Component 每次渲染都会重新执行，常量推荐放到函数外层避免性能问题，函数推荐使用 useCallback 申明。

### 4.函数

所有 Function Component 内函数必须用 React.useCallback 包裹，以保证准确性与性能。

```js
const [hide, setHide] = React.useState(false);

const handleClick = React.useCallback(() => {
  setHide(isHide => !isHide);
}, []);
```

useCallback 第二个参数必须写

### 5.发请求

发请求分为操作型发请求与渲染型发请求。

#### <b>操作型发请求</b>

操作型发请求，作为回调函数：

```js
return React.useMemo(() => {
  return <div onClick={requestService.addList} />;
}, [requestService.addList]);
```

#### <b>渲染型发请求</b>

渲染型发请求在 useAsync 中进行，比如刷新列表页，获取基础信息，或者进行搜索， 都可以抽象为依赖了某些变量，当这些变量变化时要重新取数 ：

```js
const { loading, error, value } = useAsync(async () => {
  return requestService.freshList(id);
}, [requestService.freshList, id]);
```

如果你还不知道`useAsync`,推荐阅读: [你的第一个 useAsync](https://zhuanlan.zhihu.com/p/152495680)

### 6.组件间通信

简单的组件间通信使用透传 Props 变量的方式，而频繁组件间通信使用 `React.useContext` 。<br>

以一个复杂大组件为例，如果组件内部拆分了很多模块， 但需要共享很多内部状态 ，最佳实践如下：

#### <b>定义组件内共享状态 - store.ts</b>

```js
export const StoreContext = React.createContext<{
  state: State;
  dispatch: React.Dispatch<Action>;
}>(null)

export interface State {};

export interface Action { type: 'xxx' } | { type: 'yyy' };

export const initState: State = {};

export const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    default:
      return state;
  }
};
```

#### <b>根组件注入共享状态 - main.ts</b>

```js
import { StoreContext, reducer, initState } from './store';

const AppProvider: React.FC = props => {
  const [state, dispatch] = React.useReducer(reducer, initState);

  return React.useMemo(
    () => (
      <StoreContext.Provider value={{ state, dispatch }}>
        <App />
      </StoreContext.Provider>
    ),
    [state, dispatch],
  );
};
```

#### <b>任意子组件访问/修改共享状态 - child.ts</b>

```js
import { StoreContext } from './store';

const app: React.FC = () => {
  const { state, dispatch } = React.useContext(StoreContext);

  return React.useMemo(() => <div>{state.name}</div>, [state.name]);
};
```

如上解决了 <b>多个联系紧密组件模块间便捷共享状态的问题</b> ，但有时也会遇到需要共享根组件 Props 的问题，<b>这种不可修改的状态不适合一并塞到 `StoreContext` 里</b>，我们新建一个 `PropsContext` 注入根组件的 Props：

```js
const PropsContext = React.createContext < Props > null;

const AppProvider: React.FC<Props> = props => {
  return React.useMemo(
    () => (
      <PropsContext.Provider value={props}>
        <App />
      </PropsContext.Provider>
    ),
    [props],
  );
};
```

结合参考 Redux : [React-redux hooks](https://github.com/reduxjs/react-redux/blob/master/docs/api/hooks.md)

### 7.debounce 优化

比如当输入框频繁输入时，为了保证页面流畅，我们会选择在 `onChange` 时进行 `debounce` 。然而在 Function Component 领域中，我们有更优雅的方式实现。

> 其实在 Input 组件 onChange 使用 debounce 有一个问题，就是当 Input 组件 受控 时， debounce 的值不能及时回填，导致甚至无法输入的问题。

我们站在 Function Component 思维模式下思考这个问题：<br>

1. React scheduling 通过智能调度系统优化渲染优先级，我们其实不用担心频繁变更状态会导致性能问题。
2. 如果联动一个文本还觉得慢吗？ onChange 本不慢，大部分使用值的组件也不慢，没有必要从 onChange 源头开始就 debounce 。
3. 找到渲染性能最慢的组件（比如 iframe 组件），对一些频繁导致其渲染的入参进行 useDebounce 。

下面是一个性能很差的组件，引用了变化频繁的 text （这个 text 可能是 onChange 触发改变的），我们利用 useDebounce 将其变更的频率慢下来即可：

```js
const App: React.FC = ({ text }) => {
  // 无论 text 变化多快，textDebounce 最多 1 秒修改一次
  const textDebounce = useDebounce(text, 1000);

  return useMemo(() => {
    // 使用 textDebounce，但渲染速度很慢的一堆代码
  }, [textDebounce]);
};
```

使用 textDebounce 替代 text 可以将渲染频率控制在我们指定的范围内。

> 防抖：触发高频事件后 n 秒内函数只会执行一次，如果 n 秒内高频事件再次被触发，则重新计算时间。

```js
function useDebounce(fn, delay, dep = []) {
  const { current } = useRef({ fn, timer: null });
  useEffect(
    function() {
      current.fn = fn;
    },
    [fn],
  );

  return useCallback(function f(...args) {
    if (current.timer) {
      clearTimeout(current.timer);
    }
    current.timer = setTimeout(() => {
      current.fn.call(this, ...args);
    }, delay);
  }, dep);
}
```

> 节流：高频事件触发，但在 n 秒内只会执行一次，所以节流会稀释函数的执行频率。

```js
function useThrottle(fn, delay, dep = []) {
  const { current } = useRef({ fn, timer: null });
  useEffect(
    function() {
      current.fn = fn;
    },
    [fn],
  );

  return useCallback(function f(...args) {
    if (!current.timer) {
      current.timer = setTimeout(() => {
        delete current.timer;
      }, delay);
      current.fn.call(this, ...args);
    }
  }, dep);
}
```

### <b>8.真正理解 useEffect,非常重要！！！</b>

要说清除 useEffect, 最好先从 Render 概念着手。

> 1.状态值为什么每次都不是最新的？

```js
const App = () => {
  const [temp, setTemp] = React.useState(5);

  const log = () => {
    setTimeout(() => {
      console.log('3 秒前 temp = 5，现在 temp =', temp);
    }, 3000);
  };

  return (
    <div
      onClick={() => {
        log();
        setTemp(3);
        // 3 秒前 temp = 5，现在 temp = 5
      }}
    >
      xyz
    </div>
  );
};
```

因为 React Hooks 具有 Capture Value 特性，每次 Render 都有自己的 Props 与 State, 每次 Render 都有自己的事件处理。
在 log 函数执行的那个 Render 过程里，temp 的值可以看作常量 5，执行 setTemp(3) 时会交由一个全新的 Render 渲染，所以不会执行 log 函数。而 3 秒后执行的内容是由 temp 为 5 的那个 Render 发出的，所以结果自然为 5。

原因就是 temp、log 都拥有 Capture Value 特性。

<font color="red">useEffect 在实际 DOM 渲染完毕后执行，那 useEffect 拿到的值也遵循 Capture Value 的特性</font>

> 2.如何绕过 Capture Value?

利用 useRef 就可以绕过 Capture Value 的特性。可以认为 ref 在所有 Render 过程中保持着唯一引用，因此所有对 ref 的赋值或取值，拿到的都只有一个最终状态，而不会在每个 Render 间存在隔离。

```js
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
}
```

也可以简洁的认为，ref 是 Mutable 的，而 state 是 Immutable 的。

#### <b>回收机制</b>

在组件被销毁时，通过 useEffect 注册的监听需要被销毁，这一点可以通过 useEffect 的返回值做到：

```js
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
  };
});
```

在组件被销毁时，会执行返回值函数内回调函数。同样，由于 Capture Value 特性，每次 “注册” “回收” 拿到的都是成对的固定值。

#### <b>用同步取代 “生命周期”</b>

Function Component 不存在生命周期，所以不要把 Class Component 的生命周期概念搬过来试图对号入座。Function Component 仅描述 UI 状态，React 会将其同步到 DOM，仅此而已。

既然是状态同步，那么每次渲染的状态都会固化下来，这包括 state props useEffect 以及写在 Function Component 中的所有函数。

然而舍弃了生命周期的同步会带来一些性能问题，所以我们需要告诉 React 如何比对 Effect。

#### <b>告诉 React 如何对比 Effects</b>

虽然 React 在 DOM 渲染时会 diff 内容，只对改变部分进行修改，而不是整体替换，但却做不到对 Effect 的增量修改识别。因此需要开发者通过 useEffect 的第二个参数告诉 React 用到了哪些外部变量：

```js
useEffect(() => {
  document.title = 'Hello, ' + name;
}, [name]); // Our deps
```

直到 name 改变时的 Rerender，useEffect 才会再次执行。

#### <b>细说 useEffect 依赖</b>

如果你明明使用了某个变量，却没有申明在依赖中，你等于向 React 撒了谎，后果就是，当依赖的变量改变时，useEffect 也不会再次执行。<br>

举个例子：

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

setInterval 我们只想执行一次，所以我们自以为聪明的向 React 撒了谎，将依赖写成 []。

“组件初始化执行一次 setInterval，销毁时执行一次 clearInterval，这样的代码符合预期。” 你心里可能这么想。

但是你错了，由于 useEffect 符合 Capture Value 的特性，拿到的 count 值永远是初始化的 0。相当于 setInterval 永远在 count 为 0 的 Scope 中执行，你后续的 setCount 操作并不会产生任何作用。

> 但诚实也是要付出代价，比如下面这个例子：

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

你老实告诉 React “等 count 变化后再执行吧”，那么你会得到一个好消息和两个坏消息。

好消息是，代码可以正常运行了，拿到了最新的 count。

坏消息有：

1. 计时器不准了，因为每次 count 变化时都会销毁并重新计时。

2. 频繁 生成/销毁 定时器带来了一定性能负担。

> 怎么既诚实又高效呢？

上述例子使用了 count，然而这样的代码很别扭，因为你在一个只想执行一次的 Effect 里依赖了外部变量。

既然要诚实，那只好 想办法不依赖外部变量：

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  return () => clearInterval(id);
}, []);
```

setCount 还有一种函数回调模式，你不需要关心当前值是什么，只要对 “旧的值” 进行修改即可。这样虽然代码永远运行在第一次 Render 中，但总是可以访问到最新的 state。

#### <b>将更新与动作解耦</b>

你可能发现了，上面投机取巧的方式并没有彻底解决所有场景的问题，比如同时依赖了两个 state 的情况：

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [step]);
```

你会发现不得不依赖 step 这个变量，我们又回到了 “诚实的代价” 。

利用 useEffect 的兄弟 useReducer 函数，将更新与动作解耦就可以了：

```js
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

这就是一个局部 “Redux”，由于更新变成了 dispatch({ type: "tick" }) 所以不管更新时需要依赖多少变量，在调用更新的动作里都不需要依赖任何变量。 具体更新操作在 reducer 函数里写就可以了.

#### <b>useEffect 的优势</b>

- useEffect 在渲染结束时执行，所以不会阻塞浏览器渲染进程，所以使用 Function Component 写的项目一般都有用更好的性能。

- 自然符合 React Fiber 的理念，因为 Fiber 会根据情况暂停或插队执行不同组件的 Render，如果代码遵循了 Capture Value 的特性，在 Fiber 环境下会保证值的安全访问，同时弱化生命周期也能解决中断执行时带来的问题。

- useEffect 不会在服务端渲染时执行。

- 由于在 DOM 执行完毕后才执行，所以能保证拿到状态生效后的 DOM 属性。

如需了解 Function Component 或 Hooks 基础用法，可参考：

[精读 React Hooks](https://github.com/dt-fe/weekly/blob/v2/079.%E7%B2%BE%E8%AF%BB%E3%80%8AReact%20Hooks%E3%80%8B.md)

[精读 Function Component 入门](https://github.com/dt-fe/weekly/blob/v2/104.%E7%B2%BE%E8%AF%BB%E3%80%8AFunction%20Component%20%E5%85%A5%E9%97%A8%E3%80%8B.md)

[精读《怎么用 React Hooks 造轮子》](https://github.com/dt-fe/weekly/blob/v2/080.%E7%B2%BE%E8%AF%BB%E3%80%8A%E6%80%8E%E4%B9%88%E7%94%A8%20React%20Hooks%20%E9%80%A0%E8%BD%AE%E5%AD%90%E3%80%8B.md)
