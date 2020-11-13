# 18 示例：React使用Echarts所用到的hooks

本篇文章写于 2020年11月13日，距离前面文章已经过去半年，因此本文的讲述风格和示例代码，可能和前面的章节不同。



## Echarts简介

Echarts 是百度公司推出的，基于原生 JS 的图表库，免费开源 ，可用于数据可视化项目。

官网地址：https://echarts.apache.org/zh/feature.html



## Echarts基础操作

1、**Echarts 是基于原生 JS 的库，而不是 React 组件**，需要将 “图表” 挂载到 DOM

2、echarts.init(xxx-dom) 是创建 “图表” 的入口函数，该函数将创建创建真正的图表实例，并填充到 xxx-dom 中

3、一个图表 对应一个 DOM，N 个图标需要 N 个 DOM

4、图表实例通过 setOption(option) 来设置(更新)数据



## 针对以上Echarts特性，对应的 hooks

1、使用 useRef 来勾住 jsx 中的某个 DOM

2、使用 useEffect( () => {}, [] ) 来勾住 React 第一次挂载，并通过 echarts.init(xxx-dom) 创建出真正的图表

3、使用 useState 来勾住 创建出的真正图表，以便以后做各种更新操作

4、使用 useEffect( () => {}, [xxx-echart,option] ) 来不断监听组件传递过来的数据变化，并更新图表数据



> 补充说明：尽管 NPM 中已经有 echarts-for-react 这个包，已经将 Echarts 封装成可直接使用的 React 组件，但是我并不是建议使用，因为毕竟 Echarts 并不是特别难，没有必要使用别人封装好的。学习本文后，你自己也可以轻松封装自己的 Echarts 组件，灵活方便。



## 使用(封装)Echarts示例代码

> 细节不过多说，此处只演示 2 个组件源码，使用 TypeScript 编写
>
> 1. 子组件为一个图表，图表是什么类型，由 配置数据 option 中 xAxis.type 的值决定
> 2. 父组件负责调用子组件并传递图表配置数据

**父组件：**

```
import React from 'react'
import { EChartOption } from 'echarts'
import Echart from '../../components/echart'

import './index.scss'

const option: EChartOption = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [820, 932, 901, 934, 1290, 1330, 1320],
        type: 'line'
    }]
}

const IndexPage: React.FC = () => {
    return (
        <Echart option={option} />
    )
}

export default IndexPage
```



**子组件：**

```
import React, { useState, useRef, useEffect } from 'react'
import echarts, { EChartOption, ECharts } from 'echarts'

import './index.scss'

interface EchartProp {
    option: EChartOption
}

const Echart: React.FC<EchartProp> = ({ option }) => {

    const chartRef = useRef<HTMLDivElement>(null) //用来勾住渲染后的 DOM
    const [echartsInstance, setEchartsInstance] = useState<ECharts>() //用来勾住生成后的 图表实例对象

    //仅第一次挂载时执行，将 DOM 传递给 echarts，通过 echarts.init() 得到真正的图表 JS 对象
    useEffect(() => {
        if (chartRef.current) {
            setEchartsInstance(echarts.init(chartRef.current))
        }
    }, [])

    //监听依赖变化，并根据需要更新图表数据
    useEffect(() => {
        echartsInstance?.setOption(option)
    }, [echartsInstance, option])

    return (
        <div ref={chartRef} className='echarts' />
    )
}

export default Echart
```



以上示例中，父组件功能相对简单，负责调用子组件，并将图表配置数据传递给子组件。

真正需要关注的就是 子组件，在子组件中分别用到了 useRef、useState、useEffect 这 3 个 hook，尤其是 useEffect 还被使用了 2 次。



本文其实出自我写的另外一篇学习笔记：[React-Typescript中使用Echarts.md](https://github.com/puxiao/notes/blob/master/React-Typescript%E4%B8%AD%E4%BD%BF%E7%94%A8Echarts.md) 

加油，打工人！
