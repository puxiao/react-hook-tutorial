# 前言

> 以下信息更新于 2020.11.10
>
> 最近学习了 JS 原型链、数据结构与算法，以及在思否编程课上，看了一些 卡颂(微信公众号：魔术师卡颂) 录制的《[自顶向下学 React 源码](https://ke.sifou.com/course/1650000023864436)》课程，对 React 又有了更深的认识。
>
> 此时再回顾我半年前写的这系列文章，有几点现需要补充说明一下：
>
> 1. 强烈建议你在学习 hook 之前，先学习了解一下：JS 原型链、数据与结构中的 “链” 和 “树”。  
补充强调一点：在 react 源码中，并不是使用 TypeScript，而是使用和 TS 非常类似的 flow 语法，flow 是 facebook 推出的一种 JS 静态类型检查器。我之前一直误会以为 React 源码是用 TS 写的。
> 2. 强烈推荐你先阅读我的另外一篇文章：[《自顶向下学习React源码》学习笔记#第一章：理念篇](https://github.com/puxiao/notes/blob/master/%E3%80%8A%E8%87%AA%E9%A1%B6%E5%90%91%E4%B8%8B%E5%AD%A6%E4%B9%A0React%E3%80%8B%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md) ，不需要精读，只需要大体了解一下 React 设计理念，会更加容易让你去理解 React 的渲染逻辑，利于理解 hook 。
> 3. 本系列文章中，每一个 hook 中所列出来的该 hook 源码虽然出自 React 官方源码，但实际并不是真的 hook 源码，而仅仅是对 hook 实现的简单引用。
>
> 以上信息更新于 2020.11.10




## 我是谁？

你好，欢迎你来阅读我写的关于React Hook相关的文章。

我是2020年4月才开始接触学习React的，起初摆在我面前的问题是该学习Vue还是React？  

网上关于Vue和React，有以下2条论断：  
1、Vue相当于扩展了html、而React相当于扩展了js。  
2、如果你希望快速构建应用，那么应选择Vue、如果你希望构建复杂的应用，那么应选择React。

在做了一些了解后，我决定选择学习React。不是Vue不好，而是据我了解，国内一线大厂使用React的更多一些。  


## 学习 React Hook 过程

当我决定开始学习React时，我先下载了一些React视频教程，对React、类组件开发有了基础的掌握，这个时候我接触到了 React Hook，当我稍微深入了解之后，发现 React Hook 函数组件开发才是 React 的最新主流趋势。  

> 备注：React Hook 是 React 2019年2月在16.8版本中才正式发布的。  

当我满怀激动准备学习 React Hook 时才发现相关教程非常少。

最具权威的React官方文档 翻译腔 比较重，对于 Hook 的讲解看了2遍之后依然懵懵懂懂，不明所以。 思否、掘金、雀语上面相关的文章不仅少，而且也不系统全面。 

此时我通过科学上网，在YouTube上找到了 Codevolution 专栏下的一套 “React Hooks Tutorial” 课程，开始了 React Hook 系统学习。  

其中useState、useEffect、useContext、useReducer、useCallback、useMemo、useRef、自定义Hook这些知识都来自这门课程。  

后期学习的useImperativeHandle、useLayoutEffect、useDebugValue这些知识来自于 Bitovi 专栏下的 “React Hooks — The Weird Ones” 视频课程。

## 为什么要写？

在学习每一个Hook过程中，通常我是这样进行的：  
1、看一遍视频教程  
2、看一遍React官网文档  
3、敲一遍示例代码  
4、遇到理解不了的，去各大技术站点搜索一下  
5、最后再以教给别人的口吻，写下对应Hook的教程文章  

通过这种方式，我对 React Hook 有了系统的学习，我把我写的教程文章分享出去，如果你正在准备学习 React Hook，希望能够帮助到你。

## 文章目录

[00 前言](https://github.com/puxiao/react-hook-tutorial/blob/master/00%20%E5%89%8D%E8%A8%80.md)  
[01 React Hook 简介](https://github.com/puxiao/react-hook-tutorial/blob/master/01%20React%20Hook%20%E7%AE%80%E4%BB%8B.md)  
[02 useState基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/02%20useState%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[03 useState高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/03%20useState%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)  
[04 useEffect基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/04%20useEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[05 useEffect高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/05%20useEffect%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)  
[06 useContext基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/06%20useContext%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[07 useContext高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/07%20useContext%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)  
[08 useReducer基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/08%20useReducer%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[09 useReducer高级用法](https://github.com/puxiao/react-hook-tutorial/blob/master/09%20useReducer%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md)  
[10 useCallback基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/10%20useCallback%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[11 useMemo基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/11%20useMemo%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[12 useRef基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/12%20useRef%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[13 useImperativeHandle基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/13%20useImperativeHandle%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[14 useLayoutEffect基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/14%20useLayoutEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[15 useDebugValue基础用法](https://github.com/puxiao/react-hook-tutorial/blob/master/15%20useDebugValue%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)  
[16 自定义hook](https://github.com/puxiao/react-hook-tutorial/blob/master/16%20%E8%87%AA%E5%AE%9A%E4%B9%89hook.md)  
[17 React Hook 总结](https://github.com/puxiao/react-hook-tutorial/blob/master/17%20React%20Hook%20%E6%80%BB%E7%BB%93.md)  
[18 示例：React使用Echarts所用到的hooks](https://github.com/puxiao/react-hook-tutorial/blob/master/18%20%E7%A4%BA%E4%BE%8B%EF%BC%9AReact%E4%BD%BF%E7%94%A8Echarts%E6%89%80%E7%94%A8%E5%88%B0%E7%9A%84hooks.md)  
[附01：React基础知识](https://github.com/puxiao/react-hook-tutorial/blob/master/%E9%99%8401%EF%BC%9AReact%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md)  
[附02：React扩展阅读](https://github.com/puxiao/react-hook-tutorial/blob/master/%E9%99%8402%EF%BC%9AReact%E6%89%A9%E5%B1%95%E9%98%85%E8%AF%BB.md)  

## 重要说明

本系列 React Hook 教程里的观点、思维、解释、代码 均出自我个人学习 Hook 之后的感悟和总结，难免有不准确的地方。  

甚至个别的地方掺杂了我个人的一些习惯用语和思维模式，对于 hook 的有些概念解释，我使用了自己的语言习惯，这会和React官网文档的解释略有不同，但是这些不同地方我认为是没有问题的。  

恰恰是这些不同之处，有助你更加多角度理解 React Hook。  

我写的这些教程只能作为你学习React Hook 众多参考资料中的其中一种。    


## 信息反馈

若有错误欢迎指正，本人微信同QQ (78657141)，或通过邮件联系：yangpuxiao@gmail.com


本系列文章在Github中的地址为：[https://github.com/puxiao/react-hook-tutorial](https://github.com/puxiao/react-hook-tutorial)
