# JS闭包

[toc]

**闭包**是JavaScript中的一个重要概念，彻底理解闭包对于学习各种框架实现原理，工具方法优化原理，TS类等有着非常重要的影响。

阅读此篇，最好：

+   熟悉作用域原理
+   熟悉IIFE

## 概念

在《JavaScript高级程序设计》（第三版）第7.2节指出

> ==闭包是指有权访问另一个 函数作用域中的变量的函数。==

创建闭包的常见方式，就是在一个函数内部创建另一个函数

```js
/*
 * @Description: 
 * @Author: anqing.liang
 * @Date: 2021-09-13 10:58:26
 * @LastEditTime: 2021-09-13 11:00:41
 * @LastEditors: anqing.liang
 */
const a = {
    b: 1,
    c: 2,
    d: 3,
}
const b = {
    e: 3,
    d: 4,
}
function createComparisonFunction(propertyName) {

    return function (object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];

        if (value1 < value2) {
            return -1;
        } else if (value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    };
}
const resp = createComparisonFunction('d')
console.log(resp(a, b))//-1
```

在这个例子中，

```js
var value1 = object1[propertyName];
var value2 = object2[propertyName];
```

​	这两行代码是内部函数（一个匿名函数）中的代码，这两行代码访问了外部 函数中的变量 propertyName。即使这个内部函数被返回了，而且是在其他地方被调用了，但它仍然可 以访问变量 propertyName。之所以还能够访问这个变量，是因为内部函数的作用域链中包含 createComparisonFunction()的作用域。

​	要彻底搞清楚其中的细节，必须从理解函数被调用的时候 都会发生什么入手。

​	函数内部的代码在访问变量时，会使用给定的名称从作用域链中查找变量。函数执行完毕后，局部活动对象会被销毁，内存中就只剩下全局作用域。

​	但闭包却有所不同，**在一个函数内部定义的函数会把其包含函数的活动对象添加到自己的作用域链中**。因此，在 createComparisonFunction()函数中，匿名函数的作用域链中实际上包含 createComparisonFunction()的活动对象。

​	闭包在代码中随处可见，只是有时候程序员并没有意识到他正在使用闭包罢了。使用回调函数，实际上就在使用闭包

## 闭包如何工作

​	为什么闭包能够访问其他函数作用域中的变量呢？

​	这其中最重要的一点与作用域链有关，更准确的说是与词法作用域的查找规则有关

​	关于作用域的查找可以看另一篇文章

[JS作用域]: https://blog.csdn.net/qq_42012626/article/details/120328731?spm=1001.2014.3001.5501

```js
function foo() {
    var a = 2;
    function bar() {
        console.log(a);
    } 
    return bar;
}
var baz = foo();
baz(); //2
```

​	函数bar()可以访问到它的上一级foo()的作用域，而在foo()中，我们又将bar进行返回，执行 var baz=foo()，使得baz获得函数bar()的引用，最后调用baz()也就是调用bar()

​	JS的垃圾回收机制会销毁不再使用的作用域，而在这个例子中，bar函数在定义它的词法作用域以外的地方执行，阻止了引擎对foo作用域的回收。由于bar在foo内部声明，它的作用域链中包含foo的作用域，使得foo作用域免于被引擎销毁。

​	当然，从这里也就可以看出闭包的使用对于性能开销还是有影响的。

```js
var fn; function foo() {
    var a = 2;
    function baz() {
        console.log(a);
    }
    fn = baz; // 将 baz 分配给全局变量
}
function bar() {
    fn(); 
}
foo();
bar(); // 2
```

​	这也是一个使用闭包的例子，只是他们传递内部函数的形式不同

>   无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用 域的引用，无论在何处执行这个函数都会使用闭包。

再看一个例子

```js
function wait(message) {
    setTimeout(function timer() {
        console.log(message);
    }, 1000);
}
wait("Hello, closure!");
```

​	内部函数（名为 timer）传递给 setTimeout(..)。timer 具有涵盖 wait(..) 作用域 的闭包，因此还保有对变量 message 的引用。wait(..) 执行 1000 毫秒后，它的内部作用域并不会消失，timer 函数依然保有 wait(..) 作用域的闭包。

​	深入到引擎的内部原理中，内置的工具函数 setTimeout(..) 持有对一个参数(函数)的引用。引擎会调用这个函数，在例子中就是内部的 timer 函数，而词法作用域在这个过程中保持完整。

