# 03 useState高级用法

所谓高级用法，只不过是一些深层知识点和实用技巧，你甚至可以把本章当做对前面知识点的一个巩固和学习。  

## 恢复默认值

组件需求：实现一个计数器，有3个按钮，点击后分别实现：恢复默认值、点击+1、点击-1  

实现代码：

    import React, { useState } from 'react';
    
    function Component() {
      const initCount = 0;
      const [count, setCount] = useState(initCount);
    
      return <div>
        {count}
        <button onClick={() => {setCount(initCount)}}>init</button>
        <button onClick={() => {setCount(count+1)}}>+1</button>
        <button onClick={() => {setCount(count-1)}}>-1</button>
      </div>
    }
    
    export default Component;

代码分析：  
1、通过额外定义一个变量initCount=0，作为count的默认值；  
2、任何时候想恢复默认值，直接将initCount赋值给count；

## 解决数据异步

还是基于上面那个示例，假设现在新增1个按钮，点击该按钮后执行以下代码：  

    for(let i=0; i<3; i++){
      setCount(count+1);
    }

通过for循环，执行了3次setCount(count+1)，那么你觉得count会 +3 吗？  
答案是：肯定不会

无论for循环执行几次，最终实际结果都将是仅仅执行一次 +1。  

为什么？  
类组件中setState赋值过程是异步的，同样在Hook中 setXxx 赋值也是异步的，比如上述代码中的setCount。
  
虽然执行了3次setCount(count+1)，可是每一次修改后的count并不是立即生效的。当第2次和第3次执行时获取到count的值和第1次获取到的count值是一样的，所以最终其实相当于仅执行了1次。

##### 解决办法：

你肯定第一时间想到的是这样解决方式：  

    let num = count;
    for(let i=0; i<3; i++){
      num +=1;
    }
    setCount(num);

这样做肯定没问题，只不过有更简便、性能更高的方式。  

和类组件中解决异步的办法类似，就是不直接赋值，而是采用“箭头函数返回值的形式”赋值。 

把代码修改为：  

    for(let i=0; i<3; i++){
      setCount(prevData => {return prevData+1});
      //可以简化为 setCount(prevData => prevData+1);
    }

代码分析：  
1、prevData为我们定义的一个形参，指当前count应该的值；  
2、{return prevData+1} 中，将 prevData+1，并将运算结果return出去。当然也非常推荐使用更加简化的写法：setCount(prevData => prevData+1)；  
3、最终将prevData赋值给count；

补充说明：你可以将prevData修改成任意你喜欢的变量名称，比如prev，只需要确保和后面return里的一致即可。  


## 数据类型为Objcet的修改方法

之前的示例中，每个useState对应的值都是简单的string或number，如果对应的值是object，又该如何处理呢？  

例如：  

    const [person, setPerson] = useState({name:'puxiao',age:34});

若想将age的值修改为18，该怎么写？

如果你有类组件编程经验，你肯定第一时间想是这样的：

    setPerson({age:18});

在类组件中，setState是执行的是“异步对比累加赋值”，何为“对比”？  就是先对比之前数据属性中是否有age，如果有则修改age值，同时不会影响到其他属性的值。我猜测react是使用ES6中新增加的Object.assign()这个函数来实现这一步的。  

但是，用useState定义的修改函数 setXxxx，例如setPerson中，执行的是 “异步直接赋值”。

请看实际执行的结果：  

    console.log(person);//{name:'puxiao',age:34}
    setPerson({age:18});
    console.log(person);//{age:18}

没错，虽然只是希望修改age的值，但是由于是“直接赋值”，导致{age:18}替换了整个{name:'puxiao',age:34}  


##### 正确的做法：  

我们需要先将person拷贝一份，修改之后再进行赋值。

    let newData = {...person};
    newData.age = 18;
    setPerson(newData);

以上代码还有一种简写形式：  

    setPerson({...person,age:18}); //这种简写是解构赋值带来的，并不是React提供的

代码分析：  
1、先通过...person，将原有person做一次解构，得到一份复制品(浅拷贝)；  
2、修改age的值；  
3、将修改过后的新数据，通过setPerson赋值给person；  

完整示例：  

    import React, { useState } from 'react';

    function Component() {

      const [person, setPerson] = useState({name:'puxiao',age:34});

      const nameChangeHandler = (eve) => {
        setPerson({...person,name:eve.target.value});
      }

      const ageChangeHandler = (eve) => {
        setPerson({...person,age:eve.target.value});
      }

      return <div>
        <input type='text' value={person.name} onChange={nameChangeHandler} />
        <input type='number' value={person.age} onChange={ageChangeHandler} />
        {JSON.stringify(person)}
      </div>
    }
    export default Component;


