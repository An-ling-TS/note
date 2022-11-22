>   TypeScript基础回顾

注意：下面的用例运行在node环境下

## 声明空间

TS具有两种声明空间

+   类型声明空间
+   变量声明空间

### 类型声明空间

类型声明空间中的内容用作类型注解

如

```typescript
interface A{}
type B={}
class C{}
```

### 变量声明空间

顾名思义，变量声明空间中的内容可以用作变量，使用let，function，const，var，class等声明的变量都会进入这里。注意，class声明的类同时具有类型和变量两种特质，它既会提供一个类型到类型声明空间，也会提供一个变量到变量声明空间。

## 模块

默认情况下，TS代码处于全局命名空间，也就是是说当在某个TS文件中声明了变量A时，可以在另一个TS文件中直接使用，当然，此时仅限于不飘红，如果直接运行大概率报错。

--demo11.ts

![image-20221101102038795](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101102046.png)

--demo13.ts

![image-20221101102355221](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101102355.png)

即使demo13和demo11不同级，但只要他们在同一项目下，都会报错。当然，如果编辑器直接打开子文件夹，则不会出现报错，此时的子文件夹被视为了新的项目文件。

--demo12.ts

![image-20221101102108354](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101102108.png)



直接运行demo12会报错，要让它成功运行，我们还需要一点东西

当TS文件中存在关键字import或者export时，这个TS文件中会生成一个本地作用域。本地作用域中的内容将不再自动出现在全局命名空间中，这就是文件模块。

### 模块路径

在导入模块的时候会有各种各样的路径写法

比如：

```typescript
import xxx from 'utils/utils'//使用别名
import xxx from '../../components'//使用相对路径
import xxx from 'react'//动态查找
```

前两种很容易解析，并且一般导入的都是开发者自己编写的模块，来看看第三种，第三种导入的一般是node_modules中的依赖，比如react，lodash，moment，dva等等。

动态查找将会模仿 [Node 模块解析策略](https://nodejs.org/api/modules.html#modules_all_together)

原文地址https://www.bookstack.cn/read/TypeScriptDeepDiveZH/4.md

-   当使用

    ```
    import * as foo from 'foo'
    ```

    ，将会按如下顺序查找模块：

    -   `./node_modules/foo`
    -   `../node_modules/foo`
    -   `../../node_modules/foo`
    -   直到系统的根目录

-   当使用

    ```
    import * as foo from 'something/foo'
    ```

    ，将会按照如下顺序查找内容

    -   `./node_modules/something/foo`
    -   `../node_modules/something/foo`
    -   `../../node_modules/something/foo`
    -   直到系统的根目录

通过 `declare module 'somePath'` 来声明一个全局模块的方式，用来解决查找模块路径，比如使用别名时，程序正常运行，当模块引用飘红出现模块找不到发异常时便可以暂时使用declare声明这个异常的模块来解决。

### **Node模块解析策略**

如何导入一个模块？

如果已找到模块位置

+   **main**

    当模块根目录下存在package.json文件，并且文件中指定了main时，比如

    ```json
    { "name" : "some-library",
      "main" : "./lib/some-library.js" }
    ```

    require('./some-library')语句将尝试加载./some-library/lib/some-library.js

+   **index**

    当不存在package.json 或者main不存在或者无效时，将尝试加载指定路径下的index文件

​	如果最终无法加载，抛出异常

```console
Error: Cannot find module 'xxx'
```

如果导入模块的地址不是相对地址，则还需要先对模块进行查询

先从node_modules中查询：

​	先从本级node_modules开始查询，如果不存在node_modeules或者未查到，则从上级目录开始查询，直到达到根目录

这样一级一级的查找可以让子项目或者说子模块保留并使用自身的依赖，也可以避免多个子模块子项目之间的依赖冲突

来看一个简单的例子

首先给出一个文件结构

![image-20221101145742542](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101145742.png)

+   demo2
    +   node_modules\dep_11
        +   dist
        +   index.ts
        +   package.json
    +   demo2.ts
+   node_modules\dep_11
    +   dist
    +   index.ts
    +   package.json
+   demo1.ts

demo1.ts和demo2.ts的内容一样

```typescript
import { DEP_11 } from 'dep_11'
console.log(DEP_11)
```

最外层的node_modules中dep_11的index.ts内容

```typescript
const DEP_11='dep_11';
export {DEP_11}
```

另一个dep_11的index.ts

```typescript
const DEP_11='dep_111111';
export {DEP_11}
```

他们的package.json也是一样

![image-20221101150255013](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101150255.png)

dist文件是相应的index.ts编译后的内容

![image-20221101150343531](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101150343.png)

分别运行demo1.ts和demo2.ts可以得到

![image-20221101150428243](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101150428.png)

![image-20221101150440233](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20221101150440.png)

+   **根据环境变量加载**

https://nodejs.org/api/modules.html#loading-from-the-global-folders上还提供了一种从全局模块中加载的方式，这种方式的优先级是最低的，也是最不推荐的。全局安装的模块主要是一些脚手架工具，或者命令行工具，对于项目的依赖，最好的方式还是安装在项目下的node_modules中

## 命名空间

TS有一个关键字namespace,可以创建一个命名空间，对一系列逻辑或类型进行分组

基本使用

```typescript
namespace Demo3{
    export function log3(){
        console.log('demo3')
    }
}
Demo3.log3()//demo3
```

这是转译成js的结果

```javascript
var Demo3;
(function (Demo3) {
    function log3() {
        console.log('demo3');
    }
    Demo3.log3 = log3;
})(Demo3 || (Demo3 = {}));
Demo3.log3();
```

用到了IIFE立即执行函数，在全局上挂载了一个Demo3变量，并向Demo3添加属性

我们也可以在其中进行类型声明

```typescript
declare namespace Demo33{
    type param='demo333'
}
function log3(s:Demo33.param){
    console.log(s)
}
log3('demo333')//demo333
```

还可以通过declare将其扩展到全局，这个方式可以用于对接口的响应数据进行类型定义，方便后续使用

## 环境声明

>   TypeScript 的设计目标之一是让你从现有的 JavaScript 库中安全、轻松的使用 TypeScript，你可以通过 TypeScript 声明文件来做到这一点。

>   通过 `declare` 关键字，来告诉 TypeScript，你正在试图表述一个其他地方已经存在的代码（如：写在 JavaScript、CoffeeScript 或者是像浏览器和 Node.js 运行环境里的代码）

通过declare声明的这些内容，放在.d.ts文件里，项目中常命名为 globals.d.ts 或者 vendor.d.ts

>   如果一个文件有扩展名 `.d.ts`，这意味着每个顶级的声明都必须以 `declare` 关键字作为前缀。这有利于向作者说明，在这里 TypeScript 将不会把它编译成任何代码，同时他需要确保这些在编译时存在。

### lib.d.ts

global.d.ts中通常放的是项目本身的一些声明。

>   安装 `TypeScript` 时，会顺带安装 `lib.d.ts` 等声明文件。此文件包含了 JavaScript 运行时以及 DOM 中存在各种常见的环境声明。

+   它自动包含在 TypeScript 项目的编译上下文中；
+   它能让你快速开始书写经过类型检查的 JavaScript 代码。
+   可以通过指定 `—noLib` 的编译器命令行标志（或者在 `tsconfig.json` 中指定选项 `noLib`: true）从上下文中排除此文件。

>   `lib.d.ts` 的内容主要是一些变量声明（如：`window`、`document`、`math`）和一些类似的接口声明（如：`Window`、`Document`、`Math`）。