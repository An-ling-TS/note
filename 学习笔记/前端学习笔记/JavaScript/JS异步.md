

>   这是一篇关于JavaScript异步的学习笔记，知识点主要提炼或引用自《JavaScript高级程序设计》（第三版）（第四版）与《你不知道的JavaScript》（中卷）（下卷）以及网络站点
>
>   这篇文章将结合浏览器基础，并发行为，事件循环对JS异步与同步进行学习

在此之前，需要

+   了解迭代，期约，async/await，
+   了解一定并发合作
+   了解浏览器内核（渲染进程）

# 同步异步

>   同步行为对应内存中顺序执行的处理器指令。每条指令都会严格按照它们出现的顺序来执行，而每 条指令执行后也能立即获得存储在系统本地（如寄存器或系统内存）的信息。这样的执行流程容易分析程序在执行到代码任意位置时的状态（比如变量的值）。

>   异步行为类似于系统中断，即当前进程外部的实体可以触发代码执行。异步操作经常是必 要的，因为强制进程等待一个长时间的操作通常是不可行的（同步操作则必须要等）。如果代码要访问一些高延迟的资源，比如向远程服务器发送请求并等待响应，那么就会出现长时间的等待。

在JavaScript中，同步行为往往是可以预料的，你可以在同步函数执行之前就预测出它什么情况下返回，何时返回（不考虑执行时间）,返回什么。但是对于异步行为，这些往往是难以预测的，例如返回远程服务器资源，你无法准确的预测它何时返回，返回什么数据。

# 假定四个接口

我们在这里假定两组接口，每组两个，以及一个计时器，用来在后面进行测试

```js
import axios from 'axios'
const BASE_URL = 'http://localhost:3100'
import moment from 'moment'
class Clock {
    startTime: Date | null;
    durationStartTime: Date | null;
    endTime: Date | null;
    name:string|undefined;
    constructor(name:string='clock') {
        this.startTime = null;
        this.endTime = null;
        this.durationStartTime = null;
        this.name=name
    }
    start() {
        console.log(`${this.name} start`);
        this.startTime = new Date();
        this.durationStartTime = new Date();
    }
    end() {
        console.log(`${this.name} end`);
        this.endTime = new Date();
        return this.duration();
    }
    duration() {
        if (this.endTime && this.startTime) {
            const dura = moment.duration(this.endTime.getTime() - this.startTime.getTime()).asSeconds()
            console.log(`${this.name} end duration ${dura}s`);
            return dura;
        } else if (this.durationStartTime && !this.endTime) {
            const endTime = new Date();
            const dura = moment.duration(endTime.getTime() - this.durationStartTime.getTime()).asSeconds();
            this.durationStartTime = new Date()
            console.log(`${this.name} now duration ${dura}s`);
            return dura;
        }
        console.log(0)
        return 0
    }
}
function getA() {
    return axios.get(`${BASE_URL}/getA`)
}
function getB(A: any) {
    return axios.post(`${BASE_URL}/getB`, {
        A,
    })
}
function getX() {
    return axios.get(`${BASE_URL}/getX`)
}
function getY(X: any) {
    return axios.post(`${BASE_URL}/getY`, {
        X,
    })
}
```

其中，getB接口依赖于getA接口的返回，getY接口依赖于getX接口的返回

对应的后端接口如下