## 循环和闭包

```js
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, 0);
}
```

​	很多人都知道，这会输出5次6，

```js
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        //console.log(i);
    }, 0);
}
console.log(i)//6
```

​	如果我们在for循环外面打印i 它会输出6，这里的i可以进行RHS查询显然是因为i的声明位置与调用console.log(i)的位置是同级的。for

循环中每一次使用的i都是同一个，for循环结束时保留的是使得循环终止的值，即i=6，这样是为什么延迟函数（异步在同步之后调用）的每次回调都是输出的6

​	上面两个例子与下面这个是等效的，下面这个例子会输出3次3

```js

var i = 1;
setTimeout(function timer() {
    console.log(i);
}, 0);
i++;
setTimeout(function timer() {
    console.log(i);
}, 0);
i++;
setTimeout(function timer() {
    console.log(i);
}, 0);
```

​	想要让它输出1-5，就需要每一次迭代都单独用一个作用域保存i的引用，使用let是个很好的例子

​	for 循环头部的 let 不仅将 i 绑定到了 for 循环的块中，**事实上它将其重新绑定到了循环 的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。**

```js
{ 
    let j; 
    for (j=0; j<10; j++) { 
        let i = j; // 每个迭代重新绑定！
        console.log( i );
	}
}
```

​	从这里，可以产生两个例子

```js
for (let i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, 0);
}
```

```js
for (var i = 1; i <= 5; i++) {
    let j = i; 
    setTimeout(function timer() {
        console.log(j);
    }, 0);
}
```

​	上面这两个例子都会输出1到5，是不是很有意思，这就是闭包的效果。在第二个例子中，相当于每次迭代都有一个块作用域因为延迟函数回调里定义的函数对j的引用而导致这个块作用域暂时无法关闭。

​	通俗点讲，就是for循环的每次迭代都会使用一个不同的作用域去保存i的值，理解了这个，那我们也就可以不使用let自己实现一个输出1-5的例子了

```js
for (var i = 1; i <= 5; i++) {
    function func(j) {
        setTimeout(function timer() {
            console.log(j);
        }, 0);
    }
    func(i)
}
```

​	上面这个例子就会输出1-5，

​	当然，我们也可以使用**IIFE立即执行函数**来为每一次迭代保留一个闭包作用域

```js
for (var i = 1; i <= 5; i++) {
    (function func(j) {
        setTimeout(function timer() {
            console.log(j);
        }, 0);
    })(i);
}

```

​		上面的例子会输出1-5

​		这个例子中，每次迭代都会执行这样一个过程

```js
function func(j){...}
func(i)
```

>   ​	**==在迭代内使用 IIFE 会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的 作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。==**

## 模块

​	模块是利用闭包的显著例子。

```js
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another.join(" ! "));
    }
    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
}
var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

​	上面这种形式被称为模块。最常见的实现模块模式的方法通常被称为模块暴露

​	CoolModule() 返回一个用对象字面量语法 { key: value, ... } 来表示的对象。这 个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐藏且私有的状态。可以将这个对象类型的返回值看作本质上是模块的公共API。

>   ​	从模块中返回一个实际的对象并不是必须的，也可以直接返回一个内部函 数。jQuery 就是一个很好的例子。jQuery 和 $ 标识符就是 jQuery 模块的公 共API，但它们本身都是函数（由于函数也是对象，它们本身也可以拥有属性）。

​	doSomething() 和 doAnother() 函数具有涵盖模块实例内部作用域的闭包（通过调用 CoolModule() 实现）。当通过返回一个含有属性引用的对象的方式来将函数传递到词法作用域外部时，我们已经创造了可以观察和实践闭包的条件。

​	模块模式需要具备两个必要条件:

+   必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块 实例）。
+   封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并 且可以访问或者修改私有的状态。

>   ​	一个具有函数属性的对象本身并不是真正的模块。从方便观察的角度看，一个从函数调用 所返回的，只有数据属性而没有闭包函数的对象并不是真正的模块。

​	当只需要一个实例时,我们可以使用IIFE

```js
var foo = (function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another.join(" ! "));
    }
    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
})();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

​	模块模式另一个简单但强大的变化用法是，命名将要作为公共API 返回的对象

