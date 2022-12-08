## 19 useTransition基础用法

## useTransition概念介绍
react提供了``useDeferredValue``发挥类似防抖节流的作用，而``useTransition``也是类似的作用，但是该hook是通过降低数据渲染的优先级来达到优先更新其他数据

## useTransition用来解决什么问题？
- 首先给定一个场景，开发时经常会遇到需要联想输入，也就是输入的同时要返回联想搜索结果的列表。
- 但是这个列表有时返回值非常的长，有时会导致用户输入值的更新缓慢，这里就产生了一个问题，当页面有大量UI更新的时候，怎么处理数据更新不会卡顿。
- 可以手写防抖节流，防抖有一个弊端，当我们长时间的持续输入（时间间隔小于防抖设置的时间），页面就会长时间都不到响应。而startTransition 可以指定 UI 的渲染优先级，哪些需要实时更新，哪些需要延迟更新。即使用户长时间输入最迟 5s 也会更新一次。
- 也可以用``useDeferredValue``，也可以用“可视窗口加载”的方案，
在这里介绍怎么用``useTransition``解决

## useTransition源码
回到useTransition的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。 

```typescript
export function useTransition(): [
  boolean,
  (callback: () => void, options?: StartTransitionOptions) => void,
] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useTransition();
}
```
再根据引入文件，到``react-reconciler/src/ReactInternalTypes.js``找到Dispatch里最终调用的useTransition
```typescript
  useTransition(): [
    boolean,
    (callback: () => void, options?: StartTransitionOptions) => void,
  ],
```
上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^  

## useTransition基本用法
useTransition()函数可以不传参，传参可以传一个毫秒值用来修改最迟更新时间，startTransition回调里的赋值将会被降低优先级。isPending 指示过渡任务何时活跃以显示一个等待状态。

> 代码形式
```typescript
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setCount(count + 1);
})
```
传参写法
```typescript
// 延迟两秒
const [isPending, startTransition] = useTransition(2000);

startTransition(() => {
  setCount(count + 1);
})
```

## useTransition使用示例
举例：搜索引擎的关键词联想。一般来说，对于用户在输入框中输入都希望是实时更新的，如果此时联想词比较多同时也要实时更新的话，这就可能会导致用户的输入会卡顿。这样一来用户的体验会变差，这并不是我们想要的结果。

我们将这个场景的状态更新提取出来：一个是用户输入的更新；一个是联想词的更新，这个两个更新紧急程度显然前者大于后者。

这里更新效果可能还不够明显，可以打开浏览器控制台，点击``performance insights``项，在Measure page load右边有个下拉选项，在cpu那栏的右边下拉选择``4x slowdown``可以将浏览器运行速度调慢四倍，这样卡顿会明显些。

这里拆分为两个组件，父组件是useTransition的使用，子组件是列表渲染，代码如下：

父组件：
```tsx
import { useState, useTransition } from 'react'
import ProductList from './components/ProductList'

// 列表数据的生成
export function generateProducts() {
  const products: Array<string> = []
  for (let i = 0; i < 10000; i++) {
    products.push(`Product ${i + 1}`)
  }
  return products
}

// 列表数据
const dummyProducts = generateProducts()

// 用户输入时过滤搜索，达到一个联想输入的效果
function filterProducts(filterTerm) {
  if (!filterTerm) {
    return dummyProducts
  }
  return dummyProducts.filter((product) => product.includes(filterTerm))
}

function App() {
  const [isPending, startTransition] = useTransition()
  const [filterTerm, setFilterTerm] = useState('')

  const filteredProducts = filterProducts(filterTerm)

  function updateFilterHandler(event) {
    // 列表数据赋值的运行等级
    startTransition(() => {
      setFilterTerm(event.target.value)
    })
  }

  return (
    <div id="app">
      <input type="text" onChange={updateFilterHandler} />
      {isPending && <p style={{ color: 'white' }}>更新列表。. </p>}
      <ProductList products={filteredProducts} />
    </div>
  )
}

export default App
```

子组件：
```tsx
import { useDeferredValue } from "react";

function ProductList({ products }) {
  const deferredProducts = useDeferredValue(products);
  return (
    <ul>
      {deferredProducts.map((product, index) => (
        <li key={index}>{product}</li>
      ))}
    </ul>
  );
}

export default ProductList;
```

通过这个案例，相信你对useMemo的机制和用法一定有所掌握。

---

至此，关于useTransition基础用法已经讲完。