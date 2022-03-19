# 06 useContext基础用法

## useContext概念解释
我们第三个要学习的Hook(钩子函数)是useContext，他的作用是“勾住”获取由React.createContext()创建、<XxxContext.Provider>添加设置的共享数据value值。useContext可以替代<XxxContext.Consumer>标签，简化获取共享数据的代码。  

我们知道，原本不同级别的组件之间传递属性值，必须逐层传递，即使中间层的组件不需要这些数据。  
注意：这里说的组件指React所有组件，包含类组件和函数组件。  

数据层层传递增加了组件的复杂性，降低了可复用性。为了解决这个问题，我们可以使用以下方式：  
1、在组件顶层或单独的模块中，由React.createContext()创建一个共享数据对象；  
2、在父组件中添加共享数据对象的引用，通过且只能通过<XxxContext.provider value={{xx:'xxx'}}></XxxContext.provider>的形式将数据传递给子组件。请注意传值必须使用value={obj}这种形式，若值本身为字符串则可以改为 value='xxx'；  
3、若下一层的子组件用不到共享数据对象中的数据，则可以不做任何属性标签传递；  
4、若某一层的子组件需要用到共享数据对象的数据，则可通过<XxxContext.Consumer></XxxContext.Consumer>获取到数据；  
5、在类组件中除了<XxxContext.Consumer>标签，还有另外一种获取共享数据方式：static xxx = XxxContext; 但是这种形式在函数组件中无法使用。  

简而言之<XxxContext.Provider>用来添加共享数据、<XxxContext.Consumer>用来获取共享数据。  
备注：provider单词本意为供应者、consumer单词本意为消费者，刚好对应他们相对于共享数据的关系。  

上面简单描述了React.createContext()的用法，由于本系列文章主要讲Hook的使用方法，React本身的知识点并不是重点讲解对象。若你对React.createContext()、<XxxContext.Provider>、<XxxContext.Consumer>的用法还不太明白，请通过其他途径自行学习。  

让我们回到useContext学习中。


## useContext是来解决什么问题的？
答：useContext是<XxxContext.Consumer>的替代品，可以大量简化获取共享数据值的代码。

补充说明：  
1、函数组件和类组件，对于<XxxContext.Provider>、<XxxContext.Consumer>使用方式没有任何差别。  
2、你可以在函数组件中不使用useContext，继续使用<XxxContext.Consumer>，这都没问题。只不过使用useContext后，可以让获取共享数据相关代码简单一些。  


## useContext函数源码：  
回到useContext的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useContext<T>(
      Context: ReactContext<T>,
      unstable_observedBits: number | boolean | void,): T {
      const dispatcher = resolveDispatcher();
      if (__DEV__) {
        if (unstable_observedBits !== undefined) {
          console.error(
            'useContext() second argument is reserved for future ' +
            'use in React. Passing it is not supported. ' +
            'You passed: %s.%s',
            unstable_observedBits,
            typeof unstable_observedBits === 'number' && Array.isArray(arguments[2])
            ? '\n\nDid you call array.map(useContext)? ' +
              'Calling Hooks inside a loop is not supported. ' +
              'Learn more at https://fb.me/rules-of-hooks'
            : '',
          );
      }

      // TODO: add a more generic warning for invalid values.
      if ((Context: any)._context !== undefined) {
        const realContext = (Context: any)._context;
        // Don't deduplicate because this legitimately causes bugs
        // and nobody should be using this in existing code.
        if (realContext.Consumer === Context) {
          console.error(
            'Calling useContext(Context.Consumer) is not supported, may cause bugs, and will be ' +
              'removed in a future major release. Did you mean to call useContext(Context) instead?',
          );
        } else if (realContext.Provider === Context) {
          console.error(
            'Calling useContext(Context.Provider) is not supported. ' +
              'Did you mean to call useContext(Context) instead?',
          );
        }
      }
    }
      return dispatcher.useContext(Context, unstable_observedBits);
    }

上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。之所以贴出源码只是想让你看懂以后告诉我，反正我是没看懂。^_^  


## useContext基本用法

