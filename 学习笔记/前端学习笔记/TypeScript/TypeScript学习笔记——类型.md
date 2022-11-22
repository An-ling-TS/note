## 类型

### 类型注解

类型注解使用 `:TypeAnnotation` 语法。类型声明空间中可用的任何内容都可以用作类型注解。另外，JavaScript中的原始类型以及字面量也可以用做类型注解

#### **内联注解**

使用`:{ /*Structure*/ }`语法对任何内容进行注解，并且可以不用对类型起名

```typescript
let obj:{a:string,b:number}={a:'a',b:1}
```

#### 使用值注解

TypeScript可以使用具体的值作为类型注解，比如

```typescript
let a:'str_a'|'str_A'='str_A'
let b:2|3=3
let c:number|'num'='num'
let d:'d'='d'
let e:{E:'E'}={E:'E'}
let f:[1,2,3]=[1,2,3]
function func(params:'params'):'result'{
    return 'result'
}
```

这种以值作为注解的方式常用于各种各样的配置项

#### 泛型

假如我们在声明函数时无法确定这个函数的输入输出类型，只能在调用时确定时，这个时候或许就可以使用泛型了。

```typescript
function reverse<T>(items: T[]): T[] {
    const toreturn:T[] = [];
    for (let i = items.length - 1; i >= 0; i--) {
      toreturn.push(items[i]);
    }
    return toreturn;
  }
console.log(reverse([1,2,3]))//[ 3, 2, 1 ]
console.log(reverse(['a','b','c']))//[ 'c', 'b', 'a' ]
```

上面这个例子，声明时无法确定数组的类型，只能在调用时确定，使用泛型来对输入输出进行约束

#### 联合类型 OR

如果属性为多种类型之一，如字符串或者数组，可以使用联合类型，联合类型使用 | 进行标记

```typescript
let a:string|number=2
```

#### 交叉类型 AND

如果想要同时使用多种类型，可以使用&符号构成交叉类型

```typescript
interface IPropsA{
    x:number
}
interface IPropsB{
    y:string
}
interface IPropsC{
    z:boolean
}
let obj:IPropsA&IPropsB&IPropsC={
    x:1,
    y:'y',
    z:true
}
```

#### 元组类型

JavaScript 并没有支持类似于元组的支持。开发者通常只能使用数组来表示元组，但是 TypeScript 类型系统支持它。使用 `:[typeofmember1, typeofmember2]` 能够为元组添加类型注解，元组可以包含任意数量的成员

#### interface

接口可以将多个类型注解合并到一个新的类型注解中，并对注解内容中的每一个成员进行类型检查。或者可以直接说接口声明就是命名对象类型的一种方式。

>   接口运行时的影响为 0。在 TypeScript 接口中有很多方式来声明变量的结构。

```typescript
interface IProps{...}
```

TypeScript接口是开放式的

再次使用interface可以对已存在的接口进行扩展

下面这个例子对IPropsA进行了扩展，b对象中缺失属性报错

```typescript
interface IPropsA{
    a:string
}
interface IPropsA{
    b:string
}
let b:IPropsA={b:'a'}
```

![image-20221104161407805](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221104161407.png)

**extends**

使用extends继承另一个接口的类型

```typescript
interface IPropsA{
    a:string
}
interface IPropsB extends IPropsA{
    b:string
}
let b:IPropsB={a:'a',b:'b'}
```

**implements**

使用类来实现接口

```typescript
interface Point {
    x: number;
    y: number;
}

class MyPoint implements Point {
    x: number;
    y: number; 
    constructor(x:number,y:number){
        this.x=x;
        this.y=y
    }
}
let foo: Point = new MyPoint(1,2);
console.log(foo)//MyPoint { x: 1, y: 2 }
```



使用implements实现接口后，任何对接口的修改都会导致编译错误

```typescript
interface Point {
    x: number;
    y: number;
}


//error: 类型 "MyPoint" 中缺少属性 "z"，但类型 "Point" 中需要该属性。ts(2420
class MyPoint implements Point {
    x: number;
    y: number; // Same as Point
}

interface Point {
    z: string
}
```

同时，使用implements也会限制类实例的结构



#### type

使用 `type SomeName = someValidTypeAnnotation` 的语法来创建别名，可以给任意的类型注解创建类型别名，包括基础类型

type无法直接对类型编辑进行扩展，只能使用交叉类型的方式来扩展

![image-20221104161700674](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221104161700.png)

而interface可以直接对原类型进行扩展，如上面interface中对IPropsA进行扩展的例子，并且还可以使用extends关键字进行继承扩展

## 类型检查

### typeof

使用typeof检测类型输出类型字符串

