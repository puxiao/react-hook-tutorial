# 07 useContext高级用法

所谓高级用法，只不过是一些深层知识点和实用技巧，你甚至可以把本章当做对前面知识点的一个巩固和学习。  

## 同时传递多个共享数据值给1个子组件

实现以下组件需求：  
1、有2个共享数据对象 UserContext、NewsContext；  
2、父组件为AppComponent、子组件为ChildComponent；  
3、父组件需要同时将UserContext、NewsContext的数据同时传递给子组件；  

实现代码：  

    import React,{ useContext } from 'react'

    const UserContext = React.createContext();
    const NewsContext = React.createContext();

    function AppComponent() {
      return (
        <UserContext.Provider value={{name:'puxiao'}}>
            <NewsContext.Provider value={{title:'Hello React Hook.'}}>
                <ChildComponent />
            </NewsContext.Provider>
        </UserContext.Provider>
      )
    }

    function ChildComponent(){
      const user = useContext(UserContext);
      const news = useContext(NewsContext);
      return <div>
        {user.name} - {news.title}
      </div>
    }

    export default AppComponent;

代码分析：  
1、父组件同时要实现传递2个共享数据对象value值，需要使用<XxxContext.Provider value={obj}>标签进行2次嵌套。  
2、子组件使用了useContext，他可以自由随意使用父组件传递过来的共享数据value，并不需要多次嵌套获取。  

## 同时将1个共享数据值传递给多个子组件
使用<XxxContext.Provider></XxxContext.Provider>标签将多个子组件包裹起来，即可实现。  

    <XxxContext.Provider value={{name:'puxiao'}}>
        <ComponentA />
        <ComponentB />
        <ComponentC />
    </XxxContext.Provider>

3个子组件<ComponentA /\>、<ComponentB /\>、<ComponentC /\>都可使用useContext获取共享数据值。  


## 为什么不使用Redux？
在Hook出现以前，React主要负责视图层的渲染，并不负责组件数据状态管理，所以才有了第三方Redux模块，专门来负责React的数据管理。  

但是自从有了Hook后，使用React Hook 进行函数组件开发，实现数据状态管理变得切实可行。只要根据实际项目需求，使用useContext以及下一章节要学习的useReducer，一定程度上是可以满足常见需求的。  

毕竟使用Redux会增大项目复杂度，此外还要花费学习Redux成本。  

具体需求具体分析，不必过分追求Redux。  


---

至此，关于useContext高级用法已经讲完，useContext降低了组件之间数据传递的复杂性，让我们编写代码更加心情愉悦，而不用去关心层层嵌套问题。

接下来学习第4个Hook函数useReducer。

欢迎进入下一章节：[useReducer基本用法](https://github.com/puxiao/react-hook-tutorial/blob/master/08%20useReducer%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
