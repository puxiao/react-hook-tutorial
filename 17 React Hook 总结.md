# 17 React Hook 总结

首先，对你能够坚持到现在，表示深深的赞扬，学习React Hook之路不容易。  

我们快速的回顾一下之前学习过的各个hook。  

## react hook 回顾

##### 定义变量  
[useState()](https://github.com/puxiao/react-hook-tutorial/blob/master/02%20useState%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：定义普通变量  
[useReducer()](https://github.com/puxiao/react-hook-tutorial/blob/master/08%20useReducer%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：定义有不同类型、参数的变量  

##### 组件传值
[useContext()](https://github.com/puxiao/react-hook-tutorial/blob/master/06%20useContext%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：定义和接收具有全局性质的属性传值对象，必须配合React.createContext()使用

##### 对象引用
[useRef()](https://github.com/puxiao/react-hook-tutorial/blob/master/12%20useRef%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：获取渲染后的DOM元素对象，可调用该对象原生html的方法，可能需要配合React.forwardRef()使用  
[useImperativeHandle()](https://github.com/puxiao/react-hook-tutorial/blob/master/13%20useImperativeHandle%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：获取和调用渲染后的DOM元素对象拥有的自定义方法，必须配合React.forwardRef()使用 

##### 生命周期
[useEffect()](https://github.com/puxiao/react-hook-tutorial/blob/master/04%20useEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：挂载或渲染完成后、即将被卸载前，调度  
[useLayoutEffect()](https://github.com/puxiao/react-hook-tutorial/blob/master/14%20useLayoutEffect%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：挂载或渲染完成后，同步调度  

##### 性能优化
[useCallback()](https://github.com/puxiao/react-hook-tutorial/blob/master/10%20useCallback%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：获取某处理函数的引用，必须配合React.memo()使用  
[useMemo()](https://github.com/puxiao/react-hook-tutorial/blob/master/11%20useMemo%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：获取某处理函数返回值的副本  

##### 代码调试
[useDebugValue()](https://github.com/puxiao/react-hook-tutorial/blob/master/15%20useDebugValue%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95.md)：对react开发调试工具中的自定义hook，增加额外显示信息  

##### 自定义hook
[useCustomHook()](https://github.com/puxiao/react-hook-tutorial/blob/master/16%20%E8%87%AA%E5%AE%9A%E4%B9%89hook.md)：将hook相关逻辑代码从组件中抽离，提高hook代码可复用性


## react hook 扩展阅读
[附01：React基础知识](https://github.com/puxiao/react-hook-tutorial/blob/master/%E9%99%8401%EF%BC%9AReact%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md)  
[附02：React扩展阅读](https://github.com/puxiao/react-hook-tutorial/blob/master/%E9%99%8402%EF%BC%9AReact%E6%89%A9%E5%B1%95%E9%98%85%E8%AF%BB.md)


## 信息反馈
若有错误欢迎指正，本人微信同QQ (78657141)，或通过邮件联系：yangpuxiao@gmail.com

---

至此，React Hook 你已学习完成。

真心为你鼓掌，加油，在实践中去提升自己的 React Hook 战斗值吧。  
