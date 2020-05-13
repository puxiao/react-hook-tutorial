# 10 useCallback基础用法

## useCallback概念解释
我们第五个要学习的Hook(钩子函数)是useCallback，他的作用是“勾住”组件属性中某些处理函数，创建这些函数对应在react原型链上的索引。useCallback第2个参数是处理函数中的依赖变量，只有当依赖变量发生改变时才会重新修改并创建新的一份处理函数。

##### react原型链？  
我对react原型链也不太懂，你可以简单得把 react原型链 理解成 “react定义的一块固定内存”。我们使用各种 hook 定义的“变量或函数”都存放在这块内存里，当我们需要使用时可以“对象的引用名”从该内存里获取，例如useCallback或者之前学习的useContext、useReducer等相关变量或函数。  
在本章或以后的章节中，我依然会使用 react原型链 这个词，你都按照我刚才说的“react定义的一块固定内存”概念去理解就好了。  

懵圈了没？我已经尽量总结得让你容易理解了，如果你似懂非懂没有关系，本文后面会通过示例代码让你明白。

让我们先忘掉useCallback，先来学习一下以下2个知识点。

##### 第1个知识点：React.memo() 的用法
首先我们知道，默认情况下如果父组件重新渲染，那么该父组件下的所有子组件都会随着父级的重新渲染而重新渲染。  
1、无论子组件是类组件或是函数组件。  
2、无论子组件在本次渲染过程中，子组件是否有任何相关的数据变化。  

举例，假设某父组件中有3个子组件：子组件A、子组件B、子组件C。若因为子组件A发生了某些操作，引发父组件重新渲染，这时即使子组件B和子组件C没有任何需要更改的地方，但是默认他们两个也会重新被渲染一次。  

为了减少这个不必要的重新渲染，如果是类组件，可以在组件shouldComponentUpdate(准备要开始更新前)生命周期函数中，通过比较前后两次props和state中相关的值，如果完全相等则跳过本次渲染，改为直接使用上一次渲染结果，以此提高性能提升。  

伪代码如下：  

    shouldComponentUpdate(nextProps,nextStates){
      //判断xxx值是否相同，如果相同则不进行重新渲染
      return (nextProps.xxx !== this.props.xxx); //注意是 !== 而不是 !=
    }

为了简化我们这一步操作，可以将类组件由默认继承自React.Component改为React.PureComponent。React.PureComponent默认会帮我们完成上面的浅层对比，以跳过本次重新渲染。 

请注意：React.PureComponent会对props上所有可枚举属性做一遍浅层对比。而不像 shouldComponentUpdate中可以有针对性的只对某属性做对比。  

上面将的都是类组件，与之对应的是React.memo()，这个是针对函数组件的，作用和React.PureComponent完全相同。  

React.memo()的使用方法很简单，就是把要导出的函数组件包裹在React.memo中即可。  

伪代码如下：  

    import React from 'react'
    function Xxxx() {
    return <div>xx</div>;
    }
    export default React.memo(Xxxx); //使用React.memo包裹住要导出的函数组件

请记住以下2点：
1、React.memo()只会帮我们做浅层对比，例如props.name='puxiao'或props.list=[1,2,3]，如果是props中包含复杂的数据结构，例如props.obj.list=[{age:34}]，那么有可能达不到你的预期，无法做到深层次对比。  
2、使用React.memo仅仅是让该函数组件具备了可以跳过本次渲染的基础，若组件在使用的时候属性值中有某些处理函数，那么还需要配合useCallback才可以做到跳过本次重新渲染。  

呵，话题又回到useCallback上面了。


##### 第2个知识点：=== 等比运算

在原生JS中，你认为  
1、{}==={} 为true还是false？  
2、{a:2}==={a:2} 为true还是false？  

这是一道很简单却很容易迷惑人的题目，若你对原生JS中 === 等比运算不够深入了解，你很容易会认为结果是true。  

