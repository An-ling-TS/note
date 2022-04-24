# TypeScript学习笔记——高级类型
[toc]
## 学习地址
- [TypeScript中文网](https://www.tslang.cn/docs/handbook/advanced-types.html)
- [typescript进阶篇之高级类型与条件类型(Readonly, Partial, Pick, Record)](https://www.cnblogs.com/Grewer/p/10973744.html)

---
## 映射类型
TypeScript提供了从旧类型中创建新类型的一种方式 — 映射类型。 在映射类型里，新类型以相同的形式去转换旧类型里每个属性
### 学习地址
-[TypeScript实战-18-TS高级类型-映射类型](https://blog.csdn.net/abap_brave/article/details/101073855)
### 例1
```
//最简单的映射类型和它的组成部分：
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```
它的语法与索引签名的语法类型，内部使用了 for .. in。 具有三个部分：

    1.类型变量 K，它会依次绑定到每个属性。
    2.字符串字面量联合的 Keys，它包含了要迭代的属性名的集合。
    3.属性的结果类型。

在个简单的例子里， Keys是硬编码的的属性名列表并且属性类型永远是 boolean，因此这个映射类型等同于：
```
type Flags = {
    option1: boolean;
    option2: boolean;
}
```
### 例2
在真正的应用里, 它们会基于一些已存在的类型，且按照一定的方式转换字段。 这就是 keyof和索引访问类型要做的事情：
```
type NullablePerson = { [P in keyof Person]: Person[P] | null }
type PartialPerson = { [P in keyof Person]?: Person[P] }
```
但它更有用的地方是可以有一些通用版本。
```
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }
```
在这些例子里，属性列表是 keyof T且结果类型是 T[P]的变体。 这是使用通用映射类型的一个好模版。 因为这类转换是 同态的，映射只作用于 T的属性而没有其它的。 编译器知道在添加任何新属性之前可以拷贝所有存在的属性修饰符。 例如，假设 Person.name是只读的，那么 Partial<Person>.name也将是只读的且为可选的。
### 内置的映射类型
#### Readonly
源码
```
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
```
可以将所有属性转化为只读类型
#### Partial
源码
```
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```
可以将所有属性转化为可选类型
#### Pick
源码
```
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
```
#### Record
##### 学习地址
- [typescript中高级类型Record](https://blog.csdn.net/weixin_38080573/article/details/92838045)

Readonly， Partial和 Pick是同态的，但 Record不是。 因为 Record并不需要输入类型来拷贝属性，所以它不属于同态：

    Record将一个类型的所有属性值都映射到另一个类型上并创造一个新的类型
    将K中的每个属性([P in K]),都转为T类型
源码
```
type Record<K extends string, T> = {
    [P in K]: T;
}
```
使用
```
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```
    也可以使用Recard批量包装axios请求
非同态类型本质上会创建新的属性，因此它们不会从它处拷贝属性修饰符。
