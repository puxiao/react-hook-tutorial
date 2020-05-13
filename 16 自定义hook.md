# 16 自定义hook

## 自定义hook概念解释
像useState、useEffect、useContext、useReducer、useCallback、useMemo、useRef、useImperativeHandle、useLayoutEffect、useDebugValue这10个hook是react默认自带的hook，而所谓自定义hook就是由我们自己编写的hook。  

所谓自定义hook就是把原来写在函数组件内的hook相关代码抽离出来，单独定义成一个函数，而这个抽离出来的hook函数就称之为“自定义hook钩子函数”，简称“自定义hook”。


## 自定义hook是来解决什么问题的？
答：自定义hook是将原来在组件中编写的相关hook代码抽离出组件，让hook相关代码独立存在，达到优化代码结构、相关hook代码可以重复使用的目的。  

补充说明：  
1、如果你在别人的项目代码中，发现除了react默认自带的那10个hook以外，出现了 useXxx() 这样的看着像hook的函数，可以肯定那些就是自定义的hook。   
2、随着react新版本发布，可能会出现更多新的、默认自带的hook。  


## 自定义hook基本用法

首先我们知道hook只能用在函数组件中，而函数组件本身是一个稍微特殊的函数，尽管稍微特殊但毕竟他也遵循一般函数的使用规律。 所谓“把原来写在函数组件内的hook相关代码抽离出来，单独定义成一个函数” 本质上就是把函数内部定义的变量或方法拿出来，放到函数外面单独定义成一个函数。   

这个抽离出来新定义的函数，遵循JS默认的函数用法，即函数参数可以任意设定，返回值也可以是任意内容。  

请注意：react规定所有的自定义hook函数命名时必须使用 useXxx 这种形式。

举一个最简单的例子：假设我们有一个组件，组件内部有一个count的变量，我们的代码之前是这样的：  

    import React,{useState} from 'react'
    function CurrentComponent() {
      const [count,setCount] = useState(0);//请注意这行代码，就是我们要即将抽离出去的hook
      return (
        <div>
            {count}
            <button onClick={() => setCount(count+1)}>add +1</button>
        </div>
      )
    }
    export default CurrentComponent

在上面这个组件中，通过 const [count,setCount] = useState(0) 定义了组件内的变量count和修改count的方法。那我们现在将这行相关的hook抽离出函数组件。我们计划把抽离出来的、和count相关的函数，命名为useCount，修改后的代码如下：  

    //useState.js
    import {useState} from 'react'；
    function useCount(initialValue){

      //依然使用 useState 创建countcount和setCount
      //并且将参数initialValue的值赋予给count作为默认值
      //将创建好的count和setCount作为函数返回值 return 出去

      const [count,setCount] = useState(initialValue);
      return [count,setCount];
    }
    export default useCount;

    //CurrentComponent.js
    import React from 'react'
    import useCount from './useCount';//引入useCount
    function CurrentComponent() {
      const [count,setCount] = useCount(0);//请注意这里使用的是useCount，而不是useState
      return (
        <div>
            {count}
            <button onClick={() => setCount(count+1)}>add +1</button>
        </div>
      )
    }
    export default CurrentComponent


##### 代码分析：  

1、我们将原本组件内定义的count相关代码抽离出组件，单独定义一个 useCount 函数。  
2、组件需要使用 useCount，只需先引入useCount，然后把 useCount 当成普通函数使用就好了。  
3、useCount就是我们自定义的hook。   

注意：一般自定义hook顶部是不需要引入React的，只需要引入使用到的 hook 函数即可。  
例如上面 useCount 顶部，我们写的是 import {useState} from 'react' 而不是 import React,{useState} from 'react'；  

上面举例中的useCount非常简单，内部并没有过多逻辑，在实际开发中自定义hook内部肯定要有比较复杂的逻辑。  

由于是单独定义的，所以自定义hook可以同时被多个组件引入和使用，达到代码复用的目的。

划重点，在实际项目中，通常自定义hook返回值有3种表现形式：  
1、不带返回值的函数  
2、带普通返回值的函数  
3、带特殊结构返回值的函数

以上3种不同返回值各有各的适用场景，下面就以实际示例来逐一说明。  


