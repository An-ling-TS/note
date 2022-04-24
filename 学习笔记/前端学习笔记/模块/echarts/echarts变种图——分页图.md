# 需求



​	当类目轴足够的多时，往往不得不牺牲单个条目数据在统计图上渲染的大小。

也就是说，每一条目在统计图上的大小可能会非常小，就像下面这样

![image-20211208194531432](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211208194531.png)

当然，我们可以使用和图中相同的方式来处理，使用dataZoom区域缩放系列的属性，亦或者是使用brush区域选择来处理。

但是对于一些客户而言，他们可能并不喜欢这两类解决方案，他们可能更希望每一个条目在X轴上都有对应的类目名，而非这样需要滑动或者拖动才能完全显示。当然还有一些客户也可能单纯的不习惯使用区域缩放来满足大量类目的显示。


<iframe src="https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211209114750.html" style="height:400px" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>




# 实现

修改时序图主要修改它的X轴，将其修改为类目轴，以及它的控制器显示

下面是一个简单例子

效果图

<img src="https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211208201407.png" alt="image-20211208201407131"  />

![image-20211208201250972](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211208201315.png)

```tsx
import React, { Component } from 'react';
import ReactEcharts from 'echarts-for-react';

export default class PlanEchart extends Component {
    getprops() {
        return {
            "baseOption": {
                "timeline": {
                    "axisType": "category",
                    "data": [
                        "第1页",
                        "第2页"
                    ],
                    "controlStyle": {
                        "itemSize": 24,
                        "showPlayBtn": false
                    },
                    "bottom": 0,
                    "padding": [
                        5,
                        5,
                        0,
                        5
                    ]
                }
            },
            "options": [
                {
                    "tooltip": {
                        "trigger": "axis"
                    },
                    "yAxis": [
                        {
                            "type": "value",
                        }
                    ],
                    "legend": {
                        "data": [
                            null
                        ],
                        "bottom": 0,
                        "left": "center",
                        "top": 30
                    },
                    "xAxis": [
                        {
                            "data": [
                                "仪表",
                                "其他",
                                "动设备",
                                "土建",
                                "安全",
                                "工业设计",
                                "工艺",
                                "机械设计制造及自动化"
                            ],
                            "axisLabel": {
                                "rotate": -45
                            }
                        }
                    ],
                    "series": [
                        {
                            "data": [
                                {
                                    "name": "仪表",
                                    "value": 10,
                                    "key": 12,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "其他",
                                    "value": 1,
                                    "key": 14,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "动设备",
                                    "value": 20,
                                    "key": 9,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "土建",
                                    "value": 11,
                                    "key": 13,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "安全",
                                    "value": 20,
                                    "key": 8,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "工业设计",
                                    "value": 30,
                                    "key": 4,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "工艺",
                                    "value": 10,
                                    "key": 7,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "机械设计制造及自动化",
                                    "value": 30,
                                    "key": 2,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                }
                            ],
                            "type": "bar",
                            "name": null
                        }
                    ],
                    "title": {
                        "text": "计划专业维度柱状图",
                        "x": "center"
                    },
                    "grid": {
                        "bottom": "25%"
                    }
                },
                {
                    "tooltip": {
                        "trigger": "axis"
                    },
                    "yAxis": [
                        {
                            "type": "value",
                        }
                    ],
                    "legend": {
                        "data": [
                            null
                        ],
                        "bottom": 0,
                        "left": "center",
                        "top": 30
                    },
                    "xAxis": [
                        {
                            "data": [
                                "材料成型及控制工程",
                                "特种设备",
                                "电气",
                                "设备",
                                "设备专业测试01",
                                "过程装备及控制工程",
                                "静设备",
                                "项目"
                            ],
                            "axisLabel": {
                                "rotate": -45
                            }
                        }
                    ],
                    "series": [
                        {
                            "data": [
                                {
                                    "name": "材料成型及控制工程",
                                    "value": 10,
                                    "key": 3,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "特种设备",
                                    "value": 20,
                                    "key": 15,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "电气",
                                    "value": 30,
                                    "key": 11,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "设备",
                                    "value": 10,
                                    "key": 6,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "设备专业测试01",
                                    "value": 0,
                                    "key": 1,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "过程装备及控制工程",
                                    "value": 10,
                                    "key": 5,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "静设备",
                                    "value": 4,
                                    "key": 10,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                },
                                {
                                    "name": "项目",
                                    "value": 7,
                                    "key": 16,
                                    "categoryMeaning": null,
                                    "categoryValue": null,
                                    "sort": null
                                }
                            ],
                            "type": "bar",
                            "name": null
                        }
                    ],
                    "title": {
                        "text": "计划专业维度柱状图",
                        "x": "center"
                    },
                    "grid": {
                        "bottom": "25%"
                    }
                }
            ]
        }
    }
    render() {
        return (
            <ReactEcharts
                notMerge
                style={{ height: 400, width: 300 }}
                option={
                    this.getprops()
                }
            />
        )
    }
}

```



