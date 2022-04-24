# Echart通用组件封装

[toc]

## 需求

根据传入的DataSet实例或者约定格式的DataSource数据源，进行数据的统计与统计图的绘制

## 组件文件结构

![image-20210811100433974](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210823144613.png)

+ Charts
  + utils
    + ChartProps.tsx-----------echart图表属性配置
  + Echart.tsx-----------------------组件
  + enum.ts-------------------------通用常量，枚举

## 特点说明

+ 允许传入dataSet实例和统计列进行自动统计
+ 允许同时传入统计多个统计列，并且当columns中的统计列为值集或视图时会自动翻译
+ 允许使用自定义的echartOption配置覆盖默认的echart配置，echart配置优先级为  外部自定义>组件默认>开发平台配置
+ 允许绑定echart点击事件
+ 允许使用维度顺序列表进行维度排序
+ 允许设置分页大小，维护每一页的统计列数

## 优化项

+ 主题配置接口

+ dataSet统计列类型检测

  + string类型转为pie饼图，统计方式为dataSet遍历合计。

    统计列格式

    ```tsx
    ['xxx1','xxx2']
    ```

    

  + number类型优先转为bar柱状图或者line折线图 ，并指定横坐标展示字段和key，统计方式为单行展示，统计列格式

    ```tsx
    [
        {
            name:'textFieldName',//横坐标文本字段
            key:'idFileName',//record键或标志符字段
            value:'valueFieldName',//record值字段
        }]
    ```

    

+ 

## API

```tsx
	属性 					 		说明 								类型
@param dataSet          必选 dataset实例或者约定的数据源结构
@param columns          可选 columns  dataSet统计列
@param chartType        必选 echart  类型;
@param echartOption     可选 echart option配置
@param style            可选 style样式
@param onClick          可选 echart点击事件
@param dimension        可选 维度顺序                           Array<string>
@param pageSize         可选 分页大小                           number
约定的dataset数据源 示例
[
 {
     {
         name:'运行一部',
         value:67,
         key:'yunxinyibu'
         categoryMeaning:'本日'
         categoryValue:'DAY'
     }，
 }
```

### 数据结构

### dataSet

+ dataSet实例
+ 约定格式的数据源

```tsx
[
 {
     {
         name:'xxx',				//横坐标文本
         value:67,							//纵坐标统计值
         key:'x'			 	 //横坐标对应的key，用于事件传值
         categoryMeaning:'本日'	 //维度文本
         categoryValue:'DAY'		  //维度key，用于事件传值
     }，
 }
```



###  seriesData

维度数据

```tsx
[
    name:'xxx',									//维度文本
    data:[{name:'xxx',value:20,key:'xx',...other}] //填装的数据
    type:'bar'									//echart类型
    ...echartSeriesOption,						 //echart维度配置项
]
```

###  pagingSeriesData

分页的维度数据

```tsx
Array<seriesData>
```

### echartOption

echart配置项

```tsx
{
    baseEchartOption:{},//新增字段，传递分页的基础echart配置，其它echart配置将作为维度配置属性
    ...echart配置项
}
```



## 工具方法说明

### getPageData

> 根据页数设置分页文本

```tsx
function getPageData(pages) {
    let resp = new Array()
    for (let i = 1; i <= pages; i++) {
        resp.push(`第${i}页`)
    }
    return resp;
}
```

后期可考虑将分页文本的内容配置暴露出去

### getSeriesData

> 填装维度数据

```tsx
function getseriesData(dataSet: DataSet | Array<Object>, _columns, seriesOption, dimension) {
    if (!dataSet || (!Array.isArray(dataSet) && !(dataSet instanceof DataSet)) || dataSet.length === 0) return [];
    let resp = new Array();
    if (dataSet instanceof DataSet) {
        let keysMap = new Map();//存放已发现的列不同值,与其坐标
        const radius = Math.round(100 / _columns.length)
        const offset = Math.round(radius / 3)
        let columns = _columns;
        if (dimension && dimension.length === columns.length) {
            //如果存在维度顺序，且维度长度与统计列数目一致，则以维度进行统计
            columns = dimension;
        }
        Array.from(columns).forEach((column: any, index) => {
            let seriesData = new Array();
            dataSet.forEach((record: any) => {
                let text = record.getField(column).getText()
                let key = record.getField(column).getValue()
                if (keysMap.has(text)) {
                    seriesData[keysMap.get(text)].value += 1;
                } else {
                    seriesData.push({ name: text, value: 1, key, column });
                    keysMap.set(text, seriesData.length - 1);
                }
            })
            resp.push({
                ...seriesOption,
                radius: [`${radius * index}%`, `${radius * (index + 1) - offset}%`],
                data: seriesData,
            })
        })
        //console.log('seriesData', seriesData)

    } else {
        let seriesMap = new Map();//存放不同系列的系列名与坐标
        dataSet.forEach((item: any) => {
            const { categoryMeaning } = item;
            if (seriesMap.has(categoryMeaning)) {
                resp[seriesMap.get(categoryMeaning)]?.data.push(item);
            } else {
                resp.push({ name: categoryMeaning, data: [item], ...seriesOption, });
                seriesMap.set(categoryMeaning, resp.length - 1);
            }
        })
        //console.log('resp dataSource', resp)
        resp = orderByDimension(resp, dimension);
    }

    return resp;
}
```



