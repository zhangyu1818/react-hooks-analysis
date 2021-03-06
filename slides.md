---
class: text-center
highlighter: shiki
background: https://source.unsplash.com/collection/94734566/1920x1080
---

# React Hooks æµæ

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

# ð è¯¯è§£çç¥è¯ç¹

ç½ä¸ä¸äºæç« å¯è½ä¹æ¯éå¬éè¯´ï¼æä»¥æå¯è½ä¼äº§çä¸äºéè¯¯çè§ç¹ã

- **React** ä» 17 å¹´åºåå¸ v16ï¼è¿ä»å·²æè¿ 4 å¹´ï¼ä¸ºäºå®ç°å¯ä¸­æ­æ¸²æææ¯ï¼åæ¶åºç°äº**Fiber**è¿æ ·çææ¯åè¯ï¼å®éå°ç®å v17.0.2 ç¨³å®çæ¬ä¾ç¶æ²¡æå¼å¯**Concurrent Mode**ã

- **React** ä¸­ç diff ç®æ³ï¼å¹¶ä¸æ¯æ°æ§ Fiber èç¹äºç¸å diffï¼èæ¯æ§ç Fiber èç¹åæ°ç **_JSX å¯¹è±¡_** å diffã

- **React** ä¸­ç diff å¹¶ä¸ä¼å³å®ç»ä»¶æ¯å¦ä¼æ´æ°ï¼åªä¼å³å®æ¯å¦è¦å¤ç¨ Fiber èç¹ã

- **React** ä¸­ç diff å¹¶ä¸æ¯ä½ç¨äºå½åç Fiber èç¹ï¼èæ¯ä½ç¨äºå­èç¹ã

---

# å·ç¥è¯

<v-click>

1. **Hooks**æ¹æ³å¨å½æ°ç»ä»¶åæ¬¡æè½½æ¶ååç»­æ´æ°è°ç¨çå®éä¸æ¯åä¸ä¸ªæ¹æ³ã

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

2. **React**ä¸æ¯ **TypeScript**åçï¼**Vue 2.x**ä¹ä¸æ¯ï¼å®ä»¬é½æ¯ **flow** åçã

</v-click>

---

# ä¸ä¸ªå½æ°ç»ä»¶æ¯å¦ä½ Render ç

<v-clicks>

1. **React.createElement**æè**react/jsx-runtime**çæ JSX å¯¹è±¡ã

2. è¿å¥**render**æµç¨ä¸­ç**beginWork**é¶æ®µï¼æ­¤é¶æ®µä¸ºæ°å¯¹è±¡åå»º Fiber èç¹ï¼æ§èç¹**diff**ã

3. **beginWork**é¶æ®µè¿å¥**renderWithHooks**æ¹æ³ï¼æ­¤æ¹æ³ä¸­ä¼ä¸ºå¨å±ç**Hooks**å¯¹è±¡ç»å®æ¹æ³ï¼åæ¶è°ç¨å½æ°ç»ä»¶ï¼äº§çæ°ç**children**ã

4. **completeWork**é¶æ®µå¥äºä¹ä¸å¹²ã

5. **commitRoot...** æç»åç°å¨é¡µé¢ä¸ã

</v-clicks>

---

# renderWithHooks

è¯¥æ¹æ³æ§è¡å½æ°ç»ä»¶ï¼åå°å½æ°ç»ä»¶è¿åç**children**

```ts {all|5-8|11|13|11|all}
export function renderWithHooks<Props, SecondArg>(): any {
  // çç¥ä»£ç ...
  currentlyRenderingFiber = workInProgress;
  // ä½¿ç¨mountè¿æ¯updateçhook
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
      ? HooksDispatcherOnMount
      : HooksDispatcherOnUpdate;
  // çç¥ä»£ç ...
  // æ§è¡å½æ°åè¿åçchildren
  let children = Component(props);
  // çç¥ä»£ç ...
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;
  // çç¥ä»£ç ...
  return children;
}
```

<v-clicks>

1. åå¤æ­ä½¿ç¨åªä¸ç§**HooksDispatcher**
2. èµå¼åå½æ°ç»ä»¶æ§è¡è·åå¯¹åºç**Hooks**

</v-clicks>

---

# å½æ°ç»ä»¶çæ´æ°åç