```js

const express = require('express')
const app = express()
const port = 3100

app.use(express.json())
app.use(express.urlencoded({ extended: true }))

app.get('/getA', (req, res) => {
    const resp = Math.random().toFixed(1) * 10;
    setTimeout(() => res.send({
        resp,
    }), 5000)
})
app.post('/getB', (req, res) => {
    const { body } = req
    const A = body.A
    const resp = A * 2
    setTimeout(() => res.send({
        resp,
    }), 5000)
})


app.get('/getX', (req, res) => {
    const resp = Math.random().toFixed(1) * 10;
    setTimeout(() => res.send({
        resp,
    }), 5000)
})
app.post('/getY', (req, res) => {
    const { body } = req
    const X = body.X
    const resp = X * 3
    setTimeout(() => res.send({
        resp,
    }), 5000)
})
app.listen(port, () => {
    console.log(`Example app listening at http://localhost:${port}`)
})
```

**这里暂时只讨论ES6及其以后的内容，因此关于ES6以前的异步回调及其带来的深度嵌套（地狱回调）问题暂放一边。**

# 深入之前

在深入学习JS异步之前，我们需要熟悉并掌握JS异步的使用，尤其是期约Promise和await与async关键字

## 期约Promise

>   ECMAScript 6新增的引用类型 Promise，可以通过 new 操作符来实例化。创建新期约时需要传入 执行器（executor）函数作为参数

期约是一个有状态的对象，可能处于如下 3种状态之一：

+   待定 pending
+   兑现 fulfilled，有时候也称为“解决”，resolved
+   拒绝 rejected

待定（pending）是期约的最初始状态。在待定状态下，期约可以落定（settled）为代表成功的兑现 （fulfilled）状态，或者代表失败的拒绝（rejected）状态。无论落定为哪种状态都是不可逆的。只要从待定转换为兑现或拒绝，期约的状态就不再改变。

>   重要的是，期约的状态是私有的，不能直接通过 JavaScript 检测到。这主要是为了避免根据读取到 的期约状态，以同步方式处理期约对象。另外，期约的状态也不能被外部 JavaScript 代码修改。这与不能读取该状态的原因是一样的：**期约故意将异步行为封装起来，从而隔离外部的同步代码。**

期约主要有两大用途。首先是抽象地表示一个异步操作。期约的状态代表期约是否完成。“待定” 表示尚未开始或者正在执行中。“兑现”表示已经成功完成，而“拒绝”则表示没有成功完成。

>   每个期约只要状态切换为兑现，就会有一个私有的内部值（value）。类似地， 每个期约只要状态切换为拒绝，就会有一个私有的内部理由（reason）。无论是值还是理由，都是包含原 始值或对象的不可修改的引用。二者都是可选的，而且默认值为 undefined。在期约到达某个落定状态时执行的异步代码始终会收到这个值或理由。

期约的状态是私有的，所以只能在内部进行操作。内部操作在期约的执行器函数中完成。执行 器函数主要有两项职责：初始化期约的异步行为和控制状态的最终转换。其中，控制期约状态的转换是 通过调用它的两个函数参数实现的。这两个函数参数通常都命名为resolve()和 reject()。调用 resolve()会把状态切换为兑现，调用 reject()会把状态切换为拒绝。另外，调用 reject()也会抛出错误

==**注意：执行器函数是同步执行的。这是因为执行器函数是期约的初始化程序**==

### 异常

下面是两种模式下抛出错误的情形

```js
try {
    throw new Error('foo');
}
catch (e) {
    console.log(e); // Error: foo 
}
```

```js
try {
    Promise.reject(new Error('bar'));
} catch (e) {
    console.log('e', e);
}
// UnhandledPromiseRejectionWarning: Error: bar
```

第一个 try/catch 抛出并捕获了错误，第二个 try/catch 抛出错误却没有捕获到。

第二个示例代码中确实是同步创建了一个拒绝的期约实例，而这个实例也抛出了包含拒绝理由 的错误。这里的同步代码之所以没有捕获期约抛出的错误，是因为它没有通过异步模式捕获错误。从这 里就可以看出期约真正的异步特性：**它们是同步对象（在同步执行模式中使用），但也是异步执行模式的媒介。**

拒绝期约的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队 列来处理的。因此，try/catch 块并不能捕获该错误。代码一旦开始以异步模式执行，则唯一与之交互的方式就是使用异步结构——更具体地说，就是期约的方法。

### 期约的实例方法

#### Promise.prototype.then()

then()方法接收最多 两个参数：onResolved 处理程序和 onRejected 处理程序。这两个参数都是可选的，如果提供的话，则会在期约分别进入“兑现”和“拒绝”状态时执行。

```js
function onResolved(id) {
    setTimeout(console.log, 0, id, 'resolved');
}
function onRejected(id) {
    setTimeout(console.log, 0, id, 'rejected');
}
let p1 = new Promise((resolve, reject) => setTimeout(resolve, 3000));
let p2 = new Promise((resolve, reject) => setTimeout(reject, 3000));
p1.then(() => onResolved('p1'), () => onRejected('p1'));
p2.then(() => onResolved('p2'), () => onRejected('p2'));
//（3 秒后） 
// p1 resolved
// p2 rejected
```

**期约只能转换为最终状态一次，所以这两个操作一定是互斥的。**如果想只提供 onRejected 参数，那就要在 onResolved 参数的位置上传入 undefined。

then()方法返回一个新的期约实例,这个新期约实例基于 onResovled 处理程序的返回值构建。换句话说，该处理程序的返回值会通过Promise.resolve()包装来生成新期约。如果没有提供这个处理程序，则 Promise.resolve()就会 包装上一个期约解决之后的值。如果没有显式的返回语句，则 Promise.resolve()会包装默认的返回值 undefined。

```js
let p1 = Promise.resolve('foo');
let p6 = p1.then(() => 'bar');
let p7 = p1.then(() => Promise.resolve('bar'));
setTimeout(console.log, 0, p6); // Promise { 'bar' }
setTimeout(console.log, 0, p7); // Promise { 'bar' }
```

抛出异常会返回拒绝的期约：

```js
let p1 = Promise.resolve('foo');
let p2 = p1.then(() => { throw 'baz'; });
////UnhandledPromiseRejectionWarning: baz
setTimeout(console.log, 0, p2)//Promise { <rejected> 'baz' }
```

注意，返回错误值不会触发上面的拒绝行为，而会把错误对象包装在一个解决的期约中：

```js
let p1 = Promise.resolve('foo');
let p2 = p1.then(() => Error('qux'));
setTimeout(console.log, 0, p2)//Promise { <resolved>: Error: qux }
```

onRejected 处理程序也与之类似：onRejected 处理程序返回的值也会被 Promise.resolve() 包装。

#### Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接收一个参数： onRejected处理程序。事实上，这个方法就是一个语法糖，调用它就相当于调用Promise.prototype.then(null, onRejected)

```js
let p = Promise.reject();
let onRejected = function (e) {
    setTimeout(console.log, 0, 'rejected');
};
// 这两种添加拒绝处理程序的方式是一样的： 
p.then(null, onRejected); // rejected 
p.catch(onRejected);// rejected