## 数据类型为Array的修改方法

和数据类型为Object相似，都是需要通过先拷贝一次，修改后再整体赋值。  

这里举一个简单的小例子，以下代码实现了一个类似学习计划列表的功能组件。  

    import React, { useState } from 'react';

    function Component() {

      const [str, setStr] = useState('');
      const [arr, setArr] = useState(['react', 'Koa']);

      const inputChangeHandler = (eve) => {
        setStr(eve.target.value);
      }

      const addHeadHandler = (eve) => {
        setArr([str,...arr]);//添加至头
        setStr('');
      }

      const addEndHandler = (eve) => {
        setArr([...arr, str]);//添加至尾
        setStr('');
      }

      const delHeadHandler = (eve) => {
        let new_arr = [...arr];
        new_arr.shift();//从头删除1项目
        setArr(new_arr);
      }

      const delEndHandler = (eve) => {
        let new_arr = [...arr];
        new_arr.pop();//从尾删除1项目
        setArr(new_arr);
      }

      const delByIndex = (eve) => {
        let index = eve.target.attributes.index.value;
        let new_arr = [...arr];
        new_arr.splice(index,1);//删除当前项
        setArr(new_arr);
      }

      return <div>
        <input type='text' value={str} onChange={inputChangeHandler} />
        <button onClick={addHeadHandler} >添加至头</button>
        <button onClick={addEndHandler} >添加至尾</button>
        <button onClick={delHeadHandler} >从头删除1项</button>
        <button onClick={delEndHandler} >从尾删除1项</button>
        <ul>
            {arr.map(
                (item, index) => {
                    return <li key={`item${index}`}>学习{index} -  {item}
                        <span index={index} onClick={delByIndex} style={{ cursor: 'pointer' }}>删除</span>
                    </li>
                }
            )}
        </ul>
      </div>
    }

    export default Component;


## 性能优化

通过 setXxx 设置新值，但是如果新值和当前值完全一样，那么会引发React重新渲染吗？

通过React官方文档可以知道，当使用 setXxx 赋值时，Hook会使用Object.is()来对比当前值和新值，结果为true则不渲染，结果为false就会重新渲染。

    let str='a';
    Object.is(str,'a'); //true

    let str='18';
    Object.is(str,18); //str为String类型，18为Number类型，因此结果为false
    
    let obj={name:'a'};
    Object.is(obj,{name:'a'}); //false
    //虽然obj和{name:'a'}字面上相同，但是obj==={name:'a'}为false，并且在Object.is()运算下认为两者不是同一个对象
    //事实上他们确实不是同一个对象，他们各自占用了一份内存

    let obj={name:'a'};
    let a=obj;
    let b=obj;
    Object.is(a,b); //因为a和b都指向obj，因此结果为true


由上面测试可以看出：  
1、对于简单类型的值，例如String、Number 新旧值一样的情况下是不会引起重新渲染的；  
2、对于复杂类型的值，即使新旧值 “看上去是一样的” 也会引起重新渲染。除非新旧值指向同一个对象，或者可以说成新旧值分别是同一个对象的引用；  

采用复杂类型的值不是不可以用，很多场景下都需要用到，但是请记得上面的测试结果。

为了可能存在的性能问题，如果可以，最好避免使用复杂类型的值。 


## 自定义Hook

所谓自定义Hook，就是将Hook函数从函数组件中抽离，抽离之后多个函数组件可以共用该自定义Hook，共享该Hook的逻辑。 

因为目前仅学习了useState，再多学习几个Hook函数后，会单独拿出一个篇章来讲解如何自定义Hook。 


---

至此，关于useState高级用法已经讲完，相信你已经掌握了useState的使用方法。  

useState是本系列文章讲解的第一个Hook函数，同时也是使用频率最高的Hook，甚至可以说useState是函数组件开发的基石，因此本章稍显啰嗦，但目的就是希望你能理解透彻。  

在后面讲解其他Hook函数时，将会尽量使用简洁、高冷的文章风格。  

接下来学习第2个Hook函数useEffect。

欢迎进入下一章节：[useEffect基本用法](https://github.com/puxiao/react-hook-tutorial/blob/master/04%20useEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