React ä¸å Vue å¯ä»¥ç´æ¥ä¿®æ¹å¼æ¥åå°ååºå¼æ´æ°ï¼å¿é¡»æ¾å¼çè°**Hook**æ´æ°å½æ°æ¥åèµ·æ´æ°è°åº¦ã

<v-clicks>

èå®çæ´æ°åçä¹å¾ç®åï¼

1. åå»ºæ´æ°å¯¹è±¡ï¼æ·»å è³å½å**Fiber èç¹**çæ´æ°éåã
2. åèµ·æ´æ°è°åº¦ã
3. éå**Fiber æ **ï¼éåå°å½åå½æ°ç»ä»¶æ¶ï¼éæ°è°ç¨**renderWithHooks**æ¹æ³ï¼è¿åæ°ç**children**ã

**Hook**å¼çæ´æ°åçå¨æ§è¡å½æ°ç»ä»¶æ¬èº«æ¶ï¼æ§è¡è¿ç¨ä¸­ä¸å®ä¼æ§è¡**useXXX**ç**Hook**å½æ°ï¼å¶ä¸­ä¼éåå½å**Hook**çæ´æ°éåï¼è¿åææ°ç**state**å¼ã

æä»¥å¨å½æ°ç»ä»¶éï¼å¦æåçæ´æ°ï¼ä¸å®ä¼éæ°æ§è¡å½æ°ç»ä»¶æ¬èº«ã

å¤æ¬¡æ´æ°å½æ°ç»ä»¶å¯è½å¹¶ä¸ä¼å¤æ¬¡æ´æ°è°åº¦ï¼å®ä»¬ä¼åçå¨ ä¸æ¬¡åæ­¥æ§è¡éã

</v-clicks>

---

# ä¸ºä»ä¹ Hooks æ¹æ³ä¸è½åå¨å¤æ­éï¼

```javascript
if
	useEffect
```

<v-clicks>

- ä¸ä¸ªå½æ°ç»ä»¶åå¯è½æå¤ä¸ª**Hook å½æ°**ï¼ä½æ¯ä¸ä¸ªå½æ°ç»ä»¶åªä¼å¯¹åºä¸ä¸ª**Fiber å¯¹è±¡**ã
- å½æ°ç»ä»¶æ§è¡è¿ç¨ä¸­ï¼ä¼ä¾æ¬¡æ§è¡**Hook å½æ°**ï¼å®ä»¬å¨åæ¬¡æ§è¡æ¶ä¼å°èªèº«çæ°æ®ä¾æ¬¡ä¿å­å¨**Fiber å¯¹è±¡**ä¸ã
- å¦æåå«å¤æ­ï¼åæ´æ°çæ¶åä¾æ¬¡æ¥ååº**Hook å½æ°**ï¼å°±ä¼å¯¹åºä¸ä¸äºã

</v-clicks>

<br/>
<br/>
<br/>

<v-click>

```javascript {1|2|3,4|5|all}
// hooksæ°æ®éå => [ state0, state1, state2 ], æ°æ®ç´¢å¼ => index = 0
useState0 => index = 0, state = state0, index + 1
if false
  useState1
useState2 => index = 1, state = state1, index + 1 // ææ¯è°æå¨åª ï¼ï¼ï¼
```

</v-click>

---

# å½æ°ç»ä»¶æ´æ°çåå³æ¡ä»¶

<v-clicks>

1. ç¶çº§ç»è¿äº**diff**é¶æ®µï¼äº§çäºæ°çå­çº§**Fiber èç¹**ï¼å¯¹åºè¯¥å½æ°ç»ä»¶ï¼ã

   ```javascript {all|1,2|4|all}
   jsx => { props: { value: 123 } }
   oldFiber.memoizedProps => { value: 123 }

   â¬ï¸ diffå,keyç¸åï¼typeä¸åï¼å¤ç¨Fiberèç¹
   newFiber.pendingProps = jsx.props = { value: 123 }
   ```

2. è¯¥å½æ°ç»ä»¶**Fiber èç¹**è¿å¥**beginWork**é¶æ®µï¼ç»å¤æ­**props**ä¸ç¸åã

   ```typescript {all|1,2|3|4|all}
   const oldProps = current.memoizedProps; // å½åèç¹çprops
   const newProps = workInProgress.pendingProps; // æ¬æ¬¡æ´æ°è¯¥èç¹çprops
   if (oldProps !== newProps) {
     didReceiveUpdate = true; // æ è®°ææ´æ°
   }
   ```

</v-clicks>

<v-click>

