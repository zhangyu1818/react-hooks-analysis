---
class: text-center
highlighter: shiki
background: https://source.unsplash.com/collection/94734566/1920x1080
---

# React Hooks 浅析

ZHANG YU

<a href="https://github.com/zhangyu1818" target="_blank" alt="GitHub"
  class="abs-tr m-6 text-xl icon-btn opacity-50 !border-none !hover:text-white">
<carbon-logo-github />
</a>

<span class="abs-br m-2 text-sm opacity-50">
  powered by 
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" class="hover:text-blue-600">
     slidev
  </a>
</span>

---

# 📝 误解的知识点

网上一些文章可能也是道听途说，所以有可能会产生一些错误的观点。

- **React** 从 17 年底发布 v16，迄今已有近 4 年，为了实现可中断渲染技术，同时出现了**Fiber**这样的技术名词，实际到目前 v17.0.2 稳定版本依然没有开启**Concurrent Mode**。

- **React** 中的 diff 算法，并不是新旧 Fiber 节点互相做 diff，而是旧的 Fiber 节点和新的 **_JSX 对象_** 做 diff。

- **React** 中的 diff 并不会决定组件是否会更新，只会决定是否要复用 Fiber 节点。

- **React** 中的 diff 并不是作用于当前的 Fiber 节点，而是作用于子节点。

---

# 冷知识

<v-click>

1. **Hooks**方法在函数组件初次挂载时和后续更新调用的实际不是同一个方法。

```ts {all|1,6|all}
const HooksDispatcherOnMount: Dispatcher = {
  // ...
  useReducer: mountReducer,
  useState: mountState,
};
const HooksDispatcherOnUpdate: Dispatcher = {
  // ...
  useReducer: updateReducer,
  useState: updateState,
};
```

</v-click>

<v-click>

2. **React**不是 **TypeScript**写的，**Vue 2.x**也不是，它们都是 **flow** 写的。

</v-click>

---

# 一个函数组件是如何 Render 的

<v-clicks>

1. **React.createElement**或者**react/jsx-runtime**生成 JSX 对象。

2. 进入**render**流程中的**beginWork**阶段，此阶段为新对象创建 Fiber 节点，旧节点**diff**。

3. **beginWork**阶段进入**renderWithHooks**方法，此方法中会为全局的**Hooks**对象绑定方法，同时调用函数组件，产生新的**children**。

4. **completeWork**阶段啥事也不干。

5. **commitRoot...** 最终呈现在页面上。

</v-clicks>

---

# renderWithHooks

该方法执行函数组件，取到函数组件返回的**children**

```ts {all|7-10|13|15|13|all}
export function renderWithHooks<Props, SecondArg>(): any {
  // 省略代码...
  currentlyRenderingFiber = workInProgress;
  // 使用mount还是update的hook
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
      ? HooksDispatcherOnMount
      : HooksDispatcherOnUpdate;
  // 省略代码...
  // 执行函数后返回的children
  let children = Component(props);
  // 省略代码...
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;
  // 省略代码...
  return children;
}
```

<v-clicks>

1. 先判断使用哪一种**HooksDispatcher**
2. 赋值后函数组件执行获取对应的**Hooks**

</v-clicks>

---

# 函数组件的更新原理

React 不像 Vue 可以直接修改值来做到响应式更新，必须显式的调**Hook**更新函数来发起更新调度。

<v-clicks>

而它的更新原理也很简单：

1. 创建更新对象，添加至当前**Fiber 节点**的更新队列。
2. 发起更新调度。
3. 遍历**Fiber 树**，遍历到当前函数组件时，重新调用**renderWithHooks**方法，返回新的**children**。

**Hook**值的更新发生在执行函数组件本身时，执行过程中一定会执行**useXXX**的**Hook**函数，其中会遍历当前**Hook**的更新队列，返回最新的**state**值。

所以在函数组件里，如果发生更新，一定会重新执行函数组件本身。

多次更新函数组件可能并不会多次更新调度，它们会发生在 一次同步执行里。

</v-clicks>

---

# 为什么 Hooks 方法不能写在判断里？

```javascript
if
	useEffect
```

<v-clicks>

