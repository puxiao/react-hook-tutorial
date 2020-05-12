# 17 React Hook 总结

首先，对你能够坚持到现在，表示深深的赞扬，学习React Hook之路不容易。  

我们快速的回顾一下之前学习过的各个hook。  

## react hook回顾

##### 定义变量  
useState()：定义普通变量  
useReducer()：定义有不同类型、参数的变量  

##### 组件传值
useContext()：定义和接收具有全局性质的属性传值对象

##### 对象引用
useRef()：获取渲染后的DOM元素对象，可调用该对象原生html的方法  
useImperativeHandle()：获取和调用渲染后的DOM元素对象拥有的自定义方法

##### 生命周期
useEffect()：挂载或渲染完成后、即将被卸载前，调度  
useLayoutEffect()：挂载或渲染完成后，同步调度  

##### 性能优化
useCallback()：获取某处理函数的引用  
useMemo()：获取某处理函数返回值的副本  

##### 代码调试
useDebugValue()：对react开发调试工具中的自定义hook，增加额外显示信息  

##### 自定义hook
useCustomHook()：将hook相关逻辑代码从组件中抽离，提高hook代码可复用性

---

至此，React Hook 你已学习完成。

真心为你鼓掌，加油，在实践中去提升自己的React Hook 战斗值吧。  