```



同样，catch()返回一个新的期约实例

#### Promise.prototype.finally()

Promise.prototype.finally()方法用于给期约添加 onFinally 处理程序，这个处理程序在期 约转换为解决或拒绝状态时都会执行。这个方法可以避免 onResolved 和 onRejected 处理程序中出 现冗余代码。但 onFinally 处理程序没有办法知道期约的状态是解决还是拒绝，所以这个方法主要用于添加清理代码。

```js
let p1 = Promise.resolve();
let p2 = Promise.reject();
let onFinally = function () {
    setTimeout(console.log, 0, 'Finally!')
}
p1.finally(onFinally); // Finally
p2.finally(onFinally); // Finally
```



这个新期约实例不同于 then()或 catch()方式返回的实例。因为 onFinally 被设计为一个状态 无关的方法，所以在大多数情况下它将表现为父期约的传递。对于已解决状态和被拒绝状态都是如此。

### 期约的静态方法

#### Promise.all()

>   Promise.all()静态方法创建的期约会在一组期约全部解决之后再解决。这个静态方法接收一个 可迭代对象，返回一个新期约

+   合成的期约只会在每个包含的期约都解决之后才解决

```js
let p = Promise.all([Promise.resolve(),
new Promise((resolve, reject) => setTimeout(resolve, 1000))]);
setTimeout(console.log, 0, p); // Promise <pending> 
p.then(() => setTimeout(console.log, 0, 'all() resolved!'));
// all() resolved!（大约 1 秒后）
```



+   如果至少有一个包含的期约待定，则合成的期约也会待定。如果有一个包含的期约拒绝，则合成的期约也会拒绝

```js
let p2 = Promise.all([Promise.resolve(), Promise.reject(), Promise.resolve()]);
setTimeout(console.log, 0, p2)// Promise <rejected>
//UnhandledPromiseRejectionWarning: undefined
```



+   如果所有期约都成功解决，则合成期约的解决值就是所有包含期约解决值的数组，按照迭代器顺序

```js
let p = Promise.all([Promise.resolve(3), Promise.resolve(), Promise.resolve(4)]);
p.then((values) => setTimeout(console.log, 0, values)); // [3, undefined, 4]
```



+   如果有期约拒绝，则第一个拒绝的期约会将自己的理由作为合成期约的拒绝理由。之后再拒绝的期 约不会影响最终期约的拒绝理由。不过，这并不影响所有包含期约正常的拒绝操作。合成的期约会静默处理所有包含期约的拒绝操作

```js
let p = Promise.all([Promise.reject(3),
new Promise((resolve, reject) => setTimeout(reject, 1000))]);
p.catch((reason) => setTimeout(console.log, 0, reason)); // 3
```



#### Promise.race()

>   Promise.race()静态方法返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。这个 方法接收一个可迭代对象，返回一个新期约

+   Promise.race()不会对解决或拒绝的期约区别对待。无论是解决还是拒绝，只要是第一个落定的 期约，Promise.race()就会包装其解决值或拒绝理由并返回新期约

```js
// 解决先发生，超时后的拒绝被忽略 
let p1 = Promise.race([Promise.resolve(3), new Promise((resolve, reject) => setTimeout(reject, 1000))]);
setTimeout(console.log, 0, p1); // Promise { 3 }
```

```js
// 拒绝先发生，超时后的解决被忽略 
let p2 = Promise.race([Promise.reject(4), new Promise((resolve, reject) => setTimeout(resolve, 1000))]);
setTimeout(console.log, 0, p2); // Promise { <rejected> 4 }
```

如果有一个期约拒绝，只要它是第一个落定的，就会成为拒绝合成期约的理由。之后再拒绝的期约 不会影响最终期约的拒绝理由。不过，这并不影响所有包含期约正常的拒绝操作。与 Promise.all()类似，合成的期约会静默处理所有包含期约的拒绝操作

### 期约连锁

顾名思义，期约连锁即一个期约连着一个期约。期约连锁的作用在于将一系列链式依赖的异步任务串行起来，构成一个大的串行化异步任务。期约连锁的链式结构可以显著的解决地狱回调的窘境。

```js
function delayedResolve(str) {
    return new Promise((resolve, reject) => {
        console.log(str);
        setTimeout(resolve, 1000);
    });
}
delayedResolve('p1 executor')
    .then(() => delayedResolve('p2 executor'))
    .then(() => delayedResolve('p3 executor'))
    .then(() => delayedResolve('p4 executor'))
