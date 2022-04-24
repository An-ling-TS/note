# JS对象

[toc]

>   这是一篇关于JavaScript对象与原型的综合性的学习笔记，知识点主要提炼或引用自《JavaScript高级程序设计》（第三版）（第四版）与《你不知道的JavaScript》（上卷）,以及JS源头https://developer.mozilla.org/zh-CN/

## 什么是对象？

>   ECMA-262将对象定义为一组属性的无序集合

用结构一点的话讲，对象近似一张散列表。

## 对象的创建

一般创建对象最简单的有两种方法

+   一是使用Object创建一个实例，再向这个实例添加子属性和方法
+   二是直接使用对象字面量的方式

### 属性定义

如果对于Object.defineProperty()方法有一定的了解，或者看过《JavaScript高级程序设计》，那一定会记得里面提到过这样一段内容

>   ECMA-262 使用一些内部特性来描述属性的特征。这些特性是由为 JavaScript 实现引擎的规范定义 的。因此，开发者不能在 JavaScript 中直接访问这些特性。为了将某个特性标识为内部特性，规范会用两个中括号把特性的名称括起来，比如[[Enumerable]]。

这些特性对于常规的代码工作或许意义不大，但是当你想要去封装或者实现一个工具类的时候，这或许会给你意想不到的帮助。

####  数据属性（数据描述符）

>   数据属性包含一个保存数据值的位置。值会从这个位置读取，也会写入到这个位置。数据属性有 4 个特性描述它们的行为。

+   [[Configurable]]：表示属性是否可以通过 delete 删除并重新定义，是否可以修改它的特 性，以及是否可以把它改为访问器属性。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。非严格模式下对 这个属性调用 delete 没有效果，严格模式下会抛出错误。此外，一个属性被定义为不可配置之后，就 不能再变回可配置的了。再次调用 Object.defineProperty()并修改任何非writable 属性会导致错误
+   [[Enumerable]]：表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对 象上的属性的这个特性都是 true
+   [[Writable]]：表示属性的值是否可以被修改。默认情况下，所有直接定义在对象上的属性的 这个特性都是 true。非严格模式下尝试给这个属性重新赋值会被忽略。在严格模式下，尝试修改只读属性 的值会抛出错误。
+   [[Value]]：包含属性实际的值。这就是前面提到的那个读取和写入属性值的位置。这个特性 的默认值为 undefined

**Object.defineProperty()**

如果对于双向绑定中的数据劫持有一定的学习，那一定了解这个方法。

这个方法接收 3个参数： 要给其添加属性的对象、属性的名称和一个描述符对象。最后一个参数，即描述符对象上的属性可以包
含：configurable、enumerable、writable 和 value，跟相关特性的名称一一对应。

```js
const obj = {};
Object.defineProperty(obj, 'a', {
    value: 1,
    writable: false
});
obj.a = 2;
console.log(obj.a);//1
```

而如果在严格模式下，执行到obj.a=2时就会直接抛出错误

这里还有一个细节

**在调用 Object.defineProperty()时，configurable、enumerable 和 writable 的值如果不指定，则都默认为 false**

```js
const obj = {};
Object.defineProperty(obj, 'a', {
    value: 1,
});
obj.a = 2;
console.log(obj.a);//1
```

#### 访问器属性（存取描述符）

访问器属性包含一个获取（getter）函数和一个设置（setter）函数，但它们并不是必须的。

>   在读取访问器属性时，会调用获取函数，这个函数的责任就是返回一个有效 的值。在写入访问器属性时，会调用设置函数并传入新值，这个函数必须决定对数据做出什么修改。

这一点是非常重要的，**许多前端框架关于数据这一方面的基础实现，都有用到这一点。**

访问器属性同样具有四个特性

+   [[Configurable]]：表示属性是否可以通过 delete 删除并重新定义，是否可以修改它的特 性，以及是否可以把它改为数据属性。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。
+   [[Enumerable]]：表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对 象上的属性的这个特性都是 true。
+   [[Get]]：获取函数，在读取属性时调用。默认值为 undefined。
+   [[Set]]：设置函数，在写入属性时调用。默认值为 undefined。

>   获取函数和设置函数不一定都要定义。只定义获取函数意味着属性是只读的，尝试修改属性会被忽 略。在严格模式下，尝试写入只定义了获取函数的属性会抛出错误。类似地，只有一个设置函数的属性是不能读取的，非严格模式下读取会返回 undefined，严格模式下会抛出错误。