## 不带返回值的自定义hook使用示例：  

举例：若父组件内有多个子组件，每个子组件内部都有不同的业务代码，但是所有子组件有一个相同的功能，就是当自身内部变量value发生变化时，将网页标题改为变量value的值。

首先我们知道修改网页标题是在组件内部的useEffect()函数中修改，那结合上面的使用场景，我们可以将useEffect()单独抽离出来，作为一个自定义hook，命名为 useDocumentTitle，让所有子组件都复用这个useDocumentTitle。    

useDocumentTitle 代码如下：

    import {useEffect} from 'react'
    function useDocumentTitle(value) {
      useEffect(() => {
        document.title = value;
      },[value]);
    }
    export default useDocumentTitle;


假设我们其中一个子组件的功能为：  
1、有1个number类型的变量count  
2、有1个按钮，点击按钮后将count修改为一个随机数字  
3、当组件重新渲染完成后，将网页标题修改为count的值  

子组件代码为：  

    import React,{useState} from 'react'
    import useDocumentTitle from './useDocumentTitle';
    function ChildComponent() {
      const [count,setCount] = useState(0);
      useDocumentTitle(count);//把内部变量count传给useDocumentTitle，既作为网页标题内容，同时也作为useEffect的变量依赖
      return (
        <div>
            <button onClick={() => setCount(Math.floor(Math.random()*1000))}>click me</button>
        </div>
      )
    }
    export default ChildComponent


其他子组件也使用useDocumentTitle，这样我们便将原本每个子组件都需要编写的useEffect改为统一的useDocumentTitle，实现了代码复用。  

应用场景小总结：  
在这个示例中，useDocumentTitle函数并没有任何返回值，子组件使用useDocumentTitle时就好像原本那段useEffect代码本身就定义在那里似的。  


## 带普通返回值的自定义hook使用示例：  
在本章最开始讲解“自定义hook基本用法”时，所举的useCount例子非常简单，这次我们将对useCount进行功能上的扩展。  

原本useCount只是定义了count和setCount，这次所谓的功能扩展，就是将setCount改为其他几种修改count的函数。  
例如：  
1、添加 add()  
2、减去 sub()  
3、相乘 mul()  
4、恢复初始值 reset()  

修改后的useCount代码为：  

    import {useState} from 'react'
    function useCount(initialValue){
      const [count,setCount] = useState(initialValue);
      const add = param => {setCount(prev => prev + param);}
      const sub = param => {setCount(prev => prev - param);}
      const mul = param => {setCount(prev => prev * param);}
      const reset = () => {setCount(() => initialValue);}
      return [count,add,sub,mul,reset]; //将count和定义的4个方法作为返回值 return 出去
    }
    export default useCount;

请注意：为了避免4个修改函数中得到的是旧的count，所以我们采用的是 setCount(prev => xxxxx) 这种修改方式，而不是直接使用 setCount(count xxx)。  

CurrentComponent组件想使用useCount，代码为：  

    import React from 'react'
    import useCount from './useCount'

    function CurrentComponent() {
      const [count,add,sub,mul,reset] = useCount(0); //使用useCount，并解构useCount的返回值
      return (
        <div>
            {count}
            <button onClick={() => {add(1)}}>+ 1</button>
            <button onClick={() => {sub(1)}}>- 1</button>
            <button onClick={() => {mul(2)}}>* 2</button>
            <button onClick={() => {reset()}}>reset</button>
        </div>
      )
    }
    export default CurrentComponent



对于上面这个效果，你是否觉得眼熟？  没错，在讲解useReducer时，就使用useReducer实现了类似的一个效果。  只不过这次是为了讲解自定义hook，而那次是讲解如何使用useReducer替代useState实现复杂的业务。  

应用场景小总结：  
1、我们可以在自定义hook中编写相关业务逻辑函数(方法)，并通过返回值的形式 return 出去，供其他组件调用。  


## 带特殊结构返回值的自定义hook使用示例： 
上一个代码示例讲解了“带普通返回值”的自定义hook，那这次要讲的“带特殊结构返回值”的自定义hook究竟差别在哪里？  

“带特殊结构返回值”中的特殊是指？  
答：我们把组件需要用到的多项属性设置，合并为一个对象 并 return 出去，供组件使用。  