//p1 executor
//p2 executor
//p3 executor
//p4 executor
```

#### 合成

对于一个串联期约，当它的串联节点较多时，我们可以通过类似函数合成的方式将其合并。

假定三个处理函数

```js
function add_1(x){
    return 1+x
}
function add_2(x){
    return 2+x
}
function add_3(x) {
    return 3 + x
}

```

正常链式调用写法

```js
Promise.resolve(0).then(add_1).then(add_2).then(add_3).then(console.log)//6
```

使用reduce写法

```js
[add_1, add_2, add_3, console.log].reduce((promise, fn) => promise.then(fn), Promise.resolve(0))//6
```

提炼封装

```js
function compose(...fns) {
    return (x) => fns.reduce((promise, fn) => promise.then(fn), Promise.resolve(x))
}
compose(add_1, add_2, add_3, console.log)(0)//6
```

## async/await

异步函数 async/await 是 ES8 规范新增的特性，旨在解决利用异步结构组织代码的问题，这个特性可以让同步代码能够异步执行。

### async

>   async 关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法上

为什么说这个特性可以让同步代码能够异步执行呢？

使用 async 关键字可以让函数具有异步特征，但总体上其代码仍然是同步求值的。而在参数或闭 包方面，异步函数仍然具有普通 JavaScript 函数的正常行为。

如果使用 return 关键字返回了值（如果没有 return 则会返回 undefined），这 个值会被 Promise.resolve()包装成一个期约对象。

+   **同步求值**

如下面这个例子，foo()函数仍然会 在后面的指令之前被求值

```js
async function foo() {
    console.log(1);
}
foo();
console.log(2);
//1
//2
```

+   **异步特征**

```js
async function foo() {
    console.log(1);
    return 3
}
foo().then(console.log)
console.log(2);
//1
//2
//3
```

与在期约处理程序中一样，在异步函数中抛出错误会返回拒绝的期约

```js
async function foo() {
    console.log(1); 
    throw 3;
}
// 给返回的期约添加一个拒绝处理程序 
foo().catch(console.log);
console.log(2);
//1
//2
//3
```

但是，拒绝期约的错误不会被异步函数捕获

```js
async function foo() {
    console.log(1);
    Promise.reject(3);
}
foo().catch(console.log);
console.log(2);
//1
//2
//UnhandledPromiseRejectionWarning: 3
```

### await

>   因为异步函数主要针对不会马上完成的任务，所以自然需要一种暂停和恢复执行的能力。使用await 关键字可以暂停异步函数代码的执行，等待期约解决。

```js
async function foo() {
    let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3));
    console.log(await p);
}
foo();
//3
```

**await 关键字会暂停执行异步函数后面的代码，让出 JavaScript 运行时的执行线程。**这个行 为与生成器函数中的 yield 关键字是一样的。await 关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行。

等待会抛出错误的同步操作，会返回拒绝的期约

```js
async function foo() {
    console.log(1);
    await (() => { throw 3; })();
}
// 给返回的期约添加一个拒绝处理程序 
foo().catch(console.log);
console.log(2);
//1
//2
//3
```

## 使用场景

这里我们需要使用上面假定的四个接口

**示例1**

先看一个“愚蠢”的调用

```js
async function main_1() {
    let clock = new Clock()
    clock.start()

    let resA = await getA()
    const dataA = resA.data
    console.log('dataA', dataA)
    clock.duration()

    const A = dataA.resp;
    let resB = await getB(A)
    console.log('resB', resB.data)
    clock.duration()

    let resX = await getX()
    const dataX = resX.data
    console.log('dataX', dataX)
    clock.duration()

    const X = dataX.resp;
    let resY = await getY(X)
    console.log('resY', resY.data)
    clock.duration()

    clock.end()
}

