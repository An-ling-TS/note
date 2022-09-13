# loading装饰器

实现一个loading装饰器

+   loading装饰器会向它所装饰的函数挂载一个loading属性，可以通过xxx.prototype.loading或者xxx.loading的方式调用
+   loading装饰器会侦测所装饰的函数内部的异步操作，在异步操作开始时将loading置为true，在异步操作结束后将loading置为false
+   被装饰的函数上挂载的loading属性变化时，通知上下文中的组件进行刷新

# 实现

## 原理

+   TS装饰器模式
+   Promise
+   属性描述符
+   this与上下文
+   AOP切面编程

获取原方法的引用，在装饰器内修改原方法的描述符，在原方法调用之前向其上挂载属性loading，并置为true，保留Promise的then接口原引用，并重写then接口，在then接口内对onrejected和onfulfilled进行包装，并将包装后的回调函数作为参数传入原then接口中使用

## 实现代码

```typescript
/*
 * @Description: loading装饰器
 * @Author: anqing.liang
 * @Date: 2022-09-07 11:42:42
 * @LastEditTime: 2022-09-08 11:48:28
 * @LastEditors: anqing.liang
 */
export function loading(this: any, ...rest: any): any {
    //this 为上下文组件
    //以@loading()形式调用
    if (rest.length === 0) return _loading
    //以@loading形式调用
    if (rest.length === 3 && typeof rest[0][rest[1]] === 'function') _loading.call(this, rest[0], rest[1], rest[2])
}
function _loading(target: any, prop: string, desc: any) {
    if (typeof target[prop] !== 'function') throw ('This is not a function')
    const oldFunc = desc.value
    desc.value = function () {
        const context = this
        //获取原型
        const _prototype = desc.value?.prototype || context[prop].prototype || {}
        //获取目标
        const origin = desc.value || context[prop] || {}
        //保留原生引用
        let originalPromiseThen = Promise.prototype.then;
        //向原型和目标上挂载属性 如果有需求也可以挂载其它属性或对象 如开始，结束时间，运行时长等进行埋点
        _prototype.loading = true
        origin.loading = true
        //针对react 进行刷新调用
        context.forceUpdate && context.forceUpdate()
        // 重写Promise的then接口 针对await和then链式调用
        Promise.prototype.then = function (onfulfilled: ((value: any) => any), onrejected: ((value: any) => any)): Promise<any> {
            //对回调进行包装
            function wrappedCallback(this: any, func: Function) {
                const self = this
                return function () {
                    _prototype.loading = false
                    origin.loading = false
                    //针对react 进行刷新调用
                    context.forceUpdate && context.forceUpdate()
                    //@ts-ignore
                    return func.call(self, ...arguments)
                }
            };
            // 触发原来的回调
            return originalPromiseThen.call(this, wrappedCallback(onfulfilled), wrappedCallback(onrejected));
        };
        // 返回原函数值 并恢复Promise的then接口
        const res = oldFunc.call(this, ...arguments)
        Promise.prototype.then = originalPromiseThen
        return res
    }
}
```



# 使用示例

## 页面中使用

```typescript
import React, { Component } from 'react';
import { Button } from 'antd'
import { Bind } from 'lodash-decorators'
import { loading } from '@/utils/loading'
interface IState {
    str1: string;
    str2: string;
    str3: string;
}
export default class Index extends Component<any, IState>{
    constructor(props: any) {
        super(props)
        this.state = {
            str1: '测试字符串1',
            str2: '测试字符串2',
            str3: '测试字符串3',
        }
    }
	//内部Promise链
    @Bind
    @loading
    loadDataFunc_1() {
        fetchData(this.state.str1).then((resp: any) => {
            this.setState({ str1: resp })
        })
    }
    //内部await
    @Bind
    @loading()
    async loadDataFunc_2() {
        const newStr = Array.from(this.state.str2).reverse().join('')
        await new Promise((resolve, reject) => setTimeout(() => {
            this.setState({ str2: newStr })
            //必须要有resolve或者reject
            resolve(newStr)
        }, 3000))
    }
    //还原Promise
    @Bind
    loadDataFunc_3() {
        fetchData(this.state.str3).then((resp: any) => {
            this.setState({ str3: resp })
        })
    }
    render(): React.ReactNode {
        return <React.Fragment>
            <h1>loading装饰器-内部Promise链</h1>
            <Button onClick={this.loadDataFunc_1} loading={this.loadDataFunc_1.prototype.loading} >加载数据</Button>
            <h1>{this.loadDataFunc_1.prototype.loading ? '加载中' : '加载完成'}</h1>
            <h1>{this.state.str1}</h1>
            <div style={{ height: '100px', width: '100%' }} />
            <h1>loading装饰器-内部await</h1>
            <Button onClick={this.loadDataFunc_2} loading={this.loadDataFunc_2.prototype.loading} >加载数据</Button>
            <h1>{this.loadDataFunc_2.prototype.loading ? '加载中' : '加载完成'}</h1>
            <h1>{this.state.str2}</h1>
            <div style={{ height: '100px', width: '100%' }} />
            <h1>loading装饰器-还原Promise</h1>
            <Button onClick={this.loadDataFunc_3} loading={
                //@ts-ignore
                this.loadDataFunc_3.loading
            } >加载数据</Button>
            <h1>{this.loadDataFunc_3.prototype.loading ? '加载中' : '加载完成'}</h1>
            <h1>{this.state.str3}</h1>
        </React.Fragment>
    }
}
//接口模拟
async function fetchData(base: string) {
    return new Promise(resolve => setTimeout(() => resolve(Array.from(base).reverse().join('')), 3000))
}
```



# 效果



<video width="100%" height="600" controls autoplay>
  <source src="https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220907174223.mp4" type="video/mp4">
</video>

# 缺陷

+   对于箭头函数无效
+   对于Promise链式调用失效（只会生效第一次调用=>为了避免污染）
+   仅适用于Promise（含await）
+   对于无Promise的函数不友好