```tsx
console.log(typeof { a: 2 })
console.log(typeof true)
console.log(typeof 2)
console.log(typeof 'str')
console.log(typeof undefined)
console.log(typeof null)
console.log(typeof [1, 2, 3])
console.log(typeof function () { })
console.log(typeof typeof { a: 2 })
```

输出

```tsx
object
boolean
number
string
undefined
object
object
function
string
```

## 特殊类型

### Any 类型

在 TypeScript 中，任何类型都可以被归为 any 类型。这让 any 类型成为了类型系统的顶级类型（也被称作全局超级类型）。

```
let notSure: any = 666;
notSure = "Semlinker";
notSure = false;
```

`any` 类型本质上是类型系统的一个逃逸舱。作为开发者，这给了我们很大的自由：TypeScript 允许我们对 `any` 类型的值执行任何操作，而无需事先执行任何形式的检查。比如：

```
let value: any;

value.foo.bar; // OK
value.trim(); // OK
value(); // OK
new value(); // OK
value[0][1]; // OK
```

在许多场景下，这太宽松了。使用 `any` 类型，可以很容易地编写类型正确但在运行时有问题的代码。如果我们使用 `any` 类型，就无法使用 TypeScript 提供的大量的保护机制。为了解决 `any` 带来的问题，TypeScript 3.0 引入了 `unknown` 类型。

### Unknown 类型

就像所有类型都可以赋值给 `any`，所有类型也都可以赋值给 `unknown`。这使得 `unknown` 成为 TypeScript 类型系统的另一种顶级类型（另一种是 `any`）。 `unknown` 类型的使用示例：

```
let value: unknown;

value = true; // OK
value = 42; // OK
value = "Hello World"; // OK
value = []; // OK
value = {}; // OK
value = Math.random; // OK
value = null; // OK
value = undefined; // OK
value = new TypeError(); // OK
value = Symbol("type"); // OK
```

对 `value` 变量的所有赋值都被认为是类型正确的。但是，当我们尝试将类型为 `unknown` 的值赋值给其他类型的变量时会发生什么？

```
let value: unknown;

let value1: unknown = value; // OK
let value2: any = value; // OK
let value3: boolean = value; // Error
let value4: number = value; // Error
let value5: string = value; // Error
let value6: object = value; // Error
let value7: any[] = value; // Error
let value8: Function = value; // Error
```

`unknown` 类型只能被赋值给 `any` 类型和 `unknown` 类型本身。直观地说，这是有道理的：只有能够保存任意类型值的容器才能保存 `unknown` 类型的值。毕竟我们不知道变量 `value` 中存储了什么类型的值。

现在让我们看看当我们尝试对类型为 `unknown` 的值执行操作时会发生什么。以下是我们在之前 `any` 章节看过的相同操作：

```
let value: unknown;

value.foo.bar; // Error
value.trim(); // Error
value(); // Error
new value(); // Error
value[0][1]; // Error
```

将 `value` 变量类型设置为 `unknown` 后，这些操作都不再被认为是类型正确的。通过将 `any` 类型改变为 `unknown` 类型，我们已将允许所有更改的默认设置，更改为禁止任何更改。	

### Tuple 类型

数组一般由同种类型的值组成，但有时我们需要在单个变量中存储不同类型的值，这时候我们就可以使用元组。在 JavaScript 中是没有元组的，元组是 TypeScript 中特有的类型，其工作方式类似于数组。

元组可用于定义具有有限数量的未命名属性的类型。每个属性都有一个关联的类型。使用元组时，必须提供每个属性的值。

```
let tupleType: [string, boolean];
tupleType = ["Semlinker", true];
```

在上面代码中，我们定义了一个名为 `tupleType` 的变量，它的类型是一个类型数组 `[string, boolean]`，然后我们按照正确的类型依次初始化 tupleType 变量。与数组一样，我们可以通过下标来访问元组中的元素：

```
console.log(tupleType[0]); // Semlinker
console.log(tupleType[1]); // true
```

在元组初始化的时候，如果出现类型不匹配的话，比如：

```
tupleType = [true, "Semlinker"];
```

此时，TypeScript 编译器会提示以下错误信息：

```
[0]: Type 'true' is not assignable to type 'string'.
[1]: Type 'string' is not assignable to type 'boolean'.
```

很明显是因为类型不匹配导致的。在元组初始化的时候，我们还必须提供每个属性的值，不然也会出现错误，比如：

```
tupleType = ["Semlinker"];
```

此时，TypeScript 编译器会提示以下错误信息：

```
Property '1' is missing in type '[string]' but required in type '[string, boolean]'.
```