main_1()
//clock start
//dataA { resp: 3 }
//now duration  5.142
//resB { resp: 6 }
//now duration  5.022
//dataX { resp: 2 }
//now duration  5.029
//resY { resp: 6 }
//now duration  5.01
//clock end
//end duration 20.204
```

上面这个例子将两组请求放在同一个函数main_1中访问，所有的异步任务都是串行的，哪怕AB与XY两组接口之间没有关系。计时器计时20.204秒，

程序运行

![image-20211206195807778](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211207094819.png)

**示例2**

改写一下,将其改为两个并行任务

```js
async function main_2() {
    let clock_1 = new Clock('clock_1'), clock_2 = new Clock('clock_2');

    clock_1.start()
    getA().then(resA => {
        const dataA = resA.data
        console.log('dataA', dataA)
        const A = dataA.resp;
        return A
    })
        .then(getB)
        .then(resB => {
            console.log('resB', resB.data)
            clock_1.end()
        })

    clock_2.start()
    getX().then(resX => {
        const dataX = resX.data
        console.log('dataX', dataX)
        const X = dataX.resp;
        return X
    })
        .then(getY)
        .then(resY => {
            console.log('resY', resY.data)
            clock_2.end()
        })

}

main_2()
// clock_1 start
// clock_2 start
// dataX { resp: 8 }
// dataA { resp: 2 }
// resY { resp: 24 }
// clock_2 end
// clock_2 end duration 10.113s
// resB { resp: 4 }
// clock_1 end
// clock_1 end duration 10.394s
```

计时器计时10.394s,程序运行

![image-20211207095137643](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211207095137.png)

很明显，示例2相较于示例1，节省了2个接口调用的时间，程序请求A,B接口的同时也在向X,Y接口发起请求。

当然，也可也以讲究一点，将AB与XY这两组不相关的调用拆开

```js
function loadAB() {
    let clock_1 = new Clock('clock_1');

    clock_1.start()
    getA().then(resA => {
        const dataA = resA.data
        console.log('dataA', dataA)
        const A = dataA.resp;
        return A
    })
        .then(getB)
        .then(resB => {
            console.log('resB', resB.data)
            clock_1.end()
        })

}
function loadXY() {
    let clock_2 = new Clock('clock_2');
    clock_2.start()
    getX().then(resX => {
        const dataX = resX.data
        console.log('dataX', dataX)
        const X = dataX.resp;
        return X
    })
        .then(getY)
        .then(resY => {
            console.log('resY', resY.data)
            clock_2.end()
        })
}
async function main_2() {
    loadAB();
    loadXY()
}

main_2()
// clock_1 start
// clock_2 start
// dataA { resp: 4 }
// dataX { resp: 9 }
// resB { resp: 8 }
// clock_1 end
// clock_1 end duration 10.235s
// resY { resp: 27 }
// clock_2 end
// clock_2 end duration 10.089s
```

当然，也可以继续使用async/await关键字

```js
async function loadAB() {
    let clock_1 = new Clock('clock_1');
    clock_1.start()
    let dataA = (await getA()).data
    console.log('dataA', dataA)
    const A = dataA.resp;
    let resB = (await getB(A)).data
    console.log('resB', resB)
    clock_1.end()

}
async function loadXY() {
    let clock_2 = new Clock('clock_2');
    clock_2.start()
    let dataX = (await getX()).data
    console.log('dataX', dataX)
    const X = dataX.resp;
    let resY = (await getY(X)).data
    console.log('resY', resY)
    clock_2.end()
}

async function main_3() {
    loadAB()
    loadXY()
}

main_3()
// clock_1 start
// clock_2 start
// dataX { resp: 6 }
// dataA { resp: 2 }
// resY { resp: 18 }
// clock_2 end
// clock_2 end duration 10.092s
// resB { resp: 4 }
// clock_1 end
// clock_1 end duration 10.233s
```



上面这三段代码效果大致是一样的

**示例3**

使用Promise.all()

```js
async function loadAB() {
    let clock_1 = new Clock('clock_1');
    clock_1.start()
    return getA().then(resA => {
        const dataA = resA.data
        console.log('dataA', dataA)
        const A = dataA.resp;
        return A
    })
        .then(getB)
        .then(resB => {
            console.log('resB', resB.data)
            clock_1.end()
            return resB.data
        })

}
async function loadXY() {
    let clock_2 = new Clock('clock_2');
    clock_2.start()
    return getX().then(resX => {
        const dataX = resX.data
        console.log('dataX', dataX)
        const X = dataX.resp;
        return X
    })
        .then(getY)
        .then(resY => {
            console.log('resY', resY.data)
            clock_2.end()
            return resY.data
        })
}

async function main_3() {
    let clock = new Clock()
    clock.start()
    Promise.all([loadAB(), loadXY()]).then(res => {
        console.log('B', res[0])
        console.log('Y', res[1])
        clock.end()
    })
}

main_3()

