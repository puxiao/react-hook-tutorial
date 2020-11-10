# 02 useState基础用法

## useState概念解释
我们第一个要学习的Hook(钩子函数)是useState，他的作用是“勾住”函数组件中自定义的变量。  

“勾住”？  
回顾一下 “React Hook 简介” 文中那句话：Hook本身单词意思是“钩子”，作用就是“勾住某些生命周期函数或某些数据状态，并进行某些关联触发调用”。  

“如何勾住”？
在React底层代码中，是通过自定义dispatcher，采用“发布订阅模式”实现的。  

关于“钩子”、“勾住”、“如何勾住”的概念以后在学习其他Hook函数时不再做解释。  

## useState是来解决类组件什么问题的？

答：useState能够解决类组件 **所有自定义变量只能存储在this.state** 的问题。

举例：若某组件需要有2个自定义变量name和age，那么在类组件中只能如下定义

    constructor(props) {
        super(props);
        this.state = {
          name:'puxiao',
          age:34
        }
    }

name和age只能作为this.state的一个属性。

没有对比就没有伤害，看一下使用useState后，函数组件是如何实现上述需求的

    const [name,setName] = useState('puxiao');
    const [age,setAge] = useState(34);

1、函数组件本身是一个函数，不是类，因此没有构造函数constructor(props)；  
2、任何你想定义的变量都可以单独拆分出去，独立定义，互不影响；

两段代码对比之下，你就会发现使用Hook的useState后，会让我们定义的变量相对独立，清晰简单，便于管理。  

接下来开始学习useState。   

## useState函数源码：

首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useState<S>(
      initialState: (() => S) | S,
    ): [S, Dispatch<BasicStateAction<S>>] {
      const dispatcher = resolveDispatcher();
      return dispatcher.useState(initialState);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。之所以贴出源码只是为了显得本文比较有深度。^_^  

**更新于2020.11.10，这里强调一下：React 源码中使用的是 flow 语法，根本不是 TypeScript 语法，只不过 2 者实在是太像了，以至于让我之前一直误以为 React 源码中是 TS。不过你完全可以将 TS 的泛型知识去套用到 flow 中。在此特别说明一下，至于后续章节中就不再做提醒和修改了，你就当成 TS 语法去理解也行。**

##### 补充一些TypeScript常识：  
1、react 本身采用TypeScript编写，还是补充点TS常识，方便对各个 hook 函数源码的理解。  
2、对于useState以及以后要学习的其他hook函数源码，函数参数中会反复出现<S\>、<T\>、<P\>、<I\>、<I\>，这些大写字母，react约定他们对应的单词如下：  
state -> S -> 约定表示某种“数据”  
type -> T -> 约定表示某种“类型”  
props -> P -> 约定表示“属性传值对应的props”  
initial -> I -> 约定表示某个“初始值”  
    
1、这种用<X\>包裹起来的类型声明，在TS中成为“泛型”。理论上是可以使用任意单词的，上面那些缩写只是react自己约定单词缩写。  
2、对于一段TS代码，如果出现了<S\>，那么后面所有的<S\>都将表示“某种相同类型的数据”。对于TypeScript的泛型相关知识，请自己百度学习。  


## useState基本用法

useState(value)函数会返回一个数组，该数组包含2个元素：第1个元素为我们定义的变量，第2个元素为修改该变量对应的函数名称。  

##### 代码形式：  

    const [variable,setVariable] = useState(value);
    //....
    setVariable(newValue);//修改variable的值


##### 拆解说明：  

1、const [a,b] = [a,b] 这种形式为ES6的“解构赋值”；  
2、'variable'为函数组件中自定义的变量名；  
3、'setVariable'为修改'variable'对应的函数名；  
4、'useState'为本次学习的Hook函数；  
5、'value'为变量默认值  
6、'setVariable(newValue)'为调用setVariable并将新的值newValue赋值给variable；  

#### 'variable'补充说明
1、variable为变量名，实际使用中可以修改成任意变量名，比如name、age、count等等；  
2、但是，函数组件接收父级组件属性传值的变量名为props，因此建议你不要将变量名定为props，以免混淆；   
3、我不听话，我就非要将变量名定义成props，那又会怎么样？答案是不会有什么问题，不仅不会报错而且还会正常执行。  

##### 'setVariable'补充说明：  
1、该名称采用 "set"+"变量名" 的驼峰命名形式，只是为了提高代码可读性。  
2、一般React项目都约定使用此种命名方式，所以推荐你也如此使用。  
3、当然你也可以使用任意你喜欢的命名风格，但是切记不能以数字开头。  

##### 'value'补充说明：  
1、必填项，不可缺省，若缺省则实际运行时会提示变量名未定义；  
2、值的类型可以是字符串、数字、数组、对象；  
3、值还可以为null，但不可以为undefined；  

##### 'newValue'补充说明(非常重要)：
setVariable采用 “异步直接赋值” 的形式，并不会像类组件中的setState()那样做“异步对比累加赋值”。  

“异步”？  
这里的“异步”和类组件中setState中的异步是同一个意思，都是为了优化React渲染性能而故意为之。 

"直接赋值"？  
1、在Hook中，对于简单类型数据，比如number、string类型，可以直接通过setVariable(newValue)直接进行赋值。   
2、但对于复杂类型数据，比如array、object类型，若想修改其中某一个属性值而不影响其他属性，则需要先复制出一份，修改某属性后再整体赋值。具体如何做，请看下一篇“useState高级用法”中“数据类型为Objcet/Array修改方法”内容。 

如果新值和当前值完全一样，那么会引发React重新渲染吗？请看下一篇“useState高级用法”中“性能优化”内容。

停！上面的信息量有点多，让我们把思绪先回到最基础的用法上。  

## useState使用示例：  

    //函数组件内定义变量name
    const [name,setName] = useState('nodejs'); //name默认值为nodejs
    
    //在函数组件内，某些事件交互处理函数中修改name的值，例如某次鼠标点击的处理函数handleClick
    const handleClick = () => {
      setName('koa');
      //请注意，setName('koa')是异步修改的，如果此时执行console.log(name) 输出的值依然是nodejs
      //请留意下一篇文章 “03 useState高级用法” 中 “解决数据异步” 相关部分
    }


上述代码中，我们进行了以下操作：  
1、声明一个变量name、修改name的方法setName、并将name默认值设置为'nodejs'；  
2、通过setName将name值修改为'koa'；  

注意：在一个组件中，可以不限次数使用useState()，因此，我们可以声明多个变量，例如下面代码：  

    const [name,setName] = useState('puxiao');
    const [age,setAge] = useState(34);

在该代码片段中，我们分别定义了2个变量：name、age 以及他们对应的修改函数setName、setAge。


## 练习题
用useState实现一个计数器，默认为0，每次点击+1。  

##### 完整示例：

    import React, { useState } from 'react';
    
    function Component() {
    
      const [count, setCount] = useState(0);
    
      function clickHandler(){
        setCount(count+1);
      }
    
      return <div onClick={clickHandler}>
        {count}
      </div>
    }
    
    export default Component;

请注意上述代码中，没有用到this，这就是函数组件中使用Hook的魅力之一，再也不用去关心烦人的this到底指向谁这个问题了。  

实际代码中，本人更加倾向于使用箭头函数来定义方法，所以上述 function clickHandler() 会写成：  

    const clickHandler = () => {
      setCount(count+1);
    }

---

至此，关于useState基础用法已经讲完。

欢迎进入下一章节：[useState高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/03%20useState%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)
