# 13 useImperativeHandle基础用法

## useImperativeHandle概念解释
我们第八个要学习的Hook(钩子函数)是useImperativeHandle，他的作用是“勾住”子组件中某些函数(方法)供父组件调用。  

先回顾一下之前学到的。  
第1个知识点：  
react属于单向数据流，父组件可以通过属性传值，将父组件内的函数(方法)传递给子组件，实现子组件调用父组件内函数的目的。  

第2个知识点：    
1、useRef 可以“勾住”某些本组件挂载完成或重新渲染完成后才拥有的某些对象。  
2、React.forwardRef 可以“勾住”某些子组件挂载完成或重新渲染完成后才拥有的某些对象。  
上面无论哪种情况，由于勾住的对象都是渲染后的原生html对象，父组件只能通过ref调用该原生html对象的函数(方法)。

如果父组件想调用子组件中自定义的方法，该怎么办？  
答：使用useImperativeHandle()。

让我们回到useImperativeHandle基础学习中。


## useImperativeHandle是来解决什么问题的？
答：useImperativeHandle可以让父组件获取并执行子组件内某些自定义函数(方法)。本质上其实是子组件将自己内部的函数(方法)通过useImperativeHandle添加到父组件中useRef定义的对象中。  

补充说明：  
1、useRef创建引用变量  
2、React.forwardRef将引用变量传递给子组件  
3、useImperativeHandle将子组件内定义的函数作为属性，添加到父组件中的ref对象上。  

因此，如果想使用useImperativeHandle，那么还要结合useRef、React.forwardRef一起使用。   


## useImperativeHandle函数源码：  
回到useImperativeHandle的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useImperativeHandle<T>(
      ref: {|current: T | null|} | ((inst: T | null) => mixed) | null | void,
      create: () => T,
      deps: Array<mixed> | void | null,
    ): void {
      const dispatcher = resolveDispatcher();
      return dispatcher.useImperativeHandle(ref, create, deps);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^  


## useImperativeHandle基本用法
useImperativeHandle(ref,create,[deps])函数前2个参数为必填项，第3个参数为可选项。  
第1个参数为父组件通过useRef定义的引用变量；  
第2个参数为子组件要附加给ref的对象，该对象中的属性即子组件想要暴露给父组件的函数(方法)；  
第3个参数为可选参数，为函数的依赖变量。凡是函数中使用到的数据变量都需要放入deps中，如果处理函数没有任何依赖变量，可以忽略第3个参数。

请注意：  
1、这里面说的“勾住子组件内自定义函数”本质上是子组件将内部自定义的函数添加到父组件的ref.current上面。  
2、父组件若想调用子组件暴露给自己的函数，可以通过 res.current.xxx 来访问或执行。  


##### 代码形式：  

    const xxx = () => {
        //do smoting...
    }
    useImperativeHandle(ref,() => ({xxx}));

上述代码中，useImperativeHandle(ref,() => ({xxx})) 其实是 useImperativeHandle(ref,() => {return {xxx:xxx}})的简写。  

特别注意：() => ({xxx}) 不可以再简写成 () => {xxx}，如果这样写会直接react报错。  
因为这两种写法意思完全不一样：  
1、() => ({xxx}) 表示 返回一个object对象，该对象为{xxx}  
2、() => {xxx} 表示 执行 xxx 语句代码  


##### 拆解说明：  

1、子组件内部先定义一个 xxx 函数  
2、通过useImperativeHandle函数，将 xxx函数包装成一个对象，并将该对象添加到父组件内部定义的ref中。  
3、若 xxx 函数中使用到了子组件内部定义的变量，则还需要将该变量作为 依赖变量 成为useImperativeHandle第3个参数，上面示例中则选择忽略了第3个参数。  
4、若父组件需要调用子组件内的 xxx函数，则通过：res.current.xxx()。  
5、请注意，该子组件在导出时必须被 React.forwardRef()包裹住才可以。  


## useImperativeHandle使用示例：  

举例，若某子组件的需求为：  
1、有变量count，默认值为0  
2、有一个函数 addCount，该函数体内部执行 count+1  
3、有一个按钮，点击按钮执行 addCount 函数

父组件的需求为：  
1、父组件内使用上述子组件  
2、父组件内有一个按钮，点击执行上述子组件内定义的函数 addCount

子组件的代码为：  

    import React,{useState,useImperativeHandle} from 'react'

    function ChildComponent(props,ref) {
      const [count,setCount] =  useState(0); //子组件定义内部变量count
      //子组件定义内部函数 addCount
      const addCount = () => {
        setCount(count + 1);
      }
      //子组件通过useImperativeHandle函数，将addCount函数添加到父组件中的ref.current中
      useImperativeHandle(ref,() => ({addCount}));
      return (
        <div>
            {count}
            <button onClick={addCount}>child</button>
        </div>
      )
    }

    //子组件导出时需要被React.forwardRef包裹，否则无法接收 ref这个参数
    export default React.forwardRef(ChildComponent);


父组件的代码为：  

    import React,{useRef} from 'react'
    import ChildComponent from './childComponent'

    function Imperative() {
      const childRef = useRef(null); //父组件定义一个对子组件的引用

      const clickHandle = () => {
        childRef.current.addCount(); //父组件调用子组件内部 addCount函数
      }

      return (
        <div>
            {/* 父组件通过给子组件添加 ref 属性，将childRef传递给子组件，
                子组件获得该引用即可将内部函数添加到childRef中 */}
            <ChildComponent ref={childRef} />
            <button onClick={clickHandle}>child component do somting</button>
        </div>
      )
    }

    export default Imperative;


#### 思考一下真的有必要使用useImperativeHandle吗？

从实际运行的结果，无论点击子组件还是父组件内的按钮，都将执行 addCount函数，使 count+1。  

react为单向数据流，如果为了实现这个效果，我们完全可以把需求转化成另外一种说法，即：  
1、父组件内定义一个变量count 和 addCount函数  
2、父组件把 count 和 addCount 通过属性传值 传递给子组件  
3、点击子组件内按钮时调用父组件内定义的 addCount函数，使 count +1。

你会发现即使把需求中的 父与子组件 描述对调一下，“最终实际效果”是一样的。  

所以，到底使用哪种形式，需要根据组件实际需求来做定夺。  


#### 说一个有点绕的情况
子组件导出时：  
1、假设某个子组件为了提高性能，导出时需要用React.memo包裹。  
2、可是他也需要暴露自己内部函数给父组件，导出时也需要用React.forwardRef包裹。  

子组件内部函数：  
1、假设该组件内部函数为了性能，需要用到 useCallback包裹该函数。  
2、同时为了让将该函数暴露给父级，也需要用 useImperativeHandle包裹。

呵，该怎么办，层层包裹吗？  虽然性能提升了，可是那样的代码可读性还有多少，怎么办？  
答：不知道，反正本系列文章只是单独来讲解某个hook怎么使用，这种复杂包裹的情况，你自己看着办吧。  

事实上这种情况出现的几率非常小，当我们开发react组件时，不应该为了使用某hook而使用。还是应该是依据单向数据流的原则来做设计方案。  
就好像本章中示例代码，其实完全可以不用useImperativeHandle，而是继续使用最常见的父组件属性传值给子组件的方式。   


---

至此，关于useImperativeHandle基础用法已经讲完，没有高级用法，直接进入下一个Hook。

欢迎进入下一章节：[useLayoutEffect基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/14%20useLayoutEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)
