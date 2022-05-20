# 12 useRef基础用法

## useRef概念解释

我们第七个要学习的Hook(钩子函数)是useRef，他的作用是“勾住”某些组件挂载完成或重新渲染完成后才拥有的某些对象，并返回该对象的引用。该引用在组件整个生命周期中都固定不变，该引用并不会随着组件重新渲染而失效。

上面这段话，就算你认真读几遍，估计也是一头雾水，到底说的是啥？  
我也实在想不出其他更加通俗的语言来描述useRef，不过经过下面的详细分解描述，相信能帮到你来理解useRef。  

##### “某些组件挂载完成或重新渲染完成后才拥有的某些对象”：  

这句话中的“某些对象”主要分为3种：JSX组件转换后对应的真实DOM对象、在useEffect中创建的变量、子组件内自定义的函数(方法)。

**第1：JSX组件转换后对应的真实DOM对象**：  
举例：假设在JSX中，有一个输入框<input type='text' /\>，这个标签最终将编译转换成真正的html标签中的<input type='text'/\>。  
你应该知道以下几点：  
1、JSX中小写开头的组件看似和原生html标签相似，但是并不是真的原生标签，依然是react内置组件。  
2、什么时候转换？ 虚拟DOM转化为真实DOM   
3、什么时候可访问？组件挂载完成或重新渲染完成后  

对于上面举例中的那个转换后的<input/\> 真实DOM，只有组件挂载完成或重新渲染完成后才可以访问，它就就属于“某些组件挂载完成或重新渲染完成后才拥有的某些对象”。  

特别强调：useRef只适合“勾住”小写开头的类似原生标签的组件。如果是自定义的react组件(自定义的组件必须大写字母开头)，那么是无法使用useRef的。      

思考：如何获取这个 <input/\> 真实DOM呢？  
答：用useRef。  


**第2：在useEffect中创建的变量**：  
举例，请看以下代码：  

    useEffect(() => {
        let timer = setInterval(() => {
            setCount(prevData => prevData +1);
        }, 1000);
        return () => {
            clearInterval(timer);
        }
    },[]);

上述代码中，请注意这个timer是在useEffect中才定义的。  

思考：useEffect 以外的地方，该如何获取这个 timer 的引用？  
答：用useRef 


**第3：子组件内自定义的函数(方法)**  
由于需要结合useImperativeHandle才可以实现，而useImperativeHandle目前还未学习，所以本章中不讨论这个怎么实现。  
本章只讲前2中应用场景。  



##### “并返回该对象的引用”：  

上面的前2种情况，都提到用useRef来获取对象的引用。具体如何获取，稍后在useRef用法中会有演示。  


##### “该引用在组件整个生命周期中都固定不变”：

假设通过useRef获得了该对象的引用，那么当react组件重新渲染后，如何保证该引用不丢失？  
答：react在底层帮我们做了这个工作，我们只需要相信之前的引用可以继续找到目标对象即可。

请注意：React.createRef()也有useRef相似效果，但是React.createRef无法全部适用上面提到的3种情况。  

让我们回到useRef基础学习中。


## useRef是来解决什么问题的？

答：useRef可以“获取某些组件挂载完成或重新渲染完成后才拥有的某些对象”的引用，且保证该引用在组件整个生命周期内固定不变，都能准确找到我们要找的对象。  
具体已经在useRef中做了详细阐述，这里不再重复。    

补充说明：  
1、useRef是针对函数组件的，如果是类组件则使用React.createRef()。   
2、React.createRef()也可以在函数组件中使用。  
只不过React.createRef创建的引用不能保证每次重新渲染后引用固定不变。如果你只是使用React.createRef“勾住”JSX组件转换后对应的真实DOM对象是没问题的，但是如果想“勾住”在useEffect中创建的变量，那是做不到的。

2者都想可以“勾住”，只能使用useRef。    


## 注意注意

在后面useImperativeHandle的学习中，你会知道useRef还可以 “勾住并调用” 子组件内定义的函数(方法)。 



<br>

> 以下内容更新于 2022.04.06

## 特别注意：修改 .current 的值并不会触发组件重新渲染

在本文开头介绍 useRef  时用了这句话 “useRef 是“勾住”某些组件挂载完成或重新渲染完成后才拥有的某些对象，并返回该对象的引用。”

也就是说 **先有了 组件渲染，之后才更新了 useRef 中 .current 的值。**

> 也就是说 useRef 变量的 current 的值实际上是 组件渲染 后的一个副产品。