- 一个函数组件内可能有多个**Hook 函数**，但是一个函数组件只会对应一个**Fiber 对象**。
- 函数组件执行过程中，会依次执行**Hook 函数**，它们在初次执行时会将自身的数据依次保存在**Fiber 对象**上。
- 如果包含判断，则更新的时候依次来取出**Hook 函数**，就会对应不上了。

</v-clicks>

<br/>
<br/>
<br/>

<v-click>

```javascript {1|2|3,4|5|all}
// hooks数据队列 => [ state0, state1, state2 ], 数据索引 => index = 0
useState0 => index = 0, state = state0, index + 1
if false
  useState1
useState2 => index = 1, state = state1, index + 1 // 我是谁我在哪 ？？？
```

</v-click>

---

# 函数组件更新的先决条件

<v-clicks>

1. 父级经过了**diff**阶段，产生了新的子级**Fiber 节点**（对应该函数组件）。

   ```javascript {all|1,2|4|all}
   jsx => { props: { value: 123 } }
   oldFiber.memoizedProps => { value: 123 }

   ⬇️ diff后,key相同，type不变，复用Fiber节点
   newFiber.pendingProps = jsx.props = { value: 123 }
   ```

2. 该函数组件**Fiber 节点**进入**beginWork**阶段，经判断**props**不相同。

   ```typescript {all|1,2|3|4|all}
   const oldProps = current.memoizedProps; // 当前节点的props
   const newProps = workInProgress.pendingProps; // 本次更新该节点的props
   if (oldProps !== newProps) {
     didReceiveUpdate = true; // 标记有更新
   }
   ```

</v-clicks>

<v-click>

即便**props**的值是相同的，但是由于地址引用不同，所以两者不相等。

在正常情况下，父级改变后，子组件***一定会更新***。

</v-click>

---

# 子组件会更新吗？

```tsx {all|10}
const Child = () => {
  console.log("render");
  return null;
};

const App = () => {
  const [, setState] = useState();
  return (
    <div onClick={() => setState({})}>
      <Child />
    </div>
  );
};
```

<CodeSandBox>
<iframe
  src="https://codesandbox.io/embed/romantic-napier-o2lwd?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
  style="
    width: 100%;
    height: 100%;
    border: 0;
    border-radius: 4px;
    overflow: hidden;
  "
  title="romantic-napier-o2lwd"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
/>
</CodeSandBox>

<br/>

<v-click>

- **props**默认为一个空对象。

- **<Child\/>** 组件即便没有接收任何**props**，但是由于**props**的比较是直接 **==** 比较，所以一定会重新执行。

</v-click>

---

## 避免子组件不必要的更新

<v-clicks>

1. 使用**React.memo**，其内部会有一个默认的浅比较，与**PureComponent**使用的相同比较函数。

2. 使用**useMemo**来包住子组件，让每次更新时子组件都为同一个**JSX 对象**，这样**props**的比较必然相同。

   ```tsx
   const App = () => {
     const [, setState] = useState();
     const child = useMemo(() => <Child />, []);
     return <div onClick={() => setState({})}>{child}</div>;
   };
   ```

</v-clicks>

<v-click>
  
3. 将子组件作为**children**来传递。

<div class="flex ml-5">

<div>

```tsx
const App = ({ children }) => {
    const [, setState] = useState();
    return (
        <div onClick={() => setState({})}>
            {children}
        </div>
    );
};
  <App>   			|
  	<Child />   	|    <App children={<Child />} />
  </App>			|
```

</div>

<div class="w-6 flex-none"/>

- **<Child\/>** 组件通过 **children**的形式传递给 **<App\/>** 组件，即通过**props**的形式传递
- 每当 **<App\/>** 组件更新时，父级传递的**props**并没有发生变化，也就是**children**是相同的，所以子组件不会重复更新。

</div>
  
</v-click>

---

# props 引起的组件更新

众所周知，**props**改变，组件就会更新，在上文中讲到，默认情况下，**props**为 **==** 对比，一定会引起子组件的重新执行，通常情况下使用**React.memo**来让组件对**props**做浅比较。

```json
// 默认情况
oldProps: { value:1 }, newProps: { value: 1}    =>  oldProps === newProps  🙅
// React.memo浅比较
oldProps: { value:1 }, newProps: { value: 1}    =>  value === value        🙆
```

浅比较也就是对比**props**对象的第一层。