å³ä¾¿**props**çå¼æ¯ç¸åçï¼ä½æ¯ç±äºå°åå¼ç¨ä¸åï¼æä»¥ä¸¤èä¸ç¸ç­ã

å¨æ­£å¸¸æåµä¸ï¼ç¶çº§æ¹ååï¼å­ç»ä»¶***ä¸å®ä¼æ´æ°***ã

</v-click>

---

# å­ç»ä»¶ä¼æ´æ°åï¼

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
  src="https://codesandbox.io/embed/romantic-napier-o2lwd?expanddevtools=1&fontsize=14&hidenavigation=1"
  style="
    width: 100%;
    height: 99%;
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

- **props**é»è®¤ä¸ºä¸ä¸ªç©ºå¯¹è±¡ã

- **<Child\/>** ç»ä»¶å³ä¾¿æ²¡ææ¥æ¶ä»»ä½**props**ï¼ä½æ¯ç±äº**props**çæ¯è¾æ¯ç´æ¥ **==** æ¯è¾ï¼æä»¥ä¸å®ä¼éæ°æ§è¡ã

</v-click>

---

## é¿åå­ç»ä»¶ä¸å¿è¦çæ´æ°

<v-clicks>

1. ä½¿ç¨**React.memo**ï¼å¶åé¨ä¼æä¸ä¸ªé»è®¤çæµæ¯è¾ï¼ä¸**PureComponent**ä½¿ç¨çç¸åæ¯è¾å½æ°ã

2. ä½¿ç¨**useMemo**æ¥åä½å­ç»ä»¶ï¼è®©æ¯æ¬¡æ´æ°æ¶å­ç»ä»¶é½ä¸ºåä¸ä¸ª**JSX å¯¹è±¡**ï¼è¿æ ·**props**çæ¯è¾å¿ç¶ç¸åã

   ```tsx
   const App = () => {
     const [, setState] = useState();
     const child = useMemo(() => <Child />, []);
     return <div onClick={() => setState({})}>{child}</div>;
   };
   ```

</v-clicks>

<v-click>
  
3. å°å­ç»ä»¶ä½ä¸º**children**æ¥ä¼ éã

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

- **<Child\/>** ç»ä»¶éè¿ **children**çå½¢å¼ä¼ éç» **<App\/>** ç»ä»¶ï¼å³éè¿**props**çå½¢å¼ä¼ é
- æ¯å½ **<App\/>** ç»ä»¶æ´æ°æ¶ï¼ç¶çº§ä¼ éç**props**å¹¶æ²¡æåçååï¼ä¹å°±æ¯**children**æ¯ç¸åçï¼æä»¥å­ç»ä»¶ä¸ä¼éå¤æ´æ°ã

</div>
  
</v-click>

---

# props å¼èµ·çç»ä»¶æ´æ°

ä¼æå¨ç¥ï¼**props**æ¹åï¼ç»ä»¶å°±ä¼æ´æ°ï¼å¨ä¸æä¸­è®²å°ï¼é»è®¤æåµä¸ï¼**props**ä¸º **==** å¯¹æ¯ï¼ä¸å®ä¼å¼èµ·å­ç»ä»¶çéæ°æ§è¡ï¼éå¸¸æåµä¸ä½¿ç¨**React.memo**æ¥è®©ç»ä»¶å¯¹**props**åæµæ¯è¾ã

```json
// é»è®¤æåµ
oldProps: { value:1 }, newProps: { value: 1}    =>  oldProps === newProps  ð
// React.memoæµæ¯è¾
oldProps: { value:1 }, newProps: { value: 1}    =>  value === value        ð
```

æµæ¯è¾ä¹å°±æ¯å¯¹æ¯**props**å¯¹è±¡çç¬¬ä¸å±ã

---

# èèä»¥ä¸æåµ

**setState**å ç»è¿**React.memo** ç **<Child\/>** ç»ä»¶ä¼æ´æ°åï¼

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

å½æ°ç»ä»¶çæ´æ°åçå°±æ¯éæ°æ§è¡ä¸æ¬¡å½æ°æ¬èº«ï¼è·åæ°ç**state**å¼è¿åæ°çå­èç¹ã

æä»¥**updater**æ¹æ³å¨æ¯æ¬¡æ´æ°é½ä¼æ¯ä¸ä¸ªå¨æ°çå½æ°ï¼å³ä½¿ **<Child\/>** ç»ä»¶ä½¿ç¨äº**React.memo**ï¼æ¯æ¬¡ä¼ å¥ç**updater**æ¹æ³ä¸åï¼æä»¥æ¯æ¬¡é½ä¼æ´æ°ã

