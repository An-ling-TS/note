# lodash 详解

## 介绍

[lodash中文网]: https://www.lodashjs.com/

Lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库。Lodash 遵循 MIT 开源协议发布，并且支持最新的运行环境。

Lodash 通过降低 array、number、objects、string 等等的使用难度从而让 JavaScript 变得更简单。 Lodash 的模块化方法 非常适用于：

- 遍历 array、object 和 string
- 对值进行操作和检测
- 创建符合功能的函数

对于源码中的主要关键部分，我将使用注释进行解释，或者单独使用一节进行解析

阅读此篇，最好：

+   熟悉ES6规范
+   熟悉JS原生对象与函数
+   了解迭代器
+   熟悉JS闭包
+   一定的算法基础

## 迭代器简写

**注意：这里说的迭代器仅限于lodash中的迭代器，而非JavaScript迭代器。因为lodash中的迭代器有时并非以函数形式作为参数，所以我称它为迭代器而非迭代函数，除此之外，lodash中的predicate断言函数也常常在迭代遍历中调用（或者说predicate也是一种迭代器，只不过相比寻常迭代器，predicate只返回布尔值，并且会打断循环），因此lodash中的predicate断言也可以进行简写**

>   **iteratee shorthand**

lodash里有些函数支持迭代器简写，有些不支持，那什么是迭代器简写呢？

iteratee 迭代器，常用于数组类与集合类方法中，传入一个值到迭代器中，会返回一个新的值，因此它的返回可以是任意的

如果传入的迭代器的类型直接就是一个函数，如

```js
function(o) { return o.age < 40; }
```

这种形式的，那它就是一个全写

而如果传入的是

```js
{'active': false }
```

或者

```js
['active', false]
```

亦或者

```js
 'active'
```

那么这就叫简写

举个例子

[find](#find集合查询)

```js
var users = [
  { 'user': 'barney',  'age': 36, 'active': true },
  { 'user': 'fred',    'age': 40, 'active': false },
  { 'user': 'pebbles', 'age': 1,  'active': true }
];
_.find(users, function(o) { return o.age ===40; })
_.find(users, { 'age': 40 })
_.find(users, ['age': 40 ])
```

这三个find是等效的，都会返回数组中第二个对象

```js
{ user: 'fred', age: 40, active: false }
```

而下面这个

```js
_.find(users, 'active')
```

则会返回第一个Boolean(active)为真的对象，也就是下面这个

```js
{ user: 'barney', age: 36, active: true }
```

至于为什么能够使用简写，是因为支持简写的方法使用了baseIteratee函数将这些简写转成了一个完整的方法，而关于baseIteratee是如何通过简写来生成一个完整迭代器的可以查看[baseIteratee](#baseIteratee)

## SameValueZero 

​	**SameValueZero** 是ES6中的比较算法或者说比较规范之一，lodash中有部分函数使用了**SameValueZero** 进行相等比较，它与**SameValue**的主要区别在于在**SameValueZero** 中，+0和-0是相等的。

**SameValueZero(x, y)**

>   The internal comparison abstract operation SameValueZero(x, y), where x and y are ECMAScript language values, produces true or false. Such a comparison is performed as follows:
>
>   >   ReturnIfAbrupt(x).
>   >   ReturnIfAbrupt(y).
>   >   If Type(x) is different from Type(y), return false.
>   >   If Type(x) is Undefined, return true.
>   >   If Type(x) is Null, return true.
>   >   If Type(x) is Number, then
>   >      If x is NaN and y is NaN, return true.
>   >      If x is +0 and y is −0, return true.
>   >      If x is −0 and y is +0, return true.
>   >      If x is the same Number value as y, return true.
>   >      Return false.
>   >   If Type(x) is String, then
>   >      If x and y are exactly the same sequence of code units (same length and same code units at corresponding indices) return true; otherwise, return false.
>   >   If Type(x) is Boolean, then
>   >      If x and y are both true or both false, return true; otherwise, return false.
>   >   If Type(x) is Symbol, then
>   >      If x and y are both the same Symbol value, return true; otherwise, return false.
>   >   Return true if x and y are the same Object value. Otherwise, return false.
>
>   NOTE SameValueZero differs from SameValue only in its treatment of +0 and −0.

+   如果 x 和 y 的类型不同，返回 false
+   如果 x 的类型为 Undefined ，返回 true
+   如果 x 的类型为 Null ，返回 true
+   如果 x 的类型为 Number
    +   如果 x 为 NaN 并且 y 为 NaN ，返回 true
    +   **如果 x 为 +0 并且 y 为 -0 ，返回 true**
    +   **如果 x 为 -0 并且 y 为 +0 ， 返回 true**
    +   如果 x 和 y 的数值一致，返回 true
    +   否则返回false
+   如果 x 的类型为 String，并且 x 和 y 的长度及编码相同，返回 true，否则返回 false
+   如果 x 的类型为 Boolean ，并且 x 和 y 同为 true 或同为false ，返回 true，否则返回 false
+   如果 x 的类型为 Symbol ，并且 x 和 y 具有相同的 Symbol 值，返回 true，否则返回 false
+   如果 x 和 y 指向同一个对象，返回 true， 否则返回 false

## predicate，iteratee与comparator

如果有一定观察，会发现很多lodash函数都接收有一个特别的参数，叫做predicate或者iteratee亦或者comparator，当然，也有可能具有其中的多个。

lodash的工具方法中常常会接收一个函数作为参数，即使不是函数，也可以通过一定的方法转化为函数，比如迭代器简写

lodash容许使用者通过传递predicate，iteratee与comparator，对工具函数的内部逻辑进行一定的影响，通过字面意思我们就可以知道这三个参数名代表着什么参数

+   predicate 断言函数，它的返回是布尔值，可以阻止循环，直接返回结果，获取满足条件的一个或一组值，也可以继续循环，获取满足条件的所有值，一般接收三个参数，当前值value，当前索引index，数组array
+   iteratee 迭代器，常用于数组类与集合类方法中，传入一个值到迭代器中，会返回一个新的值，因此它的返回可以是任意的
+   comparator 比较器，比较器有点类似于上面的断言函数，它不会阻断循环。比较器也返回布尔值，与断言不同的是，比较器主要用于对两个数组中值进行比较，一般接收两个参数，value当前主数组的待比较值，other辅助数组（条件数组）的被比较值。

## symmetric difference

symmetric difference，数学术语：对等差分

具体描述参见 https://en.wikipedia.org/wiki/Symmetric_difference

给出两个集合 (如集合 A = {1, 2, 3} 和集合 B = {2, 3, 4}), 而数学术语 “对等差分”的集合就是指由所有只在两个集合其中之一的元素组成的集合(A △ B = C = {1, 4}).

![image-20211201162425013](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211201162858.png)

对于传入的额外集合 (如 D = {2,3,5}), 应该按前面原则求前两个集合的结果与新集合的对等差分集合 (C △ D = {1, 4} △ {2, 3, 5} = {1, 2, 3, 4, 5})

![image-20211201162847961](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211201162848.png)

## 类数组，类对象，类数组对象

**类数组**

如果value是一个类数组

+   value不为null或undefined。
+   typeOf value !== 'function'
+   typeOf value.length==='number'&&value.length % 1\==\=0

**类对象**

如果value是一个类对象

+   value不为null或undefined
+   typeof value 返回值为 object

function、array和object都算类对象

**类数组对象**

类数组对象在类数组的基础上添加一个条件：value必须为类对象。

## 受保护的方法

lodash 中有许多方法是防止作为其他方法的迭代函数（注：即不能作为iteratee参数传递给其他方法），例如：_.every,_.filter,_.map,_.mapValues,_.reject, 和some。

受保护的方法有（注：即这些方法不能使用_.every,_.filter,_.map,_.mapValues,_.reject, 和_.some作为 iteratee 迭代函数参数） ：
`ary`, `chunk`, `curry`, `curryRight`, `drop`, `dropRight`, `every`,`fill`, `invert`, `parseInt`, `random`, `range`, `rangeRight`, `repeat`,`sampleSize`, `slice`, `some`, `sortBy`, `split`, `take`, `takeRight`,`template`, `trim`, `trimEnd`, `trimStart`, and `words`

## 位掩码bitmask

位掩码可以在lodash的很多工具方法的基础实现中找到，大多以参数形式出现，这是lodash内部定义的一个具有复合含义的值，在代码中虽然以十进制书写，但其实际意义主要由其而二进制位表示，并且其在代码中主要用到的也是位运算。

![image-20220214112246511](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220214112246.png)

从第一位到第十位的表示意义依次如下：

+   WRAP_BIND_FLAG 十进制值 1，二进制表示 0000000001，表示bind函数的标识。典型函数：bind,bindKey

+   WRAP_BIND_KEY_FLAG 十进制 2，二进制表示 0000000010，表示bindKey函数的标识。典型函数：bindKey

+   4

+   WRAP_CURRY_FLAG 十进制 8，二进制表示 0000001000，表示柯里化标识。典型函数：curry

+   WRAP_CURRY_RIGHT_FLAG 十进制16，二进制表示 0000010000，表示柯里化标识。典型函数：curryRight

+   WRAP_PARTIAL_FLAG 十进制 32，二进制表示 0000100000，表示是否应用部分参数（函数包装时传递的参数,为了参数复用，即预设参数）。典型函数：bind,bindKey,partial

+   WRAP_PARTIAL_RIGHT_FLAG 十进制64 ，二进制表示0001000000 表示 附加到提供给新函数的参数后面的参数（占位符参数，从右开始的预设参数）。典型函数：partialRight

+   WRAP_ARY_FLAG 十进制 128，二进制表示0010000000，表示ary函数的标识。典型函数：ary

+   WRAP_REARG_FLAG十进制 256，二进制表示0100000000，表示rearg函数的标识。典型函数：rearg

+   WRAP_FLIP_FLAG 十进制 512，二进制表示1000000000，表示参数翻转的标识。典型函数：flip

    

## **bind与bindKey的区别**

bindKey中返回的闭包获取对象函数是这样的

![image-20220214165006536](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220214165006.png)

bind获取的是func，func是函数包装时传入的函数引用，修改原函数无影响（实质上是创建了新函数，原来的函数依旧存在，并且正被bind使用中）。

```typescript
function bind(func: Function) {
    return function () {
        func()
    }
}
let funcA = () => console.log('funcA');
let fA = bind(funcA);
fA()//funcA
funcA = () => console.log('change funcA');//funcA存放的函数引用被修改为新函数,但是func引用无变化
fA()//funcA
```



bindKey获取的是对象内部函数的引用，函数包装时传递的是对象（引用），修改对象中的函数，其存放的函数引用发生变化，fn也就发生变化。

```typescript
function bindKey(key: string, obj: any) {
    return function () {
        obj[key]();
    }
}
let objA = {
    funcA: () => console.log('funcA'),
}
let fA = bindKey('funcA', objA);
fA()//funcA
objA.funcA = () => console.log('change funcA');//objA.funcA存放的函数引用被修改为新函数,但是objA的引用未变化
fA()//change funcA
```



## 分类

### 数组

| 序号 | 函数名                                                       | 描述                       |
| :--- | :----------------------------------------------------------- | -------------------------- |
| 1    | [chunk](#chunk数组拆分重组)                                  | 数组拆分重组               |
| 2    | [compact ](#compact数组假值过滤)                             | 数组假值过滤               |
| 3    | [concat ](#concat数组连接)                                   | 数组连接                   |
| 4    | [difference ](#difference数组指定过滤)                       | 数组指定过滤               |
| 5    | [differenceBy ](#differenceBy数组迭代过滤)                   | 数组迭代过滤               |
| 6    | [differenceWith ](#differenceWith数组条件过滤)               | 数组条件过滤               |
| 7    | [drop ](#drop去除数组前n个元素)                              | 去除数组前n个元素          |
| 8    | [dropRight ](# dropRight去除数组后n个元素)                   | 去除数组后n个元素          |
| 9    | [dropRightWhile ](#dropRightWhile数组尾部条件过滤)           | 数组尾部条件过滤           |
| 10   | [fill ](#fill数组填充)                                       | 数组填充                   |
| 11   | [findIndex ](#findIndex数组查询)                             | 数组查询                   |
| 12   | [findLastIndex ](#findLastIndex数组倒序查询)                 | 数组倒序查询               |
| 13   | [flatten ](#flatten数组一级展开)                             | 数组一级展开               |
| 14   | [flattenDeep ](#flattenDeep数组完全扁平化)                   | 数组完全扁平化             |
| 15   | [flattenDepth ](#flattenDepth数组指定层级展开)               | 数组指定层级展开           |
| 16   | [fromPairs ](#fromPairs数组转对象)                           | 数组转对象                 |
| 17   | [head ](#head获取数组第一个元素)                             | 获取数组第一个元素         |
| 18   | [indexOf ](#indexOf数组查询)                                 | 数组查询                   |
| 19   | [initial ](#initial获取数组前length-2项)                     | 获取数组前length-2项       |
| 20   | [intersection ](#intersection数组交集)                       | 数组交集                   |
| 21   | [intersectionBy ](#intersectionBy数组迭代处理交集)           | 数组迭代处理交集           |
| 22   | [intersectionWith ](#intersectionWith数组自定义交集)         | 数组自定义交集             |
| 23   | [join ](#join数组以指定分隔符转为字符串)                     | 数组以指定分隔符转为字符串 |
| 24   | [last ](#last获取数组最后一个元素)                           | 获取数组最后一个元素       |
| 25   | [lastIndexOf ](#lastIndexOf数组倒序查询)                     | 数组倒序查询               |
| 26   | [nth ](#nth数组获取指定索引元素)                             | 数组获取指定索引元素       |
| 27   | [pull ](#pull数组移除指定值)                                 | 数组移除指定值             |
| 28   | [pullAll ](#pullAll数组移除指定列表中的值)                   | 数组移除指定列表中的值     |
| 29   | [pullAllBy ](#pullAllBy数组迭代移除)                         | 数组迭代移除               |
| 30   | [pullAllWith ](#pullAllWith数组条件移除)                     | 数组条件移除               |
| 31   | [pullAt ](#pullAt数组移除对应索引元素)                       | 数组移除对应索引元素       |
| 32   | [remove ](#remove数组条件移除所有值)                         | 数组条件移除所有值         |
| 33   | [reverse ](#reverse数组倒置)                                 | 数组倒置                   |
| 34   | [slice ](#slice数组截取)                                     | 数组截取                   |
| 35   | [sortedIndex](#sortedIndex有序数组最小插入索引)              | 有序数组最小插入索引       |
| 36   | [sortedIndexBy ](#sortedIndexBy有序数组迭代最小插入索引)     | 有序数组迭代最小插入索引   |
| 37   | [sortedIndexOf ](#sortedIndexOf有序数组二分查找)             | 有序数组二分查找           |
| 38   | [sortedLastIndex](#sortedLastIndex有序数组最大插入索引)      | 有序数组最大插入索引       |
| 39   | [sortedLastIndexBy ](#sortedLastIndexBy有序数组迭代最大插入索引) | 有序数组迭代最大插入索引   |
| 40   | [sortedLastIndexOf ](#sortedLastIndexOf有序数组倒序二分查找) | 有序数组倒序二分查找       |
| 41   | [sortedUniq ](#sortedUniq有序数组去重)                       | 有序数组去重               |
| 42   | [sortedUniqBy](#sortedUniqBy有序数组迭代去重)                | 有序数组迭代去重           |
| 43   | [tail ](#tail获取数组后length-2项)                           | 获取数组后length-2项       |
| 44   | [take](#take数组提取)                                        | 数组提取                   |
| 45   | [takeRight](#takeRight数组倒序提取)                          | 数组倒序提取               |
| 46   | [takeRightWhile ](#takeRightWhile数组倒序断言提取)           | 数组倒序断言提取           |
| 47   | [takeWhile ](#takeWhile数组断言提取)                         | 数组断言提取               |
| 48   | [union ](#union数组并集去重)                                 | 数组并集去重               |
| 49   | [unionBy](#unionBy数组并集迭代去重)                          | 数组并集迭代去重           |
| 50   | [unionWith ](#unionWith数组并集比较去重)                     | 数组并集比较去重           |
| 51   | [uniq ](#uniq数组去重)                                       | 数组去重                   |
| 52   | [uniqBy](#uniqBy数组迭代去重)                                | 数组迭代去重               |
| 53   | [uniqWith](#uniqWith数组比较去重)                            | 数组比较去重               |
| 54   | [unZip](#unZip数组解压缩)                                    | 数组解压缩                 |
| 55   | [unZipWith ](#unZipWith数组指定分组)                         | 数组指定分组               |
| 56   | [without ](#without数组剔除给定值)                           | 数组剔除给定值             |
| 57   | [xor ](#xor给定数组唯一值)                                   | 给定数组唯一值             |
| 58   | [xorBy ](#xorBy给定数组迭代唯一值)                           | 给定数组迭代唯一值         |
| 59   | [xorWith ](#xorWith给定数组条件唯一值)                       | 给定数组条件唯一值         |
| 60   | [zip](#zip数组重新分组)                                      | 数组重新分组               |
| 61   | [zipObject](#zipObject数组构造对象)                          | 数组构造对象               |
| 62   | [zipObjectDeep](#zipObjectDeep数组构造对象支持深度路径)      | 数组构造对象支持深度路径   |
| 63   | [zipWith](#zipWith数组迭代分组)                              | 数组迭代分组               |

### 集合

| 序号 | 函数名                              | 描述         |
| ---- | ----------------------------------- | ------------ |
| 1    | [countBy ](#countBy集合迭代统计)   | 集合迭代统计 |
| 2    | [every](#every集合遍历检查) | 集合遍历检查 |
| 3    | [filter](#filter集合过滤) | 集合过滤 |
| 4    | [find ](#find集合查询)             | 集合查询     |
| 5    | [findLast ](#findLast集合倒序查询) | 集合倒序查询 |
| 6    | [flatMap](#flatMap迭代结果扁平化) | 迭代结果扁平化 |
| 7    | [flatMapDeep](#flatMapDeep迭代结果完全扁平化) | 迭代结果完全扁平化 |
| 8    | [flatMapDepth](#flatMapDepth迭代结果指定层级展开) | 迭代结果指定层级展开 |
| 9    | [forEach](#forEach集合遍历) | 集合遍历 |
| 10   | [forEachRight](#forEachRight集合反向遍历) | 集合反向遍历 |
| 11   | [groupBy](#groupBy集合迭代分组) | 集合迭代分组 |
| 12   | [includes](#includes集合包含查询) | 集合包含查询 |
| 13   | [invokeMap](#invokeMap集合迭代调用) | 集合迭代调用 |
| 14   | [keyBy](#keyBy集合迭代生成键) | 集合迭代生成键 |
| 15   | [map](#map集合遍历) | 集合遍历 |
| 16   | [orderBy](#orderBy集合迭代结果指定排序) | 集合迭代结果指定排序 |
| 17   | [partition](#partition集合分组) | 集合分组 |
| 18   | [reduce](#reduce集合累计操作) | 集合累计操作 |
| 19   | [reduceRight](#reduceRight集合倒序累计操作) | 集合倒序累计操作 |
| 20   | [reject](#reject集合filter反向过滤) | 集合filter反向过滤 |
| 21   | [sample](#sample获取集合中一个随机元素) | 获取集合中一个随机元素 |
| 22   | [sampleSize](#sampleSize获取集合中n个随机元素) | 获取集合中n个随机元素 |
| 23   | [shuffle](#shuffle集合随机重组为数组) | 集合随机重组为数组 |
| 24   | [size](#size获取集合长度) | 获取集合长度 |
| 25   | [some](#some集合元素存在鉴定) | 集合元素存在鉴定 |
| 26   | [sortBy](#sortBy集合迭代升序) | 集合迭代升序 |

### 对象

| 序号 | 函数名                       | 描述               |
| ---- | ---------------------------- | ------------------ |
| 1    |                              |                    |
| 2    |                              |                    |
| 3    |                              |                    |
| 4    |                              |                    |
| 5    | [at](#at根据路径获取对象值) | 根据路径获取对象值 |
| 6    |                              |                    |
| 7    |                              |                    |
| 8    |                              |                    |
| 9    |                              |                    |
| 10   |                              |                    |
| 11   |                              |                    |
| 12   |                              |                    |
| 13   |                              |                    |
| 14   |                              |                    |
| 15   |                              |                    |
| 16   |                              |                    |
| 17   |                              |                    |
| 18   |                              |                    |
| 19   |                              |                    |
| 20   |                              |                    |
| 21   |                              |                    |
| 22   |                              |                    |
| 23   | [keys](#keys对象属性名数组) | 对象属性名数组 |
| 24   |                              |                    |
| 25   |                              |                    |
| 26   |                              |                    |
| 27   |                              |                    |
| 28   |                              |                    |
| 29   |                              |                    |
| 30   |                              |                    |
| 31   |                              |                    |
| 32   |                              |                    |
| 33   |                              |                    |
| 34   |                              |                    |
| 35   |                              |                    |
| 36   |                              |                    |
| 37   |                              |                    |
| 38   |                              |                    |
| 39   |                              |                    |
| 40   |                              |                    |
| 41   |                              |                    |
| 42   | [values](#values对象值数组) | 对象值数组 |
| 43   |                              |                    |

### 函数

| 序号 | 函数名                    | 描述         |
| ---- | ------------------------- | ------------ |
| 1    | [after ](#after定次触发) | 定次触发     |
| 2    | [ary ](#ary参数次数限制) | 参数次数限制 |
| 3    | [before](#before限次触发) | 限次触发 |
| 4    | [bind](#bind函数指定this) | 函数指定this |
| 5    | [bindKey](#bindKey对象中函数指定this) | 对象中函数指定this |
| 6    | [curry](#curry函数柯里化) | 函数柯里化 |
| 7    | [curryRight](#curryRight函数柯里化倒序接收附加参数) | 函数柯里化倒序接收附加参数 |
| 8    | [debounce](#debounce函数防抖) | 函数防抖 |
| 9    | [defer](#defer函数推迟调用) | 函数推迟调用 |
| 10   | [delay](#delay函数延迟调用) | 函数延迟调用 |
| 11   | [flip](#flip函数参数翻转) | 函数参数翻转 |
| 12   |                           |              |
| 13   | [negate](#negate创建取反函数) | 创建取反函数 |
| 14   | [once](#once函数单次调用) | 函数单次调用 |
| 15   | [overArgs](#overArgs函数参数覆写) | 函数参数覆写 |
| 16   | [partial预设参数](#partial预设参数) | 预设参数 |
| 17   | [partialRight附加预设参数](#partialRight附加预设参数) | 附加预设参数 |
| 18   | [rearg函数指定参数调用](#rearg函数指定参数调用) | 函数指定参数调用 |
| 19   |                           |              |
| 20   |                           |              |
| 21   | [throttle](#throttle函数节流) | 函数节流 |
| 22   |                           |              |
| 23   |                           |              |


### 数学

| 序号 | 函数名                          | 描述     |
| ---- | ------------------------------- | -------- |
| 1    | [add ](#add加法)               | 加法     |
| 2    | [ceil ](#ceil向上舍入)         | 向上舍入 |
| 3    | [divide ](#divide两数相除)     | 两数相除 |
| 4    | [floor ](#floor向下舍入)       | 向下舍入 |
| 5    |                                 |          |
| 6    |                                 |          |
| 7    | [maen](#maen数组均值计算) | 数组均值计算 |
| 8    |                                 |          |
| 9    |                                 |          |
| 10   |                                 |          |
| 11   | [multiply ](#multiply两数相乘) | 两数相乘 |
| 12   | [round ](#round四舍五入)       | 四舍五入 |
| 13   | [subtract ](#subtract两数相减) | 两数相减 |
| 14   |                                 |          |
| 15   |                                 |          |

### 数字

| 序号 | 函数名 | 描述 |
| ---- | ------ | ---- |
| 1    |        |      |
| 2    |        |      |
| 3    |        |      |

### 语言

| 序号 | 函数名                                                 | 描述                  |
| ---- | ------------------------------------------------------ | --------------------- |
| 1    | [castArray](#castArray转换为数组)                      | 转换为数组            |
| 2    |                                                        |                       |
| 3    |                                                        |                       |
| 4    |                                                        |                       |
| 5    |                                                        |                       |
| 6    |                                                        |                       |
| 7    | [eq](#eq值比较)                                        | 值比较                |
| 8    |                                                        |                       |
| 9    |                                                        |                       |
| 10   |                                                        |                       |
| 11   | [isArray](#isArray检查数组)                            | 检查数组              |
| 12   |                                                        |                       |
| 13   | [isArrayLike](#isArrayLike检查类数组)                  | 检查类数组            |
| 14   | [isArrayLikeObject ](#isArrayLikeObject检查类数组对象) | 检查类数组对象        |
| 15   |                                                        |                       |
| 16   |                                                        |                       |
| 17   |                                                        |                       |
| 18   |                                                        |                       |
| 19   |                                                        |                       |
| 20   |                                                        |                       |
| 21   |                                                        |                       |
| 22   | [isError](#isError检查异常)                            | 检查异常              |
| 23   |                                                        |                       |
| 24   | [isFunction](#isFunction检查函数)                      | 检查函数              |
| 25   |                                                        |                       |
| 26   | [isLength](#isLength检查有效长度)                      | 检查有效长度          |
| 27   |                                                        |                       |
| 28   |                                                        |                       |
| 29   |                                                        |                       |
| 30   | [isNaN](#isNaN检查NaN)                                 | 检查NaN               |
| 31   |                                                        |                       |
| 32   | [isNil](#isNil检查null或者undefined)                   | 检查null或者undefined |
| 33   | [isNull](#isNull检查null)                              | 检查null              |
| 34   | [isNumber](#isNumber检查number)                        | 检查number            |
| 35   | [isObject](#isObject检查对象)                          | 检查对象              |
| 36   | [isObjectLike](#isObjectLike检查类对象)                | 检查类对象            |
| 37   | [isPlainObject](#isPlainObject检查普通对象)            | 检查普通对象          |
| 38   |                                                        |                       |
| 39   |                                                        |                       |
| 40   |                                                        |                       |
| 41   | [isString](#isString检查字符串)                        | 检查字符串            |
| 42   | [isSymbol ](#isSymbol检查符号)                         | 检查符号              |
| 43   |                                                        |                       |
| 44   |                                                        |                       |
| 45   |                                                        |                       |
| 46   |                                                        |                       |
| 47   |                                                        |                       |
| 48   |                                                        |                       |
| 49   |                                                        |                       |
| 50   | [toFinite](#toFinite转为有限数字)                      | 转为有限数字          |
| 51   | [toInteger ](#toInteger转为整数)                       | 转为整数              |
| 52   |                                                        |                       |
| 53   | [toNumber](#toNumber转为数字)                          | 转为数字              |
| 54   |                                                        |                       |
| 55   |                                                        |                       |
| 56   | [toString ](#toString转为字符串)                       | 转为字符串            |

### Seq

| 序号 | 函数名 | 描述 |
| ---- | ------ | ---- |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |
|      |        |      |

### 字符串

| 序号 | 函数名                            | 描述         |
| ---- | --------------------------------- | ------------ |
| 1    |                                   |              |
| 2    |                                   |              |
| 3    | [deburr](#deburr转换基本拉丁字母) | 转换基本拉丁字母 |
| 4    |                                   |              |
| 5    |                                   |              |
| 6    |                                   |              |
| 7    |                                   |              |
| 8    |                                   |              |
| 9    |                                   |              |
| 10   |                                   |              |
| 11   |                                   |              |
| 12   |                                   |              |
| 13   |                                   |              |
| 14   |                                   |              |
| 15   |                                   |              |
| 16   |                                   |              |
| 17   |                                   |              |
| 18   |                                   |              |
| 19   |                                   |              |
| 20   |                                   |              |
| 21   | [toLower](#toLower字符串转小写)  | 字符串转小写 |
| 22   | [toUpper ](#toUpper字符串转大写) | 字符串转大写 |
| 23   |                                   |              |
| 24   |                                   |              |
| 25   |                                   |              |
| 26   |                                   |              |
| 27   |                                   |              |
| 28   |                                   |              |
| 29   |                                   |              |
| 30   |                                   |              |

### 实用函数

| 序号 | 函数名                                  | 描述               |
| ---- | --------------------------------------- | ------------------ |
| 1    | [attempt](#attempt函数尝试调用)         | 函数尝试调用       |
| 2    |                                         |                    |
| 3    |                                         |                    |
| 4    |                                         |                    |
| 5    | [constant](#constant返回value)          | 返回value          |
| 6    |                                         |                    |
| 7    |                                         |                    |
| 8    |                                         |                    |
| 9    | [identity](#identity返回首个提供的参数) | 返回首个提供的参数 |
| 10   |                                         |                    |
| 11   |                                         |                    |
| 12   |                                         |                    |
| 13   |                                         |                    |
| 14   |                                         |                    |
| 15   |                                         |                    |
| 16   |                                         |                    |
| 17   |                                         |                    |
| 18   |                                         |                    |
| 19   |                                         |                    |
| 20   |                                         |                    |
| 21   |                                         |                    |
| 22   |                                         |                    |
| 23   |                                         |                    |
| 24   |                                         |                    |
| 25   |                                         |                    |
| 26   |                                         |                    |
| 27   |                                         |                    |
| 28   |                                         |                    |
| 29   |                                         |                    |
| 30   |                                         |                    |
| 31   |                                         |                    |
| 32   |                                         |                    |
| 33   |                                         |                    |
| 34   |                                         |                    |

### Properties

| 序号 | 函数名 | 描述 |
| ---- | ------ | ---- |
| 1    |        |      |
| 2    |        |      |
| 3    |        |      |
| 4    |        |      |
| 5    |        |      |
| 6    |        |      |
| 7    |        |      |

### Methods

| 序号 | 函数名 | 描述 |
| ---- | ------ | ---- |
| 1    |        |      |

### Date

| 序号 | 函数名                            | 描述                   |
| ---- | --------------------------------- | ---------------------- |
| 1    | [now](#now获取Unix纪元至今毫秒数) | 获取Unix纪元至今毫秒数 |

## add加法

**add(augend, addend)**

>    Adds two numbers.
>
>   两个数相加。

### 参数

+   augend number 被加数
+   addend number 加数

### 返回

**number**

计算后的值

### 源码

源码中涉及的函数

+   [createMathOperation](#createMathOperation)

```js
var createMathOperation = require('./_createMathOperation');

/**
 * Adds two numbers.
 *
 * @static
 * @memberOf _
 * @since 3.4.0
 * @category Math
 * @param {number} augend The first number in an addition.
 * @param {number} addend The second number in an addition.
 * @returns {number} Returns the total.
 * @example
 *
 * _.add(6, 4);
 * // => 10
 */
var add = createMathOperation(function(augend, addend) {
  return augend + addend;
}, 0);

module.exports = add;

```



## after定次触发

**after(n, func)**

> The opposite of `_.before`; this method creates a function that invokes `func` once it's called `n` or more times.
>
> _.before的反向函数;此方法创建一个函数，当他被调用`n`或更多次之后将马上触发`func` 。

### 参数

+ `n` *(number)*: `func` 方法应该在调用多少次后才执行。
+ `func` *(Function)*: 用来限定的函数。

### 返回

*(Function)*: 返回新的限定函数。

### 示例

**示例1**

```js
var saves = ['profile', 'settings'];
var done = after(saves.length, function () {
    console.log('done saving!');
});
forEach(saves, function (type) {
    console.log('type', type)
    done()
});
```

输出

```js
type profile
type settings
done saving!
```

当第二次触发done的时候，才执行done

### 源码

源码中涉及的方法

+ [toInteger](#toInteger转为整数)

```js
var toInteger = require('./toInteger');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * The opposite of `_.before`; this method creates a function that invokes
 * `func` once it's called `n` or more times.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {number} n The number of calls before `func` is invoked.
 * @param {Function} func The function to restrict.
 * @returns {Function} Returns the new restricted function.
 * @example
 *
 * var saves = ['profile', 'settings'];
 *
 * var done = _.after(saves.length, function() {
 *   console.log('done saving!');
 * });
 *
 * _.forEach(saves, function(type) {
 *   asyncSave({ 'type': type, 'complete': done });
 * });
 * // => Logs 'done saving!' after the two async saves have completed.
 */
function after(n, func) {
  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  n = toInteger(n);
  //这里用到了闭包，匿名函数引用了after中的变量n
  return function() {
    if (--n < 1) {
      return func.apply(this, arguments);
    }
  };
}

module.exports = after;

```

## ary参数次数限制

**ary(func,[n],[fuard])**

> Creates a function that invokes `func`, with up to `n` arguments, ignoring any additional arguments.
>
> 创建一个调用func的函数。调用func时最多接受 n个参数，忽略多出的参数。

### 参数

+ func 需要被限制参数个数的函数。
+ n 可选 限制的参数数量。
+ guard 可选 允许像' _.map '这样的方法使用作为迭代对象。true/false

### 返回

**function**

被限制参数数量后的方法

### 用例1

```js
let a = ['6', '5', '4']
console.log(a.map(parseInt))//[ 6, NaN, NaN ]
console.log(a.map(ary(parseInt), 1))//[ 6, 5, 4 ]
```

### 用例2

```js
let b = ary(console.log, 1)
console.log(b(1, 2, 3))
//1
//undefined
```

### 源码

源码中涉及的方法

+ [createWrap](#createWrap)

源码中涉及的常量

+ [WRAP_ARY_FLAG](#WRAP_ARY_FLAG)

```js
var createWrap = require('./_createWrap');

/** Used to compose bitmasks for function metadata. */
var WRAP_ARY_FLAG = 128;

/**
 * Creates a function that invokes `func`, with up to `n` arguments,
 * ignoring any additional arguments.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {Function} func The function to cap arguments for.
 * @param {number} [n=func.length] The arity cap.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Function} Returns the new capped function.
 * @example
 *
 * _.map(['6', '8', '10'], _.ary(parseInt, 1));
 * // => [6, 8, 10]
 */
function ary(func, n, guard) {
  n = guard ? undefined : n;
  n = (func && n == null) ? func.length : n;//如果没传n,默认参数数量func的参数个数
  return createWrap(func, WRAP_ARY_FLAG, undefined, undefined, undefined, undefined, n);
}

module.exports = ary;
```

## at根据路径获取对象值

**at(object, [paths])**

>   Creates an array of values corresponding to `paths` of `object`.
>
>   创建一个数组，值来自 object 的paths路径相应的值。

### 参数

+   object Object 要获取值的对象
+   paths Array 路径数组，支持深层路径a.b或者a[0].b等

### 返回

**Array**

返回选中值的数组。

### 源码

源码中涉及的函数

+   [baseAt](#baseAt)
+   [flatRest](#flatRest)

```js
var baseAt = require('./_baseAt'),
    flatRest = require('./_flatRest');

/**
 * Creates an array of values corresponding to `paths` of `object`.
 *
 * @static
 * @memberOf _
 * @since 1.0.0
 * @category Object
 * @param {Object} object The object to iterate over.
 * @param {...(string|string[])} [paths] The property paths to pick.
 * @returns {Array} Returns the picked values.
 * @example
 *
 * var object = { 'a': [{ 'b': { 'c': 3 } }, 4] };
 *
 * _.at(object, ['a[0].b.c', 'a[1]']);
 * // => [3, 4]
 */
var at = flatRest(baseAt);

module.exports = at;

```

## attempt函数尝试调用

**attempt(func, [args])**

>   Attempts to invoke `func`, returning either the result or the caught error object. Any additional arguments are provided to `func` when it's invoked.
>
>   尝试调用func，返回结果 或者 捕捉错误对象。任何附加的参数都会在调用时传给func。

### 参数

+   func Function 尝试调用的函数
+   args ...any 可选 rest 传递给func的参数 

### 返回

**any**

返回函数func的调用结果或者捕捉的异常对象

### 源码

源码中涉及的函数

+   [apply](#apply)
+   [baseRest](#baseRest)
+   [isError](#isError检查异常)

```js
var apply = require('./_apply'),
    baseRest = require('./_baseRest'),
    isError = require('./isError');

/**
 * Attempts to invoke `func`, returning either the result or the caught error
 * object. Any additional arguments are provided to `func` when it's invoked.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Util
 * @param {Function} func The function to attempt.
 * @param {...*} [args] The arguments to invoke `func` with.
 * @returns {*} Returns the `func` result or error object.
 * @example
 *
 * // Avoid throwing errors for invalid selectors.
 * var elements = _.attempt(function(selector) {
 *   return document.querySelectorAll(selector);
 * }, '>_>');
 *
 * if (_.isError(elements)) {
 *   elements = [];
 * }
 */
var attempt = baseRest(function(func, args) {
  try {
    return apply(func, undefined, args);
  } catch (e) {
    return isError(e) ? e : new Error(e);
  }
});

module.exports = attempt;

```

## before限次触发

**before(n, func)**

>Creates a function that invokes `func`, with the `this` binding and arguments of the created function, while it's called less than `n` times. Subsequent calls to the created function return the result of the last `func` invocation.
>
>创建一个调用func的函数，通过this绑定和创建函数的参数调用func，调用次数不超过 n 次。 之后再调用这个函数，将返回一次最后调用func的结果。

### 参数

+   n number 超过多少次不再调用func（注：限制调用func 的次数）
+   func Function 限制执行的函数。最多调用n-1次

### 返回

**Function**

返回新的限定函数。

### 示例

**示例1**

```js
let a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let newConsole = before(5, console.log)
a.forEach(i => newConsole(i))
//1
//2
//3
//4
```

**示例2**

之后再调用这个函数，将返回一次最后调用func的结果。

```
let newConsole = before(5, function (n) {
    console.log(`第${n}次调用`)
    return n
})
let res = []
res.push(newConsole(1))
res.push(newConsole(2))
res.push(newConsole(3))
res.push(newConsole(4))
res.push(newConsole(5))
res.push(newConsole(6))
res.push(newConsole(7))
console.log('res ', res)
// 第1次调用
// 第2次调用
// 第3次调用
// 第4次调用
// res [1, 2, 3, 4, 4, 4, 4]
```



### 解析

```js
var toInteger = require('./toInteger');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * Creates a function that invokes `func`, with the `this` binding and arguments
 * of the created function, while it's called less than `n` times. Subsequent
 * calls to the created function return the result of the last `func` invocation.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {number} n The number of calls at which `func` is no longer invoked.
 * @param {Function} func The function to restrict.
 * @returns {Function} Returns the new restricted function.
 * @example
 *
 * jQuery(element).on('click', _.before(5, addContactToList));
 * // => Allows adding up to 4 contacts to the list.
 */
function before(n, func) {
  var result;
  //如果func不是函数，抛出异常
  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  n = toInteger(n);
  //返回新的限定函数
  return function() {
    //如果当前调用次数小于限定次数，执行限定函数并使用result存放结果
    if (--n > 0) {
      result = func.apply(this, arguments);
    }
    if (n <= 1) {
      func = undefined;
    }
    //返回最后一次调用的结果
    return result;
  };
}

module.exports = before;

```



### 源码

源码中涉及的函数

+   [toInteger ](#toInteger转为整数)

```js
var toInteger = require('./toInteger');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * Creates a function that invokes `func`, with the `this` binding and arguments
 * of the created function, while it's called less than `n` times. Subsequent
 * calls to the created function return the result of the last `func` invocation.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {number} n The number of calls at which `func` is no longer invoked.
 * @param {Function} func The function to restrict.
 * @returns {Function} Returns the new restricted function.
 * @example
 *
 * jQuery(element).on('click', _.before(5, addContactToList));
 * // => Allows adding up to 4 contacts to the list.
 */
function before(n, func) {
  var result;
  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  n = toInteger(n);
  return function() {
    if (--n > 0) {
      result = func.apply(this, arguments);
    }
    if (n <= 1) {
      func = undefined;
    }
    return result;
  };
}

module.exports = before;

```

## bind函数指定this

**bind(func, thisArg, [partials])**

>   Creates a function that invokes `func` with the `this` binding of `thisArg` and `partials` prepended to the arguments it receives.
>
>   The `_.bind.placeholder` value, which defaults to `_` in monolithic builds, may be used as a placeholder for partially applied arguments.
>
>   创建一个调用`func`的函数，`thisArg`绑定`func`函数中的 `this` (注：`this`的上下文为`thisArg`) ，并且`func`函数会接收`partials`附加参数。
>
>   `_.bind.placeholder`值，默认是以 `_` 作为附加部分参数的占位符。

**注意:** 不同于原生的 `Function#bind`，这个方法不会设置绑定函数的 "length" 属性。

### 参数

+   func Function 绑定的函数
+   thisArg any func绑定的this对象
+   partials ...any 可选 rest 附加的部分参数  

### 返回

**Function**

返回新的绑定函数

### 示例

**示例1**

这个例子主要说明bind与bindKey的区别，bind返回闭包，保存的函数引用未变更，因此改变原来绑定的greet函数并不会影响bound

```js
var greet = function (greeting, punctuation) {
    return greeting + ' ' + this.user + punctuation;
}
var object = { 'user': 'fred' };
var bound = bind(greet, object, 'hi');
console.log(bound('!'));
//hi fred!
greet = function (greeting, punctuation) {
    return greeting + 'ya ' + this.user + punctuation;
}
console.log(bound('!'));
//hi fred!
```



### 解析

```js
var baseRest = require('./_baseRest'),//rest包装
    createWrap = require('./_createWrap'),
    getHolder = require('./_getHolder'),//获取占位符
    replaceHolders = require('./_replaceHolders');//占位符替换,返回被替换的占位符的索引数组

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_PARTIAL_FLAG = 32;

/**
 * Creates a function that invokes `func` with the `this` binding of `thisArg`
 * and `partials` prepended to the arguments it receives.
 *
 * The `_.bind.placeholder` value, which defaults to `_` in monolithic builds,
 * may be used as a placeholder for partially applied arguments.
 *
 * **Note:** Unlike native `Function#bind`, this method doesn't set the "length"
 * property of bound functions.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to bind.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {...*} [partials] The arguments to be partially applied.
 * @returns {Function} Returns the new bound function.
 * @example
 *
 * function greet(greeting, punctuation) {
 *   return greeting + ' ' + this.user + punctuation;
 * }
 *
 * var object = { 'user': 'fred' };
 *
 * var bound = _.bind(greet, object, 'hi');
 * bound('!');
 * // => 'hi fred!'
 *
 * // Bound with placeholders.
 * var bound = _.bind(greet, object, _, '!');
 * bound('hi');
 * // => 'hi fred!'
 */
var bind = baseRest(function(func, thisArg, partials) {
  //初始化位掩码
  var bitmask = WRAP_BIND_FLAG;
  //如果有附加参数
  if (partials.length) {
    var holders = replaceHolders(partials, getHolder(bind));
    bitmask |= WRAP_PARTIAL_FLAG;//位掩码加上partial的标识 或操作 例 a=2 a|=4 则a此时为6（二进制下110）
  }
  return createWrap(func, bitmask, thisArg, partials, holders);
});

// Assign default placeholders.
bind.placeholder = {};

module.exports = bind;

```

### 源码

源码中涉及到的函数

+   [baseRest](#baseRest)
+   [createWrap](#createWrap)
+   [getHolder](#getHolder)
+   [replaceHolders](#replaceHolders)

```js
var baseRest = require('./_baseRest'),
    createWrap = require('./_createWrap'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_PARTIAL_FLAG = 32;

/**
 * Creates a function that invokes `func` with the `this` binding of `thisArg`
 * and `partials` prepended to the arguments it receives.
 *
 * The `_.bind.placeholder` value, which defaults to `_` in monolithic builds,
 * may be used as a placeholder for partially applied arguments.
 *
 * **Note:** Unlike native `Function#bind`, this method doesn't set the "length"
 * property of bound functions.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to bind.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {...*} [partials] The arguments to be partially applied.
 * @returns {Function} Returns the new bound function.
 * @example
 *
 * function greet(greeting, punctuation) {
 *   return greeting + ' ' + this.user + punctuation;
 * }
 *
 * var object = { 'user': 'fred' };
 *
 * var bound = _.bind(greet, object, 'hi');
 * bound('!');
 * // => 'hi fred!'
 *
 * // Bound with placeholders.
 * var bound = _.bind(greet, object, _, '!');
 * bound('hi');
 * // => 'hi fred!'
 */
var bind = baseRest(function(func, thisArg, partials) {
  var bitmask = WRAP_BIND_FLAG;
  if (partials.length) {
    var holders = replaceHolders(partials, getHolder(bind));
    bitmask |= WRAP_PARTIAL_FLAG;
  }
  return createWrap(func, bitmask, thisArg, partials, holders);
});

// Assign default placeholders.
bind.placeholder = {};

module.exports = bind;

```

## bindKey对象中函数指定this

**bindKey(object, key, [partials])**

>   Creates a function that invokes the method at `object[key]` with `partials`  prepended to the arguments it receives. 
>
>   This method differs from `_.bind` by allowing bound functions to reference methods that may be redefined or don't yet exist. See [Peter Michaux's article](http://peter.michaux.ca/articles/lazy-function-definition-pattern) for more details.
>
>   The `_.bindKey.placeholder` value, which defaults to `_` in monolithic builds, may be used as a placeholder for partially applied arguments.
>
>   创建一个函数,在`object[key]`上通过接收`partials`附加参数，调用这个方法。
>
>   这个方法与_.bind 的不同之处在于允许重新定义绑定函数即使它还不存在。 浏览[Peter Michaux's article](http://peter.michaux.ca/articles/lazy-function-definition-pattern) 了解更多详情。
>
>   `_.bind.placeholder`值，默认是以 `_` 作为附加部分参数的占位符。

### 参数

+   object Object 需要绑定函数的对象。
+   key string 需要绑定函数对象的键。
+   partials ...any 可选 rest 提前传入的参数

### 返回

**Function**

返回新的绑定函数

### 源码

源码中涉及到的函数

+   [baseRest](#baseRest)
+   [createWrap](#createWrap)
+   [getHolder](#getHolder)
+   [replaceHolders](#replaceHolders)

```js
var baseRest = require('./_baseRest'),
    createWrap = require('./_createWrap'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_PARTIAL_FLAG = 32;

/**
 * Creates a function that invokes the method at `object[key]` with `partials`
 * prepended to the arguments it receives.
 *
 * This method differs from `_.bind` by allowing bound functions to reference
 * methods that may be redefined or don't yet exist. See
 * [Peter Michaux's article](http://peter.michaux.ca/articles/lazy-function-definition-pattern)
 * for more details.
 *
 * The `_.bindKey.placeholder` value, which defaults to `_` in monolithic
 * builds, may be used as a placeholder for partially applied arguments.
 *
 * @static
 * @memberOf _
 * @since 0.10.0
 * @category Function
 * @param {Object} object The object to invoke the method on.
 * @param {string} key The key of the method.
 * @param {...*} [partials] The arguments to be partially applied.
 * @returns {Function} Returns the new bound function.
 * @example
 *
 * var object = {
 *   'user': 'fred',
 *   'greet': function(greeting, punctuation) {
 *     return greeting + ' ' + this.user + punctuation;
 *   }
 * };
 *
 * var bound = _.bindKey(object, 'greet', 'hi');
 * bound('!');
 * // => 'hi fred!'
 *
 * object.greet = function(greeting, punctuation) {
 *   return greeting + 'ya ' + this.user + punctuation;
 * };
 *
 * bound('!');
 * // => 'hiya fred!'
 *
 * // Bound with placeholders.
 * var bound = _.bindKey(object, 'greet', _, '!');
 * bound('hi');
 * // => 'hiya fred!'
 */
var bindKey = baseRest(function(object, key, partials) {
  var bitmask = WRAP_BIND_FLAG | WRAP_BIND_KEY_FLAG;
  if (partials.length) {
    var holders = replaceHolders(partials, getHolder(bindKey));
    bitmask |= WRAP_PARTIAL_FLAG;
  }
  return createWrap(key, bitmask, object, partials, holders);
});

// Assign default placeholders.
bindKey.placeholder = {};

module.exports = bindKey;

```



## castArray转换为数组

**castArray(value)**

>   Casts `value` as an array if it's not one.
>
>   如果 `value` 不是数组, 那么强制转为数组。

### 参数

+   value any 待转换的值

### 返回

**Array**

如果value不是数组，返回转换后的数组

如果value是数组，返回value

### 源码

源码中涉及的函数

+   [isArray](#isArray检查数组)

```js
var isArray = require('./isArray');

/**
 * Casts `value` as an array if it's not one.
 *
 * @static
 * @memberOf _
 * @since 4.4.0
 * @category Lang
 * @param {*} value The value to inspect.
 * @returns {Array} Returns the cast array.
 * @example
 *
 * _.castArray(1);
 * // => [1]
 *
 * _.castArray({ 'a': 1 });
 * // => [{ 'a': 1 }]
 *
 * _.castArray('abc');
 * // => ['abc']
 *
 * _.castArray(null);
 * // => [null]
 *
 * _.castArray(undefined);
 * // => [undefined]
 *
 * _.castArray();
 * // => []
 *
 * var array = [1, 2, 3];
 * console.log(_.castArray(array) === array);
 * // => true
 */
function castArray() {
  if (!arguments.length) {
    return [];
  }
  var value = arguments[0];
  return isArray(value) ? value : [value];
}

module.exports = castArray;

```



## ceil向上舍入

**ceil(number, [precision=0])**

>   Computes `number` rounded up to `precision`.
>
>   根据 precision（精度） 向上舍入 number。（注： precision（精度）可以理解为保留几位小数。）

### 参数

+   number number 待操作值

+   precision number 可选 精度，保留的小数位数 默认precision=0

### 返回

**number**

舍入后的值

### 源码

源码中涉及的函数

+   [createRound](#createRound)

```js
var createRound = require('./_createRound');

/**
 * Computes `number` rounded up to `precision`.
 *
 * @static
 * @memberOf _
 * @since 3.10.0
 * @category Math
 * @param {number} number The number to round up.
 * @param {number} [precision=0] The precision to round up to.
 * @returns {number} Returns the rounded up number.
 * @example
 *
 * _.ceil(4.006);
 * // => 5
 *
 * _.ceil(6.004, 2);
 * // => 6.01
 *
 * _.ceil(6040, -2);
 * // => 6100
 */
var ceil = createRound('ceil');

module.exports = ceil;

```



## chunk数组拆分重组

**chunk(array,[size],[guard])**

​	将数组（array）拆分成多个 size 长度的区块，并将这些区块组成一个新数组。 如果array 无法被分割成全部等长的区块，那么最后剩余的元素将组成一个区块。

> Creates an array of elements split into groups the length of `size`.
>
> 创建一个元素数组，分成长度为' size '的组。
>
> If `array` can't be split evenly, the final chunk will be the remaining elements
>
> 如果' array '不能被平均分割，最后的块将是剩余的元素

### 参数

+ array 待分割的数组
+ size 可选 默认为1 每个数组区块的长度
+ guard 可选 允许像' _.map '这样的方法使用作为迭代对象。true/false

### 返回

**Array**

返回拆分重组后的数组

### 用例1

```js
console.log(chunk(['a', 'b', 'c', 'd'], 2))
//[ [ 'a', 'b' ], [ 'c', 'd' ] ]
console.log(chunk(['a', 'b', 'c', 'd'], 3))
//[ [ 'a', 'b', 'c' ], [ 'd' ] ]
console.log(chunk(['a', 'b', 'c', 'd']))
//[ [ 'a' ], [ 'b' ], [ 'c' ], [ 'd' ] ]
```

### 用例2

```js
let a = ['a', 'b', 'c', 'd']
let b = chunk(a, 2)
console.log(b)
//[ [ 'a', 'b' ], [ 'c', 'd' ] ]
console.log(a)
//[ 'a', 'b', 'c', 'd' ]
```

```js
let a = [{ a: 1 }, { b: 1 }]
let b = chunk(a)
a[0].a = 2
b[1][0].b = 2
console.log(b)
//[ [ { a: 2 } ], [ { b: 2 } ] ]
console.log(a)
//[ { a: 2 }, { b: 2 } ]
```

因此chunk虽然返回的是一个新数组，但是当数组中的值是一个对象时，它返回的是这个对象的引用

### 用例3

```js
console.log([['a', 'b', 'c', 'd'], [1, 2, 3, 4, 5]].map(i => chunk(i, 2, true)))
//[ [ [ 'a', 'b' ], [ 'c', 'd' ] ], [ [ 1, 2 ], [ 3, 4 ], [ 5 ] ] ]
```

### 解析

```js
function chunk(array, size, guard) {
    //是否传入guard,如果传入,判断是否是遍历方法的参数，如果是size=1,否则为传入size和0的最大值
    if ((guard ? isIterateeCall(array, size, guard) : size === undefined)) {
        size = 1;
    } else {
        //在size和0之间选个较大值
        size = nativeMax(toInteger(size), 0);
    }
    var length = array == null ? 0 : array.length;
    //如果length===0或者想要划分的模块大小小于1，返回空数组
    if (!length || size < 1) {
        return [];
    }
    var index = 0,
        resIndex = 0,
        result = Array(nativeCeil(length / size));
    //建一个长度为length/size的数组存放结果
    console.log('result:', result)
    //result: [ <2 empty items> ]
    while (index < length) {
        //添加划分的数组
        result[resIndex++] = baseSlice(array, index, (index += size));
    }
    return result;
}
console.log(chunk(['a', 'b', 'c', 'd'], 2))
```

### 源码

源码中涉及的方法

+ [baseSlice](#baseSlice)

+ [isIterateeCall](#isIterateeCall)
+ [toInteger](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    isIterateeCall = require('./_isIterateeCall'),
    toInteger = require('./toInteger');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeCeil = Math.ceil,
    nativeMax = Math.max;

/**
 * Creates an array of elements split into groups the length of `size`.
 * If `array` can't be split evenly, the final chunk will be the remaining
 * elements.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to process.
 * @param {number} [size=1] The length of each chunk
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the new array of chunks.
 * @example
 *
 * _.chunk(['a', 'b', 'c', 'd'], 2);
 * // => [['a', 'b'], ['c', 'd']]
 *
 * _.chunk(['a', 'b', 'c', 'd'], 3);
 * // => [['a', 'b', 'c'], ['d']]
 */
function chunk(array, size, guard) {
  if ((guard ? isIterateeCall(array, size, guard) : size === undefined)) {
    size = 1;
  } else {
    size = nativeMax(toInteger(size), 0);
  }
  var length = array == null ? 0 : array.length;
  if (!length || size < 1) {
    return [];
  }
  var index = 0,
      resIndex = 0,
      result = Array(nativeCeil(length / size));

  while (index < length) {
    result[resIndex++] = baseSlice(array, index, (index += size));
  }
  return result;
}

module.exports = chunk;
```

## compact数组假值过滤

**compact(Array)**

> Creates an array with all falsey values removed. The values `false`, `null`, `0`, `""`, `undefined`, and `NaN` are falsey.
>
> 创建一个新数组，包含原数组中所有的非假值元素。例如false, null,0, "", undefined, 和 NaN 都是被认为是“假值”。

### 参数

+ array  需要过滤的数组

### 返回

**Array**

过滤后的新数组

### 源码

```js

/**
 * Creates an array with all falsey values removed. The values `false`, `null`,
 * `0`, `""`, `undefined`, and `NaN` are falsey.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to compact.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * _.compact([0, 1, false, 2, '', 3]);
 * // => [1, 2, 3]
 */
function compact(array) {
    //初始化变量
    var index = -1,
        length = array == null ? 0 : array.length,
        resIndex = 0,
        result = [];

    //当index<length，即未遍历到底时
    while (++index < length) {
        //对于对象，这里是一个浅拷贝
        var value = array[index];
        if (value) {
            result[resIndex++] = value;
        }
    }
    return result;
}

```

## concat数组连接

> Creates a new array concatenating `array` with any additional arrays and/or values.
>
> 创建一个新数组，将`array`与任何数组 或 值连接在一起。

### 参数

+ array 第一个参数array是需要连接的原数组
+ rest 剩余的参数是需要被array连接的数组或值

### 返回

**Array**

连接后的的数组

### 示例1

```js
let a = [1]
let b = concat(a, 2, '3', [4, 5])
console.log(b)
//[ 1, 2, '3', 4, 5 ]
```

如果某个参数是个数组，则会把这个数组中的每个子元素拼接到目标结果上

```js
let a = [1]
let b = concat(a, 2, '3', [4, [-4, -5], 5])
console.log(b)
//[ 1, 2, '3', 4, [ -4, -5 ], 5 ]
```

[-4,-5]作为子元素被拼接

```js
let a = [1]
let b = concat(a, 2, [], [[]], null, false, undefined, '')
console.log(b)
//[ 1, 2, [], null, false, undefined, '' ]
```

没有子元素时不会被拼接，但其它的假值会被拼接

```js
let a = { a: 1 }
let c = { c: 2 }
let b = concat(a, c)
a.a = 3
c.c = 3
console.log(b)
//[ { a: 3 }, { c: 3 } ]
```

```js
let a = [[1], 2, 3]
let c = [[-1], -2, -3]
let b = concat(a, c)
a[0].push(4)
c[0].push(-4)
console.log(b)
//[ [ 1, 4 ], 2, 3, [ -1, -4 ], -2, -3 ]
```

显然，对于对象依然是引用

### 解析

```js
var arrayPush = require('./_arrayPush'),
    baseFlatten = require('./_baseFlatten'),
    copyArray = require('./_copyArray'),
    isArray = require('./isArray');

/**
 * Creates a new array concatenating `array` with any additional arrays
 * and/or values.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to concatenate.
 * @param {...*} [values] The values to concatenate.
 * @returns {Array} Returns the new concatenated array.
 * @example
 *
 * var array = [1];
 * var other = _.concat(array, 2, [3], [[4]]);
 *
 * console.log(other);
 * // => [1, 2, 3, [4]]
 *
 * console.log(array);
 * // => [1]
 */
function concat() {
  //length表示参数数量
  var length = arguments.length;
  if (!length) {
  //如果没有参数，直接返回空数组
    return [];
  }
  console.log('length ', length)
  //length  3
  //array是等待拼接的数组，即第一个参数
  var args = Array(length - 1),
      array = arguments[0],
      index = length;
  //当index>0时
  console.log('array ', array)
  //array  [ 1, 2, 3 ]
  while (index--) {
    //将剩余参数放入args中，从最后一个参数开始存放，如果参数是数组，依然将其整体存放
    args[index - 1] = arguments[index];
  }
  console.log('args ', args)
  //args  [ 4, [ 5, 6 ], '-1': [ 1, 2, 3 ] ]
  console.log('baseFlatten(args, 1) ', baseFlatten(args, 1))
  //baseFlatten(args, 1)  [ 4, 5, 6 ]
  //如果array不是数组，将其转为数组，如果是数组，则获取它的拷贝，将args的一层展开(如果参数是数组，这里相当于取出
  //它的子元素)push到array的拷贝或[array]中
  return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));
}
let a = [1, 2, 3]
let b = concat(a, 4, [5, 6])
console.log('b ', b)
//b  [ 1, 2, 3, 4, 5, 6 ]
module.exports = concat;
```

### 源码

```js
var arrayPush = require('./_arrayPush'),
    baseFlatten = require('./_baseFlatten'),
    copyArray = require('./_copyArray'),
    isArray = require('./isArray');

/**
 * Creates a new array concatenating `array` with any additional arrays
 * and/or values.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to concatenate.
 * @param {...*} [values] The values to concatenate.
 * @returns {Array} Returns the new concatenated array.
 * @example
 *
 * var array = [1];
 * var other = _.concat(array, 2, [3], [[4]]);
 *
 * console.log(other);
 * // => [1, 2, 3, [4]]
 *
 * console.log(array);
 * // => [1]
 */
function concat() {
  var length = arguments.length;
  if (!length) {
    return [];
  }
  var args = Array(length - 1),
      array = arguments[0],
      index = length;

  while (index--) {
    args[index - 1] = arguments[index];
  }
  return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));
}

module.exports = concat;

```

## constant返回value

**constant(value)**

>   Creates a function that returns `value`.
>
>   创建一个返回 `value` 的函数。

### 参数

+   value any 待返回的值

### 返回

**Function**

返回一个新函数，这个新函数返回value

### 源码

```js
/**
 * Creates a function that returns `value`.
 *
 * @static
 * @memberOf _
 * @since 2.4.0
 * @category Util
 * @param {*} value The value to return from the new function.
 * @returns {Function} Returns the new constant function.
 * @example
 *
 * var objects = _.times(2, _.constant({ 'a': 1 }));
 *
 * console.log(objects);
 * // => [{ 'a': 1 }, { 'a': 1 }]
 *
 * console.log(objects[0] === objects[1]);
 * // => true
 */
function constant(value) {
  return function() {
    return value;
  };
}

module.exports = constant;

```



## countBy集合迭代统计

**countBy(collection, [iteratee=_.identity])**

>   Creates an object composed of keys generated from the results of running each element of `collection` thru `iteratee`. The corresponding value of each key is the number of times the key was returned by `iteratee`. The iteratee is invoked with one argument: (value).
>
>   创建一个组成对象，key（键）是经过 `iteratee`（迭代函数） 执行处理`collection`中每个元素后返回的结果，每个key（键）对应的值是 `iteratee`（迭代函数）返回该key（键）的次数（注：迭代次数）。 iteratee 调用一个参数：*(value)*。

### 参数

+   collection Array|Object 待统计的集合
+   iteratee Function 可选 迭代器。迭代器的返回结果将作为最终统计结果的键（key），返回相同键（key）的次数将作为这个键的值（value）

### 返回

**Object**

返回统计结果组成的对象

### 示例

**示例1**

```js
console.log(countBy([6.1, 4.2, 6.3], Math.floor))
//{ '4': 1, '6': 2 }
console.log(countBy(['one', 'two', 'three'], 'length'))
//{ '3': 2, '5': 1 }
```



**示例2**

```js
console.log(countBy({ a: 1, b: 2, c: 1, d: 1 }, (val) => val))
//{ '1': 3, '2': 1 }
```

### 源码

源码中涉及的函数

+   [baseAssignValue](#baseAssignValue)
+   [createAggregator](#createAggregator)

```js
var baseAssignValue = require('./_baseAssignValue'),
    createAggregator = require('./_createAggregator');

/** Used for built-in method references. */
var objectProto = Object.prototype;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/**
 * Creates an object composed of keys generated from the results of running
 * each element of `collection` thru `iteratee`. The corresponding value of
 * each key is the number of times the key was returned by `iteratee`. The
 * iteratee is invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 0.5.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The iteratee to transform keys.
 * @returns {Object} Returns the composed aggregate object.
 * @example
 *
 * _.countBy([6.1, 4.2, 6.3], Math.floor);
 * // => { '4': 1, '6': 2 }
 *
 * // The `_.property` iteratee shorthand.
 * _.countBy(['one', 'two', 'three'], 'length');
 * // => { '3': 2, '5': 1 }
 */
//创建一个聚合函数，function作为setter参数传入
var countBy = createAggregator(function(result, value, key) {
  if (hasOwnProperty.call(result, key)) {
    ++result[key];
  } else {
    baseAssignValue(result, key, 1);
  }
});

module.exports = countBy;

```

## curry函数柯里化

**curry(func, [arity=func.length])**

>   Creates a function that accepts arguments of `func` and either invokes `func` returning its result, if at least `arity` number of arguments have been provided, or returns a function that accepts the remaining `func` arguments, and so on. The arity of `func` may be specified if `func.length`  is not sufficient.
>
>   The `_.curry.placeholder` value, which defaults to `_` in monolithic builds,  may be used as a placeholder for provided arguments.
>
>   **Note:** This method doesn't set the "length" property of curried functions.
>
>   创建一个函数，该函数接收 `func` 的参数，要么调用`func`返回的结果，如果 `func` 所需参数已经提供，则直接返回 `func` 所执行的结果。或返回一个函数，接受余下的`func` 参数的函数，可以使用 `func.length` 强制需要累积的参数个数。
>
>   `_.curry.placeholder`值，默认是以 `_` 作为附加部分参数的占位符。
>
>   **Note:** 这个方法不会设置 curried 函数的 "length" 属性。

### 参数

+   func Function 待柯里化的函数
+   arity number 可选 需要提供给 `func` 的参数数量，默认arity=func.length; func.length指func参数数量
+   guard Object 可选 迭代守卫

### 返回

**Function**

返回新的柯里化函数

### 示例

**示例1**

```js
var abc = function (a, b, c) {
    const res = a + b + c
    console.log(`${a}+${b}+${c}=${res}`)
    return res;
};
var curried = curry(abc);
curried(1)(2)(3)//1+2+3=6
curried(1, 2, 3)//1+2+3=6
curried(1, 2)(3)//1+2+3=6
res = curried(1)
res = res(2)
res = res(3)//1+2+3=6
```



### 源码

源码中涉及到的函数

+   [createWrap](#createWrap)

```js
var createWrap = require('./_createWrap');

/** Used to compose bitmasks for function metadata. */
var WRAP_CURRY_FLAG = 8;

/**
 * Creates a function that accepts arguments of `func` and either invokes
 * `func` returning its result, if at least `arity` number of arguments have
 * been provided, or returns a function that accepts the remaining `func`
 * arguments, and so on. The arity of `func` may be specified if `func.length`
 * is not sufficient.
 *
 * The `_.curry.placeholder` value, which defaults to `_` in monolithic builds,
 * may be used as a placeholder for provided arguments.
 *
 * **Note:** This method doesn't set the "length" property of curried functions.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Function
 * @param {Function} func The function to curry.
 * @param {number} [arity=func.length] The arity of `func`.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Function} Returns the new curried function.
 * @example
 *
 * var abc = function(a, b, c) {
 *   return [a, b, c];
 * };
 *
 * var curried = _.curry(abc);
 *
 * curried(1)(2)(3);
 * // => [1, 2, 3]
 *
 * curried(1, 2)(3);
 * // => [1, 2, 3]
 *
 * curried(1, 2, 3);
 * // => [1, 2, 3]
 *
 * // Curried with placeholders.
 * curried(1)(_, 3)(2);
 * // => [1, 2, 3]
 */
function curry(func, arity, guard) {
  arity = guard ? undefined : arity;
  var result = createWrap(func, WRAP_CURRY_FLAG, undefined, undefined, undefined, undefined, undefined, arity);
  result.placeholder = curry.placeholder;
  return result;
}

// Assign default placeholders.
curry.placeholder = {};

module.exports = curry;

```

## curryRight函数柯里化倒序接收附加参数

**curryRight(func, [arity=func.length])**

>   This method is like `_.curry` except that arguments are applied to `func` in the manner of `_.partialRight` instead of `_.partial`.
>
>   The `_.curryRight.placeholder` value, which defaults to `_` in monolithic  builds, may be used as a placeholder for provided arguments.
>
>   **Note:** This method doesn't set the "length" property of curried functions.
>
>   这个方法类似_.curry。除了它接受参数的方式用_.partialRight 代替了_.partial。
>
>   _.curryRight.placeholder值，默认是以 _ 作为附加部分参数的占位符。
>
>   **Note:** 这个方法不会设置 curried 函数的 "length" 属性。

### 参数

+   func Function 待柯里化函数
+   arity number 可选 需要提供给 `func` 的参数数量 默认arity=func.length  func.length为函数参数数量
+   guard Object 可选 迭代守卫

### 返回

**Function**

返回新的柯里化函数

### 源码

源码中涉及到的函数

+   [createWrap](#createWrap)

```js
var createWrap = require('./_createWrap');

/** Used to compose bitmasks for function metadata. */
var WRAP_CURRY_RIGHT_FLAG = 16;

/**
 * This method is like `_.curry` except that arguments are applied to `func`
 * in the manner of `_.partialRight` instead of `_.partial`.
 *
 * The `_.curryRight.placeholder` value, which defaults to `_` in monolithic
 * builds, may be used as a placeholder for provided arguments.
 *
 * **Note:** This method doesn't set the "length" property of curried functions.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {Function} func The function to curry.
 * @param {number} [arity=func.length] The arity of `func`.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Function} Returns the new curried function.
 * @example
 *
 * var abc = function(a, b, c) {
 *   return [a, b, c];
 * };
 *
 * var curried = _.curryRight(abc);
 *
 * curried(3)(2)(1);
 * // => [1, 2, 3]
 *
 * curried(2, 3)(1);
 * // => [1, 2, 3]
 *
 * curried(1, 2, 3);
 * // => [1, 2, 3]
 *
 * // Curried with placeholders.
 * curried(3)(1, _)(2);
 * // => [1, 2, 3]
 */
function curryRight(func, arity, guard) {
  arity = guard ? undefined : arity;
  var result = createWrap(func, WRAP_CURRY_RIGHT_FLAG, undefined, undefined, undefined, undefined, undefined, arity);
  result.placeholder = curryRight.placeholder;
  return result;
}

// Assign default placeholders.
curryRight.placeholder = {};

module.exports = curryRight;

```

## debounce函数防抖

**debounce(func, [wait=0], [options=])**

>   Creates a debounced function that delays invoking `func` until after `wait`  milliseconds have elapsed since the last time the debounced function was invoked. The debounced function comes with a `cancel` method to cancel  delayed `func` invocations and a `flush` method to immediately invoke them.  Provide `options` to indicate whether `func` should be invoked on the leading and/or trailing edge of the `wait` timeout. The `func` is invoked with the last arguments provided to the debounced function. Subsequent calls to the debounced function return the result of the last `func` invocation.
>
>   **Note:** If `leading` and `trailing` options are `true`, `func` is invoked on the trailing edge of the timeout only if the debounced function is invoked more than once during the `wait` timeout.
>
>   If `wait` is `0` and `leading` is `false`, `func` invocation is deferred until to the next tick, similar to `setTimeout` with a timeout of `0`. 
>
>   See [David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/) for details over the differences between `_.debounce` and `_.throttle`.
>
>   创建一个 debounced（防抖动）函数，该函数会从上一次被调用后，延迟 `wait` 毫秒后调用 `func` 方法。 debounced（防抖动）函数提供一个 `cancel` 方法取消延迟的函数调用以及 `flush` 方法立即调用。 可以提供一个 options（选项） 对象决定如何调用 `func` 方法，`options.leading` 与|或 `options.trailing` 决定延迟前后如何触发（注：是 先调用后等待 还是 先等待后调用）。 `func` 调用时会传入最后一次提供给 debounced（防抖动）函数 的参数。 后续调用的 debounced（防抖动）函数返回是最后一次 `func` 调用的结果。
>
>   **注意:** 如果 `leading` 和 `trailing` 选项为 `true`, 则 `func` 允许 trailing 方式调用的条件为: 在 `wait` 期间多次调用防抖方法。
>
>   如果 `wait` 为 `0` 并且 `leading` 为 `false`, `func`调用将被推迟到下一个点，类似`setTimeout`为`0`的超时。

### 参数

+   func Function 待包装函数
+   wait number 可选 需要延迟的毫秒数
+   options Object 可选 配置对象
    +   leading boolean 可选 指定在延迟开始前调用 默认 leading=false
    +   maxWait number 可选 设置func允许被延迟的最大值
    +   trailing boolean 可选 指定在延迟结束后调用 默认 trailing=true

### 返回

**Function**

返回新的 debounced（防抖动）函数

### 解析

```js
var isObject = require('./isObject'),
    now = require('./now'),
    toNumber = require('./toNumber');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max,
    nativeMin = Math.min;

/**
 * Creates a debounced function that delays invoking `func` until after `wait`
 * milliseconds have elapsed since the last time the debounced function was
 * invoked. The debounced function comes with a `cancel` method to cancel
 * delayed `func` invocations and a `flush` method to immediately invoke them.
 * Provide `options` to indicate whether `func` should be invoked on the
 * leading and/or trailing edge of the `wait` timeout. The `func` is invoked
 * with the last arguments provided to the debounced function. Subsequent
 * calls to the debounced function return the result of the last `func`
 * invocation.
 *
 * **Note:** If `leading` and `trailing` options are `true`, `func` is
 * invoked on the trailing edge of the timeout only if the debounced function
 * is invoked more than once during the `wait` timeout.
 *
 * If `wait` is `0` and `leading` is `false`, `func` invocation is deferred
 * until to the next tick, similar to `setTimeout` with a timeout of `0`.
 *
 * See [David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/)
 * for details over the differences between `_.debounce` and `_.throttle`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to debounce.
 * @param {number} [wait=0] The number of milliseconds to delay.
 * @param {Object} [options={}] The options object.
 * @param {boolean} [options.leading=false]
 *  Specify invoking on the leading edge of the timeout.
 * @param {number} [options.maxWait]
 *  The maximum time `func` is allowed to be delayed before it's invoked.
 * @param {boolean} [options.trailing=true]
 *  Specify invoking on the trailing edge of the timeout.
 * @returns {Function} Returns the new debounced function.
 * @example
 *
 * // Avoid costly calculations while the window size is in flux.
 * jQuery(window).on('resize', _.debounce(calculateLayout, 150));
 *
 * // Invoke `sendMail` when clicked, debouncing subsequent calls.
 * jQuery(element).on('click', _.debounce(sendMail, 300, {
 *   'leading': true,
 *   'trailing': false
 * }));
 *
 * // Ensure `batchLog` is invoked once after 1 second of debounced calls.
 * var debounced = _.debounce(batchLog, 250, { 'maxWait': 1000 });
 * var source = new EventSource('/stream');
 * jQuery(source).on('message', debounced);
 *
 * // Cancel the trailing debounced invocation.
 * jQuery(window).on('popstate', debounced.cancel);
 */
function debounce(func, wait, options) {
  var lastArgs,//上次调用参数
      lastThis,//上次调用时的this
      maxWait,//最大延迟时间毫秒数，保证func在maxWait时间段内最少执行一次，这主要用于throttle
      result,
      timerId,//定时执行
      lastCallTime,//上次调用debounced时间,注意，是debounced
      lastInvokeTime = 0,//上次执行时间
      leading = false,
      maxing = false,
      trailing = true;
  //func必须是函数，否则抛出异常
  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  //延迟时间转为有效的值
  wait = toNumber(wait) || 0;
  if (isObject(options)) {
    leading = !!options.leading;
    maxing = 'maxWait' in options;
    //如果配置中存在maxWait，则最大延迟时间取maxWait和wait之间的较大值
    maxWait = maxing ? nativeMax(toNumber(options.maxWait) || 0, wait) : maxWait;
    trailing = 'trailing' in options ? !!options.trailing : trailing;
  }

  function invokeFunc(time) {
    var args = lastArgs,
        thisArg = lastThis;
	//更新lastArgs与lastThis为undefined
    lastArgs = lastThis = undefined;
    //更新lastInvokeTime
    lastInvokeTime = time;
    //存储并返回func执行结果
    result = func.apply(thisArg, args);
    return result;
  }

  function leadingEdge(time) {
    // Reset any `maxWait` timer.
    //更新最后一次执行func时间
    lastInvokeTime = time;
    // Start the timer for the trailing edge.
    //开始timer
    timerId = setTimeout(timerExpired, wait);
    // Invoke the leading edge.
    //如果leading为true，调用func,否则返回result
    return leading ? invokeFunc(time) : result;
  }
  //获取还应等待的时间
  function remainingWait(time) {
    var timeSinceLastCall = time - lastCallTime,
        timeSinceLastInvoke = time - lastInvokeTime,
        //等待时间为需要延迟的时间减去距离上次调用的时间
        timeWaiting = wait - timeSinceLastCall;
	//如果存在最大延迟时间，则返回timeWaiting与maxWait - timeSinceLastInvoke之间的较小值
    return maxing
      ? nativeMin(timeWaiting, maxWait - timeSinceLastInvoke)
      : timeWaiting;
  }
  //判断是否应该被调用，time是当前时间戳
  function shouldInvoke(time) {
    //获取距离上一次调用与执行的时长
    var timeSinceLastCall = time - lastCallTime,
        timeSinceLastInvoke = time - lastInvokeTime;

    // Either this is the first call, activity has stopped and we're at the
    // trailing edge, the system time has gone backwards and we're treating
    // it as the trailing edge, or we've hit the `maxWait` limit.
    //当这是第一次调用denounced（这是debounce返回的防抖函数），或者距离上次调用已大于等于wait，
    //或者距离上次执行func的时长大于等于最大延迟时间时返回true
    return (lastCallTime === undefined || (timeSinceLastCall >= wait) ||
      (timeSinceLastCall < 0) || (maxing && timeSinceLastInvoke >= maxWait));
  }
  //刷新timer
  function timerExpired() {
    var time = now();
    //如果可以执行func，调用trailingEdge
    if (shouldInvoke(time)) {
      return trailingEdge(time);
    }
    //重置timer
    // Restart the timer.
    timerId = setTimeout(timerExpired, remainingWait(time));
  }

  function trailingEdge(time) {
    timerId = undefined;

    // Only invoke if we have `lastArgs` which means `func` has been
    // debounced at least once.
    //当指定在延迟结束后调用并且存在lastArgs时调用func返回结果（存在lastArgs意味着debounced至少被调用了一次，而这次
    //调用中func的执行被延迟到了现在）
    if (trailing && lastArgs) {
      return invokeFunc(time);
    }
    //清除lastArgs和lastThis
    lastArgs = lastThis = undefined;
    return result;
  }
  //取消
  function cancel() {
    if (timerId !== undefined) {
      clearTimeout(timerId);
    }
    lastInvokeTime = 0;
    lastArgs = lastCallTime = lastThis = timerId = undefined;
  }

  function flush() {
    return timerId === undefined ? result : trailingEdge(now());
  }
  //返回的防抖函数
  function debounced() {
    //获取当前时间戳
    var time = now(),
    //判断是否可调用
        isInvoking = shouldInvoke(time);

    lastArgs = arguments;//调用时的参数
    lastThis = this;
    lastCallTime = time;//更新调用debounced的时刻

    if (isInvoking) {
      if (timerId === undefined) {
        //如果timer===undefined，要么是第一次，要么前面的debounced调用已结束，或者其它原因，比如cancel之类
        return leadingEdge(lastCallTime);
      }
      //如果存在最大延迟时间，并且timer存在，重设timer并返回执行结果
      if (maxing) {
        // Handle invocations in a tight loop.
        clearTimeout(timerId);
        timerId = setTimeout(timerExpired, wait);
        return invokeFunc(lastCallTime);
      }
    }
    //第一次调用设置timer
    if (timerId === undefined) {
      timerId = setTimeout(timerExpired, wait);
    }
    return result;
  }
  debounced.cancel = cancel;
  debounced.flush = flush;
  return debounced;
}

module.exports = debounce;

```



### 源码

源码中涉及到的函数

+   [isObject](#isObject检查对象)
+   [now](#now获取Unix纪元至今毫秒数)
+   [toNumber](#toNumber转为数字)

```js
var isObject = require('./isObject'),
    now = require('./now'),
    toNumber = require('./toNumber');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max,
    nativeMin = Math.min;

/**
 * Creates a debounced function that delays invoking `func` until after `wait`
 * milliseconds have elapsed since the last time the debounced function was
 * invoked. The debounced function comes with a `cancel` method to cancel
 * delayed `func` invocations and a `flush` method to immediately invoke them.
 * Provide `options` to indicate whether `func` should be invoked on the
 * leading and/or trailing edge of the `wait` timeout. The `func` is invoked
 * with the last arguments provided to the debounced function. Subsequent
 * calls to the debounced function return the result of the last `func`
 * invocation.
 *
 * **Note:** If `leading` and `trailing` options are `true`, `func` is
 * invoked on the trailing edge of the timeout only if the debounced function
 * is invoked more than once during the `wait` timeout.
 *
 * If `wait` is `0` and `leading` is `false`, `func` invocation is deferred
 * until to the next tick, similar to `setTimeout` with a timeout of `0`.
 *
 * See [David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/)
 * for details over the differences between `_.debounce` and `_.throttle`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to debounce.
 * @param {number} [wait=0] The number of milliseconds to delay.
 * @param {Object} [options={}] The options object.
 * @param {boolean} [options.leading=false]
 *  Specify invoking on the leading edge of the timeout.
 * @param {number} [options.maxWait]
 *  The maximum time `func` is allowed to be delayed before it's invoked.
 * @param {boolean} [options.trailing=true]
 *  Specify invoking on the trailing edge of the timeout.
 * @returns {Function} Returns the new debounced function.
 * @example
 *
 * // Avoid costly calculations while the window size is in flux.
 * jQuery(window).on('resize', _.debounce(calculateLayout, 150));
 *
 * // Invoke `sendMail` when clicked, debouncing subsequent calls.
 * jQuery(element).on('click', _.debounce(sendMail, 300, {
 *   'leading': true,
 *   'trailing': false
 * }));
 *
 * // Ensure `batchLog` is invoked once after 1 second of debounced calls.
 * var debounced = _.debounce(batchLog, 250, { 'maxWait': 1000 });
 * var source = new EventSource('/stream');
 * jQuery(source).on('message', debounced);
 *
 * // Cancel the trailing debounced invocation.
 * jQuery(window).on('popstate', debounced.cancel);
 */
function debounce(func, wait, options) {
  var lastArgs,
      lastThis,
      maxWait,
      result,
      timerId,
      lastCallTime,
      lastInvokeTime = 0,
      leading = false,
      maxing = false,
      trailing = true;

  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  wait = toNumber(wait) || 0;
  if (isObject(options)) {
    leading = !!options.leading;
    maxing = 'maxWait' in options;
    maxWait = maxing ? nativeMax(toNumber(options.maxWait) || 0, wait) : maxWait;
    trailing = 'trailing' in options ? !!options.trailing : trailing;
  }

  function invokeFunc(time) {
    var args = lastArgs,
        thisArg = lastThis;

    lastArgs = lastThis = undefined;
    lastInvokeTime = time;
    result = func.apply(thisArg, args);
    return result;
  }

  function leadingEdge(time) {
    // Reset any `maxWait` timer.
    lastInvokeTime = time;
    // Start the timer for the trailing edge.
    timerId = setTimeout(timerExpired, wait);
    // Invoke the leading edge.
    return leading ? invokeFunc(time) : result;
  }

  function remainingWait(time) {
    var timeSinceLastCall = time - lastCallTime,
        timeSinceLastInvoke = time - lastInvokeTime,
        timeWaiting = wait - timeSinceLastCall;

    return maxing
      ? nativeMin(timeWaiting, maxWait - timeSinceLastInvoke)
      : timeWaiting;
  }

  function shouldInvoke(time) {
    var timeSinceLastCall = time - lastCallTime,
        timeSinceLastInvoke = time - lastInvokeTime;

    // Either this is the first call, activity has stopped and we're at the
    // trailing edge, the system time has gone backwards and we're treating
    // it as the trailing edge, or we've hit the `maxWait` limit.
    return (lastCallTime === undefined || (timeSinceLastCall >= wait) ||
      (timeSinceLastCall < 0) || (maxing && timeSinceLastInvoke >= maxWait));
  }

  function timerExpired() {
    var time = now();
    if (shouldInvoke(time)) {
      return trailingEdge(time);
    }
    // Restart the timer.
    timerId = setTimeout(timerExpired, remainingWait(time));
  }

  function trailingEdge(time) {
    timerId = undefined;

    // Only invoke if we have `lastArgs` which means `func` has been
    // debounced at least once.
    if (trailing && lastArgs) {
      return invokeFunc(time);
    }
    lastArgs = lastThis = undefined;
    return result;
  }

  function cancel() {
    if (timerId !== undefined) {
      clearTimeout(timerId);
    }
    lastInvokeTime = 0;
    lastArgs = lastCallTime = lastThis = timerId = undefined;
  }

  function flush() {
    return timerId === undefined ? result : trailingEdge(now());
  }

  function debounced() {
    var time = now(),
        isInvoking = shouldInvoke(time);

    lastArgs = arguments;
    lastThis = this;
    lastCallTime = time;

    if (isInvoking) {
      if (timerId === undefined) {
        return leadingEdge(lastCallTime);
      }
      if (maxing) {
        // Handle invocations in a tight loop.
        clearTimeout(timerId);
        timerId = setTimeout(timerExpired, wait);
        return invokeFunc(lastCallTime);
      }
    }
    if (timerId === undefined) {
      timerId = setTimeout(timerExpired, wait);
    }
    return result;
  }
  debounced.cancel = cancel;
  debounced.flush = flush;
  return debounced;
}

module.exports = debounce;

```



## deburr转换基本拉丁字母

**deburr([string=''])**

>   Deburrs `string` by converting [Latin-1 Supplement](https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)#Character_table) and [Latin Extended-A](https://en.wikipedia.org/wiki/Latin_Extended-A) letters to basic Latin letters and removing [combining diacritical marks](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks).
>
>   转换字符串`string`中[拉丁语-1补充字母](https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)#Character_table) 和[拉丁语扩展字母-A](https://en.wikipedia.org/wiki/Latin_Extended-A) 为基本的拉丁字母，并且去除组合变音标记。

### 参数

+   string string 可选 待转换的字符串

### 返回

**string**

返回转换后的结果

### 源码

源码中涉及的函数

+   [toString ](#toString转为字符串)
+   [deburrLetter](#deburrLetter)

```js
var deburrLetter = require('./_deburrLetter'),
    toString = require('./toString');

/** Used to match Latin Unicode letters (excluding mathematical operators). */
var reLatin = /[\xc0-\xd6\xd8-\xf6\xf8-\xff\u0100-\u017f]/g;

/** Used to compose unicode character classes. */
var rsComboMarksRange = '\\u0300-\\u036f',
    reComboHalfMarksRange = '\\ufe20-\\ufe2f',
    rsComboSymbolsRange = '\\u20d0-\\u20ff',
    rsComboRange = rsComboMarksRange + reComboHalfMarksRange + rsComboSymbolsRange;

/** Used to compose unicode capture groups. */
var rsCombo = '[' + rsComboRange + ']';

/**
 * Used to match [combining diacritical marks](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks) and
 * [combining diacritical marks for symbols](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks_for_Symbols).
 */
var reComboMark = RegExp(rsCombo, 'g');

/**
 * Deburrs `string` by converting
 * [Latin-1 Supplement](https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)#Character_table)
 * and [Latin Extended-A](https://en.wikipedia.org/wiki/Latin_Extended-A)
 * letters to basic Latin letters and removing
 * [combining diacritical marks](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks).
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category String
 * @param {string} [string=''] The string to deburr.
 * @returns {string} Returns the deburred string.
 * @example
 *
 * _.deburr('déjà vu');
 * // => 'deja vu'
 */
function deburr(string) {
  string = toString(string);
  return string && string.replace(reLatin, deburrLetter).replace(reComboMark, '');
}

module.exports = deburr;

```

## defer函数推迟调用

**defer(func, [args])**

>   Defers invoking the `func` until the current call stack has cleared. Any additional arguments are provided to `func` when it's invoked.
>
>   推迟调用`func`，直到当前堆栈清理完毕。 调用时，任何附加的参数会传给`func`。

### 参数

+   func Function 待推迟调用的函数
+   args ...any 可选 rest 附加参数，调用func时传给它的参数

### 返回

**number**

返回计时器id

### 源码

源码中涉及到的函数

+   [baseDelay](#baseDelay)
+   [baseRest](#baseRest)

```js
var baseDelay = require('./_baseDelay'),
    baseRest = require('./_baseRest');

/**
 * Defers invoking the `func` until the current call stack has cleared. Any
 * additional arguments are provided to `func` when it's invoked.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to defer.
 * @param {...*} [args] The arguments to invoke `func` with.
 * @returns {number} Returns the timer id.
 * @example
 *
 * _.defer(function(text) {
 *   console.log(text);
 * }, 'deferred');
 * // => Logs 'deferred' after one millisecond.
 */
var defer = baseRest(function(func, args) {
  //延迟一毫秒调用func
  return baseDelay(func, 1, args);
});

module.exports = defer;

```



## delay函数延迟调用

**delay(func, wait, [args])**

>   Invokes `func` after `wait` milliseconds. Any additional arguments are provided to `func` when it's invoked.
>
>   延迟 `wait` 毫秒后调用 `func`。 调用时，任何附加的参数会传给`func`。

### 参数

+   func Function 要延迟的函数
+   wait number 延迟时间 毫秒
+   args ...any 可选 rest 附加参数，调用时传给func

### 返回

**number**

返回计时器id

### 源码

源码中涉及到的函数

+   [baseDelay](#baseDelay)
+   [baseRest](#baseRest)
+   [toNumber](#toNumber转为数字)

```js
var baseDelay = require('./_baseDelay'),
    baseRest = require('./_baseRest'),
    toNumber = require('./toNumber');

/**
 * Invokes `func` after `wait` milliseconds. Any additional arguments are
 * provided to `func` when it's invoked.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to delay.
 * @param {number} wait The number of milliseconds to delay invocation.
 * @param {...*} [args] The arguments to invoke `func` with.
 * @returns {number} Returns the timer id.
 * @example
 *
 * _.delay(function(text) {
 *   console.log(text);
 * }, 1000, 'later');
 * // => Logs 'later' after one second.
 */
var delay = baseRest(function(func, wait, args) {
  return baseDelay(func, toNumber(wait) || 0, args);
});

module.exports = delay;

```



## difference数组指定过滤

> Creates an array of `array` values not included in the other given arrays using [`SameValueZero`] for equality comparisons. The order and references of result values are determined by the first array.
>
> 创建一个具有唯一array值的数组，每个值不包含在其他给定的数组中。（注：即创建一个新数组，这个数组中的值，为第一个数字（array 参数）排除了给定数组中的值。）该方法使用SameValueZero做相等比较。结果值的顺序是由第一个数组中的顺序确定。

**注意**: 不像_.pullAll，这个方法会返回一个新数组。

### 参数

+ array Array 要检查的数组。
+ values 可选(...Array): 排除的值。

### 返回

**Array**

返回一个过滤值后的新数组。

### 示例

**示例1**

```js
let a = [1, 2, 3, 4, 5]
let b = []
let res = difference(a)
console.log('res', res)
//res [ 1, 2, 3, 4, 5 ]
```

```js
let a = [{ a: 1 }, 2, 3, 4, 5]
let b = []
let res = difference(a)
a[0].a = -1
console.log('res', res)
res [ { a: -1 }, 2, 3, 4, 5 ]
```

第二个参数是可选的 没有第二个参数时，将返回原数组(对于数组中的对象，依然是浅拷贝)

**示例2**

基本使用

```js
let a = [1, 2, 3, 4, 5]
let b = [2, 4]
let res = difference(a, b)
console.log('res', res)
//res [ 1, 3, 5 ]
```

```js
let a = [1, { a: 2 }, 3, 4, 5, [6, 7], undefined, 4, undefined]
let b = [{ a: 2 }, 4, [6, 7], undefined, 999]
let res = difference(a, b)
console.log('res', res)
//res [ 1, { a: 2 }, 3, 5, [ 6, 7 ] ]
```

不会过滤复杂类型，因为是不同的引用

### 解析

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Creates an array of `array` values not included in the other given arrays
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons. The order and references of result values are
 * determined by the first array.
 *
 * **Note:** Unlike `_.pullAll`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...Array} [values] The values to exclude.
 * @returns {Array} Returns the new array of filtered values.
 * @see _.without, _.xor
 * @example
 *
 * _.difference([2, 1], [2, 3]);
 * // => [1]
 */
var difference = baseRest(function(array, values) {
  return isArrayLikeObject(array)
    //如果array是类数组，调用baseDifference方法,并且将所有过滤数组扁平化一级，比如difference(arr,[1,2],[3,4]) => baseDifference(arr,[1,2,3,4])
    ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true))
    : [];
});

module.exports = difference;

```



### 源码

源码中涉及的方法

+ [baseRest](#baseRest)
+ [isArrayLikeObject](#isArrayLikeObject检查类数组对象)
+ [baseDifference](#baseDifference)
+ [baseFlatten](#baseFlatten)

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Creates an array of `array` values not included in the other given arrays
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons. The order and references of result values are
 * determined by the first array.
 *
 * **Note:** Unlike `_.pullAll`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...Array} [values] The values to exclude.
 * @returns {Array} Returns the new array of filtered values.
 * @see _.without, _.xor
 * @example
 *
 * _.difference([2, 1], [2, 3]);
 * // => [1]
 */
var difference = baseRest(function(array, values) {
  return isArrayLikeObject(array)
    ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true))
    : [];
});

module.exports = difference;

```

## differenceBy数组迭代过滤

> This method is like `_.difference` except that it accepts `iteratee` which is invoked for each element of `array` and `values` to generate the criterion by which they're compared. The order and references of result values are determined by the first array. The iteratee is invoked with one argument: (value).
>
> 这个方法类似_.difference ，除了它接受一个 iteratee （注：迭代器）， 调用array 和 values 中的每个元素以产生比较的标准。 结果值是从第一数组中选择。iteratee 会调用一个参数：(value)。（注：首先使用迭代器分别迭代array 和 values中的每个元素，返回的值作为比较值）。

### 参数

+ array 要检查的数组
+ values 可选 排除的值
+ iteratee  可选 迭代器 调用array和values中的每个元素

### 返回

**Array**

过滤后的数组

### 示例

**示例1**

基本使用

```js
let a = [1, 2, 3, 4, 5, 4]
let b = [2, 4]
let res = differenceBy(a, b)
console.log('res', res)
//res [ 1, 3, 5 ]
```

**示例2**

迭代器

```js
let a = [2.1, 1.2]
let b = [2.3, 3.4]
let res = differenceBy(a, b, Math.floor)
console.log('res', res)
//res [ 1.2 ]
```

Math.floor 向下取整，a,b数组中的值会先向下取整，再进行比较

```js
let a = [2.1, 1.2]
let b = [2.3, 3.4]
let res = differenceBy(a, b, console.log)
console.log('res', res)
//2.3
//3.4
//2.1
//1.2
//res []
```

迭代器的参数是a，b中的每个元素

迭代器简写

```js
let a = [{ 'x': 2 }, { 'x': 1 }]
let b = [{ 'x': 1 }]
let res = differenceBy(a, b, 'x')
console.log('res', res)
//res [ { x: 2 } ]
```

这里会根据‘x'创建一个函数，这个函数的传入参数是a,b中的每个对象，返回值是对象的’x'属性的值

### 解析

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.difference` except that it accepts `iteratee` which
 * is invoked for each element of `array` and `values` to generate the criterion
 * by which they're compared. The order and references of result values are
 * determined by the first array. The iteratee is invoked with one argument:
 * (value).
 *
 * **Note:** Unlike `_.pullAllBy`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...Array} [values] The values to exclude.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * _.differenceBy([2.1, 1.2], [2.3, 3.4], Math.floor);
 * // => [1.2]
 *
 * // The `_.property` iteratee shorthand.
 * _.differenceBy([{ 'x': 2 }, { 'x': 1 }], [{ 'x': 1 }], 'x');
 * // => [{ 'x': 2 }]
 */
var differenceBy = baseRest(function(array, values) {
  //迭代器是values最后一个参数
  var iteratee = last(values);
  //如果是数组，说明没有传iteratee
  if (isArrayLikeObject(iteratee)) {
    iteratee = undefined;
  }
  //如果array是数组，调用baseFlatten将要排除的数组进行一层扁平化处理，使用baseIteratee生成迭代器方法，再调用数组过滤的基本实现方法baseDifference进行过滤
  return isArrayLikeObject(array)
    ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true), baseIteratee(iteratee, 2))
    : [];
});

module.exports = differenceBy;

```



### 源码

源码中涉及的方法

+ [baseDifference](#baseDifference)
+ [baseFlatten](#baseFlatten)
+ [baseIteratee](#baseIteratee)
+ [baseRest](#baseRest)
+ [isArrayLikeObject](#isArrayLikeObject检查类数组对象)
+ [last](#last获取数组最后一个元素)

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.difference` except that it accepts `iteratee` which
 * is invoked for each element of `array` and `values` to generate the criterion
 * by which they're compared. The order and references of result values are
 * determined by the first array. The iteratee is invoked with one argument:
 * (value).
 *
 * **Note:** Unlike `_.pullAllBy`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...Array} [values] The values to exclude.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * _.differenceBy([2.1, 1.2], [2.3, 3.4], Math.floor);
 * // => [1.2]
 *
 * // The `_.property` iteratee shorthand.
 * _.differenceBy([{ 'x': 2 }, { 'x': 1 }], [{ 'x': 1 }], 'x');
 * // => [{ 'x': 2 }]
 */
var differenceBy = baseRest(function(array, values) {
  var iteratee = last(values);
  if (isArrayLikeObject(iteratee)) {
    iteratee = undefined;
  }
  return isArrayLikeObject(array)
    ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true), baseIteratee(iteratee, 2))
    : [];
});

module.exports = differenceBy;

```

## differenceWith数组条件过滤

**differenceWith(array, [values], [comparator])**

>  This method is like `_.difference` except that it accepts `comparator`  which is invoked to compare elements of `array` to `values`. The order and  references of result values are determined by the first array. The comparator  is invoked with two arguments: (arrVal, othVal).
>
> 这个方法类似_.difference ，除了它接受一个 comparator （注：比较器），它调用比较array，values中的元素。 结果值是从第一数组中选择。comparator 调用参数有两个：(arrVal, othVal)。

### 参数

+ array 待过滤数组
+ values 可选 排除的值
+ comparator 可选 比较器 调用每个元素

### 返回

**Array**

过滤后的数组

### 示例

**示例1**

```js
let a = [1, 2, 3]
let b = [1, 2, 9]
let res = differenceWith(a, b, console.log)
console.log('res', res)
// 1 1
// 1 2
// 1 9
// 2 1
// 2 2
// 2 9
// 3 1
// 3 2
// 3 9
// res[1, 2, 3]
```

比较器的参数是两个，第一个是要过滤的数组中的元素，第二个是要排除的数组中的元素

**示例2**

基本使用

```js
let a = [1, 2, 3]
let b = [1, 2, 9]
let res = differenceWith(a, b, (x, y) => Math.pow(x, 2) === y)
console.log('res', res)
//res [ 2 ]
```

### 源码

源码中涉及的方法

+ [baseDifference](#baseDifference)
+ [baseFlatten](#baseFlatten)
+ [baseRest](#baseRest)
+ [isArrayLikeObject](#isArrayLikeObject检查类数组对象)
+ [last](#last获取数组最后一个元素)

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.difference` except that it accepts `comparator`
 * which is invoked to compare elements of `array` to `values`. The order and
 * references of result values are determined by the first array. The comparator
 * is invoked with two arguments: (arrVal, othVal).
 *
 * **Note:** Unlike `_.pullAllWith`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...Array} [values] The values to exclude.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
 *
 * _.differenceWith(objects, [{ 'x': 1, 'y': 2 }], _.isEqual);
 * // => [{ 'x': 2, 'y': 1 }]
 */
var differenceWith = baseRest(function(array, values) {
  var comparator = last(values);
  if (isArrayLikeObject(comparator)) {
    comparator = undefined;
  }
  return isArrayLikeObject(array)
    ? baseDifference(array, baseFlatten(values, 1, isArrayLikeObject, true), undefined, comparator)
    : [];
});

module.exports = differenceWith;

```

## divide两数相除

**divide(dividend, divisor)**

>   Divide two numbers.
>
>   两数相除

### 参数

+   dividend number 被除数
+   divisor number  除数

### 返回

**number**

计算后的结果

### 源码

源码中涉及的函数

+   [createMathOperation](#createMathOperation)

```js
var createMathOperation = require('./_createMathOperation');

/**
 * Divide two numbers.
 *
 * @static
 * @memberOf _
 * @since 4.7.0
 * @category Math
 * @param {number} dividend The first number in a division.
 * @param {number} divisor The second number in a division.
 * @returns {number} Returns the quotient.
 * @example
 *
 * _.divide(6, 4);
 * // => 1.5
 */
var divide = createMathOperation(function(dividend, divisor) {
  return dividend / divisor;
}, 1);

module.exports = divide;

```



## drop去除数组前n个元素

> Creates a slice of `array` with `n` elements dropped from the beginning.
>
> 创建一个切片数组，去除array前面的n个元素。（n默认值为1。）

### 参数

+ array 待处理数组
+ n 可选 去除个数，默认为1
+ guard 可选 是否允许作为迭代器使用

### 返回

**Array**

返回去除元素后的数组

### 源码

源码中涉及的方法

+ [baseSlice](#baseSlice)
+ [toInteger](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    toInteger = require('./toInteger');

/**
 * Creates a slice of `array` with `n` elements dropped from the beginning.
 *
 * @static
 * @memberOf _
 * @since 0.5.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {number} [n=1] The number of elements to drop.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.drop([1, 2, 3]);
 * // => [2, 3]
 *
 * _.drop([1, 2, 3], 2);
 * // => [3]
 *
 * _.drop([1, 2, 3], 5);
 * // => []
 *
 * _.drop([1, 2, 3], 0);
 * // => [1, 2, 3]
 */
function drop(array, n, guard) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  n = (guard || n === undefined) ? 1 : toInteger(n);
  return baseSlice(array, n < 0 ? 0 : n, length);
}

module.exports = drop;

```

## dropRight去除数组后n个元素

> Creates a slice of `array` with `n` elements dropped from the end.
>
> 创建一个切片数组，去除array尾部的n个元素。（n默认值为1。）

### 参数

+ array 待处理数组
+ n 可选 去除个数，默认为1
+ guard 可选 是否允许作为迭代器使用

### 返回

**Array**

返回去除元素后的数组

### 源码

源码中涉及的方法

+ [baseSlice](#baseSlice)
+ [toInteger](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    toInteger = require('./toInteger');

/**
 * Creates a slice of `array` with `n` elements dropped from the end.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {number} [n=1] The number of elements to drop.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.dropRight([1, 2, 3]);
 * // => [1, 2]
 *
 * _.dropRight([1, 2, 3], 2);
 * // => [1]
 *
 * _.dropRight([1, 2, 3], 5);
 * // => []
 *
 * _.dropRight([1, 2, 3], 0);
 * // => [1, 2, 3]
 */
function dropRight(array, n, guard) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  n = (guard || n === undefined) ? 1 : toInteger(n);
  n = length - n;
  return baseSlice(array, 0, n < 0 ? 0 : n);
}

module.exports = dropRight;

```

## dropRightWhile数组尾部条件过滤

**dropRightWhile(array, [predicate=_.identity])**

>  Creates a slice of `array` excluding elements dropped from the end. Elements are dropped until `predicate` returns falsey. The predicate is invoked with three arguments: (value, index, array).
>
> 创建一个切片数组，去除array中从 predicate 返回假值开始到尾部的部分。predicate 会传入3个参数： (value, index, array)。

### 参数

+ array 待操作数组
+ predicate=_.identity 可选 判断方法

### 返回

**Array**

返回array剩余切片。即过滤后的数组。

### 示例

**示例1**

```js
let users = [
    { 'user': 'barney', 'active': true },
    { 'user': 'fred', 'active': false },
    { 'user': 'pebbles', 'active': false }
];
let res = dropRightWhile(users, function (o) { return !o.active; })
console.log('res', res)
//res [ { user: 'barney', active: true } ]
```

**示例2**

```js
let a = [1, 2, 3, 4, 5, 4, 3, 2, 1]
let res = dropRightWhile(a, (value) => value !== 3)
console.log('res', res)
//res[1, 2, 3, 4, 5, 4, 3]
```

被去除的数组长度满足最小

### 源码

源码中涉及的方法

-   [baseIteratee](#baseIteratee)
-   [baseWhile](#baseWhile)

```js
var baseIteratee = require('./_baseIteratee'),
    baseWhile = require('./_baseWhile');

/**
 * Creates a slice of `array` excluding elements dropped from the end.
 * Elements are dropped until `predicate` returns falsey. The predicate is
 * invoked with three arguments: (value, index, array).
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'active': true },
 *   { 'user': 'fred',    'active': false },
 *   { 'user': 'pebbles', 'active': false }
 * ];
 *
 * _.dropRightWhile(users, function(o) { return !o.active; });
 * // => objects for ['barney']
 *
 * // The `_.matches` iteratee shorthand.
 * _.dropRightWhile(users, { 'user': 'pebbles', 'active': false });
 * // => objects for ['barney', 'fred']
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.dropRightWhile(users, ['active', false]);
 * // => objects for ['barney']
 *
 * // The `_.property` iteratee shorthand.
 * _.dropRightWhile(users, 'active');
 * // => objects for ['barney', 'fred', 'pebbles']
 */
function dropRightWhile(array, predicate) {
  return (array && array.length)
    ? baseWhile(array, baseIteratee(predicate, 3), true, true)
    : [];
}

module.exports = dropRightWhile;

```

## eq值比较

**eq(value, other)**

>   comparison between two values to determine if they are equivalent.
>
>   采用SameValueZero标准进行两个参数的比较

### 参数

+   value any 第一个比较值
+   other any 第二个比较值

### 返回

**boolean**

相等返回true，否则返回false，如果value=NaN并且other=NaN也返回true

### 源码

```js
/**
 * Performs a
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * comparison between two values to determine if they are equivalent.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to compare.
 * @param {*} other The other value to compare.
 * @returns {boolean} Returns `true` if the values are equivalent, else `false`.
 * @example
 *
 * var object = { 'a': 1 };
 * var other = { 'a': 1 };
 *
 * _.eq(object, object);
 * // => true
 *
 * _.eq(object, other);
 * // => false
 *
 * _.eq('a', 'a');
 * // => true
 *
 * _.eq('a', Object('a'));
 * // => false
 *
 * _.eq(NaN, NaN);
 * // => true
 */
function eq(value, other) {
  return value === other || (value !== value && other !== other);
}

module.exports = eq;

```

## every集合遍历检查

**every(collection, [predicate=_.identity])**

>   Checks if `predicate` returns truthy for **all** elements of `collection`. Iteration is stopped once `predicate` returns falsey. The predicate is invoked with three arguments: (value, index|key, collection).
>
>   通过 predicate（断言函数） 检查 collection（集合）中的 所有 元素是否都返回真值。一旦 predicate（断言函数） 返回假值，迭代就马上停止。predicate（断言函数）调用三个参数： (value, index|key, collection)。

**注意: 这个方法对于对于空集合返回 true，因为空集合的任何元素都是 true 。**

### 参数

+   collection  Array|Object 待遍历检查的集合
+   predicate Function 可选 检查函数
+   guard boolean 可选 是否可以作为迭代器使用

### 返回

**boolean**

如果所有元素经 predicate（断言函数） 检查后都都返回真值，那么就返回true，否则返回 false 。

### 源码

源码中涉及的函数

+   [arrayEvery](#arrayEvery)
+   [baseEvery](#baseEvery)
+   [baseIteratee](#baseIteratee)
+   [isIterateeCall](#isIterateeCall)
+   [isArray](#isArray检查数组)

```js
var arrayEvery = require('./_arrayEvery'),
    baseEvery = require('./_baseEvery'),
    baseIteratee = require('./_baseIteratee'),
    isArray = require('./isArray'),
    isIterateeCall = require('./_isIterateeCall');

/**
 * Checks if `predicate` returns truthy for **all** elements of `collection`.
 * Iteration is stopped once `predicate` returns falsey. The predicate is
 * invoked with three arguments: (value, index|key, collection).
 *
 * **Note:** This method returns `true` for
 * [empty collections](https://en.wikipedia.org/wiki/Empty_set) because
 * [everything is true](https://en.wikipedia.org/wiki/Vacuous_truth) of
 * elements of empty collections.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {boolean} Returns `true` if all elements pass the predicate check,
 *  else `false`.
 * @example
 *
 * _.every([true, 1, null, 'yes'], Boolean);
 * // => false
 *
 * var users = [
 *   { 'user': 'barney', 'age': 36, 'active': false },
 *   { 'user': 'fred',   'age': 40, 'active': false }
 * ];
 *
 * // The `_.matches` iteratee shorthand.
 * _.every(users, { 'user': 'barney', 'active': false });
 * // => false
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.every(users, ['active', false]);
 * // => true
 *
 * // The `_.property` iteratee shorthand.
 * _.every(users, 'active');
 * // => false
 */
function every(collection, predicate, guard) {
  var func = isArray(collection) ? arrayEvery : baseEvery;
  if (guard && isIterateeCall(collection, predicate, guard)) {
    predicate = undefined;
  }
  return func(collection, baseIteratee(predicate, 3));
}

module.exports = every;

```

## flip函数参数翻转

**flip(func)**

>   Creates a function that invokes `func` with arguments reversed.
>
>   创建一个函数，调用`func`时候接收翻转的参数。

### 参数

+   func Function 要翻转参数的函数

### 返回

**Function**

返回新函数

### 源码

源码中涉及到的函数

+   [createWrap](#createWrap)

```js
var createWrap = require('./_createWrap');

/** Used to compose bitmasks for function metadata. */
var WRAP_FLIP_FLAG = 512;

/**
 * Creates a function that invokes `func` with arguments reversed.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Function
 * @param {Function} func The function to flip arguments for.
 * @returns {Function} Returns the new flipped function.
 * @example
 *
 * var flipped = _.flip(function() {
 *   return _.toArray(arguments);
 * });
 *
 * flipped('a', 'b', 'c', 'd');
 * // => ['d', 'c', 'b', 'a']
 */
function flip(func) {
  return createWrap(func, WRAP_FLIP_FLAG);
}

module.exports = flip;

```

## fill数组填充

**fill(array, value, [start], [end])**

>   Fills elements of `array` with `value` from `start` up to, but not including, `end`.
>
>   使用 value 值来填充（替换） array，从start位置开始, 到end位置结束（但不包含end位置）。

**注意：这个方法会改变 array（注：不是创建新数组）。**

### 参数

+   array Array 要填充改变的数组。
+   value any 填充给 array 的值。
+   start number 可选 开始位置（默认0）。
+   end number 可选 结束位置（默认array.length）。

### 返回

**Array**

返回 array。

### 示例

示例1

```js
let array = [1, 2, 3, 4, 5]
let res = fill(array, 'a', 1, 4)
console.log(res)
//[ 1, 'a', 'a', 'a', 5 ]
```

示例2

```js
let array = [1, 2, 3, 4, 5]
let res = fill(array, ['a', 'b'], 1, 4)
console.log(res)
//[ 1, [ 'a', 'b' ], [ 'a', 'b' ], [ 'a', 'b' ], 5 ]
```



### 源码

源码中涉及的方法

+   [baseFill](#baseFill) fill实现的关键方法
+   [isIterateeCall](#isIterateeCall)

```js
var baseFill = require('./_baseFill'),
    isIterateeCall = require('./_isIterateeCall');

/**
 * Fills elements of `array` with `value` from `start` up to, but not
 * including, `end`.
 *
 * **Note:** This method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 3.2.0
 * @category Array
 * @param {Array} array The array to fill.
 * @param {*} value The value to fill `array` with.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = [1, 2, 3];
 *
 * _.fill(array, 'a');
 * console.log(array);
 * // => ['a', 'a', 'a']
 *
 * _.fill(Array(3), 2);
 * // => [2, 2, 2]
 *
 * _.fill([4, 6, 8, 10], '*', 1, 3);
 * // => [4, '*', '*', 10]
 */
function fill(array, value, start, end) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  //这里为什么要判断是否是迭代调用呢？
  if (start && typeof start != 'number' && isIterateeCall(array, value, start)) {
    start = 0;
    end = length;
  }
  return baseFill(array, value, start, end);
}

module.exports = fill;

```

## filter集合过滤

**filter(collection, [predicate=_.identity])**

>   Iterates over elements of `collection`, returning an array of all elements `predicate` returns truthy for. The predicate is invoked with three arguments: (value, index|key, collection).
>
>   遍历 collection（集合）元素，返回 predicate（断言函数）返回真值 的所有元素的数组。 predicate（断言函数）调用三个参数：(value, index|key, collection)。

### 参数

+   collection Array|Object 待过滤集合
+   predicate Array|Function|Object|string 可选 每次迭代调用的函数

### 返回

**Array**

返回一个新的过滤后的数组

### 源码

源码中涉及的函数

+   [arrayFilter](#arrayFilter)
+   [baseFilter](#baseFilter)
+   [baseIteratee](#baseIteratee)
+   [isArray](#isArray检查数组)

```js
var arrayFilter = require('./_arrayFilter'),
    baseFilter = require('./_baseFilter'),
    baseIteratee = require('./_baseIteratee'),
    isArray = require('./isArray');

/**
 * Iterates over elements of `collection`, returning an array of all elements
 * `predicate` returns truthy for. The predicate is invoked with three
 * arguments: (value, index|key, collection).
 *
 * **Note:** Unlike `_.remove`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new filtered array.
 * @see _.reject
 * @example
 *
 * var users = [
 *   { 'user': 'barney', 'age': 36, 'active': true },
 *   { 'user': 'fred',   'age': 40, 'active': false }
 * ];
 *
 * _.filter(users, function(o) { return !o.active; });
 * // => objects for ['fred']
 *
 * // The `_.matches` iteratee shorthand.
 * _.filter(users, { 'age': 36, 'active': true });
 * // => objects for ['barney']
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.filter(users, ['active', false]);
 * // => objects for ['fred']
 *
 * // The `_.property` iteratee shorthand.
 * _.filter(users, 'active');
 * // => objects for ['barney']
 *
 * // Combining several predicates using `_.overEvery` or `_.overSome`.
 * _.filter(users, _.overSome([{ 'age': 36 }, ['age', 40]]));
 * // => objects for ['fred', 'barney']
 */
function filter(collection, predicate) {
  var func = isArray(collection) ? arrayFilter : baseFilter;
  return func(collection, baseIteratee(predicate, 3));
}

module.exports = filter;

```



## find集合查询

**find(collection, [predicate=_.identity], [fromIndex=0])**

>   Iterates over elements of `collection`, returning the first element `predicate` returns truthy for. The predicate is invoked with three arguments: (value, index|key, collection).
>
>   遍历 `collection`（集合）元素，返回 `predicate`（断言函数）第一个返回真值的第一个元素。predicate（断言函数）调用3个参数： *(value, index|key, collection)*。

### 参数

+   collection Array|Object 待查询的集合
+   predicate Array|Function|Object|string 可选 迭代方法 传入三个参数 
    +   value 当前值
    +   index|key当前值的索引|键
    +   collection 集合
+   fromIndex number 可选 开始搜索的起始位置

### 返回

**any**

如果存在匹配元素，返回匹配元素，否则返回undefined

### 示例

**示例1**

当集合是一个对象时的用法

```js
var users = { 'user': 'barney', 'age': 36, 'active': true }
console.log(find(users, (value, key) => { return value === 36 }))
//36
```

```js
var users = { 'user': 'barney', 'age': 36, 'active': true }
console.log(find(users, (value, key) => { return key === 'age' }))
//36
```

### 源码

源码中涉及的函数

+   [createFind](#createFind)
+   [findIndex](#findIndex)

```js
var createFind = require('./_createFind'),
    findIndex = require('./findIndex');

/**
 * Iterates over elements of `collection`, returning the first element
 * `predicate` returns truthy for. The predicate is invoked with three
 * arguments: (value, index|key, collection).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to inspect.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param {number} [fromIndex=0] The index to search from.
 * @returns {*} Returns the matched element, else `undefined`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'age': 36, 'active': true },
 *   { 'user': 'fred',    'age': 40, 'active': false },
 *   { 'user': 'pebbles', 'age': 1,  'active': true }
 * ];
 *
 * _.find(users, function(o) { return o.age < 40; });
 * // => object for 'barney'
 *
 * // The `_.matches` iteratee shorthand.
 * _.find(users, { 'age': 1, 'active': true });
 * // => object for 'pebbles'
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.find(users, ['active', false]);
 * // => object for 'fred'
 *
 * // The `_.property` iteratee shorthand.
 * _.find(users, 'active');
 * // => object for 'barney'
 */
var find = createFind(findIndex);

module.exports = find;

```



## findIndex数组查询

**findIndex(array, [predicate=_.identity], [fromIndex=0])**

>   This method is like `_.find` except that it returns the index of the first element `predicate` returns truthy for instead of the element itself.
>
>   该方法类似_.find，区别是该方法返回第一个通过 predicate 判断为真值的元素的索引值（index），而不是元素本身。

### 参数

+   array Array 待搜索数组
+   predicate Array|Function|Object|string 可选 迭代方法，用于判断当前值是否是搜索的值
+   fromIndex number 可选 搜索的起始位置，默认为0

### 返回

**number**

如果存在被搜索值，则返回它的索引，否则返回-1

### 示例

示例1

```js
let users = [
    { 'user': 'barney', 'active': false },
    { 'user': 'fred', 'active': false },
    { 'user': 'pebbles', 'active': true }
];
let res = findIndex(users, function (o) { return o.user == 'barney'; })
console.log(res)
//0
```

示例2

```js
let users = [
    { 'user': 'barney', 'active': false },
    { 'user': 'fred', 'active': false },
    { 'user': 'pebbles', 'active': true }
];
let res = findIndex(users, { 'user': 'fred', 'active': false })
console.log(res)
//1
```

示例3

```js
let users = [
    { 'user': 'barney', 'active': false },
    { 'user': 'fred', 'active': false },
    { 'user': 'pebbles', 'active': true }
];
let res = findIndex(users, ['user', 'pebbles'])
console.log(res)
//2
```

示例4

```js
let users = [
    { 'user': 'barney', 'active': false },
    { 'user': 'fred', 'active': false },
    { 'user': 'pebbles', 'active': true }
];
let res = findIndex(users, 'active')
console.log(res)
//2
```



### 源码

源码中涉及的方法

+   [baseFindIndex](#baseFindIndex)

+   [baseIteratee](#baseIteratee)
+   [toInteger](#toInteger转为整数)

```js
var baseFindIndex = require('./_baseFindIndex'),
    baseIteratee = require('./_baseIteratee'),
    toInteger = require('./toInteger');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * This method is like `_.find` except that it returns the index of the first
 * element `predicate` returns truthy for instead of the element itself.
 *
 * @static
 * @memberOf _
 * @since 1.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param {number} [fromIndex=0] The index to search from.
 * @returns {number} Returns the index of the found element, else `-1`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'active': false },
 *   { 'user': 'fred',    'active': false },
 *   { 'user': 'pebbles', 'active': true }
 * ];
 *
 * _.findIndex(users, function(o) { return o.user == 'barney'; });
 * // => 0
 *
 * // The `_.matches` iteratee shorthand.
 * _.findIndex(users, { 'user': 'fred', 'active': false });
 * // => 1
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.findIndex(users, ['active', false]);
 * // => 0
 *
 * // The `_.property` iteratee shorthand.
 * _.findIndex(users, 'active');
 * // => 2
 */
function findIndex(array, predicate, fromIndex) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return -1;
  }
  var index = fromIndex == null ? 0 : toInteger(fromIndex);
  if (index < 0) {
    index = nativeMax(length + index, 0);
  }
  return baseFindIndex(array, baseIteratee(predicate, 3), index);
}

module.exports = findIndex;

```

## findLast集合倒序查询

**findLast(collection, [predicate=_.identity], [fromIndex=collection.length-1])**

>   This method is like `_.find` except that it iterates over elements of `collection` from right to left.
>
>   这个方法类似_.find ，不同之处在于，_.findLast是从右至左遍历collection （集合）元素的。

### 参数

+   collection Array|Object 待查询的集合
+   predicate Array|Function|Object|string 可选 迭代方法 传入三个参数 
    +   value 当前值
    +   index|key当前值的索引|键
    +   collection 集合
+   fromIndex number 可选 开始搜索的起始位置

### 返回

**any**

从右到左进行查找，返回满足条件的第一个元素

### 源码

源码中涉及的方法

+   [createFind](#createFind)
+   [findLastIndex](#findLastIndex)

```js
var createFind = require('./_createFind'),
    findLastIndex = require('./findLastIndex');

/**
 * This method is like `_.find` except that it iterates over elements of
 * `collection` from right to left.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to inspect.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param {number} [fromIndex=collection.length-1] The index to search from.
 * @returns {*} Returns the matched element, else `undefined`.
 * @example
 *
 * _.findLast([1, 2, 3, 4], function(n) {
 *   return n % 2 == 1;
 * });
 * // => 3
 */
var findLast = createFind(findLastIndex);

module.exports = findLast;

```



## findLastIndex数组倒序查询

**findLastIndex(array, [predicate=_.identity], [fromIndex=array.length-1])**

>   This method is like `_.findIndex` except that it iterates over elements of `collection` from right to left.
>
>   这个方式类似_.findIndex， 区别是它是从右到左的迭代集合array中的元素。

### 参数

+   array Array 待搜索的数组
+   predicate Array|Function|Object|string 可选 这个函数会在每一次迭代调用,用于判断当前值是否是被搜索的值
+   fromIndex number 可选 查询的起始位置，默认是array.length-1

### 返回

**number**

如果存在待搜索值，返回它的索引，否则返回-1

### 源码

源码中涉及的方法

+   [toInteger](#toInteger转为整数)
+   [baseIteratee](#baseIteratee)
+   [baseFindIndex](baseFindIndex)

```js
var baseFindIndex = require('./_baseFindIndex'),
    baseIteratee = require('./_baseIteratee'),
    toInteger = require('./toInteger');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max,
    nativeMin = Math.min;

/**
 * This method is like `_.findIndex` except that it iterates over elements
 * of `collection` from right to left.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param {number} [fromIndex=array.length-1] The index to search from.
 * @returns {number} Returns the index of the found element, else `-1`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'active': true },
 *   { 'user': 'fred',    'active': false },
 *   { 'user': 'pebbles', 'active': false }
 * ];
 *
 * _.findLastIndex(users, function(o) { return o.user == 'pebbles'; });
 * // => 2
 *
 * // The `_.matches` iteratee shorthand.
 * _.findLastIndex(users, { 'user': 'barney', 'active': true });
 * // => 0
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.findLastIndex(users, ['active', false]);
 * // => 2
 *
 * // The `_.property` iteratee shorthand.
 * _.findLastIndex(users, 'active');
 * // => 0
 */
function findLastIndex(array, predicate, fromIndex) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return -1;
  }
  var index = length - 1;
  if (fromIndex !== undefined) {
    index = toInteger(fromIndex);
    index = fromIndex < 0
      ? nativeMax(length + index, 0)
      : nativeMin(index, length - 1);
  }
  return baseFindIndex(array, baseIteratee(predicate, 3), index, true);
}

module.exports = findLastIndex;

```

## flatMap迭代结果扁平化

**flatMap(collection, [iteratee=_.identity])**

>   Creates a flattened array of values by running each element in `collection` thru `iteratee` and flattening the mapped results. The iteratee is invoked with three arguments: (value, index|key, collection).
>
>   创建一个扁平化（注：同阶数组）的数组，这个数组的值来自`collection`（集合）中的每一个值经过 `iteratee`（迭代函数） 处理后返回的结果，并且扁平化合并。 iteratee 调用三个参数： *(value, index|key, collection)*。

**注意：这个函数只会将结果展开一层**

### 参数

+   collection Array|Object 待处理集合
+   iteratee Array|Function|Object|string 可选 迭代器 每次迭代调用

### 返回

**Array**

返回处理后的新的扁平化数组

### 源码

源码中涉及的函数

+   [baseFlatten](#baseFlatten)
+   [map](#map集合遍历)

```js
var baseFlatten = require('./_baseFlatten'),
    map = require('./map');

/**
 * Creates a flattened array of values by running each element in `collection`
 * thru `iteratee` and flattening the mapped results. The iteratee is invoked
 * with three arguments: (value, index|key, collection).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * function duplicate(n) {
 *   return [n, n];
 * }
 *
 * _.flatMap([1, 2], duplicate);
 * // => [1, 1, 2, 2]
 */
function flatMap(collection, iteratee) {
  return baseFlatten(map(collection, iteratee), 1);
}

module.exports = flatMap;

```

## flatMapDeep迭代结果完全扁平化

**flatMapDeep(collection, [iteratee=_.identity])**

>   This method is like `_.flatMap` except that it recursively flattens the mapped results.
>
>   这个方法类似_.flatMap 不同之处在于，_.flatMapDeep 会继续扁平化递归映射的结果。

### 参数

+   collection Array|Object 待处理的集合
+   iteratee Array|Function|Object|string 可选 迭代器 每次迭代调用

### 返回

**Array**

返回处理后的新的扁平化数组

### 源码

源码中涉及否函数

+   [baseFlatten](#baseFlatten)
+   [map](#map集合遍历)

```js
var baseFlatten = require('./_baseFlatten'),
    map = require('./map');

/** Used as references for various `Number` constants. */
var INFINITY = 1 / 0;

/**
 * This method is like `_.flatMap` except that it recursively flattens the
 * mapped results.
 *
 * @static
 * @memberOf _
 * @since 4.7.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * function duplicate(n) {
 *   return [[[n, n]]];
 * }
 *
 * _.flatMapDeep([1, 2], duplicate);
 * // => [1, 1, 2, 2]
 */
function flatMapDeep(collection, iteratee) {
  return baseFlatten(map(collection, iteratee), INFINITY);
}

module.exports = flatMapDeep;

```

## flatMapDepth迭代结果指定层级展开

**flatMapDepth(collection, [iteratee=_.identity], [depth=1])**

>   This method is like `_.flatMap` except that it recursively flattens the  mapped results up to `depth` times.
>
>   该方法类似_.flatMap，不同之处在于，_.flatMapDepth 会根据指定的 depth（递归深度）继续扁平化递归映射结果。

### 参数

+   collection Array|Object 待处理的集合
+   iteratee Array|Function|Object|string 可选 迭代器 每次迭代调用
+   depth number 可选 最大递归深度，指定展开的层级，默认depth=1

### 返回

**Array**

返回处理后的新的指定层级展开的数组

### 源码

源码中涉及的函数

+   [baseFlatten](#baseFlatten)
+   [map](#map集合遍历)
+   [toInteger ](#toInteger转为整数)

```js
var baseFlatten = require('./_baseFlatten'),
    map = require('./map'),
    toInteger = require('./toInteger');

/**
 * This method is like `_.flatMap` except that it recursively flattens the
 * mapped results up to `depth` times.
 *
 * @static
 * @memberOf _
 * @since 4.7.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @param {number} [depth=1] The maximum recursion depth.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * function duplicate(n) {
 *   return [[[n, n]]];
 * }
 *
 * _.flatMapDepth([1, 2], duplicate, 2);
 * // => [[1, 1], [2, 2]]
 */
function flatMapDepth(collection, iteratee, depth) {
  depth = depth === undefined ? 1 : toInteger(depth);
  return baseFlatten(map(collection, iteratee), depth);
}

module.exports = flatMapDepth;

```



## flatten数组一级展开

**flatten(array)**

>   Flattens `array` a single level deep.
>
>   减少一级`array`嵌套深度。

### 参数

+   array Array 待展开的数组

### 返回

**Array**

展开后的新数组

### 示例

**示例1**

```js
console.log(flatten([1, [2, [3, [4]], 5]]))
//[ 1, 2, [ 3, [ 4 ] ], 5 ]
```

### 源码

源码中涉及的方法

+   [baseFlatten](#baseFlatten) flatten实现的基础方法

```js
var baseFlatten = require('./_baseFlatten');

/**
 * Flattens `array` a single level deep.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to flatten.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * _.flatten([1, [2, [3, [4]], 5]]);
 * // => [1, 2, [3, [4]], 5]
 */
function flatten(array) {
  var length = array == null ? 0 : array.length;
  return length ? baseFlatten(array, 1) : [];
}

module.exports = flatten;

```

## flattenDeep数组完全扁平化

**flattenDeep(array)**

>   Recursively flattens `array`.
>
>   将`array`递归为一维数组。

### 参数

+   array Array 待扁平化处理的数组

### 返回 

**Array**

扁平化后的新数组

### 示例

**示例1**

```js
console.log(flattenDeep([1, [2, [3, [4]], 5]]))
//[ 1, 2, 3, 4, 5 ]
```

### 源码

源码中涉及的方法

+   [baseFlatten](#baseFlatten) flattenDeep实现的基础方法

```js
var baseFlatten = require('./_baseFlatten');

/** Used as references for various `Number` constants. */
var INFINITY = 1 / 0;

/**
 * Recursively flattens `array`.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to flatten.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * _.flattenDeep([1, [2, [3, [4]], 5]]);
 * // => [1, 2, 3, 4, 5]
 */
function flattenDeep(array) {
  var length = array == null ? 0 : array.length;
  //传入一个正无穷的值作为扁平化层级
  return length ? baseFlatten(array, INFINITY) : [];
}

module.exports = flattenDeep;

```



## flattenDepth数组指定层级展开

**flattenDepth(array, [depth=1])**

>   Recursively flatten `array` up to `depth` times.
>
>   根据 `depth` 递归减少 `array` 的嵌套层级

### 参数

+   array Array 待展开的数组
+   depth number 可选 指定展开的层级 默认depth=1

### 返回

**Array**

展开后的新数组

### 示例

**示例1**

```js
console.log(flattenDepth([1, [2, [3, [4]], 5]], 2))
//[ 1, 2, 3, [ 4 ], 5 ]
```

### 源码

源码中涉及的方法

+   [toInteger](#toInteger转为整数)
+   [baseFlatten](#baseFlatten)

```js
var baseFlatten = require('./_baseFlatten'),
    toInteger = require('./toInteger');

/**
 * Recursively flatten `array` up to `depth` times.
 *
 * @static
 * @memberOf _
 * @since 4.4.0
 * @category Array
 * @param {Array} array The array to flatten.
 * @param {number} [depth=1] The maximum recursion depth.
 * @returns {Array} Returns the new flattened array.
 * @example
 *
 * var array = [1, [2, [3, [4]], 5]];
 *
 * _.flattenDepth(array, 1);
 * // => [1, 2, [3, [4]], 5]
 *
 * _.flattenDepth(array, 2);
 * // => [1, 2, 3, [4], 5]
 */
function flattenDepth(array, depth) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  depth = depth === undefined ? 1 : toInteger(depth);
  return baseFlatten(array, depth);
}

module.exports = flattenDepth;

```

## floor向下舍入

**floor(number, [precision=0])**

>   Computes `number` rounded down to `precision`.
>
>   根据 precision（精度） 向下舍入 number。（注： precision（精度）可以理解为保留几位小数。）

### 参数

+   number number 待舍入的值
+   precision number 可选 精度，保留的小数位数 默认precision=0

### 返回

**number**

舍入后的值

### 源码

源码中涉及的函数

+   [createRound](#createRound)

```js
var createRound = require('./_createRound');

/**
 * Computes `number` rounded down to `precision`.
 *
 * @static
 * @memberOf _
 * @since 3.10.0
 * @category Math
 * @param {number} number The number to round down.
 * @param {number} [precision=0] The precision to round down to.
 * @returns {number} Returns the rounded down number.
 * @example
 *
 * _.floor(4.006);
 * // => 4
 *
 * _.floor(0.046, 2);
 * // => 0.04
 *
 * _.floor(4060, -2);
 * // => 4000
 */
var floor = createRound('floor');

module.exports = floor;

```

## forEach集合遍历

forEach(collection, [iteratee=_.identity])

>   Iterates over elements of `collection` and invokes `iteratee` for each element. The iteratee is invoked with three arguments: (value, index|key, collection). Iteratee functions may exit iteration early by explicitly returning `false`.
>
>   调用 iteratee 遍历 collection(集合) 中的每个元素， iteratee 调用3个参数： (value, index|key, collection)。 如果迭代函数（iteratee）显式的返回 false ，迭代会提前退出。

### 参数

+   collection Array|Object 待遍历的集合
+   iteratee Function 可选 迭代器 每次遍历调用

### 返回

**any**

返回集合 collection

### 源码

源码中涉及的函数

+   [arrayEach](#arrayEach)
+   [baseEach](#baseEach)
+   [isArray](#isArray检查数组)
+   [castFunction](#castFunction)

```js
var arrayEach = require('./_arrayEach'),
    baseEach = require('./_baseEach'),
    castFunction = require('./_castFunction'),
    isArray = require('./isArray');

/**
 * Iterates over elements of `collection` and invokes `iteratee` for each element.
 * The iteratee is invoked with three arguments: (value, index|key, collection).
 * Iteratee functions may exit iteration early by explicitly returning `false`.
 *
 * **Note:** As with other "Collections" methods, objects with a "length"
 * property are iterated like arrays. To avoid this behavior use `_.forIn`
 * or `_.forOwn` for object iteration.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @alias each
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @returns {Array|Object} Returns `collection`.
 * @see _.forEachRight
 * @example
 *
 * _.forEach([1, 2], function(value) {
 *   console.log(value);
 * });
 * // => Logs `1` then `2`.
 *
 * _.forEach({ 'a': 1, 'b': 2 }, function(value, key) {
 *   console.log(key);
 * });
 * // => Logs 'a' then 'b' (iteration order is not guaranteed).
 */
function forEach(collection, iteratee) {
  var func = isArray(collection) ? arrayEach : baseEach;
  return func(collection, castFunction(iteratee));
}

module.exports = forEach;

```

## forEachRight集合反向遍历

**forEachRight(collection, [iteratee=_.identity])**

>   This method is like `_.forEach` except that it iterates over elements of `collection` from right to left.
>
>   这个方法类似_.forEach，不同之处在于，_.forEachRight 是从右到左遍历集合中每一个元素的。

### 参数

+   collection Array|Object 待遍历的集合
+   iteratee Function 迭代器

### 返回

**Array|Object**

返回集合

### 源码

源码中涉及的函数

+   [castFunction](#castFunction)
+   [isArrayLike](#isArrayLike检查类数组)
+   [arrayEachRight](#arrayEachRight)
+   [baseEachRight](#baseEachRight)

```js
var arrayEachRight = require('./_arrayEachRight'),
    baseEachRight = require('./_baseEachRight'),
    castFunction = require('./_castFunction'),
    isArray = require('./isArray');

/**
 * This method is like `_.forEach` except that it iterates over elements of
 * `collection` from right to left.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @alias eachRight
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @returns {Array|Object} Returns `collection`.
 * @see _.forEach
 * @example
 *
 * _.forEachRight([1, 2], function(value) {
 *   console.log(value);
 * });
 * // => Logs `2` then `1`.
 */
function forEachRight(collection, iteratee) {
  var func = isArray(collection) ? arrayEachRight : baseEachRight;
  return func(collection, castFunction(iteratee));
}

module.exports = forEachRight;

```



## fromPairs数组转对象

**fromPairs(pairs)**

>   The inverse of `_.toPairs`; this method returns an object composed from key-value `pairs`.
>
>   与[`toPairs`正好相反；这个方法返回一个由键值对`pairs`构成的对象。

### 参数

+   paris array 键值对

### 返回

**Object**

返回由键值对构成的新对象

### 示例

**示例1**

```js
console.log(fromPairs([['a', 1], ['b', 2]]))
//{ a: 1, b: 2 }
```

**示例2**

```js
console.log(fromPairs(['a', 1]))
//{ a: undefined, undefined: undefined }
```

**示例3**

```js
console.log(fromPairs([[['a', 1]], [['b', 2]]]))
//{ 'a,1': undefined, 'b,2': undefined }
```



### 解析

```js
/**
 * The inverse of `_.toPairs`; this method returns an object composed
 * from key-value `pairs`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} pairs The key-value pairs.
 * @returns {Object} Returns the new object.
 * @example
 *
 * _.fromPairs([['a', 1], ['b', 2]]);
 * // => { 'a': 1, 'b': 2 }
 */
function fromPairs(pairs) {
  var index = -1,
      length = pairs == null ? 0 : pairs.length,
      result = {};
  //这里是先自增，再比较,从循环体中看出，pairs需要是一个二维数组，并且每个子数组的长度至少为2，长度超出2（索引大于1）
  //的部分会被忽略
  while (++index < length) {
    var pair = pairs[index];
    result[pair[0]] = pair[1];
  }
  return result;
}

module.exports = fromPairs;

```



### 源码

```js
/**
 * The inverse of `_.toPairs`; this method returns an object composed
 * from key-value `pairs`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} pairs The key-value pairs.
 * @returns {Object} Returns the new object.
 * @example
 *
 * _.fromPairs([['a', 1], ['b', 2]]);
 * // => { 'a': 1, 'b': 2 }
 */
function fromPairs(pairs) {
  var index = -1,
      length = pairs == null ? 0 : pairs.length,
      result = {};

  while (++index < length) {
    var pair = pairs[index];
    result[pair[0]] = pair[1];
  }
  return result;
}

module.exports = fromPairs;

```



## groupBy集合迭代分组

**groupBy(collection, [iteratee=_.identity])**

>   Creates an object composed of keys generated from the results of running each element of `collection` thru `iteratee`. The order of grouped values is determined by the order they occur in `collection`. The corresponding value of each key is an array of elements responsible for generating the key. The iteratee is invoked with one argument: (value).
>
>   创建一个对象，key 是 `iteratee` 遍历 `collection`(集合) 中的每个元素返回的结果。 分组值的顺序是由他们出现在 `collection`(集合) 中的顺序确定的。每个键对应的值负责生成 key 的元素组成的数组。iteratee 调用 1 个参数： *(value)*。

### 参数

+   collection Array|Object 待分组的集合
+   iteratee Array|Function|Object|string 迭代器 这个迭代函数用来转换key

### 返回

**Object**

返回一个组成聚合的对象

### 示例

**示例1**

```js
console.log(groupBy(['one', 'two', 'three'], 'length'))
//{ '3': [ 'one', 'two' ], '5': [ 'three' ] }
```



### 源码

源码中涉及的函数

+   [baseAssignValue](#baseAssignValue)
+   [createAggregator](#createAggregator)

```js
var baseAssignValue = require('./_baseAssignValue'),
    createAggregator = require('./_createAggregator');

/** Used for built-in method references. */
var objectProto = Object.prototype;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/**
 * Creates an object composed of keys generated from the results of running
 * each element of `collection` thru `iteratee`. The order of grouped values
 * is determined by the order they occur in `collection`. The corresponding
 * value of each key is an array of elements responsible for generating the
 * key. The iteratee is invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The iteratee to transform keys.
 * @returns {Object} Returns the composed aggregate object.
 * @example
 *
 * _.groupBy([6.1, 4.2, 6.3], Math.floor);
 * // => { '4': [4.2], '6': [6.1, 6.3] }
 *
 * // The `_.property` iteratee shorthand.
 * _.groupBy(['one', 'two', 'three'], 'length');
 * // => { '3': ['one', 'two'], '5': ['three'] }
 */
//创建聚合函数，function作为setter参数传入
var groupBy = createAggregator(function (result, value, key) {
    if (hasOwnProperty.call(result, key)) {
        result[key].push(value);
    } else {
        baseAssignValue(result, key, [value]);
    }
});

module.exports = groupBy;

```



## head获取数组第一个元素

**head(array)**

**别名：first**

>   Gets the first element of `array`.
>
>   获取数组 `array` 的第一个元素。

### 参数

+   array Array

### 返回

**any**

返回数组第一个元素，如果不存在则返回undefined

### 源码

```js
/**
 * Gets the first element of `array`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @alias first
 * @category Array
 * @param {Array} array The array to query.
 * @returns {*} Returns the first element of `array`.
 * @example
 *
 * _.head([1, 2, 3]);
 * // => 1
 *
 * _.head([]);
 * // => undefined
 */
function head(array) {
  return (array && array.length) ? array[0] : undefined;
}

module.exports = head;

```

## identity返回首个提供的参数

**identity(value)**

>   This method returns the first argument it receives.
>
>   这个方法返回首个提供的参数。

### 参数

+   value any 待返回的值

### 返回

**any**

返回value

### 源码

```js
/**
 * This method returns the first argument it receives.
 *
 * @static
 * @since 0.1.0
 * @memberOf _
 * @category Util
 * @param {*} value Any value.
 * @returns {*} Returns `value`.
 * @example
 *
 * var object = { 'a': 1 };
 *
 * console.log(_.identity(object) === object);
 * // => true
 */
function identity(value) {
  return value;
}

module.exports = identity;

```



## includes集合包含查询

**includes(collection, value, [fromIndex=0])**

>   Checks if `value` is in `collection`. If `collection` is a string, it's checked for a substring of `value`, otherwise `SameValueZero` is used for equality comparisons. If `fromIndex` is negative, it's used as the offset from the end of `collection`.

### 参数

+   collection Array|Object 待查询集合
+   value any 待查询值
+   fromIndex number 可选 开始查询的位置，默认fromIndex=0

### 返回

**boolean**

如果存在value返回true，否则返回false

### 源码

源码中涉及的函数

+   [baseIndexOf](#baseIndexOf)
+   [isArrayLike](#isArrayLike检查类数组)
+   [isString](#isString检查字符串)
+   [toInteger ](#toInteger转为整数)
+   [values](#values对象值数组)

```js
var baseIndexOf = require('./_baseIndexOf'),
    isArrayLike = require('./isArrayLike'),
    isString = require('./isString'),
    toInteger = require('./toInteger'),
    values = require('./values');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Checks if `value` is in `collection`. If `collection` is a string, it's
 * checked for a substring of `value`, otherwise
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * is used for equality comparisons. If `fromIndex` is negative, it's used as
 * the offset from the end of `collection`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object|string} collection The collection to inspect.
 * @param {*} value The value to search for.
 * @param {number} [fromIndex=0] The index to search from.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.reduce`.
 * @returns {boolean} Returns `true` if `value` is found, else `false`.
 * @example
 *
 * _.includes([1, 2, 3], 1);
 * // => true
 *
 * _.includes([1, 2, 3], 1, 2);
 * // => false
 *
 * _.includes({ 'a': 1, 'b': 2 }, 1);
 * // => true
 *
 * _.includes('abcd', 'bc');
 * // => true
 */
function includes(collection, value, fromIndex, guard) {
  collection = isArrayLike(collection) ? collection : values(collection);
  fromIndex = (fromIndex && !guard) ? toInteger(fromIndex) : 0;

  var length = collection.length;
  if (fromIndex < 0) {
    fromIndex = nativeMax(length + fromIndex, 0);
  }
  return isString(collection)
    ? (fromIndex <= length && collection.indexOf(value, fromIndex) > -1)
    : (!!length && baseIndexOf(collection, value, fromIndex) > -1);
}

module.exports = includes;

```



## indexOf数组查询

**indexOf(array, value, [fromIndex=0])**

>   Gets the index at which the first occurrence of `value` is found in `array `  using `SameValueZero` for equality comparisons. If `fromIndex` is negative, it's used as the offset from the end of `array`.
>
>   使用SameValueZero 等值比较，返回首次 value 在数组array中被找到的 索引值， 如果 fromIndex 为负值，将从数组array尾端索引进行匹配。

### 参数

+   array Array 待查询数组
+   value any 待查询值
+   fromIndex number 可选 查询的起始位置

### 返回

**number**

返回value在array中的索引，如果不存在则返回-1

### 源码

源码中涉及的方法

+   [toInteger](#toInteger转为整数)
+   [baseIndexOf](#baseIndexOf) indexOf的基础实现方法

```js
var baseIndexOf = require('./_baseIndexOf'),
    toInteger = require('./toInteger');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Gets the index at which the first occurrence of `value` is found in `array`
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons. If `fromIndex` is negative, it's used as the
 * offset from the end of `array`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @param {number} [fromIndex=0] The index to search from.
 * @returns {number} Returns the index of the matched value, else `-1`.
 * @example
 *
 * _.indexOf([1, 2, 1, 2], 2);
 * // => 1
 *
 * // Search from the `fromIndex`.
 * _.indexOf([1, 2, 1, 2], 2, 2);
 * // => 3
 */
function indexOf(array, value, fromIndex) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return -1;
  }
  var index = fromIndex == null ? 0 : toInteger(fromIndex);
  if (index < 0) {
    index = nativeMax(length + index, 0);
  }
  return baseIndexOf(array, value, index);
}

module.exports = indexOf;

```

## initial获取数组前length-2项

**initial(array)**

>   Gets all but the last element of `array`.
>
>   获取数组`array`中除了最后一个元素之外的所有元素（注：去除数组`array`中的最后一个元素）。

### 参数

+   array Array 待处理数组

### 返回

**Array**

返回截取后的数组

### 示例

示例1

```js
console.log(initial([1, 2, 3]))
//[ 1, 2 ]
```

### 源码

源码中涉及的方法

+   [baseSlice](#baseSlice)

```js
var baseSlice = require('./_baseSlice');

/**
 * Gets all but the last element of `array`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to query.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.initial([1, 2, 3]);
 * // => [1, 2]
 */
function initial(array) {
  var length = array == null ? 0 : array.length;
  return length ? baseSlice(array, 0, -1) : [];
}

module.exports = initial;

```

## intersection数组交集

**intersection([arrays])**

>   Creates an array of unique values that are included in all given arrays using `SameValueZero` for equality comparisons. The order and references of result values are determined by the first array.
>
>   创建唯一值的数组，这个数组包含所有给定数组都包含的元素，使用SameValueZero进行相等性比较。（注：可以理解为给定数组的交集）

### 参数

+   arrays ...Array 可选 待处理的数组

### 返回

**Array**

返回一个包含所有传入数组交集元素的新数组。

### 示例

**示例1**

```js
console.log(intersection([2, 1], [2, 3]))//[ 2 ]
```

**示例2**

```js
console.log(intersection([2, 1]))//[ 2, 1 ]
```

**示例3**

```js
console.log(intersection())//[]
```

### 解析

```js
var arrayMap = require('./_arrayMap'),
    baseIntersection = require('./_baseIntersection'),
    baseRest = require('./_baseRest'),
    castArrayLikeObject = require('./_castArrayLikeObject');

/**
 * Creates an array of unique values that are included in all given arrays
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons. The order and references of result values are
 * determined by the first array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @returns {Array} Returns the new array of intersecting values.
 * @example
 *
 * _.intersection([2, 1], [2, 3]);
 * // => [2]
 */
var intersection = baseRest(function (arrays) {
    //对每一个参数调用castArrayLikeObject，将非数组参数转换为空数组
    var mapped = arrayMap(arrays, castArrayLikeObject);
    //如果mapped[0]!==arrays[0]，说明第一个参数被强制转换为空数组[]，交集为[]
    //否则调用baseIntersection
    return (mapped.length && mapped[0] === arrays[0])
        ? baseIntersection(mapped)
        : [];
});
module.exports = intersection;

```

### 源码

源码中涉及的方法

+   [arrayMap](#arrayMap)
+   [baseIntersection](#baseIntersection)
+   [baseRest](#baseRest)
+   [castArrayLikeObject](#castArrayLikeObject)

```js
var arrayMap = require('./_arrayMap'),
    baseIntersection = require('./_baseIntersection'),
    baseRest = require('./_baseRest'),
    castArrayLikeObject = require('./_castArrayLikeObject');

/**
 * Creates an array of unique values that are included in all given arrays
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons. The order and references of result values are
 * determined by the first array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @returns {Array} Returns the new array of intersecting values.
 * @example
 *
 * _.intersection([2, 1], [2, 3]);
 * // => [2]
 */
var intersection = baseRest(function(arrays) {
  var mapped = arrayMap(arrays, castArrayLikeObject);
  return (mapped.length && mapped[0] === arrays[0])
    ? baseIntersection(mapped)
    : [];
});

module.exports = intersection;

```

## intersectionBy数组迭代处理交集

**intersectionBy([arrays], [iteratee=_.identity])**

>   This method is like `_.intersection` except that it accepts `iteratee` which is invoked for each element of each `arrays` to generate the criterion by which they're compared. The order and references of result values are determined by the first array. The iteratee is invoked with one argument: (value).
>
>   这个方法类似_.intersection，区别是它接受一个 iteratee 调用每一个arrays的每个值以产生一个值，通过产生的值进行了比较。结果值是从第一数组中选择。iteratee 会传入一个参数：(value)。

### 参数

+   arrays ...Arrays 可选待处理的数组
+   iteratee  Function 迭代器 用于处理每个数组的每个值

### 返回

**Array**

返回一个包含所有传入数组交集元素的新数组。

### 示例

示例1

```js
console.log(intersectionBy([2.1, 1.2], [2.3, 3.4], Math.floor))
//[ 2.1 ]
```



示例2

```js
console.log(intersectionBy([{ 'x': 1 }], [{ 'x': 2 }, { 'x': 1 }], 'x'))
//[ { x: 1 } ]
```



### 源码

源码中涉及的方法

+   [arrayMap](#arrayMap)
+   [baseIntersection](#baseIntersection)
+   [baseIteratee](#baseIteratee)
+   [baseRest](#baseRest)
+   [castArrayLikeObject](#castArrayLikeObject)
+   [last](#last获取数组最后一个元素)

```js
var arrayMap = require('./_arrayMap'),
    baseIntersection = require('./_baseIntersection'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    castArrayLikeObject = require('./_castArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.intersection` except that it accepts `iteratee`
 * which is invoked for each element of each `arrays` to generate the criterion
 * by which they're compared. The order and references of result values are
 * determined by the first array. The iteratee is invoked with one argument:
 * (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new array of intersecting values.
 * @example
 *
 * _.intersectionBy([2.1, 1.2], [2.3, 3.4], Math.floor);
 * // => [2.1]
 *
 * // The `_.property` iteratee shorthand.
 * _.intersectionBy([{ 'x': 1 }], [{ 'x': 2 }, { 'x': 1 }], 'x');
 * // => [{ 'x': 1 }]
 */
var intersectionBy = baseRest(function(arrays) {
  var iteratee = last(arrays),
      mapped = arrayMap(arrays, castArrayLikeObject);
  //如果iteratee等于mapped最后一个，说明iteratee是个数组，不存在迭代器
  if (iteratee === last(mapped)) {
    iteratee = undefined;
  } else {
    mapped.pop();
  }
  return (mapped.length && mapped[0] === arrays[0])
    ? baseIntersection(mapped, baseIteratee(iteratee, 2))
    : [];
});

module.exports = intersectionBy;

```

## intersectionWith数组自定义交集

**intersectionWith([arrays], [comparator])**

>   This method is like `_.intersection` except that it accepts `comparator` which is invoked to compare elements of `arrays`. The order and references of result values are determined by the first array. The comparator is invoked with two arguments: (arrVal, othVal).
>
>   这个方法类似_.intersection，区别是它接受一个 comparator 调用比较arrays中的元素。结果值是从第一数组中选择。comparator 会传入两个参数：(arrVal, othVal)。

### 参数

+   arrays ...Arrays 可选 待处理的数组
+   comparator Function 可选 比较器 用于定义相交，即满足比较器条件的值被认为是相等/相交的

### 返回

**Array**

返回一个包含所有传入数组交集元素的新数组。

### 示例

**示例1**

```js
var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
var others = [{ 'x': 1, 'y': 1 }, { 'x': 3, 'y': 2 }];
console.log(intersectionWith(objects, others, (obj1, obj2) => {
    return obj1.x === obj2.x
}))
//[ { x: 1, y: 2 } ]
```

**示例2**

```js
var objects = [{ 'x': 2, 'y': 2 }, { 'x': 1, 'y': 1 }];
var others = [{ 'x': 1, 'y': 3 }, { 'x': 3, 'y': 2 }];
console.log(intersectionWith(objects, others, (obj1, obj2) => {
    return obj1.x == 1
}))
//[ { x: 1, y: 1 } ]
```

结果值是从第一数组中选择

### 源码

```js
var arrayMap = require('./_arrayMap'),
    baseIntersection = require('./_baseIntersection'),
    baseRest = require('./_baseRest'),
    castArrayLikeObject = require('./_castArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.intersection` except that it accepts `comparator`
 * which is invoked to compare elements of `arrays`. The order and references
 * of result values are determined by the first array. The comparator is
 * invoked with two arguments: (arrVal, othVal).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of intersecting values.
 * @example
 *
 * var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
 * var others = [{ 'x': 1, 'y': 1 }, { 'x': 1, 'y': 2 }];
 *
 * _.intersectionWith(objects, others, _.isEqual);
 * // => [{ 'x': 1, 'y': 2 }]
 */
var intersectionWith = baseRest(function(arrays) {
  var comparator = last(arrays),
      mapped = arrayMap(arrays, castArrayLikeObject);

  comparator = typeof comparator == 'function' ? comparator : undefined;
  if (comparator) {
    mapped.pop();
  }
  return (mapped.length && mapped[0] === arrays[0])
    ? baseIntersection(mapped, undefined, comparator)
    : [];
});

module.exports = intersectionWith;

```

## invokeMap集合迭代调用

**invokeMap(collection, path, [args])**

>   Invokes the method at `path` of each element in `collection`, returning an array of the results of each invoked method. Any additional arguments are provided to each invoked method. If `path` is a function, it's invoked for, and `this` bound to, each element in `collection`.

### 参数

+   collection Array|Object 待迭代处理的集合
+   Array|Function|string 用来调用方法的路径 或 者每次迭代调用的函数。
+   args ...any 可选 rest参数 调用每个方法的参数

### 返回

**Array**

返回迭代处理后的结果数组。

### 示例

**示例1**

```js
let res = invokeMap([2, 3, 4, 5], function (item) {
    console.log('item', item);
    console.log('this', this);
    console.log(arguments);
    return 'a'
}, 2222)
console.log(res)
// item 2222
// this[Number: 2]
// [Arguments] { '0': 2222 }
// item 2222
// this[Number: 3]
// [Arguments] { '0': 2222 }
// item 2222
// this[Number: 4]
// [Arguments] { '0': 2222 }
// item 2222
// this[Number: 5]
// [Arguments] { '0': 2222 }
// ['a', 'a', 'a', 'a']
```

**示例2**

```js
let res = invokeMap([2, 3, 4, 5], function (e) {
    return Math.pow(this, e)
}, 2)
console.log(res)
//[ 4, 9, 16, 25 ]
```



### 源码

源码中涉及的函数

+   [apply](#apply)
+   [baseEach](#baseEach)
+   [baseInvoke](#baseInvoke)
+   [baseRest](#baseRest)
+   [isArrayLike](#isArrayLike检查类数组)

```js
var apply = require('./_apply'),
    baseEach = require('./_baseEach'),
    baseInvoke = require('./_baseInvoke'),
    baseRest = require('./_baseRest'),
    isArrayLike = require('./isArrayLike');

/**
 * Invokes the method at `path` of each element in `collection`, returning
 * an array of the results of each invoked method. Any additional arguments
 * are provided to each invoked method. If `path` is a function, it's invoked
 * for, and `this` bound to, each element in `collection`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Array|Function|string} path The path of the method to invoke or
 *  the function invoked per iteration.
 * @param {...*} [args] The arguments to invoke each method with.
 * @returns {Array} Returns the array of results.
 * @example
 *
 * _.invokeMap([[5, 1, 7], [3, 2, 1]], 'sort');
 * // => [[1, 5, 7], [1, 2, 3]]
 *
 * _.invokeMap([123, 456], String.prototype.split, '');
 * // => [['1', '2', '3'], ['4', '5', '6']]
 */
var invokeMap = baseRest(function(collection, path, args) {
  var index = -1,
      isFunc = typeof path == 'function',
      result = isArrayLike(collection) ? Array(collection.length) : [];

  baseEach(collection, function(value) {
    result[++index] = isFunc ? apply(path, value, args) : baseInvoke(value, path, args);
  });
  return result;
});

module.exports = invokeMap;

```



## isArray检查数组

isArray(value)

>   Checks if `value` is classified as an `Array` object.
>
>   检查 value 是否是 Array 类对象。

### 参数

+   value any 待检查的值

### 返回

**boolean**

如果是数组，返回true，否则返回false

### 源码

```js
/**
 * Checks if `value` is classified as an `Array` object.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is an array, else `false`.
 * @example
 *
 * _.isArray([1, 2, 3]);
 * // => true
 *
 * _.isArray(document.body.children);
 * // => false
 *
 * _.isArray('abc');
 * // => false
 *
 * _.isArray(_.noop);
 * // => false
 */
var isArray = Array.isArray;

module.exports = isArray;

```



## isArrayLike检查类数组

> Checks if `value` is array-like. A value is considered array-like if it's not a function and has a `value.length` that's an integer greater than or equal to `0` and less than or equal to `Number.MAX_SAFE_INTEGER`.
>
> 检查 value 是否是类数组。 如果一个值被认为是类数组，那么它不是一个函数，并且value.length是个整数，大于等于 0，小于或等于 Number.MAX_SAFE_INTEGER。

### 参数

+ value 待判定的值

### 返回

**Boolean**

是否是类数组

### 源码

源码中涉及的方法

+ [isLength](#isLength检查有效长度)
+ [isFunction](#isFunction检查函数)

```js

/**
 * Checks if `value` is array-like. A value is considered array-like if it's
 * not a function and has a `value.length` that's an integer greater than or
 * equal to `0` and less than or equal to `Number.MAX_SAFE_INTEGER`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is array-like, else `false`.
 * @example
 *
 * _.isArrayLike([1, 2, 3]);
 * // => true
 *
 * _.isArrayLike(document.body.children);
 * // => true
 *
 * _.isArrayLike('abc');
 * // => true
 *
 * _.isArrayLike(_.noop);
 * // => false
 */
function isArrayLike(value) {
    return value != null && isLength(value.length) && !isFunction(value);
}
```

## isArrayLikeObject检查类数组对象

> This method is like `_.isArrayLike` except that it also checks if `value ` is an object.
>
> 检查是否是类数组对象，在类数组的基础上排除了string
>
> 这个方法类似_.isArrayLike。除了它还检查value是否是个对象。

### 参数

+ value 需要检查的值

### 返回

**boolean**

如果是则返回true

### 源码

源码中涉及的方法

+ [isArrayLike](#isArrayLike检查类数组)

```js
var isArrayLike = require('./isArrayLike'),
    isObjectLike = require('./isObjectLike');

/**
 * This method is like `_.isArrayLike` except that it also checks if `value`
 * is an object.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is an array-like object,
 *  else `false`.
 * @example
 *
 * _.isArrayLikeObject([1, 2, 3]);
 * // => true
 *
 * _.isArrayLikeObject(document.body.children);
 * // => true
 *
 * _.isArrayLikeObject('abc');
 * // => false
 *
 * _.isArrayLikeObject(_.noop);
 * // => false
 */
function isArrayLikeObject(value) {
  return isObjectLike(value) && isArrayLike(value);
}

module.exports = isArrayLikeObject;

```

## isError检查异常

**isError(value)**

>   Checks if `value` is an `Error`, `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, or `URIError` object.
>
>   检查 value 是否是 Error, EvalError, RangeError, ReferenceError,SyntaxError, TypeError, 或者 URIError对象。

### 参数

+   value any 待检查的值

### 返回

**boolean**

如果 value 是一个错误（Error）对象，那么返回 true，否则返回 false。

### 源码

源码中涉及的函数

+   [isObjectLike](#isObjectLike检查类对象)
+   [baseGetTag](#baseGetTag)
+   [isPlainObject](#isPlainObject检查普通对象)

```js
var baseGetTag = require('./_baseGetTag'),
    isObjectLike = require('./isObjectLike'),
    isPlainObject = require('./isPlainObject');

/** `Object#toString` result references. */
var domExcTag = '[object DOMException]',
    errorTag = '[object Error]';

/**
 * Checks if `value` is an `Error`, `EvalError`, `RangeError`, `ReferenceError`,
 * `SyntaxError`, `TypeError`, or `URIError` object.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is an error object, else `false`.
 * @example
 *
 * _.isError(new Error);
 * // => true
 *
 * _.isError(Error);
 * // => false
 */
function isError(value) {
  if (!isObjectLike(value)) {
    return false;
  }
  var tag = baseGetTag(value);
  return tag == errorTag || tag == domExcTag ||
    (typeof value.message == 'string' && typeof value.name == 'string' && !isPlainObject(value));
}

module.exports = isError;

```



## isFunction检查函数

> Checks if `value` is classified as a `Function` object.
>
> 检查' value '是否被分类为' Function '对象。

### 参数

+ value	待检查值

### 返回

**Boolean**

是否是函数方法

### 源码

```js
/**
 * Checks if `value` is classified as a `Function` object.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a function, else `false`.
 * @example
 *
 * _.isFunction(_);
 * // => true
 *
 * _.isFunction(/abc/);
 * // => false
 */
function isFunction(value) {
    if (!isObject(value)) {
        return false;
    }
    // The use of `Object#toString` avoids issues with the `typeof` operator
    // in Safari 9 which returns 'object' for typed arrays and other constructors.
    var tag = baseGetTag(value);
    return tag == funcTag || tag == genTag || tag == asyncTag || tag == proxyTag;
}
```

## isLength检查有效长度

> Checks if `value` is a valid array-like length.
>
> 检查' value '是否是一个有效的类数组长度。

### 参数

+ value	需要检查的值

### 返回

**Boolean**

是否可能是数组长度

### 源码

源码中涉及的常量

+ [MAX_SAFE_INTEGER](#Number)

```js
/**
 * Checks if `value` is a valid array-like length.
 *
 * **Note:** This method is loosely based on
 * [`ToLength`](http://ecma-international.org/ecma-262/7.0/#sec-tolength).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a valid length, else `false`.
 * @example
 *
 * _.isLength(3);
 * // => true
 *
 * _.isLength(Number.MIN_VALUE);
 * // => false
 *
 * _.isLength(Infinity);
 * // => false
 *
 * _.isLength('3');
 * // => false
 */
function isLength(value) {
    return typeof value == 'number' &&
        value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER;
}

```

## isNaN检查NaN

**isNaN(value)**

>   Checks if `value` is `NaN`.
>
>   **Note:** This method is based on [`Number.isNaN`](https://mdn.io/Number/isNaN) and is not the same as global [`isNaN`](https://mdn.io/isNaN) which returns `true` for `undefined` and other non-number values.
>
>   检查 `value` 是否是 `NaN`。
>
>   **注意:** 这个方法基于[`Number.isNaN`](https://mdn.io/Number/isNaN)，和全局的[`isNaN`](https://mdn.io/isNaN) 不同之处在于，全局的[`isNaN`](https://mdn.io/isNaN)对 于 `undefined` 和其他非数字的值返回 `true`。

### 参数

+   value any 待检测的值

### 返回

**boolean**

如果 `value` 是一个 `NaN`，那么返回 `true`，否则返回 `false`。

### 源码

源码中涉及的函数

+   [isNumber](#isNumber检查number)

```js
var isNumber = require('./isNumber');

/**
 * Checks if `value` is `NaN`.
 *
 * **Note:** This method is based on
 * [`Number.isNaN`](https://mdn.io/Number/isNaN) and is not the same as
 * global [`isNaN`](https://mdn.io/isNaN) which returns `true` for
 * `undefined` and other non-number values.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is `NaN`, else `false`.
 * @example
 *
 * _.isNaN(NaN);
 * // => true
 *
 * _.isNaN(new Number(NaN));
 * // => true
 *
 * isNaN(undefined);
 * // => true
 *
 * _.isNaN(undefined);
 * // => false
 */
function isNaN(value) {
  // An `NaN` primitive is the only value that is not equal to itself.
  // Perform the `toStringTag` check first to avoid errors with some
  // ActiveX objects in IE.
  return isNumber(value) && value != +value;
}

module.exports = isNaN;

```

## isNil检查null或者undefined

**isNil(value)**

>   Checks if `value` is `null` or `undefined`.
>
>   检查 `value` 是否是 `null` 或者 `undefined`。

### 参数

+   value any  待检测的值

### 返回

**boolean**

如果value是null或者undefined，返回true，否则返回false

### 示例

**示例1**

```js
console.log(isNil(undefined))//true
console.log(isNil(null))//true
```

### 源码

```js
/**
 * Checks if `value` is `null` or `undefined`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is nullish, else `false`.
 * @example
 *
 * _.isNil(null);
 * // => true
 *
 * _.isNil(void 0);
 * // => true
 *
 * _.isNil(NaN);
 * // => false
 */
function isNil(value) {
    return value == null;
}

module.exports = isNil;

```

## isNull检查null

**isNull(value)**

>   Checks if `value` is `null`.
>
>   检查 `value`alue 是否是 `null`。

### 参数

+   value any 待检测的值

### 返回

**boolean**

如果value是null，返回true，否则返回false

### 源码

```js
/**
 * Checks if `value` is `null`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is `null`, else `false`.
 * @example
 *
 * _.isNull(null);
 * // => true
 *
 * _.isNull(void 0);
 * // => false
 */
function isNull(value) {
  return value === null;
}

module.exports = isNull;

```



## isNumber检查number

**isNumber(value)**

>   Checks if `value` is classified as a `Number` primitive or object.
>
>   **Note:** To exclude `Infinity`, `-Infinity`, and `NaN`, which are
>   classified as numbers, use the `_.isFinite` method.
>
>   检查 `value` 是否是原始`Number`数值型 或者 对象。
>
>   注意: 要排除 Infinity, -Infinity, 以及 NaN 数值类型，用_.isFinite 方法。

### 参数

+   value any 待检测的值

### 返回

**boolean**

如果 `value` 为一个数值，那么返回 `true`，否则返回 `false`。

### 源码

源码中涉及的函数

+   [baseGetTag](#baseGetTag)
+   [isObjectLike](#isObjectLike检查类对象)

```js
var baseGetTag = require('./_baseGetTag'),
    isObjectLike = require('./isObjectLike');

/** `Object#toString` result references. */
var numberTag = '[object Number]';

/**
 * Checks if `value` is classified as a `Number` primitive or object.
 *
 * **Note:** To exclude `Infinity`, `-Infinity`, and `NaN`, which are
 * classified as numbers, use the `_.isFinite` method.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a number, else `false`.
 * @example
 *
 * _.isNumber(3);
 * // => true
 *
 * _.isNumber(Number.MIN_VALUE);
 * // => true
 *
 * _.isNumber(Infinity);
 * // => true
 *
 * _.isNumber('3');
 * // => false
 */
function isNumber(value) {
  return typeof value == 'number' ||
    (isObjectLike(value) && baseGetTag(value) == numberTag);
}

module.exports = isNumber;

```

## isObject检查对象

**isObject(value)**

>   Checks if `value` is the  language type of `Object`. (e.g. arrays, functions, objects, regexes, `new Number(0)`, and `new String('')`)
>
>   检查 value 是否为 Object 的language type。 (例如： arrays, functions, objects, regexes,new Number(0), 以及 new String(''))

### 参数

+   value any 待检查的值

### 返回

**boolean**

如果 value 为一个对象，那么返回 true，否则返回 false。

### 源码

```js
/**
 * Checks if `value` is the
 * [language type](http://www.ecma-international.org/ecma-262/7.0/#sec-ecmascript-language-types)
 * of `Object`. (e.g. arrays, functions, objects, regexes, `new Number(0)`, and `new String('')`)
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is an object, else `false`.
 * @example
 *
 * _.isObject({});
 * // => true
 *
 * _.isObject([1, 2, 3]);
 * // => true
 *
 * _.isObject(_.noop);
 * // => true
 *
 * _.isObject(null);
 * // => false
 */
function isObject(value) {
  var type = typeof value;
  return value != null && (type == 'object' || type == 'function');
}

module.exports = isObject;

```



## isObjectLike检查类对象

**isObjectLike(value)**

>   Checks if `value` is object-like. A value is object-like if it's not `null` and has a `typeof` result of "object".
>
>   检查 value 是否是 类对象。 如果一个值是类对象，那么它不应该是 null，而且 typeof 后的结果是 "object"。

### 参数

+   value any 待检查的值

### 返回

**boolean**

如果 value 为一个类对象，那么返回 true，否则返回 false。

### 源码

```js
/**
 * Checks if `value` is object-like. A value is object-like if it's not `null`
 * and has a `typeof` result of "object".
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is object-like, else `false`.
 * @example
 *
 * _.isObjectLike({});
 * // => true
 *
 * _.isObjectLike([1, 2, 3]);
 * // => true
 *
 * _.isObjectLike(_.noop);
 * // => false
 *
 * _.isObjectLike(null);
 * // => false
 */
function isObjectLike(value) {
  return value != null && typeof value == 'object';
}

module.exports = isObjectLike;

```

## isPlainObject检查普通对象

**isPlainObject(value)**

>   Checks if `value` is a plain object, that is, an object created by the `Object` constructor or one with a `[[Prototype]]` of `null`.
>
>   检查 value 是否是普通对象。 也就是说该对象由 Object 构造函数创建，或者 [[Prototype]] 为 null 。

### 参数

+   value any 待检查的值

### 返回

**boolean**

如果 `value` 为一个普通对象，那么返回 `true`，否则返回 `false`。

### 源码

源码中涉及的函数

+   [baseGetTag](#baseGetTag)
+   [isObjectLike](#isObjectLike检查类对象)
+   [getPrototype](#getPrototype)

```js
var baseGetTag = require('./_baseGetTag'),
    getPrototype = require('./_getPrototype'),
    isObjectLike = require('./isObjectLike');

/** `Object#toString` result references. */
var objectTag = '[object Object]';

/** Used for built-in method references. */
var funcProto = Function.prototype,
    objectProto = Object.prototype;

/** Used to resolve the decompiled source of functions. */
var funcToString = funcProto.toString;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/** Used to infer the `Object` constructor. */
var objectCtorString = funcToString.call(Object);

/**
 * Checks if `value` is a plain object, that is, an object created by the
 * `Object` constructor or one with a `[[Prototype]]` of `null`.
 *
 * @static
 * @memberOf _
 * @since 0.8.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a plain object, else `false`.
 * @example
 *
 * function Foo() {
 *   this.a = 1;
 * }
 *
 * _.isPlainObject(new Foo);
 * // => false
 *
 * _.isPlainObject([1, 2, 3]);
 * // => false
 *
 * _.isPlainObject({ 'x': 0, 'y': 0 });
 * // => true
 *
 * _.isPlainObject(Object.create(null));
 * // => true
 */
function isPlainObject(value) {
  if (!isObjectLike(value) || baseGetTag(value) != objectTag) {
    return false;
  }
  var proto = getPrototype(value);
  if (proto === null) {
    return true;
  }
  var Ctor = hasOwnProperty.call(proto, 'constructor') && proto.constructor;
  return typeof Ctor == 'function' && Ctor instanceof Ctor &&
    funcToString.call(Ctor) == objectCtorString;
}

module.exports = isPlainObject;

```

## isString检查字符串

isString(value)

>   Checks if `value` is classified as a `String` primitive or object.
>
>   检查 `value` 是否是原始字符串`String`或者对象。

### 参数

+   value any 待检查值

### 返回

**boolean**

如果 `value` 为一个字符串，那么返回 `true`，否则返回 `false`。

### 源码

源码中涉及的函数

+   [baseGetTag](#baseGetTag)
+   [isArray](#isArray检查数组)
+   [isObjectLike](#isObjectLike检查类对象)

```js
var baseGetTag = require('./_baseGetTag'),
    isArray = require('./isArray'),
    isObjectLike = require('./isObjectLike');

/** `Object#toString` result references. */
var stringTag = '[object String]';

/**
 * Checks if `value` is classified as a `String` primitive or object.
 *
 * @static
 * @since 0.1.0
 * @memberOf _
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a string, else `false`.
 * @example
 *
 * _.isString('abc');
 * // => true
 *
 * _.isString(1);
 * // => false
 */
function isString(value) {
  return typeof value == 'string' ||
    (!isArray(value) && isObjectLike(value) && baseGetTag(value) == stringTag);
}

module.exports = isString;

```



## isSymbol检查符号

> Checks if `value` is classified as a `Symbol` primitive or object.
>
> 检查 `value` 是否是原始 `Symbol` 或者对象。

### 参数

+ value	需要检查的值

### 返回

**Boolean**

### 源码

```js
    /**
     * Checks if `value` is classified as a `Symbol` primitive or object.
     *
     * @static
     * @memberOf _
     * @since 4.0.0
     * @category Lang
     * @param {*} value The value to check.
     * @returns {boolean} Returns `true` if `value` is a symbol, else `false`.
     * @example
     *
     * _.isSymbol(Symbol.iterator);
     * // => true
     *
     * _.isSymbol('abc');
     * // => false
     */
    function isSymbol(value) {
      return typeof value == 'symbol' ||
        (isObjectLike(value) && baseGetTag(value) == symbolTag);
    }

```


## join数组以指定分隔符转为字符串

**join(array, [separator=','])**

>   Converts all elements in `array` into a string separated by `separator`.
>
>   将 array 中的所有元素转换为由 separator 分隔的字符串。

### 参数

+   array Array 待转化的字符串
+   separator string 可选 分隔符 默认separator=','

### 返回

**string**

返回转化后的字符串

### 源码

```js
/** Used for built-in method references. */
var arrayProto = Array.prototype;

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeJoin = arrayProto.join;

/**
 * Converts all elements in `array` into a string separated by `separator`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to convert.
 * @param {string} [separator=','] The element separator.
 * @returns {string} Returns the joined string.
 * @example
 *
 * _.join(['a', 'b', 'c'], '~');
 * // => 'a~b~c'
 */
function join(array, separator) {
  return array == null ? '' : nativeJoin.call(array, separator);
}

module.exports = join;

```

## keyBy集合迭代生成键

**keyBy(collection, [iteratee=_.identity])**

>   Creates an object composed of keys generated from the results of running each element of `collection` thru `iteratee`. The corresponding value of each key is the last element responsible for generating the key. The iteratee is invoked with one argument: (value).
>
>   创建一个对象组成， key（键） 是 `collection`（集合）中的每个元素经过 `iteratee`（迭代函数） 处理后返回的结果。 每个 key（键）对应的值是生成key（键）的最后一个元素。`iteratee`（迭代函数）调用1个参数：*(value)*。

### 参数

+   collection Array|Object 待操作的集合
+   iteratee Array|Function|Object|string 可选 这个迭代函数用来转换key。

### 返回

**Object**

返回一个组成聚合的对象。

### 示例

**示例1**

```js
var array = [
    { 'dir': 'left', 'code': 97 },
    { 'dir': 'right', 'code': 100 }
];
console.log(keyBy(array, 'dir'))
//{ left: { dir: 'left', code: 97 }, right: { dir: 'right', code: 100 } }
```

**示例2**

```js
let array = [
    { key: 'a', value: 0 },
    { key: 'a', value: 1 },
    { key: 'b', value: 2 }
]
console.log(keyBy(array, 'key'))
//{ a: { key: 'a', value: 1 }, b: { key: 'b', value: 2 } }
```



### 源码

源码中涉及的函数

+   [baseAssignValue](#baseAssignValue)
+   [createAggregator](#createAggregator)

```js
var baseAssignValue = require('./_baseAssignValue'),
    createAggregator = require('./_createAggregator');

/**
 * Creates an object composed of keys generated from the results of running
 * each element of `collection` thru `iteratee`. The corresponding value of
 * each key is the last element responsible for generating the key. The
 * iteratee is invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The iteratee to transform keys.
 * @returns {Object} Returns the composed aggregate object.
 * @example
 *
 * var array = [
 *   { 'dir': 'left', 'code': 97 },
 *   { 'dir': 'right', 'code': 100 }
 * ];
 *
 * _.keyBy(array, function(o) {
 *   return String.fromCharCode(o.code);
 * });
 * // => { 'a': { 'dir': 'left', 'code': 97 }, 'd': { 'dir': 'right', 'code': 100 } }
 *
 * _.keyBy(array, 'dir');
 * // => { 'left': { 'dir': 'left', 'code': 97 }, 'right': { 'dir': 'right', 'code': 100 } }
 */
var keyBy = createAggregator(function(result, value, key) {
  baseAssignValue(result, key, value);
});

module.exports = keyBy;

```



## keys对象属性名数组

**keys(object)**

>   Creates an array of the own enumerable property names of `object`.
>
>   创建一个 `object` 的自身可枚举属性名为数组。

**注意:** 非对象的值会强制转换为对象。

### 参数

+   object Object 待操作对象

### 返回

**Array**

返回由对象可枚举属性的属性名构成的数组

### 源码

源码中涉及的函数

+   [arrayLikeKeys](#arrayLikeKeys)
+   [baseKeys](#baseKeys)
+   [isArrayLike](#isArrayLike检查类数组)

```js
var arrayLikeKeys = require('./_arrayLikeKeys'),
    baseKeys = require('./_baseKeys'),
    isArrayLike = require('./isArrayLike');

/**
 * Creates an array of the own enumerable property names of `object`.
 *
 * **Note:** Non-object values are coerced to objects. See the
 * [ES spec](http://ecma-international.org/ecma-262/7.0/#sec-object.keys)
 * for more details.
 *
 * @static
 * @since 0.1.0
 * @memberOf _
 * @category Object
 * @param {Object} object The object to query.
 * @returns {Array} Returns the array of property names.
 * @example
 *
 * function Foo() {
 *   this.a = 1;
 *   this.b = 2;
 * }
 *
 * Foo.prototype.c = 3;
 *
 * _.keys(new Foo);
 * // => ['a', 'b'] (iteration order is not guaranteed)
 *
 * _.keys('hi');
 * // => ['0', '1']
 */
function keys(object) {
  return isArrayLike(object) ? arrayLikeKeys(object) : baseKeys(object);
}

module.exports = keys;

```



## last获取数组最后一个元素

**last(array)**

>   Gets the last element of `array`.
>
>   获取数组最后一个元素

### 参数

+   array Array 待处理的数组

### 返回

**any**

返回数组的最后一个元素

### 源码

```js
/**
 * Gets the last element of `array`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to query.
 * @returns {*} Returns the last element of `array`.
 * @example
 *
 * _.last([1, 2, 3]);
 * // => 3
 */
function last(array) {
  var length = array == null ? 0 : array.length;
  return length ? array[length - 1] : undefined;
}

module.exports = last;

```

## lastIndexOf数组倒序查询

**lastIndexOf(array, value, [fromIndex=array.length-1])**

>   This method is like `_.indexOf` except that it iterates over elements of `array` from right to left.
>
>   这个方法类似_.indexOf ，区别是它是从右到左遍历array的元素。

### 参数

+   array Array 待查询的数组
+   value any 待查询的值
+   fromIndex number 可选 查询的起始位置 默认fromIndex=array.length-1

### 返回

**number**

如果存在匹配的元素，返回这个元素的索引，否则返回-1

### 源码

源码中涉及的方法

+   [baseFindIndex](#baseFindIndex)
+   [baseIsNaN](#baseIsNaN)
+   [strictLastIndexOf](#strictLastIndexOf)
+   [toInteger](#toInteger转为整数)

```js
var baseFindIndex = require('./_baseFindIndex'),
    baseIsNaN = require('./_baseIsNaN'),
    strictLastIndexOf = require('./_strictLastIndexOf'),
    toInteger = require('./toInteger');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max,
    nativeMin = Math.min;

/**
 * This method is like `_.indexOf` except that it iterates over elements of
 * `array` from right to left.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @param {number} [fromIndex=array.length-1] The index to search from.
 * @returns {number} Returns the index of the matched value, else `-1`.
 * @example
 *
 * _.lastIndexOf([1, 2, 1, 2], 2);
 * // => 3
 *
 * // Search from the `fromIndex`.
 * _.lastIndexOf([1, 2, 1, 2], 2, 2);
 * // => 1
 */
function lastIndexOf(array, value, fromIndex) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return -1;
  }
  var index = length;
  if (fromIndex !== undefined) {
    index = toInteger(fromIndex);
    index = index < 0 ? nativeMax(length + index, 0) : nativeMin(index, length - 1);
  }
  return value === value
    ? strictLastIndexOf(array, value, index)
    : baseFindIndex(array, baseIsNaN, index, true);
}

module.exports = lastIndexOf;

```

## map集合遍历

**map(collection, [iteratee=_.identity])**

>   Creates an array of values by running each element in `collection` thru `iteratee`. The iteratee is invoked with three arguments: (value, index|key, collection).
>
>   创建一个数组， value（值） 是 `iteratee`（迭代函数）遍历 `collection`（集合）中的每个元素后返回的结果。 iteratee（迭代函数）调用3个参数：*(value, index|key, collection)*.

### 参数

+   collection Array|Object 待遍历的集合
+   iteratee Array|Function|Object|string 可选 迭代器 每次遍历调用

### 返回

**Array**

返回新的处理后的数组

### 源码

源码中涉及的函数

+   [arrayMap](#arrayMap)
+   [baseIteratee](#baseIteratee)
+   [baseMap](#baseMap)
+   [isArray](#isArray检查数组)

```js
var arrayMap = require('./_arrayMap'),
    baseIteratee = require('./_baseIteratee'),
    baseMap = require('./_baseMap'),
    isArray = require('./isArray');

/**
 * Creates an array of values by running each element in `collection` thru
 * `iteratee`. The iteratee is invoked with three arguments:
 * (value, index|key, collection).
 *
 * Many lodash methods are guarded to work as iteratees for methods like
 * `_.every`, `_.filter`, `_.map`, `_.mapValues`, `_.reject`, and `_.some`.
 *
 * The guarded methods are:
 * `ary`, `chunk`, `curry`, `curryRight`, `drop`, `dropRight`, `every`,
 * `fill`, `invert`, `parseInt`, `random`, `range`, `rangeRight`, `repeat`,
 * `sampleSize`, `slice`, `some`, `sortBy`, `split`, `take`, `takeRight`,
 * `template`, `trim`, `trimEnd`, `trimStart`, and `words`
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new mapped array.
 * @example
 *
 * function square(n) {
 *   return n * n;
 * }
 *
 * _.map([4, 8], square);
 * // => [16, 64]
 *
 * _.map({ 'a': 4, 'b': 8 }, square);
 * // => [16, 64] (iteration order is not guaranteed)
 *
 * var users = [
 *   { 'user': 'barney' },
 *   { 'user': 'fred' }
 * ];
 *
 * // The `_.property` iteratee shorthand.
 * _.map(users, 'user');
 * // => ['barney', 'fred']
 */
function map(collection, iteratee) {
  var func = isArray(collection) ? arrayMap : baseMap;
  return func(collection, baseIteratee(iteratee, 3));
}

module.exports = map;

```

## maen数组均值计算

**mean(array)**

>   Computes the mean of the values in `array`.
>
>   计算 `array` 的平均值。

### 参数

+   array Array 待计算的数组

### 返回

**number**

返回计算后的均值

### 示例

**示例1**

```js
console.log(mean([4, 2, 8, 6]))//5
```

**示例2**

```js
console.log(mean([true, false]))//0.5
```

**示例3**

```js
console.log(mean(['a', 'b']))//NaN
```

### 源码

源码中涉及的函数

+   [baseMean](#baseMean)
+     [identity](#identity返回首个提供的参数)

```js
var baseMean = require('./_baseMean'),
    identity = require('./identity');

/**
 * Computes the mean of the values in `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Math
 * @param {Array} array The array to iterate over.
 * @returns {number} Returns the mean.
 * @example
 *
 * _.mean([4, 2, 8, 6]);
 * // => 5
 */
function mean(array) {
  return baseMean(array, identity);
}

module.exports = mean;

```

## memoize缓存化函数

**memoize(func, [resolver])**

>   Creates a function that memoizes the result of `func`. If `resolver` is provided, it determines the cache key for storing the result based on the arguments provided to the memoized function. By default, the first argument provided to the memoized function is used as the map cache key. The `func` is invoked with the `this` binding of the memoized function.
>
>   **Note:** The cache is exposed as the `cache` property on the memoized function. Its creation may be customized by replacing the `_.memoize.Cache` constructor with one whose instances implement the [`Map`](http://ecma-international.org/ecma-262/7.0/#sec-properties-of-the-map-prototype-object) method interface of `clear`, `delete`, `get`, `has`, and `set`.
>
>   创建一个会缓存 `func` 结果的函数。 如果提供了 `resolver` ，就用 resolver 的返回值作为 key 缓存函数的结果。 默认情况下用第一个参数作为缓存的 key。 `func` 在调用时 `this` 会绑定在缓存函数上。
>
>   **注意**: 缓存会暴露在缓存函数的 `cache` 上。 它是可以定制的，只要替换了 `_.memoize.Cache` 构造函数，或实现了[`Map`](http://ecma-international.org/ecma-262/6.0/#sec-properties-of-the-map-prototype-object) 的 `delete`, `get`, `has`, 和 `set`方法。

### 参数

+   func Function 需要缓存化的函数
+   resolver Function 可选 这个函数的返回值作为缓存的 key

### 返回

**Function**

返回缓存化后的函数

### 示例

**示例1**

```js
var func = memoize(function (s) {
    console.log('s', s)
    return s
}, function (...rest) {
    console.log('rest', rest)
    return 'a'
})
var res = ['a', 'b', 'c'].map(val => func(val))
console.log(func.cache.get('a'))//a
//rest [ 'a' ]
//s a
//rest [ 'b' ]
//rest [ 'c' ]
//a
```

**示例2**

```js
var func = memoize(function (s) {
    return s
})
var res = ['a', 'b', 'c'].map(val => func(val))
console.log(func.cache.get('a'))//a
console.log(func.cache.get('b'))//b
console.log(func.cache.get('c'))//c
```



### 解析

```js
var MapCache = require('./_MapCache');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';


function memoize(func, resolver) {
  if (typeof func != 'function' || (resolver != null && typeof resolver != 'function')) {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  var memoized = function() {
    var args = arguments,//获取参数
        //获取缓存键，如果没有resolver则以第一个参数作为键
        key = resolver ? resolver.apply(this, args) : args[0],
        cache = memoized.cache;
	//如果缓存中已经有了结果，直接返回缓存的结果
    if (cache.has(key)) {
      return cache.get(key);
    }
    //调用func获取结果并缓存值
    var result = func.apply(this, args);
    memoized.cache = cache.set(key, result) || cache;
    return result;
  };
  memoized.cache = new (memoize.Cache || MapCache);
  return memoized;
}

// Expose `MapCache`.
memoize.Cache = MapCache;

module.exports = memoize;

```



### 源码

源码中涉及到的函数

```js
var MapCache = require('./_MapCache');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * Creates a function that memoizes the result of `func`. If `resolver` is
 * provided, it determines the cache key for storing the result based on the
 * arguments provided to the memoized function. By default, the first argument
 * provided to the memoized function is used as the map cache key. The `func`
 * is invoked with the `this` binding of the memoized function.
 *
 * **Note:** The cache is exposed as the `cache` property on the memoized
 * function. Its creation may be customized by replacing the `_.memoize.Cache`
 * constructor with one whose instances implement the
 * [`Map`](http://ecma-international.org/ecma-262/7.0/#sec-properties-of-the-map-prototype-object)
 * method interface of `clear`, `delete`, `get`, `has`, and `set`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to have its output memoized.
 * @param {Function} [resolver] The function to resolve the cache key.
 * @returns {Function} Returns the new memoized function.
 * @example
 *
 * var object = { 'a': 1, 'b': 2 };
 * var other = { 'c': 3, 'd': 4 };
 *
 * var values = _.memoize(_.values);
 * values(object);
 * // => [1, 2]
 *
 * values(other);
 * // => [3, 4]
 *
 * object.a = 2;
 * values(object);
 * // => [1, 2]
 *
 * // Modify the result cache.
 * values.cache.set(object, ['a', 'b']);
 * values(object);
 * // => ['a', 'b']
 *
 * // Replace `_.memoize.Cache`.
 * _.memoize.Cache = WeakMap;
 */
function memoize(func, resolver) {
  if (typeof func != 'function' || (resolver != null && typeof resolver != 'function')) {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  var memoized = function() {
    var args = arguments,
        key = resolver ? resolver.apply(this, args) : args[0],
        cache = memoized.cache;

    if (cache.has(key)) {
      return cache.get(key);
    }
    var result = func.apply(this, args);
    memoized.cache = cache.set(key, result) || cache;
    return result;
  };
  memoized.cache = new (memoize.Cache || MapCache);
  return memoized;
}

// Expose `MapCache`.
memoize.Cache = MapCache;

module.exports = memoize;

```



## multiply两数相乘

**multiply(multiplier, multiplicand)**

>   Multiply two numbers.
>
>   两数相乘

### 参数

+   multiplier number 乘数
+   multiplicand number 被乘数

### 返回

**number**

计算后的结果

### 源码

源码中涉及的函数

+   [createMathOperation](#createMathOperation)

```js
var createMathOperation = require('./_createMathOperation');

/**
 * Multiply two numbers.
 *
 * @static
 * @memberOf _
 * @since 4.7.0
 * @category Math
 * @param {number} multiplier The first number in a multiplication.
 * @param {number} multiplicand The second number in a multiplication.
 * @returns {number} Returns the product.
 * @example
 *
 * _.multiply(6, 4);
 * // => 24
 */
var multiply = createMathOperation(function(multiplier, multiplicand) {
  return multiplier * multiplicand;
}, 1);

module.exports = multiply;

```

## negate创建取反函数

**negate(predicate)**

>   Creates a function that negates the result of the predicate `func`. The `func` predicate is invoked with the `this` binding and arguments of the created function.
>
>   创建一个针对断言函数 `func` 结果取反的函数。 `func` 断言函数被调用的时候，`this` 绑定到创建的函数，并传入对应参数。

### 参数

+   predicate Function 需要对结果取反的函数

### 返回

**Function**

返回一个新的取反函数。

### 示例

**示例1**

```js
function isEven(n) {
    return n % 2 == 0;
}
console.log([1, 2, 3, 4, 5, 6].filter(negate(isEven)))
//[ 1, 3, 5 ]
```



### 源码

```js
/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * Creates a function that negates the result of the predicate `func`. The
 * `func` predicate is invoked with the `this` binding and arguments of the
 * created function.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {Function} predicate The predicate to negate.
 * @returns {Function} Returns the new negated function.
 * @example
 *
 * function isEven(n) {
 *   return n % 2 == 0;
 * }
 *
 * _.filter([1, 2, 3, 4, 5, 6], _.negate(isEven));
 * // => [1, 3, 5]
 */
function negate(predicate) {
  if (typeof predicate != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  return function() {
    var args = arguments;
    switch (args.length) {
      case 0: return !predicate.call(this);
      case 1: return !predicate.call(this, args[0]);
      case 2: return !predicate.call(this, args[0], args[1]);
      case 3: return !predicate.call(this, args[0], args[1], args[2]);
    }
    return !predicate.apply(this, args);
  };
}

module.exports = negate;

```

## now获取Unix纪元至今毫秒数

**now()**

>   Gets the timestamp of the number of milliseconds that have elapsed since the Unix epoch (1 January 1970 00:00:00 UTC).
>
>   获得 Unix 纪元 *(1 January `1970 00`:00:00 UTC)* 直到现在的毫秒数。

### 返回

**number**

返回时间戳。

### 源码

```js
var root = require('./_root');

/**
 * Gets the timestamp of the number of milliseconds that have elapsed since
 * the Unix epoch (1 January 1970 00:00:00 UTC).
 *
 * @static
 * @memberOf _
 * @since 2.4.0
 * @category Date
 * @returns {number} Returns the timestamp.
 * @example
 *
 * _.defer(function(stamp) {
 *   console.log(_.now() - stamp);
 * }, _.now());
 * // => Logs the number of milliseconds it took for the deferred invocation.
 */
var now = function () {
    return root.Date.now();
};
module.exports = now;

```



## nth数组获取指定索引元素

**nth(array, [n=0])**

>    Gets the element at index `n` of `array`. If `n` is negative, the nth element from the end is returned.
>
>   获取array数组的第n个元素。如果n为负数，则返回从数组结尾开始的第n个元素。

### 参数

+   array Array 待处理数组
+   n number 可选 索引 默认n=0

### 返回

**any**

返回数组的第n个元素

### 源码

源码中涉及的方法

+   [toInteger](#toInteger转为整数)
+   [baseNth](#baseNth)

```js
var baseNth = require('./_baseNth'),
    toInteger = require('./toInteger');

/**
 * Gets the element at index `n` of `array`. If `n` is negative, the nth
 * element from the end is returned.
 *
 * @static
 * @memberOf _
 * @since 4.11.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {number} [n=0] The index of the element to return.
 * @returns {*} Returns the nth element of `array`.
 * @example
 *
 * var array = ['a', 'b', 'c', 'd'];
 *
 * _.nth(array, 1);
 * // => 'b'
 *
 * _.nth(array, -2);
 * // => 'c';
 */
function nth(array, n) {
  return (array && array.length) ? baseNth(array, toInteger(n)) : undefined;
}

module.exports = nth;

```



## omit 对象属性过滤

**omit(object,[props])**

### 参数

+ **object**	源对象，等待过滤的对象
+ **[props]**   属性字符串数组，需要被过滤掉的属性

### 返回

**object**

返回一个源对象的浅拷贝，内部属性不包含[props]中的属性

### 示例1——基础用法

```tsx
import { omit } from 'lodash';
const obj_1 = {
    a: '1',
    b: 2,
    c: true,
    d: {
        a: 2
    },
}
const obj_2 = omit(obj_1, ['a', 'b'])
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { a: 2 } }
console.log('obj_1', obj_1)
//obj_1 { a: '1', b: 2, c: true, d: { a: 2 } }
obj_1.d.a = 3
obj_1.c = true
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { a: 3 } }
```

由示例1可以看出使用omit过滤生成的对象是原来对象的一个**浅拷贝**

### 示例2——使用JSON完成深拷贝

```tsx
import { omit } from 'lodash';
const obj_1 = {
    a: '1',
    b: 2,
    c: true,
    d: {
        a: 2
    },
}
const obj_2 = JSON.parse(JSON.stringify(omit(obj_1, ['a', 'b'])))
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { a: 2 } }
console.log('obj_1', obj_1)
//obj_1 { a: '1', b: 2, c: true, d: { a: 2 } }
obj_1.d.a = 3
obj_1.c = false
console.log('obj_2', obj_2)
obj_2 { c: true, d: { a: 2 } }
```

在omit外包装上JSON的两个转化方法，可以深拷贝一份omit过滤后的对象

### 示例3——带路径的属性

首先是一个普通用例

```tsx
import { omit } from 'lodash';
const obj_1 = {
    a: '1',
    b: 2,
    c: true,
    d: {
        a: 2,
        b: 3,
        c: {
            a: 5,
        }
    },
}
const obj_2 = omit(obj_1, ['a', 'b'])
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { a: 2, b: 3, c: { a: 5 } } }
console.log('obj_1', obj_1)
//obj_1 { a: '1', b: 2, c: true, d: { a: 2, b: 3, c: { a: 5 } } }
obj_1.d.b = 4
obj_1.d.c.a = 4
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { a: 2, b: 4, c: { a: 4 } } }
```

从打印结果中发现，obj_2.d是obj_1.d的引用

再来下一个例子

```tsx
import { omit } from 'lodash';
const obj_1 = {
    a: '1',
    b: 2,
    c: true,
    d: {
        a: 2,
        b: 3,
        c: {
            a: 5,
        }
    },
}
const obj_2 = omit(obj_1, ['a', 'b', 'd.a'])
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { b: 3, c: { a: 5 } } }
console.log('obj_1', obj_1)
//obj_1 { a: '1', b: 2, c: true, d: { a: 2, b: 3, c: { a: 5 } } }
obj_1.d.b = 4
obj_1.d.c.a = 4
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { b: 3, c: { a: 5 } } }
```

首先，使用 **.** 是可以过滤子对象的属性的

但是当我们修改

> obj_1.d.b = 4
> obj_1.d.c.a = 4

发现obj_2相对应的属性值并没有被修改，显然此时obj_2.d已经是不同于obj_1.d新对象了，并且其子对象也并不是与obj_1指向同一个地址，此时obj_2的子对象d是obj_1.d的深拷贝

### 解析

```js
/**
* The opposite of `_.pick`; this method creates an object composed of the
* own and inherited enumerable property paths of `object` that are not omitted.
*
* **Note:** This method is considerably slower than `_.pick`.
*
* @static
* @since 0.1.0
* @memberOf _
* @category Object
* @param {Object} object The source object.
* @param {...(string|string[])} [paths] The property paths to omit.
* @returns {Object} Returns the new object.
* @example
*
* var object = { 'a': 1, 'b': '2', 'c': 3 };
*
* _.omit(object, ['a', 'c']);
* // => { 'b': '2' }
*/
var omit = flatRest(function (object, paths) {
    var result = {};
    if (object == null) {
        return result;
    }
    var isDeep = false;
    paths = arrayMap(paths, function (path) {
        path = castPath(path, object);
        //如果path是属性路径如a.b 形式，则isDeep=true
        console.log('path', path)
         //path [ 'a' ]
		//path [ 'b' ]
		//path [ 'd', 'a' ]
        isDeep || (isDeep = path.length > 1);
        return path;
    });
    console.log('paths', paths)
    //paths [ [ 'a' ], [ 'b' ], [ 'd', 'a' ] ]
    console.log('isDeep', isDeep)
    //isDeep true
    //拷贝object对象的所有键值
    copyObject(object, getAllKeysIn(object), result);
    if (isDeep) {
        result = baseClone(result, CLONE_DEEP_FLAG | CLONE_FLAT_FLAG | CLONE_SYMBOLS_FLAG, customOmitClone);
    }
    console.log('result', result)
    //result { a: '1', b: 2, c: true, d: { a: 2, b: 3, c: { a: 5 } } }
    var length = paths.length;
    while (length--) {
        baseUnset(result, paths[length]);
        console.log(`result——${length}`, result)
         //result——2 { a: '1', b: 2, c: true, d: { b: 3, c: { a: 5 } } }
		//result——1 { a: '1', c: true, d: { b: 3, c: { a: 5 } } }
		//result——0 { c: true, d: { b: 3, c: { a: 5 } } }
    }
    return result;
});
const obj_1 = {
    a: '1',
    b: 2,
    c: true,
    d: {
        a: 2,
        b: 3,
        c: {
            a: 5,
        }
    },
}
const obj_2 = omit(obj_1, ['a', 'b', 'd.a'])
console.log('obj_2', obj_2)
//obj_2 { c: true, d: { b: 3, c: { a: 5 } } }
```

### 源码

源码内涉及的方法

+ [flatRest](#flatRest)
+ [arrayMap](#arrayMap)
+ [castPath](#castPath)
+ [getAllKeysIn](#getAllKeysIn)
+ [copyObject](#copyObject)

```js
var arrayMap = require('./_arrayMap'),
    baseClone = require('./_baseClone'),
    baseUnset = require('./_baseUnset'),
    castPath = require('./_castPath'),
    copyObject = require('./_copyObject'),
    customOmitClone = require('./_customOmitClone'),
    flatRest = require('./_flatRest'),
    getAllKeysIn = require('./_getAllKeysIn');

/** Used to compose bitmasks for cloning. */
var CLONE_DEEP_FLAG = 1,
    CLONE_FLAT_FLAG = 2,
    CLONE_SYMBOLS_FLAG = 4;

/**
 * The opposite of `_.pick`; this method creates an object composed of the
 * own and inherited enumerable property paths of `object` that are not omitted.
 *
 * **Note:** This method is considerably slower than `_.pick`.
 *
 * @static
 * @since 0.1.0
 * @memberOf _
 * @category Object
 * @param {Object} object The source object.
 * @param {...(string|string[])} [paths] The property paths to omit.
 * @returns {Object} Returns the new object.
 * @example
 *
 * var object = { 'a': 1, 'b': '2', 'c': 3 };
 *
 * _.omit(object, ['a', 'c']);
 * // => { 'b': '2' }
 */
var omit = flatRest(function(object, paths) {
  var result = {};
  if (object == null) {
    return result;
  }
  var isDeep = false;
  paths = arrayMap(paths, function(path) {
    path = castPath(path, object);
    isDeep || (isDeep = path.length > 1);
    return path;
  });
  copyObject(object, getAllKeysIn(object), result);
  if (isDeep) {
    result = baseClone(result, CLONE_DEEP_FLAG | CLONE_FLAT_FLAG | CLONE_SYMBOLS_FLAG, customOmitClone);
  }
  var length = paths.length;
  while (length--) {
    baseUnset(result, paths[length]);
  }
  return result;
});

module.exports = omit;
```

## once函数单次调用

**once(func)**

>   Creates a function that is restricted to invoking `func` once. Repeat calls  to the function return the value of the first invocation. The `func` is invoked with the `this` binding and arguments of the created function.
>
>   创建一个只能调用 `func` 一次的函数。 重复调用返回第一次调用的结果。 `func` 调用时， `this` 绑定到创建的函数，并传入对应参数。

### 参数

+   func Function 待包装的函数

### 返回

**Function**

返回新的只能调用一次的函数

### 示例

**示例1**

```js
var count = 0
const func = once((x) => {
    count++
    console.log('count=', count)
    console.log('x*x=', x * x)
    return x * x
})
let arr = [func(2), func(3), func(4)]
console.log(arr)
//count= 1
//x*x= 4
//[ 4, 4, 4 ]
```



### 源码

源码中涉及到的函数

+   [before](#before限次触发)

```js
var before = require('./before');

/**
 * Creates a function that is restricted to invoking `func` once. Repeat calls
 * to the function return the value of the first invocation. The `func` is
 * invoked with the `this` binding and arguments of the created function.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to restrict.
 * @returns {Function} Returns the new restricted function.
 * @example
 *
 * var initialize = _.once(createApplication);
 * initialize();
 * initialize();
 * // => `createApplication` is invoked once
 */
function once(func) {
  return before(2, func);
}

module.exports = once;

```



## orderBy集合迭代结果指定排序

**orderBy(collection, [iteratees=[_.identity]], [orders])**

>   This method is like `_.sortBy` except that it allows specifying the sort orders of the iteratees to sort by. If `orders` is unspecified, all values are sorted in ascending order. Otherwise, specify an order of "desc" for descending or "asc" for ascending sort order of corresponding values.
>
>   此方法类似于_.sortBy，除了它允许指定 iteratee（迭代函数）结果如何排序。 如果没指定 orders（排序），所有值以升序排序。 否则，指定为"desc" 降序，或者指定为 "asc" 升序，排序对应值。

### 参数

+   collection Array|Object 待操作的集合
+   iteratees Array[]|Function[]|Object[]|string[] 可选 排序的迭代函数 迭代器集合
+   orders string[] 迭代函数的排序顺序

### 返回

**Array**

返回排序后的新数组

### 示例

**示例1**

按“用户”升序排序，按“年龄”降序排序

```js
var users = [
    { 'user': 'fred', 'age': 48 },
    { 'user': 'barney', 'age': 34 },
    { 'user': 'fred', 'age': 40 },
    { 'user': 'barney', 'age': 36 }
];
console.log(orderBy(users, ['user', 'age'], ['asc', 'desc']))
// [
//     { user: 'barney', age: 36 },
//     { user: 'barney', age: 34 },
//     { user: 'fred', age: 48 },
//     { user: 'fred', age: 40 }
// ]
```

### 源码

源码中涉及的函数

+   [baseOrderBy](#baseOrderBy)
+   [isArray](#isArray检查数组)

```js
var baseOrderBy = require('./_baseOrderBy'),
    isArray = require('./isArray');

/**
 * This method is like `_.sortBy` except that it allows specifying the sort
 * orders of the iteratees to sort by. If `orders` is unspecified, all values
 * are sorted in ascending order. Otherwise, specify an order of "desc" for
 * descending or "asc" for ascending sort order of corresponding values.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Array[]|Function[]|Object[]|string[]} [iteratees=[_.identity]]
 *  The iteratees to sort by.
 * @param {string[]} [orders] The sort orders of `iteratees`.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.reduce`.
 * @returns {Array} Returns the new sorted array.
 * @example
 *
 * var users = [
 *   { 'user': 'fred',   'age': 48 },
 *   { 'user': 'barney', 'age': 34 },
 *   { 'user': 'fred',   'age': 40 },
 *   { 'user': 'barney', 'age': 36 }
 * ];
 *
 * // Sort by `user` in ascending order and by `age` in descending order.
 * _.orderBy(users, ['user', 'age'], ['asc', 'desc']);
 * // => objects for [['barney', 36], ['barney', 34], ['fred', 48], ['fred', 40]]
 */
function orderBy(collection, iteratees, orders, guard) {
  if (collection == null) {
    return [];
  }
  if (!isArray(iteratees)) {
    iteratees = iteratees == null ? [] : [iteratees];
  }
  orders = guard ? undefined : orders;
  if (!isArray(orders)) {
    orders = orders == null ? [] : [orders];
  }
  return baseOrderBy(collection, iteratees, orders);
}

module.exports = orderBy;

```

## overArgs函数参数覆写

**overArgs(func, [transforms=[_.identity]])**

>   Creates a function that invokes `func` with its arguments transformed.
>
>   创建一个函数，调用`func`时参数为相对应的`transforms`的返回值。

### 参数

+   func Function 待包装函数
+   transforms Array[Function]|Function 可选 对func的参数进行处理的函数，返回值将作为func的新参数

### 返回

**Function**

返回新的函数

### 示例

示例1

```js
var func = overArgs(function (x, y) {
    return [x, y];
}, [(n) => n * n, (n) => 3 * n]);
console.log(func(9, 3));
//[81, 9]
```

示例2

```js
var func = overArgs(function (x, y) {
    return [x, y];
}, [(n) => n * n]);
console.log(func(9, 3));
// [81, 3]
```



示例3

```js
var func = overArgs(function (x, y) {
    return [x, y];
}, (n) => n * n);
console.log(func(9, 3));
// [81, 3]
```



### 解析

```js
var apply = require('./_apply'),
    arrayMap = require('./_arrayMap'),
    baseFlatten = require('./_baseFlatten'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    baseUnary = require('./_baseUnary'),
    castRest = require('./_castRest'),
    isArray = require('./isArray');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMin = Math.min;
var overArgs = castRest(function(func, transforms) {
  //如果transforms长度为一，并且第一项为数组（castRest转换后的参数会外加一个[]），调用arrayMap对transforms
  //中的每一个参数处理函数执行baseUnary（创建只接受一个参数的函数）
  transforms = (transforms.length == 1 && isArray(transforms[0]))
    ? arrayMap(transforms[0], baseUnary(baseIteratee))
    : arrayMap(baseFlatten(transforms, 1), baseUnary(baseIteratee));
	//获取transform长度
  var funcsLength = transforms.length;
  return baseRest(function(args) {
    var index = -1,
        //取处理函数数量与func参数数量的较小值，一个参数对应一个参数处理函数
        length = nativeMin(args.length, funcsLength);
	//对参数进行处理
    while (++index < length) {
      args[index] = transforms[index].call(this, args[index]);
    }
    return apply(func, this, args);
  });
});

module.exports = overArgs;

```



### 源码

源码中涉及到的函数

+   [apply](#apply)
+   [arrayMap](#arrayMap)
+   [baseFlatten](#baseFlatten)
+   [baseIteratee](#baseIteratee)
+   [baseRest](#baseRest)
+   [baseUnary](#baseUnary)
+   [castRest](#castRest)
+   [isArray](#isArray检查数组)

```js
var apply = require('./_apply'),
    arrayMap = require('./_arrayMap'),
    baseFlatten = require('./_baseFlatten'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    baseUnary = require('./_baseUnary'),
    castRest = require('./_castRest'),
    isArray = require('./isArray');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMin = Math.min;

/**
 * Creates a function that invokes `func` with its arguments transformed.
 *
 * @static
 * @since 4.0.0
 * @memberOf _
 * @category Function
 * @param {Function} func The function to wrap.
 * @param {...(Function|Function[])} [transforms=[_.identity]]
 *  The argument transforms.
 * @returns {Function} Returns the new function.
 * @example
 *
 * function doubled(n) {
 *   return n * 2;
 * }
 *
 * function square(n) {
 *   return n * n;
 * }
 *
 * var func = _.overArgs(function(x, y) {
 *   return [x, y];
 * }, [square, doubled]);
 *
 * func(9, 3);
 * // => [81, 6]
 *
 * func(10, 5);
 * // => [100, 10]
 */
var overArgs = castRest(function(func, transforms) {
  transforms = (transforms.length == 1 && isArray(transforms[0]))
    ? arrayMap(transforms[0], baseUnary(baseIteratee))
    : arrayMap(baseFlatten(transforms, 1), baseUnary(baseIteratee));

  var funcsLength = transforms.length;
  return baseRest(function(args) {
    var index = -1,
        length = nativeMin(args.length, funcsLength);

    while (++index < length) {
      args[index] = transforms[index].call(this, args[index]);
    }
    return apply(func, this, args);
  });
});

module.exports = overArgs;

```

##  partial预设参数

**partial(func, [partials])**

>   Creates a function that invokes `func` with `partials` prepended to the arguments it receives. This method is like `_.bind` except it does **not** alter the `this` binding.
>
>   The `_.partial.placeholder` value, which defaults to `_` in monolithic builds, may be used as a placeholder for partially applied arguments. 
>
>   **Note:** This method doesn't set the "length" property of partially applied functions.
>
>   创建一个函数。 该函数调用 func，并传入预设的 partials 参数。 这个方法类似_.bind，除了它不会绑定 this。
>
>   这个 _.partial.placeholder 的值，默认是以 _ 作为附加部分参数的占位符。
>
>   **注意:** 这个方法不会设置 "length" 到函数上。

### 参数

+   func Function 需要预设参数的函数
+   partials ...any 可选 预设的参数

### 返回

**Function**

返回有预设参数的函数

### 源码

源码中涉及到的函数

+   [baseRest](#baseRest)
+   [createWrap](#createWrap)
+   [getHolder](#getHolder)
+   [replaceHolders](#replaceHolders)

```js
var baseRest = require('./_baseRest'),
    createWrap = require('./_createWrap'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders');

/** Used to compose bitmasks for function metadata. */
var WRAP_PARTIAL_FLAG = 32;

/**
 * Creates a function that invokes `func` with `partials` prepended to the
 * arguments it receives. This method is like `_.bind` except it does **not**
 * alter the `this` binding.
 *
 * The `_.partial.placeholder` value, which defaults to `_` in monolithic
 * builds, may be used as a placeholder for partially applied arguments.
 *
 * **Note:** This method doesn't set the "length" property of partially
 * applied functions.
 *
 * @static
 * @memberOf _
 * @since 0.2.0
 * @category Function
 * @param {Function} func The function to partially apply arguments to.
 * @param {...*} [partials] The arguments to be partially applied.
 * @returns {Function} Returns the new partially applied function.
 * @example
 *
 * function greet(greeting, name) {
 *   return greeting + ' ' + name;
 * }
 *
 * var sayHelloTo = _.partial(greet, 'hello');
 * sayHelloTo('fred');
 * // => 'hello fred'
 *
 * // Partially applied with placeholders.
 * var greetFred = _.partial(greet, _, 'fred');
 * greetFred('hi');
 * // => 'hi fred'
 */
var partial = baseRest(function(func, partials) {
  var holders = replaceHolders(partials, getHolder(partial));
  return createWrap(func, WRAP_PARTIAL_FLAG, undefined, partials, holders);
});

// Assign default placeholders.
partial.placeholder = {};

module.exports = partial;

```

## partialRight附加预设参数

**partialRight(func, [partials])**

>   This method is like `_.partial` except that partially applied arguments are appended to the arguments it receives.
>
>   The `_.partialRight.placeholder` value, which defaults to `_` in monolithic builds, may be used as a placeholder for partially applied arguments.
>
>   **Note:** This method doesn't set the "length" property of partially applied functions.
>
>   这个函数类似_.partial，除了预设参数被附加到接受参数的后面。
>
>   这个 _.partialRight.placeholder 的值，默认是以 _ 作为附加部分参数的占位符。
>
>   **注意**: 这个方法不会设置 "length" 到函数上。

### 参数

+   func Function 待预设参数的函数
+   partials ...any 可选 预设的参数

### 返回

**Function**

返回预设参数的函数。

### 源码

源码中涉及到的函数

+   [baseRest](#baseRest)
+   [createWrap](#createWrap)
+   [getHolder](#getHolder)
+   [replaceHolders](#replaceHolders)

```js
var baseRest = require('./_baseRest'),
    createWrap = require('./_createWrap'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders');

/** Used to compose bitmasks for function metadata. */
var WRAP_PARTIAL_RIGHT_FLAG = 64;

/**
 * This method is like `_.partial` except that partially applied arguments
 * are appended to the arguments it receives.
 *
 * The `_.partialRight.placeholder` value, which defaults to `_` in monolithic
 * builds, may be used as a placeholder for partially applied arguments.
 *
 * **Note:** This method doesn't set the "length" property of partially
 * applied functions.
 *
 * @static
 * @memberOf _
 * @since 1.0.0
 * @category Function
 * @param {Function} func The function to partially apply arguments to.
 * @param {...*} [partials] The arguments to be partially applied.
 * @returns {Function} Returns the new partially applied function.
 * @example
 *
 * function greet(greeting, name) {
 *   return greeting + ' ' + name;
 * }
 *
 * var greetFred = _.partialRight(greet, 'fred');
 * greetFred('hi');
 * // => 'hi fred'
 *
 * // Partially applied with placeholders.
 * var sayHelloTo = _.partialRight(greet, 'hello', _);
 * sayHelloTo('fred');
 * // => 'hello fred'
 */
var partialRight = baseRest(function(func, partials) {
  var holders = replaceHolders(partials, getHolder(partialRight));
  return createWrap(func, WRAP_PARTIAL_RIGHT_FLAG, undefined, partials, holders);
});

// Assign default placeholders.
partialRight.placeholder = {};

module.exports = partialRight;

```



## partition集合分组

**partition(collection, [predicate=_.identity])**

>   Creates an array of elements split into two groups, the first of which contains elements `predicate` returns truthy for, the second of which  contains elements `predicate` returns falsey for. The predicate is invoked with one argument: (value).

### 参数

+   collection Array|Object 待分组的集合
+   predicate Array|Function|Object|string 可选 断言 迭代器 每次迭代调用的函数

### 返回

**Array**

返回分组后的数组

### 源码

源码中涉及的函数

+   [createAggregator](#createAggregator)

```js
var createAggregator = require('./_createAggregator');

/**
 * Creates an array of elements split into two groups, the first of which
 * contains elements `predicate` returns truthy for, the second of which
 * contains elements `predicate` returns falsey for. The predicate is
 * invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the array of grouped elements.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'age': 36, 'active': false },
 *   { 'user': 'fred',    'age': 40, 'active': true },
 *   { 'user': 'pebbles', 'age': 1,  'active': false }
 * ];
 *
 * _.partition(users, function(o) { return o.active; });
 * // => objects for [['fred'], ['barney', 'pebbles']]
 *
 * // The `_.matches` iteratee shorthand.
 * _.partition(users, { 'age': 1, 'active': false });
 * // => objects for [['pebbles'], ['barney', 'fred']]
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.partition(users, ['active', false]);
 * // => objects for [['barney', 'pebbles'], ['fred']]
 *
 * // The `_.property` iteratee shorthand.
 * _.partition(users, 'active');
 * // => objects for [['fred'], ['barney', 'pebbles']]
 */
var partition = createAggregator(function(result, value, key) {
  //key为true，存到result[0] key的值由断言产生，true|false
  result[key ? 0 : 1].push(value);
}, function() { return [[], []]; });

module.exports = partition;

```

## pull数组移除指定值

**pull(array, [values])**

>   Removes all given values from `array` using `SameValueZero`  for equality comparisons.
>
>   移除数组array中所有和给定值相等的元素，使用SameValueZero 进行全等比较。

**注意：pull类函数是直接在原数组上进行操作的**

### 参数

+   array Array 待操作数组
+   [values] rest any 可选 待移除的值

### 返回

**Array**

处理后的数组

### 示例

**示例1**

```js
let array = ['a', 'b', 'c', 'a', 'b', 'c'];
res = pull(array, 'a', 'c')
console.log('array', array)//array [ 'b', 'b' ]
console.log('res', res)//res [ 'b', 'b' ]
```



### 源码

源码中涉及的方法

+   [baseRest](#baseRest)
+   [pullAll](#pullAll数组移除指定列表中的值)

```js
var baseRest = require('./_baseRest'),
    pullAll = require('./pullAll');

/**
 * Removes all given values from `array` using
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons.
 *
 * **Note:** Unlike `_.without`, this method mutates `array`. Use `_.remove`
 * to remove elements from an array by predicate.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {...*} [values] The values to remove.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = ['a', 'b', 'c', 'a', 'b', 'c'];
 *
 * _.pull(array, 'a', 'c');
 * console.log(array);
 * // => ['b', 'b']
 */
var pull = baseRest(pullAll);

module.exports = pull;

```

## pullAll数组移除指定列表中的值

pullAll(array, values)

>   This method is like `_.pull` except that it accepts an array of values to remove.
>
>   这个方法类似_.pull，区别是这个方法接收一个要移除值的数组。

**注意：pull类函数是直接在原数组上进行操作的**

### 参数

+   array Array 待处理数组
+   values Array 要移除值的数组

### 返回

**Array**

处理后的数组

### 示例

示例1

```js
console.log(pullAll(['a', 'b', 'c', 'a', 'b', 'c'], ['a', 'c']))//[ 'b', 'b' ]
```



### 源码

源码中涉及的方法

+   [basePullAll](#basePullAll)  pullAll实现的基础方法

```js
var basePullAll = require('./_basePullAll');

/**
 * This method is like `_.pull` except that it accepts an array of values to remove.
 *
 * **Note:** Unlike `_.difference`, this method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {Array} values The values to remove.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = ['a', 'b', 'c', 'a', 'b', 'c'];
 *
 * _.pullAll(array, ['a', 'c']);
 * console.log(array);
 * // => ['b', 'b']
 */
function pullAll(array, values) {
  return (array && array.length && values && values.length)
    ? basePullAll(array, values)
    : array;
}

module.exports = pullAll;

```

## pullAllBy数组迭代移除

**pullAllBy(array, values, [iteratee=_.identity])**

>   This method is like `_.pullAll` except that it accepts `iteratee` which is invoked for each element of `array` and `values` to generate the criterion by which they're compared. The iteratee is invoked with one argument: (value).
>
>   这个方法类似于_.pullAll ，区别是这个方法接受一个 iteratee（迭代函数） 调用 array 和 values的每个值以产生一个值，通过产生的值进行了比较。iteratee 会传入一个参数： (value)。

**注意：pull类函数是直接在原数组上进行操作的**

### 参数

+   array Array 待处理的数组
+   values Array 要移除的值
+   iteratee Array|Function|Object|string 可选 迭代器

### 返回

**Array**

返回修改后的数组

### 源码

源码中涉及的方法

+   [baseIteratee](#baseIteratee)
+   [basePullAll](#basePullAll)

```js
var baseIteratee = require('./_baseIteratee'),
    basePullAll = require('./_basePullAll');

/**
 * This method is like `_.pullAll` except that it accepts `iteratee` which is
 * invoked for each element of `array` and `values` to generate the criterion
 * by which they're compared. The iteratee is invoked with one argument: (value).
 *
 * **Note:** Unlike `_.differenceBy`, this method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {Array} values The values to remove.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = [{ 'x': 1 }, { 'x': 2 }, { 'x': 3 }, { 'x': 1 }];
 *
 * _.pullAllBy(array, [{ 'x': 1 }, { 'x': 3 }], 'x');
 * console.log(array);
 * // => [{ 'x': 2 }]
 */
function pullAllBy(array, values, iteratee) {
  return (array && array.length && values && values.length)
    ? basePullAll(array, values, baseIteratee(iteratee, 2))
    : array;
}

module.exports = pullAllBy;

```

## pullAllWith数组条件移除

**pullAllWith(array, values, [comparator])**

>   This method is like `_.pullAll` except that it accepts `comparator` which is invoked to compare elements of `array` to `values`. The comparator is invoked with two arguments: (arrVal, othVal).
>
>   这个方法类似于_.pullAll，区别是这个方法接受 comparator 调用array中的元素和values比较。comparator 会传入两个参数：(arrVal, othVal)。

**注意：pull类函数是直接在原数组上进行操作的**

### 参数

+   array Array 待处理数组
+   values Array 要移除的值
+   comparator Function 可选 比较器

### 返回

**Array**

返回修改后的数组

### 源码

源码中涉及的方法

+   [basePullAll](#basePullAll)

```js
var basePullAll = require('./_basePullAll');

/**
 * This method is like `_.pullAll` except that it accepts `comparator` which
 * is invoked to compare elements of `array` to `values`. The comparator is
 * invoked with two arguments: (arrVal, othVal).
 *
 * **Note:** Unlike `_.differenceWith`, this method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 4.6.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {Array} values The values to remove.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = [{ 'x': 1, 'y': 2 }, { 'x': 3, 'y': 4 }, { 'x': 5, 'y': 6 }];
 *
 * _.pullAllWith(array, [{ 'x': 3, 'y': 4 }], _.isEqual);
 * console.log(array);
 * // => [{ 'x': 1, 'y': 2 }, { 'x': 5, 'y': 6 }]
 */
function pullAllWith(array, values, comparator) {
  return (array && array.length && values && values.length)
    ? basePullAll(array, values, undefined, comparator)
    : array;
}

module.exports = pullAllWith;

```

## pullAt数组移除对应索引元素

**pullAt(array, [indexes])**

>   Removes elements from `array` corresponding to `indexes` and returns an  array of removed elements.
>
>   根据索引 `indexes`，移除`array`中对应的元素，并返回被移除元素的数组。

**注意：pull类函数是直接在原数组上进行操作的**

### 参数

+   array Array 待操作的数组
+   indexs ...(number|number[]) 可选 待移除的值的索引

### 返回

**Array**

返回移除元素组成的新数组。

### 示例

**示例1**

```js
let array = [5, 10, 15, 20];
let evens = pullAt(array, 1, 3);
console.log(array)//[ 5, 15 ]
console.log(evens)//[ 10, 20 ]
```

### 解析

```js
var arrayMap = require('./_arrayMap'),
    baseAt = require('./_baseAt'),
    basePullAt = require('./_basePullAt'),
    compareAscending = require('./_compareAscending'),
    flatRest = require('./_flatRest'),
    isIndex = require('./_isIndex');

/**
 * Removes elements from `array` corresponding to `indexes` and returns an
 * array of removed elements.
 *
 * **Note:** Unlike `_.at`, this method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {...(number|number[])} [indexes] The indexes of elements to remove.
 * @returns {Array} Returns the new array of removed elements.
 * @example
 *
 * var array = ['a', 'b', 'c', 'd'];
 * var pulled = _.pullAt(array, [1, 3]);
 *
 * console.log(array);
 * // => ['a', 'c']
 *
 * console.log(pulled);
 * // => ['b', 'd']
 */
var pullAt = flatRest(function(array, indexes) {
  var length = array == null ? 0 : array.length,
      //从数组中提取对应坐标的元素作为返回的结果
      result = baseAt(array, indexes);
  //先使用arrayMap对indexe索引数组的每一项进行处理，检查是否为索引，然后对处理后的结果进行升序排序
  //最后调用basePullAt移除数组中指定索引的值
  basePullAt(array, arrayMap(indexes, function(index) {
    return isIndex(index, length) ? +index : index;
  }).sort(compareAscending));

  return result;
});

module.exports = pullAt;

```



### 源码

源码中涉及的函数

+   [arrayMap](#arrayMap)
+   [baseAt](#baseAt)
+   [basePullAt](#basePullAt)
+   [compareAscending](#compareAscending)
+   [flatRest](#flatRest)
+   [isIndex](#isIndex)

```js
var arrayMap = require('./_arrayMap'),
    baseAt = require('./_baseAt'),
    basePullAt = require('./_basePullAt'),
    compareAscending = require('./_compareAscending'),
    flatRest = require('./_flatRest'),
    isIndex = require('./_isIndex');

/**
 * Removes elements from `array` corresponding to `indexes` and returns an
 * array of removed elements.
 *
 * **Note:** Unlike `_.at`, this method mutates `array`.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {...(number|number[])} [indexes] The indexes of elements to remove.
 * @returns {Array} Returns the new array of removed elements.
 * @example
 *
 * var array = ['a', 'b', 'c', 'd'];
 * var pulled = _.pullAt(array, [1, 3]);
 *
 * console.log(array);
 * // => ['a', 'c']
 *
 * console.log(pulled);
 * // => ['b', 'd']
 */
var pullAt = flatRest(function(array, indexes) {
  var length = array == null ? 0 : array.length,
      result = baseAt(array, indexes);

  basePullAt(array, arrayMap(indexes, function(index) {
    return isIndex(index, length) ? +index : index;
  }).sort(compareAscending));

  return result;
});

module.exports = pullAt;

```

## rearg函数指定参数调用

**rearg(func, indexes)**

>   Creates a function that invokes `func` with arguments arranged according to the specified `indexes` where the argument value at the first index is provided as the first argument, the argument value at the second index is provided as the second argument, and so on.
>
>   创建一个函数,调用`func`时，根据指定的 `indexes` 调整对应位置参数。其中第一个索引值是对应第一个参数，第二个索引值是作为第二个参数，依此类推。

### 参数

+   func Function 待包装的函数
+   indexes ...(number|number[]) 排列参数的位置

### 返回

**Function**

返回新的函数。

### 示例

**示例1**

索引2表示函数rearged将使用第三个实参a作为第一个参数，索引0表示rearged函数将使用第1个参数b作为第二个参数

```js
var rearged = rearg(function (a, b, c) {
    return [a, b, c];
}, [2, 0, 1]);
console.log(rearged('b', 'c', 'a'))
//[ 'a', 'b', 'c' ]
```



### 源码

源码中涉及到的函数

+   [createWrap](#createWrap)
+   [flatRest](#flatRest)

```JS
var createWrap = require('./_createWrap'),
    flatRest = require('./_flatRest');

/** Used to compose bitmasks for function metadata. */
var WRAP_REARG_FLAG = 256;

/**
 * Creates a function that invokes `func` with arguments arranged according
 * to the specified `indexes` where the argument value at the first index is
 * provided as the first argument, the argument value at the second index is
 * provided as the second argument, and so on.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Function
 * @param {Function} func The function to rearrange arguments for.
 * @param {...(number|number[])} indexes The arranged argument indexes.
 * @returns {Function} Returns the new function.
 * @example
 *
 * var rearged = _.rearg(function(a, b, c) {
 *   return [a, b, c];
 * }, [2, 0, 1]);
 *
 * rearged('b', 'c', 'a')
 * // => ['a', 'b', 'c']
 */
var rearg = flatRest(function(func, indexes) {
  return createWrap(func, WRAP_REARG_FLAG, undefined, undefined, undefined, indexes);
});

module.exports = rearg;

```



## reduce集合累计操作

**reduce(collection, [iteratee=_.identity], [accumulator])**

>   Reduces `collection` to a value which is the accumulated result of running each element in `collection` thru `iteratee`, where each successive invocation is supplied the return value of the previous. If `accumulator` is not given, the first element of `collection` is used as the initial  value. The iteratee is invoked with four arguments: (accumulator, value, index|key, collection).
>
>   Many lodash methods are guarded to work as iteratees for methods like `_.reduce`, `_.reduceRight`, and `_.transform`.
>
>   The guarded methods are: `assign`, `defaults`, `defaultsDeep`, `includes`, `merge`, `orderBy`, and `sortBy`
>
>   压缩 collection（集合）为一个值，通过 iteratee（迭代函数）遍历 collection（集合）中的每个元素，每次返回的值会作为下一次迭代使用(注：作为iteratee（迭代函数）的第一个参数使用)。 **如果没有提供 accumulator，则 collection（集合）中的第一个元素作为初始值。(注：accumulator参数在第一次迭代的时候作为iteratee（迭代函数）第一个参数使用。) iteratee 调用4个参数：**
>   **(accumulator, value, index|key, collection).**
>
>   lodash 中有许多方法是防止作为其他方法的迭代函数（注：即不能作为iteratee参数传递给其他方法），例如：_.reduce,_.reduceRight, 和_.transform。
>
>   受保护的方法有（注：即这些方法不能使用_.reduce,_.reduceRight, 和_.transform作为 iteratee 迭代函数参数）：
>
>   assign, defaults, defaultsDeep, includes, merge, orderBy, 和 sortBy

### 参数

+   collection Array|Object 待操作的集合
+   iteratee Function 可选 每次迭代调用的函数 调用4个参数(accumulator, value, index|key, collection)
+   accumulator any 可选 初始值

### 返回

**any**

返回累加后的值

### 源码

源码中涉及的函数

+   [arrayReduce](#arrayReduce)
+   [baseEach](#baseEach)
+   [baseIteratee](#baseIteratee)
+   [baseReduce](#baseReduce)
+   [isArray](#isArray检查数组)

```js
var arrayReduce = require('./_arrayReduce'),
    baseEach = require('./_baseEach'),
    baseIteratee = require('./_baseIteratee'),
    baseReduce = require('./_baseReduce'),
    isArray = require('./isArray');

/**
 * Reduces `collection` to a value which is the accumulated result of running
 * each element in `collection` thru `iteratee`, where each successive
 * invocation is supplied the return value of the previous. If `accumulator`
 * is not given, the first element of `collection` is used as the initial
 * value. The iteratee is invoked with four arguments:
 * (accumulator, value, index|key, collection).
 *
 * Many lodash methods are guarded to work as iteratees for methods like
 * `_.reduce`, `_.reduceRight`, and `_.transform`.
 *
 * The guarded methods are:
 * `assign`, `defaults`, `defaultsDeep`, `includes`, `merge`, `orderBy`,
 * and `sortBy`
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @param {*} [accumulator] The initial value.
 * @returns {*} Returns the accumulated value.
 * @see _.reduceRight
 * @example
 *
 * _.reduce([1, 2], function(sum, n) {
 *   return sum + n;
 * }, 0);
 * // => 3
 *
 * _.reduce({ 'a': 1, 'b': 2, 'c': 1 }, function(result, value, key) {
 *   (result[value] || (result[value] = [])).push(key);
 *   return result;
 * }, {});
 * // => { '1': ['a', 'c'], '2': ['b'] } (iteration order is not guaranteed)
 */
function reduce(collection, iteratee, accumulator) {
  var func = isArray(collection) ? arrayReduce : baseReduce,
      initAccum = arguments.length < 3;
  //如果参数数量小于3，即accumulator未传入，initAccum为true，使用集合第一个元素作为初始值
  return func(collection, baseIteratee(iteratee, 4), accumulator, initAccum, baseEach);
}

module.exports = reduce;

```

## reduceRight集合倒序累计操作

**reduceRight(collection, [iteratee=_.identity], [accumulator])**

>This method is like `_.reduce` except that it iterates over elements of `collection` from right to left.
>
>这个方法类似_.reduce ，除了它是从右到左遍历collection（集合）中的元素的。

### 参数

+   collection Array|Object 待操作的集合
+   iteratee Function 可选 迭代器 每次迭代调用的函数。
+   accumulator any 可选 初始值

### 返回

**any**

返回累加后的值

### 源码

源码中涉及的函数

+   [arrayReduceRight](#arrayReduceRight)
+   [baseEachRight](#baseEachRight)
+   [baseIteratee](#baseIteratee)
+   [baseReduce](#baseReduce)
+   [isArray](#isArray检查数组)

```js
var arrayReduceRight = require('./_arrayReduceRight'),
    baseEachRight = require('./_baseEachRight'),
    baseIteratee = require('./_baseIteratee'),
    baseReduce = require('./_baseReduce'),
    isArray = require('./isArray');

/**
 * This method is like `_.reduce` except that it iterates over elements of
 * `collection` from right to left.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [iteratee=_.identity] The function invoked per iteration.
 * @param {*} [accumulator] The initial value.
 * @returns {*} Returns the accumulated value.
 * @see _.reduce
 * @example
 *
 * var array = [[0, 1], [2, 3], [4, 5]];
 *
 * _.reduceRight(array, function(flattened, other) {
 *   return flattened.concat(other);
 * }, []);
 * // => [4, 5, 2, 3, 0, 1]
 */
function reduceRight(collection, iteratee, accumulator) {
  var func = isArray(collection) ? arrayReduceRight : baseReduce,
      initAccum = arguments.length < 3;

  return func(collection, baseIteratee(iteratee, 4), accumulator, initAccum, baseEachRight);
}

module.exports = reduceRight;

```

## reject集合filter反向过滤

**reject(collection, [predicate=_.identity])**

>   The opposite of `_.filter`; this method returns the elements of `collection`  that `predicate` does **not** return truthy for.
>
>   _.filter的反向方法;此方法 返回 predicate（断言函数） 不 返回 truthy（真值）的collection（集合）元素（注释：非真）。

### 参数

+   collection Array|Object 待过滤的集合
+   predicate Array|Function|Object|string 可选 断言 

### 返回

**Array**

返回过滤后的新数组

### 源码

源码中涉及到的函数

+   [arrayFilter](#arrayFilter)
+   [baseFilter](#baseFilter)
+   [baseIteratee](#baseIteratee)
+   [isArray](#isArray检查数组)
+   [negate](#negate创建取反函数)

```js
var arrayFilter = require('./_arrayFilter'),
    baseFilter = require('./_baseFilter'),
    baseIteratee = require('./_baseIteratee'),
    isArray = require('./isArray'),
    negate = require('./negate');

/**
 * The opposite of `_.filter`; this method returns the elements of `collection`
 * that `predicate` does **not** return truthy for.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new filtered array.
 * @see _.filter
 * @example
 *
 * var users = [
 *   { 'user': 'barney', 'age': 36, 'active': false },
 *   { 'user': 'fred',   'age': 40, 'active': true }
 * ];
 *
 * _.reject(users, function(o) { return !o.active; });
 * // => objects for ['fred']
 *
 * // The `_.matches` iteratee shorthand.
 * _.reject(users, { 'age': 40, 'active': true });
 * // => objects for ['barney']
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.reject(users, ['active', false]);
 * // => objects for ['fred']
 *
 * // The `_.property` iteratee shorthand.
 * _.reject(users, 'active');
 * // => objects for ['barney']
 */
function reject(collection, predicate) {
  var func = isArray(collection) ? arrayFilter : baseFilter;
  return func(collection, negate(baseIteratee(predicate, 3)));
}

module.exports = reject;

```

## remove数组条件移除所有值

**remove(array, [predicate=_.identity])**

>   Removes all elements from `array` that `predicate` returns truthy for and returns an array of the removed elements. The predicate is invoked with three arguments: (value, index, array).
>
>   移除数组中predicate（断言）返回为真值的所有元素，并返回移除元素组成的数组。predicate（断言） 会传入3个参数： (value, index, array)。

**注意：remove会改变原数组，因为remove的基础实现与pullAt的基础实现函数一致**

### 参数

+   array Array 待操作的数组
+   predicate Function 可选 断言函数

### 返回

**Array**

被移除的值组成的新数组

### 源码

源码中涉及的函数

+   [baseIteratee](#baseIteratee)
+   [basePullAt](#basePullAt)

```js
var baseIteratee = require('./_baseIteratee'),
    basePullAt = require('./_basePullAt');

/**
 * Removes all elements from `array` that `predicate` returns truthy for
 * and returns an array of the removed elements. The predicate is invoked
 * with three arguments: (value, index, array).
 *
 * **Note:** Unlike `_.filter`, this method mutates `array`. Use `_.pull`
 * to pull elements from an array by value.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the new array of removed elements.
 * @example
 *
 * var array = [1, 2, 3, 4];
 * var evens = _.remove(array, function(n) {
 *   return n % 2 == 0;
 * });
 *
 * console.log(array);
 * // => [1, 3]
 *
 * console.log(evens);
 * // => [2, 4]
 */
function remove(array, predicate) {
  var result = [];
  if (!(array && array.length)) {
    return result;
  }
  var index = -1,
      indexes = [],
      length = array.length;

  predicate = baseIteratee(predicate, 3);
  while (++index < length) {
    var value = array[index];
    if (predicate(value, index, array)) {
      result.push(value);
      indexes.push(index);
    }
  }
  basePullAt(array, indexes);
  return result;
}

module.exports = remove;

```

## reverse数组倒置

**reverse(array)**

>   Reverses `array` so that the first element becomes the last, the second element becomes the second to last, and so on.
>
>   反转array，使得第一个元素变为最后一个元素，第二个元素变为倒数第二个元素，依次类推。

**注意：reverse会改变原数组，它是基于Array.prototype.reverse;**

### 参数

+   array Array 待处理的数组

### 返回

**Array**

倒置后的数组

### 源码

```js
/** Used for built-in method references. */
var arrayProto = Array.prototype;

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeReverse = arrayProto.reverse;

/**
 * Reverses `array` so that the first element becomes the last, the second
 * element becomes the second to last, and so on.
 *
 * **Note:** This method mutates `array` and is based on
 * [`Array#reverse`](https://mdn.io/Array/reverse).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to modify.
 * @returns {Array} Returns `array`.
 * @example
 *
 * var array = [1, 2, 3];
 *
 * _.reverse(array);
 * // => [3, 2, 1]
 *
 * console.log(array);
 * // => [3, 2, 1]
 */
function reverse(array) {
  return array == null ? array : nativeReverse.call(array);
}

module.exports = reverse;

```

## round四舍五入

**round(number, [precision=0])**

>   Computes `number` rounded to `precision`.
>
>   根据 precision（精度） 四舍五入 number。

### 参数

+   number number 待舍入的值
+   precision number 可选 精度，保留的小数位数 默认precision=0

### 返回

**number** 

舍入后的值

### 源码

源码中涉及到的函数

+   [createRound](#createRound)

```js
var createRound = require('./_createRound');

/**
 * Computes `number` rounded to `precision`.
 *
 * @static
 * @memberOf _
 * @since 3.10.0
 * @category Math
 * @param {number} number The number to round.
 * @param {number} [precision=0] The precision to round to.
 * @returns {number} Returns the rounded number.
 * @example
 *
 * _.round(4.006);
 * // => 4
 *
 * _.round(4.006, 2);
 * // => 4.01
 *
 * _.round(4060, -2);
 * // => 4100
 */
var round = createRound('round');

module.exports = round;

```

## sample获取集合中一个随机元素

**sample(collection)**

> Gets a random element from `collection`.
>
> 从collection（集合）中获得一个随机元素。

### 参数

+   collection Array|Object 待取值的集合

### 返回

**any**

返回一个随机元素

### 源码

源码中涉及到的函数

+   [arraySample](#arraySample)
+   [baseSample](#baseSample)
+   [isArray](#isArray检查数组)

```js
var arraySample = require('./_arraySample'),
    baseSample = require('./_baseSample'),
    isArray = require('./isArray');

/**
 * Gets a random element from `collection`.
 *
 * @static
 * @memberOf _
 * @since 2.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to sample.
 * @returns {*} Returns the random element.
 * @example
 *
 * _.sample([1, 2, 3, 4]);
 * // => 2
 */
function sample(collection) {
  var func = isArray(collection) ? arraySample : baseSample;
  return func(collection);
}

module.exports = sample;

```

## sampleSize获取集合中n个随机元素

**sampleSize(collection, [n=1])**

>   Gets `n` random elements at unique keys from `collection` up to the size of `collection`.
>
>   从`collection`（集合）中获得 `n` 个随机元素。

### 参数

+   collection Array|Object 待获取的集合
+   n number 可选 获取的元素个数 默认n=1
+   guard Object 可选  是否可以作为遍历参数被_.map之类的方法调用.

### 返回

**Array**

返回随机元素组成的数组

### 源码

源码中涉及的函数

+   [arraySampleSize](#arraySampleSize)
+   [baseSampleSize](#baseSampleSize)
+   [isArray](#isArray检查数组)
+   [toInteger ](#toInteger转为整数)
+   [isIterateeCall](#isIterateeCall)

```js
var arraySampleSize = require('./_arraySampleSize'),
    baseSampleSize = require('./_baseSampleSize'),
    isArray = require('./isArray'),
    isIterateeCall = require('./_isIterateeCall'),
    toInteger = require('./toInteger');

/**
 * Gets `n` random elements at unique keys from `collection` up to the
 * size of `collection`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection The collection to sample.
 * @param {number} [n=1] The number of elements to sample.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the random elements.
 * @example
 *
 * _.sampleSize([1, 2, 3], 2);
 * // => [3, 1]
 *
 * _.sampleSize([1, 2, 3], 4);
 * // => [2, 3, 1]
 */
function sampleSize(collection, n, guard) {
  if ((guard ? isIterateeCall(collection, n, guard) : n === undefined)) {
    n = 1;
  } else {
    n = toInteger(n);
  }
  var func = isArray(collection) ? arraySampleSize : baseSampleSize;
  return func(collection, n);
}

module.exports = sampleSize;

```

## shuffle集合随机重组为数组

**shuffle(collection)**

>   Creates an array of shuffled values, using a version of the Fisher-Yates shuffle
>
>   创建一个被打乱值的集合。 使用[Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher-Yates_shuffle) 版本。

[洗牌算法]: https://zhuanlan.zhihu.com/p/110630952

简单概述：

进行size次循环，每次循环,在index到lastIndex位置上的值中随机娶一个放到index位置上，可以得出其放置值x的几率为

P(0号位不放x) · P(1号位不放x) · ... · P(index位放x)=
$$
\frac{size-1}{size}*\frac{size-2}{size-1}*\frac{size-3}{size-2}*...*\frac{size-(index-1)-1}{size-(index-1)}*\frac{1}{size-index}=\frac{1}{size}
$$
这是洗牌算法的等概率

### 参数

+   collection Array|Object 要打乱的集合

### 返回

**Array**

返回打乱的新数组。

### 示例

**示例1**

```js
console.log(shuffle({
    a: 0, b: 1, c: 2, d: 3, e: 4
}))//[ 0, 2, 4, 1, 3 ]
```

### 源码

源码中涉及到的函数

+   [arrayShuffle](#arrayShuffle)
+   [baseShuffle](#baseShuffle)
+     [isArray](#isArray检查数组)

```js
var arrayShuffle = require('./_arrayShuffle'),
    baseShuffle = require('./_baseShuffle'),
    isArray = require('./isArray');

/**
 * Creates an array of shuffled values, using a version of the
 * [Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher-Yates_shuffle).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to shuffle.
 * @returns {Array} Returns the new shuffled array.
 * @example
 *
 * _.shuffle([1, 2, 3, 4]);
 * // => [4, 1, 3, 2]
 */
function shuffle(collection) {
    var func = isArray(collection) ? arrayShuffle : baseShuffle;
    return func(collection);
}

module.exports = shuffle;

```

## size获取集合长度

**size(collection)**

>   Gets the size of `collection` by returning its length for array-like values or the number of own enumerable string keyed properties for objects.
>
>   返回`collection`（集合）的长度，如果集合是类数组或字符串，返回其 length ；如果集合是对象，返回其可枚举属性的个数。

### 参数

+   collection Array|Object 待操作的集合

### 返回

**number**

返回集合的长度

### 源码

源码中涉及到的函数

+   [baseKeys](#baseKeys)
+   [stringSize](#stringSize)
+   [getTag](#getTag)
+   [isArrayLike](#isArrayLike检查类数组)
+   [isString](#isString检查字符串)

```js
var baseKeys = require('./_baseKeys'),
    getTag = require('./_getTag'),//获取数据的类型
    isArrayLike = require('./isArrayLike'),
    isString = require('./isString'),
    stringSize = require('./_stringSize');

/** `Object#toString` result references. */
var mapTag = '[object Map]',
    setTag = '[object Set]';

/**
 * Gets the size of `collection` by returning its length for array-like
 * values or the number of own enumerable string keyed properties for objects.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object|string} collection The collection to inspect.
 * @returns {number} Returns the collection size.
 * @example
 *
 * _.size([1, 2, 3]);
 * // => 3
 *
 * _.size({ 'a': 1, 'b': 2 });
 * // => 2
 *
 * _.size('pebbles');
 * // => 7
 */
function size(collection) {
  if (collection == null) {
    return 0;
  }
  if (isArrayLike(collection)) {
    return isString(collection) ? stringSize(collection) : collection.length;
  }
  var tag = getTag(collection);
  if (tag == mapTag || tag == setTag) {
    return collection.size;
  }
  return baseKeys(collection).length;
}

module.exports = size;

```



## slice数组截取

**slice(array, [start=0], [end=array.length])**

>   Creates a slice of `array` from `start` up to, but not including, `end`.
>
>   裁剪数组array，从 start 位置开始到end结束，但不包括 end 本身的位置。

**Note: 这个方法用于代替Array.slice 来确保数组正确返回。**

### 参数

+   array Array 待裁剪的数组
+   start number 可选 裁剪的开始位置 默认start=0
+   end number 可选 裁剪的结束位置 默认end=array.length

### 返回

**Array**

返回 数组array 裁剪部分的新数组。

### 示例

**示例1**

基础用法

```js
let a = [0, 1, 2, 3, 4, 5, 6]
let b = slice(a, 2, 4)
console.log(a)//[0, 1, 2, 3, 4, 5, 6]
console.log(b)//[ 2, 3 ]
```



### 源码

源码中涉及的函数

+   [baseSlice](#baseSlice)
+   [isIterateeCall](#isIterateeCall)
+   [toInteger](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    isIterateeCall = require('./_isIterateeCall'),
    toInteger = require('./toInteger');

/**
 * Creates a slice of `array` from `start` up to, but not including, `end`.
 *
 * **Note:** This method is used instead of
 * [`Array#slice`](https://mdn.io/Array/slice) to ensure dense arrays are
 * returned.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to slice.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns the slice of `array`.
 */
function slice(array, start, end) {
    var length = array == null ? 0 : array.length;
    if (!length) {
        return [];
    }
    if (end && typeof end != 'number' && isIterateeCall(array, start, end)) {
        start = 0;
        end = length;
    }
    else {
        start = start == null ? 0 : toInteger(start);
        end = end === undefined ? length : toInteger(end);
    }
    return baseSlice(array, start, end);
}

module.exports = slice;

```

## some集合元素存在鉴定

**some(collection, [predicate=_.identity])**

>   Checks if `predicate` returns truthy for **any** element of `collection`.  Iteration is stopped once `predicate` returns truthy. The predicate is invoked with three arguments: (value, index|key, collection).
>
>   通过 `predicate`（断言函数） 检查`collection`（集合）中的元素是否存在 任意 truthy（真值）的元素，一旦 `predicate`（断言函数） 返回 truthy（真值），遍历就停止。 predicate 调用3个参数：*(value, index|key, collection)*。

### 参数

+   collection Array|Object 待操作的集合
+   predicate Array|Function|Object|string  可选 断言 
+   guard Object 可选 迭代守卫

### 返回

**boolean**

如果存在元素经 predicate 检查为 truthy（真值），返回 `true` ，否则返回 `false` 。

### 示例

**示例1**

```js
console.log(some([1, 2, 3, 4, 5, 6], (n) => n * n == 9))//true
```

### 源码

源码中涉及到的函数

+   [arraySome](#arraySome)
+   [baseSome](#baseSome)
+   [baseIteratee](#baseIteratee)
+   [isIterateeCall](#isIterateeCall)
+     [isArray](#isArray检查数组)

```js
var arraySome = require('./_arraySome'),
    baseIteratee = require('./_baseIteratee'),
    baseSome = require('./_baseSome'),
    isArray = require('./isArray'),
    isIterateeCall = require('./_isIterateeCall');

/**
 * Checks if `predicate` returns truthy for **any** element of `collection`.
 * Iteration is stopped once `predicate` returns truthy. The predicate is
 * invoked with three arguments: (value, index|key, collection).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {boolean} Returns `true` if any element passes the predicate check,
 *  else `false`.
 * @example
 *
 * _.some([null, 0, 'yes', false], Boolean);
 * // => true
 *
 * var users = [
 *   { 'user': 'barney', 'active': true },
 *   { 'user': 'fred',   'active': false }
 * ];
 *
 * // The `_.matches` iteratee shorthand.
 * _.some(users, { 'user': 'barney', 'active': false });
 * // => false
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.some(users, ['active', false]);
 * // => true
 *
 * // The `_.property` iteratee shorthand.
 * _.some(users, 'active');
 * // => true
 */
function some(collection, predicate, guard) {
  var func = isArray(collection) ? arraySome : baseSome;
  if (guard && isIterateeCall(collection, predicate, guard)) {
    predicate = undefined;
  }
  return func(collection, baseIteratee(predicate, 3));
}

module.exports = some;

```



## sortBy集合迭代升序

**sortBy(collection, [iteratees=[_.identity]])**

>   Creates an array of elements, sorted in ascending order by the results of running each element in a collection thru each iteratee. This method performs a stable sort, that is, it preserves the original sort order of equal elements. The iteratees are invoked with one argument: (value).
>
>   创建一个元素数组。 以 iteratee 处理的结果升序排序。 这个方法执行稳定排序，也就是说相同元素会保持原始排序。 iteratees 调用1个参数： *(value)*。

### 参数

+   collection Array|Object 待操作的集合
+   iteratees ...(Array|Array[]|Function|Function[]|Object|Object[]|string|string[]) 可选 rest 决定排序

### 返回

**Array**

返回排序后的数组

### 示例

**示例1**

先根据user值升序，再根据匿名函数的处理结果升序

```js
var users = [
    { 'user': 'fred', 'age': 48 },
    { 'user': 'barney', 'age': 36 },
    { 'user': 'fred', 'age': 30 },
    { 'user': 'barney', 'age': 34 }
];
console.log(sortBy(users, 'user', function (o) {
    return Math.round(o.age);
}))
// [
//     { user: 'barney', age: 34 },
//     { user: 'barney', age: 36 },
//     { user: 'fred', age: 30 },
//     { user: 'fred', age: 48 }
// ]
```

### 解析

```js
var baseFlatten = require('./_baseFlatten'),
    baseOrderBy = require('./_baseOrderBy'),
    baseRest = require('./_baseRest'),
    isIterateeCall = require('./_isIterateeCall');

/**
 * Creates an array of elements, sorted in ascending order by the results of
 * running each element in a collection thru each iteratee. This method
 * performs a stable sort, that is, it preserves the original sort order of
 * equal elements. The iteratees are invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {...(Function|Function[])} [iteratees=[_.identity]]
 *  The iteratees to sort by.
 * @returns {Array} Returns the new sorted array.
 * @example
 *
 * var users = [
 *   { 'user': 'fred',   'age': 48 },
 *   { 'user': 'barney', 'age': 36 },
 *   { 'user': 'fred',   'age': 30 },
 *   { 'user': 'barney', 'age': 34 }
 * ];
 *
 * _.sortBy(users, [function(o) { return o.user; }]);
 * // => objects for [['barney', 36], ['barney', 34], ['fred', 48], ['fred', 30]]
 *
 * _.sortBy(users, ['user', 'age']);
 * // => objects for [['barney', 34], ['barney', 36], ['fred', 30], ['fred', 48]]
 */
var sortBy = baseRest(function (collection, iteratees) {
    if (collection == null) {
        return [];
    }
    var length = iteratees.length;
    console.log('iteratees1', iteratees)
    //iteratees1 [ 'user', [Function (anonymous)] ]
    //如果是遍历器的参数，则遍历器为空
    if (length > 1 && isIterateeCall(collection, iteratees[0], iteratees[1])) {
        iteratees = [];
    } 
    //如果前三个用于排序的方法为遍历器的参数，指定遍历器为第一个
    else if (length > 2 && isIterateeCall(iteratees[0], iteratees[1], iteratees[2])) {
        iteratees = [iteratees[0]];
        console.log('iteratees2', iteratees)
    }
    console.log('iteratees3', iteratees)
    //iteratees3 [ 'user', [Function (anonymous)] ]
    return baseOrderBy(collection, baseFlatten(iteratees, 1), []);
});
var users = [
    { 'user': 'fred', 'age': 48 },
    { 'user': 'barney', 'age': 36 },
    { 'user': 'fred', 'age': 30 },
    { 'user': 'barney', 'age': 34 }
];
console.log(sortBy(users, 'user', function (o) {
    return Math.round(o.age);
}))
// [
//     { user: 'barney', age: 34 },
//     { user: 'barney', age: 36 },
//     { user: 'fred', age: 30 },
//     { user: 'fred', age: 48 }
// ]
module.exports = sortBy;

```

### 源码

源码中涉及的函数

+   [baseFlatten](#baseFlatten)
+   [baseOrderBy](#baseOrderBy)
+   [baseRest](#baseRest)
+   [isIterateeCall](#isIterateeCall)

```js
var baseFlatten = require('./_baseFlatten'),
    baseOrderBy = require('./_baseOrderBy'),
    baseRest = require('./_baseRest'),
    isIterateeCall = require('./_isIterateeCall');

/**
 * Creates an array of elements, sorted in ascending order by the results of
 * running each element in a collection thru each iteratee. This method
 * performs a stable sort, that is, it preserves the original sort order of
 * equal elements. The iteratees are invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Collection
 * @param {Array|Object} collection The collection to iterate over.
 * @param {...(Function|Function[])} [iteratees=[_.identity]]
 *  The iteratees to sort by.
 * @returns {Array} Returns the new sorted array.
 * @example
 *
 * var users = [
 *   { 'user': 'fred',   'age': 48 },
 *   { 'user': 'barney', 'age': 36 },
 *   { 'user': 'fred',   'age': 30 },
 *   { 'user': 'barney', 'age': 34 }
 * ];
 *
 * _.sortBy(users, [function(o) { return o.user; }]);
 * // => objects for [['barney', 36], ['barney', 34], ['fred', 48], ['fred', 30]]
 *
 * _.sortBy(users, ['user', 'age']);
 * // => objects for [['barney', 34], ['barney', 36], ['fred', 30], ['fred', 48]]
 */
var sortBy = baseRest(function(collection, iteratees) {
  if (collection == null) {
    return [];
  }
  var length = iteratees.length;
  if (length > 1 && isIterateeCall(collection, iteratees[0], iteratees[1])) {
    iteratees = [];
  } else if (length > 2 && isIterateeCall(iteratees[0], iteratees[1], iteratees[2])) {
    iteratees = [iteratees[0]];
  }
  return baseOrderBy(collection, baseFlatten(iteratees, 1), []);
});

module.exports = sortBy;

```

## sortedIndex有序数组最小插入索引

**sortedIndex(array, value)**

>   Uses a binary search to determine the lowest index at which `value` should be inserted into `array` in order to maintain its sort order.
>
>   使用二进制的方式检索来决定 `value`值 应该插入到数组中 尽可能小的索引位置，以保证`array`的排序。

### 参数

+   array Array 待检查的有序数组
+   value any 待评估的值

### 返回

**number**

应该在数组中插入的最小索引位置坐标

### 示例

示例1

```js
console.log(sortedIndex([30, 50], 40))//1
```



示例2

```js
console.log(sortedIndex(['aabee', 'aadee'], 'aacee'))//1
```



### 源码

源码中涉及的函数

+   [baseSortedIndex](#baseSortedIndex)

```js
var baseSortedIndex = require('./_baseSortedIndex');

/**
 * Uses a binary search to determine the lowest index at which `value`
 * should be inserted into `array` in order to maintain its sort order.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 * @example
 *
 * _.sortedIndex([30, 50], 40);
 * // => 1
 */
function sortedIndex(array, value) {
  return baseSortedIndex(array, value);
}

module.exports = sortedIndex;

```

## sortedIndexBy有序数组迭代最小插入索引

**sortedIndexBy(array, value, [iteratee=_.identity])**

>   This method is like `_.sortedIndex` except that it accepts `iteratee` which is invoked for `value` and each element of `array` to compute their sort ranking. The iteratee is invoked with one argument: (value).
>
>   这个方法类似_.sortedIndex ，除了它接受一个 iteratee （迭代函数），调用每一个数组（array）元素，返回结果和value 值比较来计算排序。iteratee 会传入一个参数：(value)。

### 参数

+   array Array 待检查的有序数组
+   value any 待评估的值
+   iteratee Function 可选 迭代器 支持简写

### 返回

**number**

应该在数组中插入的索引位置

### 源码

源码中涉及的函数

+   [baseIteratee](#baseIteratee)
+   [baseSortedIndexBy](#baseSortedIndexBy)

```js
var baseIteratee = require('./_baseIteratee'),
    baseSortedIndexBy = require('./_baseSortedIndexBy');

/**
 * This method is like `_.sortedIndex` except that it accepts `iteratee`
 * which is invoked for `value` and each element of `array` to compute their
 * sort ranking. The iteratee is invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 * @example
 *
 * var objects = [{ 'x': 4 }, { 'x': 5 }];
 *
 * _.sortedIndexBy(objects, { 'x': 4 }, function(o) { return o.x; });
 * // => 0
 *
 * // The `_.property` iteratee shorthand.
 * _.sortedIndexBy(objects, { 'x': 4 }, 'x');
 * // => 0
 */
function sortedIndexBy(array, value, iteratee) {
  return baseSortedIndexBy(array, value, baseIteratee(iteratee, 2));
}

module.exports = sortedIndexBy;

```

## sortedIndexOf有序数组二分查找

**sortedIndexOf(array, value)**

>   This method is like `_.indexOf` except that it performs a binary search on a sorted `array`.
>
>   这个方法类似_.indexOf，除了它是在已经排序的数组array上执行二进制检索。

### 参数

+   array Array 待搜索的数组
+   value any 待搜索的值

### 返回

**number**

返回匹配的索引，如果不存在，则返回-1

### 源码

源码中涉及的函数

+   [baseSortedIndex](#baseSortedIndex)
+   [eq](#eq值比较)

```js
var baseSortedIndex = require('./_baseSortedIndex'),
    eq = require('./eq');

/**
 * This method is like `_.indexOf` except that it performs a binary
 * search on a sorted `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @returns {number} Returns the index of the matched value, else `-1`.
 * @example
 *
 * _.sortedIndexOf([4, 5, 5, 5, 6], 5);
 * // => 1
 */
function sortedIndexOf(array, value) {
  var length = array == null ? 0 : array.length;
  if (length) {
    var index = baseSortedIndex(array, value);
    if (index < length && eq(array[index], value)) {
      return index;
    }
  }
  return -1;
}

module.exports = sortedIndexOf;

```

## sortedLastIndex有序数组最大插入索引

**sortedLastIndex(array, value)**

>   This method is like `_.sortedIndex` except that it returns the highest index at which `value` should be inserted into `array` in order to maintain its sort order.
>
>   此方法类似于_.sortedIndex，除了 它返回 value值 在 array 中尽可能大的索引位置（index）。

### 参数

+   array Array 待检查的有序数组
+   value any 待评估的值

### 返回

**number**

应该在数组中插入的最大索引位置坐标

### 源码

源码中涉及的函数

+   [baseSortedIndex](#baseSortedIndex)

```js
var baseSortedIndex = require('./_baseSortedIndex');

/**
 * This method is like `_.sortedIndex` except that it returns the highest
 * index at which `value` should be inserted into `array` in order to
 * maintain its sort order.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 * @example
 *
 * _.sortedLastIndex([4, 5, 5, 5, 6], 5);
 * // => 4
 */
function sortedLastIndex(array, value) {
  return baseSortedIndex(array, value, true);
}

module.exports = sortedLastIndex;

```

## sortedLastIndexBy有序数组迭代最大插入索引

**sortedLastIndexBy(array, value, [iteratee=_.identity])**

>   This method is like `_.sortedLastIndex` except that it accepts `iteratee` which is invoked for `value` and each element of `array` to compute their sort ranking. The iteratee is invoked with one argument: (value).

### 参数

+   array Array 待检查的有序数组
+   value any 待评估的值
+   iteratee Function 可选 迭代器 调用每个元素 支持简写

### 返回

**number**

返回 value值 应该在数组array中插入的最大索引位置 index。

### 源码

源码中涉及到的函数

+   [baseIteratee](#baseIteratee)
+   [baseSortedIndexBy](#baseSortedIndexBy)

```js
var baseIteratee = require('./_baseIteratee'),
    baseSortedIndexBy = require('./_baseSortedIndexBy');

/**
 * This method is like `_.sortedLastIndex` except that it accepts `iteratee`
 * which is invoked for `value` and each element of `array` to compute their
 * sort ranking. The iteratee is invoked with one argument: (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 * @example
 *
 * var objects = [{ 'x': 4 }, { 'x': 5 }];
 *
 * _.sortedLastIndexBy(objects, { 'x': 4 }, function(o) { return o.x; });
 * // => 1
 *
 * // The `_.property` iteratee shorthand.
 * _.sortedLastIndexBy(objects, { 'x': 4 }, 'x');
 * // => 1
 */
function sortedLastIndexBy(array, value, iteratee) {
  return baseSortedIndexBy(array, value, baseIteratee(iteratee, 2), true);
}

module.exports = sortedLastIndexBy;

```

## sortedLastIndexOf有序数组倒序二分查找

**sortedLastIndexOf(array, value)**

>   This method is like `_.lastIndexOf` except that it performs a binary search on a sorted `array`.
>
>   这个方法类似_.lastIndexOf，除了它是在已经排序的数组array上执行二进制检索。

### 参数

+   array Array 待查找的有序数组
+   value any 待查找的值

### 返回

**number**

如果value在数组中存在，返回从右开始的第一个与value相等的元素的索引（即当数组中有多个value时，返回其中最大的索引），如果不存在，则返回-1

### 源码

源码中涉及的函数

+   [baseSortedIndex](#baseSortedIndex)
+   [eq](#eq值比较)

```js
var baseSortedIndex = require('./_baseSortedIndex'),
    eq = require('./eq');

/**
 * This method is like `_.lastIndexOf` except that it performs a binary
 * search on a sorted `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @returns {number} Returns the index of the matched value, else `-1`.
 * @example
 *
 * _.sortedLastIndexOf([4, 5, 5, 5, 6], 5);
 * // => 3
 */
function sortedLastIndexOf(array, value) {
  var length = array == null ? 0 : array.length;
  if (length) {
    var index = baseSortedIndex(array, value, true) - 1;
    if (eq(array[index], value)) {
      return index;
    }
  }
  return -1;
}

module.exports = sortedLastIndexOf;

```

## sortedUniq有序数组去重

**sortedUniq(array)**

>   This method is like `_.uniq` except that it's designed and optimized for sorted arrays.
>
>   这个方法类似_.uniq，除了它会优化排序数组。

### 参数

+   array Array 待去重的有序数组

### 返回

**Array**

去重后的新的有序数组

### 示例

示例1

```js
let a = [1, 1, 2, 2, 3]
let b = sortedUniq(a)
console.log(a)//[ 1, 1, 2, 2, 3 ]
console.log(b)//[ 1, 2, 3 ]
```

### 源码

源码中涉及的函数

+   [baseSortedUniq](#baseSortedUniq)

```js
var baseSortedUniq = require('./_baseSortedUniq');

/**
 * This method is like `_.uniq` except that it's designed and optimized
 * for sorted arrays.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @returns {Array} Returns the new duplicate free array.
 * @example
 *
 * _.sortedUniq([1, 1, 2]);
 * // => [1, 2]
 */
function sortedUniq(array) {
  return (array && array.length)
    ? baseSortedUniq(array)
    : [];
}

module.exports = sortedUniq;

```

## sortedUniqBy有序数组迭代去重

**sortedUniqBy(array, [iteratee])**

>   This method is like `_.uniqBy` except that it's designed and optimized for sorted arrays.
>
>   这个方法类似_.uniqBy，除了它会优化排序数组

### 参数

+   array Array 待去重的有序数组
+   iteratee Function 迭代器 调用每个元素

### 返回

**Array**

返回去重后的新数组

### 源码

源码中涉及的函数

+   [baseIteratee](#baseIteratee)
+   [baseSortedUniq](#baseSortedUniq)

```js
var baseIteratee = require('./_baseIteratee'),
    baseSortedUniq = require('./_baseSortedUniq');

/**
 * This method is like `_.uniqBy` except that it's designed and optimized
 * for sorted arrays.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 * @example
 *
 * _.sortedUniqBy([1.1, 1.2, 2.3, 2.4], Math.floor);
 * // => [1.1, 2.3]
 */
function sortedUniqBy(array, iteratee) {
  return (array && array.length)
    ? baseSortedUniq(array, baseIteratee(iteratee, 2))
    : [];
}

module.exports = sortedUniqBy;

```

## subtract两数相减

**subtract(minuend, subtrahend)**

>   Subtract two numbers.
>
>   两数相减

### 参数

+   minuend number 被减数
+   subtrahend number  减数

### 返回

**number**

计算后的值

### 源码

源码中涉及的函数

+   [createMathOperation](#createMathOperation)

```js
var createMathOperation = require('./_createMathOperation');

/**
 * Subtract two numbers.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Math
 * @param {number} minuend The first number in a subtraction.
 * @param {number} subtrahend The second number in a subtraction.
 * @returns {number} Returns the difference.
 * @example
 *
 * _.subtract(6, 4);
 * // => 2
 */
var subtract = createMathOperation(function(minuend, subtrahend) {
  return minuend - subtrahend;
}, 0);

module.exports = subtract;

```



## tail获取数组后length-2项

**tail(array)**

>   Gets all but the first element of `array`.
>
>   获取除了array数组第一个元素以外的全部元素。

### 参数

+   array Array 待操作的数组

### 返回

**Array**

返回 array 数组的切片（除了array数组第一个元素以外的全部元素）。

### 源码

源码中涉及的函数

+   [baseSlice](#baseSlice)

```js
var baseSlice = require('./_baseSlice');

/**
 * Gets all but the first element of `array`.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.tail([1, 2, 3]);
 * // => [2, 3]
 */
function tail(array) {
  var length = array == null ? 0 : array.length;
  return length ? baseSlice(array, 1, length) : [];
}

module.exports = tail;

```

## take数组提取

**take(array, [n=1])**

### 参数

+   array Array 待提取的数组
+   n number 可选 提取长度 默认n=1

### 返回

**Array**

返回 array 数组的切片（从起始元素开始n个元素）。

### 源码

源码中涉及的函数

+   [baseSlice](#baseSlice)
+   [toInteger](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    toInteger = require('./toInteger');

/**
 * Creates a slice of `array` with `n` elements taken from the beginning.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {number} [n=1] The number of elements to take.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.take([1, 2, 3]);
 * // => [1]
 *
 * _.take([1, 2, 3], 2);
 * // => [1, 2]
 *
 * _.take([1, 2, 3], 5);
 * // => [1, 2, 3]
 *
 * _.take([1, 2, 3], 0);
 * // => []
 */
function take(array, n, guard) {
  if (!(array && array.length)) {
    return [];
  }
  n = (guard || n === undefined) ? 1 : toInteger(n);
  return baseSlice(array, 0, n < 0 ? 0 : n);
}

module.exports = take;

```

## takeRight数组倒序提取

**takeRight(array, [n=1])**

>   Creates a slice of `array` with `n` elements taken from the end.
>
>   创建一个数组切片，从`array`数组的最后一个元素开始提取`n`个元素。

### 参数

+   array Array 待提取的数组
+   n number 可选 提取的长度 默认n=1

### 返回

**Array**

 返回 array 数组的切片（从结尾元素开始n个元素）。

### 源码

+   [baseSlice](#baseSlice)
+   [toInteger ](#toInteger转为整数)

```js
var baseSlice = require('./_baseSlice'),
    toInteger = require('./toInteger');

/**
 * Creates a slice of `array` with `n` elements taken from the end.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {number} [n=1] The number of elements to take.
 * @param- {Object} [guard] Enables use as an iteratee for methods like `_.map`.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * _.takeRight([1, 2, 3]);
 * // => [3]
 *
 * _.takeRight([1, 2, 3], 2);
 * // => [2, 3]
 *
 * _.takeRight([1, 2, 3], 5);
 * // => [1, 2, 3]
 *
 * _.takeRight([1, 2, 3], 0);
 * // => []
 */
function takeRight(array, n, guard) {
  var length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  n = (guard || n === undefined) ? 1 : toInteger(n);
  n = length - n;
  return baseSlice(array, n < 0 ? 0 : n, length);
}

module.exports = takeRight;

```

## takeRightWhile数组倒序断言提取

**takeRightWhile(array, [predicate=_.identity])**

>   Creates a slice of `array` with elements taken from the end. Elements are taken until `predicate` returns falsey. The predicate is invoked with three arguments: (value, index, array).
>
>   从`array`数组的最后一个元素开始提取元素，直到 `predicate` 返回假值。`predicate` 会传入三个参数： *(value, index, array)*。

### 参数

+   array Array 待提取的数组
+   predicate Array|Function|Object|string 可选 断言函数 每次迭代调用的函数

### 返回

**Array**

返回 array 数组的切片。

### 源码

源码中涉及的函数

+   [baseWhile](#baseWhile)
+   [baseIteratee](#baseIteratee)

```js
var baseIteratee = require('./_baseIteratee'),
    baseWhile = require('./_baseWhile');

/**
 * Creates a slice of `array` with elements taken from the end. Elements are
 * taken until `predicate` returns falsey. The predicate is invoked with
 * three arguments: (value, index, array).
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'active': true },
 *   { 'user': 'fred',    'active': false },
 *   { 'user': 'pebbles', 'active': false }
 * ];
 *
 * _.takeRightWhile(users, function(o) { return !o.active; });
 * // => objects for ['fred', 'pebbles']
 *
 * // The `_.matches` iteratee shorthand.
 * _.takeRightWhile(users, { 'user': 'pebbles', 'active': false });
 * // => objects for ['pebbles']
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.takeRightWhile(users, ['active', false]);
 * // => objects for ['fred', 'pebbles']
 *
 * // The `_.property` iteratee shorthand.
 * _.takeRightWhile(users, 'active');
 * // => []
 */
function takeRightWhile(array, predicate) {
  return (array && array.length)
    ? baseWhile(array, baseIteratee(predicate, 3), false, true)
    : [];
}

module.exports = takeRightWhile;

```

## takeWhile数组断言提取

takeWhile(array, [predicate=_.identity])

>   Creates a slice of `array` with elements taken from the beginning. Elements are taken until `predicate` returns falsey. The predicate is invoked with three arguments: (value, index, array).
>
>   从array数组的起始元素开始提取元素，，直到 predicate 返回假值。predicate 会传入三个参数： (value, index, array)。

### 参数

+   array Array 待提取的数组
+   predicate Array|Function|Object|string 可选 断言函数 每次迭代调用的函数

### 返回

**Array**

返回 array 数组的切片。

### 源码

源码中涉及的函数

+   [baseIteratee](#baseIteratee)
+   [baseWhile](#baseWhile)

```js
var baseIteratee = require('./_baseIteratee'),
    baseWhile = require('./_baseWhile');

/**
 * Creates a slice of `array` with elements taken from the beginning. Elements
 * are taken until `predicate` returns falsey. The predicate is invoked with
 * three arguments: (value, index, array).
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to query.
 * @param {Function} [predicate=_.identity] The function invoked per iteration.
 * @returns {Array} Returns the slice of `array`.
 * @example
 *
 * var users = [
 *   { 'user': 'barney',  'active': false },
 *   { 'user': 'fred',    'active': false },
 *   { 'user': 'pebbles', 'active': true }
 * ];
 *
 * _.takeWhile(users, function(o) { return !o.active; });
 * // => objects for ['barney', 'fred']
 *
 * // The `_.matches` iteratee shorthand.
 * _.takeWhile(users, { 'user': 'barney', 'active': false });
 * // => objects for ['barney']
 *
 * // The `_.matchesProperty` iteratee shorthand.
 * _.takeWhile(users, ['active', false]);
 * // => objects for ['barney', 'fred']
 *
 * // The `_.property` iteratee shorthand.
 * _.takeWhile(users, 'active');
 * // => []
 */
function takeWhile(array, predicate) {
  return (array && array.length)
    ? baseWhile(array, baseIteratee(predicate, 3))
    : [];
}

module.exports = takeWhile;

```

## throttle函数节流

**throttle(func, [wait=0], [options=])**

>   Creates a throttled function that only invokes `func` at most once per every `wait` milliseconds. The throttled function comes with a `cancel` method to cancel delayed `func` invocations and a `flush` method to immediately invoke them. Provide `options` to indicate whether `func` should be invoked on the leading and/or trailing edge of the `wait` timeout. The `func` is invoked with the last arguments provided to the throttled function. Subsequent calls to the throttled function return the result of the last `func` invocation.
>
>   **Note:** If `leading` and `trailing` options are `true`, `func` is invoked on the trailing edge of the timeout only if the throttled function is invoked more than once during the `wait` timeout.
>
>   If `wait` is `0` and `leading` is `false`, `func` invocation is deferred until to the next tick, similar to `setTimeout` with a timeout of `0`.
>
>   See [David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/) for details over the differences between `_.throttle` and `_.debounce`.
>
>   创建一个节流函数，在 wait 秒内最多执行 `func` 一次的函数。 该函数提供一个 `cancel` 方法取消延迟的函数调用以及 `flush` 方法立即调用。 可以提供一个 options 对象决定如何调用 `func` 方法， options.leading 与|或 options.trailing 决定 wait 前后如何触发。 `func` 会传入最后一次传入的参数给这个函数。 随后调用的函数返回是最后一次 `func` 调用的结果。
>
>   **注意:** 如果 `leading` 和 `trailing` 都设定为 `true` 则 `func` 允许 trailing 方式调用的条件为: 在 `wait` 期间多次调用。
>
>   如果 `wait` 为 `0` 并且 `leading` 为 `false`, `func`调用将被推迟到下一个点，类似`setTimeout`为`0`的超时。
>
>   查看[David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/) 了解[`_.throttle`](https://www.lodashjs.com/docs/lodash.throttle#throttle) 与[`_.debounce`](https://www.lodashjs.com/docs/lodash.throttle#debounce) 的区别。

### 参数

+   func Function 要节流的函数
+   wait number 可选 要节流的时间 毫秒 默认 wait=0
+   options Object 可选 配置对象
    +    leading boolean 可选 指定在延迟开始前调用 默认 leading=false
    +    trailing boolean 可选 指定在延迟结束后调用 默认 trailing=true

### 返回

**Function**

返回节流的函数

### 源码

源码中使用的函数

+   [debounce](#debounce函数防抖)

+   [isObject](#isObject检查对象)

```js
var debounce = require('./debounce'),
    isObject = require('./isObject');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * Creates a throttled function that only invokes `func` at most once per
 * every `wait` milliseconds. The throttled function comes with a `cancel`
 * method to cancel delayed `func` invocations and a `flush` method to
 * immediately invoke them. Provide `options` to indicate whether `func`
 * should be invoked on the leading and/or trailing edge of the `wait`
 * timeout. The `func` is invoked with the last arguments provided to the
 * throttled function. Subsequent calls to the throttled function return the
 * result of the last `func` invocation.
 *
 * **Note:** If `leading` and `trailing` options are `true`, `func` is
 * invoked on the trailing edge of the timeout only if the throttled function
 * is invoked more than once during the `wait` timeout.
 *
 * If `wait` is `0` and `leading` is `false`, `func` invocation is deferred
 * until to the next tick, similar to `setTimeout` with a timeout of `0`.
 *
 * See [David Corbacho's article](https://css-tricks.com/debouncing-throttling-explained-examples/)
 * for details over the differences between `_.throttle` and `_.debounce`.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Function
 * @param {Function} func The function to throttle.
 * @param {number} [wait=0] The number of milliseconds to throttle invocations to.
 * @param {Object} [options={}] The options object.
 * @param {boolean} [options.leading=true]
 *  Specify invoking on the leading edge of the timeout.
 * @param {boolean} [options.trailing=true]
 *  Specify invoking on the trailing edge of the timeout.
 * @returns {Function} Returns the new throttled function.
 * @example
 *
 * // Avoid excessively updating the position while scrolling.
 * jQuery(window).on('scroll', _.throttle(updatePosition, 100));
 *
 * // Invoke `renewToken` when the click event is fired, but not more than once every 5 minutes.
 * var throttled = _.throttle(renewToken, 300000, { 'trailing': false });
 * jQuery(element).on('click', throttled);
 *
 * // Cancel the trailing throttled invocation.
 * jQuery(window).on('popstate', throttled.cancel);
 */
function throttle(func, wait, options) {
  var leading = true,
      trailing = true;

  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  if (isObject(options)) {
    leading = 'leading' in options ? !!options.leading : leading;
    trailing = 'trailing' in options ? !!options.trailing : trailing;
  }
  return debounce(func, wait, {
    'leading': leading,
    'maxWait': wait,
    'trailing': trailing
  });
}

module.exports = throttle;

```



## toFinite转为有限数字

**toFinite(value)**

>   Converts `value` to a finite number.
>
>   转换 `value` 为一个有限数字。

### 参数

+   value any 待转换的值

### 返回

**number**

转换后的值

### 源码

源码中涉及的函数

+   [toNumber](#toNumber转为数字)

```js
var toNumber = require('./toNumber');

/** Used as references for various `Number` constants. */
var INFINITY = 1 / 0,
    MAX_INTEGER = 1.7976931348623157e+308;

/**
 * Converts `value` to a finite number.
 *
 * @static
 * @memberOf _
 * @since 4.12.0
 * @category Lang
 * @param {*} value The value to convert.
 * @returns {number} Returns the converted number.
 * @example
 *
 * _.toFinite(3.2);
 * // => 3.2
 *
 * _.toFinite(Number.MIN_VALUE);
 * // => 5e-324
 *
 * _.toFinite(Infinity);
 * // => 1.7976931348623157e+308
 *
 * _.toFinite('3.2');
 * // => 3.2
 */
function toFinite(value) {
  if (!value) {
    return value === 0 ? value : 0;
  }
  value = toNumber(value);
  if (value === INFINITY || value === -INFINITY) {
    var sign = (value < 0 ? -1 : 1);
    return sign * MAX_INTEGER;
  }
  return value === value ? value : 0;
}

module.exports = toFinite;

```

## toInteger转为整数

### 参数

+ value 需要转换的值

### 返回

**Number**

被转换后的整数

### 源码

```js
/**
 * Converts `value` to an integer.
 *
 * **Note:** This method is loosely based on
 * [`ToInteger`](http://www.ecma-international.org/ecma-262/7.0/#sec-tointeger).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to convert.
 * @returns {number} Returns the converted integer.
 * @example
 *
 * _.toInteger(3.2);
 * // => 3
 *
 * _.toInteger(Number.MIN_VALUE);
 * // => 0
 *
 * _.toInteger(Infinity);
 * // => 1.7976931348623157e+308
 *
 * _.toInteger('3.2');
 * // => 3
 */
function toInteger(value) {
    var result = toFinite(value),
        remainder = result % 1;

    return result === result ? (remainder ? result - remainder : result) : 0;
}
```

## toLower字符串转小写

**toLower([string=''])**

>   Converts `string`, as a whole, to lower case just like toLowerCase
>
>   转换整个string字符串的字符为小写，类似String#toLowerCase。

### 参数 

+   string string 可选 默认为'' 待转换的字符串

### 返回

**string**

转换后的字符串

### 源码

源码中涉及的函数

+   [toString](#toString转为字符串)

```js
var toString = require('./toString');

/**
 * Converts `string`, as a whole, to lower case just like
 * [String#toLowerCase](https://mdn.io/toLowerCase).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category String
 * @param {string} [string=''] The string to convert.
 * @returns {string} Returns the lower cased string.
 * @example
 *
 * _.toLower('--Foo-Bar--');
 * // => '--foo-bar--'
 *
 * _.toLower('fooBar');
 * // => 'foobar'
 *
 * _.toLower('__FOO_BAR__');
 * // => '__foo_bar__'
 */
function toLower(value) {
  return toString(value).toLowerCase();
}

module.exports = toLower;

```

## toNumber转为数字

**toNumber(value)**

>   Converts `value` to a number.
>
>   转换 `value` 为一个数字。

### 参数

+   value any 待转换的值

### 返回

**number**

转换后的值

### 源码

源码中涉及的函数

+   [isSymbol ](#isSymbol检查符号)
+   [isObject](#isObject检查对象)
+   [baseTrim](#baseTrim)

```js
var baseTrim = require('./_baseTrim'),
    isObject = require('./isObject'),
    isSymbol = require('./isSymbol');

/** Used as references for various `Number` constants. */
var NAN = 0 / 0;

/** Used to detect bad signed hexadecimal string values. */
var reIsBadHex = /^[-+]0x[0-9a-f]+$/i;

/** Used to detect binary string values. */
var reIsBinary = /^0b[01]+$/i;

/** Used to detect octal string values. */
var reIsOctal = /^0o[0-7]+$/i;

/** Built-in method references without a dependency on `root`. */
var freeParseInt = parseInt;

/**
 * Converts `value` to a number.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to process.
 * @returns {number} Returns the number.
 * @example
 *
 * _.toNumber(3.2);
 * // => 3.2
 *
 * _.toNumber(Number.MIN_VALUE);
 * // => 5e-324
 *
 * _.toNumber(Infinity);
 * // => Infinity
 *
 * _.toNumber('3.2');
 * // => 3.2
 */
function toNumber(value) {
  if (typeof value == 'number') {
    return value;
  }
  if (isSymbol(value)) {
    return NAN;
  }
  if (isObject(value)) {
    var other = typeof value.valueOf == 'function' ? value.valueOf() : value;
    value = isObject(other) ? (other + '') : other;
  }
  if (typeof value != 'string') {
    return value === 0 ? value : +value;
  }
  value = baseTrim(value);
  var isBinary = reIsBinary.test(value);
  return (isBinary || reIsOctal.test(value))
    ? freeParseInt(value.slice(2), isBinary ? 2 : 8)
    : (reIsBadHex.test(value) ? NAN : +value);
}

module.exports = toNumber;

```



## toString转为字符串

> Converts `value` to a string. An empty string is returned for `null` and `undefined` values. The sign of `-0` is preserved.
>
> 将' value '转换为字符串。对于' null '和' undefined '值返回一个空字符串。符号' -0 '被保留。

### 参数

+   value 需要转换的值

### 返回

**String**

被转换后的字符串

### 源码

源码中设及的函数

+   [baseToString](#baseToString)

```js
var baseToString = require('./_baseToString');

/**
 * Converts `value` to a string. An empty string is returned for `null`
 * and `undefined` values. The sign of `-0` is preserved.
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Lang
 * @param {*} value The value to convert.
 * @returns {string} Returns the converted string.
 * @example
 *
 * _.toString(null);
 * // => ''
 *
 * _.toString(-0);
 * // => '-0'
 *
 * _.toString([1, 2, 3]);
 * // => '1,2,3'
 */
function toString(value) {
  return value == null ? '' : baseToString(value);
}

module.exports = toString;

```

## toUpper字符串转大写

**toUpper([string=''])**

>   Converts `string`, as a whole, to upper case just like toUpperCase
>
>   转换整个string字符串的字符为大写，类似String#toUpperCase.

### 参数

+   string string 可选 待转换的值 默认为''

### 返回

**string**

转换后的字符串

### 源码

源码中涉及的函数

+   [toString](#toString转为字符串)

```js
var toString = require('./toString');

/**
 * Converts `string`, as a whole, to upper case just like
 * [String#toUpperCase](https://mdn.io/toUpperCase).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category String
 * @param {string} [string=''] The string to convert.
 * @returns {string} Returns the upper cased string.
 * @example
 *
 * _.toUpper('--foo-bar--');
 * // => '--FOO-BAR--'
 *
 * _.toUpper('fooBar');
 * // => 'FOOBAR'
 *
 * _.toUpper('__foo_bar__');
 * // => '__FOO_BAR__'
 */
function toUpper(value) {
  return toString(value).toUpperCase();
}

module.exports = toUpper;

```



## union数组并集去重

**union([arrays])**

>   Creates an array of unique values, in order, from all given arrays using SameValueZero for equality comparisons.
>
>   创建一个按顺序排列的唯一值的数组。所有给定数组的元素值使用SameValueZero做等值比较。（注： arrays（数组）的并集，按顺序返回，返回数组的元素是唯一的）

### 参数

+   arrays ...Array 可选 待操作的各个数组

### 返回

**Array**

返回新的去重后的联合数组

### 源码

源码中涉及的函数

+   [baseFlatten](#baseFlatten)
+   [baseRest](#baseRest)
+   [baseUniq](#baseUniq)
+   [isArrayLikeObject](#isArrayLikeObject检查类数组对象)

```js
var baseFlatten = require('./_baseFlatten'),
    baseRest = require('./_baseRest'),
    - = require('./_baseUniq'),
    isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Creates an array of unique values, in order, from all given arrays using
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @returns {Array} Returns the new array of combined values.
 * @example
 *
 * _.union([2], [1, 2]);
 * // => [2, 1]
 */
var union = baseRest(function(arrays) {
  return baseUniq(baseFlatten(arrays, 1, isArrayLikeObject, true));
});

module.exports = union;

```

## unionBy数组并集迭代去重

**unionBy([arrays], [iteratee=_.identity])**

>   This method is like `_.union` except that it accepts `iteratee` which is invoked for each element of each `arrays` to generate the criterion by which uniqueness is computed. Result values are chosen from the first  array in which the value occurs. The iteratee is invoked with one argument: (value).
>
>   这个方法类似_.union ，除了它接受一个 iteratee （迭代函数），调用每一个数组（array）的每个元素以产生唯一性计算的标准。iteratee 会传入一个参数：(value)。

### 参数

+   arrays ...Array 可选 待并和待去重的各数组
+   iteratee Array|Function|Object|string 可选 迭代器 调用每个元素

### 返回

**Array**

返回一个新的联合数组。

### 源码

源码中涉及到的函数

+   [baseFlatten](#baseFlatten)
+   [baseIteratee](#baseIteratee)
+   [baseRest](#baseRest)
+   [baseUniq](#baseUniq)
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)
+   [last ](#last获取数组最后一个元素)

```js
var baseFlatten = require('./_baseFlatten'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    baseUniq = require('./_baseUniq'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.union` except that it accepts `iteratee` which is
 * invoked for each element of each `arrays` to generate the criterion by
 * which uniqueness is computed. Result values are chosen from the first
 * array in which the value occurs. The iteratee is invoked with one argument:
 * (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new array of combined values.
 * @example
 *
 * _.unionBy([2.1], [1.2, 2.3], Math.floor);
 * // => [2.1, 1.2]
 *
 * // The `_.property` iteratee shorthand.
 * _.unionBy([{ 'x': 1 }], [{ 'x': 2 }, { 'x': 1 }], 'x');
 * // => [{ 'x': 1 }, { 'x': 2 }]
 */
var unionBy = baseRest(function(arrays) {
  var iteratee = last(arrays);
  if (isArrayLikeObject(iteratee)) {
    iteratee = undefined;
  }
  return baseUniq(baseFlatten(arrays, 1, isArrayLikeObject, true), baseIteratee(iteratee, 2));
});

module.exports = unionBy;

```

## unionWith数组并集比较去重

**unionWith([arrays], [comparator])**

>   This method is like `_.union` except that it accepts `comparator` which is invoked to compare elements of `arrays`. Result values are chosen from the first array in which the value occurs. The comparator is invoked with two arguments: (arrVal, othVal).
>
>   这个方法类似_.union， 除了它接受一个 comparator 调用比较arrays数组的每一个元素。 comparator 调用时会传入2个参数： (arrVal, othVal)。

### 参数

+   arrays ...Array 可选 待并和待去重的各数组
+   comparator Function 可选 比较器，调用每个元素。

### 返回

**Array**

返回一个新的联合数组。

### 源码

源码中涉及否函数

+   [baseFlatten](#baseFlatten)
+   [baseRest](#baseRest)
+   [baseUniq](#baseUniq)
+   [last ](#last获取数组最后一个元素)
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

```js
var baseFlatten = require('./_baseFlatten'),
    baseRest = require('./_baseRest'),
    baseUniq = require('./_baseUniq'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.union` except that it accepts `comparator` which
 * is invoked to compare elements of `arrays`. Result values are chosen from
 * the first array in which the value occurs. The comparator is invoked
 * with two arguments: (arrVal, othVal).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of combined values.
 * @example
 *
 * var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
 * var others = [{ 'x': 1, 'y': 1 }, { 'x': 1, 'y': 2 }];
 *
 * _.unionWith(objects, others, _.isEqual);
 * // => [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }, { 'x': 1, 'y': 1 }]
 */
var unionWith = baseRest(function(arrays) {
  var comparator = last(arrays);
  comparator = typeof comparator == 'function' ? comparator : undefined;
  return baseUniq(baseFlatten(arrays, 1, isArrayLikeObject, true), undefined, comparator);
});

module.exports = unionWith;

```

## uniq数组去重

**uniq(array)**

>   Creates a duplicate-free version of an array, using SameValueZero for equality comparisons, in which only the first occurrence of each element is kept. The order of result values is determined by the order they occur in the array.
>
>   创建一个去重后的array数组副本。使用了SameValueZero 做等值比较。只有第一次出现的元素才会被保留。

### 参数

+   array Array 待去重数组

### 返回

**Array**

返回新的去重后的数组。

### 源码

源码中涉及的函数

+   [baseUniq](#baseUniq)

```js
var baseUniq = require('./_baseUniq');

/**
 * Creates a duplicate-free version of an array, using
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons, in which only the first occurrence of each element
 * is kept. The order of result values is determined by the order they occur
 * in the array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @returns {Array} Returns the new duplicate free array.
 * @example
 *
 * _.uniq([2, 1, 2]);
 * // => [2, 1]
 */
function uniq(array) {
  return (array && array.length) ? baseUniq(array) : [];
}

module.exports = uniq;

```

## uniqBy数组迭代去重

**uniqBy(array, [iteratee=_.identity])**

>   This method is like `_.uniq` except that it accepts `iteratee` which is invoked for each element in `array` to generate the criterion by which uniqueness is computed. The order of result values is determined by the order they occur in the array. The iteratee is invoked with one argument: (value).
>
>   这个方法类似_.uniq ，除了它接受一个 iteratee （迭代函数），调用每一个数组（array）的每个元素以产生唯一性计算的标准。iteratee 调用时会传入一个参数：(value)。

### 参数

+   array Array 待去重的数组
+   iteratee Array|Function|Object|string 可选 迭代器 调用每个元素

### 返回

**Array**

返回新的去重后的数组。

### 源码

源码中涉及的函数

+   [baseIteratee](#baseIteratee)
+   [baseUniq](#baseUniq)

```js
var baseIteratee = require('./_baseIteratee'),
    baseUniq = require('./_baseUniq');

/**
 * This method is like `_.uniq` except that it accepts `iteratee` which is
 * invoked for each element in `array` to generate the criterion by which
 * uniqueness is computed. The order of result values is determined by the
 * order they occur in the array. The iteratee is invoked with one argument:
 * (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 * @example
 *
 * _.uniqBy([2.1, 1.2, 2.3], Math.floor);
 * // => [2.1, 1.2]
 *
 * // The `_.property` iteratee shorthand.
 * _.uniqBy([{ 'x': 1 }, { 'x': 2 }, { 'x': 1 }], 'x');
 * // => [{ 'x': 1 }, { 'x': 2 }]
 */
function uniqBy(array, iteratee) {
  return (array && array.length) ? baseUniq(array, baseIteratee(iteratee, 2)) : [];
}

module.exports = uniqBy;

```

## uniqWith数组比较去重

**uniqWith(array, [comparator])**

>   This method is like `_.uniq` except that it accepts `comparator` which is invoked to compare elements of `array`. The order of result values is determined by the order they occur in the array.The comparator is invoked with two arguments: (arrVal, othVal).
>
>   这个方法类似_.uniq， 除了它接受一个 comparator 调用比较arrays数组的每一个元素。 comparator 调用时会传入2个参数： (arrVal, othVal)。

### 参数

+   array Array 待去重的数组
+   comparator Function 可选 比较器  返回true会视为两元素相同，影响去重

### 返回

**Array**

返回去重后的新的数组

### 源码

源码中涉及的函数

+   [baseUniq](#baseUniq)

```js
var baseUniq = require('./_baseUniq');

/**
 * This method is like `_.uniq` except that it accepts `comparator` which
 * is invoked to compare elements of `array`. The order of result values is
 * determined by the order they occur in the array.The comparator is invoked
 * with two arguments: (arrVal, othVal).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 * @example
 *
 * var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }, { 'x': 1, 'y': 2 }];
 *
 * _.uniqWith(objects, _.isEqual);
 * // => [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }]
 */
function uniqWith(array, comparator) {
  comparator = typeof comparator == 'function' ? comparator : undefined;
  return (array && array.length) ? baseUniq(array, undefined, comparator) : [];
}

module.exports = uniqWith;

```

## unZip数组解压缩

**unzip(array)**

>   This method is like `_.zip` except that it accepts an array of grouped elements and creates an array regrouping the elements to their pre-zip configuration.
>
>   这个方法类似于_.zip，除了它接收分组元素的数组，并且创建一个数组，分组元素到打包前的结构。（：返回数组的第一个元素包含所有的输入数组的第一元素，第一个元素包含了所有的输入数组的第二元素，依此类推。）

### 参数

+   array Array 要处理的分组元素数组。

### 返回

**Array**

返回重组元素的新数组。

### 解析

```js
var arrayFilter = require('./_arrayFilter'),
    arrayMap = require('./_arrayMap'),
    baseProperty = require('./_baseProperty'),//通过键名，获取对象键值
    baseTimes = require('./_baseTimes'),//调用n次函数，返回每次调用结果组成的数组
    isArrayLikeObject = require('./isArrayLikeObject');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * This method is like `_.zip` except that it accepts an array of grouped
 * elements and creates an array regrouping the elements to their pre-zip
 * configuration.
 *
 * @static
 * @memberOf _
 * @since 1.2.0
 * @category Array
 * @param {Array} array The array of grouped elements to process.
 * @returns {Array} Returns the new array of regrouped elements.
 * @example
 *
 * var zipped = _.zip(['a', 'b'], [1, 2], [true, false]);
 * // => [['a', 1, true], ['b', 2, false]]
 *
 * _.unzip(zipped);
 * // => [['a', 'b'], [1, 2], [true, false]]
 */
function unzip(array) {
  //如果数组不存在或者长度为0，返回空数组
  if (!(array && array.length)) {
    return [];
  }
  var length = 0;
  //过滤，group为array中的每个元素，筛选出所有类数组对象，并取得最大数组长度，这个最大长度length就是要划分的组数
  array = arrayFilter(array, function(group) {
    if (isArrayLikeObject(group)) {
      length = nativeMax(group.length, length);
      return true;
    }
  });
  //调用length次函数（因为要划出length个组），每次调用的结果组成一个结果数组返回
  return baseTimes(length, function(index) {
    //每次调用中，使用arrayMap，对数组中的每一项进行处理，获取数组中每个元素（此时的元素也都是数组）的第index项，并将所有
    //元素的第index项组成一个数组，作为第index次函数调用的结果
    return arrayMap(array, baseProperty(index));
  });
}

module.exports = unzip;

```



### 源码

源码中涉及的函数

+   [arrayFilter](#arrayFilter)

+   [arrayMap](#arrayMap)

+   [baseProperty](#baseProperty)

+   [baseTimes](#baseTimes)

+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

    ```js
    var arrayFilter = require('./_arrayFilter'),
        arrayMap = require('./_arrayMap'),
        baseProperty = require('./_baseProperty'),
        baseTimes = require('./_baseTimes'),
        isArrayLikeObject = require('./isArrayLikeObject');
    
    /* Built-in method references for those with the same name as other `lodash` methods. */
    var nativeMax = Math.max;
    
    /**
     * This method is like `_.zip` except that it accepts an array of grouped
     * elements and creates an array regrouping the elements to their pre-zip
     * configuration.
     *
     * @static
     * @memberOf _
     * @since 1.2.0
     * @category Array
     * @param {Array} array The array of grouped elements to process.
     * @returns {Array} Returns the new array of regrouped elements.
     * @example
     *
     * var zipped = _.zip(['a', 'b'], [1, 2], [true, false]);
     * // => [['a', 1, true], ['b', 2, false]]
     *
     * _.unzip(zipped);
     * // => [['a', 'b'], [1, 2], [true, false]]
     */
    function unzip(array) {
      if (!(array && array.length)) {
        return [];
      }
      var length = 0;
      array = arrayFilter(array, function(group) {
        if (isArrayLikeObject(group)) {
          length = nativeMax(group.length, length);
          return true;
        }
      });
      return baseTimes(length, function(index) {
        return arrayMap(array, baseProperty(index));
      });
    }
    
    module.exports = unzip;
    
    ```

## unZipWith数组指定分组

**unzipWith(array, [iteratee=_.identity])**

>   This method is like `_.unzip` except that it accepts `iteratee` to specify how regrouped values should be combined. The iteratee is invoked with the  elements of each group: (...group).
>
>   此方法类似于_.unzip，除了它接受一个iteratee指定重组值应该如何被组合。iteratee 调用时会传入每个分组的值： (...group)。

### 参数

+   array Array 要处理的分组元素数组
+   iteratee Function 可选 迭代器 这个函数用来组合重组的值，调用每个分组后的所有组内元素，rest参数

### 返回

**Array**

返回重组元素的新数组。

### 示例

**示例1**

```js
console.log(unzipWith([[1, 10, 100], [2, 20, 200]], Math.max))
//[ 2, 20, 200 ]
```

**示例2**

```js
console.log(unzipWith([[1, 10, 100], [2, 20, 200]], add))
//[ 3, 30, 300 ]
```

**示例3**

```js
console.log(unzipWith([[1, 10, 100], [2, 20, 200]], (...rest) => {
    console.log(rest)
    return 'a'
}))
//[ 1, 2 ]
//[ 10, 20 ]
//[ 100, 200 ]
//[ 'a', 'a', 'a' ]
```

### 源码

源码中涉及的函数

+   [apply](#apply)
+   [arrayMap](#arrayMap)
+   [unZip](#unZip数组解压缩)

```js
var apply = require('./_apply'),
    arrayMap = require('./_arrayMap'),
    unzip = require('./unzip');
/**
 * This method is like `_.unzip` except that it accepts `iteratee` to specify
 * how regrouped values should be combined. The iteratee is invoked with the
 * elements of each group: (...group).
 *
 * @static
 * @memberOf _
 * @since 3.8.0
 * @category Array
 * @param {Array} array The array of grouped elements to process.
 * @param {Function} [iteratee=_.identity] The function to combine
 *  regrouped values.
 * @returns {Array} Returns the new array of regrouped elements.
 * @example
 *
 * var zipped = _.zip([1, 2], [10, 20], [100, 200]);
 * // => [[1, 10, 100], [2, 20, 200]]
 *
 * _.unzipWith(zipped, _.add);
 * // => [3, 30, 300]
 */
function unzipWith(array, iteratee) {
    if (!(array && array.length)) {
        return [];
    }
    var result = unzip(array);
    if (iteratee == null) {
        return result;
    }
    return arrayMap(result, function (group) {
        return apply(iteratee, undefined, group);
    });
}
module.exports = unzipWith;

```

## values对象值数组

**values(object)**

>   Creates an array of the own enumerable string keyed property values of `object`.

**注意:** 非对象的值会强制转换为对象。

### 参数

+   object Object 待获取值数组的对象

### 返回

**Array**

返回对象可枚举属性的值数组

### 源码

源码中涉及的函数

+   [baseValues](#baseValues)
+   [keys](#keys对象属性名数组)

```js
var baseValues = require('./_baseValues'),
    keys = require('./keys');

/**
 * Creates an array of the own enumerable string keyed property values of `object`.
 *
 * **Note:** Non-object values are coerced to objects.
 *
 * @static
 * @since 0.1.0
 * @memberOf _
 * @category Object
 * @param {Object} object The object to query.
 * @returns {Array} Returns the array of property values.
 * @example
 *
 * function Foo() {
 *   this.a = 1;
 *   this.b = 2;
 * }
 *
 * Foo.prototype.c = 3;
 *
 * _.values(new Foo);
 * // => [1, 2] (iteration order is not guaranteed)
 *
 * _.values('hi');
 * // => ['h', 'i']
 */
function values(object) {
  return object == null ? [] : baseValues(object, keys(object));
}

module.exports = values;

```



## without数组剔除给定值

**without(array, [values])**

>   Creates an array excluding all given values using SameValueZero for equality comparisons.
>
>   创建一个剔除所有给定值的新数组，剔除值的时候，使用SameValueZero做相等比较。

**注意: 不像_.pull, 这个方法会返回一个新数组。**

注意:和difference不同，without接收的是不定量的给定值...any ,每一个参数是一个给定值。difference接收的是不定量的给定值数组...Array，...Array每一个都是一个由给定值构成的数组

### 参数

+   array Array 待过滤的数组
+   values ...any 可选，待剔除的值

### 返回

**Array**

返回过滤值后的新数组。

### 示例

**示例1**

```js
console.log(without([2, 1, 2, 3], 1, 2))//[ 3 ]
```

### 源码

源码中涉及的函数

+   [baseDifference](#baseDifference)
+   [baseRest](#baseRest) 
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

```js
var baseDifference = require('./_baseDifference'),
    baseRest = require('./_baseRest'),
    isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Creates an array excluding all given values using
 * [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons.
 *
 * **Note:** Unlike `_.pull`, this method returns a new array.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {Array} array The array to inspect.
 * @param {...*} [values] The values to exclude.
 * @returns {Array} Returns the new array of filtered values.
 * @see _.difference, _.xor
 * @example
 *
 * _.without([2, 1, 2, 3], 1, 2);
 * // => [3]
 */
var without = baseRest(function(array, values) {
  return isArrayLikeObject(array)
    ? baseDifference(array, values)
    : [];
});

module.exports = without;

```

## xor给定数组唯一值

**xor([arrays])**

>   Creates an array of unique values that is the symmetric difference of the given arrays. The order of result values is determined by the order they occur in the arrays.
>
>   创建一个给定数组唯一值的数组，使用symmetric difference做等值比较。返回值的顺序取决于他们数组的出现顺序。

从一系列数组中选取唯一值

### 参数

+   array ...Array 可选 rest参数，待检查的数组

### 返回

**Array**

返回过滤后的新数组

### 示例

**示例1**

```js
console.log(xor([2, 1], [2, 3]))//[ 1, 3 ]
```

### 源码

源码中涉及的函数

+   [arrayFilter](#arrayFilter)
+   [baseRest](#baseRest)
+   [baseXor](#baseXor)
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

```js
var arrayFilter = require('./_arrayFilter'),
    baseRest = require('./_baseRest'),
    baseXor = require('./_baseXor'),
    isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Creates an array of unique values that is the
 * [symmetric difference](https://en.wikipedia.org/wiki/Symmetric_difference)
 * of the given arrays. The order of result values is determined by the order
 * they occur in the arrays.
 *
 * @static
 * @memberOf _
 * @since 2.4.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @returns {Array} Returns the new array of filtered values.
 * @see _.difference, _.without
 * @example
 *
 * _.xor([2, 1], [2, 3]);
 * // => [1, 3]
 */
var xor = baseRest(function(arrays) {
  return baseXor(arrayFilter(arrays, isArrayLikeObject));
});

module.exports = xor;

```

## xorBy给定数组迭代唯一值

**xorBy([arrays], [iteratee=_.identity])**

>   This method is like `_.xor` except that it accepts `iteratee` which is invoked for each element of each `arrays` to generate the criterion by  which by which they're compared. The order of result values is determined  by the order they occur in the arrays. The iteratee is invoked with one argument: (value).
>
>   这个方法类似_.xor ，除了它接受 iteratee（迭代器），这个迭代器 调用每一个 arrays（数组）的每一个值，以生成比较的新值。iteratee 调用一个参数：(value).

### 参数

+   arrays ...Array 可选 rest 待检查的各个数组
+   iteratee Array|Function|Object|string 可选 迭代器 调用每一个元素

### 返回

**Array**

返回过滤后的新数组

### 源码

源码中涉及的函数

+   [arrayFilter](#arrayFilter)
+   [baseIteratee](#baseIteratee)
+   [baseRest](#baseRest)
+   [baseXor](#baseXor)
+   [last ](#last获取数组最后一个元素)
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

```js
var arrayFilter = require('./_arrayFilter'),
    baseIteratee = require('./_baseIteratee'),
    baseRest = require('./_baseRest'),
    baseXor = require('./_baseXor'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.xor` except that it accepts `iteratee` which is
 * invoked for each element of each `arrays` to generate the criterion by
 * which by which they're compared. The order of result values is determined
 * by the order they occur in the arrays. The iteratee is invoked with one
 * argument: (value).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [iteratee=_.identity] The iteratee invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * _.xorBy([2.1, 1.2], [2.3, 3.4], Math.floor);
 * // => [1.2, 3.4]
 *
 * // The `_.property` iteratee shorthand.
 * _.xorBy([{ 'x': 1 }], [{ 'x': 2 }, { 'x': 1 }], 'x');
 * // => [{ 'x': 2 }]
 */
var xorBy = baseRest(function(arrays) {
  var iteratee = last(arrays);
  if (isArrayLikeObject(iteratee)) {
    iteratee = undefined;
  }
  return baseXor(arrayFilter(arrays, isArrayLikeObject), baseIteratee(iteratee, 2));
});

module.exports = xorBy;

```

## xorWith给定数组条件唯一值

**xorWith([arrays], [comparator])**

>   This method is like `_.xor` except that it accepts `comparator` which is  invoked to compare elements of `arrays`. The order of result values is  determined by the order they occur in the arrays. The comparator is invoked  with two arguments: (arrVal, othVal).
>
>   该方法是像_.xor，除了它接受一个 comparator ，以调用比较数组的元素。 comparator 调用2个参数：(arrVal, othVal).

### 参数

+   arrays ...Array 可选 rest 待检查的各个数组
+   comparator Function 可选 比较器

### 返回

**Array**

返回过滤后的新数组

### 源码

源码中涉及的函数

+   [arrayFilter](#arrayFilter)
+   [baseRest](#baseRest)
+   [baseXor](#baseXor)
+   [last ](#last获取数组最后一个元素)
+   [isArrayLikeObject ](#isArrayLikeObject检查类数组对象)

```js
var arrayFilter = require('./_arrayFilter'),
    baseRest = require('./_baseRest'),
    baseXor = require('./_baseXor'),
    isArrayLikeObject = require('./isArrayLikeObject'),
    last = require('./last');

/**
 * This method is like `_.xor` except that it accepts `comparator` which is
 * invoked to compare elements of `arrays`. The order of result values is
 * determined by the order they occur in the arrays. The comparator is invoked
 * with two arguments: (arrVal, othVal).
 *
 * @static
 * @memberOf _
 * @since 4.0.0
 * @category Array
 * @param {...Array} [arrays] The arrays to inspect.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 * @example
 *
 * var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }];
 * var others = [{ 'x': 1, 'y': 1 }, { 'x': 1, 'y': 2 }];
 *
 * _.xorWith(objects, others, _.isEqual);
 * // => [{ 'x': 2, 'y': 1 }, { 'x': 1, 'y': 1 }]
 */
var xorWith = baseRest(function(arrays) {
  var comparator = last(arrays);
  comparator = typeof comparator == 'function' ? comparator : undefined;
  return baseXor(arrayFilter(arrays, isArrayLikeObject), undefined, comparator);
});

module.exports = xorWith;

```

## zip数组重新分组

**zip([arrays])**

>   Creates an array of grouped elements, the first of which contains the  first elements of the given arrays, the second of which contains the second elements of the given arrays, and so on.
>
>   创建一个分组元素的数组，数组的第一个元素包含所有给定数组的第一个元素，数组的第二个元素包含所有给定数组的第二个元素，以此类推。

### 参数

+   arrays ...Array 可选 rest 待重新分组的各个数组

### 返回

**Array**

返回分组元素的新数组。

### 源码

源码中涉及的函数

+   [baseRest](#baseRest)
+   [unZip](#unZip数组解压缩)

```js
var baseRest = require('./_baseRest'),
    unzip = require('./unzip');

/**
 * Creates an array of grouped elements, the first of which contains the
 * first elements of the given arrays, the second of which contains the
 * second elements of the given arrays, and so on.
 *
 * @static
 * @memberOf _
 * @since 0.1.0
 * @category Array
 * @param {...Array} [arrays] The arrays to process.
 * @returns {Array} Returns the new array of grouped elements.
 * @example
 *
 * _.zip(['a', 'b'], [1, 2], [true, false]);
 * // => [['a', 1, true], ['b', 2, false]]
 */
var zip = baseRest(unzip);

module.exports = zip;

```

## zipObject数组构造对象

**zipObject([props=[]], [values=[]])**

>   This method is like `_.fromPairs` except that it accepts two arrays, one of property identifiers and one of corresponding values.
>
>   这个方法类似_.fromPairs，除了它接受2个数组，第一个数组中的值作为属性标识符（属性名），第二个数组中的值作为相应的属性值。

### 参数

+   props Array 属性名数组
+   values Array 属性值数组

### 返回

**Object**

返回构造好的新对象

### 示例

**示例1**

```js
console.log(zipObject(['a', '3'], [1, 2]))//{ '3': 2, a: 1 }
```

### 源码

源码中涉及的函数

+   [assignValue](#assignValue)
+   [baseZipObject](#baseZipObject)

```js
var assignValue = require('./_assignValue'),
    baseZipObject = require('./_baseZipObject');

/**
 * This method is like `_.fromPairs` except that it accepts two arrays,
 * one of property identifiers and one of corresponding values.
 *
 * @static
 * @memberOf _
 * @since 0.4.0
 * @category Array
 * @param {Array} [props=[]] The property identifiers.
 * @param {Array} [values=[]] The property values.
 * @returns {Object} Returns the new object.
 * @example
 *
 * _.zipObject(['a', 'b'], [1, 2]);
 * // => { 'a': 1, 'b': 2 }
 */
function zipObject(props, values) {
  return baseZipObject(props || [], values || [], assignValue);
}

module.exports = zipObject;

```

## zipObjectDeep数组构造对象支持深度路径

**zipObjectDeep([props=[]], [values=[]])**

>   This method is like `_.zipObject` except that it supports property paths.
>
>   这个方法类似_.zipObject，除了它支持属性路径。

### 参数

+   props Array 属性名数组
+   values Array 属性值数组

### 返回

**Object**

返回新的构造好的数组

### 示例

**示例1**

```js
console.log(JSON.stringify(zipObjectDeep(['a.b[0].c', 'a.b[1].d'], [1, 2])))
//{"a":{"b":[{"c":1},{"d":2}]}}
```



### 源码

源码中涉及的函数

+   [baseSet](#baseSet)
+   [baseZipObject](#baseZipObject)

```js
var baseSet = require('./_baseSet'),
    baseZipObject = require('./_baseZipObject');

/**
 * This method is like `_.zipObject` except that it supports property paths.
 *
 * @static
 * @memberOf _
 * @since 4.1.0
 * @category Array
 * @param {Array} [props=[]] The property identifiers.
 * @param {Array} [values=[]] The property values.
 * @returns {Object} Returns the new object.
 * @example
 *
 * _.zipObjectDeep(['a.b[0].c', 'a.b[1].d'], [1, 2]);
 * // => { 'a': { 'b': [{ 'c': 1 }, { 'd': 2 }] } }
 */
function zipObjectDeep(props, values) {
  return baseZipObject(props || [], values || [], baseSet);
}

module.exports = zipObjectDeep;

```

## zipWith数组迭代分组

>   This method is like `_.zip` except that it accepts `iteratee` to specify how grouped values should be combined. The iteratee is invoked with the elements of each group: (...group).
>
>   这个方法类似于_.zip，不同之处在于它接受一个 iteratee（迭代函数），来 指定分组的值应该如何被组合。 该iteratee调用每个组的元素： (...group).

### 参数

+   array ...Array 可选 rest 待重组的数组
+   iteratee Function 可选 用来组合分组的值

### 返回

**Array**

返回新的重组后的数组

### 源码

源码中涉及的函数

+   [baseRest](#baseRest)
+   [unZipWith ](#unZipWith数组指定分组)

```js
var baseRest = require('./_baseRest'),
    unzipWith = require('./unzipWith');

/**
 * This method is like `_.zip` except that it accepts `iteratee` to specify
 * how grouped values should be combined. The iteratee is invoked with the
 * elements of each group: (...group).
 *
 * @static
 * @memberOf _
 * @since 3.8.0
 * @category Array
 * @param {...Array} [arrays] The arrays to process.
 * @param {Function} [iteratee=_.identity] The function to combine
 *  grouped values.
 * @returns {Array} Returns the new array of grouped elements.
 * @example
 *
 * _.zipWith([1, 2], [10, 20], [100, 200], function(a, b, c) {
 *   return a + b + c;
 * });
 * // => [111, 222]
 */
var zipWith = baseRest(function(arrays) {
  var length = arrays.length,
      iteratee = length > 1 ? arrays[length - 1] : undefined;

  iteratee = typeof iteratee == 'function' ? (arrays.pop(), iteratee) : undefined;
  return unzipWith(arrays, iteratee);
});

module.exports = zipWith;

```

## 内部对象

### MapCache

>    Creates a map cache object to store key-value pairs.
>
>   创建一个map缓存对象来存储键值对

#### 实现

```js
var mapCacheClear = require('./_mapCacheClear'),
    mapCacheDelete = require('./_mapCacheDelete'),
    mapCacheGet = require('./_mapCacheGet'),
    mapCacheHas = require('./_mapCacheHas'),
    mapCacheSet = require('./_mapCacheSet');

/**
 * Creates a map cache object to store key-value pairs.
 *
 * @private
 * @constructor
 * @param {Array} [entries] The key-value pairs to cache.
 */
function MapCache(entries) {
  var index = -1,
      length = entries == null ? 0 : entries.length;

  this.clear();
  while (++index < length) {
    var entry = entries[index];
    this.set(entry[0], entry[1]);
  }
}

// Add methods to `MapCache`.
MapCache.prototype.clear = mapCacheClear;
MapCache.prototype['delete'] = mapCacheDelete;
MapCache.prototype.get = mapCacheGet;
MapCache.prototype.has = mapCacheHas;
MapCache.prototype.set = mapCacheSet;

module.exports = MapCache;

```



## 内部方法

### apply

>   A faster alternative to `Function#apply`, this function invokes `func`  with the `this` binding of `thisArg` and the arguments of `args`.
>
>   与#apply函数相比，这个函数通过绑定thisArg和args的参数来调用func。

#### 参数

+   func Function 待调用函数
+   thisArg any func绑定的this
+   args Array func的参数

#### 返回

**Any**

返回func函数的调用结果

#### 源码

```js
/**
 * A faster alternative to `Function#apply`, this function invokes `func`
 * with the `this` binding of `thisArg` and the arguments of `args`.
 *
 * @private
 * @param {Function} func The function to invoke.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {Array} args The arguments to invoke `func` with.
 * @returns {*} Returns the result of `func`.
 */
function apply(func, thisArg, args) {
  switch (args.length) {
    case 0: return func.call(thisArg);
    case 1: return func.call(thisArg, args[0]);
    case 2: return func.call(thisArg, args[0], args[1]);
    case 3: return func.call(thisArg, args[0], args[1], args[2]);
  }
  return func.apply(thisArg, args);
}

module.exports = apply;

```

### arrayAggregator

>   A specialized version of `baseAggregator` for arrays.
>
>   针对数组的baseAggregator的另一实现

#### 参数

+   array Array 待统计数组
+   setter Function 累加器的赋值函数
+   iteratee 用于为累加器转换key
+   accumulator Object 初始化的累加器对象

#### 返回

**Function**？？

**Object** 这里应该返回的是一个累加器对象

#### 解析

```js
/**
 * A specialized version of `baseAggregator` for arrays.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} setter The function to set `accumulator` values.
 * @param {Function} iteratee The iteratee to transform keys.
 * @param {Object} accumulator The initial aggregated object.
 * @returns {Function} Returns `accumulator`.
 */
function arrayAggregator(array, setter, iteratee, accumulator) {
  var index = -1,
      length = array == null ? 0 : array.length;
  //遍历数组
  while (++index < length) {
    var value = array[index];
    //setter前3个参数指result，value，key，作用为向result对象中的key赋值
    //iteratee(value)生成累加器对象的键key
    setter(accumulator, value, iteratee(value), array);
  }
  return accumulator;
}

module.exports = arrayAggregator;

```



#### 源码

```js
/**
 * A specialized version of `baseAggregator` for arrays.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} setter The function to set `accumulator` values.
 * @param {Function} iteratee The iteratee to transform keys.
 * @param {Object} accumulator The initial aggregated object.
 * @returns {Function} Returns `accumulator`.
 */
function arrayAggregator(array, setter, iteratee, accumulator) {
  var index = -1,
      length = array == null ? 0 : array.length;

  while (++index < length) {
    var value = array[index];
    setter(accumulator, value, iteratee(value), array);
  }
  return accumulator;
}

module.exports = arrayAggregator;

```



### arrayEach

>   A specialized version of `_.forEach` for arrays without support for iteratee shorthands.
>
>   forEach函数的特殊版本，不支持迭代器简写

#### 参数

+   array Array 可选 待遍历数组
+   iteratee Function 迭代器，每次遍历调用

#### 返回

**Array**

返回array

#### 源码

```js
/**
 * A specialized version of `_.forEach` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns `array`.
 */
function arrayEach(array, iteratee) {
  var index = -1,
      length = array == null ? 0 : array.length;

  while (++index < length) {
    if (iteratee(array[index], index, array) === false) {
      break;
    }
  }
  return array;
}

module.exports = arrayEach;

```

### arrayEachRight

>   A specialized version of `_.forEachRight` for arrays without support for iteratee shorthands.
>
>   forEachRight的特殊实现版本，不支持迭代器简写

#### 参数

+   array Array 可选 待遍历的数组
+   iteratee Function 迭代器 每次遍历调用

#### 返回

**Array** 

返回数组array

#### 源码

```js
/**
 * A specialized version of `_.forEachRight` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns `array`.
 */
function arrayEachRight(array, iteratee) {
  var length = array == null ? 0 : array.length;

  while (length--) {
    if (iteratee(array[length], length, array) === false) {
      break;
    }
  }
  return array;
}

module.exports = arrayEachRight;

```

### arrayEvery

>   A specialized version of `_.every` for arrays without support for iteratee shorthands.
>
>   every函数的特殊实现版本，不支持迭代器简写

#### 参数

+   array Array 待遍历的数组
+   predicate Function 断言函数，每次遍历调用

#### 返回

**boolean**

如果所有元素经 predicate（断言函数） 检查后都都返回真值，那么就返回true，否则返回 false 。

#### 源码

```js
/**
 * A specialized version of `_.every` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {boolean} Returns `true` if all elements pass the predicate check,
 *  else `false`.
 */
function arrayEvery(array, predicate) {
  var index = -1,
      length = array == null ? 0 : array.length;

  while (++index < length) {
    if (!predicate(array[index], index, array)) {
      return false;
    }
  }
  return true;
}

module.exports = arrayEvery;

```



### arrayFilter

>   A specialized version of `_.filter` for arrays without support for iteratee shorthands.
>
>   filter的实现，不支持迭代器简写

#### 参数

+   array  Array 待过滤数组
+   predicate Function 断言函数？过滤条件， 调用每个元素，返回true上的，将被存放到结果数组中

#### 返回

**Array**

返回新的过滤后的数组

#### 源码

```js
/**
 * A specialized version of `_.filter` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {Array} Returns the new filtered array.
 */
function arrayFilter(array, predicate) {
  var index = -1,
      length = array == null ? 0 : array.length,
      resIndex = 0,
      result = [];

  while (++index < length) {
    var value = array[index];
    if (predicate(value, index, array)) {
      result[resIndex++] = value;
    }
  }
  return result;
}

module.exports = arrayFilter;

```

### arrayLikeKeys

>   Creates an array of the enumerable property names of the array-like `value`.
>
>   创建数组类' value '的可枚举属性名的数组。

#### 参数

+   value any 待操作值
+   inherited boolean 是否返回继承的属性名。

#### 返回

**Array**

返回属性名数组

#### 源码

```js
var baseTimes = require('./_baseTimes'),
    isArguments = require('./isArguments'),
    isArray = require('./isArray'),
    isBuffer = require('./isBuffer'),
    isIndex = require('./_isIndex'),
    isTypedArray = require('./isTypedArray');

/** Used for built-in method references. */
var objectProto = Object.prototype;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/**
 * Creates an array of the enumerable property names of the array-like `value`.
 *
 * @private
 * @param {*} value The value to query.
 * @param {boolean} inherited Specify returning inherited property names.
 * @returns {Array} Returns the array of property names.
 */
function arrayLikeKeys(value, inherited) {
  var isArr = isArray(value),
      isArg = !isArr && isArguments(value),
      isBuff = !isArr && !isArg && isBuffer(value),
      isType = !isArr && !isArg && !isBuff && isTypedArray(value),
      skipIndexes = isArr || isArg || isBuff || isType,
      result = skipIndexes ? baseTimes(value.length, String) : [],
      length = result.length;

  for (var key in value) {
    if ((inherited || hasOwnProperty.call(value, key)) &&
        !(skipIndexes && (
           // Safari 9 has enumerable `arguments.length` in strict mode.
           key == 'length' ||
           // Node.js 0.10 has enumerable non-index properties on buffers.
           (isBuff && (key == 'offset' || key == 'parent')) ||
           // PhantomJS 2 has enumerable non-index properties on typed arrays.
           (isType && (key == 'buffer' || key == 'byteLength' || key == 'byteOffset')) ||
           // Skip index properties.
           isIndex(key, length)
        ))) {
      result.push(key);
    }
  }
  return result;
}

module.exports = arrayLikeKeys;

```



### arrayMap

效果同Array.map()

#### 参数

+ array	需要处理的数组
+ iteratee   对数组每一项进行处理的方法

#### 返回

**Array**

返回被处理后的数组

#### 源码

```js
/**
 * A specialized version of `_.map` for arrays without support for iteratee
 * shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns the new mapped array.
 */
function arrayMap(array, iteratee) {
  var index = -1,
      length = array == null ? 0 : array.length,
      result = Array(length);

  while (++index < length) {
    result[index] = iteratee(array[index], index, array);
  }
  return result;
}

module.exports = arrayMap;
```

### arrayReduce

>   A specialized version of `_.reduce` for arrays without support for iteratee shorthands.
>
>   reduce函数的数组实现版本，不支持迭代器简写

#### 参数

+   array Array 待迭代的数组
+   iteratee Function 迭代器
+   accumulator any 可选 初始值
+   initAccum boolean 可选 是否使用数组第一个元素作为初始值

#### 返回

**any**

返回累加后的值

#### 解析

```js
/**
 * A specialized version of `_.reduce` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @param {*} [accumulator] The initial value.
 * @param {boolean} [initAccum] Specify using the first element of `array` as
 *  the initial value.
 * @returns {*} Returns the accumulated value.
 */
function arrayReduce(array, iteratee, accumulator, initAccum) {
  var index = -1,
      length = array == null ? 0 : array.length;
  //如果initAccum为true并且数组不为空，使用数组第一个元素作为初始值
  if (initAccum && length) {
    accumulator = array[++index];
  }
  while (++index < length) {
    //使用迭代函数处理累计值与当前值，迭代器的返回值作为下一次迭代的累计值
    accumulator = iteratee(accumulator, array[index], index, array);
  }
  //返回累计结果
  return accumulator;
}

module.exports = arrayReduce;

```

#### 源码

```js
/**
 * A specialized version of `_.reduce` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @param {*} [accumulator] The initial value.
 * @param {boolean} [initAccum] Specify using the first element of `array` as
 *  the initial value.
 * @returns {*} Returns the accumulated value.
 */
function arrayReduce(array, iteratee, accumulator, initAccum) {
  var index = -1,
      length = array == null ? 0 : array.length;

  if (initAccum && length) {
    accumulator = array[++index];
  }
  while (++index < length) {
    accumulator = iteratee(accumulator, array[index], index, array);
  }
  return accumulator;
}

module.exports = arrayReduce;

```

### arrayReduceRight

>   A specialized version of `_.reduceRight` for arrays without support for iteratee shorthands.
>
>   reduceRight函数的数组实现版本，不支持迭代器简写

#### 参数

+   array Array 待操作的数组
+   iteratee Function 迭代器
+   accumulator any 可选 初始值
+   initAccum boolean 可选 是否使用数组的最后一个元素作为初始值

#### 返回

**any**

返回累加后的 值

#### 源码

```js
/**
 * A specialized version of `_.reduceRight` for arrays without support for
 * iteratee shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @param {*} [accumulator] The initial value.
 * @param {boolean} [initAccum] Specify using the last element of `array` as
 *  the initial value.
 * @returns {*} Returns the accumulated value.
 */
function arrayReduceRight(array, iteratee, accumulator, initAccum) {
  var length = array == null ? 0 : array.length;
  //如果initAccum为true并且数组不为空，使用数组最后一个元素作为初始值
  if (initAccum && length) {
    accumulator = array[--length];
  }
  while (length--) {
    accumulator = iteratee(accumulator, array[length], length, array);
  }
  return accumulator;
}

module.exports = arrayReduceRight;

```



### arraySample

>   A specialized version of `_.sample` for arrays.
>
>   sample函数的数组实现版本

#### 参数

+   collection Array 待取值的数组

#### 返回

**any**

返回一个数组中的随机元素

#### 源码

```js
var baseRandom = require('./_baseRandom');

/**
 * A specialized version of `_.sample` for arrays.
 *
 * @private
 * @param {Array} array The array to sample.
 * @returns {*} Returns the random element.
 */
function arraySample(array) {
  var length = array.length;
  return length ? array[baseRandom(0, length - 1)] : undefined;
}

module.exports = arraySample;

```

### arraySampleSize

>   A specialized version of `_.sampleSize` for arrays.
>
>   sampleSize 函数的数组实现版本

#### 参数

+   array Array 待取值的数组
+   n number 待取值的个数

#### 返回

**Array**

返回n个随机元素组成的数组

#### 源码

```js
var baseClamp = require('./_baseClamp'),//限制数字大小在给定的返回内
    copyArray = require('./_copyArray'),
    shuffleSelf = require('./_shuffleSelf');

/**
 * A specialized version of `_.sampleSize` for arrays.
 *
 * @private
 * @param {Array} array The array to sample.
 * @param {number} n The number of elements to sample.
 * @returns {Array} Returns the random elements.
 */
function arraySampleSize(array, n) {
  return shuffleSelf(copyArray(array), baseClamp(n, 0, array.length));
}

module.exports = arraySampleSize;

```

### arrayShuffle

>   A specialized version of `_.shuffle` for arrays.
>
>   shuffle的数组实现版本

#### 参数

+   array Array 待洗牌的数组

#### 返回

**Array**

返回打乱后的新数组

#### 源码

源码中涉及到的函数

+   [shuffleSelf](#shuffleSelf)
+   [copyArray](#copyArray)

```js
var copyArray = require('./_copyArray'),
    shuffleSelf = require('./_shuffleSelf');

/**
 * A specialized version of `_.shuffle` for arrays.
 *
 * @private
 * @param {Array} array The array to shuffle.
 * @returns {Array} Returns the new shuffled array.
 */
function arrayShuffle(array) {
  return shuffleSelf(copyArray(array));
}

module.exports = arrayShuffle;

```

### arraySome

>   A specialized version of `_.some` for arrays without support for iteratee shorthands.
>
>   some函数的数组实现版本，不支持迭代器简写

#### 参数

+   array Array 待遍历的数组
+   predicate Function 断言函数

#### 返回

**boolean**

如果存在元素通过了predicate函数的检查，则返回true，否则返回false

#### 源码

```js
/**
 * A specialized version of `_.some` for arrays without support for iteratee
 * shorthands.
 *
 * @private
 * @param {Array} [array] The array to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {boolean} Returns `true` if any element passes the predicate check,
 *  else `false`.
 */
function arraySome(array, predicate) {
  var index = -1,
      length = array == null ? 0 : array.length;

  while (++index < length) {
    if (predicate(array[index], index, array)) {
      return true;
    }
  }
  return false;
}

module.exports = arraySome;

```



### assignValue

>    Assigns `value` to `key` of `object` if the existing value is not equivalent using SameValueZero for equality comparisons.
>
>   将值赋给对象对应的键，使用SameValueZero 做相等比较

#### 参数

+   object Object 待操作的对象
+   key string 待赋值的属性名
+   value any 值

#### 返回

**void**

#### 源码

源码中涉及的函数

+   [baseAssignValue](#baseAssignValue)
+   [eq](#eq值比较)

```js
var baseAssignValue = require('./_baseAssignValue'),
    eq = require('./eq');

/** Used for built-in method references. */
var objectProto = Object.prototype;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/**
 * Assigns `value` to `key` of `object` if the existing value is not equivalent
 * using [`SameValueZero`](http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero)
 * for equality comparisons.
 *
 * @private
 * @param {Object} object The object to modify.
 * @param {string} key The key of the property to assign.
 * @param {*} value The value to assign.
 */
function assignValue(object, key, value) {
  var objValue = object[key];
  if (!(hasOwnProperty.call(object, key) && eq(objValue, value)) ||
      (value === undefined && !(key in object))) {
    baseAssignValue(object, key, value);
  }
}

module.exports = assignValue;

```

### baseAggregator

>   Aggregates elements of `collection` on `accumulator` with keys transformed by `iteratee` and values set by `setter`.
>
>   聚合累加器上的集合元素，其键由iterateee转换，值由setter设置。

#### 参数

+   array Array|Object 待统计集合
+   setter Function 累加器的赋值函数
+   iteratee 用于为累加器转换key
+   accumulator Object 初始化的累加器对象

#### 返回

**Object** 

这里返回的是一个累加器对象

#### 解析

```js
var baseEach = require('./_baseEach');

/**
 * Aggregates elements of `collection` on `accumulator` with keys transformed
 * by `iteratee` and values set by `setter`.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} setter The function to set `accumulator` values.
 * @param {Function} iteratee The iteratee to transform keys.
 * @param {Object} accumulator The initial aggregated object.
 * @returns {Function} Returns `accumulator`.
 */
function baseAggregator(collection, setter, iteratee, accumulator) {
  //遍历集合
  baseEach(collection, function(value, key, collection) {
    //setter前3个参数指result，value，key，作用为向result对象中的key赋值
    //iteratee(value)生成累加器对象的键key
    setter(accumulator, value, iteratee(value), collection);
  });
  return accumulator;
}

module.exports = baseAggregator;

```



#### 源码

源码中涉及的函数

+   [baseEach](#baseEach)

```js
var baseEach = require('./_baseEach');

/**
 * Aggregates elements of `collection` on `accumulator` with keys transformed
 * by `iteratee` and values set by `setter`.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} setter The function to set `accumulator` values.
 * @param {Function} iteratee The iteratee to transform keys.
 * @param {Object} accumulator The initial aggregated object.
 * @returns {Function} Returns `accumulator`.
 */
function baseAggregator(collection, setter, iteratee, accumulator) {
  baseEach(collection, function(value, key, collection) {
    setter(accumulator, value, iteratee(value), collection);
  });
  return accumulator;
}

module.exports = baseAggregator;

```



### baseAssignValue

>   The base implementation of `assignValue` and `assignMergeValue` without value checks.
>
>   assignValue和assignMergeValue的基础实现函数，用于给对象分派指定值的键

#### 参数

+   object Object 待操作的对象
+   key string 键名
+   value any 分配给键的值

#### 返回

**void**

#### 源码

源码中涉及的函数

+   [defineProperty](#defineProperty)

```js
var defineProperty = require('./_defineProperty');

/**
 * The base implementation of `assignValue` and `assignMergeValue` without
 * value checks.
 *
 * @private
 * @param {Object} object The object to modify.
 * @param {string} key The key of the property to assign.
 * @param {*} value The value to assign.
 */
function baseAssignValue(object, key, value) {
  if (key == '__proto__' && defineProperty) {
    defineProperty(object, key, {
      'configurable': true,
      'enumerable': true,
      'value': value,
      'writable': true
    });
  } else {
    object[key] = value;
  }
}

module.exports = baseAssignValue;

```



### baseAt

>   The base implementation of `_.at` without support for individual paths
>
>   at函数的基本实现，不支持单独的路径，即paths是一个数组

#### 参数

+   object Object 待操作对象
+   paths Array<string> 可选 待提取的坐标

#### 返回

**Array**

返回对应索引（键值）的元素所组成的数组

#### 解析

```js
var get = require('./get');

/**
 * The base implementation of `_.at` without support for individual paths.
 *
 * @private
 * @param {Object} object The object to iterate over.
 * @param {string[]} paths The property paths to pick.
 * @returns {Array} Returns the picked elements.
 */
function baseAt(object, paths) {
  var index = -1,
      length = paths.length,
      result = Array(length),
      //如果待操作对象为空则跳过
      skip = object == null;
  //遍历路径，如果skip===true，则存入undefined，否则从object中获取对应的值
  while (++index < length) {
    result[index] = skip ? undefined : get(object, paths[index]);
  }
  return result;
}

module.exports = baseAt;

```



#### 源码

```js
var get = require('./get');

/**
 * The base implementation of `_.at` without support for individual paths.
 *
 * @private
 * @param {Object} object The object to iterate over.
 * @param {string[]} paths The property paths to pick.
 * @returns {Array} Returns the picked elements.
 */
function baseAt(object, paths) {
  var index = -1,
      length = paths.length,
      result = Array(length),
      skip = object == null;

  while (++index < length) {
    result[index] = skip ? undefined : get(object, paths[index]);
  }
  return result;
}

module.exports = baseAt;

```

### baseDelay

>   The base implementation of `_.delay` and `_.defer` which accepts `args` to provide to `func`.
>
>   delay和defer函数的基础实现，接受传给func的附加参数

#### 参数

+   func Function 待延迟调用的函数
+   wait number 延迟调用时间 毫秒
+   args Array 附加参数

#### 返回

**number|Object**

返回定时器id

#### 源码

```js
/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/**
 * The base implementation of `_.delay` and `_.defer` which accepts `args`
 * to provide to `func`.
 *
 * @private
 * @param {Function} func The function to delay.
 * @param {number} wait The number of milliseconds to delay invocation.
 * @param {Array} args The arguments to provide to `func`.
 * @returns {number|Object} Returns the timer id or timeout object.
 */
function baseDelay(func, wait, args) {
  if (typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  return setTimeout(function() { func.apply(undefined, args); }, wait);
}

module.exports = baseDelay;

```



### baseDifference

> The base implementation of methods like `_.difference` without support for excluding multiple arrays or iteratee shorthands.
>
> _.difference方法的基本实现，不支持排除多个数组或迭代体缩写。

#### 参数

+ array 要检查的数组
+ values 要排除的值。
+ iteratee 可选 每个元素调用的迭代对象。
+ comparator 可选  比较器

#### 返回

#### 源码

```js
var SetCache = require('./_SetCache'),
    arrayIncludes = require('./_arrayIncludes'),
    arrayIncludesWith = require('./_arrayIncludesWith'),
    arrayMap = require('./_arrayMap'),
    baseUnary = require('./_baseUnary'),
    cacheHas = require('./_cacheHas');

/** Used as the size to enable large array optimizations. */
var LARGE_ARRAY_SIZE = 200;

/**
 * The base implementation of methods like `_.difference` without support
 * for excluding multiple arrays or iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Array} values The values to exclude.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of filtered values.
 */
function baseDifference(array, values, iteratee, comparator) {
  var index = -1,
      includes = arrayIncludes,
      isCommon = true,
      length = array.length,
      result = [],
      valuesLength = values.length;

  if (!length) {
    return result;
  }
  if (iteratee) {
    values = arrayMap(values, baseUnary(iteratee));
  }
  if (comparator) {
    includes = arrayIncludesWith;
    isCommon = false;
  }
  else if (values.length >= LARGE_ARRAY_SIZE) {
    includes = cacheHas;
    isCommon = false;
    values = new SetCache(values);
  }
  outer:
  while (++index < length) {
    var value = array[index],
        computed = iteratee == null ? value : iteratee(value);

    value = (comparator || value !== 0) ? value : 0;
    if (isCommon && computed === computed) {
      var valuesIndex = valuesLength;
      while (valuesIndex--) {
        if (values[valuesIndex] === computed) {
          continue outer;
        }
      }
      result.push(value);
    }
    else if (!includes(values, computed, comparator)) {
      result.push(value);
    }
  }
  return result;
}

module.exports = baseDifference;

```

### baseEach

>   The base implementation of `_.forEach` without support for iteratee shorthands.
>
>   forEach函数的基本实现，不支持迭代器简写

#### 参数

+   collection Array|Object 待遍历集合
+   iteratee Function 迭代器

#### 返回

**Array|Object**

返回集合

#### 源码

```js
var baseForOwn = require('./_baseForOwn'),
    createBaseEach = require('./_createBaseEach');

/**
 * The base implementation of `_.forEach` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array|Object} Returns `collection`.
 */
var baseEach = createBaseEach(baseForOwn);

module.exports = baseEach;

```

### baseEachRight

>   The base implementation of `_.forEachRight` without support for iteratee shorthands. 
>
>   forEachRight 函数的基础实现，不支持迭代器简写

#### 参数

+   collection Array|Object 待遍历的集合
+   iteratee Function 迭代器

#### 返回

**Array|Object**

返回集合

#### 源码

```js
var baseForOwnRight = require('./_baseForOwnRight'),
    createBaseEach = require('./_createBaseEach');

/**
 * The base implementation of `_.forEachRight` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array|Object} Returns `collection`.
 */
var baseEachRight = createBaseEach(baseForOwnRight, true);

module.exports = baseEachRight;

```

### baseEvery

>   The base implementation of `_.every` without support for iteratee shorthands.
>
>   every函数的基础实现，不支持迭代器简写

#### 参数

+   collection Array|Object 待检查的集合
+   predicate Function 断言函数，每次遍历调用

#### 返回

**boolean**

如果所有元素经 predicate（断言函数） 检查后都都返回真值，那么就返回`true`，否则返回 `false` 。

#### 源码

源码中涉及的函数

+   [baseEach](#baseEach)

```js
var baseEach = require('./_baseEach');

/**
 * The base implementation of `_.every` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {boolean} Returns `true` if all elements pass the predicate check,
 *  else `false`
 */
function baseEvery(collection, predicate) {
  var result = true;
  baseEach(collection, function(value, index, collection) {
    result = !!predicate(value, index, collection);
    return result;
  });
  return result;
}

module.exports = baseEvery;

```



### baseFill

>   The base implementation of `_.fill` without an iteratee call guard.
>
>   ' _. fill'的基本实现。非迭代调用

#### 参数

+   array Array 待填充数组
+   value any 填充的值
+   start number 可选 填充的开始位置 默认0
+   end number 可选 填充的结束位置 默认array.length 填充不含end位置

#### 返回

**Array**

填充后的数组

#### 示例

示例1

```js
let array = [1, 2, 3, 4, 5]
let res = baseFill(array, 'a', -3)
console.log(res)
//[ 1, 2, 'a', 'a', 'a' ]
```

示例2

```js
let array = [1, 2, 3, 4, 5]
let res = baseFill(array, 'a', -4, -2)
console.log(res)
//[ 1, 'a', 'a', 4, 5 ]
```

#### 解析

```js
var toInteger = require('./toInteger'),
    toLength = require('./toLength');

/**
 * The base implementation of `_.fill` without an iteratee call guard.
 *
 * @private
 * @param {Array} array The array to fill.
 * @param {*} value The value to fill `array` with.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns `array`.
 */
function baseFill(array, value, start, end) {
  //数组长度
  var length = array.length;
  //转为整值
  start = toInteger(start);
  if (start < 0) {
   	//start<0时，如果start的绝对值（-start）大于数组长度，则start=0，从开始位置填充，否则start=length+start（数组
    //长度减去start的绝对值），即填充的开始位置为从右开始数，第|start|个 如示例1，-3即从右数第三个位置
    start = -start > length ? 0 : (length + start);
  }
  //结束位置大于数组长度或者未传入，则end=length,否则转为整值
  end = (end === undefined || end > length) ? length : toInteger(end);
  if (end < 0) {
    //如果结束位置小于0，则end=end+length，即end=length-|end|，同样从末尾开始数
    end += length;
  }
  //如果开始位置在结束位置之后，则end=0,即不填充
  end = start > end ? 0 : toLength(end);
  while (start < end) {
    array[start++] = value;
  }
  return array;
}

module.exports = baseFill;

```



#### 源码

源码中涉及的方法

+   [toInteger](#toInteger转为整数)

```js
var toInteger = require('./toInteger'),
    toLength = require('./toLength');

/**
 * The base implementation of `_.fill` without an iteratee call guard.
 *
 * @private
 * @param {Array} array The array to fill.
 * @param {*} value The value to fill `array` with.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns `array`.
 */
function baseFill(array, value, start, end) {
  var length = array.length;

  start = toInteger(start);
  if (start < 0) {
    start = -start > length ? 0 : (length + start);
  }
  end = (end === undefined || end > length) ? length : toInteger(end);
  if (end < 0) {
    end += length;
  }
  end = start > end ? 0 : toLength(end);
  while (start < end) {
    array[start++] = value;
  }
  return array;
}

module.exports = baseFill;

```

### baseFilter

>   The base implementation of `_.filter` without support for iteratee shorthands.
>
>   filter函数的基础实现，不支持迭代器简写

#### 参数

+   collection Array|Object 待过滤的集合
+   predicate Function 断言

#### 返回

**Array**

返回一个新的过滤后的数组

#### 源码

源码中涉及的函数

+   [baseEach](#baseEach)

```js
var baseEach = require('./_baseEach');

/**
 * The base implementation of `_.filter` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {Array} Returns the new filtered array.
 */
function baseFilter(collection, predicate) {
  var result = [];
  baseEach(collection, function(value, index, collection) {
    if (predicate(value, index, collection)) {
      result.push(value);
    }
  });
  return result;
}

module.exports = baseFilter;

```



### baseFindIndex

>   The base implementation of `_.findIndex` and `_.findLastIndex` without support for iteratee shorthands.
>
>   ' _.findIndex'的和 'findLastIndex '基本实现。不支持迭代体缩写。

#### 参数

+   array Array
+   predicate Function 迭代器 判断当前值是否是被搜索的值
+   fromIndex number 起始搜索位置
+   fromRight boolean 可选 是否从fromIndex开始向左搜索

#### 返回

**number**

如果存在被搜索的值，返回它的索引，否则返回-1

#### 解析

```js
/**
 * The base implementation of `_.findIndex` and `_.findLastIndex` without
 * support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} predicate The function invoked per iteration.
 * @param {number} fromIndex The index to search from.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {number} Returns the index of the matched value, else `-1`.
 */
function baseFindIndex(array, predicate, fromIndex, fromRight) {
  //数组长度
  //如果是从右向左开始搜索，则index=fromIndex+1,否则index=fromIndex-1
  //搜索位置包含fromIndex（因此要+1或者-1）
  var length = array.length,
      index = fromIndex + (fromRight ? 1 : -1);
  //如果是从右开始搜索，则index递减，否则index递增
  //这里有一个有意思的细节，使用的是index--而非--index，使用index--即先执行while判断，再执行index-=1，保证了index=0
  //时也可以进入while循环（因为index=1时，while(index--)先判断while(1)，再执行index-=1，传入index=0到循环体中。
  while ((fromRight ? index-- : ++index < length)) {
    //如果通过判断，返回索引
    if (predicate(array[index], index, array)) {
      return index;
    }
  }
  //未搜索到，返回-1
  return -1;
}

module.exports = baseFindIndex;

```



#### 源码

```js
/**
 * The base implementation of `_.findIndex` and `_.findLastIndex` without
 * support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} predicate The function invoked per iteration.
 * @param {number} fromIndex The index to search from.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {number} Returns the index of the matched value, else `-1`.
 */
function baseFindIndex(array, predicate, fromIndex, fromRight) {
  var length = array.length,
      index = fromIndex + (fromRight ? 1 : -1);

  while ((fromRight ? index-- : ++index < length)) {
    if (predicate(array[index], index, array)) {
      return index;
    }
  }
  return -1;
}

module.exports = baseFindIndex;

```



### baseFlatten

> The base implementation of `_.flatten` with support for restricting flattening.
>
> ' _. flatten'的基本实现。支持限制压平。即将数组扁平化处理

#### 参数

+ array 需要扁平化的数组
+ depth 扁平化层次（递归深度）
+ predicate 可选 每次迭代调用的函数。自定义的检查函数
+ isStrict 仅限于通过' predicate '检查的值。
+ result 可选 每次递归的结果数组

#### 返回

**Array**

 返回扁平化后的数组

#### 示例

示例1

```js
let a = [1, 2, [3, 4], [5, [6, 7]], [8, [9, 10], [11, [12, 13]]]]
let res = baseFlatten(a, 3)
console.log('res', res)
//res [1, 2, 3,  4,  5,  6,	7, 8, 9, 10, 11, 12, 13]
```

示例2

```js
let a = [1, 2, [3, 4], [5, [6, 7]], [8, [9, 10], [11, [12, 13]]]]
let res = baseFlatten(a, 2)
console.log('res', res)
//res [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, [ 12, 13 ] ]
```

示例3

```js
let a = ['a', 'b', 1,]
let res = baseFlatten(a, 1, (s) => s !== 1, true)
console.log('res', res)
//res [ 'a', 'b' ]
```

```js
let a = ['a', 'b', 1, ['c', 'd', 1], ['e', ['f', 1, 'g']]]
let res = baseFlatten(a, 2, (s) => s !== 1, true)
console.log('res', res)
//res  ['a', 'b', 'c', 'd', 'e', 'f', 1, 'g']
```

```js
let a = ['a', 'b', 1, ['c', 'd', 1], ['e', ['f', 1, 'g']]]
let res = baseFlatten(a, 2, (s) => s !== 1, false)
console.log('res', res)
//res['a', 'b', 1, 'c', 'd', 1, 'e', 'f', 1, 'g']
```

```js
let a = ['a', 'b', 1, ['c', 'd', 1], ['e', ['f', 1, 'g']]]
let res = baseFlatten(a, 3, (s) => s !== 1, true)
console.log('res', res)
//res['a', 'b', 'c', 'd', 'e', 'f', 'g']
```

baseFlatten进行数组扁平化的同时还可以进行数组的过滤，但是如果想要过滤掉所有的值，则必须过量递归，即传入的扁平化层次需要大于全扁平化所需要的值，即当数组为[[[[1]]]]四层时，全部破坏需要depth=3，则全扁平化过滤需要depth>3

#### 解析

```js
var arrayPush = require('./_arrayPush'),
    isFlattenable = require('./_isFlattenable');

/**
 * The base implementation of `_.flatten` with support for restricting flattening.
 *
 * @private
 * @param {Array} array The array to flatten.
 * @param {number} depth The maximum recursion depth.
 * @param {boolean} [predicate=isFlattenable] The function invoked per iteration.
 * @param {boolean} [isStrict] Restrict to values that pass `predicate` checks.
 * @param {Array} [result=[]] The initial result value.
 * @returns {Array} Returns the new flattened array.
 */
function baseFlatten(array, depth, predicate, isStrict, result) {
  var index = -1,
      length = array.length;
  //初始化检查函数，如果没有则默认为isFlattenable方法
  predicate || (predicate = isFlattenable);
  //第一次迭代，result不存在，则初始化为空数组
  result || (result = []);

  while (++index < length) {
    var value = array[index];
    if (depth > 0 && predicate(value)) {
      if (depth > 1) {
        // Recursively flatten arrays (susceptible to call stack limits).
        //递归
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        //depth===1
        arrayPush(result, value);
      }
    } else if (!isStrict) {
      //如果不需要检查则直接存值
      result[result.length] = value;
    }
  }
  return result;
}

module.exports = baseFlatten;

```



#### 源码

```js
var arrayPush = require('./_arrayPush'),
    isFlattenable = require('./_isFlattenable');

/**
 * The base implementation of `_.flatten` with support for restricting flattening.
 *
 * @private
 * @param {Array} array The array to flatten.
 * @param {number} depth The maximum recursion depth.
 * @param {boolean} [predicate=isFlattenable] The function invoked per iteration.
 * @param {boolean} [isStrict] Restrict to values that pass `predicate` checks.
 * @param {Array} [result=[]] The initial result value.
 * @returns {Array} Returns the new flattened array.
 */
function baseFlatten(array, depth, predicate, isStrict, result) {
  var index = -1,
      length = array.length;

  predicate || (predicate = isFlattenable);
  result || (result = []);

  while (++index < length) {
    var value = array[index];
    if (depth > 0 && predicate(value)) {
      if (depth > 1) {
        // Recursively flatten arrays (susceptible to call stack limits).
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        arrayPush(result, value);
      }
    } else if (!isStrict) {
      result[result.length] = value;
    }
  }
  return result;
}

module.exports = baseFlatten;

```



### baseGetAllKeys

> The base implementation of `getAllKeys` and `getAllKeysIn` which uses `keysFunc` and `symbolsFunc` to get the enumerable property names and symbols of `object`.
>
> ' getAllKeys '和' getAllKeysIn '的基本实现使用' keysFunc '和' symbolsFunc '来获得' object '的可枚举属性名和符号。

#### 参数

+ object
+ keysFunc    来获得' object '的可枚举属性名的方法
+ symbolsFunc    来获得' object '的符号的方法

#### 源码

```js
/**
 * The base implementation of `getAllKeys` and `getAllKeysIn` which uses
 * `keysFunc` and `symbolsFunc` to get the enumerable property names and
 * symbols of `object`.
 *
 * @private
 * @param {Object} object The object to query.
 * @param {Function} keysFunc The function to get the keys of `object`.
 * @param {Function} symbolsFunc The function to get the symbols of `object`.
 * @returns {Array} Returns the array of property names and symbols.
 */
function baseGetAllKeys(object, keysFunc, symbolsFunc) {
    var result = keysFunc(object);
    return isArray(object) ? result : arrayPush(result, symbolsFunc(object));
}
```

### baseGetTag

>   The base implementation of `getTag` without fallbacks for buggy environments.
>
>   ' getTag '的基本实现，在 buggy environments中没有回退。

类型字符串获取

#### 参数

+   value any 待检查的值

#### 返回

**string**

返回类型字符串

#### 源码

```js
var Symbol = require('./_Symbol'),
    getRawTag = require('./_getRawTag'),
    objectToString = require('./_objectToString');

/** `Object#toString` result references. */
var nullTag = '[object Null]',
    undefinedTag = '[object Undefined]';

/** Built-in value references. */
var symToStringTag = Symbol ? Symbol.toStringTag : undefined;

/**
 * The base implementation of `getTag` without fallbacks for buggy environments.
 *
 * @private
 * @param {*} value The value to query.
 * @returns {string} Returns the `toStringTag`.
 */
function baseGetTag(value) {
  if (value == null) {
    return value === undefined ? undefinedTag : nullTag;
  }
  return (symToStringTag && symToStringTag in Object(value))
    ? getRawTag(value)
    : objectToString(value);
}

module.exports = baseGetTag;

```



### baseIndexOf

>   The base implementation of `_.indexOf` without `fromIndex` bounds checks.
>
>   indexOf 的基本实现，没有fromIndex的范围检查（传入的fromIndex都大于0）

#### 参数

+   array Array 待查询数组
+   value any 待查询值
+   fromIndex number 可选 查询的起始位置

#### 返回

**number**

返回value在array中的索引，如果不存在则返回-1

#### 源码

源码中涉及的方法

+   [baseFindIndex](#baseFindIndex)

```js
var baseFindIndex = require('./_baseFindIndex'),
    baseIsNaN = require('./_baseIsNaN'),
    strictIndexOf = require('./_strictIndexOf');

/**
 * The base implementation of `_.indexOf` without `fromIndex` bounds checks.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @param {number} fromIndex The index to search from.
 * @returns {number} Returns the index of the matched value, else `-1`.
 */
function baseIndexOf(array, value, fromIndex) {
  //如果value是NaN，则调用baseFindIndex
  return value === value
    ? strictIndexOf(array, value, fromIndex)
    : baseFindIndex(array, baseIsNaN, fromIndex);
}

module.exports = baseIndexOf;

```

### baseIntersection

>   he base implementation of methods like `_.intersection`, without support for iteratee shorthands, that accepts an array of arrays to inspect.
>
>   交叉方法的基本实现，不支持iteratee缩写，接受嵌套数组检查。

#### 参数

+   arrays 待处理的嵌套数组
+   iteratee Function 可选 迭代器方法
+   comparator Function 可选 比较器方法

#### 返回

**Array**

返回由数组交集组成的新数组

#### 解析

```js
var SetCache = require('./_SetCache'),//Set缓存数组
    arrayIncludes = require('./_arrayIncludes'),//同Array.includes方法
    arrayIncludesWith = require('./_arrayIncludesWith'),//类似Array.includes方法，但接收一个函数作为比较方法
    arrayMap = require('./_arrayMap'),
    baseUnary = require('./_baseUnary'),//创建一个只有一个参数的方法，忽略其他的参数
    cacheHas = require('./_cacheHas');//判断缓存中是否存在某个元素

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMin = Math.min;

/**
 * The base implementation of methods like `_.intersection`, without support
 * for iteratee shorthands, that accepts an array of arrays to inspect.
 *
 * @private
 * @param {Array} arrays The arrays to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of shared values.
 */
function baseIntersection(arrays, iteratee, comparator) {
  //初始化比较器，如果comparator存在，则使用arrayIncludesWith
  var includes = comparator ? arrayIncludesWith : arrayIncludes,
      //第一个子数组的长度
      length = arrays[0].length,
      //嵌套数组的长度
      othLength = arrays.length,
      othIndex = othLength,
      caches = Array(othLength),
      //结果数组的最大长度
      maxLength = Infinity,
      result = [];
  //遍历嵌套数组 先比较，再递减
  while (othIndex--) {
    //array为当前循环的元素
    var array = arrays[othIndex];
    //如果othIndex>0并且有迭代器，对array中的每一项调用iteratee进行处理，因为此时iteratee被baseUnary包装，因此只能
    //接收一个参数array[index]，处理后的值覆盖掉原数组。
    if (othIndex && iteratee) {
      array = arrayMap(array, baseUnary(iteratee));
    }
    //结果数组的最大长度是所有子数组中最短数组的长度，因为是取交集
    maxLength = nativeMin(array.length, maxLength);
    //缓存优化，当othIndex===0时，chches[0]=undefined|new SetCache()
    caches[othIndex] = !comparator && (iteratee || (length >= 120 && array.length >= 120))
      ? new SetCache(othIndex && array)
      : undefined;
  }
  array = arrays[0];

  var index = -1,
      seen = caches[0];//第一个缓存数组，undefined或者new SetCache(0) 即seen初始为空

  outer:
  //遍历第一个子数组，因为是结果是取交集，因此result中有的，第一个子数组也有，通过遍历第一个子数组元素，检查其它子数组是否
  //也有这个元素来获取最后结果
  while (++index < length && result.length < maxLength) {
    //如果迭代器存在，调用迭代器对数组元素进行处理，computed为将进行比较的值，其它子数组已经在上面的循环中调用迭代器
    //对子元素进行了处理
    var value = array[index],
        computed = iteratee ? iteratee(value) : value;
	//如果比较器存在或者value!==0，value保持不变，否则value=0
    value = (comparator || value !== 0) ? value : 0;
    //如果seen中有computed，或者result中有computed，说明当前value已经通过判断,不需要再次处理，进入下一次循环
    if (!(seen//如果seen有值，判断是否含有computed，否则判断结果result中是否含有computed，如果为否，进入if
          ? cacheHas(seen, computed)
          : includes(result, computed, comparator)
        )) {
      othIndex = othLength;//其他数组的索引，值为嵌套数组的长度
      //遍历嵌套子数组|缓存 因为是先递减，再比较，因此排除第一个子数组
      while (--othIndex) {
        //从缓存中取出其它某个数组
        var cache = caches[othIndex];
        if (!(cache//cache存在则判断cache中是否存在computed，否则判断子数组中是否存在computed，如果都为否，跳过本次
              //循环，检查第一个子数组的下一个元素
              ? cacheHas(cache, computed)
              : includes(arrays[othIndex], computed, comparator))
            ) {
          continue outer;
        }
      }
      //如果代码执行到这里，通过了上面while循环的检查，说明当前value在arrays的所有子数组中都存在。
      //如果seen有值，向seen中加入computed，如果seen中有computed，说明当前value已经通过判断
      if (seen) {
        seen.push(computed);
      }
      result.push(value);
    }
  }
  return result;
}

module.exports = baseIntersection;

```



#### 源码

```js
var SetCache = require('./_SetCache'),
    arrayIncludes = require('./_arrayIncludes'),
    arrayIncludesWith = require('./_arrayIncludesWith'),
    arrayMap = require('./_arrayMap'),
    baseUnary = require('./_baseUnary'),
    cacheHas = require('./_cacheHas');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMin = Math.min;

/**
 * The base implementation of methods like `_.intersection`, without support
 * for iteratee shorthands, that accepts an array of arrays to inspect.
 *
 * @private
 * @param {Array} arrays The arrays to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of shared values.
 */
function baseIntersection(arrays, iteratee, comparator) {
  var includes = comparator ? arrayIncludesWith : arrayIncludes,
      length = arrays[0].length,
      othLength = arrays.length,
      othIndex = othLength,
      caches = Array(othLength),
      maxLength = Infinity,
      result = [];

  while (othIndex--) {
    var array = arrays[othIndex];
    if (othIndex && iteratee) {
      array = arrayMap(array, baseUnary(iteratee));
    }
    maxLength = nativeMin(array.length, maxLength);
    caches[othIndex] = !comparator && (iteratee || (length >= 120 && array.length >= 120))
      ? new SetCache(othIndex && array)
      : undefined;
  }
  array = arrays[0];

  var index = -1,
      seen = caches[0];

  outer:
  while (++index < length && result.length < maxLength) {
    var value = array[index],
        computed = iteratee ? iteratee(value) : value;

    value = (comparator || value !== 0) ? value : 0;
    if (!(seen
          ? cacheHas(seen, computed)
          : includes(result, computed, comparator)
        )) {
      othIndex = othLength;
      while (--othIndex) {
        var cache = caches[othIndex];
        if (!(cache
              ? cacheHas(cache, computed)
              : includes(arrays[othIndex], computed, comparator))
            ) {
          continue outer;
        }
      }
      if (seen) {
        seen.push(computed);
      }
      result.push(value);
    }
  }
  return result;
}

module.exports = baseIntersection;

```

### baseInvoke

>   The base implementation of `_.invoke` without support for individual method arguments.
>
>   invoke的基本实现。不支持单个方法参数的调用。

#### 参数

+   object Object 待操作的对象
+   path Array|string 调用函数的路径
+   args 调用函数的参数

#### 返回

**any**

返回处理后的结果

#### 解析

```js
var apply = require('./_apply'),
    castPath = require('./_castPath'),
    last = require('./last'),
    parent = require('./_parent'),
    toKey = require('./_toKey');

/**
 * The base implementation of `_.invoke` without support for individual
 * method arguments.
 *
 * @private
 * @param {Object} object The object to query.
 * @param {Array|string} path The path of the method to invoke.
 * @param {Array} args The arguments to invoke the method with.
 * @returns {*} Returns the result of the invoked method.
 */
function baseInvoke(object, path, args) {
  //使用castPath将path切为单个键名，并依旧存到path中，此时path为数组
  path = castPath(path, object);
  object = parent(object, path);
  //获取调用函数
  var func = object == null ? object : object[toKey(last(path))];
  return func == null ? undefined : apply(func, object, args);
}

module.exports = baseInvoke;

```



#### 源码

```js
var apply = require('./_apply'),
    castPath = require('./_castPath'),
    last = require('./last'),
    parent = require('./_parent'),
    toKey = require('./_toKey');

/**
 * The base implementation of `_.invoke` without support for individual
 * method arguments.
 *
 * @private
 * @param {Object} object The object to query.
 * @param {Array|string} path The path of the method to invoke.
 * @param {Array} args The arguments to invoke the method with.
 * @returns {*} Returns the result of the invoked method.
 */
function baseInvoke(object, path, args) {
  path = castPath(path, object);
  object = parent(object, path);
  var func = object == null ? object : object[toKey(last(path))];
  return func == null ? undefined : apply(func, object, args);
}

module.exports = baseInvoke;

```



### baseIsNaN

>   The base implementation of `_.isNaN` without support for number objects.
>
>   ' _. '的基本实现。isNaN '不支持数字对象。(Number)

#### 参数

+   value any 待检查的值

#### 返回

**boolean**

如果是NaN，返回true，否则返回false

#### 源码

```js
/**
 * The base implementation of `_.isNaN` without support for number objects.
 *
 * @private
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is `NaN`, else `false`.
 */
function baseIsNaN(value) {
  return value !== value;
}

module.exports = baseIsNaN;

```



### baseIteratee

**<font color='red'>关键方法</font>**

创建一个函数，通过创建函数的参数调用 value 函数。 如果value是一个属性名，传入包含这个属性名的对象，回调返回对应属性名的值。 如果value是一个对象，传入的元素有相同的对象属性，回调返回 `true` 。 其他情况返回 `false` 。

> The base implementation of `_.iteratee`.
>
> ' _.iteratee '的基本实现。

可以将简写的迭代器转为一个完整的迭代器函数

#### 参数

+ value 可选 要转换为迭代对象（callback ）的值。

#### 返回

**Function**

 返回回调函数

#### 源码

```js
var baseMatches = require('./_baseMatches'),
    baseMatchesProperty = require('./_baseMatchesProperty'),
    identity = require('./identity'),
    isArray = require('./isArray'),
    property = require('./property');

/**
 * The base implementation of `_.iteratee`.
 *
 * @private
 * @param {*} [value=_.identity] The value to convert to an iteratee.
 * @returns {Function} Returns the iteratee.
 */
function baseIteratee(value) {
  // Don't store the `typeof` result in a variable to avoid a JIT bug in Safari 9.
  // See https://bugs.webkit.org/show_bug.cgi?id=156034 for more details.
  if (typeof value == 'function') {
    return value;
  }
  if (value == null) {
    return identity;
  }
  if (typeof value == 'object') {
    return isArray(value)
      ? baseMatchesProperty(value[0], value[1])
      : baseMatches(value);
  }
  return property(value);
}

module.exports = baseIteratee;

```

### baseKeys

>   The base implementation of `_.keys` which doesn't treat sparse arrays as dense.
>
>   keys的基本实现。不把稀疏数组看作密集数组的键。

#### 参数.

+   object Object 待操作的对象

#### 返回

**Array**

返回属性名构成的数组

#### 源码

```js
var isPrototype = require('./_isPrototype'),
    nativeKeys = require('./_nativeKeys');

/** Used for built-in method references. */
var objectProto = Object.prototype;

/** Used to check objects for own properties. */
var hasOwnProperty = objectProto.hasOwnProperty;

/**
 * The base implementation of `_.keys` which doesn't treat sparse arrays as dense.
 *
 * @private
 * @param {Object} object The object to query.
 * @returns {Array} Returns the array of property names.
 */
function baseKeys(object) {
  if (!isPrototype(object)) {
    return nativeKeys(object);
  }
  var result = [];
  for (var key in Object(object)) {
    if (hasOwnProperty.call(object, key) && key != 'constructor') {
      result.push(key);
    }
  }
  return result;
}

module.exports = baseKeys;

```



### baseMap

>   The base implementation of `_.map` without support for iteratee shorthands.
>
>   map函数的基础实现函数，不支持迭代器简写。

#### 参数

+   collection Array|Object 待遍历的集合
+   iteratee Function 迭代器 每次迭代调用

#### 返回

**Array**

返回遍历处理后的新数组

#### 源码

源码中涉及的函数

+   [baseEach](#baseEach)
+   [isArrayLike](#isArrayLike检查类数组)

```js
var baseEach = require('./_baseEach'),
    isArrayLike = require('./isArrayLike');

/**
 * The base implementation of `_.map` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns the new mapped array.
 */
function baseMap(collection, iteratee) {
  var index = -1,
      result = isArrayLike(collection) ? Array(collection.length) : [];
  //使用baseEach遍历，将iteratee迭代器处理后的值存到result中
  baseEach(collection, function(value, key, collection) {
    result[++index] = iteratee(value, key, collection);
  });
  return result;
}

module.exports = baseMap;

```



### baseNth

>   The base implementation of `_.nth` which doesn't coerce arguments.
>
>   _.nth的基本实现，不强制参数

#### 参数

+   array Array 待查询数组
+   n number 索引

#### 返回

**any**

返回指定索引的元素

#### 源码

源码中涉及的方法

+   [isIndex](#isIndex)

```js
var isIndex = require('./_isIndex');

/**
 * The base implementation of `_.nth` which doesn't coerce arguments.
 *
 * @private
 * @param {Array} array The array to query.
 * @param {number} n The index of the element to return.
 * @returns {*} Returns the nth element of `array`.
 */
function baseNth(array, n) {
  var length = array.length;
  if (!length) {
    return;
  }
  n += n < 0 ? length : 0;
  return isIndex(n, length) ? array[n] : undefined;
}

module.exports = baseNth;

```

### baseOrderBy

>   The base implementation of `_.orderBy` without param guards.
>
>   orderBy的基本实现。没有参数守卫的。

#### 参数

+   collection Array|Object 待处理的集合
+   iteratees Function[]|Object[]|string[] 迭代器集合
+   orders string[] 迭代函数的排序顺序

#### 返回

**Array**

返回排序后的新数组

#### 解析

```js
var arrayMap = require('./_arrayMap'),
    baseGet = require('./_baseGet'),
    baseIteratee = require('./_baseIteratee'),
    baseMap = require('./_baseMap'),
    baseSortBy = require('./_baseSortBy'),
    baseUnary = require('./_baseUnary'),
    compareMultiple = require('./_compareMultiple'),
    identity = require('./identity'),
    isArray = require('./isArray');

/**
 * The base implementation of `_.orderBy` without param guards.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function[]|Object[]|string[]} iteratees The iteratees to sort by.
 * @param {string[]} orders The sort orders of `iteratees`.
 * @returns {Array} Returns the new sorted array.
 */
function baseOrderBy(collection, iteratees, orders) {
    if (iteratees.length) {
        iteratees = arrayMap(iteratees, function (iteratee) {
            if (isArray(iteratee)) {
                return function (value) {
                    return baseGet(value, iteratee.length === 1 ? iteratee[0] : iteratee);
                }
            }
            return iteratee;
        });
    } else {
        //如果iteratees不存在，直接使用原值
        iteratees = [identity];
    }
    console.log('iterattees1', iteratees)
    //iterattees1 [ 'user', 'age' ]
    var index = -1;
    //封装迭代器
    iteratees = arrayMap(iteratees, baseUnary(baseIteratee));
    console.log('iterattees2', iteratees)
    //iterattees2 [ [Function (anonymous)], [Function (anonymous)] ]
    //遍历集合
    var result = baseMap(collection, function (value, key, collection) {
        //遍历迭代器数组，使用criteria存放每个迭代器处理当前值的结果
        var criteria = arrayMap(iteratees, function (iteratee) {
            return iteratee(value);
        });
        console.log({ 'criteria': criteria, 'index': ++index, 'value': value })
        return { 'criteria': criteria, 'index': ++index, 'value': value };
    });
    // {
    //     criteria: ['fred', 48],
    //     index: 0,
    //     value: { user: 'fred', age: 48 }
    // }
    // {
    //     criteria: ['barney', 34],
    //     index: 2,
    //     value: { user: 'barney', age: 34 }
    // }
    // {
    //     criteria: ['fred', 40],
    //     index: 4,
    //     value: { user: 'fred', age: 40 }
    // }
    // {
    //     criteria: ['barney', 36],
    //     index: 6,
    //     value: { user: 'barney', age: 36 }
    // }
    //对result进行排序，调用compareMultiple方法，并将结果返回
    return baseSortBy(result, function (object, other) {
        return compareMultiple(object, other, orders);
    });
}
var users = [
    { 'user': 'fred', 'age': 48 },
    { 'user': 'barney', 'age': 34 },
    { 'user': 'fred', 'age': 40 },
    { 'user': 'barney', 'age': 36 }
];
console.log(baseOrderBy(users, ['user', 'age'], ['asc', 'desc']))
module.exports = baseOrderBy;

```



#### 源码

```js
var arrayMap = require('./_arrayMap'),
    baseGet = require('./_baseGet'),
    baseIteratee = require('./_baseIteratee'),
    baseMap = require('./_baseMap'),
    baseSortBy = require('./_baseSortBy'),
    baseUnary = require('./_baseUnary'),
    compareMultiple = require('./_compareMultiple'),
    identity = require('./identity'),
    isArray = require('./isArray');

/**
 * The base implementation of `_.orderBy` without param guards.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function[]|Object[]|string[]} iteratees The iteratees to sort by.
 * @param {string[]} orders The sort orders of `iteratees`.
 * @returns {Array} Returns the new sorted array.
 */
function baseOrderBy(collection, iteratees, orders) {
  if (iteratees.length) {
    iteratees = arrayMap(iteratees, function(iteratee) {
      if (isArray(iteratee)) {
        return function(value) {
          return baseGet(value, iteratee.length === 1 ? iteratee[0] : iteratee);
        }
      }
      return iteratee;
    });
  } else {
    iteratees = [identity];
  }

  var index = -1;
  iteratees = arrayMap(iteratees, baseUnary(baseIteratee));

  var result = baseMap(collection, function(value, key, collection) {
    var criteria = arrayMap(iteratees, function(iteratee) {
      return iteratee(value);
    });
    return { 'criteria': criteria, 'index': ++index, 'value': value };
  });

  return baseSortBy(result, function(object, other) {
    return compareMultiple(object, other, orders);
  });
}

module.exports = baseOrderBy;

```



### baseProperty

>   The base implementation of `_.property` without support for deep paths.
>
>   property函数的基本实现，不支持深度路径

#### 参数

+   key string 要获取的属性的键名

#### 返回

**Function**

返回新的访问器函数。

#### 源码

```js
/**
 * The base implementation of `_.property` without support for deep paths.
 *
 * @private
 * @param {string} key The key of the property to get.
 * @returns {Function} Returns the new accessor function.
 */
function baseProperty(key) {
  return function(object) {
    return object == null ? undefined : object[key];
  };
}

module.exports = baseProperty;

```



### basePullAll

>   The base implementation of `_.pullAllBy` without support for iteratee shorthands.
>
>   _.pullAllBy方法的基础实现，不支持迭代器简写

#### 参数

+   array Array 待处理的数组
+   values Array 要移除的值的数组
+   iteratee Function 可选 迭代器
+   comparator Function 可选 比较器

#### 返回

**Array**

返回处理后的数组

#### 解析

```js
var arrayMap = require('./_arrayMap'),
    baseIndexOf = require('./_baseIndexOf'),
    baseIndexOfWith = require('./_baseIndexOfWith'),
    baseUnary = require('./_baseUnary'),
    copyArray = require('./_copyArray');

/** Used for built-in method references. */
var arrayProto = Array.prototype;

/** Built-in value references. */
var splice = arrayProto.splice;

/**
 * The base implementation of `_.pullAllBy` without support for iteratee
 * shorthands.
 *
 * @private
 * @param {Array} array The array to modify.
 * @param {Array} values The values to remove.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns `array`.
 */
function basePullAll(array, values, iteratee, comparator) {
   //如果存在比较器，使用接收比较器作为参数的基础索引查询方法baseIndexOfWith
  var indexOf = comparator ? baseIndexOfWith : baseIndexOf,
      index = -1,
      length = values.length,
      seen = array;
  //如果要移除的数组和待处理数组相同，拷贝一份要移除的数组
  if (array === values) {
    values = copyArray(values);
  }
  //如果存在迭代器，使用迭代器对数组每一项进行处理 seen为处理后的数组
  if (iteratee) {
    seen = arrayMap(array, baseUnary(iteratee));
  }
  //遍历要移除的数组
  while (++index < length) {
    var fromIndex = 0,
        //要移除的值
        value = values[index],
        //如果存在迭代器，使用迭代器处理一下要移除的值，将处理后的值放入computed
        computed = iteratee ? iteratee(value) : value;
	//在seen中查询computed，直到查询不到为止
    while ((fromIndex = indexOf(seen, computed, fromIndex, comparator)) > -1) {
      if (seen !== array) {
        //如果不等，说明存在遍历器，移除computed
        splice.call(seen, fromIndex, 1);
      }
       //移除computed对应的value
      splice.call(array, fromIndex, 1);
    }
  }
  return array;
}

module.exports = basePullAll;

```



#### 源码

```js
var arrayMap = require('./_arrayMap'),
    baseIndexOf = require('./_baseIndexOf'),
    baseIndexOfWith = require('./_baseIndexOfWith'),
    baseUnary = require('./_baseUnary'),
    copyArray = require('./_copyArray');

/** Used for built-in method references. */
var arrayProto = Array.prototype;

/** Built-in value references. */
var splice = arrayProto.splice;

/**
 * The base implementation of `_.pullAllBy` without support for iteratee
 * shorthands.
 *
 * @private
 * @param {Array} array The array to modify.
 * @param {Array} values The values to remove.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns `array`.
 */
function basePullAll(array, values, iteratee, comparator) {
  var indexOf = comparator ? baseIndexOfWith : baseIndexOf,
      index = -1,
      length = values.length,
      seen = array;

  if (array === values) {
    values = copyArray(values);
  }
  if (iteratee) {
    seen = arrayMap(array, baseUnary(iteratee));
  }
  while (++index < length) {
    var fromIndex = 0,
        value = values[index],
        computed = iteratee ? iteratee(value) : value;

    while ((fromIndex = indexOf(seen, computed, fromIndex, comparator)) > -1) {
      if (seen !== array) {
        splice.call(seen, fromIndex, 1);
      }
      splice.call(array, fromIndex, 1);
    }
  }
  return array;
}

module.exports = basePullAll;

```

### basePullAt

#### 参数

+   array Array 待操作数组
+   indexes Array<number> 可选 待移除值的索引

#### 返回

**Array**

返回处理后的数组

#### 源码

```js
var baseUnset = require('./_baseUnset'),
    isIndex = require('./_isIndex');

/** Used for built-in method references. */
var arrayProto = Array.prototype;

/** Built-in value references. */
var splice = arrayProto.splice;

/**
 * The base implementation of `_.pullAt` without support for individual
 * indexes or capturing the removed elements.
 *
 * @private
 * @param {Array} array The array to modify.
 * @param {number[]} indexes The indexes of elements to remove.
 * @returns {Array} Returns `array`.
 */
function basePullAt(array, indexes) {
  var length = array ? indexes.length : 0,
      lastIndex = length - 1;

  while (length--) {
    var index = indexes[length];
    if (length == lastIndex || index !== previous) {
      var previous = index;
      if (isIndex(index)) {
        splice.call(array, index, 1);
      } else {
        baseUnset(array, index);
      }
    }
  }
  return array;
}

module.exports = basePullAt;

```

### baseReduce

>   The base implementation of `_.reduce` and `_.reduceRight`, without support  for iteratee shorthands, which iterates over `collection` using `eachFunc`.
>
>   reduce和reduceRight函数的基础实现函数，不支持迭代器简写，使用eachFunc遍历集合

#### 参数

+   collection Array|Object 待操作的集合
+   iteratee Function 迭代器
+   accumulator any 初始值
+   initAccum boolean 是否使用集合第一个元素作为初始值
+   eachFunc Function 集合遍历的函数

#### 返回

**any**

返回累计后的值

#### 解析

```js
/**
 * The base implementation of `_.reduce` and `_.reduceRight`, without support
 * for iteratee shorthands, which iterates over `collection` using `eachFunc`.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @param {*} accumulator The initial value.
 * @param {boolean} initAccum Specify using the first or last element of
 *  `collection` as the initial value.
 * @param {Function} eachFunc The function to iterate over `collection`.
 * @returns {*} Returns the accumulated value.
 */
function baseReduce(collection, iteratee, accumulator, initAccum, eachFunc) {
  //使用eachFunc（reduce使用baseEach，reduceRight使用baseEachRight）进行遍历
  eachFunc(collection, function(value, index, collection) {
    //initAccum为true时使用当前值（也就是数组第一个或者数组最后一个值）作为初始值，然后initAccum置为false
    accumulator = initAccum
      ? (initAccum = false, value)
      : iteratee(accumulator, value, index, collection);
  });
  return accumulator;
}

module.exports = baseReduce;

```

#### 源码

```js
/**
 * The base implementation of `_.reduce` and `_.reduceRight`, without support
 * for iteratee shorthands, which iterates over `collection` using `eachFunc`.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @param {*} accumulator The initial value.
 * @param {boolean} initAccum Specify using the first or last element of
 *  `collection` as the initial value.
 * @param {Function} eachFunc The function to iterate over `collection`.
 * @returns {*} Returns the accumulated value.
 */
function baseReduce(collection, iteratee, accumulator, initAccum, eachFunc) {
  eachFunc(collection, function(value, index, collection) {
    accumulator = initAccum
      ? (initAccum = false, value)
      : iteratee(accumulator, value, index, collection);
  });
  return accumulator;
}

module.exports = baseReduce;

```

### baseRest

> The base implementation of `_.rest` which doesn't validate or coerce arguments.
>
> ' _.rest '的基本实现 '，它不验证或强制参数。

#### 参数

+ func 要应用rest形参的函数。
+ start 可选 剩余参数的起始位置。默认为func.length-1

#### 返回

**Function**

返回一个新的可以传入rest形参的函数

（rest形参 为解决传入的参数数量不一定， rest parameter(Rest 参数) 本身就是数组，数组的相关的方法都可以用。以“...”开头，它是一个数组）

#### 源码

```js
var identity = require('./identity'),
    overRest = require('./_overRest'),
    setToString = require('./_setToString');

/**
 * The base implementation of `_.rest` which doesn't validate or coerce arguments.
 *
 * @private
 * @param {Function} func The function to apply a rest parameter to.
 * @param {number} [start=func.length-1] The start position of the rest parameter.
 * @returns {Function} Returns the new function.
 */
function baseRest(func, start) {
  return setToString(overRest(func, start, identity), func + '');
}

module.exports = baseRest;

```

### baseSample

>   The base implementation of `_.sample`.
>
>   sample的基础实现函数

#### 参数

+   collection Array|Object 待取值的集合

#### 返回

**any**

返回集合中的一个随机值

#### 源码

源码中涉及到的函数

+   [arraySample](#arraySample)
+   [values](#values对象值数组)

```js
var arraySample = require('./_arraySample'),
    values = require('./values');

/**
 * The base implementation of `_.sample`.
 *
 * @private
 * @param {Array|Object} collection The collection to sample.
 * @returns {*} Returns the random element.
 */
function baseSample(collection) {
  return arraySample(values(collection));
}

module.exports = baseSample;

```

### baseSampleSize

>   The base implementation of `_.sampleSize` without param guards.
>
>   sampleSize的基础实现函数，没有迭代守卫参数

#### 参数

+   collection Array|Object 待取值的集合
+   n number 要获取的随机元素的个数

#### 返回

**Array** 

返回n个随机元素组成的数组

#### 源码

```js
var baseClamp = require('./_baseClamp'),
    shuffleSelf = require('./_shuffleSelf'),
    values = require('./values');

/**
 * The base implementation of `_.sampleSize` without param guards.
 *
 * @private
 * @param {Array|Object} collection The collection to sample.
 * @param {number} n The number of elements to sample.
 * @returns {Array} Returns the random elements.
 */
function baseSampleSize(collection, n) {
  var array = values(collection);
  return shuffleSelf(array, baseClamp(n, 0, array.length));
}

module.exports = baseSampleSize;

```



### baseSet

>   The base implementation of `_.set`.
>
>   set函数的基础实现

#### 参数

+   object Object 待处理的对象
+   path Array|string 待设置属性的路径
+   value any 待设置的值
+   customizer Function 可选 自定义路径创建

#### 返回

**Object**

返回操作后的对象

#### 解析

```js
var assignValue = require('./_assignValue'),
    castPath = require('./_castPath'),
    isIndex = require('./_isIndex'),
    isObject = require('./isObject'),
    toKey = require('./_toKey');

/**
 * The base implementation of `_.set`.
 *
 * @private
 * @param {Object} object The object to modify.
 * @param {Array|string} path The path of the property to set.
 * @param {*} value The value to set.
 * @param {Function} [customizer] The function to customize path creation.
 * @returns {Object} Returns `object`.
 */
function baseSet(object, path, value, customizer) {
  //如果不是对象，直接返回object
  if (!isObject(object)) {
    return object;
  }
  //将path处理为路径数组
  path = castPath(path, object);

  var index = -1,
      length = path.length,
      lastIndex = length - 1,
      nested = object;
  //遍历路径数组
  while (nested != null && ++index < length) {
    var key = toKey(path[index]),
        newValue = value;
	//如果待设置的属性为下面三个，则直接返回object
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      return object;
    }

    if (index != lastIndex) {
      //保存原值
      var objValue = nested[key];
      //获取新值
      newValue = customizer ? customizer(objValue, key, nested) : undefined;
      if (newValue === undefined) {
        //如果原值是个对象，将这个对象赋给newValue，否则如果index+1是个索引值（数字），说明设置的属性是个数组，赋[]
        newValue = isObject(objValue)
          ? objValue
          : (isIndex(path[index + 1]) ? [] : {});
      }
    }
    //赋值
    assignValue(nested, key, newValue);
    nested = nested[key];
  }
  return object;
}

module.exports = baseSet;

```



#### 源码

```js
var assignValue = require('./_assignValue'),
    castPath = require('./_castPath'),
    isIndex = require('./_isIndex'),
    isObject = require('./isObject'),
    toKey = require('./_toKey');

/**
 * The base implementation of `_.set`.
 *
 * @private
 * @param {Object} object The object to modify.
 * @param {Array|string} path The path of the property to set.
 * @param {*} value The value to set.
 * @param {Function} [customizer] The function to customize path creation.
 * @returns {Object} Returns `object`.
 */
function baseSet(object, path, value, customizer) {
  if (!isObject(object)) {
    return object;
  }
  path = castPath(path, object);

  var index = -1,
      length = path.length,
      lastIndex = length - 1,
      nested = object;

  while (nested != null && ++index < length) {
    var key = toKey(path[index]),
        newValue = value;

    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      return object;
    }

    if (index != lastIndex) {
      var objValue = nested[key];
      newValue = customizer ? customizer(objValue, key, nested) : undefined;
      if (newValue === undefined) {
        newValue = isObject(objValue)
          ? objValue
          : (isIndex(path[index + 1]) ? [] : {});
      }
    }
    assignValue(nested, key, newValue);
    nested = nested[key];
  }
  return object;
}

module.exports = baseSet;

```

### baseShuffle

>   The base implementation of `_.shuffle`.
>
>   shuffle 的基本实现

#### 参数

+   collection Array|Object 待洗牌的数组

#### 返回

**Array**

返回新的打乱后的数组

#### 源码

源码中涉及到的函数

+   [shuffleSelf](#shuffleSelf)
+   [values](#values对象值数组)

```js
var shuffleSelf = require('./_shuffleSelf'),
    values = require('./values');

/**
 * The base implementation of `_.shuffle`.
 *
 * @private
 * @param {Array|Object} collection The collection to shuffle.
 * @returns {Array} Returns the new shuffled array.
 */
function baseShuffle(collection) {
  return shuffleSelf(values(collection));
}

module.exports = baseShuffle;

```



### baseSlice

与Array.slice相同

> The base implementation of `_.slice` without an iteratee call guard.
>
> ' _.slice '的基本实现。没有迭代调用守卫的Slice。

#### 参数

+ array 待截取数组
+ start 开始位置
+ end 结束位置

#### 返回

**Array**截取的数组

#### 源码

```js
/**
 * The base implementation of `_.slice` without an iteratee call guard.
 *
 * @private
 * @param {Array} array The array to slice.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns the slice of `array`.
 */
function baseSlice(array, start, end) {
  var index = -1,
      length = array.length;

  if (start < 0) {
    start = -start > length ? 0 : (length + start);
  }
  end = end > length ? length : end;
  if (end < 0) {
    end += length;
  }
  length = start > end ? 0 : ((end - start) >>> 0);
  start >>>= 0;

  var result = Array(length);
  while (++index < length) {
    result[index] = array[index + start];
  }
  return result;
}

module.exports = baseSlice;

```

### baseSome

>   The base implementation of `_.some` without support for iteratee shorthands.
>
>   some函数的基础实现函数，不支持迭代器简写

#### 参数

+   collection Array|Object 待操作的集合
+   predicate Function 可选 断言

#### 返回

**boolean**

如果集合中存在元素通过了predicate检查，则返回true，否则返回false

#### 源码

源码中涉及到的函数

+   [baseEach](#baseEach)

```js
var baseEach = require('./_baseEach');

/**
 * The base implementation of `_.some` without support for iteratee shorthands.
 *
 * @private
 * @param {Array|Object} collection The collection to iterate over.
 * @param {Function} predicate The function invoked per iteration.
 * @returns {boolean} Returns `true` if any element passes the predicate check,
 *  else `false`.
 */
function baseSome(collection, predicate) {
  var result;

  baseEach(collection, function(value, index, collection) {
    //predicate 返回true时，!result 为false 打断遍历
    result = predicate(value, index, collection);
    return !result;
  });
  return !!result;
}

module.exports = baseSome;

```



### baseSortedIndex

>   The base implementation of `_.sortedIndex` and `_.sortedLastIndex` which performs a binary search of `array` to determine the index at which `value` should be inserted into `array` in order to maintain its sort order.

#### 参数

+   array Array 待操作的有序数组
+   value any 待评估的值
+   retHighest boolean 可选 指定返回最高限定索引

#### 返回

**number**

返回数组中应该插入值的索引

#### 解析

```js
var baseSortedIndexBy = require('./_baseSortedIndexBy'),
    identity = require('./identity'),
    isSymbol = require('./isSymbol');

/** Used as references for the maximum length and index of an array. */
var MAX_ARRAY_LENGTH = 4294967295,
    HALF_MAX_ARRAY_LENGTH = MAX_ARRAY_LENGTH >>> 1;

/**
 * The base implementation of `_.sortedIndex` and `_.sortedLastIndex` which
 * performs a binary search of `array` to determine the index at which `value`
 * should be inserted into `array` in order to maintain its sort order.
 *
 * @private
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {boolean} [retHighest] Specify returning the highest qualified index.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 */
function baseSortedIndex(array, value, retHighest) {
  var low = 0,
      high = array == null ? low : array.length;
  //如果value是数字
  if (typeof value == 'number' && value === value && high <= HALF_MAX_ARRAY_LENGTH) {
    //二分法获取插入位置
    while (low < high) {
      var mid = (low + high) >>> 1,
          computed = array[mid];

      if (computed !== null && !isSymbol(computed) &&
          (retHighest ? (computed <= value) : (computed < value))) {
        low = mid + 1;
      } else {
        high = mid;
      }
    }
    return high;
  }
  return baseSortedIndexBy(array, value, identity, retHighest);
}

module.exports = baseSortedIndex;

```



#### 源码

```js
var baseSortedIndexBy = require('./_baseSortedIndexBy'),
    identity = require('./identity'),
    isSymbol = require('./isSymbol');

/** Used as references for the maximum length and index of an array. */
var MAX_ARRAY_LENGTH = 4294967295,
    HALF_MAX_ARRAY_LENGTH = MAX_ARRAY_LENGTH >>> 1;

/**
 * The base implementation of `_.sortedIndex` and `_.sortedLastIndex` which
 * performs a binary search of `array` to determine the index at which `value`
 * should be inserted into `array` in order to maintain its sort order.
 *
 * @private
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {boolean} [retHighest] Specify returning the highest qualified index.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 */
function baseSortedIndex(array, value, retHighest) {
  var low = 0,
      high = array == null ? low : array.length;

  if (typeof value == 'number' && value === value && high <= HALF_MAX_ARRAY_LENGTH) {
    while (low < high) {
      var mid = (low + high) >>> 1,
          computed = array[mid];

      if (computed !== null && !isSymbol(computed) &&
          (retHighest ? (computed <= value) : (computed < value))) {
        low = mid + 1;
      } else {
        high = mid;
      }
    }
    return high;
  }
  return baseSortedIndexBy(array, value, identity, retHighest);
}

module.exports = baseSortedIndex;

```

### baseSortedIndexBy

>   The base implementation of `_.sortedIndexBy` and `_.sortedLastIndexBy`  which invokes `iteratee` for `value` and each element of `array` to compute their sort ranking. The iteratee is invoked with one argument; (value).
>
>   sortedLastIndexBy的基本实现。，它调用iteratee来计算数组的值和每个元素的排序。iteratee调用时只有一个参数;(value)。

#### 参数

+   array Array 待检查的有序数组
+   value any 待评估的值
+   iteratee Function 迭代器
+   retHighest boolean 是否指定返回最高限定索引

#### 返回

**number**

应该在数组中插入的索引位置

#### 解析

```js
var isSymbol = require('./isSymbol');

/** Used as references for the maximum length and index of an array. */
var MAX_ARRAY_LENGTH = 4294967295,
    MAX_ARRAY_INDEX = MAX_ARRAY_LENGTH - 1;

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeFloor = Math.floor,
    nativeMin = Math.min;

/**
 * The base implementation of `_.sortedIndexBy` and `_.sortedLastIndexBy`
 * which invokes `iteratee` for `value` and each element of `array` to compute
 * their sort ranking. The iteratee is invoked with one argument; (value).
 *
 * @private
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {Function} iteratee The iteratee invoked per element.
 * @param {boolean} [retHighest] Specify returning the highest qualified index.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 */
function baseSortedIndexBy(array, value, iteratee, retHighest) {
  var low = 0,
      high = array == null ? 0 : array.length;
  if (high === 0) {
    return 0;
  }
  //使用迭代器对value进行处理，并重新赋值给value
  value = iteratee(value);
  //对value进行类型判断
  var valIsNaN = value !== value,
      valIsNull = value === null,
      valIsSymbol = isSymbol(value),
      valIsUndefined = value === undefined;
  //二分法
  while (low < high) {
    var mid = nativeFloor((low + high) / 2),
        //对当前数组中的比较值进行处理，将处理后的数据赋值给computed
        computed = iteratee(array[mid]),
        /判断computed类型
        othIsDefined = computed !== undefined,
        othIsNull = computed === null,
        othIsReflexive = computed === computed,
        othIsSymbol = isSymbol(computed);

    if (valIsNaN) {
      var setLow = retHighest || othIsReflexive;
    } else if (valIsUndefined) {
      setLow = othIsReflexive && (retHighest || othIsDefined);
    } else if (valIsNull) {
      setLow = othIsReflexive && othIsDefined && (retHighest || !othIsNull);
    } else if (valIsSymbol) {
      setLow = othIsReflexive && othIsDefined && !othIsNull && (retHighest || !othIsSymbol);
    } else if (othIsNull || othIsSymbol) {
      setLow = false;
    } else {
      setLow = retHighest ? (computed <= value) : (computed < value);
    }
    if (setLow) {
      low = mid + 1;
    } else {
      high = mid;
    }
  }
  return nativeMin(high, MAX_ARRAY_INDEX);
}

module.exports = baseSortedIndexBy;

```



#### 源码

```js
var isSymbol = require('./isSymbol');

/** Used as references for the maximum length and index of an array. */
var MAX_ARRAY_LENGTH = 4294967295,
    MAX_ARRAY_INDEX = MAX_ARRAY_LENGTH - 1;

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeFloor = Math.floor,
    nativeMin = Math.min;

/**
 * The base implementation of `_.sortedIndexBy` and `_.sortedLastIndexBy`
 * which invokes `iteratee` for `value` and each element of `array` to compute
 * their sort ranking. The iteratee is invoked with one argument; (value).
 *
 * @private
 * @param {Array} array The sorted array to inspect.
 * @param {*} value The value to evaluate.
 * @param {Function} iteratee The iteratee invoked per element.
 * @param {boolean} [retHighest] Specify returning the highest qualified index.
 * @returns {number} Returns the index at which `value` should be inserted
 *  into `array`.
 */
function baseSortedIndexBy(array, value, iteratee, retHighest) {
  var low = 0,
      high = array == null ? 0 : array.length;
  if (high === 0) {
    return 0;
  }

  value = iteratee(value);
  var valIsNaN = value !== value,
      valIsNull = value === null,
      valIsSymbol = isSymbol(value),
      valIsUndefined = value === undefined;

  while (low < high) {
    var mid = nativeFloor((low + high) / 2),
        computed = iteratee(array[mid]),
        othIsDefined = computed !== undefined,
        othIsNull = computed === null,
        othIsReflexive = computed === computed,
        othIsSymbol = isSymbol(computed);

    if (valIsNaN) {
      var setLow = retHighest || othIsReflexive;
    } else if (valIsUndefined) {
      setLow = othIsReflexive && (retHighest || othIsDefined);
    } else if (valIsNull) {
      setLow = othIsReflexive && othIsDefined && (retHighest || !othIsNull);
    } else if (valIsSymbol) {
      setLow = othIsReflexive && othIsDefined && !othIsNull && (retHighest || !othIsSymbol);
    } else if (othIsNull || othIsSymbol) {
      setLow = false;
    } else {
      setLow = retHighest ? (computed <= value) : (computed < value);
    }
    if (setLow) {
      low = mid + 1;
    } else {
      high = mid;
    }
  }
  return nativeMin(high, MAX_ARRAY_INDEX);
}

module.exports = baseSortedIndexBy;

```

### baseSortedUniq

>   The base implementation of `_.sortedUniq` and `_.sortedUniqBy` without support for iteratee shorthands.
>
>   _.sortedUniq 和 _.sortedUniqBy的基本实现函数，不支持迭代器简写

#### 参数

+   array Array 待去重的数组
+   iteratee Function 可选 迭代器 不支持简写 调用每个元素

#### 返回

**Array**

返回去重后的新数组

#### 解析

```js
var eq = require('./eq');

/**
 * The base implementation of `_.sortedUniq` and `_.sortedUniqBy` without
 * support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 */
function baseSortedUniq(array, iteratee) {
  var index = -1,
      length = array.length,
      resIndex = 0,
      result = [];
  //遍历array
  while (++index < length) {
    var value = array[index],
        //如果存在迭代器，使用迭代器处理每个值，否则使用原值
        //将处理好的值或者原值赋给computed
        computed = iteratee ? iteratee(value) : value;
	//如果index=0，将array[0]存到结果中，并使用seen保存当前处理值
    //如果index>0,将当前处理值computed和seen中保存的上一次存到结果数组中的值所对应的处理值进行比较，如果不等，
    //存放当前处理值，将当前value存放到结果数组中
    if (!index || !eq(computed, seen)) {
      var seen = computed;
      result[resIndex++] = value === 0 ? 0 : value;
    }
  }
  return result;
}

module.exports = baseSortedUniq;

```



#### 源码

```js
var eq = require('./eq');

/**
 * The base implementation of `_.sortedUniq` and `_.sortedUniqBy` without
 * support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 */
function baseSortedUniq(array, iteratee) {
  var index = -1,
      length = array.length,
      resIndex = 0,
      result = [];

  while (++index < length) {
    var value = array[index],
        computed = iteratee ? iteratee(value) : value;

    if (!index || !eq(computed, seen)) {
      var seen = computed;
      result[resIndex++] = value === 0 ? 0 : value;
    }
  }
  return result;
}

module.exports = baseSortedUniq;

```

### baseTimes

>   The base implementation of `_.times` without support for iteratee shorthands or max array length checks.
>
>   times函数的基础实现，不支持迭代器简写和最大数组长度检查

#### 参数

+   n number 迭代器的调用次数
+   iteratee Function 迭代器 被调用n次的函数  参数为当前调用次数

#### 返回

**Array**

返回迭代器每次调用结果组成的数组

#### 源码

```js
/**
 * The base implementation of `_.times` without support for iteratee shorthands
 * or max array length checks.
 *
 * @private
 * @param {number} n The number of times to invoke `iteratee`.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns the array of results.
 */
function baseTimes(n, iteratee) {
  var index = -1,
      result = Array(n);

  while (++index < n) {
    result[index] = iteratee(index);
  }
  return result;
}

module.exports = baseTimes;

```



### baseToString

> The base implementation of `_.toString` which doesn't convert nullish values to empty strings.
>
> ' _.toString '的基本实现。它不会将空值转换为空字符串。

效果类似于toSting

#### 参数

+ value	需要被转换的值

#### 返回

**string**

被转换后的值

#### 源码

源码中涉及的方法

+ [arrayMap](#arrayMap)

+ [symbolToString](#symbolToString)

源码中涉及的常量

+ [INFINITY](#Number)

```js
/**
 * The base implementation of `_.toString` which doesn't convert nullish
 * values to empty strings.
 *
 * @private
 * @param {*} value The value to process.
 * @returns {string} Returns the string.
 */
function baseToString(value) {
    // Exit early for strings to avoid a performance hit in some environments.
    if (typeof value == 'string') {
        return value;
    }
    if (isArray(value)) {
        // Recursively convert values (susceptible to call stack limits).
        return arrayMap(value, baseToString) + '';
    }
    if (isSymbol(value)) {
        return symbolToString ? symbolToString.call(value) : '';
    }
    var result = (value + '');
    return (result == '0' && (1 / value) == -INFINITY) ? '-0' : result;
}
```

### baseTrim

>   The base implementation of `_.trim`.
>
>   trim函数的基础实现

#### 参数

+   string string 待操作的字符串

#### 返回

**string**

返回移除空格后的字符串

#### 源码

```js
var trimmedEndIndex = require('./_trimmedEndIndex');

/** Used to match leading whitespace. */
var reTrimStart = /^\s+/;

/**
 * The base implementation of `_.trim`.
 *
 * @private
 * @param {string} string The string to trim.
 * @returns {string} Returns the trimmed string.
 */
function baseTrim(string) {
  return string
    ? string.slice(0, trimmedEndIndex(string) + 1).replace(reTrimStart, '')
    : string;
}

module.exports = baseTrim;

```



### baseUniq

>   The base implementation of `_.uniqBy` without support for iteratee shorthands.
>
>   uniqBy函数的基本实现，不支持迭代器简写

#### 参数

+   array Array 待去重数组
+   iteratee Function 可选 迭代器 调用每个参数
+   comparator Function 可选 比较器

#### 返回

**Array**

返回新的去重后的数组

#### 解析

```js
var SetCache = require('./_SetCache'),
    arrayIncludes = require('./_arrayIncludes'),
    arrayIncludesWith = require('./_arrayIncludesWith'),
    cacheHas = require('./_cacheHas'),
    createSet = require('./_createSet'),
    setToArray = require('./_setToArray');

/** Used as the size to enable large array optimizations. */
var LARGE_ARRAY_SIZE = 200;

/**
 * The base implementation of `_.uniqBy` without support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 */
function baseUniq(array, iteratee, comparator) {
  var index = -1,
      includes = arrayIncludes,//是否包含的函数引用
      length = array.length,
      isCommon = true,//是否是普通比较，即arrayIncludes
      result = [],
      seen = result;
  //如果存在比较器，则includes的引用为arrayIncludesWith函数
  if (comparator) {
    isCommon = false;
    includes = arrayIncludesWith;
  }//否则如果待去重的数组长度大于最大长度限制（数组太大）
  else if (length >= LARGE_ARRAY_SIZE) {
    //如果没有迭代器，直接使用将数组转为set集合去重，再将set集合转为数组返回
    var set = iteratee ? null : createSet(array);
    if (set) {
      return setToArray(set);
    }
    isCommon = false;
    includes = cacheHas;
    //缓存
    seen = new SetCache;
  }//如果长度小于限制，且无比较器
  else {
    seen = iteratee ? [] : result;
  }
  //遍历待去重的数组
  outer:
  while (++index < length) {
    var value = array[index],
        //如果存在迭代器，使用迭代器处理当前值并保存，否则直接保存当前值
        computed = iteratee ? iteratee(value) : value;

    value = (comparator || value !== 0) ? value : 0;
    //如果是普通比较，即arrayIncludes，并且computed不是NaN
    if (isCommon && computed === computed) {
      var seenIndex = seen.length;
      //倒序遍历seen
      while (seenIndex--) {
        //如果seen中存在computed，即当前结果中已有computed，则outer循环跳入下一轮
        if (seen[seenIndex] === computed) {
          continue outer;
        }
      }
      //如果seen中不存在computed
      //存在迭代器，seen保存处理后的值computed
      if (iteratee) {
        seen.push(computed);
      }
      //结果数组保存当前value
      result.push(value);
    }
   	//如果是其它包含函数引用，则调用引用进行判断，如果seen中不包含computed，进入if
    else if (!includes(seen, computed, comparator)) {
      //如果seen和result引用不同，即上述语句seen = new SetCache或者seen = iteratee ? [] : result中，
      //seen为缓存或者iteratee迭代器存在
      //seen中保存处理后的值computed
      if (seen !== result) {
        seen.push(computed);
      }
      //结果数组保存当前value
      result.push(value);
    }
  }
  return result;
}

module.exports = baseUniq;

```



#### 源码

```js
var SetCache = require('./_SetCache'),
    arrayIncludes = require('./_arrayIncludes'),
    arrayIncludesWith = require('./_arrayIncludesWith'),
    cacheHas = require('./_cacheHas'),
    createSet = require('./_createSet'),
    setToArray = require('./_setToArray');

/** Used as the size to enable large array optimizations. */
var LARGE_ARRAY_SIZE = 200;

/**
 * The base implementation of `_.uniqBy` without support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new duplicate free array.
 */
function baseUniq(array, iteratee, comparator) {
  var index = -1,
      includes = arrayIncludes,
      length = array.length,
      isCommon = true,
      result = [],
      seen = result;

  if (comparator) {
    isCommon = false;
    includes = arrayIncludesWith;
  }
  else if (length >= LARGE_ARRAY_SIZE) {
    var set = iteratee ? null : createSet(array);
    if (set) {
      return setToArray(set);
    }
    isCommon = false;
    includes = cacheHas;
    seen = new SetCache;
  }
  else {
    seen = iteratee ? [] : result;
  }
  outer:
  while (++index < length) {
    var value = array[index],
        computed = iteratee ? iteratee(value) : value;

    value = (comparator || value !== 0) ? value : 0;
    if (isCommon && computed === computed) {
      var seenIndex = seen.length;
      while (seenIndex--) {
        if (seen[seenIndex] === computed) {
          continue outer;
        }
      }
      if (iteratee) {
        seen.push(computed);
      }
      result.push(value);
    }
    else if (!includes(seen, computed, comparator)) {
      if (seen !== result) {
        seen.push(computed);
      }
      result.push(value);
    }
  }
  return result;
}

module.exports = baseUniq;

```

### baseValues

>   The base implementation of `_.values` and `_.valuesIn` which creates an array of `object` property values corresponding to the property names of `props`.
>
>   values和valuesIn的基本实现。获取由指定属性值构成的新数组

#### 参数

+   object Object 待操作对象
+   props Array 对象的待获取值的属性名数值

#### 返回

**Array**

返回由对象中指定值构成的数组

#### 源码

源码中涉及的函数

+   [arrayMap](#arrayMap)

```js
var arrayMap = require('./_arrayMap');

/**
 * The base implementation of `_.values` and `_.valuesIn` which creates an
 * array of `object` property values corresponding to the property names
 * of `props`.
 *
 * @private
 * @param {Object} object The object to query.
 * @param {Array} props The property names to get values for.
 * @returns {Object} Returns the array of property values.
 */
function baseValues(object, props) {
  return arrayMap(props, function(key) {
    return object[key];
  });
}

module.exports = baseValues;

```



### baseWhile

>   The base implementation of methods like `_.dropWhile` and `_.takeWhile` without support for iteratee shorthands.
>
>   ' _. takeWhile _.dropWhile'等方法的基本实现。不支持迭代者简写的。

#### 参数

-   array 待处理数组
-   predicate 迭代 判断函数
-   isDrop 可选 指定删除元素而不是获取元素。
-   fromRight 可选 指定从右到左迭代。

#### 返回 

**Array**

过滤后的数组

#### 解析

```js
var baseSlice = require('./_baseSlice');

/**
 * The base implementation of methods like `_.dropWhile` and `_.takeWhile`
 * without support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to query.
 * @param {Function} predicate The function invoked per iteration.
 * @param {boolean} [isDrop] Specify dropping elements instead of taking them.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {Array} Returns the slice of `array`.
 */
function baseWhile(array, predicate, isDrop, fromRight) {
  var length = array.length,
      //如果指定从右（末尾）开始迭代，则索引初始值为数组长度
      index = fromRight ? length : -1;

  //当fromRight为true时从末尾开始遍历，直到predicate判断方法返回false，得到此时的index索引值
  while ((fromRight ? index-- : ++index < length) &&
    predicate(array[index], index, array)) {}

  //如果是删除元素，返回baseSlice(array, (fromRight ? 0 : index), (fromRight ? index + 1 : length))截取的数组
  //否则返回baseSlice(array, (fromRight ? index + 1 : 0), (fromRight ? length : index))截取的数组
  return isDrop
    ? baseSlice(array, (fromRight ? 0 : index), (fromRight ? index + 1 : length))
    : baseSlice(array, (fromRight ? index + 1 : 0), (fromRight ? length : index));
}

module.exports = baseWhile;

```



#### 源码

源码中涉及的函数

+   [baseSlice](#baseSlice)

```js
var baseSlice = require('./_baseSlice');

/**
 * The base implementation of methods like `_.dropWhile` and `_.takeWhile`
 * without support for iteratee shorthands.
 *
 * @private
 * @param {Array} array The array to query.
 * @param {Function} predicate The function invoked per iteration.
 * @param {boolean} [isDrop] Specify dropping elements instead of taking them.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {Array} Returns the slice of `array`.
 */
function baseWhile(array, predicate, isDrop, fromRight) {
  var length = array.length,
      index = fromRight ? length : -1;

  while ((fromRight ? index-- : ++index < length) &&
    predicate(array[index], index, array)) {}

  return isDrop
    ? baseSlice(array, (fromRight ? 0 : index), (fromRight ? index + 1 : length))
    : baseSlice(array, (fromRight ? index + 1 : 0), (fromRight ? length : index));
}

module.exports = baseWhile;

```

### baseXor

>   The base implementation of methods like `_.xor`, without support for iteratee shorthands, that accepts an array of arrays to inspect.
>
>   xor函数的基本实现函数，不支持迭代器简写

#### 参数

+   arrays Array 待检查数组列表（这至少是一个二维数组）
+   iteratee Function 可选 迭代器 调用每个元素
+   comparator Function 可选 比较器

#### 返回

**Array**

返回新的过滤后的数组

#### 解析

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseUniq = require('./_baseUniq');

/**
 * The base implementation of methods like `_.xor`, without support for
 * iteratee shorthands, that accepts an array of arrays to inspect.
 *
 * @private
 * @param {Array} arrays The arrays to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of values.
 */
function baseXor(arrays, iteratee, comparator) {
  //length 为元素个数，即待检查的数组个数
  var length = arrays.length;
  if (length < 2) {
    //如果只有一个待检查数组，对这个数组进行去重并返回，如果没有待检查数组，则直接返回空数组[]
    return length ? baseUniq(arrays[0]) : [];
  }
  var index = -1,
      result = Array(length);
  //遍历待检查的数组列表
  while (++index < length) {
    var array = arrays[index],
        othIndex = -1;
	//遍历待检查的数组列表
    while (++othIndex < length) {
      //如果内外围当前遍历到的不是同一个数组
      if (othIndex != index) {
        //调用baseDifference，当result[index]不存在时（内循环刚开始第一次遍历,内循环中othIndex等于0或1时）,
        //取出array与arrays[othIndex]中的不同值（唯一值）组成数组并放到result[index]中
        //当result[index]存在时，取出result[index]与arrays[othIndex]中的不同值（唯一值）组成数组并放
        //到result[index]中
        result[index] = baseDifference(result[index] || array, arrays[othIndex], iteratee, comparator);
      }
    }
  }
  //result中每一个元素都是一个数组，每个元素都是当前下标对应的arrays列表中的子数组与其它子数组进行对等差分求集的结果
  //所以还需要将result展开去重获取结果并返回
  return baseUniq(baseFlatten(result, 1), iteratee, comparator);
}

module.exports = baseXor;

```



#### 源码

源码中涉及的函数

+   [baseDifference](#baseDifference)
+   [baseFlatten](#baseFlatten)
+   [baseUniq](#baseUniq)

```js
var baseDifference = require('./_baseDifference'),
    baseFlatten = require('./_baseFlatten'),
    baseUniq = require('./_baseUniq');

/**
 * The base implementation of methods like `_.xor`, without support for
 * iteratee shorthands, that accepts an array of arrays to inspect.
 *
 * @private
 * @param {Array} arrays The arrays to inspect.
 * @param {Function} [iteratee] The iteratee invoked per element.
 * @param {Function} [comparator] The comparator invoked per element.
 * @returns {Array} Returns the new array of values.
 */
function baseXor(arrays, iteratee, comparator) {
  var length = arrays.length;
  if (length < 2) {
    return length ? baseUniq(arrays[0]) : [];
  }
  var index = -1,
      result = Array(length);

  while (++index < length) {
    var array = arrays[index],
        othIndex = -1;

    while (++othIndex < length) {
      if (othIndex != index) {
        result[index] = baseDifference(result[index] || array, arrays[othIndex], iteratee, comparator);
      }
    }
  }
  return baseUniq(baseFlatten(result, 1), iteratee, comparator);
}

module.exports = baseXor;

```

### baseZipObject

>   This base implementation of `_.zipObject` which assigns values using `assignFunc`.
>
>   zipObject函数的基本实现，使用assignFunc进行对象的赋值

#### 参数

+   props Array 属性名数组
+   values Array 属性值数组
+   assignFunc 分配值的函数

#### 返回

**Object**

返回构造好的新的对象

#### 源码

```js
/**
 * This base implementation of `_.zipObject` which assigns values using `assignFunc`.
 *
 * @private
 * @param {Array} props The property identifiers.
 * @param {Array} values The property values.
 * @param {Function} assignFunc The function to assign values.
 * @returns {Object} Returns the new object.
 */
function baseZipObject(props, values, assignFunc) {
  var index = -1,
      length = props.length,
      valsLength = values.length,
      result = {};

  while (++index < length) {
    var value = index < valsLength ? values[index] : undefined;
    assignFunc(result, props[index], value);
  }
  return result;
}

module.exports = baseZipObject;

```



### castArrayLikeObject

>   Casts `value` to an empty array if it's not an array like object.
>
>   将' value '强制转换为空数组(如果它不是数组类对象)。

#### 参数

+   value  any 待转换的值

#### 返回

**Array|Object**

原value或空数组

#### 源码

源码中涉及的方法

+   [isArrayLikeObject](#isArrayLikeObject检查类数组对象)

```js
var isArrayLikeObject = require('./isArrayLikeObject');

/**
 * Casts `value` to an empty array if it's not an array like object.
 *
 * @private
 * @param {*} value The value to inspect.
 * @returns {Array|Object} Returns the cast array-like object.
 */
function castArrayLikeObject(value) {
  return isArrayLikeObject(value) ? value : [];
}

module.exports = castArrayLikeObject;

```

### castFunction

>   Casts `value` to `identity` if it's not a function.
>
>   如果不是函数，则将' value '转换为' identity '。

#### 参数

+   value any 待检查的值

#### 返回

**Function**

如果value是函数，返回value，否则返回identity

#### 源码

```js
var identity = require('./identity');

/**
 * Casts `value` to `identity` if it's not a function.
 *
 * @private
 * @param {*} value The value to inspect.
 * @returns {Function} Returns cast function.
 */
function castFunction(value) {
  return typeof value == 'function' ? value : identity;
}

module.exports = castFunction;

```



### castPath

>  Casts `value` to a path array if it's not one.
>
> 将' value '强制转换为路径数组(如果它不是1的话)。

#### 参数

+ value	需要转换的字符串
+ object 向isKey传递的参数，用于判断value是否是属性名（纯字母）形式的字符串

#### 返回

**Array**

属性名数组

#### 源码

源码内涉及的方法

+ [isKey](#isKey)

```js
    /**
     * Casts `value` to a path array if it's not one.
     *
     * @private
     * @param {*} value The value to inspect.
     * @param {Object} [object] The object to query keys on.
     * @returns {Array} Returns the cast property path array.
     */
    function castPath(value, object) {
      if (isArray(value)) {
        return value;
      }
      return isKey(value, object) ? [value] : stringToPath(toString(value));
    }
```

### compareAscending

>   Compares values to sort them in ascending order.
>
>   比较值以升序排序。

#### 参数

+   value any 待比较的值
+   other any 待比较的值

#### 返回

**number**

value更大，返回1，value更小，返回-1，否则返回0

#### 源码

源码中涉及的函数

+   [isSymbol](#isSymbol检查符号)

```js
var isSymbol = require('./isSymbol');

/**
 * Compares values to sort them in ascending order.
 *
 * @private
 * @param {*} value The value to compare.
 * @param {*} other The other value to compare.
 * @returns {number} Returns the sort order indicator for `value`.
 */
function compareAscending(value, other) {
  if (value !== other) {
    var valIsDefined = value !== undefined,
        valIsNull = value === null,
        valIsReflexive = value === value,
        valIsSymbol = isSymbol(value);

    var othIsDefined = other !== undefined,
        othIsNull = other === null,
        othIsReflexive = other === other,
        othIsSymbol = isSymbol(other);

    if ((!othIsNull && !othIsSymbol && !valIsSymbol && value > other) ||
        (valIsSymbol && othIsDefined && othIsReflexive && !othIsNull && !othIsSymbol) ||
        (valIsNull && othIsDefined && othIsReflexive) ||
        (!valIsDefined && othIsReflexive) ||
        !valIsReflexive) {
      return 1;
    }
    if ((!valIsNull && !valIsSymbol && !othIsSymbol && value < other) ||
        (othIsSymbol && valIsDefined && valIsReflexive && !valIsNull && !valIsSymbol) ||
        (othIsNull && valIsDefined && valIsReflexive) ||
        (!othIsDefined && valIsReflexive) ||
        !othIsReflexive) {
      return -1;
    }
  }
  return 0;
}

module.exports = compareAscending;

```

### composeArgs

>   Creates an array that is the composition of partially applied arguments,  placeholders, and provided arguments into a single array of arguments.
>
>   创建一个数组，由args，partials，holders组成

#### 参数

+   args Array 提供的参数
+   partials Array 提前传递的参数
+   holders Array 提前传入参数中的占位符索引数组
+   isCurried boolean 可选 是否柯里化

#### 返回

**Array**

返回组合参数的新数组。

#### 示例

**示例1**

```js
console.log(composeArgs(['hi', 'am', '!'], ['__lodash_placeholder__', 'I', '__lodash_placeholder__', 'Difa'], [0, 2]))
//[ 'hi', 'I', 'am', 'Difa', '!' ]
```



#### 解析

```js
/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Creates an array that is the composition of partially applied arguments,
 * placeholders, and provided arguments into a single array of arguments.
 *
 * @private
 * @param {Array} args The provided arguments.
 * @param {Array} partials The arguments to prepend to those provided.
 * @param {Array} holders The `partials` placeholder indexes.
 * @params {boolean} [isCurried] Specify composing for a curried function.
 * @returns {Array} Returns the new array of composed arguments.
 */
function composeArgs(args, partials, holders, isCurried) {
  var argsIndex = -1,
      argsLength = args.length,//调用时传入的参数数量
      holdersLength = holders.length,//提前传入的参数中占位符数量
      leftIndex = -1,
      leftLength = partials.length,//提前传入的参数数量
      rangeLength = nativeMax(argsLength - holdersLength, 0),//调用时参数中不是用于替换占位符的参数的数量
      result = Array(leftLength + rangeLength),
      isUncurried = !isCurried;
  //先保存提前传入的参数
  while (++leftIndex < leftLength) {
    result[leftIndex] = partials[leftIndex];
  }
  //假如占位符有2个，调用时传了三个参数，则前两个参数用于占位符替换，最后一个参数附加到所有参数末尾
  while (++argsIndex < holdersLength) {
    if (isUncurried || argsIndex < argsLength) {
      //在占位符对应位置保存调用时参数（使用调用时参数替换占位符）
      result[holders[argsIndex]] = args[argsIndex];
    }
  }
  //调用时的非用于占位符替换的参数附加到末尾
  while (rangeLength--) {
    result[leftIndex++] = args[argsIndex++];
  }
  return result;
}

module.exports = composeArgs;

```



#### 源码

```js
/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Creates an array that is the composition of partially applied arguments,
 * placeholders, and provided arguments into a single array of arguments.
 *
 * @private
 * @param {Array} args The provided arguments.
 * @param {Array} partials The arguments to prepend to those provided.
 * @param {Array} holders The `partials` placeholder indexes.
 * @params {boolean} [isCurried] Specify composing for a curried function.
 * @returns {Array} Returns the new array of composed arguments.
 */
function composeArgs(args, partials, holders, isCurried) {
  var argsIndex = -1,
      argsLength = args.length,
      holdersLength = holders.length,
      leftIndex = -1,
      leftLength = partials.length,
      rangeLength = nativeMax(argsLength - holdersLength, 0),
      result = Array(leftLength + rangeLength),
      isUncurried = !isCurried;

  while (++leftIndex < leftLength) {
    result[leftIndex] = partials[leftIndex];
  }
  while (++argsIndex < holdersLength) {
    if (isUncurried || argsIndex < argsLength) {
      result[holders[argsIndex]] = args[argsIndex];
    }
  }
  while (rangeLength--) {
    result[leftIndex++] = args[argsIndex++];
  }
  return result;
}

module.exports = composeArgs;

```

### composeArgsRight

#### 参数

#### 返回

#### 示例

**示例1**

```js
console.log(composeArgsRight(['hi', 'am', '!'], ['__lodash_placeholder__', 'I', '__lodash_placeholder__', 'Difa'], [0, 2]))
//[ 'hi', 'am', 'I', '!', 'Difa' ]
```



#### 解析

```js
/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * This function is like `composeArgs` except that the arguments composition
 * is tailored for `_.partialRight`.
 *
 * @private
 * @param {Array} args The provided arguments.
 * @param {Array} partials The arguments to append to those provided.
 * @param {Array} holders The `partials` placeholder indexes.
 * @params {boolean} [isCurried] Specify composing for a curried function.
 * @returns {Array} Returns the new array of composed arguments.
 */
function composeArgsRight(args, partials, holders, isCurried) {
  var argsIndex = -1,
      argsLength = args.length,//调用时参数数量
      holdersIndex = -1,
      holdersLength = holders.length,//提前传入的参数中占位符数量
      rightIndex = -1,
      rightLength = partials.length,
      rangeLength = nativeMax(argsLength - holdersLength, 0),//调用时参数中不是用于替换占位符的参数的数量
      result = Array(rangeLength + rightLength),
      isUncurried = !isCurried;
  //先保存调用时参数中不是用于替换占位符的参数
  while (++argsIndex < rangeLength) {
    result[argsIndex] = args[argsIndex];
  }
  //再保存提前传入的参数
  var offset = argsIndex;
  while (++rightIndex < rightLength) {
    result[offset + rightIndex] = partials[rightIndex];
  }
  //将提前传入的参数中的占位符进行替换
  while (++holdersIndex < holdersLength) {
    if (isUncurried || argsIndex < argsLength) {
      result[offset + holders[holdersIndex]] = args[argsIndex++];
    }
  }
  return result;
}

module.exports = composeArgsRight;

```



#### 源码

```js
/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * This function is like `composeArgs` except that the arguments composition
 * is tailored for `_.partialRight`.
 *
 * @private
 * @param {Array} args The provided arguments.
 * @param {Array} partials The arguments to append to those provided.
 * @param {Array} holders The `partials` placeholder indexes.
 * @params {boolean} [isCurried] Specify composing for a curried function.
 * @returns {Array} Returns the new array of composed arguments.
 */
function composeArgsRight(args, partials, holders, isCurried) {
  var argsIndex = -1,
      argsLength = args.length,
      holdersIndex = -1,
      holdersLength = holders.length,
      rightIndex = -1,
      rightLength = partials.length,
      rangeLength = nativeMax(argsLength - holdersLength, 0),
      result = Array(rangeLength + rightLength),
      isUncurried = !isCurried;

  while (++argsIndex < rangeLength) {
    result[argsIndex] = args[argsIndex];
  }
  var offset = argsIndex;
  while (++rightIndex < rightLength) {
    result[offset + rightIndex] = partials[rightIndex];
  }
  while (++holdersIndex < holdersLength) {
    if (isUncurried || argsIndex < argsLength) {
      result[offset + holders[holdersIndex]] = args[argsIndex++];
    }
  }
  return result;
}

module.exports = composeArgsRight;

```



### copyArray

>   Copies the values of `source` to `array`.
>
>   拷贝源数组到指定数组或者空数组上

对于源数组中的元素，拷贝的是一份引用（浅拷贝）

#### 参数

+   source Array 待拷贝的源数组
+   array Array 可选 指定数组 默认array=[]

#### 返回

**Array**

返回拷贝后的数组array

#### 示例

示例1

```js
let a = [1, { a: 2}]
let b = copyArray(a)
b[1].a = 0
console.log(a)//[ 1, { a: 0 } ]
```

#### 源码

```js
/**
 * Copies the values of `source` to `array`.
 *
 * @private
 * @param {Array} source The array to copy values from.
 * @param {Array} [array=[]] The array to copy values to.
 * @returns {Array} Returns `array`.
 */
function copyArray(source, array) {
  var index = -1,
      length = source.length;

  array || (array = Array(length));
  while (++index < length) {
    array[index] = source[index];
  }
  return array;
}

module.exports = copyArray;

```



### copyObject

> Copies properties of source to object.
>
> 将源的属性复制到对象。

#### 参数

+ source    要复制属性的对象。
+ props    要复制的属性标识符
+  **可选** object    要将属性复制到的对象。
+  **可选** customizer    自定义复制值的函数。customizer(object[key], source[key], key, object, source)
  + object[key]    object.key的值
  +  source[key]    source.key的值
  + key    props数组中的项
  + object    要复制的属性标识符
  + source    要复制属性的对象。

#### 返回

**object**

复制后的对象

**注意：**复制后的对象是**source**的一份浅拷贝

#### 示例1

```js
let obj_1 = {
    a: 1,
    b: 2,
    c: {
        d: 3,
        e: 4,
    }
}
let obj_2 = copyObject(obj_1, ['a', 'c'])
console.log(obj_2)
//{ a: 1, c: { d: 3, e: 4 } }
obj_1.c.d = 4
console.log(obj_2)
//{ a: 1, c: { d: 4, e: 4 } }	这是一份浅拷贝
```

#### 示例2

```js
let obj_1 = {
    a: 1,
    b: 2,
    c: {
        d: 3,
        e: 4,
        f: {
            g: 2
        }
    }
}
let obj_2 = copyObject(obj_1, ['a', 'c.f', 'c[d]'])
console.log(obj_2)
//{ a: 1, 'c.f': undefined, 'c[d]': undefined }
```

**注意：**copyObject函数不支持深度路径名的属性拷贝

#### 示例3

```js
let obj_1 = {
    a: 1,
    b: 2,
    c: {
        d: 3,
        e: 4,
        f: {
            g: 2
        }
    },
    d: {
        e: 1
    }
}
let obj_3 = {
    a: 2,
    f: {
        g: 3
    },
    d: {
        e: 2,
        f: 3
    }
}
let res = copyObject(obj_1, ['a', 'c',], obj_3)
console.log(res)
/*
显然，这里的a值是以obj_1为准的，即将obj_1中出现在props数组中的值拷贝出来，与obj_3
{
  a: 1,
  f: { g: 3 },
  d: { e: 2, f: 3 },
  c: { d: 3, e: 4, f: { g: 2 } }
}
*/
let res_2 = copyObject(obj_1, ['a', 'c',], obj_3)
obj_1.c.d = 4
obj_3.f.g = 4
console.log(res_2)
/*
返回的对象中既有obj_1的子对象引用，也有obj_3的子对象引用
{
  a: 1,
  b: 3,
  f: { g: 4 },
  d: { e: 2, f: 3 },
  c: { d: 4, e: 4, f: { g: 2 } }
}
*/
```



#### 源码

```js
/**
       * Copies properties of `source` to `object`.
       *
       * @private
       * @param {Object} source The object to copy properties from.
       * @param {Array} props The property identifiers to copy.
       * @param {Object} [object={}] The object to copy properties to.
       * @param {Function} [customizer] The function to customize copied values.
       * @returns {Object} Returns `object`.
       */
function copyObject(source, props, object, customizer) {
    //如果object参数为空则是拷贝到空对象中
    var isNew = !object;
    object || (object = {});

    var index = -1,
        length = props.length;
	//依次取props中的值在source中做键
    while (++index < length) {
        var key = props[index];

        //如果有自定义的复制方法则调用自定义的复制方法获取copy值
        var newValue = customizer
            ? customizer(object[key], source[key], key, object, source)
            : undefined;

        if (newValue === undefined) {
            newValue = source[key];
        }
        if (isNew) {
            baseAssignValue(object, key, newValue);
        } else {
            assignValue(object, key, newValue);
        }
    }
    return object;
}
```

### createAggregator

>   Creates a function like `_.groupBy`.
>
>   创建类似groupBy,countBy的聚合函数

#### 参数

+   setter Function 累加器对象的赋值函数
+   initializer function 可选 累加器对象的初始化函数

#### 返回

**Function**

返回一个新的聚合函数

#### 解析

```js
var arrayAggregator = require('./_arrayAggregator'),
    baseAggregator = require('./_baseAggregator'),
    baseIteratee = require('./_baseIteratee'),
    isArray = require('./isArray');

/**
 * Creates a function like `_.groupBy`.
 *
 * @private
 * @param {Function} setter The function to set accumulator values.
 * @param {Function} [initializer] The accumulator object initializer.
 * @returns {Function} Returns the new aggregator function.
 */
function createAggregator(setter, initializer) {
  return function(collection, iteratee) {
    //如果是数组，使用arrayAggregator统计，否则使用baseAggregator
    var func = isArray(collection) ? arrayAggregator : baseAggregator,
        accumulator = initializer ? initializer() : {};
	//这里返回统计结果，累加器对象
    return func(collection, setter, baseIteratee(iteratee, 2), accumulator);
  };
}

module.exports = createAggregator;

```



#### 源码

源码中涉及的函数

+   [isArray](#isArray检查数组)
+   [baseIteratee](#baseIteratee)
+   [baseAggregator](#baseAggregator)
+   [arrayAggregator](#arrayAggregator)

```js
var arrayAggregator = require('./_arrayAggregator'),
    baseAggregator = require('./_baseAggregator'),
    baseIteratee = require('./_baseIteratee'),
    isArray = require('./isArray');

/**
 * Creates a function like `_.groupBy`.
 *
 * @private
 * @param {Function} setter The function to set accumulator values.
 * @param {Function} [initializer] The accumulator object initializer.
 * @returns {Function} Returns the new aggregator function.
 */
function createAggregator(setter, initializer) {
  return function(collection, iteratee) {
    var func = isArray(collection) ? arrayAggregator : baseAggregator,
        accumulator = initializer ? initializer() : {};

    return func(collection, setter, baseIteratee(iteratee, 2), accumulator);
  };
}

module.exports = createAggregator;

```



### createBaseEach

>    Creates a `baseEach` or `baseEachRight` function.
>
>   创建一个baseEach或者baseEachRight函数

#### 参数

+   eachFunc Function 遍历集合的函数
+   fromRight boolean 可选 是否从右向左遍历

#### 返回

**Function**

返回新的遍历函数

#### 解析

```js
var isArrayLike = require('./isArrayLike');

/**
 * Creates a `baseEach` or `baseEachRight` function.
 *
 * @private
 * @param {Function} eachFunc The function to iterate over a collection.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {Function} Returns the new base function.
 */
function createBaseEach(eachFunc, fromRight) {
  return function(collection, iteratee) {
    if (collection == null) {
      return collection;
    }
    //如果集合不是类数组，是个其它对象，使用eachFunc遍历并返回遍历结果
    if (!isArrayLike(collection)) {
      return eachFunc(collection, iteratee);
    }
    //如果是个数组
    var length = collection.length,
        index = fromRight ? length : -1,
        //将collection转为对象,针对字符串
        iterable = Object(collection);
	//根据fromRight选择方向进行遍历 如果迭代器返回false,则结束遍历
    while ((fromRight ? index-- : ++index < length)) {
      if (iteratee(iterable[index], index, iterable) === false) {
        break;
      }
    }
    //返回传入的集合
    return collection;
  };
}

module.exports = createBaseEach;

```



#### 源码

源码中涉及的函数

+   [isArrayLike](#isArrayLike检查类数组)

```js
var isArrayLike = require('./isArrayLike');

/**
 * Creates a `baseEach` or `baseEachRight` function.
 *
 * @private
 * @param {Function} eachFunc The function to iterate over a collection.
 * @param {boolean} [fromRight] Specify iterating from right to left.
 * @returns {Function} Returns the new base function.
 */
function createBaseEach(eachFunc, fromRight) {
  return function(collection, iteratee) {
    if (collection == null) {
      return collection;
    }
    if (!isArrayLike(collection)) {
      return eachFunc(collection, iteratee);
    }
    var length = collection.length,
        index = fromRight ? length : -1,
        iterable = Object(collection);

    while ((fromRight ? index-- : ++index < length)) {
      if (iteratee(iterable[index], index, iterable) === false) {
        break;
      }
    }
    return collection;
  };
}

module.exports = createBaseEach;

```

### createBind

>   Creates a function that wraps `func` to invoke it with the optional `this` binding of `thisArg`.
>
>    创建一个包装func的方法，调用这个方法可以使用可选的this对象.

#### 参数

+   func Function 待包装函数
+   bitmask number 位掩码标志

#### 返回

**Function**

返回新的包装函数

#### 解析

```js
var createCtor = require('./_createCtor'),//创建一个可以创建函数实例的方法
    root = require('./_root');//根对象，node环境下为global,浏览器环境下为window

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1;

/**
 * Creates a function that wraps `func` to invoke it with the optional `this`
 * binding of `thisArg`.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createBind(func, bitmask, thisArg) {
  var isBind = bitmask & WRAP_BIND_FLAG,
      Ctor = createCtor(func);

  function wrapper() {
    var fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;//如果是根对象调用，使用func，
    //否则使用Ctor
    return fn.apply(isBind ? thisArg : this, arguments);
  }
  return wrapper;
}

module.exports = createBind;

```



#### 源码

```js
var createCtor = require('./_createCtor'),
    root = require('./_root');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1;

/**
 * Creates a function that wraps `func` to invoke it with the optional `this`
 * binding of `thisArg`.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createBind(func, bitmask, thisArg) {
  var isBind = bitmask & WRAP_BIND_FLAG,
      Ctor = createCtor(func);

  function wrapper() {
    var fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;
    return fn.apply(isBind ? thisArg : this, arguments);
  }
  return wrapper;
}

module.exports = createBind;

```

### createCurry

>   Creates a function that wraps `func` to enable currying.
>
>   创建一个对' func '进行包装的函数以启用柯里化

#### 参数

+   func Function 待包装函数
+   bitmask number 位掩码标识
+   arity number 需要提供给 `func` 的参数数量

#### 返回

**Function**

返回新的包装函数

#### 解析

```js
var apply = require('./_apply'),
    createCtor = require('./_createCtor'),//创建一个可以创建函数实例的方法
    createHybrid = require('./_createHybrid'),//创建混合方法
    createRecurry = require('./_createRecurry'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders'),
    root = require('./_root');

/**
 * Creates a function that wraps `func` to enable currying.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {number} arity The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createCurry(func, bitmask, arity) {
  var Ctor = createCtor(func);//Ctor可以创建函数实例

  function wrapper() {
    var length = arguments.length,//调用时参数数量
        args = Array(length),
        index = length,
        placeholder = getHolder(wrapper);//获取占位符
	//参数保存到args中
    while (index--) {
      args[index] = arguments[index];
    }
    //如果参数数量小于3并且第一个和最后一个参数不是占位符，则占位符占位符索引数组为[]，否则执行replaceHolders函数
    //获取占位符索引数组。
    var holders = (length < 3 && args[0] !== placeholder && args[length - 1] !== placeholder)
      ? []
      : replaceHolders(args, placeholder);
	//实际有效参数数量（不含占位符）
    length -= holders.length;
    //如果实际参数数量小于限制，需要进行参数累计，也就是需要函数处理后续的参数
    if (length < arity) {
      return createRecurry(
        func, bitmask, createHybrid, wrapper.placeholder, undefined,
        args, holders, undefined, undefined, arity - length);
    }
    //参数数量满足arity，调用函数并返回结果
    var fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;
    return apply(fn, this, args);
  }
  return wrapper;
}

module.exports = createCurry;

```



#### 源码

源码中涉及到的函数

+   [createRecurry](#createRecurry)
+   [createHybrid](#createHybrid)

```js
var apply = require('./_apply'),
    createCtor = require('./_createCtor'),
    createHybrid = require('./_createHybrid'),
    createRecurry = require('./_createRecurry'),
    getHolder = require('./_getHolder'),
    replaceHolders = require('./_replaceHolders'),
    root = require('./_root');

/**
 * Creates a function that wraps `func` to enable currying.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {number} arity The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createCurry(func, bitmask, arity) {
  var Ctor = createCtor(func);

  function wrapper() {
    var length = arguments.length,
        args = Array(length),
        index = length,
        placeholder = getHolder(wrapper);

    while (index--) {
      args[index] = arguments[index];
    }
    var holders = (length < 3 && args[0] !== placeholder && args[length - 1] !== placeholder)
      ? []
      : replaceHolders(args, placeholder);

    length -= holders.length;
    if (length < arity) {
      return createRecurry(
        func, bitmask, createHybrid, wrapper.placeholder, undefined,
        args, holders, undefined, undefined, arity - length);
    }
    var fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;
    return apply(fn, this, args);
  }
  return wrapper;
}

module.exports = createCurry;

```



### createFind

>   Creates a `_.find` or `_.findLast` function.
>
>   创建一个find或者findLast方法

#### 参数

+   findIndexFunc Function 查询集合索引的方法，如findIndex,findLastIndex 目前lodash中createFind的参数就只有前面两种

#### 返回

**Function**

返回一个方法，这个方法可以对集合进行查询，返回集合中匹配的元素

#### 解析

```js
var baseIteratee = require('./_baseIteratee'),
    isArrayLike = require('./isArrayLike'),
    keys = require('./keys');

/**
 * Creates a `_.find` or `_.findLast` function.
 *
 * @private
 * @param {Function} findIndexFunc The function to find the collection index.
 * @returns {Function} Returns the new find function.
 */
function createFind(findIndexFunc) {
  return function(collection, predicate, fromIndex) {
    //使用collection为基础创建一个对象
    var iterable = Object(collection);
    //如果不是类数组，（即为普通对象）
    if (!isArrayLike(collection)) {
      //生成迭代器函数
      var iteratee = baseIteratee(predicate, 3);
      //collection重新赋值为它的键列表
      collection = keys(collection);
      //重写判断，这个判断方法调用上面的迭代器，并返回迭代器的判断结果
      predicate = function(key) { return iteratee(iterable[key], key, iterable); };
    }
    //通过findIndex|findLastIndex获取匹配元素的索引
    var index = findIndexFunc(collection, predicate, fromIndex);
    //如果index<0，说明不存在匹配的元素，返回undefined
    //如果index>-1,如果iteratee有值，即!isArrayLike(collection)===true，返回iterable[collection[index]]
    //collection此时为传入的对象的键列表，collection[index]为某个键字符串，否则返回iterable[index]
    return index > -1 ? iterable[iteratee ? collection[index] : index] : undefined;
  };
}

module.exports = createFind;

```



#### 源码

```js
var baseIteratee = require('./_baseIteratee'),
    isArrayLike = require('./isArrayLike'),
    keys = require('./keys');

/**
 * Creates a `_.find` or `_.findLast` function.
 *
 * @private
 * @param {Function} findIndexFunc The function to find the collection index.
 * @returns {Function} Returns the new find function.
 */
function createFind(findIndexFunc) {
  return function(collection, predicate, fromIndex) {
    var iterable = Object(collection);
    if (!isArrayLike(collection)) {
      var iteratee = baseIteratee(predicate, 3);
      collection = keys(collection);
      predicate = function(key) { return iteratee(iterable[key], key, iterable); };
    }
    var index = findIndexFunc(collection, predicate, fromIndex);
    return index > -1 ? iterable[iteratee ? collection[index] : index] : undefined;
  };
}

module.exports = createFind;

```

### createHybrid

**<font color='red'>核心</font>**

>    Creates a function that wraps `func` to invoke it with optional `this` binding of `thisArg`, partial application, and currying.
>
>   创建一个封装了func的函数，使用可选的thisArg , 提前传入的参数和柯里化。

#### 参数

+ func Function|string 需要包装的函数
+ bitmask number 位掩码标识
+ thisArg any 可选 func的this对象
+ partials Array 可选 附加在新函数前面的实参 函数包装时传参
+ holders Array 可选 占位符的索引
+ partialsRight Array 可选 附加到提供给新函数的参数后面的参数。 （占位符参数）
+ holdersRight Array  可选 占位符的索引
+ argPos Array 可选 新函数的参数位置
+ ary  number 可选 可用参数
+ arity number  可选 可用参数数量

#### 返回

**Function**

返回被包装的函数

#### 解析

```js
var composeArgs = require('./_composeArgs'),
    composeArgsRight = require('./_composeArgsRight'),
    countHolders = require('./_countHolders'),
    createCtor = require('./_createCtor'),//创建一个可以创建函数实例的方法
    createRecurry = require('./_createRecurry'),
    getHolder = require('./_getHolder'),
    reorder = require('./_reorder'),
    replaceHolders = require('./_replaceHolders'),
    root = require('./_root');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_FLAG = 8,
    WRAP_CURRY_RIGHT_FLAG = 16,
    WRAP_ARY_FLAG = 128,
    WRAP_FLIP_FLAG = 512;

/**
 * Creates a function that wraps `func` to invoke it with optional `this`
 * binding of `thisArg`, partial application, and currying.
 *
 * @private
 * @param {Function|string} func The function or method name to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to prepend to those provided to
 *  the new function.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [partialsRight] The arguments to append to those provided
 *  to the new function.
 * @param {Array} [holdersRight] The `partialsRight` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createHybrid(func, bitmask, thisArg, partials, holders, partialsRight, holdersRight, argPos, ary, arity) {
  //判断函数类别
  var isAry = bitmask & WRAP_ARY_FLAG,
      isBind = bitmask & WRAP_BIND_FLAG,
      isBindKey = bitmask & WRAP_BIND_KEY_FLAG,
      isCurried = bitmask & (WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG),
      isFlip = bitmask & WRAP_FLIP_FLAG,
      Ctor = isBindKey ? undefined : createCtor(func);
  //调用包装后的函数实际上是执行wrapper()
  function wrapper() {
    var length = arguments.length,//包装函数调用时的参数数量
        args = Array(length),
        index = length;
	//遍历参数数组，使用args保存所有参数，因为函数包装调用的是createHybrid，说明包装时传递的参数中存在占位符，
    //所以倒序保存参数
    while (index--) {
      args[index] = arguments[index];
    }
    if (isCurried) {
      //获取占位符与占位符数量
      var placeholder = getHolder(wrapper),
          holdersCount = countHolders(args, placeholder);
    }
	//参数组合，partials是函数进行包装时传入的参数
    if (partials) {
      args = composeArgs(args, partials, holders, isCurried);
    }
    //从右向左组合参数，partialsRight是函数进行包装时传入的参数
    if (partialsRight) {
      args = composeArgsRight(args, partialsRight, holdersRight, isCurried);
    }
    //调用时参数数量减去调用时占位符数量，为实际有效参数数量 这里主要是柯里化在使用
    length -= holdersCount;
    if (isCurried && length < arity) {
      //获取新的占位符索引数组
      var newHolders = replaceHolders(args, placeholder);
      //返回函数引用用于处理后续参数（createRecurry会调用createHybrid进行返回，这里的循环调用是个经典，整体的调用顺序
      //大概是这样curry-》createWrap-》createCurry得到wrapper 
      //当包装函数wrapper调用时-》（参数数量不满足条件下）createRecurry-》createHybrid得到wrapper）
      //当包装函数wrapper调用时-》（参数数量满足条件下）-》fn.apply(thisBinding, args)得到结果
      return createRecurry(
        func, bitmask, createHybrid, wrapper.placeholder, thisArg,
        args, newHolders, argPos, ary, arity - length
      );
    }
    var thisBinding = isBind ? thisArg : this,
        fn = isBindKey ? thisBinding[func] : func;//获取对象的方法
	//length重置为参数长度
    length = args.length;
    if (argPos) {//如果有指定参数位置，则此处调用reorder对参数进行重组，如rearg函数
      args = reorder(args, argPos);
    } else if (isFlip && length > 1) {
      args.reverse();
    }
    //如果被包装函数是 WRAP_ARY_FLAG 类型并且限制参数数量合法，则缩减参数到限制的数量
    //这里应用的典型是 ary函数（创建一个调用func的函数。调用func时最多接受 n个参数，忽略多出的参数）
    if (isAry && ary < length) {
      args.length = ary;
    }
    if (this && this !== root && this instanceof wrapper) {
      fn = Ctor || createCtor(fn);
    }
    return fn.apply(thisBinding, args);//调用func，并且传入的参数个数为ary个
  }
  //返回包装壳函数引用
  return wrapper;
}

module.exports = createHybrid;

```



#### 源码

源码中涉及到的函数

+   [composeArgsRight](#composeArgsRight)

```js
var composeArgs = require('./_composeArgs'),
    composeArgsRight = require('./_composeArgsRight'),
    countHolders = require('./_countHolders'),
    createCtor = require('./_createCtor'),
    createRecurry = require('./_createRecurry'),
    getHolder = require('./_getHolder'),
    reorder = require('./_reorder'),
    replaceHolders = require('./_replaceHolders'),
    root = require('./_root');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_FLAG = 8,
    WRAP_CURRY_RIGHT_FLAG = 16,
    WRAP_ARY_FLAG = 128,
    WRAP_FLIP_FLAG = 512;

/**
 * Creates a function that wraps `func` to invoke it with optional `this`
 * binding of `thisArg`, partial application, and currying.
 *
 * @private
 * @param {Function|string} func The function or method name to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to prepend to those provided to
 *  the new function.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [partialsRight] The arguments to append to those provided
 *  to the new function.
 * @param {Array} [holdersRight] The `partialsRight` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createHybrid(func, bitmask, thisArg, partials, holders, partialsRight, holdersRight, argPos, ary, arity) {
  var isAry = bitmask & WRAP_ARY_FLAG,
      isBind = bitmask & WRAP_BIND_FLAG,
      isBindKey = bitmask & WRAP_BIND_KEY_FLAG,
      isCurried = bitmask & (WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG),
      isFlip = bitmask & WRAP_FLIP_FLAG,
      Ctor = isBindKey ? undefined : createCtor(func);

  function wrapper() {
    var length = arguments.length,
        args = Array(length),
        index = length;

    while (index--) {
      args[index] = arguments[index];
    }
    if (isCurried) {
      var placeholder = getHolder(wrapper),
          holdersCount = countHolders(args, placeholder);
    }
    if (partials) {
      args = composeArgs(args, partials, holders, isCurried);
    }
    if (partialsRight) {
      args = composeArgsRight(args, partialsRight, holdersRight, isCurried);
    }
    length -= holdersCount;
    if (isCurried && length < arity) {
      var newHolders = replaceHolders(args, placeholder);
      return createRecurry(
        func, bitmask, createHybrid, wrapper.placeholder, thisArg,
        args, newHolders, argPos, ary, arity - length
      );
    }
    var thisBinding = isBind ? thisArg : this,
        fn = isBindKey ? thisBinding[func] : func;

    length = args.length;
    if (argPos) {
      args = reorder(args, argPos);
    } else if (isFlip && length > 1) {
      args.reverse();
    }
    if (isAry && ary < length) {
      args.length = ary;
    }
    if (this && this !== root && this instanceof wrapper) {
      fn = Ctor || createCtor(fn);
    }
    return fn.apply(thisBinding, args);
  }
  return wrapper;
}

module.exports = createHybrid;

```



### createMathOperation

>   Creates a function that performs a mathematical operation on two values.
>
>   创建一个函数，该函数对两个值执行数学运算。

#### 参数

+   operator Function 操作函数
+   defaultValue number 可选 用于' undefined '参数的值，无参时的结果值。

#### 返回

**Function**

返回新的数学运算函数。

#### 解析

```js
var baseToNumber = require('./_baseToNumber'),
    baseToString = require('./_baseToString');

/**
 * Creates a function that performs a mathematical operation on two values.
 *
 * @private
 * @param {Function} operator The function to perform the operation.
 * @param {number} [defaultValue] The value used for `undefined` arguments.
 * @returns {Function} Returns the new mathematical operation function.
 */
function createMathOperation(operator, defaultValue) {
  return function(value, other) {
    var result;
    //如果两个参数都不传，操作函数的结果为defaultValue
    if (value === undefined && other === undefined) {
      return defaultValue;
    }
    if (value !== undefined) {
      result = value;
    }
    
    if (other !== undefined) {
      //如果value为空，返回other
      if (result === undefined) {
        return other;
      }
      //如果某个参数是string，则都转为string
      if (typeof value == 'string' || typeof other == 'string') {
        value = baseToString(value);
        other = baseToString(other);
      } else {
        value = baseToNumber(value);
        other = baseToNumber(other);
      }
      result = operator(value, other);
    }
    return result;
  };
}

module.exports = createMathOperation;

```



#### 源码

```js
var baseToNumber = require('./_baseToNumber'),
    baseToString = require('./_baseToString');

/**
 * Creates a function that performs a mathematical operation on two values.
 *
 * @private
 * @param {Function} operator The function to perform the operation.
 * @param {number} [defaultValue] The value used for `undefined` arguments.
 * @returns {Function} Returns the new mathematical operation function.
 */
function createMathOperation(operator, defaultValue) {
  return function(value, other) {
    var result;
    if (value === undefined && other === undefined) {
      return defaultValue;
    }
    if (value !== undefined) {
      result = value;
    }
    if (other !== undefined) {
      if (result === undefined) {
        return other;
      }
      if (typeof value == 'string' || typeof other == 'string') {
        value = baseToString(value);
        other = baseToString(other);
      } else {
        value = baseToNumber(value);
        other = baseToNumber(other);
      }
      result = operator(value, other);
    }
    return result;
  };
}

module.exports = createMathOperation;

```

### createPartial

>   Creates a function that wraps `func` to invoke it with the `this` binding of `thisArg` and `partials` prepended to the arguments it receives.
>
>   创建一个封装了func的函数来调用它，使用可选的this和提前传入的参数

#### 参数

+   func Function 待包装的函数
+   bitmask number 位掩码标志
+   thisArg any  func的this对象
+   partials Array  附加在新函数前面的实参 函数包装时传参

#### 返回

**Function**

返回包装函数

#### 解析

```js
var apply = require('./_apply'),
    createCtor = require('./_createCtor'),
    root = require('./_root');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1;

/**
 * Creates a function that wraps `func` to invoke it with the `this` binding
 * of `thisArg` and `partials` prepended to the arguments it receives.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {Array} partials The arguments to prepend to those provided to
 *  the new function.
 * @returns {Function} Returns the new wrapped function.
 */
function createPartial(func, bitmask, thisArg, partials) {
  var isBind = bitmask & WRAP_BIND_FLAG,
      Ctor = createCtor(func);
  //调用包装后的函数，实际上是执行wrapper()
  function wrapper() {
    var argsIndex = -1,
        argsLength = arguments.length,//arguements是实际调用时传的参数
        leftIndex = -1,
        leftLength = partials.length,//partials是函数包装时传递的参数，当函数包装以后，调用包装结果时，此参数固定不
        //变
        args = Array(leftLength + argsLength),
        fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;

    while (++leftIndex < leftLength) {
      args[leftIndex] = partials[leftIndex];
    }
    //arguments,partials合起来是所有的参数
    while (argsLength--) {
      args[leftIndex++] = arguments[++argsIndex];
    }
    return apply(fn, isBind ? thisArg : this, args);
  }
  return wrapper;
}

module.exports = createPartial;

```



#### 源码

```js
var apply = require('./_apply'),
    createCtor = require('./_createCtor'),
    root = require('./_root');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1;

/**
 * Creates a function that wraps `func` to invoke it with the `this` binding
 * of `thisArg` and `partials` prepended to the arguments it receives.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {*} thisArg The `this` binding of `func`.
 * @param {Array} partials The arguments to prepend to those provided to
 *  the new function.
 * @returns {Function} Returns the new wrapped function.
 */
function createPartial(func, bitmask, thisArg, partials) {
  var isBind = bitmask & WRAP_BIND_FLAG,
      Ctor = createCtor(func);

  function wrapper() {
    var argsIndex = -1,
        argsLength = arguments.length,
        leftIndex = -1,
        leftLength = partials.length,
        args = Array(leftLength + argsLength),
        fn = (this && this !== root && this instanceof wrapper) ? Ctor : func;

    while (++leftIndex < leftLength) {
      args[leftIndex] = partials[leftIndex];
    }
    while (argsLength--) {
      args[leftIndex++] = arguments[++argsIndex];
    }
    return apply(fn, isBind ? thisArg : this, args);
  }
  return wrapper;
}

module.exports = createPartial;

```

### createRecurry

#### 参数

+   func Function 待包装函数
+   bitmask number 位掩码标识
+   wrapFunc Function 包装函数
+   placeholder any 占位符
+   thisArg any 可选 绑定的this
+   partials Array 可选 附加的参数（已获取的参数）
+   holders Array 可选 占位符索引数组
+   argPos Array 可选 参数位置
+   ary number 可选 func函数的参数上限
+   arity number 可选 需要提供给func的参数数量

#### 返回

**Function**

返回新的包装函数

#### 解析

```js
var isLaziable = require('./_isLaziable'),
    setData = require('./_setData'),
    setWrapToString = require('./_setWrapToString');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_BOUND_FLAG = 4,
    WRAP_CURRY_FLAG = 8,
    WRAP_PARTIAL_FLAG = 32,
    WRAP_PARTIAL_RIGHT_FLAG = 64;

/**
 * Creates a function that wraps `func` to continue currying.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {Function} wrapFunc The function to create the `func` wrapper.
 * @param {*} placeholder The placeholder value.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to prepend to those provided to
 *  the new function.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createRecurry(func, bitmask, wrapFunc, placeholder, thisArg, partials, holders, argPos, ary, arity) {
  var isCurry = bitmask & WRAP_CURRY_FLAG,
      newHolders = isCurry ? holders : undefined,
      newHoldersRight = isCurry ? undefined : holders,
      newPartials = isCurry ? partials : undefined,
      //curryRight函数会在这里设置newPartialsRight=partials，并将newPartials置为undefined
      newPartialsRight = isCurry ? undefined : partials;
  //添加WRAP_PARTIAL_FLAG标识 移除WRAP_PARTIAL_RIGHT_FLAG标识
  bitmask |= (isCurry ? WRAP_PARTIAL_FLAG : WRAP_PARTIAL_RIGHT_FLAG);
  bitmask &= ~(isCurry ? WRAP_PARTIAL_RIGHT_FLAG : WRAP_PARTIAL_FLAG);

  if (!(bitmask & WRAP_CURRY_BOUND_FLAG)) {
    bitmask &= ~(WRAP_BIND_FLAG | WRAP_BIND_KEY_FLAG);
  }
  var newData = [
    func, bitmask, thisArg, newPartials, newHolders, newPartialsRight,
    newHoldersRight, argPos, ary, arity
  ];

  var result = wrapFunc.apply(undefined, newData);
  //如果缓存中存在，设置数据
  if (isLaziable(func)) {
    setData(result, newData);
  }
  result.placeholder = placeholder;
  //返回包装函数
  return setWrapToString(result, func, bitmask);
}

module.exports = createRecurry;

```



#### 源码

```js
var isLaziable = require('./_isLaziable'),
    setData = require('./_setData'),
    setWrapToString = require('./_setWrapToString');

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_BOUND_FLAG = 4,
    WRAP_CURRY_FLAG = 8,
    WRAP_PARTIAL_FLAG = 32,
    WRAP_PARTIAL_RIGHT_FLAG = 64;

/**
 * Creates a function that wraps `func` to continue currying.
 *
 * @private
 * @param {Function} func The function to wrap.
 * @param {number} bitmask The bitmask flags. See `createWrap` for more details.
 * @param {Function} wrapFunc The function to create the `func` wrapper.
 * @param {*} placeholder The placeholder value.
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to prepend to those provided to
 *  the new function.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createRecurry(func, bitmask, wrapFunc, placeholder, thisArg, partials, holders, argPos, ary, arity) {
  var isCurry = bitmask & WRAP_CURRY_FLAG,
      newHolders = isCurry ? holders : undefined,
      newHoldersRight = isCurry ? undefined : holders,
      newPartials = isCurry ? partials : undefined,
      newPartialsRight = isCurry ? undefined : partials;

  bitmask |= (isCurry ? WRAP_PARTIAL_FLAG : WRAP_PARTIAL_RIGHT_FLAG);
  bitmask &= ~(isCurry ? WRAP_PARTIAL_RIGHT_FLAG : WRAP_PARTIAL_FLAG);

  if (!(bitmask & WRAP_CURRY_BOUND_FLAG)) {
    bitmask &= ~(WRAP_BIND_FLAG | WRAP_BIND_KEY_FLAG);
  }
  var newData = [
    func, bitmask, thisArg, newPartials, newHolders, newPartialsRight,
    newHoldersRight, argPos, ary, arity
  ];

  var result = wrapFunc.apply(undefined, newData);
  if (isLaziable(func)) {
    setData(result, newData);
  }
  result.placeholder = placeholder;
  return setWrapToString(result, func, bitmask);
}

module.exports = createRecurry;

```



### createRound

>   Built-in method references for those with the same name as other `lodash` methods.
>
>   与其他' lodash '方法同名的内置方法引用。

#### 参数

+   methodName string 方法名

#### 返回

**Function** 

返回新的约进函数

#### 解析

```js
var root = require('./_root'),
    toInteger = require('./toInteger'),
    toNumber = require('./toNumber'),
    toString = require('./toString');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeIsFinite = root.isFinite,
    nativeMin = Math.min;

/**
 * Creates a function like `_.round`.
 *
 * @private
 * @param {string} methodName The name of the `Math` method to use when rounding.
 * @returns {Function} Returns the new round function.
 */
function createRound(methodName) {
    //获取原生函数的引用
    var func = Math[methodName];
    return function (number, precision) {
        //转数字
        number = toNumber(number);
        precision = precision == null ? 0 : nativeMin(toInteger(precision), 292);
        if (precision && nativeIsFinite(number)) {
            // Shift with exponential notation to avoid floating-point issues.
            //使用指数符号进行移位，以避免浮点数问题。
            // See [MDN](https://mdn.io/round#Examples) for more details.
            var pair = (toString(number) + 'e').split('e'),
                //先将number缩放至10的precision次方倍，再进行约进
                value = func(pair[0] + 'e' + (+pair[1] + precision));
            console.log(pair[0] + 'e' + (+pair[1] + precision))
            //6.004e2 ->600.4
            console.log('pair', pair)
            //pair [ '6.004', '' ]
            console.log('value', value)
            //value 601

            pair = (toString(value) + 'e').split('e');
            console.log('pair', pair)
            //pair [ '601', '' ]
            console.log( +(pair[0] + 'e' + (+pair[1] - precision)))
            //6.01
            //缩放回去
            return +(pair[0] + 'e' + (+pair[1] - precision));
        }
        return func(number);
    };
}
var ceil = createRound('ceil');
console.log(ceil(6.004, 2))
//6.01
module.exports = createRound;

```



#### 源码

```js
var root = require('./_root'),
    toInteger = require('./toInteger'),
    toNumber = require('./toNumber'),
    toString = require('./toString');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeIsFinite = root.isFinite,
    nativeMin = Math.min;

/**
 * Creates a function like `_.round`.
 *
 * @private
 * @param {string} methodName The name of the `Math` method to use when rounding.
 * @returns {Function} Returns the new round function.
 */
function createRound(methodName) {
  var func = Math[methodName];
  return function(number, precision) {
    number = toNumber(number);
    precision = precision == null ? 0 : nativeMin(toInteger(precision), 292);
    if (precision && nativeIsFinite(number)) {
      // Shift with exponential notation to avoid floating-point issues.
      // See [MDN](https://mdn.io/round#Examples) for more details.
      var pair = (toString(number) + 'e').split('e'),
          value = func(pair[0] + 'e' + (+pair[1] + precision));

      pair = (toString(value) + 'e').split('e');
      return +(pair[0] + 'e' + (+pair[1] - precision));
    }
    return func(number);
  };
}

module.exports = createRound;

```



### **createWrap**

**<font color='red'>核心方法</font>**

createWrap是很多方法的基础包装方法

> Creates a function that either curries or invokes `func` with optional `this` binding and partially applied arguments.
>
> 创建一个函数，该函数使用可选的' this '绑定和部分应用参数来curry或调用' func '。

#### 参数

+ func Function|string 需要包装的函数
+ bitmask number 位掩码标识
    + 以二进制位进行计算与表示，因为一个二进制位可以表示一个标志位，因此一个位掩码表示多个flag
+ thisArg any 可选 func的this对象
+ partials Array 可选 附加在新函数前面的实参
+ holders Array 可选 占位符的索引数组
+ argPos Array 可选 新函数的参数位置
+ ary  number 可选 可用参数
+ arity number  可选 可用参数数量

#### 返回

**Function**

返回被包装后的函数

#### 解析

```js
var baseSetData = require('./_baseSetData'),
    createBind = require('./_createBind'),
    createCurry = require('./_createCurry'),
    createHybrid = require('./_createHybrid'),//创建混合方法
    createPartial = require('./_createPartial'),
    getData = require('./_getData'),
    mergeData = require('./_mergeData'),
    setData = require('./_setData'),
    setWrapToString = require('./_setWrapToString'),
    toInteger = require('./toInteger');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_FLAG = 8,
    WRAP_CURRY_RIGHT_FLAG = 16,
    WRAP_PARTIAL_FLAG = 32,//是否有附加参数标记（包装时传参）
    WRAP_PARTIAL_RIGHT_FLAG = 64;//是否有附加到提供给新函数的参数后面的参数的标记（包装后调用传的参数）

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Creates a function that either curries or invokes `func` with optional
 * `this` binding and partially applied arguments.
 *
 * @private
 * @param {Function|string} func The function or method name to wrap.
 * @param {number} bitmask The bitmask flags.
 *    1 - `_.bind`
 *    2 - `_.bindKey`
 *    4 - `_.curry` or `_.curryRight` of a bound function
 *    8 - `_.curry`
 *   16 - `_.curryRight`
 *   32 - `_.partial`
 *   64 - `_.partialRight`
 *  128 - `_.rearg`
 *  256 - `_.ary`
 *  512 - `_.flip`
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to be partially applied.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createWrap(func, bitmask, thisArg, partials, holders, argPos, ary, arity) {
  //是否是bindKey方法 	&=》按位与 2的任意次方之间的按位与操作都是0
  var isBindKey = bitmask & WRAP_BIND_KEY_FLAG;
  //如果不是bindKey方法并且func不是函数，则抛出错误
  if (!isBindKey && typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  //附加参数长度
  var length = partials ? partials.length : 0;
  //如果无附加参数
  if (!length) {
    //清除WRAP_PARTIAL_FLAG和WRAP_PARTIAL_RIGHT_FLAG标记位
    bitmask &= ~(WRAP_PARTIAL_FLAG | WRAP_PARTIAL_RIGHT_FLAG);
    //无附加参数，则partials与holders置为undefined
    partials = holders = undefined;
  }
  ary = ary === undefined ? ary : nativeMax(toInteger(ary), 0);
  //arity处理
  arity = arity === undefined ? arity : toInteger(arity);
  length -= holders ? holders.length : 0;//如果有占位符，实际参数长度为length减去占位符数量

  if (bitmask & WRAP_PARTIAL_RIGHT_FLAG) {
    var partialsRight = partials,
        holdersRight = holders;

    partials = holders = undefined;
  }
  //如果不是bindKey，获取func的源数据
  var data = isBindKey ? undefined : getData(func);
  //参数存入newData中
  var newData = [
    func, bitmask, thisArg, partials, holders, partialsRight, holdersRight,
    argPos, ary, arity
  ];

  if (data) {
    mergeData(newData, data);
  }
  func = newData[0];
  bitmask = newData[1];
  thisArg = newData[2];
  partials = newData[3];
  holders = newData[4];
  arity = newData[9] = newData[9] === undefined
    ? (isBindKey ? 0 : func.length)
    : nativeMax(newData[9] - length, 0);
  //如果arity为0，清除WRAP_CURRY_FLAG 和 WRAP_CURRY_RIGHT_FLAG 标识位
  if (!arity && bitmask & (WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG)) {
    bitmask &= ~(WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG);
  }
  //如果是 WRAP_BIND_FLAG 类函数，并且无任何附加参数（如果无附加参数，前面if分支会重置WRAP_PARTIAL_FLAG和
  //WRAP_PARTIAL_RIGHT_FLAG标记位），直接调用createBind进行包装
  if (!bitmask || bitmask == WRAP_BIND_FLAG) {
    var result = createBind(func, bitmask, thisArg);
  } else if (bitmask == WRAP_CURRY_FLAG || bitmask == WRAP_CURRY_RIGHT_FLAG) {
    //函数柯里化分支
    result = createCurry(func, bitmask, arity);
  } else if ((bitmask == WRAP_PARTIAL_FLAG || bitmask == (WRAP_BIND_FLAG | WRAP_PARTIAL_FLAG)) && !holders.length) {
    //如果存在附加参数，并且没有占位符（说明函数包装时没有_占位），调用createPartial进行包装
    result = createPartial(func, bitmask, thisArg, partials);
  } else {
    //不满足上面分支，调用createHybrid，传入所有参数（函数包装时存在占位符会走此分支）
    result = createHybrid.apply(undefined, newData);
  }
  //设置源数据
  var setter = data ? baseSetData : setData;
  //给包装之后的方法添加数据（用于优化），添加toStirng方法，并返回func
  return setWrapToString(setter(result, newData), func, bitmask);
}

module.exports = createWrap;

```



#### 源码

源码中涉及到的函数

+   [createHybrid](#createHybrid)
+   [createCurry](#createCurry),
+   [createPartial](#createPartial)

```js
var baseSetData = require('./_baseSetData'),
    createBind = require('./_createBind'),
    createCurry = require('./_createCurry'),
    createHybrid = require('./_createHybrid'),
    createPartial = require('./_createPartial'),
    getData = require('./_getData'),
    mergeData = require('./_mergeData'),
    setData = require('./_setData'),
    setWrapToString = require('./_setWrapToString'),
    toInteger = require('./toInteger');

/** Error message constants. */
var FUNC_ERROR_TEXT = 'Expected a function';

/** Used to compose bitmasks for function metadata. */
var WRAP_BIND_FLAG = 1,
    WRAP_BIND_KEY_FLAG = 2,
    WRAP_CURRY_FLAG = 8,
    WRAP_CURRY_RIGHT_FLAG = 16,
    WRAP_PARTIAL_FLAG = 32,
    WRAP_PARTIAL_RIGHT_FLAG = 64;

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMax = Math.max;

/**
 * Creates a function that either curries or invokes `func` with optional
 * `this` binding and partially applied arguments.
 *
 * @private
 * @param {Function|string} func The function or method name to wrap.
 * @param {number} bitmask The bitmask flags.
 *    1 - `_.bind`
 *    2 - `_.bindKey`
 *    4 - `_.curry` or `_.curryRight` of a bound function
 *    8 - `_.curry`
 *   16 - `_.curryRight`
 *   32 - `_.partial`
 *   64 - `_.partialRight`
 *  128 - `_.rearg`
 *  256 - `_.ary`
 *  512 - `_.flip`
 * @param {*} [thisArg] The `this` binding of `func`.
 * @param {Array} [partials] The arguments to be partially applied.
 * @param {Array} [holders] The `partials` placeholder indexes.
 * @param {Array} [argPos] The argument positions of the new function.
 * @param {number} [ary] The arity cap of `func`.
 * @param {number} [arity] The arity of `func`.
 * @returns {Function} Returns the new wrapped function.
 */
function createWrap(func, bitmask, thisArg, partials, holders, argPos, ary, arity) {
  var isBindKey = bitmask & WRAP_BIND_KEY_FLAG;
  if (!isBindKey && typeof func != 'function') {
    throw new TypeError(FUNC_ERROR_TEXT);
  }
  var length = partials ? partials.length : 0;
  if (!length) {
    bitmask &= ~(WRAP_PARTIAL_FLAG | WRAP_PARTIAL_RIGHT_FLAG);
    partials = holders = undefined;
  }
  ary = ary === undefined ? ary : nativeMax(toInteger(ary), 0);
  arity = arity === undefined ? arity : toInteger(arity);
  length -= holders ? holders.length : 0;

  if (bitmask & WRAP_PARTIAL_RIGHT_FLAG) {
    var partialsRight = partials,
        holdersRight = holders;

    partials = holders = undefined;
  }
  var data = isBindKey ? undefined : getData(func);

  var newData = [
    func, bitmask, thisArg, partials, holders, partialsRight, holdersRight,
    argPos, ary, arity
  ];

  if (data) {
    mergeData(newData, data);
  }
  func = newData[0];
  bitmask = newData[1];
  thisArg = newData[2];
  partials = newData[3];
  holders = newData[4];
  arity = newData[9] = newData[9] === undefined
    ? (isBindKey ? 0 : func.length)
    : nativeMax(newData[9] - length, 0);

  if (!arity && bitmask & (WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG)) {
    bitmask &= ~(WRAP_CURRY_FLAG | WRAP_CURRY_RIGHT_FLAG);
  }
  if (!bitmask || bitmask == WRAP_BIND_FLAG) {
    var result = createBind(func, bitmask, thisArg);
  } else if (bitmask == WRAP_CURRY_FLAG || bitmask == WRAP_CURRY_RIGHT_FLAG) {
    result = createCurry(func, bitmask, arity);
  } else if ((bitmask == WRAP_PARTIAL_FLAG || bitmask == (WRAP_BIND_FLAG | WRAP_PARTIAL_FLAG)) && !holders.length) {
    result = createPartial(func, bitmask, thisArg, partials);
  } else {
    result = createHybrid.apply(undefined, newData);
  }
  var setter = data ? baseSetData : setData;
  return setWrapToString(setter(result, newData), func, bitmask);
}

module.exports = createWrap;

```

### deburrLetter

>   Used by `_.deburr` to convert Latin-1 Supplement and Latin Extended-A letters to basic Latin letters.
>
>   用于_.deburr将拉丁-1补充字母和拉丁扩展字母转换为基本拉丁字母。

#### 源码

```js
var basePropertyOf = require('./_basePropertyOf');

/** Used to map Latin Unicode letters to basic Latin letters. */
var deburredLetters = {
  // Latin-1 Supplement block.
  '\xc0': 'A',  '\xc1': 'A', '\xc2': 'A', '\xc3': 'A', '\xc4': 'A', '\xc5': 'A',
  '\xe0': 'a',  '\xe1': 'a', '\xe2': 'a', '\xe3': 'a', '\xe4': 'a', '\xe5': 'a',
  '\xc7': 'C',  '\xe7': 'c',
  '\xd0': 'D',  '\xf0': 'd',
  '\xc8': 'E',  '\xc9': 'E', '\xca': 'E', '\xcb': 'E',
  '\xe8': 'e',  '\xe9': 'e', '\xea': 'e', '\xeb': 'e',
  '\xcc': 'I',  '\xcd': 'I', '\xce': 'I', '\xcf': 'I',
  '\xec': 'i',  '\xed': 'i', '\xee': 'i', '\xef': 'i',
  '\xd1': 'N',  '\xf1': 'n',
  '\xd2': 'O',  '\xd3': 'O', '\xd4': 'O', '\xd5': 'O', '\xd6': 'O', '\xd8': 'O',
  '\xf2': 'o',  '\xf3': 'o', '\xf4': 'o', '\xf5': 'o', '\xf6': 'o', '\xf8': 'o',
  '\xd9': 'U',  '\xda': 'U', '\xdb': 'U', '\xdc': 'U',
  '\xf9': 'u',  '\xfa': 'u', '\xfb': 'u', '\xfc': 'u',
  '\xdd': 'Y',  '\xfd': 'y', '\xff': 'y',
  '\xc6': 'Ae', '\xe6': 'ae',
  '\xde': 'Th', '\xfe': 'th',
  '\xdf': 'ss',
  // Latin Extended-A block.
  '\u0100': 'A',  '\u0102': 'A', '\u0104': 'A',
  '\u0101': 'a',  '\u0103': 'a', '\u0105': 'a',
  '\u0106': 'C',  '\u0108': 'C', '\u010a': 'C', '\u010c': 'C',
  '\u0107': 'c',  '\u0109': 'c', '\u010b': 'c', '\u010d': 'c',
  '\u010e': 'D',  '\u0110': 'D', '\u010f': 'd', '\u0111': 'd',
  '\u0112': 'E',  '\u0114': 'E', '\u0116': 'E', '\u0118': 'E', '\u011a': 'E',
  '\u0113': 'e',  '\u0115': 'e', '\u0117': 'e', '\u0119': 'e', '\u011b': 'e',
  '\u011c': 'G',  '\u011e': 'G', '\u0120': 'G', '\u0122': 'G',
  '\u011d': 'g',  '\u011f': 'g', '\u0121': 'g', '\u0123': 'g',
  '\u0124': 'H',  '\u0126': 'H', '\u0125': 'h', '\u0127': 'h',
  '\u0128': 'I',  '\u012a': 'I', '\u012c': 'I', '\u012e': 'I', '\u0130': 'I',
  '\u0129': 'i',  '\u012b': 'i', '\u012d': 'i', '\u012f': 'i', '\u0131': 'i',
  '\u0134': 'J',  '\u0135': 'j',
  '\u0136': 'K',  '\u0137': 'k', '\u0138': 'k',
  '\u0139': 'L',  '\u013b': 'L', '\u013d': 'L', '\u013f': 'L', '\u0141': 'L',
  '\u013a': 'l',  '\u013c': 'l', '\u013e': 'l', '\u0140': 'l', '\u0142': 'l',
  '\u0143': 'N',  '\u0145': 'N', '\u0147': 'N', '\u014a': 'N',
  '\u0144': 'n',  '\u0146': 'n', '\u0148': 'n', '\u014b': 'n',
  '\u014c': 'O',  '\u014e': 'O', '\u0150': 'O',
  '\u014d': 'o',  '\u014f': 'o', '\u0151': 'o',
  '\u0154': 'R',  '\u0156': 'R', '\u0158': 'R',
  '\u0155': 'r',  '\u0157': 'r', '\u0159': 'r',
  '\u015a': 'S',  '\u015c': 'S', '\u015e': 'S', '\u0160': 'S',
  '\u015b': 's',  '\u015d': 's', '\u015f': 's', '\u0161': 's',
  '\u0162': 'T',  '\u0164': 'T', '\u0166': 'T',
  '\u0163': 't',  '\u0165': 't', '\u0167': 't',
  '\u0168': 'U',  '\u016a': 'U', '\u016c': 'U', '\u016e': 'U', '\u0170': 'U', '\u0172': 'U',
  '\u0169': 'u',  '\u016b': 'u', '\u016d': 'u', '\u016f': 'u', '\u0171': 'u', '\u0173': 'u',
  '\u0174': 'W',  '\u0175': 'w',
  '\u0176': 'Y',  '\u0177': 'y', '\u0178': 'Y',
  '\u0179': 'Z',  '\u017b': 'Z', '\u017d': 'Z',
  '\u017a': 'z',  '\u017c': 'z', '\u017e': 'z',
  '\u0132': 'IJ', '\u0133': 'ij',
  '\u0152': 'Oe', '\u0153': 'oe',
  '\u0149': "'n", '\u017f': 's'
};

/**
 * Used by `_.deburr` to convert Latin-1 Supplement and Latin Extended-A
 * letters to basic Latin letters.
 *
 * @private
 * @param {string} letter The matched letter to deburr.
 * @returns {string} Returns the deburred letter.
 */
var deburrLetter = basePropertyOf(deburredLetters);

module.exports = deburrLetter;

```



### defineProperty

#### 源码

源码中涉及的函数

+   [getNative](#getNative)

```js
var getNative = require('./_getNative');

var defineProperty = (function() {
  try {
    var func = getNative(Object, 'defineProperty');
    func({}, '', {});
    return func;
  } catch (e) {}
}());

module.exports = defineProperty;

```



### flatRest

> A specialized version of `baseRest` which flattens the rest array.
>
> “baseRest”的一个专门版本，它可以使其余的阵列变平。

#### 参数

+ func	要对其应用rest参数的函数。

#### 返回

**Function**

返回包装后的函数

#### 源码

```js
/**
 * A specialized version of `baseRest` which flattens the rest array.
 *
 * @private
 * @param {Function} func The function to apply a rest parameter to.
 * @returns {Function} Returns the new function.
 */
function flatRest(func) {
    return setToString(overRest(func, undefined, flatten), func + '');
}
```



### getAllKeysIn

> Creates an array of own and inherited enumerable property names and symbols of `object`.
>
> 创建一个包含自己的和继承的可枚举属性名和' object '符号的数组。

#### 参数

+ object	待分析对象

#### 返回

**Array**

包含自己的和继承的可枚举属性名和' object '符号的数组

#### 源码

源码中涉及的方法

+ [baseGetAllKeys](#baseGetAllKeys)
+ [keysIn](#keysIn)

```js
/**
 * Creates an array of own and inherited enumerable property names and
 * symbols of `object`.
 *
 * @private
 * @param {Object} object The object to query.
 * @returns {Array} Returns the array of property names and symbols.
 */
function getAllKeysIn(object) {
    return baseGetAllKeys(object, keysIn, getSymbolsIn);
}
```

### getHolder

>   Gets the argument placeholder value for `func`.
>
>   获取' func '的参数占位符值。

#### 参数

+   func Function 待检查函数

#### 返回

**any**

返回占位符值

#### 源码

```js
/**
 * Gets the argument placeholder value for `func`.
 *
 * @private
 * @param {Function} func The function to inspect.
 * @returns {*} Returns the placeholder value.
 */
function getHolder(func) {
  var object = func;
  return object.placeholder;
}

module.exports = getHolder;

```



### getNative

>   Gets the native function at `key` of `object`.
>
>   从原生对象中获取原生方法

#### 参数

+   object Object 待查询对象
+   key string 方法名

#### 返回

**Function|undefined**

如果方法存在，返回这个方法，否则返回undefined

#### 源码

```js
var baseIsNative = require('./_baseIsNative'),
    getValue = require('./_getValue');

/**
 * Gets the native function at `key` of `object`.
 *
 * @private
 * @param {Object} object The object to query.
 * @param {string} key The key of the method to get.
 * @returns {*} Returns the function if it's native, else `undefined`.
 */
function getNative(object, key) {
  var value = getValue(object, key);
  return baseIsNative(value) ? value : undefined;
}

module.exports = getNative;

```

### getPrototype

#### 源码

```js
var overArg = require('./_overArg');

/** Built-in value references. */
var getPrototype = overArg(Object.getPrototypeOf, Object);

module.exports = getPrototype;

```

### getTag

>   Gets the `toStringTag` of `value`.
>
>   获取value的类型字符串

**<font color='red'>重要函数</font>**

#### 参数

+   value 待检测值

#### 返回

**string**

类型字符串

#### 源码

```js
var DataView = require('./_DataView'),
    Map = require('./_Map'),
    Promise = require('./_Promise'),
    Set = require('./_Set'),
    WeakMap = require('./_WeakMap'),
    baseGetTag = require('./_baseGetTag'),
    toSource = require('./_toSource');

/** `Object#toString` result references. */
var mapTag = '[object Map]',
    objectTag = '[object Object]',
    promiseTag = '[object Promise]',
    setTag = '[object Set]',
    weakMapTag = '[object WeakMap]';

var dataViewTag = '[object DataView]';

/** Used to detect maps, sets, and weakmaps. */
var dataViewCtorString = toSource(DataView),
    mapCtorString = toSource(Map),
    promiseCtorString = toSource(Promise),
    setCtorString = toSource(Set),
    weakMapCtorString = toSource(WeakMap);

/**
 * Gets the `toStringTag` of `value`.
 *
 * @private
 * @param {*} value The value to query.
 * @returns {string} Returns the `toStringTag`.
 */
var getTag = baseGetTag;

// Fallback for data views, maps, sets, and weak maps in IE 11 and promises in Node.js < 6.
if ((DataView && getTag(new DataView(new ArrayBuffer(1))) != dataViewTag) ||
    (Map && getTag(new Map) != mapTag) ||
    (Promise && getTag(Promise.resolve()) != promiseTag) ||
    (Set && getTag(new Set) != setTag) ||
    (WeakMap && getTag(new WeakMap) != weakMapTag)) {
  getTag = function(value) {
    var result = baseGetTag(value),
        Ctor = result == objectTag ? value.constructor : undefined,
        ctorString = Ctor ? toSource(Ctor) : '';

    if (ctorString) {
      switch (ctorString) {
        case dataViewCtorString: return dataViewTag;
        case mapCtorString: return mapTag;
        case promiseCtorString: return promiseTag;
        case setCtorString: return setTag;
        case weakMapCtorString: return weakMapTag;
      }
    }
    return result;
  };
}

module.exports = getTag;

```



### isIndex

>   Checks if `value` is a valid array-like index.
>
>   检查value是否可能是数组的索引

#### 参数

+   value any 待检查值
+   length number 可选 数组长度 默认length=MAX_SAFE_INTEGER

#### 返回

**boolean**

如果是索引，返回true，否则返回false

#### 示例

**示例1**

```js
console.log(isIndex(2, 3))//true
```



**示例2**

```js
console.log(isIndex(-1, 3))//false
```



**示例3**

```js
console.log(isIndex('2', 3))//true
```



**示例4**

```js
console.log(isIndex(5, 3))//false
```



#### 解析

```js
/** Used as references for various `Number` constants. */
var MAX_SAFE_INTEGER = 9007199254740991;

/** Used to detect unsigned integer values. */
var reIsUint = /^(?:0|[1-9]\d*)$/;

/**
 * Checks if `value` is a valid array-like index.
 *
 * @private
 * @param {*} value The value to check.
 * @param {number} [length=MAX_SAFE_INTEGER] The upper bounds of a valid index.
 * @returns {boolean} Returns `true` if `value` is a valid index, else `false`.
 */
function isIndex(value, length) {
    //获取value类型，初始化length
    var type = typeof value;
    length = length == null ? MAX_SAFE_INTEGER : length;
	//如果length存在并且value满足=》{value类型为数字，或者value是非symbol但满足正则的其它类型}，并且value大于
    //-1，能被1整除（是整数），且value小于数组长度length，则返回true，否则返回false
    return !!length &&
        (type == 'number' ||
            (type != 'symbol' && reIsUint.test(value))) &&
        (value > -1 && value % 1 == 0 && value < length);
}

module.exports = isIndex;
```



#### 源码

```js
/** Used as references for various `Number` constants. */
var MAX_SAFE_INTEGER = 9007199254740991;

/** Used to detect unsigned integer values. */
var reIsUint = /^(?:0|[1-9]\d*)$/;

/**
 * Checks if `value` is a valid array-like index.
 *
 * @private
 * @param {*} value The value to check.
 * @param {number} [length=MAX_SAFE_INTEGER] The upper bounds of a valid index.
 * @returns {boolean} Returns `true` if `value` is a valid index, else `false`.
 */
function isIndex(value, length) {
    var type = typeof value;
    length = length == null ? MAX_SAFE_INTEGER : length;

    return !!length &&
        (type == 'number' ||
            (type != 'symbol' && reIsUint.test(value))) &&
        (value > -1 && value % 1 == 0 && value < length);
}

module.exports = isIndex;
```



### isIterateeCall

> Checks if the given arguments are from an iteratee call.
>
> 检查给定的参数是否来自迭代器调用。

**当object[index]===value时返回true**

#### 参数

+   value any 可能的迭代值参数
+   index any 可能的迭代对象索引或键参数
+   object any 可能的迭代对象参数

#### 返回

**boolean**

如果参数来自于迭代器调用，返回true，否则返回false

#### 源码

```js
/**
* Checks if the given arguments are from an iteratee call.
*
* @private
* @param {*} value The potential iteratee value argument.
* @param {*} index The potential iteratee index or key argument.
* @param {*} object The potential iteratee object argument.
* @returns {boolean} Returns `true` if the arguments are from an iteratee call,
*  else `false`.
*/
function isIterateeCall(value, index, object) {
    if (!isObject(object)) {
        return false;
    }
    var type = typeof index;
    if (type == 'number'
        ? (isArrayLike(object) && isIndex(index, object.length))
        : (type == 'string' && index in object)
    ) {
        return eq(object[index], value);
    }
    return false;
}
```



### isKey

> Checks if `value` is a property name and not a property path.
>
> 检查' value '是否是属性名而不是属性路径。

#### 参数

+ value	需要判断的字符串
+ object  如果value在object中就直接返回true

#### 返回

**Boolean**

如果value是属性名就返回true，否则返回false

#### 源码

源码中涉及的方法

+ [isSymbol](#isSymbol检查符号)

源码中涉及的常量

+ [reIsDeepProp](#reIsDeepProp，reIsPlainProp，rePropName)
+ [reIsPlainProp](#reIsDeepProp，reIsPlainProp，rePropName)

```js
    /**
     * Checks if `value` is a property name and not a property path.
     *
     * @private
     * @param {*} value The value to check.
     * @param {Object} [object] The object to query keys on.
     * @returns {boolean} Returns `true` if `value` is a property name, else `false`.
     */
    function isKey(value, object) {
      if (isArray(value)) {
          //如果是数组，则返回false，
        return false;
      }
      var type = typeof value;
      if (type == 'number' || type == 'symbol' || type == 'boolean' ||
          value == null || isSymbol(value)) {
        return true;
      }
       //当value为非字母字符串时返回true
       //当value不是obj.a，obj[a]这类格式时，返回true
       //当object不为空且value在object中时，value是属性名
      return reIsPlainProp.test(value) || !reIsDeepProp.test(value) ||
        (object != null && value in Object(object));
    }
```

### keysIn

>  Creates an array of the own and inherited enumerable property names of `object`.\
>
> 创建一个包含自己的和继承的' object '可枚举属性名的数组。

#### 参数

+ object	需要识别的对象

#### 返回

**Array**

包含自己的和继承的' object '可枚举属性名的数组。

#### 示例1

```js
console.log(keysIn({
    a: 1,
    b: [],
    c: ''
}))//[ 'a', 'b', 'c' ]
console.log(keysIn([1, 2, 3]))//[ '0', '1', '2' ]
```



#### 源码

源码中涉及的方法

+ [isArrayLike](#isArrayLike检查类数组)

```js

/**
 * Creates an array of the own and inherited enumerable property names of `object`.
 *
 * **Note:** Non-object values are coerced to objects.
 *
 * @static
 * @memberOf _
 * @since 3.0.0
 * @category Object
 * @param {Object} object The object to query.
 * @returns {Array} Returns the array of property names.
 * @example
 *
 * function Foo() {
 *   this.a = 1;
 *   this.b = 2;
 * }
 *
 * Foo.prototype.c = 3;
 *
 * _.keysIn(new Foo);
 * // => ['a', 'b', 'c'] (iteration order is not guaranteed)
 */
function keysIn(object) {
    return isArrayLike(object) ? arrayLikeKeys(object, true) : baseKeysIn(object);
}
```



### memoizeCapped

>  A specialized version of `_.memoize` which clears the memoized function's cache when it exceeds `MAX_MEMOIZE_SIZE`.
>
> _.memoize的特殊版本。当超过' MAX_MEMOIZE_SIZE '时清除memoized函数的缓存。

**记忆函数（Memoization）**是一种用于长递归或长迭代操作性能优化的编程实践。

**记忆函数实现原理**：使用一组参数初次调用函数时，缓存参数和计算结果，当再次使用相同的参数调用该函数时，直接返回相应的缓存结果。

**注意**: 记忆化函数不能有副作用。

#### 参数

+ Function 出现递归的函数

#### 返回

Function 包装后的函数

#### 源码

源码中涉及的方法

+ [memoize](#memoize)

```js
/**
 * A specialized version of `_.memoize` which clears the memoized function's
 * cache when it exceeds `MAX_MEMOIZE_SIZE`.
 *
 * @private
 * @param {Function} func The function to have its output memoized.
 * @returns {Function} Returns the new memoized function.
 */
function memoizeCapped(func) {
    var result = memoize(func, function (key) {
        if (cache.size === MAX_MEMOIZE_SIZE) {
            cache.clear();
        }
        return key;
    });

    var cache = result.cache;
    return result;
}
```

### reorder

>   Reorder `array` according to the specified indexes where the element at the first index is assigned as the first element, the element at the second index is assigned as the second element, and so on.
>
>   根据指定的索引重新排序数组，第一个索引处的元素被赋值为第一个元素，第二个索引处的元素被赋值为第二个元素，以此类推。

#### 参数

+   array Array 参数数组
+   indexes Array 参数索引数组

#### 返回

**Array**

返回重组后的数组

#### 源码

源码中涉及到的函数

+   [copyArray](#copyArray)

```js
var copyArray = require('./_copyArray'),
    isIndex = require('./_isIndex');

/* Built-in method references for those with the same name as other `lodash` methods. */
var nativeMin = Math.min;

/**
 * Reorder `array` according to the specified indexes where the element at
 * the first index is assigned as the first element, the element at
 * the second index is assigned as the second element, and so on.
 *
 * @private
 * @param {Array} array The array to reorder.
 * @param {Array} indexes The arranged array indexes.
 * @returns {Array} Returns `array`.
 */
function reorder(array, indexes) {
  var arrLength = array.length,
      length = nativeMin(indexes.length, arrLength),
      oldArray = copyArray(array);

  while (length--) {
    //使用indexes中的值对array数组进行重组
    var index = indexes[length];
    array[length] = isIndex(index, arrLength) ? oldArray[index] : undefined;
  }
  return array;
}

module.exports = reorder;

```



### replaceHolders

>   Replaces all `placeholder` elements in `array` with an internal placeholder and returns an array of their indexes.
>
>   用一个内部占位符替换数组中的所有占位符元素，并返回包含其索引的数组。

#### 参数

+   array Array 待修改的数组
+   placeholder any 要替换的占位符。

#### 返回

**Array**

返回占位符索引的新数组。

#### 源码

```js
/** Used as the internal argument placeholder. */
var PLACEHOLDER = '__lodash_placeholder__';

/**
 * Replaces all `placeholder` elements in `array` with an internal placeholder
 * and returns an array of their indexes.
 *
 * @private
 * @param {Array} array The array to modify.
 * @param {*} placeholder The placeholder to replace.
 * @returns {Array} Returns the new array of placeholder indexes.
 */
function replaceHolders(array, placeholder) {
  var index = -1,
      length = array.length,
      resIndex = 0,
      result = [];
  //遍历数组
  while (++index < length) {
    var value = array[index];
    if (value === placeholder || value === PLACEHOLDER) {
      array[index] = PLACEHOLDER;
      //保存修改的位置索引
      result[resIndex++] = index;
    }
  }
  return result;
}

module.exports = replaceHolders;

```



### shuffleSelf

>   A specialized version of `_.shuffle` which mutates and sets the size of `array`.
>
>   shuffle函数的特殊实现版本，会改变并设置数组的大小

#### 参数

>   array Array 待洗牌的数组
>
>   size number 可选 指定返回数组大小（要获取的随机元素数量） 默认size=array.length

#### 返回

**Array**

返回被打乱的数组array的前size个元素

#### 解析

进行size次循环，每次循环,在index到lastIndex位置上的值中随机娶一个放到index位置上，可以得出其放置值x的几率为

P(0号位不放x) · P(1号位不放x) · ... · P(index位放x)=
$$
\frac{length-1}{length}*\frac{length-2}{length-1}*\frac{length-3}{length-2}*...*\frac{length-(index-1)-1}{length-(index-1)}*\frac{1}{length-index}=\frac{1}{length}
$$
洗牌算法的等概率

```js
var baseRandom = require('./_baseRandom');

/**
 * A specialized version of `_.shuffle` which mutates and sets the size of `array`.
 *
 * @private
 * @param {Array} array The array to shuffle.
 * @param {number} [size=array.length] The size of `array`.
 * @returns {Array} Returns `array`.
 */
function shuffleSelf(array, size) {
  //初始化各值
  var index = -1,
      length = array.length,
      lastIndex = length - 1;

  size = size === undefined ? length : size;
  //进行size次循环，每次循环,在index到lastIndex位置上的值中随机娶一个放到index位置上，
  //可以得出其放置值x的几率为
  //P(0号位不放x)*P(1号位不放x)*...*P(index位放x)=(length-1)/length * (length-2)/(length-1) * ... * 
  //1/(length-index)=1/length
  while (++index < size) {
    //获取[index,lastIndex]中的随机索引
    var rand = baseRandom(index, lastIndex),
        value = array[rand];
	//交换array[rand]与array[index]的值
    array[rand] = array[index];
    array[index] = value;
  }
  //取size个值返回（因为只需要size个值，因此循环size次）
  array.length = size;
  return array;
}

module.exports = shuffleSelf;

```



#### 源码

```js
var baseRandom = require('./_baseRandom');

/**
 * A specialized version of `_.shuffle` which mutates and sets the size of `array`.
 *
 * @private
 * @param {Array} array The array to shuffle.
 * @param {number} [size=array.length] The size of `array`.
 * @returns {Array} Returns `array`.
 */
function shuffleSelf(array, size) {
  var index = -1,
      length = array.length,
      lastIndex = length - 1;

  size = size === undefined ? length : size;
  while (++index < size) {
    var rand = baseRandom(index, lastIndex),
        value = array[rand];
	
    array[rand] = array[index];
    array[index] = value;
  }
  array.length = size;
  return array;
}

module.exports = shuffleSelf;

```



### symbolToString

#### 源码

```js
/** Used to convert symbols to primitives and strings. */
var symbolProto = Symbol ? Symbol.prototype : undefined,
    symbolValueOf = symbolProto ? symbolProto.valueOf : undefined,
    symbolToString = symbolProto ? symbolProto.toString : undefined;
```

### strictLastIndexOf

>   A specialized version of `_.lastIndexOf` which performs strict equality comparisons of values, i.e. `===`.
>
>   _.lastIndexOf的一个特殊版本。对值执行严格的相等比较，例如===。

#### 参数

+   array Array 待查询数组
+   value any 待搜索值
+   fromIndex 查询的起始位置

#### 返回

**number**

如果存在待搜索的元素，返回这个元素的索引，否则返回-1

#### 源码

```js
/**
 * A specialized version of `_.lastIndexOf` which performs strict equality
 * comparisons of values, i.e. `===`.
 *
 * @private
 * @param {Array} array The array to inspect.
 * @param {*} value The value to search for.
 * @param {number} fromIndex The index to search from.
 * @returns {number} Returns the index of the matched value, else `-1`.
 */
function strictLastIndexOf(array, value, fromIndex) {
  var index = fromIndex + 1;
  while (index--) {
    if (array[index] === value) {
      return index;
    }
  }
  return index;
}

module.exports = strictLastIndexOf;

```

### stringSize

>   Gets the number of symbols in `string`.
>
>   获取string长度

#### 参数

+   string string 待获取长度的字符串

#### 返回

**number**

返回string的大小

#### 源码

```js
var asciiSize = require('./_asciiSize'),
    hasUnicode = require('./_hasUnicode'),
    unicodeSize = require('./_unicodeSize');

/**
 * Gets the number of symbols in `string`.
 *
 * @private
 * @param {string} string The string to inspect.
 * @returns {number} Returns the string size.
 */
function stringSize(string) {
  return hasUnicode(string)
    ? unicodeSize(string)
    : asciiSize(string);
}

module.exports = stringSize;

```



### stringToPath

> Converts `string` to a property path array.
>
> 将' string '转换为属性路径数组。

#### 参数

+ string	待分割字符串

#### 返回

**Array**

分割后的属性数组

#### 示例1

```js
console.log(stringToPath('a.b.c.d'))//[ 'a', 'b', 'c', 'd' ]
console.log(stringToPath('a'))//[ 'a' ]
console.log(stringToPath('ab[0].b'))//[ 'ab', '0', 'b' ]
console.log(stringToPath('abc.d.e'))//[ 'abc', 'd', 'e' ]
console.log(stringToPath('abc\d.e'))//[ 'abcd', 'e' ]
console.log(stringToPath('abc/d.e'))//[ 'abc/d', 'e' ]
```



#### 源码

源码中涉及的方法

+ [memoizeCapped](#memoizeCapped)

```js
var memoizeCapped = require('./_memoizeCapped');

/** Used to match property names within property paths. */
var rePropName = /[^.[\]]+|\[(?:(-?\d+(?:\.\d+)?)|(["'])((?:(?!\2)[^\\]|\\.)*?)\2)\]|(?=(?:\.|\[\])(?:\.|\[\]|$))/g;

/** Used to match backslashes in property paths. */
var reEscapeChar = /\\(\\)?/g;

/**
 * Converts `string` to a property path array.
 *
 * @private
 * @param {string} string The string to convert.
 * @returns {Array} Returns the property path array.
 */
var stringToPath = memoizeCapped(function(string) {
  var result = [];
  if (string.charCodeAt(0) === 46 /* . */) {
    result.push('');
  }
  string.replace(rePropName, function(match, number, quote, subString) {
    result.push(quote ? subString.replace(reEscapeChar, '$1') : (number || match));
  });
  return result;
});

module.exports = stringToPath;

```





## 内部常量

### reIsDeepProp，reIsPlainProp，rePropName

> Used to match property names within property paths
>
> 用于匹配属性路径中的属性名

```js
  /** Used to match property names within property paths. */
  var reIsDeepProp = /\.|\[(?:[^[\]]*|(["'])(?:(?!\1)[^\\]|\\.)*?\1)\]/,
      reIsPlainProp = /^\w*$/,
      rePropName = /[^.[\]]+|\[(?:(-?\d+(?:\.\d+)?)|(["'])((?:(?!\2)[^\\]|\\.)*?)\2)\]|(?=(?:\.|\[\])(?:\.|\[\]|$))/g;
```

#### 示例1

```js
var reIsDeepProp = /\.|\[(?:[^[\]]*|(["'])(?:(?!\1)[^\\]|\\.)*?\1)\]/,
    reIsPlainProp = /^\w*$/,
    rePropName = /[^.[\]]+|\[(?:(-?\d+(?:\.\d+)?)|(["'])((?:(?!\2)[^\\]|\\.)*?)\2)\]|(?=(?:\.|\[\])(?:\.|\[\]|$))/g;

let obj_3 = {
    a: 1,
    b: 2,
    c: {
        a: 1,
        b: {
            a: 2
        }
    }
}
let res;
res = reIsDeepProp.test('obj_3.a')
console.log('res', res)//res true
res = reIsDeepProp.test('obj_3.c.a')
console.log('res', res)//res true
res = reIsDeepProp.test('obj_3[a]')
console.log('res', res)//res true
res = reIsDeepProp.test('obj_3[c].a')
console.log('res', res)//res true
res = reIsDeepProp.test('obj_3\\a')
console.log('res', res)//res false
res = reIsDeepProp.test('obj_3/a')
console.log('res', res)//res false
```

#### 示例2

```js
let res;
res = rePropName.test('obj_3.a')
console.log('res', res)//res true
res = rePropName.test('obj_3.c.a')
console.log('res', res)//res true
res = rePropName.test('obj_3[a]')
console.log('res', res)//res false
res = rePropName.test('obj_3[c].a')
console.log('res', res)//res true
```

### reEscapeChar

> Used to match backslashes in property paths.
>
> 用于匹配属性路径中的反斜杠。

#### 源码

```js
  /** Used to match backslashes in property paths. */
  var reEscapeChar = /\\(\\)?/g;
```