### getXData

>  获取横坐标数据

```tsx
function getXData(seriesData) {
    if (!seriesData || seriesData.length === 0) return [];
    const { data } = seriesData[0]
    let resp = new Array();
    data?.forEach(item => {
        if (!resp.includes(item.name)) {
            resp.push(item.name);
        }
    })
    return resp;
}
```

### getLegendData

> 获取图例数据

```tsx
function getLegendData(seriesData) {
    if (!seriesData) return []
    return seriesData.map(item => item.name)
}
```

### orderByDimension

> 根据维度列表重新排列seriesData

```tsx
function orderByDimension(seriesData, dimension) {
    if (!dimension || dimension.length !== seriesData.length) return seriesData;
    let resp = new Array(dimension.length);
    seriesData.forEach(item => {
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

### getPagingOption

>根据分页大小配置多列option

```tsx
function getPagingOption(UcOption, seriesData, pageSize) {
    if (!pageSize || pageSize <= 0 || seriesData.length === 0) return UcOption
    let pagingSeriesData = getPagingSeriesData(seriesData, pageSize);
    const { baseEchartOption, ...other } = UcOption
    let resp = {
        baseOption: {},
        options: [],
    }
    resp.options = pagingSeriesData.map(seriesData => {
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
            ...baseEchartOption?.timeline,
        },
        ...omit(baseEchartOption, 'timeline'),
    }
    return resp;
}
```

### getPagingSeriesData

> 获取分页的seriesData列表

```tsx
/**
 * 获取分页的pagingSeriesData列表
 * @param seriesData 
 * @param pageSize 
 * @returns 
 */
function getPagingSeriesData(seriesData, pageSize) {
    if (!pageSize || pageSize <= 0) return seriesData;
    let resp = new Array();
    for (let i in seriesData) {
        //设置模板，主要更换data，模板中的其他值不变
        let template = seriesData[i];
        //分页次数
        const steps = Math.ceil((template?.data?.length || 0) / pageSize);
        for (let i = 0; i < steps; i++) {
            const start = i * pageSize;
            //当截取范围超出时，指定截取范围为data.length
            const end = Math.min((i + 1) * pageSize, template?.data?.length || 0)
            const newData = template.data.slice(start, end)
            if (resp.length <= i) {
                //当页数小于分页步骤时，说明这是第一次进行该页数据添加
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
```

### getPieOption

> 获取饼状图配置

```tsx
// 饼状图
export const getPieOption = (dataSet: DataSet | Array<Object>, columns, echartOption, dimension) => {
    const { legend, series = [], tooltip, ...others } = echartOption;
    const seriesOption = {
        name: others?.title?.text || null,
        type: 'pie',
        ...series[0],
        //center: ['50%', '50%'],
    }
    const option = {
        ...others,
        legend: {
            type: 'scroll',
            bottom: 0,
            left: 'center',
            ...legend,
        },
        tooltip: {
            trigger: 'item',
            formatter: '<b>{a}</b> <br/>{b} : {c} ({d}%)',
            ...tooltip,
        },
        series: getseriesData(dataSet, columns, seriesOption, dimension)
    };
    return option;
};
```

### getBarOption

> 获取柱状图配置

```tsx
// 柱状图
export const getBarOption = (dataSet: DataSet | Array<Object>, columns, echartOption, dimension, pageSize?) => {
    const { legend, xAxis = [], series = [], ...others } = echartOption;
    const seriesOption = {
        ...series[0],
        type: 'bar',
    }
    const seriesData = getseriesData(dataSet, columns, seriesOption, dimension);
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
                type: 'value'
            }
        ],
        legend: {
            ...legend,
            data: getLegendData(seriesData),
            bottom: 0,
            left: 'center',
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
```

## 用例

```tsx
<Row >
    <Col span={12}>
        <Echart
            columns={['planStatus']}
            dataSet={this.echartPieDS}
            echartOption={{
                title: { text: '计划状态饼图分析', x: 'center' }
            }}
            style={{ height: 400 }}
            onClick={(params) => handlePieClick(params)}
            chartType={ChartType.PIE} />
    </Col>
    <Col span={12}>
        <Echart
            dataSet={this.echartBarDataSource}
            echartOption={{
                title: { text: '各部门本日/本周/本月计划柱状图', x: 'center' },
                legend: { top: 30 },
            }}
            style={{ height: 400 }}
            onClick={(params) => handleBarClick(params)}
            dimension={['DAY', 'WEEK', 'MONTH']}
            pageSize={4}
            chartType={ChartType.BAR} />
    </Col>
</Row>
```

## 效果

![image-20210813182053771](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210813182053.png)