</v-clicks>

---
clicks: 2
---

# è§£å³æ¹æ³

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
  
ä½¿ç¨**useCallback**ï¼ä½¿æ¯æ¬¡ä¼ éç**updater**é½ä¸ºåä¸ä¸ªæ¹æ³ã
  
**React.memo**æµå¯¹æ¯ç¸åï¼**<Child \/>** ç»ä»¶ä¸æ´æ°ã
  
</v-click>

---
clicks: 3
---

# å¿åå½æ°æ¯å¦å¯¹æ§è½æå½±åï¼

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

å½æ°ç»ä»¶çæ´æ°ä¸å®ä¼æ§è¡å½æ°ç»ä»¶æ¬èº«ï¼æä»¥å¿åå½æ°åå½åå½æ°å¨æ¯æ¬¡æ´æ°é½ä¼äº§çå½æ°å£°æã

</v-click>

<v-click at="3">
  
<div class="w-1/2">

v8 å¼æä¸åå«æ¸²æ 1000 ä¸ªç»ä»¶çèæ¶ã

![](https://cdn.jsdelivr.net/gh/zhangyu1818/blog@files/files/render-benchmark.png)

</div>
  
</v-click>

---

# **useCallback**å**useMemo**æ¯æä¼è§£ï¼

<v-click>

useCallback å useMemo ä¹ä¼æé¢å¤çæ§è½æ¶èã

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

- æ¯æ¬¡æ§è¡å¸¦æ¥çé¢å¤å½æ°å£°æåä¾èµæ°ç»ã
- æ¯æ¬¡æ§è¡éè¦éåä¾èµæ°ç»æ¥åé¢å¤çå¯¹æ¯ã

</v-clicks>

<v-click>

ä½¿ç¨å®ä»¬ä¸ä¸å®ä¼å¸¦æ¥æ§è½çæåã

</v-click>

---

# åºç¨åºæ¯

åäº«ä¸æè§å¾éç¨çåºç¨åºæ¯ã

<v-click>

**useCallback**

- å­ç»ä»¶æ**React.memo**æ¶ï¼ä¸å¸æä»ä»ä¼ éçæ ç¶æå½æ°å¯¼è´å¯¼è´å­ç»ä»¶å½æ°éæ°æ§è¡ã

</v-click>
  
<v-click>
  
å¦æå­ç»ä»¶æ¬æ¥å°±ä¼æ¥æ¶**state**å¼æ¥æ´æ°ï¼é£ä¹**useCallback**æ æ³è®©å­ç»ä»¶ä¸éæ°æ¸²æï¼åæ¶è¿ä¼äº§çé¢å¤çä¸ä¸ªå½æ°å£°æåä¾èµæ°ç»ï¼é¢å¤çä¾èµæ°ç»å¯¹æ¯ãï¼ä¸æ¬¡æ´æ°åå­ä¸­åæ¶æ2ä¸ªå½æ°å2ä¸ªä¾èµæ°ç»ï¼ã
  
</v-click>

<v-click>

**useMemo**

- è®©ä¸ä¸ªç»ä»¶åºå®ä¸ºåä¸ä¸ª**JSX å¯¹è±¡**ï¼è¿æ ·å¨å¯¹æ¯æ¶**props**ç¸åã
- å°ä¸ä¸ªä¸åå«**state**å¼çå¯¹è±¡ç¼å­åä¼ éç»å­ç»ä»¶ï¼ä¸ä¼è®©å­ç»ä»¶æ´æ°ã
- ä¸ä¸ªç»ä»¶åæå¤§éæ°æ®è®¡ç®ï¼æ¯æ¬¡æ´æ°é½ä¼éæ°è®¡ç®ï¼ç¡®ä¿æ§è½å¯ä»¥ä½¿ç¨**useMemo**ç¼å­è®¡ç®ç»æã

</v-click>

---

# useEffect å useLayoutEffect

å¹³æ¶å¯è½å¾å°ä¼ä½¿ç¨**useLayoutEffect**ï¼å®å**useEffect**æä»ä¹åºå«å¢ï¼

<v-clicks>
    
å¶å®å®ä»¬ä¹é´æå¤§çåºå«åªæ¯åæ­¥åå¼æ­¥çåºå«ã
    
- useEffect å¨ä¸æ¬¡æ¸²ææµç¨ç»æï¼é¡µé¢åç°åçæä¸æ¶å»å¼æ­¥è°ç¨ã
    
- useLayoutEffect å¨ä¸æ¬¡æ¸²ææµç¨ç»æï¼é¡µé¢åç°ä¹ååæ­¥è°ç¨ï¼æç§æ§è¡æ¶æºæ¥è®²å®ç­åäº**componentDidMount**ã
    
</v-clicks>

<br/>

<v-click>

ä¹å°±æ¯è¯´ä¼å½±å DOM æ¸²æçæä½ï¼é½å¯ä»¥ç¨**useLayoutEffect**ï¼æ¯å¦æ³å¨ç»ä»¶**Mount**åï¼æ¿å°å®é DOM èç¹åä¸äºæä½ã

å¦æä½¿ç¨**useEffect**çè¯ï¼é¡µé¢åç°åçæä¸æ¶å»æè°ç¨æä»¬ç DOM ä¿®æ¹ï¼å¯è½ä¼é æå±å¹éªçã

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

**useRef**å¤§å¤æ°æåµé½æ¯ç¨æ¥è·å DOM èç¹ã

é¤äºæ®éçç¨æ³ï¼æä»¬ä¹å¯ä»¥ç¨æ¥ä¿å­å½æ°ä½åçä¸´æ¶åéã

```javascript
const value = useRef({ value: 1 });
```

ä¿®æ¹å®ä¹ä¸ä¼ä½¿ç»ä»¶åçæ´æ°ï¼æä»¥å®ä¸åºè¯¥ç¨æ¥ä½ä¸º**useEffect**çä¾èµé¡¹ã

<v-click>

```javascript
useEffect(() => {
  // xxxx
}, [eleRef.current]);
```

è¿æ ·åæ¯æ æçã

</v-click>

---

å¨å¼éç¹æè·¯ï¼å½æ°ä¹æ¯åéï¼å¶å®ä¹å¯ä»¥ç¨**useRef**ã

```javascript
const callback = useRef(() => null);
```

ä½¿ç¨**useRef**æ¥ä»ä»ç¨æ¥ä¿å­å½æ°çè³ä¼ä¼äº**useCallback**ã

<v-click>

ç¨é­åå**useRef**æ¥å®ç°ä¸ä¸ªæå¼ºç**useCallback**ã

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

# æ´¾ç state

æ´¾ç state ç¨éä¿çè¯æ¥è®²å°±æ¯å° props çå¼èµå¼ç» stateã

è¿ä¸ªåè½å¨ç±»ç»ä»¶éå®ç°éå¸¸ç®åï¼æä¸é¨æä¾ççå½å¨æå½æ°**getDerivedStateFromProps**ï¼ä½æ¯**Hooks**æ²¡æå®ç°è¿ä¸ªæ¹æ³ã

```javascript
static getDerivedStateFromProps(props){
    if(props.value){
        return { value: props.value }; // æpropsçvalueèµå¼ç»stateçvalue
    }
}
```

ä¸ºä»ä¹ä¼æè¿æ ·çéæ±å¢ï¼å¶å®å¨ç»ä»¶å¼åè¿ç¨ä¸­å¾å¸¸è§ã

---

å¯¹ä½¿ç¨èæ¥è¯´æ¯åæ§ç»ä»¶ï¼å¿é¡»ä¼ å¥**value**å**onChange**ã

```javascript
const Input = ({ value, onChange }) => {
  return <input value={value} onChange={onChange} />;
};
```

ä¸ä¸ªå¥½çç»ä»¶ï¼ä¸åºè¯¥éè¦ä½¿ç¨èæä¾é¢å¤ç**value**æ¥æ§å¶è¾å¥æ¡ã

<v-click>

å¯¹ä½¿ç¨èæ¯éåæ§ç»ä»¶ï¼åªè½éè¿**onChange**è·åå¼ï¼æ æ³ä¼ å¥**value**ä¿®æ¹å¼ ã

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

å¦æç±åé¨æ§å¶ï¼å¤é¨å°±æ æ³ä¿®æ¹è¾å¥æ¡çå¼ã

</v-click>

---

ä¸ä¸ªè¾å¥æ¡ç»ä»¶ï¼åæ§åéåæ§åºè¯¥æ¯å¯éçï¼æä»¬å¸ææ²¡æä¼ å¥**value**çæ¶åï¼è¾å¥æ¡å¯ä»¥è¾å¥å¼ï¼ä¼ å¥**value**æ¶ï¼è¾å¥æ¡çå¼ç±å¤é¨æ¥æ§å¶ã

```javascript {all|3,4,5|all}
const Input = ({ value, onChange }) => {
  const [inputValue, setInputValue] = useState(value);
  useEffect(() => {
    setInputValue(value); // å¤é¨valueå¼æ¹åï¼åæ­¥åé¨valueå¼
  }, [value]);
  const onValueChange = (event) => {
    setInputValue(event.target.value);
    onChange?.(event); // è¾å¥æ¡å¼æ¹åï¼åæ¶ä¿®æ¹å¤é¨å¼
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

å¾å¤æ¶åç»ä»¶å¼åä¼éå°è¿æ ·çæåµï¼æ³æ**props**ä¸­ç**value**èµå¼ç»**state**ï¼è¿ç§æåµå°±å«æ´¾ç**state**ã

<v-click>

å¦æå¤é¨ç**props**æ¹åï¼æä»¬åéè¦éæ°è®¾ç½®**state**ï¼å¯è½ä½¿ç¨ä¸è¿°æ¹æ³ï¼ä½æ¯è¿ç§åæ³å¶å®æ¯æé®é¢çã

1. å¤é¨**value**æ¹åï¼è§¦åè¯¥ **<Input\>** ç»ä»¶æ´æ°ï¼ç¬¬ä¸æ¬¡"render"ã

2. åé¨**useEffect**æ£æ¥å°ä¾èµé¡¹**value**æ¹åï¼è°ç¨**setInputValue**ï¼ç¬¬äºæ¬¡"render"ã

è¿æ ·çåæ³æ¯æ æ³é¿åç¬¬äºæ¬¡æ¸²æçé®é¢ã

</v-click>

---

# React å®ç½ä¸çç­æ¡

```javascript {all|1,2|4,5,6,78|all}
function Input({ value, onChange }) {
  const [inputValue, setInputValue] = useState(value);
  const prevValue = useRef(null);

  if (value !== prevValue.current) {
    // value èªä¸æ¬¡æ¸²æä»¥æ¥åçè¿æ¹åãæ´æ° inputValueã
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

> åçè¿æè®¸æç¹å¥æªï¼ä½æ¸²ææé´çä¸æ¬¡æ´æ°æ°æ°å°±æ¯ getDerivedStateFromProps ä¸ç´ä»¥æ¥çæ¦å¿µã

<v-click>

å®ä¾ç¶ä¼æ§è¡ä¸¤æ¬¡ï¼ä½æ¯ä¸æ¯ä¸¤æ¬¡"render"ã

å®éå®ç½æ¹æ¡ä¹ä¸æ¯æä¼è§£ã

</v-click>

---

# ææçæä¼è§£

æä»¬ææçæ¯ä¸è¦åºç°ç¬¬äºæ¬¡å½æ°çæ§è¡ã

èä¸éè¿**setState**å°±è½ä¿®æ¹å¼çæ¹æ³åªæ**useRef**äºã

<iframe src="https://codesandbox.io/embed/vigilant-ives-1yicg?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:30vh; border:0; border-radius: 4px; overflow:hidden;"
     title="vigilant-ives-1yicg"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

---

# æ°æ®ç®¡çåºæ¯æä¹æ´æ°æ°æ®çï¼

å¨Reactéï¼æä»¬çæ³è¦è§¦åç»ä»¶æ´æ°ï¼åªè½ç¨ç¸å³ç**setState**æ¹æ³ï¼ä½æ¯ç¬¬ä¸æ¹çæ°æ®ç®¡çåºæ¯æä¹åå°æ´æ°çå¢ï¼

åReduxï¼å®åªæ¯ä¸ä¸ªä¸Reactæ¯«æ å³ç³»çJSåï¼æ³è¦å¨Raectéä½¿ç¨è¿éè¦å®è£React-Reduxã

<v-click>

å®ä»¬çæ°æ®æ´æ°åçé½ç¦»ä¸å¼**forceUpdate**ã

</v-click>

<v-click>

```javascript
const useForceUpdate = () => {
  return useReducer((x) => x + 1, 0)[1];
};
```
å¨Hookéçå®ç°æ éå°±æ¯å¼ºå¶çè®¾ç½®ä¸ä¸ª**state**å¼ï¼è®©å®æ´æ°æ°æ®ã

</v-click>

---

Reduxæ¯ä¸ä¸ªåå¸è®¢éçæºå¶ï¼åªéè¦å¨**subscribe**éæ·»å ä¸**forceUpdate**ï¼å°±å¯ä»¥å¾è½»æ¾çå®ç°**React-Redux**ã

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

**React-Redux**çå®æ¹**Hook**æ¹æ¡å¶å®å°±æ¯è¿æ ·çãç°å¨æµè¡çç¬¬ä¸æ¹æ°æ®æµï¼åºæ¬é½æ¯è¿æ ·è®¢éæ´æ°å**forceUpdate**çæ¹æ¡ã

</v-click>

---

# æ´å¥½çReactçæ°æ®æµæ¹æ¡

```javascript
const reducer = (value, type) => (type === "increase" ? value + 1 : value - 1);
const { Provider } = React.createContext();
// ...
const [state, dispatch] = useReducer(reducer, 0);
return <Provider value={{ state, dispatch }} >
// ...
```
<v-click>

å®æ¹çæ¹æ¡å°**Hooks**æ¾è¿**Context**ï¼è¿ç§æ¹æ¡ä¸ç¤¾åºç¬¬ä¸æ¹æ¹æ¡çä¸åç¹å¨äºå®ä¸éè¦**forceUpdate**ï¼æ¯æ¬¡æ´æ°çåæ¶**Context**ä¹ä¼æ¹åï¼å®å®å¨å¨çå®æ¹**Hooks**ã

ç¤¾åºåºäºè®¢éæºå¶**forceUpdate**æ´æ°å¦**React-Reudx**ï¼å®ä»¬ç**Context**æ¯ä¸åçï¼æä»¥æè½æä¾ä¸ä¸ªç¨³å®å¯æ§ç**store**ã

</v-click>

---

# Contextçé®é¢

**Context**è§£å³æ¹æ¡çé®é¢æ¯è¾ä¸¥éï¼ä¸»è¦å°±æ¯æ å³çãéå¤çæ¸²æã

```javascript
    // context => { a:1, b:2 }
    // ç»ä»¶A
    const { a } = useContext
    // ç»ä»¶B
    const { b } = useContext
    b += 1
``` 

ä¸æ¦**Context**æ´æ°ï¼å¶ä½ææä½¿ç¨äº**Context**çç»ä»¶é½ä¼æ´æ°ï¼åå å°±æ¯å®æ¯ä¸ä¸ªæ´ä½ã

å¶å®è¿ä¸ªé®é¢å®æ¹ä¹æä¾äºä¸ä¸ªä¸å¤ªåå¥½ç**feature**ã

---

# observedBits

è¿ç®æ¯ä¸ä¸ªéèçæ¯è¾æ·±çä¸è¥¿ï¼ææ¨æµä»19å¹´å¼å§å°±æäºã

```javascript
const bits = {
  user: 0b01,
  password: 0b10,
}
const context = useContext(Context, bits.user); // è¿ä¸ªcontextä½¿ç¨äºuserå­æ®µ

let result = 0;
// æ è¯userå­æ®µåçåå
if (oldValue.user !== newValue.user){
  result |= bits.user; // 0 -> 0b01
}
return result;
```

éè¿äºè¿å¶çå¤æ­æ¯å¦éè¦æ´æ°ã

<CodeSandBox>
<iframe src="https://codesandbox.io/embed/zen-ishizaka-q0yzi?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="zen-ishizaka-q0yzi"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
  />
</CodeSandBox>

---

å®æ¹çæ¹æ¡

```javascript
const {a} = useContextSelector(Context, context => context.a);
const {b} = useContextSelector(Context, context => context.b);
const derived = useMemo(() => computeDerived(a, b), [a, b]);
```

åªæå¨**experimental**åæä¼æçåè½ã

---

# Tailwindcss

[Tailwind CSS](https://tailwindcss.com/) æ¯ä¸ä¸ªåè½ç±»ä¼åç CSS æ¡æ¶ï¼å®éæäºè¯¸å¦ flex, pt-4, text-center å rotate-90 è¿æ ·ççç±»ï¼å®ä»¬è½ç´æ¥å¨èæ¬æ è®°è¯­è¨ä¸­ç»åèµ·æ¥ï¼æå»ºåºä»»ä½è®¾è®¡ã
