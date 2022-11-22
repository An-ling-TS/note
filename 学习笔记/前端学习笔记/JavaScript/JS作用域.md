# JS作用域

[toc]

***这篇文章知识点引用自红宝书《JavaScript高级程序设计》（第四版） 《你不知道的JavaScript》（上卷）***

​	之所以采用第四版的红宝书，是因为第四版相比第三版增加了ES6的内容

​	简而言之，这是一篇**阅读摘要**或者**学习笔记**,大部分内容均引自**原文**

阅读此篇，最好：

+   了解一定编译原理
+   了解TypeScript

**什么是作用域？**

> 就是能够储存变量当中的值，并且能在之后对这个 值进行访问或修改

JS中作用域有：全局作用域、函数作用域。没有块作用域的概念。ECMAScript 6(简称ES6)中新增了块级作用域，使用let声明的变量只能在块级作用域里访问，有“暂时性死区”的特性（也就是说声明前不可用）。

**块作用域由 { } 包括，if语句和for语句里面的{ }也属于块作用域。**



## 变量

提到作用域，就不得不提到变量与JavaScript中声明变量的三个关键字

>ECMAScript 变量是松散类型的，意思是变量可以用于保存任何类型的数据。每个变量只不过是一 个用于保存任意值的命名占位符。有 3 个关键字可以声明变量：var、const 和 let。其中，var 在 ECMAScript 的所有版本中都可以使用，而 const 和 let 只能在 ECMAScript 6 及更晚的版本中使用。

### var

> var message;

上述定义了一个名为message的变量，它的值为 **undefined**

#### var声明作用域

> 使用 var 操作符定义的变量会成为包含它的函数的局部变量。比如，使用 var 在一个函数内部定义一个变量，就意味着该变量将在函数退出时被销毁

```js
function test() {
    var message = "hi"; // 局部变量
}
test();
console.log(message); // 出错
```

报错

```js
ReferenceError: message is not defined
    at Object.<anonymous> (d:\vscode\temp\demo.js:38:13)
    at Module._compile (internal/modules/cjs/loader.js:1063:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
    at Module.load (internal/modules/cjs/loader.js:928:32)
    at Function.Module._load (internal/modules/cjs/loader.js:769:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12)
    at internal/main/run_main_module.js:17:47
```

当然，如果我们使用TypeScript，就能直接显式的在书写代码的时候就获得报错信息

![image-20210916103154287](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916103201.png)

这里，message 变量是在函数内部使用 var 定义的。函数叫 test()，调用它会创建这个变量并给 它赋值。调用之后变量随即被销毁，因此示例中的最后一行会导致错误。不过，在函数内定义变量时省 略 var 操作符，可以创建一个全局变量

```js
function test() {
    message = "hi"; // 全局变量
}
test();
console.log(message); // "hi"
```

去掉之前的 var 操作符之后，message 就变成了全局变量。只要调用一次函数 test()，就会定义 这个变量，并且可以在函数外部访问到。

但是在严格模式下，如果像这样给未声明的变量赋值，则会导致抛出 ReferenceError。

但是这种写法在TypeScript中是不被允许的

![image-20210916103815929](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916103815.png)

#### var声明提升

使用 var 时，下面的代码不会报错。这是因为使用这个关键字声明的变量会自动提升到函数作用域 顶部：

```js
function foo() {
 console.log(age);
 var age = 26;
}
foo(); // undefined
```

因为 ECMAScript 运行时把它看成等价于如下代码：

```js
function foo() {
 var age;
 console.log(age);
 age = 26;
}
foo(); // undefined
```

这就是所谓的“提升”（hoist），也就是把所有变量声明都拉到函数作用域的顶部。

但是，这种写法在TypeScript中依然是不被允许的

![image-20210916104648125](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916104648.png)

此外，反复多次 使用 var 声明同一个变量也没有问题：

```js
function foo() {
 var age = 16;
 var age = 26;
 var age = 36;
 console.log(age);
}
foo(); // 36
```

### let

> let 跟 var 的作用差不多，但有着非常重要的区别。最明显的区别是，let 声明的范围是块作用域， 而 var 声明的范围是函数作用域。这也是 JavaScript 中的新概念。块 级作用域由最近的一对包含花括号{}界定。换句话说，if 块、while 块、function 块，甚至连单独 的块也是 let 声明变量的作用域。

```js
if (true) {
    var name = 'Matt';
    console.log(name); // Matt
}
console.log(name); // Matt 
if (true) {
    let age = 26;
    console.log(age); // 26
}
console.log(age); // ReferenceError: age is not defined
```

> age 变量之所以不能在 if 块外部被引用，是因为它的作用域仅限于该块内部。块作用域 是函数作用域的子集，因此适用于 var 的作用域限制同样也适用于 let。

当然，这种可以被检测的错误在TypeScript中也会得到提示

![image-20210916105206406](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916105206.png)

let 也不允许同一个块作用域中出现冗余声明。这样会导致报错：

```js
var name;
var name;
let age;
let age;
//SyntaxError: Identifier 'age' has already been declared
```

TypeScript中：

![image-20210916105449075](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916105449.png)

> JavaScript 引擎会记录用于变量声明的标识符及其所在的块作用域，因此嵌套使用相同的标 识符不会报错，而这是因为同一个块中没有重复声明

```js
var name = 'Nicholas';
console.log(name); // 'Nicholas'
if (true) {
 var name = 'Matt';
 console.log(name); // 'Matt'
}
let age = 30;
console.log(age); // 30
if (true) {
 let age = 26;
 console.log(age); // 26
} 
```

> 对声明冗余报错不会因混用 let 和 var 而受影响。这两个关键字声明的并不是不同类型的变量， 它们只是指出变量在相关作用域如何存在

```js
var name1;
let name1; // SyntaxError
let age1;
var age1; // SyntaxError
```

#### 暂时性死区

> let 与 var 的另一个重要的区别，就是 let 声明的变量不会在作用域中被提升

```js
// name 会被提升
console.log(name); // undefined
var name = 'Matt';
// age 不会被提升
console.log(age); //  Cannot access 'age' before initialization
let age = 26;
```

