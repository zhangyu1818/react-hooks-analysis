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

- **React** 从 17 年底发布 v16，迄今已有近 4 年，为了实现可中断渲染技术，同时出现了**Fiber**这样的技术名词，实际到目前v17.0.2稳定版本依然没有开启**Concurrent Mode**。

- **React** 中的diff算法，并不是新旧Fiber节点互相做diff，而是旧的Fiber节点和新的 ***JSX对象*** 做diff。

- **React** 中的diff并不会决定组件是否会更新，只会决定是否要复用Fiber节点。

- **React** 中的diff并不是作用于当前的Fiber节点，而是作用于子节点。


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

# 一个JSX对象

```json
{
  $$typeof: REACT_ELEMENT_TYPE,
  type,
  props
}
```

- 不同的**JSX对象**拥有不同的 **$$typeof** 字段，它的值默认为数字，会升级为**symbol**。

- **type**字段则是该**JSX对象**的表现形式，通常为字符串（原生组件）、函数、类。

- **props**字段则是过滤了**key**和**ref**的对象，一个父级函数可以获取任意**children**对象的**props**。


---

# 一次渲染的流程

<v-clicks>

1. 发起更新调度，确定**WorkInProgress**节点。

2. 节点进入**beginWork**阶段，深度遍历，将WorkInProgress执行子级节点，此阶段就是**diff**发生的阶段。

3. 节点进入**completeWork**阶段，将WorkInProgress指向同级或父级节点，同时组装**EffectList**，标记那些节点包含副作用。

4. 节点树遍历完成后进入**commitRoot**阶段。

5. **EffectList**中的节点经过**BeforeMutation**、**Mutation**、**Layout**阶段处理。

6. 页面呈现。
  
</v-clicks>

<v-click>

2 - 3 阶段称为**render**阶段，这个阶段在未来的**Concurrent Mode**中是可中断的。

4 - 5 阶段称为**commit**阶段，这个阶段包含生命周期和**Effect Hooks**的处理，不可中断。
  
</v-click>


---

# 一个函数组件是如何Render的

<v-clicks>

1. **React.createElement**或者**react/jsx-runtime**生成JSX对象。

2. 进入**render**流程中的**beginWork**阶段，此阶段为新对象创建Fiber节点，旧节点**diff**。

3. **beginWork**阶段进入**renderWithHooks**方法，此方法中会为全局的**Hooks**对象绑定方法，同时调用函数组件，产生新的**children**。

4. **completeWork**阶段啥事也不干。

5. **commitRoot...** 最终呈现在页面上。
  
</v-clicks>


---

# renderWithHooks

该方法执行函数组件，取到函数组件返回的**children**

```ts {all|7-10|13|15|13|all}
export function renderWithHooks<Props, SecondArg>(
// ...
): any {
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

React不像Vue可以直接修改值来做到响应式更新，必须显式的调**Hook**更新函数来发起更新调度。

<v-clicks>

而它的更新原理也很简单：

1. 创建更新对象，添加至当前**Fiber节点**的更新队列。
2. 发起更新调度。
3. 遍历**Fiber树**，遍历到当前函数组件时，重新调用**renderWithHooks**方法，返回新的**children**。

**Hook**值的更新发生在执行函数组件本身时，执行过程中一定会执行**useXXX**的**Hook**函数，其中会遍历当前**Hook**的更新队列，返回最新的**state**值。

所以在函数组件里，如果发生更新，一定会重新执行函数组件本身。

多次更新函数组件可能并不会多次更新调度，它们会发生在 一次同步执行里。
  
</v-clicks>


---

# 为什么Hooks方法不能写在判断里？

```javascript
if
	useEffect
```

<v-clicks>

- 一个函数组件内可能有多个**Hook函数**，但是一个函数组件只会对应一个**Fiber对象**。
- 函数组件执行过程中，会依次执行**Hook函数**，它们在初次执行时会将自身的数据依次保存在**Fiber对象**上。
- 如果包含判断，则更新的时候依次来取出**Hook函数**，就会对应不上了。

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

1. 父级经过了**diff**阶段，产生了新的子级**Fiber节点**（对应该函数组件）。

   ```javascript {all|1,2|4|all}
   jsx => { props: { value: 123 } }
   oldFiber.memoizedProps => { value: 123 }
  
   ⬇️ diff后,key相同，type不变，复用Fiber节点
   newFiber.pendingProps = jsx.props = { value: 123 }
   ```

2. 该函数组件**Fiber节点**进入**beginWork**阶段，经判断**props**不相同。

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

<br/>

<v-click>

- **props**默认为一个空对象。

- **<Child\/>** 组件即便没有接收任何**props**，但是由于**props**的比较是直接 **==** 比较，所以一定会重新执行。

</v-click>


---

## 避免子组件不必要的更新

<v-clicks>

1. 使用**React.memo**，其内部会有一个默认的浅比较，与**PureComponent**使用的相同比较函数。
  
2. 使用**useMemo**来包住子组件，让每次更新时子组件都为同一个**JSX对象**，这样**props**的比较必然相同。

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

# props引起的组件更新

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

v8引擎下分别渲染1000个组件的耗时。

![](https://cdn.jsdelivr.net/gh/zhangyu1818/blog@files/files/render-benchmark.png)

</div>
  
</v-click>


---

# **useCallback**和**useMemo**是最优解？

<v-click>

useCallback和useMemo也会有额外的性能消耗。
  
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

基于我对源码的理解，分享下我觉得适用的应用场景。

<v-click>

**useCallback**

- 子组件有**React.memo**时，不希望仅仅传递的无状态函数导致导致子组件函数重新执行。

</v-click>
  
<v-click>
  
如果子组件本来就会接收**state**值来更新，那么**useCallback**无法让子组件不重新渲染，同时还会产生额外的一个函数声明和依赖数组，额外的依赖数组对比。（一次更新内存中同时有2个函数和2个依赖数组）。
  
</v-click>

<v-click>

**useMemo**

- 让一个组件固定为同一个**JSX对象**，这样在对比时**props**相同。
- 将一个不包含**state**值的对象缓存后传递给子组件，不会让子组件更新。
- 一个组件内有大量数据计算，每次更新都会重新计算，确保性能可以使用**useMemo**缓存计算结果。
  
</v-click>

---