## 包装

接下来我们就进行一些简单的封装，将后端数据通过分页图的形式展现

最终效果

<iframe src="https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211209104913.mp4" style="height:400px" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

首先，定义后端数据格式，我们接收一个对象数组

```typescript
 echartData:Array<DataItem>
```

数组中的每一条目数据结构是这样的

```typescript
interface DataItem{
    name:string;//当前统计的类目名，用于在X轴上显示
    key:number|string;//当前统计类目的键，用于向事件传参（当前变种图中暂未使用）
    value:number;//当前统计项的值
    categoryMeaning:string|null;//当前统计的维度
    categoryValue:string|null//维度键
    [key:string]:any;//其它自定义属性
}
```

返回数据示例

```typescript
[
    ...
    {
        "name": "机动设备处",
        "value": 13,
        "key": 19,
        "categoryMeaning": '日',
        "categoryValue": "day",
        "sort": null
    },
    ...
    {
        "name": "机动设备处",
        "value": 23,
        "key": 19,
        "categoryMeaning": '月',
        "categoryValue": "month",
        "sort": null
    },
    ...
]
```

定义好了数据结构，接下来要做的就是封装

首先看一下封装好后的调用

```tsx
//@ts-nocheck
import React, { Component } from 'react';
import ReactEcharts from 'echarts-for-react';
import { fetchData } from './service/echartService';
import { getBarOption } from './utils/echartProps';
interface DataItem {
    name: string;
    key: number | string;
    value: number;
    categoryMeaning: string | null;
    categoryValue: string | null
    [key: string]: any;
}
interface IState {
    echartData: Array<DataItem>
}
export default class PlanEchart extends Component<any, IState> {
    constructor(props) {
        super(props)
        this.state = {
            echartData: []
        }
    }
    componentDidMount() {
        fetchData().then(echartData => this.setState({ echartData }))
    }

    render() {
        const { echartData } = this.state
        return (
            <ReactEcharts
                notMerge
                style={{ height: 400, width: 500 }}
                option={
                    getBarOption(echartData, {
                        title: { text: '分页柱状图', x: 'center' },
                        legend: { top: 30 },
                        xAxis: [{
                            axisLabel: {
                                rotate: -45
                            }
                        }],
                        grid: {
                            bottom: '25%'
                        }
                    }, ['month', 'day'], 5)
                }
            />
        )
    }
}

```



然后就是函数封装了，因为考虑到不止一种图，因此尽量的封装了更多的内置函数，以便于其它统计图的使用