useContext(context)函数可以传入1个参数，该参数为共享数据对象的实例，useContext函数会返回该共享对象实例的value值。  
 

##### 代码形式：  

    import GlobalContext from './global-context'; //引入共享数据对象
    
    function Component(){
      const global = useContext(GlobalContext); //在函数组件中声明一个变量来代表该共享数据对象的value值

      //若想获取共享数据对象中的属性xxx的值，直接使用global.xxx即可
      return <div>
        {global.xxx}
      </div>
    }


##### 拆解说明：  

1、子组件(函数组件)需要先引入共享数据对象GlobalContext；  
2、内部定义一个常量global，用来接收useContext函数返回GlobalContext的value值；  
3、函数组件在return时，可以不使用<GlobalCount.Customer>标签，而是直接使用global.xx来获取共享数据；  
4、请注意，这里执行的依然是单向数据流，只可以获取global.xx，不可以直接更改global.xx;  

#### '引入GlobalContext'补充说明
示例中是通过import方式引入的，如果直接把GlobalContext定义在该组件内部，那不是就不用import了吗？  
答：是的，你可以这么做。只不过定义在外部单独的模块中，各个组件都可以引用。  

#### 'global'补充说明
为了代码语义化，上述代码中使用到了global这个单词，但是请注意，该单词和原生JS中global(全局变量)无任何关联。实际项目中你可以使用任意具有语义的相关单词。比如定义用户共享数据你可以定义为UserContext、新闻共享数据你可以定义为NewsContext等。  


## useContext使用示例：  

举例：若某React组件一共由3层组件嵌套而成，从外到里分别是AppComponent、MiddleComponent、ChildComponent。AppComponent需要传递数据给ChildComponent。  

若使用useContext来实现，代码示例如下：  

    //global-context.js
    import React from 'react';
    const GlobalContext = React.createContext(); //请注意，这里还可以给React.createContext()传入一个默认值
    //例如：const GlobalContext = React.createContext({name:'Yang',age:18})
    //假如<GlobalContext.Provider>中没有设置value的值，就会使用上面定义的默认值
    export default GlobalContext;

    ...

    //component.js
    import React, { useContext } from 'react';
    import GlobalContext from './global-context';

    function AppComponent() {
      //标签<GlobalContext.Provider>中向下传递数据，必须使用value这个属性，且数据必须是键值对类型的object
      //如果不添加value，那么子组件获取到的共享数据value值是React.createContext(defaultValues)中的默认值defaultValues
      return <div>
        <GlobalContext.Provider value={{name:'puxiao',age:34}}>
            <MiddleComponent />
        </GlobalContext.Provider>
      </div>
    }

    function MiddleComponent(){
      //MiddleComponent 不需要做任何 “属性数据传递接力”，因此降低该组件数据传递复杂性，提高组件可复用性
      return <div>
        <ChildComponent />
      </div>
    }

    function ChildComponent(){
      const global = useContext(GlobalContext); //获取共享数据对象的value值
      //忘掉<GlobalContext.Consumer>标签，直接用global获取需要的值
      return <div>
        {global.name} - {global.age}
      </div>
    }

    export default AppComponent;

假如ChildComponent不使用useContext，而是使用<GlobalContext.Consumer>标签，那么代码相应修改为：  

    function ChildComponent(){
      return <GlobalContext.Consumer>
        {
            global => {
                return <div>{global.name} - {global.age}</div>
            }
        }
      </GlobalContext.Consumer>
    }

使用useContext可以大大降低获取数据代码复杂性。  

请注意：useContext只是简化了获取共享数据value的代码，但是对于<XxxContext.Provider>的使用没有做任何改变，如果组件需要设置2个XxxContext，那么依然需要进行<XxxContext.Provider>嵌套。  


上述代码中AppComponent只向下传递出去1个共享数据对象value值，那如果需要同时传递多个共享数据对象的value值，那该如何实现？  
关于这个问题，会在 useContext高级用法中讲解。  


---

至此，关于useContext基础用法已经讲完。

欢迎进入下一章节：[useContext高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/07%20useContext%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)
