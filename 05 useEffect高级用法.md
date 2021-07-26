# 05 useEffect高级用法

所谓高级用法，只不过是一些深层知识点和实用技巧，你甚至可以把本章当做对前面知识点的一个巩固和学习。  

## 让useEffect只在挂载后和卸载前执行一次

让我们实现 “04 useEffect基础用法” 中 举例2 提到的功能。

组件需求：  
1、若某类组件中有变量a，默认值为0，当组件第一次被挂载后或组件重新渲染后，将网页标题显示为a的值。  
2、当组件第一次被挂载后执行一个自动累加器 setInterval，每1秒 a 的值+1。为了防止内存泄露，我们在该组件即将被卸载前清除掉该累加器。    

需求分析：  
关于自动累加器的操作，只关联 “组件挂载后和组件卸载前” 这2个生命周期函数中，那useEffect还包含了每次组件重新渲染后，这该怎么办？  

答：useEffect函数的第2个参数表示该依赖关系，**将useEffect的第2个参数，设置为空数组 []**，即表示告诉React，这个useEffect不依赖任何变量的更新所引发的组件重新渲染，以后此组件再更新也不需要调用此useEffect。

这样就可以实现只在第一次挂载后和卸载前调用此useEffect的目的。    

    import React, { useState,useEffect} from 'react';

    function Component() {
      const [a, setA] = useState(0);//定义变量a，并且默认值为0

      //定义第1个useEffect，专门用来处理自动累加器
      useEffect(() => {
        let timer = setInterval(() => {setA(a+1)},1000);// <-- 请注意这行代码，暗藏玄机
        return () => {
            clearInterval(timer);
        }
      }, []);//此处第2个参数为[]，告知React以后该组件任何更新引发的重新渲染都与此useEffect无关

      //定义第2个useEffect，专门用来处理网页标题更新
      useEffect(() => {
        document.title = `${a} - ${Math.floor(Math.random()*100)}`;
      },[a])
      return <div> {a} </div>
    }

    export default Component;

以上代码实际运行正确吗？  
答：不正确！

？  
小朋友，脸上是否有很多问号？？？  

实际运行会发现，当组件挂载后，确实会执行一次 setA(a+1)，a 的值修改为了 1，然后... a 的值一直为 1，并没有继续累加。  

上述代码会收到react的一个错误警告提示：Either include it or remove the dependency array. You can also do a functional update 'setA(a => ...)' if you only need 'a' in the 'setA' call.  
该错误警告意思是：如果你确认你传入的第2个参数是空数组，那么你可能会用到 setA(a => ...) 这种方式来更新a的值。  

问题出在哪里？  

让我们再看看那行有玄机的代码：  

    let timer = setInterval(() => {setA(a+1)},1000);  

再看看 react 给我们的错误警告提示：You can also do a functional update 'setA(a => ...)' if you only need 'a' in the 'setA' call.  你可能会用到 setA(a => ...) 这种方式来更新a的值。

setA(a => ...)  这是在 “03 useState高级用法”中，解决数据异步 时讲的更新方式。

那我们就按照提示，将那行代码修改为：  

    let timer = setInterval(() => {setA(a => a+1)},1000);  

再次执行，错误提示警告没有了，组件也完全按照我们的预期来执行了。react自带的语法检查真的好智能。    

##### 为什么会有这个问题？
关于刚才setInterval中累加 a 的值遇到的问题，React官方文档中也有类似示例，只不过他们用的变量是count，而我们这里用的变量是 a。

我们看看从React官方文档中引用的话：  
> 有时候，你的 effect 可能会使用一些频繁变化的值。你可能会忽略依赖列表中 state，但这通常会引起 Bug，传入空的依赖数组 []，意味着该 hook 只在组件挂载时运行一次，并非重新渲染时。但如此会有问题，在 setInterval 的回调中，count 的值不会发生变化。因为当 effect 执行时，我们会创建一个闭包，并将 count 的值被保存在该闭包当中，且初值为 0。每隔一秒，回调就会执行 setCount(0 + 1)，因此，count 永远不会超过 1。

再次重复一遍：如果useEffect函数第2个参数为空数组，那么react会将该useEffect的第1个参数 effect 建立一个闭包，该闭包里的变量 a 被永远设定为当初的值，即 0。尽管setInterval正常工作，每次都“正常执行了”，可是 setA(a+1)中 a 的值一直没变化，一直都是当初的0，所以造成 0 + 1 一直都等于 1 的结果。

而如果修改成 setA(a => a+1) 的形式，那么就解决了 a 数据异步的问题，每次都是读取最新当前 a 的值。

这个点是使用 useEffect 很容易掉进去的一个坑，切记切记。

或者以后养成都用异步更新数据的习惯。  


## 性能优化

通过上面的例子，我们其实已经实现了 前文中 举例2 和举例3 的效果。

咦~ 刚才讲的是举例2，没有将举例3啊... 因为举例3中提到的类组件中有多个变量数据，在函数组件中这个问题本身是靠useState来解决的，跟useEffect无关。  

接下来讲一下useEffect函数第2个参数提高性能的正确用法。

举例：若一个组件中有一个自定义变量obj，obj有两个属性a、b，当a发生变化时，网页标题也跟着a发生变化。  
补充说明：  
1、我们为了让a、b都可以发生变化，将在组件中创建2个按钮，点击之后分别可以修改a、b的值；  
2、为了更加清楚看到每次渲染，我们在网页标题中 a 的后面再增加一个随机数字；  

我们首先看以下代码：  

    import React, { useState,useEffect} from 'react';
    function Component() {
      const [obj,setObj] = useState({a:0,b:0});
      useEffect(() => {
        document.title = `${obj.a} - ${Math.floor(Math.random()*50)}`;
      }); //注意此时我们并未设置useEffect函数的第2个参数

      //如果下面代码看不懂，你需要重新去温习useState高级用法中的“数据类型为Objcet，修改方法”
      return <div>
        {JSON.stringify(obj)}
        <button onClick={() => {setObj({...obj,a:obj.a+1})}}>a+1</button> 
        <button onClick={() => {setObj({...obj,b:obj.b+1})}}>b+1</button>
      </div>
    }
    export default Component;

由于我们在网页标题中添加了随机数，因此实际运行你会发现即使修改b的值，也会引发网页标题重新“变更一次”。  

理由显而易见，修改b的值也会触发组件重新渲染，进而触发useEffect中的代码。

正确的做法应该是我们给useEffect添加上第2个参数：[obj.a]，明确告诉React，只有当obj.a变更引发的重新渲染才执行本条useEffect。

    useEffect(() => {
       document.title = `${obj.a} - ${Math.floor(Math.random()*50)}`;
     },[obj.a]); //第2个参数为数组，该数组中可以包含多个变量

添加过[obj.a]之后，再次运行，无论obj.b或者其他数据变量引发的组件重新渲染，都不会执行该useEffect。

因此达到提高性能的目的。  


---

至此，关于useEffect高级用法已经讲完，相信useState和useEffect的组合使用，已经能够让你写出一些简单的React Hook 组件。    

接下来学习第3个Hook函数useContext。

欢迎进入下一章节：[useContext基本用法](https://github.com/puxiao/react-hook-tutorial/blob/master/06%20useContext%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)