对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性， 如果找到就会返回这个属性的值，如果没有找到名称相同的属性，会遍历可能存在的 [[Prototype]] 链，也就是原型链，如果无论如何都没有找到名称相同的属性，那 [[Get]] 操作会返回值undefined。看到这里，我们就有了一个小小的发现，访问对象属性和访问一个独立的变量时有一点不同，如果引用一个词法作用域中不存在的变量，会抛出ReferenceError 异常，但是我们引用一个不存在的对象属性时却并不会抛出异常，而是返回undefined

先来看一个简单的例子

```js
let book = {
    year_: 2017,
    edition: 1
}
Object.defineProperty(book, "year", {
    get() {
        return this.year_;
    }, set(newValue) {
        if (newValue > 2017) {
            this.year_ = newValue;
            this.edition += newValue - 2017;
        }
    }
});
book.year = 2018;
console.log(book.year_)//2018
console.log(book.edition); // 2
```

再来看一个稍微复杂一点的例子

```js
function Archiver() {
    var temperature = null;
    var archive = [];

    Object.defineProperty(this, 'temperature', {
        get: function () {
            console.log('get!');
            return temperature;
        },
        set: function (value) {
            temperature = value;
            archive.push({ val: temperature });
        }
    });

    this.getArchive = function () { return archive; };
}

var arc = new Archiver();
arc.temperature; // 'get!'
arc.temperature = 11;
arc.temperature = 13;
console.log(arc.getArchive()); // [{ val: 11 }, { val: 13 }]
```

上面的这一个例子实现了属性赋值操作的记录自动保存功能，每一次对temperature进行赋值的时候都会保存这一次的赋值记录。

除了使用Object.defineProperty()重写get与set，我们也可以直接在对象字面量中重写

```js
let obj = {
    get a() {
        console.log('get a ')
        return a * 2
    },
    set a(val) {
        console.log('set a')
        a = val
    }
};
obj.a = 3
//set a
console.log(obj.a)
//get a 
//6
```

**==注意：getter 和 setter 是成对出现的，缺少setter会抛出ReferenceError 异常，缺少getter，读取时会返回undefined，也就是说，单个的getter或者setter无法定义一个对象属性。==**

#### 保持不变

​	通过前面的种种特性，我们可以保持一个对象的一定的不变性。

**常量**

​	结合 writable:false 和 configurable:false 就可以创建一个真正的常量属性（不可修改、 重定义或者删除）

**禁止扩展**

​	禁止一个对象添加新属性并且保留已有属性，使用 Object.prevent Extensions(..)

```js
var myObject = {
    a: 2
};
Object.preventExtensions(myObject);
myObject.b = 3;
console.log(myObject.b); // undefined
```

​	在非严格模式下，创建属性 b 会静默失败。在严格模式下，将会抛出 TypeError 错误。

**密封**

​	Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。所以，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（虽然可以修改属性的值）。

**冻结**

​	Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为writable:false，这样就无法修改它们的值。

​	这个方法是可以应用在对象上的级别最高的不可变性，它会禁止对于对象本身及其任意 直接属性的修改（但是这个对象引用的其他对象是不受影响的）。
​	因此可以“深度冻结”一个对象，具体方法为，首先在这个对象上调用 Object.freeze(..)， 然后遍历它引用的所有对象并在这些对象上调用 Object.freeze(..)。但是这样做有可能会在无意中冻结其他（共享）对象。



除了上面的两种较为简单的创建对象方式，还有许多方式可以创建对象，当然，他们中的大多数都是上面两种方式的二次封装。

### 工厂函数模式

工厂函数可以按照特定接口创 建对象，宛如一条流水线可以不断的生成对象。

```js
function createPerson(name, age, job) {
    let o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function () {
        console.log(this.name);
    };
    return o;
}
let person1 = createPerson("Nicholas", 29, "Software Engineer");
let person2 = createPerson("Greg", 27, "Doctor");
```

createPerson()接收 3个参数，根据这几个参数构建了一个包含 Person 信息的对象。 可以用不同的参数多次调用这个函数，每次都会返回包含 3个属性和 1个方法的对象。

这种工厂模式虽 然可以解决创建多个类似对象的问题，但没有解决对象标识问题（即新创建的对象是什么类型）。也就是说，你知道person1是一个对象，知道它有3个属性一个方法，但也就仅限于此了。

### 构造函数模式

