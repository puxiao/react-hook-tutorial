# 15 useDebugValue基础用法

## useDebugValue概念解释
我们第十个要学习的Hook(钩子函数)是useDebugValue，他的作用是“勾住”React开发调试工具中的自定义hook标签，让useDebugValue勾住的自定义hook可以显示额外的信息。  

##### “React开发调试工具”是什么？  
答：谷歌浏览器中的一个扩展插件，名字叫“React Developer Tools”，方便我们在谷歌浏览器上进行react项目调试。  

如何安装？  
答：可在Chrome扩展程序商店搜索并安装。由于国内网络原因，如果你不会科学上网，那么可以通过国内的一些Chrome扩展程序商店网站，下载“React Developer Tools”离线的crx安装文件进行安装。具体办法可以自己百度。  

“React开发调试工具”的使用简单说明：  
如果该扩展程序安装成功，那么会有以下几种情况：  
1、对于本机开发调试的项目网页，该插件图标会变成橘黄色，且图标中间有一个小虫子，表示可以进行react源码式的调试，当代码出现错误时会精准定位出错的代码位置。  

2、对于别人开发的项目网页，该插件图标会变成蓝色，表示该网页由react开发，当代码出现错误时不能精准定位出错的代码位置。  
例如阿里云后台、腾讯云后台、百度翻译这些网页都是用react开发，访问这些网页你就会看到 调试工具图光标为蓝色。  这些大厂都用react，所以虽然学习过程中很痛苦，但是是值得的。

3、对于没有使用react的网页，该插件图标会变成灰色。  


让我们回到useDebugValue基础学习中。


## useDebugValue是来解决什么问题的？
答：useDebugValue的目的是“在react开发者工具自定义hook标签中显示额外信息”，方便我们“一眼就能找到”对应的自定义hook。   

补充说明：  
1、react官网文档中明确表示，在普通项目开发中不推荐使用useDebugValue，默认的调试输出已经很清晰可用了。  
2、除非你的自定义 hook 是作为共享库中的一部分才有价值。这样其他人更容易注意到你自定义的hook状态变化。    

##### 自定义hook？
你可能注意到本章中提到了“自定义hook”，没错。像之前学习的useState、useContext等等都是react自带的hook，这些默认的hook是我们项目开发所需要用到的各种钩子函数。

但是实际开发中，我们需要借助这些基础的、默认的、自带的hook函数，通过组合以及添加业务逻辑代码，形成自己的hook函数。  

具体如何自定义hook，稍后会单独有一章如何“自定义hook”中详细讲述。  

## useDebugValue函数源码：  
回到useDebugValue的学习中，首先看一下React源码中的[ReactHooks.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)。  

    //备注：源码采用TypeScript编写，如果不懂TS代码，阅读起来稍显困难
    export function useDebugValue<T>(
      value: T,
      formatterFn: ?(value: T) => mixed,
    ): void {
        if (__DEV__) {
        const dispatcher = resolveDispatcher();
        return dispatcher.useDebugValue(value, formatterFn);
      }
    }


上述代码看不懂没关系，本系列教程只是讲述“如何使用Hook”，并不是“Hook源码分析”。^_^  


## useDebugValue基本用法

useDebugValue(value,formatterFn)函数第1个参数为我们要额外显示的内容变量。第2个参数是可选的，是对第1个参数值的数据化格式函数。

请注意：  
1、useDebugValue应该在自定义hook中使用，如果直接在组件内使用是无效的，不会报错也不会有任何额外信息展示。    
1、一般调试不需要使用useDebugValue，除非你编写的hook是公共库中的一部分，实在是想凸显额外信息，引起别人注意。  
2、如果使用useDebugValue，最好设置第2个参数，向react开发调试工具讲清楚如何格式化展示第1个参数。    

##### 代码形式：  

    useDebugValue(xxx, xxx => xxxxx)


##### 拆解说明：  

1、xxx 为我们要重点关注的变量。  
2、xxx => xxxxx 是 (xxx) => {return xxxxx} 的简写。表明如何格式化变量xxx。  

## 如何在react调试工具中查看useDebugValue表现形式  

前提条件：  
1、在谷歌浏览器中成功安装了react开发调试工具  
2、react项目中使用了自定义hook，且hook中使用了useDebugValue  

那么你可以进行一下步骤：  
1、打开react调试网页，例如http://localhost:3000/  
2、打开谷歌浏览器调试面板(快捷键为F12)  
3、找到并点击“Components”一栏  
4、在右侧窗口中，找到“hooks”，在“hooks”下就能看到自定义hook中useDebugValue自定义显示的信息。

具体还是以下面实际例子来说明。   

## useDebugValue使用示例：  

举例：useTime是我们自定义的一个hook函数，那么在这个自定义hook中，可以通过useDebugValue对变量time进行额外信息展示。   

代码示例如下：

    //自定义hook：useTime
    function useTime(date){
      const [time,setTime] = useState(date);
      useDebugValue(time,time => new Date(time));//请注意这一行代码
      return [time,setTime];
    }

    //组件中使用useTime，伪代码片段
    const [time,setTime] = useTime(Date.now());//请注意此处使用的是自定义hook：useTime

代码分析：  
1、我们在自定义hook中，使用了useDebugValue  
2、useDebugValue第1个参数是time，向react开发调试工具表明要重点关注的变量是time。  
3、第2个参数是对time的一个格式化函数。由于time实际为一个时间戳数字，通过time => new Date(time)将时间戳转化成具体的可读时间字符串，例如此时此刻：Mon May 11 2020 14:27:39 GMT+0800 (中国标准时间)


##### 具体表现  
在谷歌浏览器调试面板的“Component”右侧，你会看到：  

    hooks  
      time:Mon May 11 2020 14:27:39 GMT+0800 (中国标准时间)  
        State:1589178459090  

假设不使用useDebuValue，默认看到的是：  

    hooks  
      time: 
        State:1589178459090  

“Mon May 11 2020 14:27:39 GMT+0800 (中国标准时间)  ”就是useDebugValue额外展示出的信息。  

你甚至还可以使用模板字符串，对格式化数据进行修改，比如将原本的第2个参数 time => new Date(time) 修改为：time => \`看这里 ${new Date(time)}\`  

在谷歌浏览器调试面板的“Component”右侧，你会看到：  

    hooks  
      time:看这里 Mon May 11 2020 14:27:39 GMT+0800 (中国标准时间)  
        State:1589178459090  


##### 再次强调，对于一般性的项目开发，是不需要使用useDebugValue来额外标记某些变量的，默认的调试输出足够我们使用了。  


---

至此，关于useDebugValue基础用法已经讲完，没有高级用法，直接进入下一个Hook。  

不！基于react 16.13版本的全部 hook，终于讲完了，没有下一个hook了。  

能坚持到现在，真的不容易，默认自带的 react hook 学完后，还需要学习如何自定义hook...   
扶我起来，再坚持一下。  

欢迎进入下一章节：[自定义hook](https://github.com/puxiao/react-hook-tutorial/blob/master/16%20%E8%87%AA%E5%AE%9A%E4%B9%89hook.md)