如果你轻松回答出来：以上均为false，那么恭喜你是个明白人。  
如果你疑惑了一下或者你的答案是true，那么你可以自己去JS里测试一下看结果是什么。  

答案是2者均是false。  

以{}==={}为例，虽然从字面上 === 左右两侧完全相同的，但是实际上在JS中 左右两侧分别为独立的{}对象，各自占有各自的内存空间，因此他们对比的结果是false。

相反，看下面的代码：  

    let obj = {};
    let obj2 = obj;
    obj2.name='react';
    console.log(obj===obj2); //true

上面输出结果为true，为何obj===obj2为true？  因为 obj和obj2都是对同一个对象的索引(引用)，所以对比结果为true，因为他们最终指向同一个对象。  

还记得本文开头对于useCallback概念解释中的那段文字吗？useCallback的作用是“勾住”组件属性中某些处理函数，创建这些函数对应在react原型链上的索引。  

呵，话题又回到useCallback上面了。

划重点：记住“useCallback”和“原型链上处理函数的索引”这两个关键词，基本上你就对useCallback的原理理解一大半了。

让我们回到useCallback基础学习中。


## useCallback是来解决什么问题的？
答：useCallback的目的是“阻止组件不必要的重新渲染”。  
useCallback可以将组件的某些处理函数挂载到react底层原型链上，并返回该处理函数的索引，当组件每次即将要重新渲染时，确保props中该处理函数为同一函数(因为是索引，所以===运算结果一定为true)，跳过本次无意义的重新渲染，达到提高组件性能的目的。当然前提是该组件在导出时使用了React.memo()。    

补充说明：  
假设某组件使用到了myfun这个处理函数，回忆一下前面提到的JS中===运算规则，考虑一下。  

默认不使用useCallback，其实组件执行了以下伪代码：  

    let obj = {}; //上一次渲染时创建的props
    obj.myfun={xxx}; //props中的myfun属性值，实为独立创建的{xxx}
    
    let obj2 = {}; //本次渲染时创建的props
    obj2.myfun={xxx}; //props中的myfun属性值，实为独立创建的{xxx}
    
    if(obj.myfun === obj2.myfun){
      //跳过本次重新渲染，改为使用上一次渲染结果即可
    }
    
由于obj.myfun 和 obj2.myfun 为分别独立创建的函数{xxx}，所以对比结果为false，也就意味着无法跳过本次重新渲染，尽管函数{xxx}字面相同。  


相反，如果使用useCallback，其实组件执行了以下伪代码：  

    let myfun = {xxx}; //独立定义处理函数myfun
    
    let obj = {}; //上一次渲染时创建的props
    obj.myfun = myfun; //props中的myfun属性值，实为myfun的引用
    
    let obj2 = {}; //本次渲染时创建的props
    obj2.myfun = myfun; //props中的myfun属性值，实为myfun的引用

    if(obj.myfun === obj2.myfun){
      //跳过本次重新渲染，改为使用上一次渲染结果即可
    }

此时 obj.myfun 和 obj2.myfun 均为myfun的引用，因此该对比结果为true，也就意味着可以顺利跳过本次渲染，达到提高组件性能的目的。  

以上是代码仅仅是为了示意默认子组件为什么会被迫重新渲染，以及useCallback作用机理。   

只有理解了这个机理，才会明白何时使用useCallback。  不要滥用useCallback。

多说一句：你是否觉得React Hook 很绕？ 对，这就是Hook学习起来难度大的一些原因，但当你充分理解React的编程哲学思想后，用起来会如鱼得水。加油！  


