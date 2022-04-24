# utils工具方法

[toc]

## getStatusColor 根据状态返回字体颜色与背景色

> 色系已统一

### 源码

```tsx
export const getStatusColor = (status) => {
    status as ColorType;
    let statusColor = { bg: '', font: '' };
    switch (status) {
        case ColorType.DRAFT: statusColor = { bg: `#DEF7FB`, font: '#0DBEE0' }; break;//拟定
        case ColorType.APPROVED: statusColor = { bg: `#FEF1DE`, font: `#F7A835` }; break;//已批准
        case ColorType.APPROVING: statusColor = { bg: `#E9F8FA`, font: `#0000CC` }; break;//审批中
        case ColorType.CANCEL: statusColor = { bg: `rgba(87, 102, 121, 0.16)`, font: `#576679` }; break;//取消
        case ColorType.UNEFFECTED: statusColor = { bg: `volcano`, font: `orange` }; break;//未生效
        case ColorType.DONE: statusColor = { bg: `#E2EEFF`, font: `orange` }; break;//已完成
        case ColorType.EFFECTED: statusColor = { bg: `#FFFBE6`, font: `#FABD3F` }; break;//已生效
        case ColorType.REAPPROVE: statusColor = { bg: `#FFF0F6`, font: `#EB2F96` }; break;//重新审批
        case ColorType.REJECTED: statusColor = { bg: `#FFE8E8`, font: `#FF7070` }; break;//驳回
        case ColorType.BACKUP: statusColor = { bg: `#DEF7FB`, font: '#0DBEE0' }; break;//储备中
        case ColorType.PROGRESSING: statusColor = { bg: `lime`, font: `cyan` }; break;//执行中
        case ColorType.PROCESSING: statusColor = { bg: `#E2EEFF`, font: `#3889FF` }; break;//已接单
        case ColorType.PART: statusColor = { bg: `orange`, font: `lime` }; break;//部分通过
        case ColorType.APPROVAL: statusColor = { bg: `lime`, font: `orange` }; break;//待审批
        default: statusColor = { bg: `rgba(87, 102, 121, 0.16)`, font: `orange` }; break;//
    }
    return statusColor
}
```

## isDelay 延时标记

### 源码

```tsx
import moment from 'moment';
/**
 * 判断date是否延期
 * @param date 
 * @returns 
 */
export function isDelay(date) {
    return moment(moment(date).format('YYYY-MM-DD')) < moment(moment().format('YYYY-MM-DD'))
}
```

### 用例

```tsx
events: {
            load: ({ dataSet }) => {
                //行信息加载完毕后对数据打上是否延期标记
                dataSet.forEach(record => {
                    if (isDelay(record.get('planEnd'))) {
                        record.setState('isDelay', true)
                    } else {
                        record.setState('isDelay', false)
                    }
                })
            }
        },
```

## getValidationMessage表单字段校验信息

> 获取字段级校验信息

### 源码

```tsx
export const getValidationMessage = (record, fields) => {
    for (let i in fields) {
        let msg = record.getField(fields[i].name).getValidationMessage()
        if (msg !== undefined) {
            return msg
        }
    }
}
```

### 用例

```tsx
notification.error({
                description: '',
                message:  getValidationMessage(headerRecord, PlanHeaderFields),
            });