```js
var foo = (function CoolModule(id) {
    function change() {
        // 修改公共 API 
        publicAPI.identify = identify2;
    }
    function identify1() {
        console.log(id);
    }
    function identify2() {
        console.log(id.toUpperCase());
    }
    var publicAPI = {
        change: change,
        identify: identify1
    };
    return publicAPI;
})("foo module");
foo.identify(); // foo module 
foo.change();
foo.identify(); // FOO MODULE
```

​	通过在模块实例的内部保留对公共 API 对象的内部引用，可以从内部对模块实例进行修 改，包括添加或删除方法和属性，以及修改它们的值。

### 基于函数的模块机制

​	大多数模块依赖加载器 / 管理器本质上都是将这种模块定义封装进一个友好的API。这里介绍一些核心概念。

```js
var MyModules = (function Manager() {
    var modules = {};
    function define(name, deps, impl) {
        for (var i = 0; i < deps.length; i++) {
            deps[i] = modules[deps[i]];
        } 
        modules[name] = impl.apply(impl, deps);
    }
    function get(name) {
        return modules[name];
    } 
    return {
        define: define, 
        get: get
    };
})();
```

​	这段代码的核心是 modules[name] = impl.apply(impl, deps)。为了模块的定义引入了包装 函数（可以传入任何依赖），并且将返回值，也就是模块的API，储存在一个根据名字来管理的模块列表中。

​	define函数接收的三个参数中，name是模块名，deps是当前定义的模块的依赖列表，它是一个字符串数组，每一个元素都是所依赖的模块名，impl则是name模块的定义。

​	modules中存放的是已经定义的模块，deps[i] = modules[deps[i]];会根据deps[i]的值，也就是模块名字符串来从modules中获取已定义的模块，并将这些模块存放到deps中。

```js
var MyModules = (function Manager() {
    var modules = {};
    function define(name, deps, impl) {
        for (var i = 0; i < deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }
    function get(name) {
        return modules[name];
    }
    return {
        define: define,
        get: get
    };
})();
MyModules.define("bar", [], function () {
    function hello(who) {
        return "Let me introduce: " + who;
    }
    return {
        hello: hello
    };
});
MyModules.define("foo", ["bar"], function (bar) {
    var hungry = "hippo";
    function awesome() {
        console.log(bar.hello(hungry).toUpperCase());
    }
    return {
        awesome: awesome
    };
});
var bar = MyModules.get("bar");
var foo = MyModules.get("foo");
console.log(bar.hello("hippo")); // Let me introduce: hippo
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

​	"foo" 和 "bar" 模块都是通过一个返回公共API 的函数来定义的。"foo" 甚至接受 "bar" 的 示例作为依赖参数，并能相应地使用它。

### 基于独立文件的模块机制/ES6模块

>   ​	ES6 中为模块增加了一级语法支持。通过模块系统进行加载时，ES6 会将文件当作独立 的模块来处理。每个模块都可以导入其他模块或特定的API 成员，同样也可以导出自己的API 成员。

>   ​	基于函数的模块并不是一个能被稳定识别的模式（编译器无法识别），它们 的API 语义只有在运行时才会被考虑进来。因此可以在运行时修改一个模块的API。
>
>   ​	相比之下，ES6 模块API 更加稳定（API 不会在运行时改变）。由于编辑器知 道这一点，因此可以在（的确也这样做了）编译期检查对导入模块的API 成 员的引用是否真实存在。如果API 引用并不存在，编译器会在运行时抛出一个或多个“早期”错误，而不会像往常一样在运行期采用动态的解决方案。

​	ES6 的模块没有“行内”格式，必须被定义在独立的文件中（一个文件一个模块）。浏览 器或引擎有一个默认的“模块加载器”可以在导入模块时异步地加载模块文件。

bar.mjs

```js
export default function hello(who) {
    return "Let me introduce: " + who;
}
```

foo.mjs

```js
import hello from "./bar.mjs";
var hungry = "hippo";
function awesome() {
    console.log(hello(hungry).toUpperCase()
    );
}
awesome()//LET ME INTRODUCE: HIPPO
```

​	为什么使用 .mjs ? 这是因为我的示例是在node.js下运行的，直接使用ES6模块语法会报错，需要使用mjs扩展，并将 .js 后缀改为 .mjs
