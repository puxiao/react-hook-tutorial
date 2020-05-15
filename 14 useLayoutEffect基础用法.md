# 14 useLayoutEffect基础用法

## useLayoutEffect概念解释
我们第九个要学习的Hook(钩子函数)是useLayoutEffect，他的作用是“勾住”挂载或重新渲染完成这2个组件生命周期函数。useLayoutEffect使用方法、所传参数和useEffect完全相同。

他们的不同点在于，你可以把useLayoutEffect等同于componentDidMount、componentDidUpdate，因为他们调用阶段是相同的。而useEffect是在componentDidMount、componentDidUpdate调用之后才会触发的。

也就是说，当组件所有DOM都渲染完成后，同步调用useLayoutEffect，然后再调用useEffect。

useLayoutEffect永远要比useEffect先触发完成。

那通常在useLayoutEffect阶段我们可以做什么呢？  
答：在触发useLayoutEffect阶段时，页面全部DOM已经渲染完成，此时可以获取当前页面所有信息，包括页面显示布局等，你可以根据需求修改调整页面。  

请注意，useLayoutEffect对页面的某些修改调整可能会触发组件重新渲染。如果是对DOM进行一些样式调整是不会触发重新渲染的，这点和useEffect是相同的。  

在react官方文档中，明确表示只有在useEffect不能满足你组件需求的情况下，才应该考虑使用useLayoutEffect。  官方推荐优先使用useEffect。  

请注意：如果是服务器渲染，无论useEffect还是useLayoutEffect 都无法在JS代码加载完成之前执行，因此都会收到错误警告。  服务器渲染时若想使用useEffect，解决方案不在本章中讨论。

让我们回到useLayoutEffect基础学习中。


## useLayoutEffect是来解决什么问题的？
答：useLayoutEffect的作用是“当页面挂载或渲染完成时，再给你一次机会对页面进行修改”。  

如果你选择使用useLayoutEffect，对页面进行了修改，更改样式不会引发重新渲染，但是修改变量则会触发再次渲染。  
如果你不使用useLayoutEffect，那么之后就应该调用useEffect。  

补充说明：  
1、优先使用useEffect，useEffect无法满足需求时再考虑使用useLayoutEffect。  
2、useLayoutEffect先触发，useEffect后触发。  
3、useEffect和useLayoutEffect在服务器端渲染时，都不行，需要寻求别的解决方案。  

## useLayoutEffect函数源码：  
回到useLayoutEffect的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useLayoutEffect(
      create: () => (() => void) | void,
      deps: Array<mixed> | void | null,
    ): void {
      const dispatcher = resolveDispatcher();
      return dispatcher.useLayoutEffect(create, deps);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^  你只需知道useLayoutEffect的用法和useEffect一模一样即可。  


## useLayoutEffect基本用法

useLayoutEffect的用法和useEffect的用法相同，所以不再阐述。  


## useLayoutEffect使用示例：  

请原谅，目前竟然找不到一个useLayoutEffect合适的例子，因为能够想到的应用场景其实都可以用useEffect来代替。  

那只能贴出一段简单的代码，让你看确认一下，useLayoutEffect先于useEffect触发调用。

代码示例如下：

    import React,{useState,useEffect,useLayoutEffect} from 'react'

    function LayoutEffect() {
      const [count,setCount] = useState(0);

      useEffect(() => {
        console.log('useEffect...');
      },[count]);

      useLayoutEffect(() => {
        console.log('useLayoutEffect...');
      },[count]);

      return (
        <div>
            {count}
            <button onClick={() => {setCount(count+1)}}>Click</button>
        </div>
      )
    }
    export default LayoutEffect


实际运行就会发现：  
无论是首次挂载，还是重新渲染，console面板中，输出顺序都是  
useLayoutEffect...  
useEffect...  

也就确认，先执行useLayoutEffect，后执行useEffect。  

---

至此，关于useLayoutEffect基础用法已经讲完，没有高级用法，直接进入下一个Hook。

欢迎进入下一章节：[useDebugValue基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/15%20useDebugValue%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)