还是以示例来讲解会更容易理解，假设我们有一个登录组件，功能为：  
1、有一个用户名输入框  
2、有一个密码输入框  
3、有一个提交按钮  

补充说明，为了简化代码，我们并不做真正的登录验证，点击提交按钮后：  
1、仅仅是alert一下用户名和密码，即表示登录  
2、同时清除用户名和密码输入框里的内容  

需求分析：  
1、每个输入框都是一个<input\>，都需要绑定一个变量，都需要设置onChange事件  
2、每一个输入框都需要清空内容  

我们将定义一个自定义hook，命名为useInput，useInput来实现这2个输入框共有的业务逻辑。 

useInput的代码为：  

    import {useState} from 'react'
    function useInput(initialValue) {
      const [value,setValue] = useState(initialValue); //定义输入框对应的值value
      //定义reset函数，用来重置输入框
      const reset = () => {
        setValue(initialValue);
      }
      //定义一个 bind 对象，该对象有 value 和 onChange 2个属性
      const bind = {
        value,
        onChange: eve => {
            setValue(eve.target.value)
        }
      }
      return [value,reset,bind];//将输入框的值、重置输入框函数、定义的bind对象作为返回值 return 出去
    }
    export default useInput

请注意：在useInput中，返回值 value、reset 我们很容易理解，但是 bind 是来做什么的？  
答：这个 bind 就是我们前面提到的“带特殊结构返回值”，bind对象本身结构由2个属性value和onChange组成。   
至于 bind 怎么用，很快揭晓。  


登录组件LoginForm的代码为：  

    import React from 'react'
    import useInput from './useInput';
    function LoginForm() {
      const [usename,resetUsename,bindUsename] = useInput(''); //定义用户名输入框相关的变量
      const [password,resetPassword,bindPassword] = useInput(''); //定义密码输入框相关的变量

      const submitHandle = (eve) => {
        eve.preventDefault(); //阻止form真正提交
        alert(`usename:${usename}\rpassword:${password}`); //通过alert，弹出用户名和密码的值
        resetUsename(); //重置用户名输入框
        resetPassword(); //重置密码输入框
      }

      //请特别留意用户名和密码输入框中的 {...bindUsename}和{...bindPassword}
      return (
        <form onSubmit={submitHandle}>
            <label>usename:</label>
            <input type='text' {...bindUsename} />
            <label>password:</label>
            <input type='password' {...bindPassword} />
            <input type='submit' value='login' />
        </form>
      )
    }
    export default LoginForm;

对于获取输入框的值、以及调用输入框对应的reset()函数，相信你很容易理解。  

下面对 {...bindUsename} 和 {...bindPassword} 做进一步说明：  
1、首先我们知道 {...obj} 这种在原生JS中，相当于把obj对象进行解构，然后得到一个浅拷贝的新对象。  
2、但是在上面的代码中并不是这个意思，千万不要被迷惑。 在JSX中的某组件，如果要添加某属性，格式为 xxx={xxx}。  

例如常见的给一个输入框绑定某变量，同时添加onChange事件，一般写法为：

    <input type='text' value={xx} onChange={xxxx} />

而我们本次代码中，采用的是：  

    <input type='text' {...bindUsename} />
    <input type='password' {...bindPassword} />

这里面的 {...bindUsename}  {...bindPassword} 其实相当于把 bindUsename 和 bindPassword 进行了解构，就好像直接写在这里似的。  

如果<input\>中有非常多相同的属性，那么把这些相同属性提炼到 useInput 的 bind 中，这样可以简化组件里的代码。  

应用场景小总结：  
1、在自定义hook中，将组件需要的多项属性合并成一个对象，供组件属性解构使用，会简化组件代码，提高代码复用率。  

相信通过上面3个示例，对自定义hook的返回值不同形式的演示，举一反三，会帮助你灵活的编写自定义hook。  

---

至此，关于自定义hook已经讲完。

我们对之前所有学过的hook进行一次小总结。  

欢迎进入下一章节：[React Hook 总结](https://github.com/puxiao/react-hook-tutorial/blob/master/17%20React%20Hook%20%E6%80%BB%E7%BB%93.md)