```typescript
/*
 * @Description: 
 * @Author: anqing.liang
 * @Date: 2021-12-09 10:07:19
 * @LastEditTime: 2021-12-09 10:07:20
 * @LastEditors: anqing.liang
 */
import { isEmpty, omit } from 'lodash';
import { assign, filterNullValueObject } from './utils'
interface DataItem {
    name: string;
    key: number | string;
    value: number;
    categoryMeaning: string | null;
    categoryValue: string | null
    [key: string]: any;
}
/**
 * 柱状图配置
 * @param dataSet 数据源
 * @param echartOption 自定义配置
 * @param dimension 维度 用于维度排序 如果不传，使用默认排序
 * @param pageSize 页码 用于指定每一页所含的类目数
 * @returns 
 */
export const getBarOption = (dataSet: Array<DataItem>, echartOption: any, dimension = [], pageSize?: number) => {
    const { legend, xAxis = [], yAxis = [], series = [], ...others } = echartOption;

    const seriesOption = {
        ...series[0],
        type: 'bar',
    }
    const seriesData = getseriesData(dataSet, seriesOption, dimension);
    const option = {
        tooltip: {
            trigger: 'axis'
        },
        // dataZoom: {
        //     start: 0,
        //     type: "inside"
        // },
        yAxis: [
            {
                type: 'value',
                ...yAxis[0]
            }
        ],
        legend: {

            data: getLegendData(seriesData),
            bottom: 0,
            left: 'center',
            ...legend,
        },
        xAxis: [
            {
                data: getXData(seriesData),
                ...xAxis[0],
            },
        ],
        series: seriesData,
        ...others,
    };
    if (pageSize && pageSize > 0) {
        let resp = getPagingOption(option, seriesData, pageSize);
        return resp;
    }


    return isEmpty(seriesData) ? false : option;
};
/**
 * 在data中对column列中的不同值进行统计
 * @param data 
 * @param columns 
 */
function getseriesData(dataSet: Array<DataItem>, seriesOption: any, dimension: any, series?: any) {
    if (!dataSet || dataSet.length === 0) return [];
    let resp = new Array();
    let seriesMap = new Map();//存放不同系列的系列名与坐标
    dataSet.forEach((item: any) => {
        const { categoryMeaning } = item;
        if (seriesMap.has(categoryMeaning)) {
            resp[seriesMap.get(categoryMeaning)]?.data.push(item);
        } else {
            let data = filterNullValueObject({ name: categoryMeaning, data: [item], ...seriesOption })
            //如果存在为系列单独配置的属性，则使用单独配置的属性
            //如果前面的赋值未生效，手动给name赋值
            data.name = categoryMeaning
            if (series && series[resp.length]) {
                data = assign(data, series[resp.length])
            }
            resp.push(data);
            seriesMap.set(categoryMeaning, resp.length - 1);
        }
    })
    //根据维度排序
    resp = orderByDimension(resp, dimension);


    return resp;
}
/**
 * 根据分页大小配置多列option
 * @param UcOption  基础echart配置
 * @param seriesData 系列数据
 * @param pageSize  页大小
 * @returns 
 */
function getPagingOption(UcOption: any, seriesData: any, pageSize: number) {
    if (!pageSize || pageSize <= 0 || seriesData.length === 0) return UcOption
    let pagingSeriesData = getPagingSeriesData(seriesData, pageSize);
    const { baseEchartOption, ...other } = UcOption
    let resp = {
        baseOption: {},
        options: [],
    }
    resp.options = pagingSeriesData.map((seriesData: any) => {
        return {
            ...other,
            legend: {
                ...UcOption?.legend,
                data: getLegendData(seriesData),
            },
            xAxis: [
                {
                    ...UcOption?.xAxis[0],
                    data: getXData(seriesData),
                },
            ],
            yAxis: [
                {
                    ...UcOption?.yAxis[0],
                }
            ],
            series: seriesData,

        }
    })
    resp.baseOption = {
        timeline: {
            axisType: "category",
            data: getPageData(resp.options.length),
            controlStyle: {
                itemSize: 24,
                showPlayBtn: false,
            },
            bottom: 0,
            padding: [5, 5, 0, 5],
            ...baseEchartOption?.timeline,
        },
        ...omit(baseEchartOption, 'timeline'),
    }
    return resp;
}
/**
 * 根据seriesData获取Legend
 * @param seriesData
 */
function getLegendData(seriesData: any) {
    if (!seriesData) return []
    return seriesData.map((item: any) => item.name)
}
/**
 * 根据seriesData获取横坐标
 * @param seriesData
 */
function getXData(seriesData: any) {
    if (!seriesData || seriesData.length === 0) return [];
    const { data } = seriesData[0]
    let resp = new Array();
    data?.forEach((item: any) => {
        if (!resp.includes(item.name)) {
            resp.push(item.name);
        }
    })
    return resp;
}
/**
 * 获取分页的pagingSeriesData列表
 * @param seriesData 
 * @param pageSize 
 * @returns 
 */
function getPagingSeriesData(seriesData: any, pageSize: number) {
    if (!pageSize || pageSize <= 0) return seriesData;
    let resp = new Array();
    //系列遍历
    for (let i in seriesData) {
        //设置模板，主要更换data，模板中的其他值不变
        let template = seriesData[i];
        //分页次数 向上取整
        const steps = Math.ceil((template?.data?.length || 0) / pageSize);
        for (let i = 0; i < steps; i++) {
            const start = i * pageSize;
            //当截取范围超出时，指定截取范围为data.length 
            const end = Math.min((i + 1) * pageSize, template?.data?.length || 0)
            const newData = template.data.slice(start, end)
            if (resp.length <= i) {
                //当页数小于分页步骤时，说明这是该系列第一次进行该页数据添加
                resp.push([{
                    ...template,
                    data: newData,
                }])
            } else {
                resp[i].push({
                    ...template,
                    data: newData,
                })
            }
        }
    }
    return resp;
}
//页码显示
function getPageData(pages: number) {
    let resp = new Array()
    for (let i = 1; i <= pages; i++) {
        resp.push(`第${i}页`)
    }
    return resp;
}
//维度排序
function orderByDimension(seriesData: any, dimension: any) {
    if (!dimension || dimension.length !== seriesData.length) return seriesData;
    //if (!dimension || dimension.length !== seriesData.length) return seriesData;
    let resp = new Array(dimension.length);
    seriesData.forEach((item: any) => {
        let index = dimension.indexOf(item?.data[0]?.categoryValue);
        if (index > -1) {
            resp[index] = item
        }
    })
    if (resp.filter(Boolean).length !== dimension.length) {
        return seriesData
    }
    return resp
}

```

