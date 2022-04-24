lodash为我们提供了非常丰富的工具函数，我们可以通过使用其中一个或者多个实现大量需求，而不需要自身去琢磨开发，减少工作中的代码量。

# 对象数组合并

## 需求

+   指定某一属性作为基准属性
+   对于对象数组中的对象，所指定的属性，其值相等时，将这些对象合并为一个

存在一个如下的对象数组

```js
[
    ...
    {
        a:1,
        id:101
    },
    ...
    {
        a:2,
        id:101,
        b:3,
    },
]
```

我希望将它处理为如下形式

```js
[
    ...
    {
        a:2,
        id:101,
        b:3
    }
    ...
]
```

## 实现

**map(values(groupBy(input, 'id')), (subArr) => reduce(subArr, merge, {}))**

**使用示例**

```js
import { groupBy, merge, values, map, reduce } from 'lodash'
let input = [
    {
        a: 101,
        id: '101',
        b: 'b101',
        d: {
            e: 101
        }
    },
    {
        a: 102,
        id: '102',
        b: 'b102'
    },
    {
        a: 103,
        id: '103',
        b: 'b103'
    },
    {
        a: 1001,
        id: '101',
        b: 'b101',
        c: 101,
        d: {
            e: 1001,
            f: 101
        }
    },
    {
        a: 1002,
        id: '102',
        b: 'b1002'
    },
]
let output = map(values(groupBy(input, 'id')), (subArr) => reduce(subArr, merge, {}))
console.log(output)
```

输出

```js
[
  { a: 1001, id: '101', b: 'b101', d: { e: 1001, f: 101 }, c: 101 },
  { a: 1002, id: '102', b: 'b1002' },
  { a: 103, id: '103', b: 'b103' }
]
```

## 解析

可以看到，我们使用了5个lodash函数进行搭配使用

+   merge 进行对象递归合并
+   groupBy 将数组根据迭代器进行分组
+   values 将对象值转为数组
+   reduce 操作叠加
+   map 遍历迭代

## 额外

基于这五个函数及其系列函数，我们可以进行更加丰富的对象数组合并操作

### 右合并

将reduce函数更换为reduceRight可以实现右合并

```js
let output = map(values(groupBy(input, 'id')), (subArr) => reduceRight(subArr, merge, {}))
```

### 自定义合并

将merge函数更换为其它函数可以实现自定义合并

```js
let output = map(values(groupBy(input, 'id')), (subArr) => reduce(subArr, (res, obj) => {
    return merge(res, obj)
}, {}))
```