**这句话暗含了另外一层含义：主动更新 useRef 变量的 .current 的值并不会触发组件重新渲染。**

例如下面这个示例：

```
import { useRef } from "react";

export default function MyButton() {
  const countRef = useRef(0)

  const handleClick = () => {
    countRef.current = countRef.current + 1
  };

  return <button onClick={handleClick}>Click me {countRef.current}</button>;
}
```

实际运行就会发现，在点击事件中我们修改了 countRef.current 的值，尽管该值确实发生了变化，可是并不会触发组件的重新渲染。

> 使用 useState() 产生的变量值发生变化后，是会触发组件重新渲染的。



<br>

> 以上内容更新于 2022.04.06



<br>


## useRef函数源码：  

回到useRef的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useRef<T>(initialValue: T): {|current: T|} {
      const dispatcher = resolveDispatcher();
      return dispatcher.useRef(initialValue);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^


## useRef基本用法

useRef(initialValue)函数只有1个可选参数，该参数为默认“勾住”的对象。绝大多数实际的情况是，默认“勾住”的对象在JSX未编译前(组件挂载或重新渲染后)根本不存在，所以更多时候都会传一个 null 作为默认值。如果不传任何参数，那么react默认将参数设置为undefined。  

就目前本人所理解的，日常使用过程中useRef(null)和useRef() 实际上是没有什么区别的。  

---

以下更新于 2020.12.10



**补充一下 React + TypeScript 知识点：**

useRef(null) 和 useRef() 在 React + TypeScript 项目中还是有差别的。

假设我们要勾住一个 <canvas /\> DOM元素，那么：

```
const canvasRef1 = useRef<HTMLCanvasElement>(null)
const canvasRef2 = useRef<HTMLCanvasElement>()
```

上面代码中：

1. canvasRef1.current 的类型为：HTMLCanvasElement | null
2. canvasRef2.current 的类型为：HTMLCanvasElement | null | undefined



以上更新于 2020.12.10

---




第2遍强调：本文提到的组件，默认都是指小写开头的类似原生标签的组件，不可以是自定义组件。  

接下来具体说说useRef关联对象的2种用法：  
1、针对 JSX组件，通过属性 ref={xxxRef} 进行关联。  
2、针对 useEffect中的变量，通过 xxxRef.current 进行关联。  


##### 代码形式：  

    //先定义一个xxRef引用变量，用于“勾住”某些组件挂载完成或重新渲染完成后才拥有的某些对象
    const xxRef = useRef(null);
    
    //针对 JSX组件，通过属性 ref={xxxRef} 进行关联
    <xxx ref={xxRef} />
    
    //针对 useEffect中的变量，通过 xxxRef.current 进行关联
    useEffect(() => {
       xxRef.current = xxxxxx;
    },[]);


##### 拆解说明：  

1、具体讲解已在上面示例代码中做了多项注释，此处不再重复；  

#### 'ref'补充说明

1、组件的 ref 为特殊属性名，他并不存在组件属性传值的 props 中。  
2、如果给一个组件设定了 ref 属性名，但是对应的值却不是由 useRef 创建的，那么实际运行中会收到react的报错，无法正常渲染。  

#### '<xxx\>'补充说明

1、useRef只能针对react中小写开头的类似原生标签的组件，所以这里用的是 <xxx\> 而不是 <Xxx\>。  

#### 'xxxRef.current'补充说明

1、当需要使用“勾住”的对象时，也是通过xxRef.current来获取该对象的。


## useRef使用示例1：  

若我们有一个组件，该组件只有一个输入框，我们希望当该组件挂载到网页后，自动获得输入焦点。  

需求分析：  
1、我们可以很轻松使用<input \>创建出这个输入框。  
2、需要使用useRef “勾住”这个输入框，当它被挂载到网页后，通过操作原生html的方法，将焦点赋予该输入框上。  

完整代码如下：  

    import React,{useEffect,useRef} from 'react'
    
    function Component() {
      //先定义一个inputRef引用变量，用于“勾住”挂载网页后的输入框
      const inputRef = useRef(null);
    
      useEffect(() => {
        //inputRef.current就是挂载到网页后的那个输入框，一个真实DOM，因此可以调用html中的方法focus()
        inputRef.current.focus();
      },[]);
    
      return <div>
          {/* 通过 ref 属性将 inputRef与该输入框进行“挂钩” */}
          <input type='text' ref={inputRef} />
        </div>
    }
    export default Component

注意：  
1、在给组件设置 ref 属性时，只需传入 inputRef，千万不要传入 inputRef.current。  
2、在“勾住”渲染后的真实DOM输入框后，能且只能调用原生html中该标签拥有的方法。  




## useRef使用示例2：  

若我们有一个组件，该组件的功能需求如下：  
1、组件中有一个变量count，当该组件挂载到网页后，count每秒自动 +1。  
2、组件中有一个按钮，点击按钮可以停止count自动+1。

需求分析：  
1、声明内部变量count用 useState  
2、可以在useEffect 通过setInterval创建一个计时器timer，实现count每秒自动 +1  
3、当组件卸载前，需要通过 clearInterval 将timer清除  
4、按钮点击处理函数中，也要通过 clearInterval 将timer清除   

假设我们不使用useRef，那该如何实现？  

为了确保timer可以被useEffect以外地方也能访问，我们通常做法是将timer声明提升到useEffect以外。  
代码如下：  

    import React,{useState,useEffect} from 'react'
    
    function Component() {
      const [count,setCount] = useState(0);
      const [timer,setTimer] = useState(null); //单独声明定义timer，目的是为了让组件内所有地方都可以访问到timer
    
      useEffect(() => {
        //需要用setTimer()包裹住 setInterval()
        setTimer(setInterval(() => {
            setCount((prevData) => {return prevData +1});
        }, 1000));
        return () => {
          //清除掉timer
          clearInterval(timer);
        }
      },[]);
    
      const clickHandler = () => {
        //清除掉timer
        clearInterval(timer);
      };
    
      return (
        <div>
            {count}
            <button onClick={clickHandler} >stop</button>
        </div>
      )
    }
    
    export default Component


如果使用useRef，该如何实现？
代码如下：  

    import React,{useState,useEffect,useRef} from 'react'
    
    function Component() {
      const [count,setCount] =  useState(0);
      const timerRef = useRef(null);//先定义一个timerRef引用变量，用于“勾住”useEffect中通过setIntervale创建的计时器
    
      useEffect(() => {
        //将timerRef.current与setIntervale创建的计时器进行“挂钩”
        timerRef.current = setInterval(() => {
            setCount((prevData) => { return prevData +1});
        }, 1000);
        return () => {
            //通过timerRef.current，清除掉计时器
            clearInterval(timerRef.current);
        }
      },[]);
    
      const clickHandler = () => {
        //通过timerRef.current，清除掉计时器
        clearInterval(timerRef.current);
      };
    
      return (
        <div>
            {count}
            <button onClick={clickHandler} >stop</button>
        </div>
      )
    }
    
    export default Component



**两种实现方式对比：**  

1、两种实现方式最主要的差异地方在于 如何创建组件内对计时器的引用。  
2、两种创建引用的方式，分别是：用useState创建的timer、用useRef创建的timerRef  
3、在使用setInterval时，相对来说timerRef.current更加好用简单，结构清晰，不需要像 setTimer那样需要再多1层包裹。  
4、timer更像是一种react对计时器的映射，而timerRef直接就是真实DOM中计时器的引用，timerRef能够调用更多的原生html中的JS方法和属性。  


**结论：**  
1、如果需要对渲染后的DOM节点进行操作，必须使用useRef。  
2、如果需要对渲染后才会存在的变量对象进行某些操作，建议使用useRef。  

第3遍强调：useRef只适合“勾住”小写开头的类似原生标签的组件。如果是自定义的react组件(自定义的组件必须大写字母开头)，那么是无法使用useRef的。 



<br>

> 以下内容更新于 2022.05.20

## useRef使用示例3：父组件调用子组件中的函数

**首先特别强调：除非情况非常特殊，否则一般情况下都不要采用 父组件调用子组件的函数 这种策略。**



<br>

**使用 useRef 实现父组件调用子组件中的函数 实现思路：**

1. 父组件中通过 useRef 定义一个钩子变量，例如 childFunRef

2. 父组件通过参数配置，将 childFunRef 传递给子组件

3. 子组件在自己的 useEffect() 中定义一个函数，例如 doSomting()

   > 划重点：一定要在 useEffect() 中定义 doSomting()，不能直接在子组件内部定义。
   >
   > 因为如果 doSomting() 定义在子组件内部，那么就会造成每一次组件刷新都会重新生成一份 doSomthing()

4. 然后将 doSomting() 赋值到 childFunRef.current 中

5. 这样，当父组件想调用子组件中的 doSomting() 时，可执行 childFunRef.current.doSomting()



<br>

具体示例代码：

**ParentComponent**

```
import { useRef } from "react";
import ChildComponent from "./child";

const ParentComponent = () => {
  const childFunRef = useRef();
  const handleOnClick = () => {
    if (childFunRef.current) {
      childFunRef.current.doSomething();
    }
  };
  return (
    <div>
      <ChildComponent funRef={childFunRef} />
      <button onClick={handleOnClick}>执行子项的doSomething()</button>
    </div>
  );
};

export default ParentComponent;
```



<br>

**ChildComponent**

```
import { useEffect, useState } from "react";

const ChildComponent = ({ funRef }) => {
  const [num, setNum] = useState(0);
  useEffect(() => {
    const doSomething = () => {
      setNum(Math.floor(Math.random() * 100));
    };
    funRef.current = { doSomething }; //在子组件中修改父组件中定义的childFunRef的值
  }, [funRef]);
  return <div>{num}</div>;
};

export default ChildComponent;
```



<br>

**特别说明：**

1. 下一章要讲解的 useImperativeHandle 也是用来实现 父组件调用子组件内定义的函数的。

2. 再次强调，如非必要，真的不要使用 父组件调用子组件内函数 这种策略。

   > 最近我遇到了一个需求，子组件是一个第三方写好的轮播图，父组件需要调用这个轮播图的 next() 的函数来切换下一张，所以才使用了这种策略。



> 以上内容更新于 2022.05.20



<br>

---



> 以下内容更新于2020.11.18

#### 在 TypeScript 中使用 useRef 创建计时器注意事项：

在上面代码示例中，请注意这一行代码：

```
timerRef.current = setInterval(() => {
        setCount((prevData) => { return prevData +1});
    }, 1000);
```

如果是在 TS 语法下，上面的代码会报错误：

```
不能将类型“Timeout”分配给类型“number”。
```

**Timeout ???**

造成这个错误提示的原因是：

1. TypeScript 是运行在 Nodejs 环境下的，TS 编译之后的代码是运行在浏览器环境下的。
2. Nodejs 和浏览器中的 window 他们各自实现了一套自己的 setInterval 
3. 原来代码 timerRef.current = setInterval( ... ) 中 setInterval 会被 TS 认为是 Nodejs 定义的 setInterval，而  Nodejs 中 setInterval 返回的类型就是 NodeJS.Timeout。
4. 所以，我们需要将上述代码修改为：timerRef.current = window.setInterval( ... )，明确我们调用的是 window.setInterval，而不是 Nodejs 的 setInterval。



**附一个 TS 代码示例：**

```
import React, { useRef, useEffect } from 'react'

const MyTemp = () => {
    const timer = useRef<number | undefined>()

    useEffect(() => {
        timer.current = window.setInterval(() => {
            console.log(0)
        }, 1000)

        return () => {
            clearInterval(timer.current)
        }
    }, [])
    return (
        <div></div>
    )
}

export default MyTemp
```

> 以上内容更新于2020.11.18

____



> 以下内容更新于2020.12.03

#### 在 TypeScript 中给 useRef.current 赋值的注意事项

在 jsx 文件中，以下代码是不会有问题的。

```
const myRef = useRef(null)
...
myRef.current = xxxx
```

但是，在我们使用 TypeScript 之后，按照习惯改成以下代码：

```
const myRef = useRef<Xxx>(null)
...
myRef.current = xxx
```

此时，会收到 TypeScript 的报错：**无法分配到 "current" ，因为 myRef.current 是只读属性。**



**报错原因：**

React 的作者并没有规定使用 useRef(null) 之后 myRef.current 就不可以再修改了。

但是 TypeScript 的作者认为，若使用 useRef(null) 之后，myRef 就应该交由 React 来托管，外界不应该有权利去修改 myRef.current，因此此时会把 myRef.current 当做只读属性。



**解决方式：**

解决方式1：不给 useRef 设置 null 这个默认值

```
const myRef = useRef<Xxx>()
```

解决方式2：就是将原本的类型定义，修改成以下：

```
const myRef = useRef<Xxx | null>(null)

//或者是

const myRef = useRef<Xxx | undefined>()
```

myRef.current 的数据类型，除了 Xxx 之外，再加上 null 或 undefined ，这样 TypeScript 就认为  myRef.current 可能中途会发生修改，因此不会再将其设置为只读属性，此时再去执行 `myRef.current = xxx` 不再会报错。



**验证一下：**

我们再去看看上面 2020.11.18 更新的 TypeScript 代码示例中：

由于将来需要执行 timer.current =  window.setInterval ( ... )，也就是说需要给 timer.current 赋值。

所以在定义时就使用以下方式，以确保 timer.current 不会被 TS 认为是只读属性：

```
const timer = useRef<number | undefined>()
```



> 以上内容更新于2020.12.03

---



## 那如何“勾住”自定义组件中的“小写开头的类似原生标签的组件”？  

答：使用React.forwardRef()。  

##### 你是否思考过这个问题：自定义组件到底是什么？  

首先看一下“小写开头的类似原生标签的组件”，例如<button\>、<input \>，我们很容易理解他是react内置的类似原生DOM的组件，最终都将直接转换成对应的真实DOM。  

那自定义组件又该如何理解，如何定义呢？  

假设我们有一个自定义组件<MyComponent\>，那么有以下几点是可以肯定的：  
1、<MyComponent\>内部return出去的，可以是小写开头的类似原生标签的组件，也可以是其他自定义组件。   
2、无论嵌套多少次，最底层组件return出去的，一定是小写开头的类似原生标签的组件。  
3、<MyComponent\>内部一定创建了变量、处理函数等等。  
4、挂载或渲染后的实际网页中，并不会存在<MyComponent\>这个标签，存在的依然是各种原生html标签。

为了简化更加容易理解，暂时姑且先把“小写开头的类似原生标签的组件”直接当做“原生html标签”。  
那么“自定义组件”和“原生html标签”究竟区别在哪里呢？  

先不回答这个问题，再说另外一个问题：交互式html页面都有哪些构成？  
答：有各种html标签 + JS对象(JS中定义的变量和函数)  

那我们使用react开发页面，“原生html标签”有了，那还缺什么？ 当然是 JS对象(JS中定义的变量和函数)。  

再回顾一下问题：“自定义组件”和“原生html标签”究竟区别在哪里呢？  
答：“自定义组件”除了拥有“原生html标签”，还拥有JS对象(JS中定义的变量和函数)。  

再回顾一下开始的疑问：自定义组件到底是什么？  
答：其实根本不存在自定义组件，所谓自定义组件，只不过是react给我们的各种语法糖，react并没有创造另外一门语言，react整体就是原生JS的语法糖。  

JSX语法 + Hook 组合起来，形成一个强大的语法糖，让你编写html标签和JS更加简便而已。
语法糖的对象就是：原生html标签 + JS对象(JS中定义的变量和函数)。  

这就解释了为啥自定义组件 return 出去的内容，最外层必须有一个原生html标签。 说白了，无论你怎么定义，折腾这个自定义组件，本质上都要保证这个自定义组件最终都能转换成一段原生html代码。  

上面这一大段话都是“简单到不能再简单的道理”，但是你只有理解透这一层，理解自定义组件、理解react究竟是什么之后，你会对于学习React各种API和Hook才会更加容易理解和接受。  

让我们回到 那如何“勾住”自定义组件中的“小写开头的类似原生标签的组件”？ 这个问题上来。

##### React.forwardRef() 的具体用法 

React.forwardRef()包裹住要输出的组件，且将第2个参数设置为 ref 即可，示例代码：  


    import React from 'react'
    
    const ChildComponent = React.forwardRef((props,ref) => {
      //子组件通过将第2个参数ref 添加到内部真正的“小写开头的类似原生标签的组件”中 
      return <button ref={ref}>{props.label}</button>
    });
    
    /* 上面的子组件直接在父组件内定义了，如果子组件是单独的.js文件，则可以通过
       export default React.forwardRef(ChildComponent) 这种形式  */
    
    function Forward() {
      const ref = React.useRef();//父组件定义一个ref
      const clickHandle = () =>{
        console.log(ref.current);//父组件获得渲染后子组件中对应的DOM节点引用
      }
      return (
        <div>
            {/* 父组件通过给子组件添加属性 ref={ref} 将ref作为参数传递给子组件 */}
            <ChildComponent label='child bt' ref={ref} />
            <button onClick={clickHandle} >get child bt ref</button>
        </div>
      )
    }
    export default Forward;

---

至此，关于useRef()基础用法、React.forwardRef()已经讲完。 这2个函数的掌握，会对下一个要讲的Hook：useImperativeHandle 非常有用。  

欢迎进入下一章节：[useImperativeHandle基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/13%20useImperativeHandle%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)
