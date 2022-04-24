# TypeScript学习笔记——符号

[toc]

## 非空断言符号 ！

### 介绍

> 在上下文中当类型检查器无法断定类型时，一个新的后缀表达式操作符 `!` 可以用于断言操作对象是非 null 和非undefined 类型。**具体而言，`x!` 将从 x 值域中排除 `null` 和 `undefined` 。**

### 例1 

```tsx
function sayHello(name: string | undefined) {
  let sname: string = name; // Error
}
/**
对于以上代码，ts编译器将报错，报错信息：
Type 'string | undefined' is not assignable to type 'string'.
Type 'undefined' is not assignable to type 'string'.
*/
```

要解决上述问题，我们可以简单加个条件判断：

```tsx
function sayHello(name: string | undefined) {
  let sname: string;
  if (name) {
    sname = name;
  }
}
```

而使用 ！可以更简单的解决上述问题

```ts
function sayHello(name: string | undefined) {
  let sname: string = name!;
}
```

### 例2 忽略 undefined 和 null 类型

```tsx
function myFunc(maybeString: string | undefined | null) {
  // Type 'string | null | undefined' is not assignable to type 'string'.
  // Type 'undefined' is not assignable to type 'string'. 
  const onlyString: string = maybeString; // Error
  const ignoreUndefinedAndNull: string = maybeString!; // Ok
}
```

### 例3 调用函数时忽略 undefined 类型

```tsx
type NumGenerator = () => number;

function myFunc(numGenerator: NumGenerator | undefined) {
   // Object is possibly 'undefined'. 
   // Cannot invoke an object which is possibly 'undefined'.
   const num1 = numGenerator(); // Error
   const num2 = numGenerator!(); //OK
}
```

### 例4 使用非空断言操作符的注意事项

因为 `!` 非空断言操作符会从编译生成的 JavaScript 代码中移除，所以在实际使用的过程中，要特别注意。

下面我们来举两个简单的示例：

**示例一**

```tsx
const a: number | undefined = undefined;
const b: number = a!;
console.log(b); 
```

以上 TS 代码会编译生成以下 ES5 代码：

```tsx
"use strict";
const a = undefined;
const b = a;
console.log(b);
```

虽然在 TS 代码中，我们使用了非空断言，使得 `const b: number = a!;` 语句可以通过 TypeScript 类型检查器的检查。但在生成的 ES5 代码中，`!` 非空断言操作符被移除了，所以在浏览器中执行以上代码，在控制台会输出 `undefined`。

**示例二**

```
type NumGenerator = () => number;

function myFunc(numGenerator: NumGenerator | undefined) {
   const num1 = numGenerator!();
}

// Uncaught TypeError: numGenerator is not a function
myFunc(undefined); // Error
```

以上 TS 代码会编译生成以下 ES5 代码：

```
"use strict";
function myFunc(numGenerator) {
  var num1 = numGenerator();
}

// Uncaught TypeError: numGenerator is not a function
myFunc(undefined); // Error
```

若在浏览器中运行以上代码，在控制台会输出以下错误信息：

```
Uncaught TypeError: numGenerator is not a function
    at myFunc (eval at <anonymous> (main-3.js:1239), <anonymous>:3:16)
    at eval (eval at <anonymous> (main-3.js:1239), <anonymous>:6:1)
    at main-3.js:1239
```

很明显在运行时，undefined 并不是函数对象，所以就不能正常调用。

> 需要注意的是，非空断言操作符仅在启用 `strictNullChecks` 标志的时候才生效。当关闭该标志时，编译器不会检查 undefined 类型和 null 类型的赋值。

## ?? 空值合并运算符

在 TypeScript 3.7 版本中除了引入了前面介绍的可选链 `?.` 之外，也引入了一个新的逻辑运算符 —— 空值合并运算符 `??`。**当左侧操作数为 null 或 undefined 时，其返回右侧的操作数，否则返回左侧的操作数**。

与逻辑或 `||` 运算符不同，逻辑或会在左操作数为 falsy 值时返回右侧操作数。也就是说，如果你使用 || 来为某些变量设置默认的值时，你可能会遇到意料之外的行为。比如为 falsy 值（''、NaN 或 0）时。

这里来看一个具体的例子：

```tsx
const foo = null ?? 'default string';
console.log(foo); // 输出："default string"
const baz = 0 ?? 42;
console.log(baz); // 输出：0 
```

以上 TS 代码经过编译后，会生成以下 ES5 代码：

```tsx
"use strict";
var _a, _b;
var foo = (_a = null) !== null && _a !== void 0 ? _a : 'default string';
console.log(foo); // 输出："default string"
var baz = (_b = 0) !== null && _b !== void 0 ? _b : 42;
console.log(baz); // 输出：0
```

通过观察以上代码，我们更加直观的了解到，空值合并运算符是如何解决前面 `||` 运算符存在的潜在问题。下面我们来介绍空值合并运算符的特性和使用时的一些注意事项。

### 短路

当空值合并运算符的左表达式不为 `null` 或 `undefined` 时，不会对右表达式进行求值。

```go
function A() { console.log('A was called'); return undefined;}
function B() { console.log('B was called'); return false;}
function C() { console.log('C was called'); return "foo";}
console.log(A() ?? C());
console.log(B() ?? C());
```

上述代码运行后，控制台会输出以下结果：

```go
A was called 
C was called 
foo 
B was called 
false 
```

### 不能与 && 或 || 操作符共用

若空值合并运算符 `??` 直接与 AND（&&）和 OR（||）操作符组合使用 ?? 是不行的。这种情况下会抛出 SyntaxError。

```go
// '||' and '??' operations cannot be mixed without parentheses.(5076)
null || undefined ?? "foo"; // raises a SyntaxError
// '&&' and '??' operations cannot be mixed without parentheses.(5076)
true && undefined ?? "foo"; // raises a SyntaxError
```

但当使用括号来显式表明优先级时是可行的，比如：

```go
(null || undefined ) ?? "foo"; // 返回 "foo"
```

### 与可选链操作符 ?. 的关系

空值合并运算符针对 undefined 与 null 这两个值，可选链式操作符 `?.`  也是如此。可选链式操作符，对于访问属性可能为 undefined 与 null 的对象时非常有用。

```go
interface Customer {
  name: string;
  city?: string;
}
let customer: Customer = {
  name: "Semlinker"
};
let customerCity = customer?.city ?? "Unknown city";
console.log(customerCity); // 输出：Unknown city
```

前面我们已经介绍了空值合并运算符的应用场景和使用时的一些注意事项，该运算符不仅可以在 TypeScript 3.7  以上版本中使用。当然你也可以在 JavaScript 的环境中使用它，但你需要借助 Babel，在 Babel 7.8.0  版本也开始支持空值合并运算符。

#### ??和||的区别

?? 排除null和undefined 当??左边是false时，返回false，也即不排除false

```tsx
const a = {
    b: 1,
}
const c = false ?? {
    d: 2
}
console.log(c)
// false
```

||左边是false时，继续执行（返回）右侧

```tsx
const a = {
    b: 1,
}
const c = false || {
    d: 2
}
console.log(c)
//{d: 2}
```