---

# 考虑以下情况

**setState**后 经过**React.memo** 的 **<Child\/>** 组件会更新吗？

```tsx {1|5|5,8|all}
const Child = React.memo(() => null);

const App = () => {
  const [, setState] = useState();
  const updater = () => setState({});
  return (
    <div onClick={() => setState({})}>
      <Child updater={updater} />
    </div>
  );
};
```

<v-clicks>

函数组件的更新原理就是重新执行一次函数本身，获取新的**state**值返回新的子节点。

所以**updater**方法在每次更新都会是一个全新的函数，即使 **<Child\/>** 组件使用了**React.memo**，每次传入的**updater**方法不同，所以每次都会更新。

</v-clicks>

---
clicks: 2
---

# 解决方法

```tsx {all|5,8|all}
const Child = React.memo(() => null);

const App = () => {
  const [, setState] = useState();
  const updater = useCallback(() => setState({}), []);
  return (
    <div onClick={() => setState({})}>
      <Child updater={updater} />
    </div>
  );
};
```

<v-click at="2">
  
使用**useCallback**，使每次传递的**updater**都为同一个方法。
  
**React.memo**浅对比相同，**<Child \/>** 组件不更新。
  
</v-click>

---
clicks: 3
---

# 匿名函数是否对性能有影响？

```tsx {all|2,6,7|all}
const App = () => {
  return <div onClick={() => {}} />;
};

const App = () => {
  const onClick = () => {};
  return <div onClick={onClick} />;
};
```

<br/>

<v-click at="2">

函数组件的更新一定会执行函数组件本身，所以匿名函数和命名函数在每次更新都会产生函数声明。

</v-click>

<v-click at="3">
  
<div class="w-1/2">

v8 引擎下分别渲染 1000 个组件的耗时。