> 在解析代码时，JavaScript 引擎也会注意出现在块后面的 let 声明，只不过在此之前不能以任何方 式来引用未声明的变量。在 let 声明之前的执行瞬间被称为“暂时性死区”（temporal dead zone），在此 阶段引用任何后面才声明的变量都会抛出 ReferenceError。

当然，在TypeScript中都不支持就是了

![image-20210916110527694](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916110527.png)

#### 全局声明

> 与 var 关键字不同，使用 let 在全局作用域中声明的变量不会成为 window 对象的属性（var 声 明的变量则会）

```html
<script>
        var name = 'Matt';
        console.log(window.name); // 'Matt'
        let age = 26;
        console.log(window.age); // undefined
</script>
```

不过，let 声明仍然是在全局作用域中发生的，相应变量会在页面的生命周期内存续。因此，为了 避免 SyntaxError，必须确保页面不会重复声明同一个变量。

#### 条件声明

> 在使用 var 声明变量时，由于声明会被提升，JavaScript 引擎会自动将多余的声明在作用域顶部合 并为一个声明。因为 let 的作用域是块，所以不可能检查前面是否已经使用 let 声明过同名变量，同 时也就不可能在没有声明的情况下声明它。

```html
<script>
 var name = 'Nicholas';
 let age = 26;
</script>
<script>
 // 假设脚本不确定页面中是否已经声明了同名变量
 // 那它可以假设还没有声明过
 var name = 'Matt';
 // 这里没问题，因为可以被作为一个提升声明来处理
 // 不需要检查之前是否声明过同名变量
 let age = 36;
 // 如果 age 之前声明过，这里会报错
</script>
```

使用 try/catch 语句或 typeof 操作符也不能解决，因为条件块中 let 声明的作用域仅限于该块。

```html
<script>
    let name = 'Nicholas';
    let age = 36;
</script>
<script>
    // 假设脚本不确定页面中是否已经声明了同名变量
    // 那它可以假设还没有声明过
    if (typeof name === 'undefined') {
        let name;
    }
    // name 被限制在 if {} 块的作用域内
    // 因此这个赋值形同全局赋值
    name = 'Matt';
    try {
        console.log(age); // 如果 age 没有声明过，则会报错
    }
    catch (error) {
        let age;
    }
    // age 被限制在 catch {}块的作用域内
    // 因此这个赋值形同全局赋值
    age = 26;
</script>
```

> 对于 let 这个新的 ES6 声明关键字，不能依赖条件声明模式

#### for循环中的let声明

 let 出现之前，for 循环定义的迭代变量会渗透到循环体外部：

```js
for (var i = 0; i < 5; ++i) {
    // 循环逻辑
}
console.log(i); // 5
```

改成使用 let 之后，这个问题就消失了，因为迭代变量的作用域仅限于 for 循环块内部

```js
for (let i = 0; i < 5; ++i) {
    // 循环逻辑
}
console.log(i);// i is not defined
```

TypeScript中：

![image-20210916111629235](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916111629.png)

在使用 var 的时候，最常见的问题就是对迭代变量的奇特声明和修改：

```js
for (var i = 0; i < 5; ++i) {
 setTimeout(() => console.log(i), 0)
} 
//5
//5
//5
//5
//5
```

 实际输出 5、5、5、5、5 

之所以会这样，是因为在退出循环时，迭代变量保存的是导致循环退出的值：5。在之后执行超时 逻辑时，所有的 i 都是同一个变量，因而输出的都是同一个最终值。

而更进一步的原理，就需要去了解JavaScript的事件循环机制了

而在使用 let 声明迭代变量时，JavaScript 引擎在后台会为每个迭代循环声明一个新的迭代变量。 每个 setTimeout 引用的都是不同的变量实例，所以 console.log 输出的是我们期望的值，也就是循 环执行过程中每个迭代变量的值。

```js
for (let i = 0; i < 5; ++i) {
 setTimeout(() => console.log(i), 0)
} 
//0
//1
//2
//3
//4
```

这种每次迭代声明一个独立变量实例的行为适用于所有风格的 for 循环，包括 for-in 和 for-of 循环。

### const

> const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且 尝试修改 const 声明的变量会导致运行时错误。

```js
// const 也不允许重复声明
const name1 = 'Matt';
const name1 = 'Nicholas'; 
```

```js
// const 声明的作用域也是块
const name2 = 'Matt';
if (true) {
    const name2 = 'Nicholas';
}
console.log(name2);//Matt
```

> const 声明的限制只适用于它指向的变量的引用。换句话说，如果 const 变量引用的是一个对象， 那么修改这个对象内部的属性并不违反 const 的限制。

```js
const person = {};
person.name = 'Matt'; // ok
```

> JavaScript 引擎会为 for 循环中的 let 声明分别创建独立的变量实例，虽然 const 变量跟 let 变 量很相似，但是不能用 const 来声明迭代变量（因为迭代变量会自增）：

```js
for (const i = 0; i < 10; ++i) {}//Assignment to constant variable.
```

TypeScript中则会直接飘红

![image-20210916113220857](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210916113220.png)

不过，如果你只想用 const 声明一个不会被修改的 for 循环变量，那也是可以的。也就是说，每 次迭代只是创建一个新变量。这对 for-of 和 for-in 循环特别有意义：

```js
let i = 0;
for (const j = 7; i < 5; ++i) {
 console.log(j);
}
// 7, 7, 7, 7, 7
for (const key in {a: 1, b: 2}) {
 console.log(key);
}
// a, b
for (const value of [1,2,3,4,5]) {
 console.log(value);
}
// 1, 2, 3, 4, 5 
```

# 作用域链

想要了解作用域链首先得了解执行上下文，在《JavaScript高级程序设计》（第四版）中有如下一段：