```

![image-20210813175220777](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210813175312.png)

## assign 对象深度覆盖与合并

### 源码

```tsx
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
function assign(target: any, ...others: any) {
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
```

### 用例

```tsx
const obj1 = {
    a: 1,
    b: {
        c: 2,
        d: 4,
    },
}
const obj2 = {
    a: 2,
    b: {
        c: 3,
        e: 5,
        f: 6,
    },
}
const obj3 = null
const obj4 = {
    a: 3,
    b: {
        c: 4,
        d: 5,
        f: 7,
    }
}
const resp = assign(obj1, obj2, obj3, obj4)
console.log(resp)//{ a: 3, b: { c: 4, d: 5, e: 5, f: 7 } }
```

## filterNullValueObject 对象空属性过滤

### 源码

```tsx
function filterNullValueObject(obj: any) {
    if (typeof obj !== 'object' || Array.isArray(obj)) {
        return { }
    }
    interface Obj {
        [key: string]: any
    }
    let newObj: Obj = { }
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

### 用例

```tsx
const obj = {
    a: 1,
    b: '2',
    c: undefined,
    d: '',
    e: [],
    f: null,
    g: 0,
}
const res2 = filterNullValueObject(obj)
console.log('res2', res2)//{ a: 1, b: '2', g: 0 }
```

## checkNullProps 对象空属性检查

> 暂不支持深层路径检查

### 参数

+ obj 需要检查的对象
+ _props 可选 需要检查的对象属性
+ flag 可选 是否全部返回

### 返回

**Array|String**

flag===true时返回所有为空的属性组成的数组

flag===false时返回第一个为空值的属性名字符串

### 源码

```tsx
//检查对象空属性
function checkNullProps(obj: any, _props?: Array<String>, flag = true,) {
    const props = _props || []
    const checkAllFlag = props.length === 0;
    const res = Object.keys(obj).filter(key => {
        if (!checkAllFlag && !(props.indexOf(key) > -1)) {
            return false
        }
        if (Array.isArray(obj[key])) return obj[key].length === 0
        if (typeof obj[key] === 'object') return !Boolean(obj[key]) || Object.keys(obj[key]).length === 0
        if (typeof obj[key] === 'string') return !Boolean(obj[key].trim())
        return !Boolean(obj[key]) && obj[key] !== 0
    })
    return flag ? res : res.length > 0 ? res[0] : ''
}
```

### 用例1

```ts
console.log(checkNullProps({
    a: '2',
    b: " ",
    c: null,
    d: 0.0,
}))
//[ 'b', 'c' ]
```

### 用例2

```ts
console.log(checkNullProps({
    a: '2',
    b: " ",
    c: null,
    d: 0.0,
}, ['b', 'a']))
//[ 'b' ]
```

### 用例3

```ts
console.log(checkNullProps({
    a: '2',
    b: " ",
    c: null,
    d: 0.0,
}, ['b', 'a'], false))
//b
```

## distinct 对象数组定向过滤

>   对象数组，根据某个props属性进行去重，包含前后两个方向。
>
>   默认向后去重

### 参数

+   array 待过滤数组
+   props 可选 过滤属性
+   backward 可选 是否向后过滤 默认为true

### 返回

**Array<any>**

返回过滤后的对象数组

### 源码

```typescript
//非Boolean假值检查 [],{},null,undefined,''
function isFalse(value: any) {
    if (value === undefined) return true;
    const type = typeof value;
    if (type === 'object') {
        if (value === null) return true;
        if (Array.isArray(value) && value.length === 0) return true;
        if (Object.keys(value).length === 0) return true;
    }
    if (type === 'string' && value.trim().length === 0) return true;
    return false
}
//获取props不同真值构成的新数组
function distinct(array: Array<any>, props?: string, backward = true) {
    //debugger
    if (!props) {
        return Array.from(new Set(array))
    }
    let res = new Array();//存放结果
    let baseTypeData = new Set();//存放基础类型
    let propsData = new Array();
    let indexData = new Array();
    array.forEach((i, index) => {
        if (typeof i === 'object' && !Array.isArray(i)) {
            if (isFalse(i[props])) {
                propsData.push(i[props]);
                indexData.push(index)
            } else {
                const propsIndex = propsData.indexOf(i[props])
                //如果propsData中存在，说明前面的遍历已经遇到了.array[i]是props重复项
                if (propsIndex > -1) {
                    //如果是向后保存，即props相同时保存靠后的对象
                    if (backward) {
                        //更新indexData
                        indexData.splice(propsIndex, 1);
                        indexData.push(index);
                        //保存propsData与indexData顺序一致
                        propsData.splice(propsIndex, 1);
                        propsData.push(i[props])
                    }
                } else {
                    propsData.push(i[props]);
                    indexData.push(index)
                }
            }
        }
    })
    array.forEach((i, index) => {
        if (typeof i === 'object' && indexData.indexOf(index) > -1) {
            res.push(i)
        } else if (typeof i !== 'object' && !baseTypeData.has(i)) {
            res.push(i);
            baseTypeData.add(i)
        }
    })
    return res;
}
```

用例

```js
console.log(distinct([1, 1, 
    { a: 1, b: 1 },
     { a: '1', b: 2 }, 
     { a: 2, b: 3 }, 
     { a: 1, b: 4 }, 
     { c: 1, b: 5 }, 
     { a: 1, b: 6 }], 'a'))
//[ 1, { a: '1', b: 2 }, { a: 2, b: 3 }, { c: 1, b: 5 }, { a: 1, b: 6 } ]
```

## WindowResizeEvent 浏览器窗口大小监听事件列表

>    WindowResizeEvent
>
>    浏览器视口大小事件监听类
>
>    用于解决同一页面下不同组件多次window.onresize监听页面大小导致的覆盖问题

### 源码

```typescript
interface EventItem {
    funcName: string;
    func: Function;
}
/**
 * WindowResizeEvent
 * 浏览器视口大小事件监听类
 * 用于解决同一页面下不同组件多次window.onresize监听页面大小导致的覆盖问题
 */
export class WindowResizeEvent {
    private static eventList: Array<EventItem> = [];
    private static debounceTime: number = 2000;
    constructor() {
        console.log('WindowResizeEvent.eventList', WindowResizeEvent.eventList)
        this.onResize()
    }
    static getDebounceTime() {
        return WindowResizeEvent.debounceTime
    }
    static setDebounceTime(time: number) {
        WindowResizeEvent.debounceTime = time;
    }
    static getEventList() {
        return WindowResizeEvent.eventList;
    }
    static clearAll() {
        WindowResizeEvent.eventList = []
    }
    static clear(funcName: string) {
        WindowResizeEvent.eventList = WindowResizeEvent.eventList.filter((event: EventItem) => event.funcName != funcName)
    }
    setFunc(funcName: string, func: Function) {
        console.log('funcName', funcName)
        WindowResizeEvent.eventList.push({
            funcName,
            func,
        })
    }
    onResize() {
        if (window) {
            window.onresize = debounce(() => {
                console.log('this.eventList', WindowResizeEvent.eventList)
                WindowResizeEvent.eventList.forEach(eventItem => {
                    if (typeof eventItem.func === 'function') {
                        eventItem.func()
                    }
                })
            }, WindowResizeEvent.debounceTime)
        }
    }
}
```

### 用例

```js
componentDidMount() {
   ..
    // 监听页面全屏事件
    new WindowResizeEvent().setFunc('setFullScreen', () => {
        // 全屏
        if (document.fullscreenElement) {
            this.setState({ isFullScreen: true });
        }
        // 不是全屏
        else {
            this.setState({ isFullScreen: false });
        }
    })
}
```