## useCallback函数源码：  
回到useCallbackt的学习中，首先看一下React源码中的ReactHooks.js。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useCallback<T>(
      callback: T,
      deps: Array<mixed> | void | null,
    ): T {
      const dispatcher = resolveDispatcher();
      return dispatcher.useCallback(callback, deps);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^  
不过请注意第2个参数，deps为该函数依赖的数据变量，值为Array<mixed> 或 void 或 null。 意味着如果该函数没有依赖的情况下，可以传入空数组[]或void或null。个人建议是传入空数组。  

补充一点TypeScript知识：像 <T\>(callback:T):T 这种类型定义称为“泛型”，里面 T 的含义为“一模一样的同类型”。  
举例：  
1、若T为function，即参数callback类型为function，那么函数返回值也为function。  
2、若T为object，即参数callback类型为object，那么函数返回值也为object。  


## useCallback基本用法

useCallback(callback,deps)函数通常传入2个值，第1个值为我们定义的一个“处理函数”，通常为一个箭头函数。第2个值为该处理函数中存在的依赖变量，请注意凡是处理函数中有的数据变量都需要放入deps中。如果处理函数没有任何依赖变量，可以传入一个空数组[]。  

特别强调一下：useCallback中的第2个依赖变量数组和useEffect中第2个依赖变量数组，作用完全不相同。  
useEffect中第2个依赖变量数组是真正起作用的，是具有关键性质的。而useCallback中第2个依赖变量数组目前作用来说仅仅是起到一个辅助作用。  

仅仅是辅助？辅助什么了？甚至你还可能会有一个疑问，既然处理函数中所有的依赖变量都需要做为第2个参数的内容，为啥React不智能一些，让我们不传第2个参数，省略掉这一步？  

在React官方文档中，针对第2个参数有以下这段话：

> 注意：依赖项数组不会作为参数传给回调函数。虽然从概念上来说它表现为：所有回调函数中引用的值都应该出现在依赖项数组中。未来编译器会更加智能，届时自动创建数组将成为可能。  

自己体会吧，你品，你细品。
 

##### 代码形式：  

    import Button from './button'; //引入我们自定义的一个组件<Button>

    //组件内部声明一个age变量
    const [age,setAge] = useState(34);

    //通过useCallback，将鼠标点击处理函数保存到React底层原型链中，并获取该函数的引用，将引用赋值给clickHandler
    const clickHandler = useCallback(() => {
        setAge(age+1);
      },[age]);
    //由于该处理函数中使用到了age这个变量，因此useCallback的第2个参数中，需要将age添加进去

    //使用该处理函数，实为使用该处理函数的在React底层原型链上的引用
    return <Button clickHandler={clickHandler}></Button>


##### 拆解说明：  

1、具体讲解已在上面示例代码中做了多项注释，此处不再重复；  


#### 'age'补充说明
1、上述代码示例中，age为该组件通过useState创建的内部变量，事实上也可以是父组件通过属性传值的props.xx中的变量。  
2、只要依赖变量不发生变化，那么重新渲染时就可以一直使用之前创建的那个函数，达到阻止本次渲染，提升性能的目的。但是如果依赖变量发生变化，那么下次重新渲染时根据变量重新创建一份处理函数并替换React底层原型链上原有的处理函数。  

#### 'clickHandler'补充说明
再次强调，clickHandler实际上是真正的处理函数在React底层原型链上的引用。   

#### '<Button\>'补充说明
<Button\>为我们自定义的一个组件，在上述代码中相当于“子组件”。  

事实上，上面的示例伪代码其实是错误的，仅仅是为了演示useCallback的使用方法，实际组件运用中，如果父组件中只有1个子组件，那其实完全没有必要使用useCallback。只有父组件同时有多个子组件时，才有必要去做性能优化，防止某一个子组件引发的重新渲染也导致其他子组件跟着重新渲染。  

## useCallback使用示例：  

若我们有一个自定组件<Button\>，代码如下：  

    import React from 'react'
    function Button({label,clickHandler}) {
        //为了方便我们查看该子组件是否被重新渲染，这里增加一行console.log代码
        console.log(`rendering ... ${label}`);
        return <button onClick={clickHandler}>{label}</button>;
    }
    export default React.memo(Button); //使用React.memo()包裹住要导出的组件


现在，我们要实现一个组件，功能如下：  
1、组件内部有2个变量age，salary  
2、有2个自定义组件Button，点击之后分别可以修改age，salary值  

若我们不使用useCallback，代码示例如下：

    import React,{useState,useCallback,useEffect} from 'react';
    import Button from './button';

    function Mybutton() {
      const [age,setAge] = useState(34);
      const [salary,setSalary] = useState(7000);

      useEffect(() => {
        document.title = `Hooks - ${Math.floor(Math.random()*100)}`;
      });

      const clickHandler01 = () => {
        setAge(age+1);
      };

      const clickHandler02 = () => {
        setSalary(salary+1);
      };

      return (
        <div>
            {age} - {salary}
            <Button label='Bt01' clickHandler={clickHandler01}></Button>
            <Button label='Bt02' clickHandler={clickHandler02}></Button>
        </div>
      )
    }

实际运行中你会发现，无论点击哪个按钮，都会收到：   
rendering ... Bt01  
rendering ... Bt02  

你只是点击操作了其中一个按钮，另外一个按钮也要跟着重新渲染一次，试想一下如果该组件中有100个子组件都要跟着重新渲染，那真的是性能浪费。  

我们再看一下如果使用useCallback，代码示例如下：  

    import React,{useState,useCallback,useEffect} from 'react';
    import Button from './button';

    function Mybutton() {
      const [age,setAge] = useState(34);
      const [salary,setSalary] = useState(7000);

      useEffect(() => {
        document.title = `Hooks - ${Math.floor(Math.random()*100)}`;
      });

      //使用useCallback()包裹住原来的处理函数
      const clickHandler01 = useCallback(() => {
        setAge(age+1);
      },[age]);

      //使用useCallback()包裹住原来的处理函数
      const clickHandler02 = useCallback(() => {
        setSalary(salary+1);
      },[salary]);

      return (
        <div>
            {age} - {salary}
            <Button label='Bt01' clickHandler={clickHandler01}></Button>
            <Button label='Bt02' clickHandler={clickHandler02}></Button>
        </div>
      )
    }

修改后的代码，实际运行就会发现，当点击某个按钮时，仅仅是当前按钮重新做了一次渲染，另外一个按钮则没有重新渲染，而是直接使用上一次渲染结果。

使用useCallback减少子组件没有必要的渲染目的达成。  

useCallback用法很简单，就是包裹住原本的处理函数。关键点在于你要理解useCallback背后的机理，才能知道在什么情况下可以使用useCallback。否则很容易滥用 useCallback，反而造成性能的浪费。  


## 思考题

假设上面示例代码中，做以下修改：每个按钮上新增一个属性：random={Math.floor(Math.random()*100)}  

    <Button label='Bt01' clickHandler={clickHandler01}></Button>
    <Button label='Bt02' clickHandler={clickHandler02}></Button>
    修改为
    <Button label='Bt01' clickHandler={clickHandler01} random={Math.floor(Math.random()*100)}></Button>
    <Button label='Bt02' clickHandler={clickHandler02} random={Math.floor(Math.random()*100)}></Button>

那么请问，此时我们针对性能优化而使用的useCallback还有意义吗？  

答：没有任何意义，虽然我们使用useCallback保证了每次clickHandler是相同的，可是 random 的值每次却是随机不一样的，尽管子组件<Button\>并没有使用到 random 这个值，但是它的加入造成了 props 每次都不一样(其实是 props.random 不一样)，结果就是子组件每一次都会被重新渲染。所以此时useCallback已经失去了存在的意义。  

---

至此，关于useCallback基础用法已经讲完，没有高级用法，直接进入下一个Hook。  

欢迎进入下一章节：[useMemo基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/11%20useMemo%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
