# TypeScript学习笔记——语法

[toc]

## 学习地址

[了不起的 TypeScript 入门教程]: https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&amp;mid=2247484495&amp;idx=1&amp;sn=a01a57572d38cc51922f57f24289a7b7&amp;chksm=ea47a517dd302c010b9001a1061a6d7dd9e5d2bcc89ccefa94cd98a11c60848084e745501432&amp;scene=178&amp;cur_album_id=1340120212242513920#rd



## 断言

### 类型断言

> 类型断言好比其他语言里的类型转换，但是不进行特殊的数据检查和解构。它没有运行时的影响，只是在编译阶段起作用。

类型断言有两种形式：

####  “尖括号” 语法

```
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

#### as 语法

```
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

### 类型守卫

> A type guard is some expression that performs a runtime check that guarantees the type in some scope. —— TypeScript 官方文档

类型保护是可执行运行时检查的一种表达式，用于确保该类型在一定的范围内。换句话说，类型保护可以保证一个字符串是一个字符串，尽管它的值也可以是一个数值。类型保护与特性检测并不是完全不同，其主要思想是尝试检测属性、方法或原型，以确定如何处理值。目前主要有四种的方式来实现类型保护：

```

```

#### typeof 关键字

```
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
      return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
      return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

`typeof` 类型保护只支持两种形式：`typeof v === "typename"` 和 `typeof v !== typename`，`"typename"` 必须是 `"number"`， `"string"`， `"boolean"` 或 `"symbol"`。 但是 TypeScript 并不会阻止你与其它字符串比较，语言不会把那些表达式识别为类型保护。

####  instanceof 关键字

```
interface Padder {
  getPaddingString(): string;
}

class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) {}
  getPaddingString() {
    return Array(this.numSpaces + 1).join(" ");
  }
}

class StringPadder implements Padder {
  constructor(private value: string) {}
  getPaddingString() {
    return this.value;
  }
}

let padder: Padder = new SpaceRepeatingPadder(6);

if (padder instanceof SpaceRepeatingPadder) {
  // padder的类型收窄为 'SpaceRepeatingPadder'
}
```

####  自定义类型保护的类型谓词

```
function isNumber(x: any): x is number {
  return typeof x === "number";
}

function isString(x: any): x is string {
  return typeof x === "string";
}
```