// clock start
// clock_1 start
// clock_2 start
// dataA { resp: 7 }
// dataX { resp: 1 }
// resB { resp: 14 }
// clock_1 end
// clock_1 end duration 10.217s
// resY { resp: 3 }
// clock_2 end
// clock_2 end duration 10.162s
// B { resp: 14 }
// Y { resp: 3 }
// clock end
// clock end duration 10.303s
```



**在一定情况下尽可能使用async/await异步函数**，因为Promise的异常捕捉是以运行时的错误处理逻辑保证的，JS引擎会为此尽量保存调用栈，这会带来额外的开销。

## 并发协作

并发是指两个或多个事件链随时间发展交替执行，以至于从更高的层次来看，就像是同时 在运行（尽管在任意时刻只处理一个事件）。

并发协作是一种并发合作方式。它的目的在于将一个长期运行的任务分割为多个子任务，JS引擎每次执行一个子任务，并将下一个子任务插到任务队列末尾，使得其它并发任务可以有机会执行。这一方式有点类似于轮转调度机制，它可以解决JS执行一个长时间的运算时，页面无法交互响应的问题

示例

```js
var res = []; // response(..)从Ajax调用中取得结果数组 
function response(data) {
    // 添加到已有的res数组 
    res = res.concat( // 创建一个新的变换数组把所有data值加倍 
        data.map(function (val) {
            return val * 2;
        }));
}
// ajax(..)是某个库中提供的某个Ajax函数 
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

修改,异步地 批处理这些结果。每次处理之后返回事件循环，让其他等待事件有机会运行。