> ​	执行上下文（以下简称“上下文”）的概念在 JavaScript 中是颇为重要的。变量或函数的上下文决定 了它们可以访问哪些数据，以及它们的行为。**每个上下文都有一个关联的变量对象**（variable object）， 而这个上下文中定义的所有变量和函数都存在于这个对象上。虽然无法通过代码访问变量对象，但后台 处理数据会用到它。
>
> ​	全局上下文是最外层的上下文。根据 ECMAScript 实现的宿主环境，表示全局上下文的对象可能不一 样。在浏览器中，全局上下文就是我们常说的 window 对象（第 12 章会详细介绍），因此所有通过 var 定 义的全局变量和函数都会成为 window 对象的属性和方法。使用 let 和 const 的顶级声明不会定义在全局上下文中，但在作用域链解析上效果是一样的。上下文在其所有代码都执行完毕后会被销毁，包括定义 在它上面的所有变量和函数（全局上下文在应用程序退出前才会被销毁，比如关闭网页或退出浏览器）。
>
> ​	 **每个函数调用都有自己的上下文**。当代码执行流进入函数时，函数的上下文被推到一个上下文栈上。 在函数执行完之后，上下文栈会弹出该函数上下文，将控制权返还给之前的执行上下文。ECMAScript 程序的执行流就是通过这个上下文栈进行控制的。
>
> ​	上下文中的代码在执行的时候，会创建变量对象的一个**作用域链**（scope chain）。这个作用域链决定 了各级上下文中的代码在访问变量和函数时的顺序。**代码正在执行的上下文的变量对象始终位于作用域 链的最前端**。如果上下文是函数，则其活动对象（activation object）用作变量对象。活动对象最初只有 一个定义变量：arguments。（全局上下文中没有这个变量。）作用域链中的下一个变量对象来自包含上 下文，再下一个对象来自再下一个包含上下文。以此类推直至全局上下文；全局上下文的变量对象始终 是作用域链的最后一个变量对象。 
>
> ​	代码执行时的标识符解析是通过沿作用域链逐级搜索标识符名称完成的。搜索过程始终从作用域链 的最前端开始，然后逐级往后，直到找到标识符。（如果没有找到标识符，那么通常会报错。）

当然，在《你不知道的JavaScript》上卷中的描述相比上面的可能更容易理解

> 当一个块或函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。因此，在当前作用 域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量，或抵达最外层的作用域（也就是全局作用域）为止。

再来看看书中的例子

```js
var color = "blue";
function changeColor() {
    console.log(color)//blue
    if (color === "blue") {
        color = "red";
    } else {
        color = "blue";
    }
}
changeColor();
```

函数 changeColor()的作用域链包含两个对象：一个是它自己的变量对象（就 是定义 arguments 对象的那个），另一个是全局上下文的变量对象。这个函数内部之所以能够访问变量 color，就是因为可以在作用域链中找到它

使用let和const也是一样的

```js
let color = "blue";
const color_b = 'red'
function changeColor() {
    console.log(color)//blue
    console.log(color_b)//red
}
changeColor();
```

如果我们在函数内部重新声明一个同名变量呢

```js
let color = "blue";
const color_b = 'red'
function changeColor() {
    let color = 'yellow'
    console.log(color)//yellow
    const color_b = 'black'
    console.log(color_b)//black
}
changeColor();
console.log(color)//black
console.log(color_b)//blue
```

> 搜索过程始终从作用域链 的最前端开始，然后逐级往后，直到找到标识符。

在上面的这个例子中，changeColor()在执行时，其上下文便是在**作用域链 的最前端**

此外，局部作用域中定义的变量可用于在局部上下文中替换全局变量。看一看下面这个例子：

```js
var color = "blue";
function changeColor() {
 let anotherColor = "red";
 function swapColors() {
 let tempColor = anotherColor;
 anotherColor = color;
 color = tempColor;
 // 这里可以访问 color、anotherColor 和 tempColor
 }
 // 这里可以访问 color 和 anotherColor，但访问不到 tempColor
 swapColors();
}
// 这里只能访问 color
changeColor();
```

以上代码涉及 3 个上下文：全局上下文、changeColor()的局部上下文和 swapColors()的局部 上下文。全局上下文中有一个变量 color 和一个函数 changeColor()。changeColor()的局部上下文中 有一个变量 anotherColor 和一个函数 swapColors()，但在这里可以访问全局上下文中的变量 color。 swapColors()的局部上下文中有一个变量 tempColor，只能在这个上下文中访问到。全局上下文和 changeColor()的局部上下文都无法访问到 tempColor。而在 swapColors()中则可以访问另外两个 上下文中的变量，因为它们都是父上下文。