下面则是使用的工具函数

```typescript
/**
 *
 * @param target 目标对象 被覆盖的对象
 * @param orign  源对象 覆盖的对象
 * @returns
 */
function func(target: any, orign: any) {
    if (orign === null || target === null) return JSON.parse(JSON.stringify(orign))
    if (typeof target === 'object' && typeof orign === 'object') {
        if (Array.isArray(orign) || Array.isArray(target)) {
            return JSON.parse(JSON.stringify(orign))
        }
        //深拷贝target
        let resp = JSON.parse(JSON.stringify(target));
        for (let key in orign) {
            if (key in resp) {
                resp[key] = func(resp[key], orign[key]);
            } else {
                //如果resp中不存在key属性，则直接合并
                resp[key] = JSON.parse(JSON.stringify(orign[key]))
            }
        }
        return resp
    } else {
        //如果有一个是基础类型，则直接返回第二个参数替换
        return JSON.parse(JSON.stringify(orign))
    }
}
/**
 *
 * @param target 目标对象 被覆盖的对象
 * @param others  源对象 覆盖的对象
 * @returns
 */
//@ts-ignore
export function assign(target: any, ...others: any) {
    if (target === null || target === undefined) {
        throw new TypeError('Cannot convert undefined or null to object');
        //不允许向空目标中合并
    }
    let resp = JSON.parse(JSON.stringify(target))
    for (let index = 1; index < arguments.length; index++) {
        let nextSource = arguments[index];
        if (nextSource !== null && nextSource !== undefined) {
            resp = func(resp, nextSource)
        }
    }
    return resp
}
//对象空属性过滤
export function filterNullValueObject(obj: any) {
    if (typeof obj !== 'object' || Array.isArray(obj)) {
        return {}
    }
    interface Obj {
        [key: string]: any
    }
    let newObj: Obj = {}
    Object.keys(obj).forEach((item: string) => {
        if (obj[item] === 0 || Boolean(obj[item])) {
            if ((!Array.isArray(obj[item]) || (Array.isArray(obj[item]) && obj[item].length > 0))) {
                newObj[item] = obj[item]
            }
        }
    })
    return newObj;
}
```