![](https://cdn.jsdelivr.net/gh/zhangyu1818/blog@files/files/render-benchmark.png)

</div>
  
</v-click>

---

# **useCallback**和**useMemo**是最优解？

<v-click>

useCallback 和 useMemo 也会有额外的性能消耗。

</v-click>

<v-click>
  
```typescript {all|2,3|5,6|all}
function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  if (areHookInputsEqual(nextDeps, prevDeps)) {
    return prevState[0];
  }
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

</v-click>
  
<br/>

<v-clicks>

- 每次执行带来的额外函数声明和依赖数组。
- 每次执行需要遍历依赖数组来做额外的对比。

</v-clicks>

<v-click>

使用它们不一定会带来性能的提升。

</v-click>

---

# 应用场景

分享下我觉得适用的应用场景。

<v-click>

**useCallback**

- 子组件有**React.memo**时，不希望仅仅传递的无状态函数导致导致子组件函数重新执行。

</v-click>
  
<v-click>
  
如果子组件本来就会接收**state**值来更新，那么**useCallback**无法让子组件不重新渲染，同时还会产生额外的一个函数声明和依赖数组，额外的依赖数组对比。（一次更新内存中同时有2个函数和2个依赖数组）。
  
</v-click>

<v-click>

**useMemo**

- 让一个组件固定为同一个**JSX 对象**，这样在对比时**props**相同。
- 将一个不包含**state**值的对象缓存后传递给子组件，不会让子组件更新。
- 一个组件内有大量数据计算，每次更新都会重新计算，确保性能可以使用**useMemo**缓存计算结果。

</v-click>

---

# useEffect 和 useLayoutEffect

平时可能很少会使用**useLayoutEffect**，它和**useEffect**有什么区别呢？

<v-clicks>
    
其实它们之间最大的区别只是同步和异步的区别。
    
- useEffect 在一次渲染流程结束，页面呈现后的某一时刻异步调用。
    
- useLayoutEffect 在一次渲染流程结束，页面呈现之前同步调用，按照执行时机来讲它等同于**componentDidMount**。
    
</v-clicks>

<br/>

<v-click>

也就是说会影响 DOM 渲染的操作，都可以用**useLayoutEffect**，比如想在组件**Mount**后，拿到实际 DOM 节点做一些操作。

如果使用**useEffect**的话，页面呈现后的某一时刻才调用我们的 DOM 修改，可能会造成屏幕闪烁。

</v-click>

<CodeSandBox>
<iframe src="https://codesandbox.io/embed/nervous-galois-oe4mu?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:100%; border:0; border-radius: 4px; overflow:hidden;"
     title="nervous-galois-oe4mu"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   />
</CodeSandBox>

---

# useRef

**useRef**大多数情况都是用来获取 DOM 节点。

除了普通的用法，我们也可以用来保存函数体内的临时变量。

```javascript
const value = useRef({ value: 1 });
```

修改它也不会使组件发生更新，所以它不应该用来作为**useEffect**的依赖项。

<v-click>

```javascript
useEffect(() => {
  // xxxx
}, [eleRef.current]);
```

如果代码这样才生效，一定是没写对。

</v-click>

---

在开阔点思路，函数也是变量，其实也可以用**useRef**。

```javascript
const callback = useRef(() => null);
```

使用**useRef**来仅仅用来保存函数甚至会优于**useCallback**。

<v-click>

用闭包和**useRef**来实现一个最强的**useCallback**。

```javascript {all|1,2|5,7,8,9,10,11|all}
const useCallback = (fn) => {
  const fnRef = useRef(fn);
  const cbRef = useRef();

  fnRef.current = fn;

  if (!cbRef.current) {
    cbRef.current = (...args) => {
      return fnRef.current(args);
    };
  }

  return cbRef.current;
};
```

</v-click>

---

# 派生 state

派生 state 用通俗的话来讲就是将 props 的值赋值给 state。

这个功能在类组件里实现非常简单，有专门提供的生命周期函数**getDerivedStateFromProps**，但是**Hooks**没有实现这个方法。

```javascript
static getDerivedStateFromProps(props){
    if(props.value){
        return { value: props.value }; // 把props的value赋值给state的value
    }
}
```

为什么会有这样的需求呢？其实在组件开发过程中很常见。

---

对使用者来说是受控组件，必须传入**value**和**onChange**。

```javascript
const Input = ({ value, onChange }) => {
  return <input value={value} onChange={onChange} />;
};
```

一个好的组件，不应该需要使用者提供额外的**value**来控制输入框。

<v-click>

对使用者是非受控组件，只能通过**onChange**获取值，无法传入**value**修改值 。

```javascript
const Input = ({ onChange }) => {
  const [value, setValue] = useState();
  return (
    <input
      value={value}
      onChange={(event) => {
        setValue(event.target.value);
        onChange(event);
      }}
    />
  );
};
```

如果由内部控制，外部就无法修改输入框的值。

</v-click>

---

一个输入框组件，受控和非受控应该是可选的，我们希望没有传入**value**的时候，输入框可以输入值，传入**value**时，输入框的值由外部来控制。

```javascript {all|3,4,5|all}
const Input = ({ value, onChange }) => {
  const [inputValue, setInputValue] = useState(value);
  useEffect(() => {
    setInputValue(value); // 外部value值改变，同步内部value值
  }, [value]);
  const onValueChange = (event) => {
    setInputValue(event.target.value);
    onChange?.(event); // 输入框值改变，同时修改外部值
  };
  return <input value={inputValue} onChange={onValueChange} />;
};
```

<CodeSandBox>
<iframe src="https://codesandbox.io/embed/quirky-dubinsky-iq29f?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:100%; border:0; border-radius: 4px; overflow:hidden;"
     title="quirky-dubinsky-iq29f"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   />
</CodeSandBox>

很多时候组件开发会遇到这样的情况，想把**props**中的**value**赋值给**state**，这种情况就叫派生**state**。

<v-click>

如果外部的**props**改变，我们又需要重新设置**state**，可能使用上述方法，但是这种写法其实是有问题的。

1. 外部**value**改变，触发该 **<Input\>** 组件更新，第一次"render"。

2. 内部**useEffect**检查到依赖项**value**改变，调用**setInputValue**，第二次"render"。

这样的写法是无法避免第二次渲染的问题。

</v-click>

---

# React 官网上的答案

```javascript {all|1,2|4,5,6,78|all}
function Input({ value, onChange }) {
  const [inputValue, setInputValue] = useState(value);
  const prevValue = useRef(null);

  if (value !== prevValue.current) {
    // value 自上次渲染以来发生过改变。更新 inputValue。
    setInputValue(value);
    prevValue.current = value;
  }

  const onInputChange = (event) => {
    setInputValue(event.target.value);
    onChange(event);
  };

  return <input value={inputValue} onChange={onInputChange} />;
}
```

<CodeSandBox>
    <iframe src="https://codesandbox.io/embed/polished-thunder-xfkcb?fontsize=14&hidenavigation=1&theme=light"
         style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
         title="polished-thunder-xfkcb"
         allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
         sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
       />
</CodeSandBox>

> 初看这或许有点奇怪，但渲染期间的一次更新恰恰就是 getDerivedStateFromProps 一直以来的概念。

它依然会执行两次，但是不是两次"render"。

实际官网方案也不是最优解。

---

# 期望的最优解

我们期望的是不要出现第二次函数的执行。

而不通过**setState**就能修改值的方法只有**useRef**了。

<iframe src="https://codesandbox.io/embed/vigilant-ives-1yicg?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:30vh; border:0; border-radius: 4px; overflow:hidden;"
     title="vigilant-ives-1yicg"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

---

# 数据管理库是怎么更新数据的？

在React里，我们的想要触发组件更新，只能用相关的**setState**方法，但是第三方的数据管理库是怎么做到更新的呢？

像Redux，它只是一个与React毫无关系的JS包，想要在Raect里使用还需要安装React-Redux。

<v-click>

它们的数据更新原理都离不开**forceUpdate**。

</v-click>

<v-click>

```javascript
const useForceUpdate = () => {
  return useReducer((x) => x + 1, 0)[1];
};
```
在Hook里的实现无非就是强制的设置一个**state**值，让它更新数据。

</v-click>

---

Redux是一个发布订阅的机制，只需要在**subscribe**里添加上**forceUpdate**，就可以很轻松的实现**React-Redux**。

```javascript {all|2|3|4|5|6,7,8|9,10,11,12,13,14,15|16|all}
const useSelector = (selector) => {
  const [, forceUpdate] = useReducer((s) => s + 1, 0);
  const { store } = useReduxContext();
  const lastState = useRef();
  const selectedState = selector(store.getState());
  useEffect(() => {
    lastState.current = selectedState;
  });
  useEffect(() => {
    return store.subscribe(() => {
      const newState = store.getState();
      if (newState !== lastState.current) {
        forceUpdate();
      }
    });
  }, []);
  return selectedState;
};
```
<v-click>

**React-Redux**的官方**Hook**方案其实就是这样的。现在流行的第三方数据流，基本都是这样订阅更新再**forceUpdate**的方案。

</v-click>

---

# 更好的React的数据流方案

```javascript
const reducer = (value, type) => (type === "increase" ? value + 1 : value - 1);
const { Provider } = React.createContext();
// ...
const [state, dispatch] = useReducer(reducer, 0);
return <Provider value={{ state, dispatch }} >
// ...
```
<v-click>

官方的方案将**Hooks**放进**Context**，这种方案与社区第三方方案的不同点在于它不需要**forceUpdate**，每次更新的同时**Context**也会改变，完完全全的官方**Hooks**。

社区基于订阅机制**forceUpdate**更新如**React-Reudx**，它们的**Context**是不变的，所以才能提供一个稳定可控的**store**。

</v-click>

---

# Context的问题

**Context**解决方案的问题比较严重，主要就是无关的、重复的渲染。

```javascript
    // context => { a:1, b:2 }
    // 组件A
    const { a } = useContext
    // 组件B
    const { b } = useContext
    b += 1
``` 

一旦**Context**更新，其余所有使用了**Context**的组件都会更新，原因就是它是一个整体。

其实这个问题官方也提供了一个不太友好的**feature**。

---

# observedBits

这算是一个隐藏的比较深的东西，我推测从19年开始就有了。

```javascript
const bits = {
  user: 0b01,
  password: 0b10,
}
const context = useContext(Context, bits.user); // 这个context使用了user字段

let result = 0;
// 标识user字段发生变化
if (oldValue.user !== newValue.user){
  result |= bits.user; // 0 -> 0b01
}
return result;
```

通过二进制的判断是否需要更新。

<CodeSandBox>
<iframe src="https://codesandbox.io/embed/zen-ishizaka-q0yzi?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="zen-ishizaka-q0yzi"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
  />
</CodeSandBox>

