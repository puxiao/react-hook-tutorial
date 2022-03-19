# 01 React Hook 简介

首先，欢迎你来学习React Hook，通过本教程你会了解到React Hook工作原理以及我们推荐使用Hook的理由。

## 学习前提

在学习本课程之前，需要你对以下知识点有基础的了解：  
1、React基础原理；  
2、函数组件(Functional components)和类组件(class components)，属性传参(props)，自定义内部数据(state)，生命周期函数等；  
3、使用谷歌浏览器且安装了“React Developer Tools”调试工具。  

本系列文章适合有一定React开发基础的人，若是React新手，建议先从阅读React官方中文文档学起。  

接下来正式开始本教程。


## 什么是Hooks？

Hook是React 16.8版本中新增的一个新特性，丰富扩展了原有函数组件的功能，让函数组件拥有了像类组件一样的相似特性。

在之前版本中函数组件不能使用React生命周期函数，Hook本身单词意思是“钩子”，作用就是“勾住某些生命周期函数或某些数据状态，并进行某些关联触发调用”。  

不同的Hook(钩子)有不同的作用，可以勾住不同的“点”，比如“勾住组件更新完成对应的生命周期函数”、“勾住某props值的变化”等。

正因为React有多个内置Hook，所以本小节的标题才是“什么是Hooks？”，没错，用到了Hook的复数单词Hooks。 

##### 特别提醒：在React官网中使用的是Hook，而在有些教程中使用的是Hooks。在本教程中Hook和Hooks是同一个意思，不要纠结什么时候用单数什么时候是复数。  

##### 请注意：  

1、尽管函数组件拥有了类组件多大多数的相似特性，但有一点除外：函数组件中没有类组件中“自定义state”的特性，因此你无法在函数组件中使用“this.state.xx”这样的代码。  

没有不代表功能的缺失，恰恰相反，因为当你充分了解Hooks之后，你会发现函数组件内部自定义数据状态功能远远超出类组件。   

2、Hooks只能运行在函数组件中，不能运行在类组件中。  
补充：准确来说，Hooks只能运行在函数组件的“内部顶层中”，不能运行在if/for等其他函数的代码体内，不允许被if/for等包裹住。  

3、Hooks函数必须为纯函数，所谓纯函数就是函数内部不能修改可能影响执行结果的任意参数，确保每次执行的代码结果都是一样的。  


## 为什么要用Hooks？

先说一下类组件的一些缺点：  

##### 缺点一：复杂且不容易理解的“this”  
例如事件绑定处理函数，都需要bind(this)才可以正确执行。  
例如想获取某些自定义属性，都需要使用this.state.xx或this.props.xx。

这样造成代码不够精简，并且有些时候热更新不能正常运行。  

##### 缺点二：组件数据状态逻辑不能重用、组件之间传值过程复杂  

“组件数据状态逻辑不能重用”，详细解释如下：  
“组件数据状态”是由：定义数据、默认赋值、获取数据、修改数据、数据逻辑几个环节构成。 由于类组件中的组件数据状态state必须写在该组件构造函数内部，无法将state抽离出组件，因此别的组件如果有类似state逻辑，也必须内部自己实现一次，所以才得出“组件数据状态逻辑不能重用”的结论。  

“组件之间传值过程复杂”，详细解释如下：  
React本身为单向数据流，即父组件可以传值给子组件，但子组件不允许直接修改父组件中的数据状态。  

子组件为了达到修改父组件中的数据状态，通常采用“高阶组件(HOC)”或“父组件暴露修改函数给子组件(render props)”这2种方式。 这2种方式都会让组件变得复杂且降低可复用性。  


##### 缺点三：复杂场景下代码难以组织在一起  

复杂场景下，比如数据获取(data fetching)和事件订阅(event listeners)，相关代码难以组织在一起。  

“相关代码难以组织在一起”，详细解释如下：  

第1个“难以组织”的原因：数据获取和事件订阅被分散在不同生命周期函数中。

例如数据获取：组件第一次被挂载(componentDidMount)、组件每次更新完毕(componentDidUpdate)  
例如事件监听：组件第一次被挂载(componentDidMount)、组件即将被卸载(componentWillUnmount)  


第2个“难以组织”的原因：内部state数据只能是整体，无法被拆分更细致  

类组件中所有内部数据都被储存在this.state中，例如某个组件定义有2个内部数据 name,age，那么永远都是this.state.name、this.state.age。 name和age永远都只是this.state中的一个属性，无法做到将name和age拆分成独立对象个体。

所有内部数据都储存在this.state中，当内部数据复杂时，势必增加维护this.state的难度和复杂性。  

“复杂场景下代码难以组织在一起”会造成另外一个延伸性问题：加大了代码自动测试难度。


##### Hooks是如何解决上述类组件的缺点？