![img](http://www.ituring.com.cn/figures/2020/JavaScriptWebDeve4th/009.png)

图中的矩形表示不同的上下文。**内部上下文可以通过作用域链访问外部上下文中的一切，但外 部上下文无法访问内部上下文中的任何东西。**上下文之间的连接是线性的、有序的。**每个上下文都可以 到上一级上下文中去搜索变量和函数，但任何上下文都不能到下一级上下文中去搜索。**swapColors() 局部上下文的作用域链中有 3 个对象：swapColors()的变量对象、changeColor()的变量对象和全局 变量对象。swapColors()的局部上下文首先从自己的变量对象开始搜索变量和函数，搜不到就去搜索 上一级变量对象。changeColor()上下文的作用域链中只有 2 个对象：它自己的变量对象和全局变量 对象。因此，它不能访问 swapColors()的上下文。

> **函数参数被认为是当前上下文中的变量，因此也跟上下文中的其他变量遵循相同的 访问规则。**

​	**==遍历嵌套作用域链的规则很简单：引擎从当前的执行作用域开始查找变量，如果找不到， 就向上一级继续查找。当抵达最外层的全局作用域时，无论找到还是没找到，查找过程都会停止。==**

## 把作用域链比喻成一个建筑

![image-20211020103150524](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211020103157.png)

​	这个建筑代表程序中的嵌套作用域链。第一层楼代表当前的执行作用域，也就是你所处的 位置。建筑的顶层代表全局作用域。
​	[LHS 和 RHS](#LHS与RHS) 引用都会在当前楼层进行查找，如果没有找到，就会坐电梯前往上一层楼， 如果还是没有找到就继续向上，以此类推。一旦抵达顶层（全局作用域），可能找到了你所需的变量，也可能没找到，但无论如何查找过程都将停止。

## 标识符查找

上面的几个例子都可以通过标识符查找来解释

> 当在特定上下文中为读取或写入而引用一个标识符时，必须通过搜索确定这个标识符表示什么。搜 索开始于作用域链前端，以给定的名称搜索对应的标识符。如果在局部上下文中找到该标识符，则搜索 停止，变量确定；如果没有找到变量名，则继续沿作用域链搜索。（注意，作用域链中的对象也有一个 原型链，因此搜索可能涉及每个对象的原型链。）这个过程一直持续到搜索至全局上下文的变量对象。 如果仍然没有找到标识符，则说明其未声明。

```js
var color = 'blue';
function getColor() {
 return color;
}
console.log(getColor()); // 'blue' 
```

​	调用函数 getColor()时会引用变量 color。为确定 color 的值会进行两步搜索。 第一步，搜索 getColor()的变量对象，查找名为 color 的标识符。结果没找到，于是继续搜索下一 个变量对象（来自全局上下文），然后就找到了名为 color 的标识符。因为全局变量对象上有 color 的定义，所以搜索结束。 对这个搜索过程而言，**引用局部变量会让搜索自动停止**，而不继续搜索下一级变量对象。也就是说， **如果局部上下文中有一个同名的标识符，那就不能在该上下文中引用父上下文中的同名标识符**

```js
var color = 'blue';
function getColor() {
 let color = 'red';
 return color;
}
console.log(getColor()); // 'red'
```

上面这个例子，getColor()执行时，返回了color，它会优先在当前的上下文中查找，引用局部变量color，返回'red'

使用块级作用域声明并不会改变搜索流程，但可以给词法层级添加额外的层次：

```js
var color = 'blue';
function getColor() {
 let color = 'red';
 {
 let color = 'green';
 return color;
 }
}
console.log(getColor()); // 'green' 
```

​	在这个修改后的例子中，getColor()内部声明了一个名为 color 的局部变量。在调用这个函数时， 变量会被声明。在执行到函数返回语句时，代码引用了变量 color。于是开始在局部上下文中搜索这个 标识符，结果找到了值为'green'的变量 color。因为变量已找到，搜索随即停止，所以就使用这个局 部变量。这意味着函数会返回'green'。在局部变量 color 声明之后的任何代码都无法访问全局变量 color，除非使用完全限定的写法 window.color。

> 注意 标识符查找并非没有代价。访问局部变量比访问全局变量要快，因为不用切换作用 域。不过，JavaScript 引擎在优化标识符查找上做了很多工作，将来这个差异可能就微不 足道了。

## 作用域链增强

> ​	虽然执行上下文主要有全局上下文和函数上下文两种（eval()调用内部存在第三种上下文），但有 其他方式来增强作用域链。某些语句会导致在作用域链前端临时添加一个上下文，这个上下文在代码执 行后会被删除。通常在两种情况下会出现这个现象，即代码执行到下面任意一种情况时：
>
> + try/catch 语句的 catch 块
> + with语句
>
> ​    这两种情况下，都会在作用域链前端添加一个变量对象。对 with 语句来说，会向**==作用域链前端==**添 加指定的对象；对 catch 语句而言，则会创建一个**==新的变量对象==**，这个变量对象会包含要抛出的错误 对象的声明。

```js
function buildUrl() {
 let qs = "?debug=true";
 with(location){
 let url = href + qs;
 }
 return url;
} 
```

这里，with 语句将 location 对象作为上下文，因此 location 会被添加到作用域链前端。 buildUrl()函数中定义了一个变量 qs。当 with 语句中的代码引用变量 href 时，实际上引用的是 location.href，也就是自己变量对象的属性。在引用 qs 时，引用的则是定义在 buildUrl()中的那 个变量，它定义在函数上下文的变量对象上。而在 with 语句中使用 var 声明的变量 url 会成为函数 上下文的一部分，可以作为函数的值被返回；但像这里使用 let 声明的变量 url，因为被限制在块级作 用域，所以在 with 块之外没有定义。

作用域链增强的更深入学习在下文的[欺骗词法](#欺骗词法)中

# 深入

上面的内容是作用域的表现形式，但如果想要更加深入，还需要一些其它的知识点。

## LHS与RHS

对于一个简单的例子

> var a = 2;

编译器会执行两个步骤

> 1. 遇到 var a，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的 集合中。如果是，编译器会忽略该声明，继续进行编译；否则它会要求作用域在当前作 用域的集合中声明一个新的变量，并命名为 a。
> 2. 接下来编译器会为引擎生成运行时所需的代码，这些代码被用来处理 a = 2 这个赋值 操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫作 a 的变量。如果是，引擎就会使用这个变量；如果否，引擎会继续查找该变量

在编译器第二步生成的代码中，引擎执行这些代码，会通过查找变量 a 来判断它是 否已声明过。查找的过程由作用域进行协助。

引擎会对 ‘=’ 左边的变量a进行LHS查询，查询a本身是否存在声明以及a本身这个容器（或者说地址？）



而如果是这样的

> var a=2;
>
> var b=a;

当执行 b=a 时，对于a，引擎不再关心这个容器（或者说它的地址），而是需要a的值，这时候对a进行的是RHS查询



> 当变量出现在赋值操作的左侧时进行 LHS 查询，出现在右侧时进行RHS 查询。

> 讲得更准确一点，RHS 查询与简单地查找某个变量的值别无二致，而 LHS 查询则是试图 找到变量的容器本身，从而可以对其赋值。从这个角度说，RHS 并不是真正意义上的“赋值操作的右侧”，更准确地说是“非左侧”。

> LHS 和 RHS 的含义是“赋值操作的左侧或右侧”并不一定意味着就是“= 赋值操作符的左侧或右侧”。赋值操作还有其他几种形式，因此在概念上最 好将其理解为“赋值操作的目标是谁（LHS）”以及“谁是赋值操作的源头（RHS）”。

分析一下这个例子

```js
function foo(a) { 
    console.log( a ); // 2
}
foo( 2 );
```

最后一行 foo(..) 函数的调用需要对 foo 进行 RHS 引用，意味着“去找到 foo 的值，并把 它给我”。并且 (..) 意味着 foo 的值需要被执行

代码中隐式的 a＝2 操作可能很容易被忽略掉。这个操作发生在 2 被当作参数传递给 foo(..) 函数时，2 会被分配给参数 a。为了给参数 a（隐式地）分配值，需要进行一次LHS 查询。

这里还有对 a 进行的 RHS 引用，并且将得到的值传给了 console.log(..)。console. log(..) 本身也需要一个引用才能执行，因此会对 console 对象进行 RHS 查询，并且检查得到的值中是否有一个叫作 log 的方法。假设在 log(..) 函数的原生实现中它可以接受参数，在将 2 赋 值给其中第一个（也许叫作 arg1）参数之前，这个参数需要进行 LHS 引用查询。

> 你可能会倾向于将函数声明 function foo(a) {... 概念化为普通的变量声明 和赋值，比如 var foo、foo ＝ function(a) {...。如果这样理解的话，这 个函数声明将需要进行 LHS 查询。 然而还有一个重要的细微差别，编译器可以在代码生成的同时处理声明和值 的定义，比如在引擎执行代码时，并不会有线程专门用来将一个函数值“分 配给”foo。因此，将函数声明理解成前面讨论的 LHS 查询和赋值的形式并不合适。

### 异常

​	在变量还没有声明（在任何作用域中都无法找到该变量）的情况下，LHS和RHS查询的行为是不一样的。

分析下面的代码

```js
function foo(a) { 
    console.log( a + b ); 
    b = a;
}
foo( 2 );
```

​	第一次对 b 进行 RHS 查询时是无法找到该变量的。也就是说，这是一个“未声明”的变 量，因为在任何相关的作用域中都无法找到它。
​	如果 RHS 查询在所有嵌套的作用域中遍寻不到所需的变量，引擎就会抛出 ReferenceError 异常。值得注意的是，ReferenceError 是非常重要的异常类型。
​	相较之下，当引擎执行 LHS 查询时，如果在顶层（全局作用域）中也无法找到目标变量， 全局作用域中就会创建一个具有该名称的变量，并将其返还给引擎，前提是程序运行在非“严格模式”下。

> “不，这个变量之前并不存在，但是我很热心地帮你创建了一个。”

> ​	ES5 中引入了“严格模式”。同正常模式，或者说宽松 / 懒惰模式相比，严格模式在行为上 有很多不同。其中一个不同的行为是严格模式禁止自动或隐式地创建全局变量。因此，在 严格模式中 LHS 查询失败时，并不会创建并返回一个全局变量，引擎会抛出同 RHS 查询 失败时类似的 ReferenceError 异常。 
>
> ​	接下来，如果 RHS 查询找到了一个变量，但是你尝试对这个变量的值进行不合理的操作， 比如试图对一个非函数类型的值进行函数调用，或着引用 null 或 undefined 类型的值中的 属性，那么引擎会抛出另外一种类型的异常，叫作 TypeError。 
>
> ​	**==ReferenceError 同作用域判别失败相关，而 TypeError 则代表作用域判别成功了，但是对结果的操作是非法或不合理的。==**

## 词法作用域

作用域有两种主要工作模型

+ 词法作用域
+ 动态作用域

这里主要学习的是词法作用域。词法作用域是最为普遍的，大多数编程语言采用的都是词法作用域

### 词法阶段

> ​	大部分标准语言编译器的第一个工作阶段叫作词法化（也叫单词化）。**==词法化的过程会对源代码中的字符进行检查，如果是有状态的解析过程，还会赋予单词语义。==**

​	词法作用域就是定义在词法阶段的作用域。换句话说，**==词法作用域是由你在写 代码时将变量和块作用域写在哪里来决定的==**，因此当词法分析器处理代码时会保持作用域不变（大部分情况下是这样的）

> ​	有一些欺骗词法作用域的方法，这些方法在词法分析器处理过后依 然可以修改作用域，但是这种机制可能有点难以理解。事实上，让词法作用域根据词法关系保持书写时的自然关系不变，是一个非常好的最佳实践。

分析如下代码

```js
function foo(a) { 
    var b = a * 2;
	function bar(c) { 
        console.log( a, b, c );
	} 
    bar( b * 3 );
}
foo( 2 ); // 2, 4, 12
```

在这个例子中有三个逐级嵌套的作用域。为了帮助理解，可以将它们想象成几个逐级包含 的气泡。

![image-20211020113200943](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211020113201.png)

1号	包含着整个全局作用域，其中只有一个标识符：foo。 

2号	包含着 foo 所创建的作用域，其中有三个标识符：a、bar 和 b。

3号	包含着 bar 所创建的作用域，其中只有一个标识符：c。

​	作用域气泡由其对应的作用域块代码写在哪里决定，它们是逐级包含的。

​	作用域气泡的结构和互相之间的位置关系给引擎提供了足够的位置信息，引擎用这些信息 来查找标识符的位置。

​	在上一个代码片段中，引擎执行 console.log(..) 声明，并查找 a、b 和 c 三个变量的引 用。它首先从最内部的作用域，也就是 bar(..) 函数的作用域气泡开始查找。引擎无法在 这里找到 a，因此会去上一级到所嵌套的 foo(..) 的作用域中继续查找。在这里找到了 a， 因此引擎使用了这个引用。对 b 来讲也是一样的。而对 c 来说，引擎在 bar(..) 中就找到了它。

​	==作用域查找会在找到第一个匹配的标识符时停止。==在多层的嵌套作用域中可以定义同名的 标识符，这叫作“==遮蔽效应==”（内部的标识符“遮蔽”了外部的标识符）。抛开遮蔽效应， 作用域查找始终从运行时所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见
第一个匹配的标识符为止。

>   ​	全局变量会自动成为全局对象（比如浏览器中的 window 对象）的属性，因此 可以不直接通过全局对象的词法名称，而是间接地通过对全局对象属性的引 用来对其进行访问。
>
>   ​	**window.a**
>   ​	通过这种技术可以访问那些被同名变量所遮蔽的全局变量。但非全局的变量如果被遮蔽了，无论如何都无法被访问到。

**==无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处 的位置决定。==**

>   词法作用域查找只会查找一级标识符，比如 a、b 和 c。如果代码中引用了 foo.bar.baz， 词法作用域查找只会试图查找 foo 标识符，找到这个变量后，对象属性访问规则会分别接管对 bar 和 baz 属性的访问。

### 欺骗词法

JavaScript 中有两种机制来实现这个目的

+ eval
+ with

也即上文中的作用域链增强

但要注意的是**==欺骗词法作用域会导致性能 下降。==**

#### eval

​	JavaScript 中的 eval(..) 函数可以接受一个字符串为参数，并将其中的内容视为好像在书 写时就存在于程序中这个位置的代码。换句话说，可以在你写的代码中用程序生成代码并运行，就好像代码是写在那个位置的一样。

分析如下代码

```js
function foo(str, a) { 
    eval( str ); // 欺骗！ 
    console.log( a, b );
}
var b = 2;
foo( "var b = 3;", 1 ); // 1, 3
```

​		eval(..) 调用中的 "var b = 3;" 这段代码会被当作本来就在那里一样来处理。由于那段代 码声明了一个新的变量 b，因此它对已经存在的 foo(..) 的词法作用域进行了修改。事实 上，和前面提到的原理一样，这段代码实际上在 foo(..) 内部创建了一个变量 b，并遮蔽 了外部（全局）作用域中的同名变量。
​		当 console.log(..) 被执行时，会在 foo(..) 的内部同时找到 a 和 b，但是永远也无法找到外部的 b。因此会输出“1, 3”而不是正常情况下会输出的“1, 2”。

#### with

​	**with 通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象 本身。**

```js
var obj = { a: 1, b: 2, c: 3};
// 单调乏味的重复 "obj" 
obj.a = 2; 
obj.b = 3; 
obj.c = 4;
// 简单的快捷方式 
with (obj) { 
    a = 3; 
    b = 4; 
    c = 5;
}
```



再来看一个例子

```js
function foo(obj) {
    with (obj) {
        a = 2;
    }
}
var o1 = {
    a: 3
};
var o2 = {
    b: 3
};
foo(o1); console.log(o1.a); // 2
foo(o2); console.log(o2.a); // undefined
console.log(a); // 2
```

​	在最后一句console.log(a)中，我们打印的结果时2，而不是猜想中的undefined，这是因为在全局作用域上新增了一个a的声明

​	这个例子中创建了 o1 和 o2 两个对象。其中一个具有 a 属性，另外一个没有。foo(..) 函 数接受一个 obj 参数，该参数是一个对象引用，并对这个对象引用执行了 with(obj) {..}。 在 with 块内部，我们写的代码看起来只是对变量 a 进行简单的词法引用，实际上就是一个
LHS 引用，并将 2 赋值给它。

​	当我们将 o1 传递进去，a＝2 赋值操作找到了 o1.a 并将 2 赋值给它，这在后面的 console. log(o1.a) 中可以体现。而当 o2 传递进去，o2 并没有 a 属性，因此不会创建这个属性，o2.a 保持 undefined。

​	但是实际上 a = 2 赋值操作创建了一个全局的变量 a。

>   ​	with 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对 象的属性也会被处理为定义在这个作用域中的词法标识符。

​	eval(..) 函数如果接受了含有一个或多个声明的代码，就会修改其所处的词法作用域，而 with 声明实际上是根据你传递给它的对象**凭空创建了一个全新的词法作用域。**

​		当我们传递 o1 给 with 时，with 所声明的作用域是 o1，而这个作用域中含 有一个同 o1.a 属性相符的标识符。但当我们将 o2 作为作用域时，其中并没有 a 标识符，因此进行了正常的 LHS 标识符查找

​		o2 的作用域、foo(..) 的作用域和全局作用域中都没有找到标识符 a，因此当 a＝2 执行 时，自动创建了一个全局变量（因为是非严格模式）。

>   **eval(..) 和 with 会被严格模式所影响（限 制）。with 被完全禁止，而在保留核心功能的前提下，间接或非安全地使用eval(..) 也被禁止了。**

**==大量的使用eval和with，会严重影响性能==**

## 函数作用域

​		**函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复 用（事实上在嵌套的作用域中也可以使用）。**这种设计方案是非常有用的，能充分利用JavaScript 变量可以根据需要改变值类型的“动态”特性。

### 隐藏内部实现

>   ​	对函数的传统认知就是先声明一个函数，然后再向里面添加代码。但反过来想也可以带来 一些启示：从所写的代码中挑选出一个任意的片段，然后用函数声明对它进行包装，实际上就是把这些代码“隐藏”起来了。
>
>   ​	实际的结果就是在这个代码片段的周围创建了一个作用域气泡，也就是说这段代码中的任 何声明（变量或函数）都将绑定在这个新创建的包装函数的作用域中，而不是先前所在的 作用域中。换句话说，可以把变量和函数包裹在一个函数的作用域中，然后用这个作用域来“隐藏”它们。

**最小授权或最小暴露原则。**

>   这个原则是指在软件设计中，应该最小限度地暴露必 要内容，而将其他内容都“隐藏”起来，比如某个模块或对象的API 设计。

```js
function doSomething(a) {
    b = a + doSomethingElse(a * 2);
    console.log(b * 3);
}
function doSomethingElse(a) {
    return a - 1;
}
var b;
doSomething(2); // 15
```

​	在这个代码片段中，变量 b 和函数 doSomethingElse(..) 应该是 doSomething(..) 内部具体 实现的“私有”内容。给予外部作用域对 b 和 doSomethingElse(..) 的“访问权限”不仅 没有必要，而且可能是“危险”的，因为它们可能被有意或无意地以非预期的方式使用， 从而导致超出了 doSomething(..) 的适用条件。更“合理”的设计会将这些私有的具体内容隐藏在 doSomething(..) 内部

```js
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }
    var b; b = a + doSomethingElse(a * 2); console.log(b * 3);
}
doSomething(2); // 15
```

​	现在，b 和 doSomethingElse(..) 都无法从外部被访问，而只能被 doSomething(..) 所控制。 功能性和最终效果都没有受影响，但是设计上将具体内容私有化了，设计良好的软件都会依此进行实现。

#### 规避冲突

>   ​	“隐藏”作用域中的变量和函数所带来的另一个好处，是可以避免同名标识符之间的冲突， 两个标识符可能具有相同的名字但用途却不一样，无意间可能造成命名冲突。冲突会导致变量的值被意外覆盖。

**全局命名空间**

​	变量冲突的一个典型例子存在于全局作用域中。当程序中加载了多个第三方库时，如果它 们没有妥善地将内部私有的函数或变量隐藏起来，就会很容易引发冲突。

​	这些库通常会在全局作用域中声明一个名字足够独特的变量，通常是一个对象。这个对象 被用作库的命名空间，所有需要暴露给外界的功能都会成为这个对象（命名空间）的属性，而不是将自己的标识符暴漏在顶级的词法作用域中。

举个例子，比如

```js
var MyReallyCoolLibrary = {
    awesome: "stuff", doSomething: function () { // ...
    }, doAnotherThing: function () {
// ...
    }
}
```

**模块管理**

>   ​	另外一种避免冲突的办法和现代的模块机制很接近，就是从众多模块管理器中挑选一个来 使用。使用这些工具，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显式地导入到另外一个特定的作用域中。
>
>   ​	显而易见，这些工具并没有能够违反词法作用域规则的“神奇”功能。它们只是利用作用 域的规则强制所有标识符都不能注入到共享作用域中，而是保持在私有、无冲突的作用域中，这样可以有效规避掉所有的意外冲突。

### 函数作用域

​	**使用包装函数将隐藏内部变量和函数声明虽然有效，但并不理想**

比如

```js
var a = 2; 
function foo() { 
    var a = 3; 
    console.log(a); // 3
} 
foo(); 
console.log(a); // 2
```

​	会导致一些额外的问题。首先， 必须声明一个具名函数 foo()，意味着 foo 这个名称本身“污染”了所在作用域（在这个 例子中是全局作用域）。其次，必须显式地通过函数名（foo()）调用这个函数才能运行其中的代码。

 JavaScript提供了另一种解决方案

```js
var a = 2;
(function foo() { 
    var a = 3;
    console.log(a); // 3
})(); 
console.log(a); // 2
```

​	包装函数的声明以 (function... 而不仅是以 function... 开始。尽管看上去这并不 是一个很显眼的细节，但实际上却是非常重要的区别。函数会被当作函数表达式而不是一个标准的函数声明来处理。

>   ​	==区分函数声明和表达式最简单的方法是看 function 关键字出现在声明中的位 置（不仅仅是一行代码，而是整个声明中的位置）。如果 function 是声明中的第一个词，那么就是一个函数声明，否则就是一个函数表达式。==

​	**函数声明和函数表达式之间最重要的区别是它们的名称标识符将会绑定在何处。** 

​	比较一下前面两个代码片段。第一个片段中 foo 被绑定在所在作用域中，可以直接通过 foo() 来调用它。第二个片段中 foo 被绑定在函数表达式自身的函数中而不是所在作用域中。
​	换句话说，(function foo(){ .. }) 作为函数表达式意味着 foo 只能在 .. 所代表的位置中 被访问，外部作用域则不行。foo 变量名被隐藏在自身中意味着不会非必要地污染外部作用域。

#### 匿名和具名

函数表达式最常用的场景当属作为回调函数了

```js
setTimeout(function () {
    console.log("I waited 1 second!");
}, 1000);
```

​		**==这叫作匿名函数表达式，因为 function().. 没有名称标识符。函数表达式可以是匿名的， 而函数声明则不可以省略函数名——在 JavaScript 的语法中这是非法的。==**

​	匿名函数表达式书写起来简单快捷，很多库和工具也倾向鼓励使用这种风格的代码。但是 它也有几个缺点需要考虑。

+   匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。
+   如果没有函数名，当函数需要引用自身时只能使用已经过期的 arguments.callee 引用， 比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身。
+   匿名函数省略了对于代码可读性 / 可理解性很重要的函数名。一个描述性的名称可以让 代码不言自明

​    行内函数表达式非常强大且有用——匿名和具名之间的区别并不会对这点有任何影响。给函 数表达式指定一个函数名可以有效解决以上问题。始终给函数表达式命名是一个最佳实践：

```js
setTimeout(function timeoutHandler() {
    console.log("I waited 1 second!");
}, 1000);
```

#### IIFE 立即执行函数表达式

```js
var a = 2;
(function foo() { 
    var a = 3;
    console.log(a); // 3
})(); 
console.log(a); // 2
```

​	由于函数被包含在一对 ( ) 括号内部，因此成为了一个表达式，通过在末尾加上另外一个 ( ) 可以立即执行这个函数，比如 (function foo(){ .. })()。第一个 ( ) 将函数变成表达式，第二个 ( ) 执行了这个函数。

​	除了传统的(function foo(){...})()形式，还有另一种写法，(function foo(){...}()) 。第一种形式中函数表达式被包含在 ( ) 中，然后在后面用另一个 () 括 号来调用。第二种形式中用来调用的 () 括号被移进了用来包装的 ( ) 括号中。

​	IIFE 的另一个非常普遍的进阶用法是把它们当作函数调用并传递参数进去。

```js
var a = 2;
(function IIFE(global) {
    var a = 3;
    console.log(a); // 3 
    console.log(global.a); // 2
})(window);
console.log(a); // 2
```

​	将 window 对象的引用传递进去，但将参数命名为 global，因此在代码风格上对全局 对象的引用变得比引用一个没有“全局”字样的变量更加清晰。当然可以从外部作用域传 递任何你需要的东西，并将变量命名为任何你觉得合适的名字。这对于改进代码风格是非常有帮助的。

​	IIFE 还有一种变化的用途是倒置代码的运行顺序，将需要运行的函数放在第二位，在 IIFE 执行之后当作参数传递进去。这种模式在UMD（Universal Module Definition）项目中被广泛使用。

```js
var a=2;
(function IIFE(def) {
    def(window);
})(function def(global) {
    var a = 3; 
    console.log(a); // 3 
    console.log( global.a ); // 2
});
```

​		函数表达式 def 定义在片段的第二部分，然后当作参数（这个参数也叫作 def）被传递进 IIFE 函数定义的第一部分中。最后，参数 def（也就是传递进去的函数）被调用，并将window 传入当作 global 参数的值。

## 块作用域

```js
for (var i = 0; i < 2; i++) {
    console.log(i);
}
console.log('end i', i)
//0
//1
//end i 2
```

​	我们在 for 循环的头部直接定义了变量 i，通常是因为只想在 for 循环内部的上下文中使 用 i，而忽略了 i 会被绑定在外部作用域（函数或全局）中的事实。

**==这就是块作用域的用处。变量的声明应该距离使用的地方越近越好，并最大限度地本地 化。==**

```js
var foo = true;
if (foo) {
    var bar = foo * 2;
    console.log(bar);//2
}
console.log(bar)//2
```

​	bar 变量仅在 if 声明的上下文中使用，因此如果能将它声明在 if 块内部中会是一个很有 意义的事情。但是，当使用 var 声明变量时，它写在哪里都是一样的，因为它们最终都会属于外部作用域。

**块作用域是一个用来对之前的最小授权原则进行扩展的工具，将代码从在函数中隐藏信息 扩展为在块中隐藏信息。**

### with

with是块作用域的一个典型例子，用 with 从对象中创建出的作用域仅在 with 声明中而非外 部作用域中有效。

### try/catch

>   JavaScript 的 ES3 规范中规定 try/catch 的 catch 分句会创建一个块作 用域，其中声明的变量仅在 catch 内部有效。

```js
try {
    undefined(); // 执行一个非法操作来强制制造一个异常
} catch (err) {
    console.log(err); // 能够正常执行！
}
console.log(err); // ReferenceError: err not found
```

err 仅存在 catch 分句内部，当试图从别处引用它时会抛出错误。

### let

​	let 关键字可以将变量绑定到所在的任意作用域中（通常是 { .. } 内部）。换句话说，let 为其声明的变量隐式地了所在的块作用域。

```js
var foo = true; 
if (foo) { 
    let bar = foo * 2; 
    bar = something( bar ); 
    console.log( bar );
}
console.log( bar ); // ReferenceError
```

​	用 let 将变量附加在一个已经存在的块作用域上的行为是隐式的。在开发和修改代码的过 程中，如果没有密切关注哪些块作用域中有绑定的变量，并且习惯性地移动这些块或者将其包含在其他的块中，就会导致代码变得混乱。

​	为块作用域显式地创建块可以部分解决这个问题，使变量的附属关系变得更加清晰。通常 来讲，显式的代码优于隐式或一些精巧但不清晰的代码。显式的块作用域风格非常容易书写，并且和其他语言中块作用域的工作原理一致

```js
var foo = true;
if (foo) { 
    { // <-- 显式的快 
        let bar = foo * 2;
        bar = something( bar ); 
        console.log( bar );
	}
}
console.log( bar ); // ReferenceError
```

​	只要声明是有效的，在声明中的任意位置都可以使用 { .. } 括号来为 let 创建一个用于绑 定的块。在这个例子中，我们在 if 声明内部显式地创建了一个块，如果需要对其进行重构，整个块都可以被方便地移动而不会对外部 if 声明的位置和语义产生任何影响。

​	**let循环**

```js
for (let i=0; i<10; i++) { 
    console.log( i );
}
console.log( i ); // ReferenceError
```

·	for 循环头部的 let 不仅将 i 绑定到了 for 循环的块中，事实上它将其重新绑定到了循环 的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。

```js
{ 
    let j; 
    for (j=0; j<10; j++) { 
        let i = j; // 每个迭代重新绑定！ 
        console.log( i );
	}
}
```

### const

​	除了 let 以外，ES6 还引入了 const，同样可以用来创建块作用域变量，但其值是固定的 （常量）。之后任何试图修改值的操作都会引起错误。

## 提升

```js
a = 2;
var a;
console.log(a);//2
```

```js
console.log(a);
var a = 2;//undefined
```

如果说第一个例子的输出许多人可能会清楚，那第二个例子的输出一定会出乎很多人的意料。

但正确的思考思路是，**包括变量和函数在内的所有声明都会在任何代码被执行前首先 被处理。**

JavaScript 会将其看成两个 声明：var a; 和 a = 2;。**第一个定义声明是在编译阶段进行的。第二个赋值声明会被留在原地等待执行阶段。**

因此，第二个代码片段的实际流程是

```js
var a; 
console.log( a );
a = 2;
```

这个过程就好像变量和函数声明从它们在代码中出现的位置被“移动” 到了最上面。这个过程就叫作提升。

**函数声明会被提升，但是函数表达式却不会被提升。**即使是具名的函数表达式，名称标识符在赋值之前也无法在所在作用域中使用

```js
foo(); // 不是 ReferenceError, 而是 TypeError! 
var foo = function bar() { // ...
};
```

### 函数优先

**==函数声明和变量声明都会被提升，但是函数会首先被提升，然后才是变量。==**

```js
foo(); // 1 
var foo;
function foo() {
    console.log(1);
}
foo = function () {
    console.log(2);
};
```

这个例子会输出1，而不是2，尽管var foo在前面

但是却会被引擎理解为这种形式

```js
function foo() { 
    console.log( 1 );
} 
var foo
foo(); // 1
foo = function() { 
    console.log( 2 );
};
```

var foo被视为重复声明，会被忽略掉

出现在后面的函数声明还是可以覆盖前面的

```js
foo(); // 3 
function foo() {
    console.log(1);
}
var foo = function () {
    console.log(2);
};
function foo() {
    console.log(3);
}
```

