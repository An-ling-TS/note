# CommonJS模块，ES6模块
[toc]
## 学习地址
- [CommonJS模块与ES6模块的区别](https://www.cnblogs.com/unclekeith/p/7679503.html)
- [ES6 module](https://www.cnblogs.com/unclekeith/p/7137047.html)
## CommonJS模块
- **1.对于基本数据类型，属于复制。即会被模块缓存。同时，在另一个模块可以对该模块输出的变量重新赋值。**
```
// b.js
let count = 1
let plusCount = () => {
  count++
}
setTimeout(() => {
  console.log('b.js-1', count)
}, 1000)
module.exports = {
  count,
  plusCount
}

// a.js
let mod = require('./b.js')
console.log('a.js-1', mod.count)
mod.plusCount()
console.log('a.js-2', mod.count)
setTimeout(() => {
    mod.count = 3
    console.log('a.js-3', mod.count)
}, 2000)

node a.js
a.js-1 1
a.js-2 1
b.js-1 2  // 1秒后
a.js-3 3  // 2秒后
```

以上代码可以看出，b模块export的count变量，是一个复制行为。在plusCount方法调用之后，a模块中的count不受影响。同时，可以在b模块中更改a模块中的值。如果希望能够同步代码，可以export出去一个getter。
```
...
// 其他代码相同
module.exports = {
  get count () {
    return count
  },
  plusCount
}

node a.js
a.js-1 1
a.js-2 1
b.js-1 2  // 1秒后
a.js-3 2  // 2秒后， 由于没有定义setter，因此无法对值进行设置。所以还是返回2
```
- **2.对于复杂数据类型，属于浅拷贝。由于两个模块引用的对象指向同一个内存空间，因此对该模块的值做修改时会影响另一个模块。**
```
// b.js
let obj = {
  count: 1
}
let plusCount = () => {
  obj.count++
}
setTimeout(() => {
  console.log('b.js-1', obj.count)
}, 1000)
setTimeout(() => {
  console.log('b.js-2', obj.count)
}, 3000)
module.exports = {
  obj,
  plusCount
}

// a.js
var mod = require('./b.js')
console.log('a.js-1', mod.obj.count)
mod.plusCount()
console.log('a.js-2', mod.obj.count)
setTimeout(() => {
  mod.obj.count = 3
  console.log('a.js-3', mod.obj.count)
}, 2000)

node a.js
a.js-1 1
a.js-2 2
b.js-1 2
a.js-3 3
b.js-2 3
```
当执行a模块时，首先打印obj.count的值为1，然后通过plusCount方法，再次打印时为2。接着在a模块修改count的值为3，此时在b模块的值也为3。

- **3.当使用require命令加载某个模块时，就会运行整个模块的代码。**
- **4.当使用require命令加载同一个模块时，不会再执行该模块，而是取到缓存之中的值。也就是说，==CommonJS模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存==。**
- **5.循环加载时，属于加载时执行。即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。**
```
3, 4, 5可以使用同一个例子说明

// b.js
exports.done = false
let a = require('./a.js')
console.log('b.js-1', a.done)
exports.done = true
console.log('b.js-2', '执行完毕')

// a.js
exports.done = false
let b = require('./b.js')
console.log('a.js-1', b.done)
exports.done = true
console.log('a.js-2', '执行完毕')

// c.js
let a = require('./a.js')
let b = require('./b.js')

console.log('c.js-1', '执行完毕', a.done, b.done)

node c.js
b.js-1 false
b.js-2 执行完毕
a.js-1 true
a.js-2 执行完毕
c.js-1 执行完毕 true true
```
    过程：
    1.在Node.js中执行c模块。此时遇到require关键字，执行a.js中所有代码。
    2.在a模块中exports之后，通过require引入了b模块，执行b模块的代码。
    3.在b模块中exports之后，又require引入了a模块，此时执行a模块的代码。
    4.a模块只执行exports.done = false这条语句。
    5.回到b模块，打印b.js-1, exports, b.js-2。b模块执行完毕。
    6.回到a模块，接着打印a.js-1, exports, b.js-2。a模块执行完毕
    7.回到c模块，接着执行require，需要引入b模块。由于在a模块中已经引入过了，所以直接就可以输出值了。
    8.结束。
**从以上结果和分析过程可以看出，当遇到require命令时，会执行对应的模块代码。当循环引用时，有可能只输出某模块代码的一部分。当引用同一个模块时，不会再次加载，而是获取缓存。**

## ES6模块
### 特性
- **1.es6模块中的值属于【动态只读引用】。只说明一下复杂数据类型。**
- **2.对于只读来说，即不允许修改引入变量的值，import的变量是只读的，不论是基本数据类型还是复杂数据类型。当模块遇到import命令时，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。**
- **3.对于动态来说，原始值发生变化，import加载的值也会发生变化。不论是基本数据类型还是复杂数据类型。**
```
// b.js
export let counter = {
  count: 1
}
setTimeout(() => {
  console.log('b.js-1', counter.count)
}, 1000)

// a.js
import { counter } from './b.js'
counter = {}
console.log('a.js-1', counter)

// Syntax Error: "counter" is read-only
```
虽然不能将counter重新赋值一个新的对象，但是可以给对象添加属性和方法。此时不会报错。这种行为类型与关键字const的用法
```
// a.js
import { counter } from './b.js'
counter.count++
console.log(counter)

// 2
```
- **4.循环加载时，ES6模块是动态引用。只要两个模块之间存在某个引用，代码就能够执行。**
```
// b.js
import {foo} from './a.js';
export function bar() {
  console.log('bar');
  if (Math.random() > 0.5) {
    foo();
  }
}

// a.js
import {bar} from './b.js';
export function foo() {
  console.log('foo');
  bar();
  console.log('执行完毕');
}
foo();

babel-node a.js
foo
bar
执行完毕

// 执行结果也有可能是
foo
bar
foo
bar
执行完毕
执行完毕
```
### export
**==export用于输出模块的对外接口。export命令只要处于模块顶层就可以使用==*
#### 输出形式
```

1. export var obj = { name: 'keith' } // 直接输出

2. var obj = { name: 'keith' }
   export { obj }  // 使用该种形式输出时需要添加大括号
   export obj   // 不添加大括号时会报错，因为我们要输出的是该对象的引用。注意是对该对象的引用，而不是拷贝。这也意味着在该模块改变name属性，会导致另一个模块下name属性的变化，这点与CommonJS不同，CommonJS只是对某个对象的拷贝
   var num = function () { return 123 }
   export { num }  // 正确
   export num // 报错，输出对num的引用，而不是直接输出函数num

3. var obj = { name: 'keith' }
   export { obj as person }  // export命令支持接口的重命名

4  var obj = { name: 'keith' }
   export default obj
   // 输出默认值时不需要添加大括号, export default在一个模块中只能使用一次，但export命令可以使用多次

   // export default的本质：就是将某个变量命名为default
   // export default num
   // 等同于 export { num as default }
   // import Num from './module.js'
   // 等同于 import { defualt as Num } from './module.js'

5  export { num, obj }
   // export命令处于模块顶层的任何位置，都可以将变量输出
   // 不管是用var声明的变量，还是let声明的变量
   // 在执行过程中export命令会被默认放置在整个模块的最底层。
   var num = function () { return 123 };
   let obj = { name: 'keith' }
   // 相当于
   var num = function () { return 123 };
   var obj = { name: 'keith' }
   // ..函数、对象等其他数据类型
   // 被放置在模块最底层
   export { name, obj }
   ```
### import
- **==import命令用于引入其他模块提供的功能接口。与export命令一样，import命令只能用于模块顶层==**
#### 导入形式
```
1. import { num, obj } from './export.js'  // 模块名，可以不添加.js后缀，但需要写配置文件，与Node知识相关
   // import入的接口名字，要与export出的名字对应
   console.log(num()) // 123
   console.log(obj.name) // 'keith'

2. import { obj as person } from './export.js'
   // 可以修改import进来的变量名
   console.log(person.name) // 'keith'

3. import * as $ from './export.js'
   // 使用*用于模块的整体加载，并重命名为$对象.此时可以通过$对象取到export的对外接口
   console.log($.num()) // 123
   console.log($.obj.name) // 'keith'

4. import 'normalize.css'
   // 用于加载整个模块，此时不需要添加变量名

5. let obj = { name: 'keith' }
   export default obj

   import person from './module.js'
   import boy from './module.js'
   // 对应于export default 命令
   // 此时import进来的接口不需要添加大括号
   // 且支持import时的任意命名
   console.log(person.name) // 'keith'
   console.log(boy.name) // 'keith'

6. console.log(obj.name)  // 'keith'
   import { obj } from './export.js'
   // 与export相反，import默认会被提升到模块最顶层
   // 即
   import { obj } from './export.js'
   console.log(obj)
```
### export与import的复合写法
- **==如果在某个模块中引入了其他模块，又导出了该模块，可以采用export和import的复合写法==**
```
1. 使用{}导出模块
export { Hello, World } from './modules'
// 相当于
import { Hello, World } from './moudles'
export { Hello, World }

2. 改写模块的名字
export { Hello as Person } from './modules'
// 相当于
import { Hello as Person } from './modules'
export { Person }

3. 改写默认export default模块的名字
export { default as Person } from './modules'
// 相当于
import Person from './modoules'  // ./modules里的模块是通过export default导出的
export { Person }
```