ECMAScript中的构造函数是用于创建特定类型对象的。像 Object 和 Array 这 样的原生构造函数，运行时可以直接在执行环境中使用。当然也可以自定义构造函数，以函数的形式为自己的对象类型定义属性和方法。

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function () {
        console.log(this.name);
    };
}
let person1 = new Person("Nicholas", 29, "Software Engineer");
let person2 = new Person("Greg", 27, "Doctor");
person1.sayName(); // Nicholas
person2.sayName(); // Greg
```

构造函数模式和工厂函数模式区别

+   没有显式地创建对象。
+    属性和方法直接赋值给了 this。
+   没有 return。
+   首字母大写。
+   使用new操作符创建实例

定义自定义构造函数可以确保实例被标识为特定类型。

```js
console.log(person1 instanceof Person)//true
```

创建 Person 的实例，使用 new 操作符。以这种方式调用构造函数会执行这些操作。

+   在内存中创建一个新对象。
+   这个新对象内部的[[Prototype]]特性被赋值为构造函数的 prototype 属性
+   构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）。
+   执行构造函数内部的代码（给新对象添加属性）。
+   如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象

person1 和 person2 分别保存着 Person 的不同实例。这两个对象都有一个 constructor 属性指向 Person

```js
console.log(person1.constructor == Person); // true 
console.log(person2.constructor == Person); // true
```

构造函数不一定要写成函数声明的形式。赋值给变量的函数表达式也可以表示构造函数

```js
let Person = function(name, age, job) { 
    this.name = name; 
    this.age = age; 
    this.job = job;
	this.sayName = function() { 
    	console.log(this.name);
	};
}
```

**问题**

构造函数的主要问题在于，其定义的方法会在每个实例上 都创建一遍。对前面的例子而言，person1 和 person2 都有名为 sayName()的方法，但这两个方 法不是同一个 Function 实例。ECMAScript中的函数是对象，因此每次定义函数时，都会 初始化一个对象。

不同实例上的函数虽然同名却不相等

```js
console.log(person1.sayName == person2.sayName)//false
```



前面的Person构造函数和下面这个是等价的。

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = new Function("console.log(this.name)"); 
}
```



既然person1和person2乃至其它的Person实例的sayName函数都是实现相同的功能，那我们为什么不直接使用一个sayName实例呢？

要解决这个问题，可以把函数定义转移到构造函数外部

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}
function sayName() {
    console.log(this.name);
}
let person1 = new Person("Nicholas", 29, "Software Engineer");
let person2 = new Person("Greg", 27, "Doctor");
person1.sayName(); // Nicholas
person2.sayName(); // Greg
```

sayName()被定义在了构造函数外部。在构造函数内部，sayName 属性等于全局sayName() 函数。因为这一次 sayName 属性中包含的只是一个指向外部函数的指针，所以 person1 和 person2 共享了定义在全局作用域上的 sayName()函数。这样虽然解决了相同逻辑的函数重复定义的问题，但 全局作用域也因此被搞乱了，因为那个函数实际上只能在一个对象上调用。如果这个对象需要多个方法， 那么就要在全局作用域中定义多个函数。这会导致自定义类型引用的代码不能很好地聚集一起。这个新问题可以通过原型模式来解决。

### 原型模式

每个函数都会创建一个 prototype 属性，这个属性是一个对象，包含应该由特定引用类型的实例 共享的属性和方法。实际上，这个对象就是通过调用构造函数创建的对象的原型。使用原型对象的好处 是，在它上面定义的属性和方法可以被对象实例共享。原来在构造函数中直接赋给对象实例的值，可以直接赋值给它们的原型。

```js
function Person() { }
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
    console.log(this.name);
};
let person1 = new Person();
person1.sayName(); // "Nicholas"
let person2 = new Person();
person2.sayName(); // "Nicholas"
console.log(person1.sayName == person2.sayName); // true
```

与构造函数模式不同，使用这种原型模式定义的属性和方法是由所有实例共享的。

>   无论何时，只要创建一个函数，就会按照特定的规则为这个函数创建一个 prototype 属性（指向 原型对象）。默认情况下，所有原型对象自动获得一个名为 constructor 的属性，指回与之关联的构 造函数。对前面的例子而言，Person.prototype.constructor 指向 Person。然后，因构造函数而异，可能会给原型对象添加其他属性和方法。
>
>   在自定义构造函数时，原型对象默认只会获得 constructor 属性，其他的所有方法都继承自 Object。每次调用构造函数创建一个新实例，这个实例的内部[[Prototype]]指针就会被赋值为构 造函数的原型对象。脚本中没有访问这个[[Prototype]]特性的标准方式，但 Firefox、Safari 和 Chrome 会在每个对象上暴露__proto__属性，通过这个属性可以访问对象的原型。在其他实现中，这个特性 完全被隐藏了。关键在于理解这一点：实例与构造函数原型之间有直接的联系，但实例与构造函数之间没有。