```js
var res = []; // response(..)从Ajax调用中取得结果数组 
function response(data) { // 一次处理1000个
    var chunk = data.splice(0, 1000);
    // 添加到已有的res组 
    res = res.concat( // 创建一个新的数组把chunk中所有值加倍 
        chunk.map(function (val) {
            return val * 2;
        }));
    if (data.length > 0) { // 异步调度下一次批处理 
        setTimeout(function () {
            response(data);
        }, 0);
    }
}
// ajax(..)是某个库中提供的某个Ajax函数 
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

把数据集合放在最多包含 1000 条项目的块中。这样，每一子任务运行时间会很短，事件循环队列的交替运行会提高网页 的响应

# 深入

## Rendering Engine与事件循环

浏览器是多进程的，但与前端开发者关联最紧密的，应该是其中的**Rendering Engine**，渲染引擎，或者说浏览器内核，当然，也可以叫它渲染进程，Renderer进程。

Renderer进程是多线程的，它的主要线程有这些

+   GUI渲染线程
+   JS引擎线程
+   事件触发线程
+   定时触发器线程
+   异步http请求线程

**GUI渲染线程** 

GUI渲染线程是用来进行页面绘制的，HTML解析与DOM树构建就是靠它完成的，需要注意的是**==GUI渲染线程与JS引擎线程是互斥的==**

**JS引擎线程**

也叫JS内核，它维护一个执行栈，执行JS脚本并对事件队列中的事件进行处理，一个Tab页（renderer进程）中无论什么时候都只有一个JS线程在运行（单线程）。但，正如前面所说的，**==GUI渲染线程与JS引擎线程是互斥的==**，当JS引擎执行时GUI线程会被挂起，也就是说，如果JS引擎长时间执行，就会导致页面渲染阻塞，无法响应。这也是上文**并发协作**的基本思路与原理，即让JS引擎线程与GUI渲染线程交替运行。当然，最好的办法是尽量避免CPU密集型计算，将工作交给后端:grin:

**事件触发线程**

事件触发线程是浏览器用来控制事件循环的，它管理一个消息队列message queue，当JS引擎执行遇到setTimeOut，异步请求或者一些其它异步任务时，会将对应的任务移交给对应线程，当事件被触发时或者异步任务有了结果，如定时器计时完毕，异步请求成功，事件触发线程会将对应事件的事件处理程序，如回调等添加到消息队列末尾。事件触发线程会在JS引擎执行栈为空时将消息队列头部任务添加进执行栈。

**定时触发器线程**

setInterval与setTimeout所在线程，进行计时。

**异步http请求线程**

XMLHttpRequest请求执行时的线程，检测到状态变更时，如果有回调函数，异步线程就产生状态变更事件，由事件触发线程将这个回调再放入消息队列中，由JavaScript引擎执行。

==**事件循环队列，消息队列，事件队列，ES6之前的任务队列以及ES6之后的宏任务队列应该都指的是同一个**==



**工作原理：**

JS引擎从上到下顺序执行JS脚本

+   同步任务依次压入执行栈执行
+   异步任务由对应线程执行
+   异步任务有结果时，事件触发线程将其压入消息队列
+   JS引擎执行栈中任务执行完毕，执行任务队列中的所有微任务并清空任务队列
+   GUI渲染线程进行渲染
+   如果消息队列不为空，读取消息队列中队首的宏任务，并执行所有对应的压入任务队列中的微任务。

**优先级** 

**==任务队列>消息队列==**

## 任务队列 job queue

在 ES6 中，有一个新的概念建立在事件循环队列之上，叫作任务队列（job queue）。这个 概念带来的最大影响可能是 Promise 的异步特性。因为Promise的回调便是由任务队列来执行。它是挂在事件循环队列的每个 tick 之后 的一个队列。在事件循环的每个 tick 中，可能出现的异步动作不会导致一个完整的新事件添加到事件循环队列中，而会在当前 tick 的任务队列末尾添加一个项目（一个任务）

## Promise

在前面，我们分别学习了期约的日常使用和它的基石任务队列。现在，我们来了解一下有关于Promise的其它内容。

### 类型检查

生成Promise的场景非常多，有的场景我们明确知晓它就是会生成一个Promise示例，但对于一些我们了解甚少的场景，我们可能难以把握。

使用 p instanceof Promise ,的确有一定的检查效用，但是当Promise实例来自于其他浏览器窗口（iframe等)，这个浏览 器窗口自己的 Promise 可能和当前窗口 /frame 的不同，这样的检查就无法识别 Promise实例。



**thenable**

这是检查一个对象是否是Promise实例的关键。任何具有 then(..) 方法的对象和函数，我们认为，任何这样的值就是 Promise 一致的 thenable。

表现出来，就是这样

```js
if ( 
    p !== null && (
		typeof p === "object" || typeof p === "function"
	) &&
    typeof p.then === "function"
) { 
	// 假定这是一个thenable!
}else{
    // 不是thenable
}
```

看起来很粗犷，但实际就是这样。因此这也会导致一定的问题，也就是具有then的对象可能会被错误识别。

对于Promise.resolve，我们知道这个静态方法能够包装任何非期约值，包括错误对象，并将其转换为解决的期约，而对于期约值，它呈幂等特性。

```js
let t = {
    a: 1
}
let p = Promise.resolve(t)
console.log(p)//Promise { { a: 1 } }
```

```js
let p = new Promise((resolve, reject) => { resolve(t) })
console.log(p)//Promise { { a: 1 } }
```

上面两个是等效的。我们再来试一下

```js
let o = {
    then() {
        console.log('not promise')
    }
}
let p = new Promise((resolve, reject) => { resolve(o) })
console.log(p)
//Promise { <pending> }
//not promise
```

```js
let p = Promise.resolve(o)
console.log(p)
//Promise { <pending> }
//not promise
```

显然，对象o被误解了。

```js
let o = {
    then(func) {
        func()
        console.log('not promise')
    }
}
let p = Promise.resolve(o)
p.then(r => console.log('p.then()'))
//not promise
//p.then()
```

上面这个例子就更清晰了，对象o被resolve包装，由于o具有then，被认为是个期约，由于幂等的关系，对象o被原封不动的返回，调用p.then()就相当于调用了o.then()，注意，这里还有一个细节，输出结果是

```js
//not promise
//p.then()
```

而非

```js
//p.then()
//not promise
```



假如我们改下名字

```js
let o = {
    test() {
        console.log('not promise')
    }
}
let p = Promise.resolve(o)
console.log(p)//Promise { { test: [Function: test] } }
```

看，又正常了。

## 非重入特性

对于期约而言，当期约进入落定状态时，与该状态相关的处理程序仅仅会被排期，而非立即执行。跟在添加这个处 理程序的代码之后的同步代码一定会在处理程序之前先执行。这个特性由 JavaScript运行时保证，被称为“非重入”（non-reentrancy） 特性。

```js
// 创建解决的期约 
let p = Promise.resolve();
// 添加解决处理程序
p.then(() => console.log('onResolved handler'));
console.log('then() returns');
// then() returns
// onResolved handler
```

这个例子中，在一个解决期约上调用 then()会把 onResolved 处理程序推进消息队列。但这个 处理程序在当前线程上的同步代码执行完成前不会执行。

非重入适用于 onResolved/onRejected 处理程序、catch()处理程序和 finally()处理程序。也就是这些程序内部的代码会被认为是异步代码。

## 停止和恢复

>    JavaScript运行时在碰 到 await 关键字时，会记录在哪里暂停执行。等到 await 右边的值可用了，JavaScript运行时会向消息 队列中推送一个任务，这个任务会恢复异步函数的执行。因此，即使 await 后面跟着一个立即可用的值，函数的其余部分也会被异步求值。

```js
async function foo() {
    console.log(2);
    await null;
    console.log(4);
}
console.log(1);
foo();
console.log(3);
// 1
// 2
// 3
// 4
```



## 异步迭代

当异步执行与迭代器协议相遇的时候，它们会碰撞出极其绚烂的火花——异步迭代

下面这个类同时包含异步与同步迭代

```js
class Emitter {
    constructor(max) {
        this.max = max;
        this.syncIdx = 0;
        this.asyncIdx = 0;
    }
    *[Symbol.iterator]() {
        while (this.syncIdx < this.max) {
            yield this.syncIdx++;
        }
    }
    async *[Symbol.asyncIterator]() {
        while (this.asyncIdx < this.max) {
            yield this.asyncIdx++
        }
    }
    // *[Symbol.asyncIterator]() {
    //     while (this.asyncIdx < this.max) {
    //         yield new Promise((resolve) => resolve(this.asyncIdx++));
    //     }
    // }
}
const emitter = new Emitter(5);
function syncCount() {
    const syncCounter = emitter[Symbol.iterator]();
    for (const x of syncCounter) {
        console.log(x);
    }
}
async function asyncCount() {
    const asyncCounter = emitter[Symbol.asyncIterator]();
    for await (const x of asyncCounter) {
        console.log(x);
    }
}
syncCount();
// 0
// 1
// 2
// 3
// 4
asyncCount();
// 0
// 1
// 2
// 3
// 4
```

那么异步迭代有什么用呢？——有序。

>   迭代器返回的期约往往会在不确定的时间解决，比如各种请求，而且它们返回的顺序是乱的。异步迭代器 会尽可能模拟同步迭代器，包括每次迭代时代码的按顺序执行。为此，异步迭代器会维护一个回调队列，以保证早期值的迭代器处理程序总是会在处理晚期值之前完成，即使后面的值早于之前的值解决。

我们修改一下上面的例子

```js
/*
 * @Description: 
 * @Author: anqing.liang
 * @Date: 2021-11-05 14:30:30
 * @LastEditTime: 2022-01-10 17:55:43
 * @LastEditors: anqing.liang
 */