如果你现在迫切想知道答案，我想对你说：恭喜你，欢迎进入Hooks的世界。  

类组件缺点一：复杂且不容易理解的“this”  
Hooks解决方式：函数组件和普通JS函数非常相似，在普通JS函数中定义的变量、方法都可以不使用“this.”，而直接使用该变量或函数，因此你不再需要去关心“this”了。  

类组件缺点二：组件数据状态逻辑不能重用  
Hooks解决方式：  
通过自定义Hook，可以数据状态逻辑从组件中抽离出去，这样同一个Hook可以被多个组件使用，解决组件数据状态逻辑并不能重用的问题。  

类组件缺点二：组件之间传值过程复杂、缺点三：复杂场景下代码难以组织在一起  
Hooks解决方式：  
通过React内置的useState()函数，可以将不同数据分别从"this.state"中独立拆分出去。降低数据复杂度和可维护性，同时解决类组件缺点三中“内部state数据只能是整体，无法被拆分更细”的问题。

通过React内置的useEffect()函数，将componentDidMount、componentDidUpdate、componentWillUncount 3个生命周期函数通过Hook(钩子)关联成1个处理函数，解决事件订阅分散在多个生命周期函数的问题。

##### 最为关键的是，hook还能实现一些类组件根本不能实现的功能，比如全局共享数据，代替Redux。

如果阅读过上面文字，你依然一头雾水，不要着急，你现在只需要对Hooks有一个大体了解即可。  
随着后面的深入学习，你将逐个掌握Hooks的关键用法。  

##### 你只需记住一个结论：忘掉类组件，使用Hook进行函数组件开发，将是一个明智选择。  

<br/>

> 以下内容更新于 2021.01.10

下面讲解一下 React 的生命周期函数，面对如此复杂的生命周期函数，是没有必要过于了解和研究的，目前来说，一般只需学习使用 useEffect 这个 hook 即可。

useEffect 这个 hook 会在稍后讲解。

## React生命周期函数

React 一次状态更新，一共分为 2 个阶段、4 个生命周期。

**2 个阶段：**

1. render阶段：包含Diff算法，计算出状态变化
2. commit渲染阶段：ReactDom渲染器，将状态变化渲染在视图中

**4个生命周期：**

1. Mout(第一次挂载)
2. Update(更新)
3. Unmout(卸载)
4. Error(子项发生错误)



| 生命周期函数              | 所属阶段   | 所属生命周期        |
| ------------------------- | ---------- | ------------------- |
| constructor               | Render阶段 | Mount               |
| componentWillReceiveProps | Render阶段 | Update              |
| getDerivedStateFromProps  | Render阶段 | 并存于Moun、Update  |
| getDerivedStateFromError  | Render阶段 | Error               |
| shouldComponentUpdate     | Render阶段 | Update              |
| componentWillMount        | Render阶段 | Mount               |
| componentWillUpdate       | Render阶段 | Update              |
| render                    | Render阶段 | 并存于Mount、Update |
| componentDidMount         | Commit阶段 | Mount               |
| getSnapshotBeforeUpdate   | Commit阶段 | Update              |
| componentDidUpdate        | Commit阶段 | Update              |
| componentWillUnmount      | Commit阶段 | Unmount             |
| componentDidCatch         | Commit阶段 | Error               |

**注意事情：**

componentWillReceiveProps、componentWillMount、componentWillUpdate 这 3 个生命周期函数正在逐步被 React 官方放弃使用，不推荐继续使用这 3 个生命周期函数。

与之对应的是 getDerivedStateFromProps、getDerivedStateFromError 这 2 个这是被推荐使用的。

关于各个生命周期函数详细介绍，可以参考 React 官方文档：
https://zh-hans.reactjs.org/docs/react-component.html#commonly-used-lifecycle-methods


**补充说明：**

**目前并不是所有的生命周期函数都对应有 hook 函数。**

再次重复一遍，这些生命周期函数你只需大致了解，初学者只需学会 useEffect 这个 hook 即可。



> 以上内容更新于 2021.01.10


## 本节小结

1、Hook是React 16.8及以上版本才拥有的特性。  
2、Hook只是React“增加”的概念和一些API，对原有React体系并没有任何破坏。  
3、Hook有很多优势，比如不需要使用“this”、数据状态细致拆分、数据状态逻辑抽离出组件、代码组织更加自由灵活等。  
4、Hook只能用于函数组件，不能用于类组件中。  
5、Hook虽好，但React依然保留对类组件的支持，如果你就是不喜欢Hook，更偏向于继续使用类组件，那么也是可以的，只是你需要继续面对类组件的一些缺点。  

---

至此，你对Hook有了一个初步概念，接下来开始学习第1个Hook函数 useState。

欢迎进入下一章节：[useState基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/02%20useState%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)