const moment = require("moment");
class Emitter {
    constructor(max) {
        this.max = max;
        this.syncIdx = 0;
        this.asyncIdx = 0;
        this.asyncQueue = [
            /**
             * {
             *      task:1,
             *      done:true|false,
             *      start:
             *      end:
             *      time:
             * }
             */
        ];
    }
    async *[Symbol.asyncIterator]() {
        while (this.asyncIdx < this.max) {
            yield new Promise((resolve) => {
                const time = Math.floor(Math.random() * 10000)
                this.asyncQueue.push({
                    task: this.asyncIdx,
                    done: false,
                    start: moment().format('hh:mm:ss'),
                    time
                })
                console.log('task:', this.asyncQueue)
                setTimeout(() => {
                    this.asyncQueue[this.asyncIdx].done = true
                    this.asyncQueue[this.asyncIdx].end = moment().format('hh:mm:ss')
                    resolve(this.asyncIdx++);
                }, time);
            });
        }
    }
}
const emitter = new Emitter(5);
async function asyncCount() {
    const asyncCounter = emitter[Symbol.asyncIterator]();
    for await (const x of asyncCounter) {
        console.log(x);
    }
}
asyncCount();

```

输出

```js
task: [{ task: 0, done: false, start: '05:57:52', time: 7684 }]
0
task: [
    { task: 0, done: true, start: '05:57:52', time: 7684, end: '05:58:00' },
    { task: 1, done: false, start: '05:58:00', time: 1847 }
]
1
task: [
    { task: 0, done: true, start: '05:57:52', time: 7684, end: '05:58:00' },
    { task: 1, done: true, start: '05:58:00', time: 1847, end: '05:58:02' },
    { task: 2, done: false, start: '05:58:02', time: 3291 }
]
2
task: [
    { task: 0, done: true, start: '05:57:52', time: 7684, end: '05:58:00' },
    { task: 1, done: true, start: '05:58:00', time: 1847, end: '05:58:02' },
    { task: 2, done: true, start: '05:58:02', time: 3291, end: '05:58:05' },
    { task: 3, done: false, start: '05:58:05', time: 5291 }
]
3
task: [
    { task: 0, done: true, start: '05:57:52', time: 7684, end: '05:58:00' },
    { task: 1, done: true, start: '05:58:00', time: 1847, end: '05:58:02' },
    { task: 2, done: true, start: '05:58:02', time: 3291, end: '05:58:05' },
    { task: 3, done: true, start: '05:58:05', time: 5291, end: '05:58:10' },
    { task: 4, done: false, start: '05:58:11', time: 9254 }
]
4
```

一目了然

**而如果期约被拒绝，那么被拒绝的期约会强制退出迭代器**

