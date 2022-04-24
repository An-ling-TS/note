# MarkDown语法

[toc]

**本文一切语法均在Typora上测试**

## 基本语法

### 目录 [toc]

```markdown
[toc]
```

![image-20210818160847958](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210818160847.png)

### 内容缩进 	>+空格

> text

```markdown
缩进：
>+空格
取消缩进：
shift+tab
```

### 加粗

**text**

```
**text**
```

### 斜体

*text*

```
*text*
```

斜体加粗

***text***

```
***text***
```

### 代码块```

符号：```

或者：```文件后缀名

加上文件后缀名可以使得代码块与相应语言对应，获得颜色高亮标记

### 分割线

---

----

***

****

任意三个或三个以上的 -或者*

## 链接跳转

### 页内跳转

MarkDown中是通过定义链接的方式来定义跳转的，在这里，跳转也称为**锚点**，跳转的目标称为**锚点目标**；

所以，在 MarkDown 中实现页面内跳转的方法就是：定义一个 `锚点目标` 和 对应的**锚点**，用户点击**锚点**便可跳转到对应的**锚点**目标` 位置处；

#### **锚点的定义**

锚点就是一个链接，另外，由于在MarkDown中可以直接写HTML，所以在MarkDown中实现锚点有两种方式：MarkDown方式 和 HTML方式；

<span id="跳转到这">跳转到这</span>

#### MarkDown锚点

MarkDown锚点本质上就是一个MarkDown链接，只是链接地址的格式为：

```markdown
链接地址 = #目标内容
```

MarkDown锚点 的定义也有两种方式

##### 行内式

```objectivec
锚点 = [内容](#目标内容 "标题")
```

**说明：**

- `标题` 是可选的，可以用单引号 或 双引号；

**转换成HTML后，会生成如下标签：**

```xml
<a href="#目标内容" title="标题">内容</a>
```

**示例：**

```csharp
[MarkDown方式的锚点](#MarkDown锚点)
```

**渲染成HTML后，会生成如下标签：**

```xml
<a href="#MarkDown锚点" >MarkDown方式的锚点</a>
```

[MarkDown方式的锚点](#跳转到这)

[MarkDown方式的锚点](#跳转到这2)

[MarkDown方式的锚点](#页内跳转)

[MarkDown方式的锚点](#字体)

<a href="#跳转到这" >MarkDown方式的锚点</a>

##### 参考式

**语法：**

```objectivec
锚点 = [内容][参考标识符]
参考标识符 = [标识符]: #目标内容 "标题"
```

**说明：**

- `锚点` 和 `参考标识符` 的定义没有先后顺序；
- `[内容]` 和 `[参考标识符]` 之间可以有一个空格，也可以没有空格；
- 如果 `内容` 和 `参考标识符` 一样，也可简写成 `[参考标识符][]` ;
- `标题` 是可选的，可以用单引号、双引号或是圆括弧包着；

**转换成HTML后，会生成如下标签：**

```xml
<a href="#目标内容" title="标题">内容</a>
```

**示例：**

```css
[MarkDown方式的锚点]: #MarkDown锚点
[MD锚点][MarkDown方式的锚点]
[MarkDown方式的锚点][]
```

**渲染成HTML后，会生成如下标签：**

```xml
<a href="#MarkDown锚点">MD锚点</a>
<a href="#MarkDown锚点">MarkDown方式的锚点</a>
```

#### HTML锚点

HTML锚点本质上就是一个a链接，格式为：

```xml
<a href="#目标内容">内容</a>
```

**注意：**

- `HTML锚点` 的目标锚点只能是标签形式的锚点目标

**示例：**

```xml
<a href="#html锚点">HTML方式的锚点</a>
```

#### 锚点目标的定义

锚点目标有2种定义方式：MarkDown形式 和 标签形式；

#### MarkDown形式的锚点目标

MarkDown形式的锚点目标的定义其实就是**标题**的定义，即：任何级别的标题可以直接作为锚点目标；

```undefined
标题内容 = 目标内容
```

所以，类Setext形式 和 类atx形式 的标题都可作为 `锚点目标` ；

**锚点目标定义的示例：**

```bash
这是一个锚点目标
====

这是一个锚点目标
---

# 这是一个锚点目标
## 这是一个锚点目标
### 这是一个锚点目标
#### 这是一个锚点目标
##### 这是一个锚点目标
###### 这是一个锚点目标
```

**注意：**

- [锚点](#锚点的定义)的 `目标内容` 中不能有大家字母和空格，所以如果锚点目标的 `目标内容` 中有大写字母或空格，则需要在定义锚点中的 `目标内容` 时，把大写字母改成小写字母，把空格改成 `-`；
- 锚点的 `目标内容` 中不能含有以下字符：半角点(即英文中的句号)

#### 标签形式的锚点目标

因为MarkDown链接会被转成a标签，并且MarkDown中也可以写标签，所以可以利用HTML的锚点机制直接定义一个带 `id` 特性的任意标签 或 带 `name` 特性的 a 标签（注意：在HTML5中，a标签已经不再支持 name 特性）作为锚点目标，然后把MarkDown中的锚点地址的目标内容设置为 `id` 或 `name` 特性的值；这样便可以实现页面内跳转；

这种形式的锚点目标的定义格式为：

```xml
<标签名 id="目标内容">元素内容</标签名>
```

或

```xml
<a name="目标内容">元素内容</a>
```

**注意：**

- 标签形式的锚点目标的id特性值中是不能含有中文字符；
- `name` 特性只能应用在 a 标签上；
- HTML5不支持通过a标签的 `name` 特性来定义锚点目标；

**示例：**

```xml
<div id="这是锚点目标">跳转到这里</div>
```

或

```xml
<a name="这是锚点目标">跳转到这里</a>
```

<span id="跳转到这2">
    跳转到这2
</span>

## 字体

### 颜色

```
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>
```

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>

### 文字底色

```
<table><tr><td bgcolor=yellow>背景色yellow</td></tr></table>
```

<table><tr><td bgcolor=yellow>背景色yellow</td></tr></table>

## 公式块语法

### 矩阵

普通矩阵，不带括号

> \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix}

$$
\begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix}
$$

带中括号

> ```swift
> \left[ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right]
> ```

$$
\left[ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right]
$$

带大括号

> ```swift
> \left\{ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}
> ```

$$
\left\{ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}
$$

矩阵带参数

> ```swift
> A= \left\{ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}
> ```

$$
A= \left\{ \begin{matrix} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{matrix} \right\}
$$

矩阵带省略号

> //\cdots为水平方向的省略号
> //\vdots为竖直方向的省略号
> //\ddots为斜线方向的省略号

> ```
> A= \left\{ \begin{matrix} a & b & \cdots & e\\ f & g & \cdots & j \\ \vdots & \vdots & \ddots & \vdots \\ p & q & \cdots & t \end{matrix} \right\}
> ```

$$
A= \left\{ \begin{matrix} a & b & \cdots & e\\ f & g & \cdots & j \\ \vdots & \vdots & \ddots & \vdots \\ p & q & \cdots & t \end{matrix} \right\}
$$

矩阵带横线

> //array必须为array
>  //{cccc|c}中的c表示矩阵元素，可以控制|的位置
>
> ```swift
> A= \left\{ \begin{array}{cccc|c} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{array} \right\}
> ```

$$
A= \left\{ \begin{array}{cccc|c} a & b & c & d & e\\ f & g & h & i & j \\ k & l & m & n & o \\ p & q & r & s & t \end{array} \right\}
$$

### 上标、下标与组合

上标符号，符号：`^`，如：$x^4$
$$
x^4
$$


下标符号，符号：`_`，如：$x_1$
$$
x_1
$$


组合符号，符号：`{}`，如：${16}_{8}O{2+}_{2}$
$$
{16}_{8}O{2+}_{2}
$$

### 汉字、字体与格式

汉字形式，符号：`\mbox{}`，如：$V_{\mbox{初始}}$
$$
V_{\mbox{初始}}
$$
字体控制，符号：`\displaystyle`，如：$\displaystyle \frac{x+y}{y+z}$
$$
\displaystyle \frac{x+y}{y+z}
$$
下划线符号，符号：`\underline`，如：$\underline{x+y}$
$$
\underline{x+y}
$$
标签，符号`\tag{数字}`，如：$\tag{11}$
$$
\tag{11}
$$
上大括号，符号：`\overbrace{算式}`，如：$\overbrace{a+b+c+d}^{2.0}$
$$
\overbrace{a+b+c+d}^{2.0}
$$
下大括号，符号：`\underbrace{算式}`，如：$a+\underbrace{b+c}_{1.0}+d$
$$
a+\underbrace{b+c}_{1.0}+d
$$
上位符号，符号：`\stacrel{上位符号}{基位符号}`，如：$\vec{x}\stackrel{\mathrm{def}}{=}{x_1,\dots,x_n}$
$$
\vec{x}\stackrel{\mathrm{def}}{=}{x_1,\dots,x_n}
$$

### 占位符

两个quad空格，符号：`\qquad`，如：$x \qquad y$
$$
x \qquad y
$$
quad空格，符号：`\quad`，如：$x \quad y$
$$
x \quad y
$$
大空格，符号`\`，如：$x \  y$
$$
x \  y
$$
中空格，符号`\:`，如：$x \: y$
$$
x \: y
$$
小空格，符号`\,`，如：$x \, y$
$$
x  \, y
$$
没有空格，符号``，如：$x    y$
$$
x    y
$$
紧贴，符号`\!`，如：$x \! y$
$$
x \! y
$$

### 定界符与组合

括号，符号：`（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)`，如：$（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)$
$$
（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)
$$
中括号，符号：`[]`，如：$[x+y]$
$$
[x+y]
$$
大括号，符号：`\{ \}`，如：$\\{x+y\\}$
$$
\{x+y\}
$$
自适应括号，符号：`\left \right`，如：$\left(x\right)$，$\left(x{yz}\right)$
$$
\left(x{yz}\right)
\left(x\right)
$$
组合公式，符号：`{上位公式 \choose 下位公式}`，如：${n+1 \choose k}={n \choose k}+{n \choose k-1}$
$$
{n+1 \choose k}={n \choose k}+{n \choose k-1}
$$
组合公式，符号：`{上位公式 \atop 下位公式}`，如：$\sum_{k_0,k_1,\ldots>0 \atop k_0+k_1+\cdots=n}A_{k_0}A_{k_1}\cdots$
$$
\sum_{k_0,k_1,\ldots>0 \atop k_0+k_1+\cdots=n}A_{k_0}A_{k_1}\cdots
$$

### 四则运算

加法运算，符号：`+`，如：$x+y=z$
$$
x+y=z
$$
减法运算，符号：`-`，如：$x-y=z$
$$
x-y=z
$$
加减运算，符号：`\pm`，如：$x \pm y=z$
$$
x \pm y
$$
减甲运算，符号：`\mp`，如：$x \mp y=z$
$$
x \mp y=z
$$
乘法运算，符号：`\times`，如：$x \times y=z$
$$
x \times y=z
$$
点乘运算，符号：`\cdot`，如：$x \cdot y=z$
$$
x \cdot y=z
$$
星乘运算，符号：`\ast`，如：$x \ast y=z$
$$
x \ast y=z
$$
除法运算，符号：`\div`，如：$x \div y=z$
$$
x \div y=z
$$
斜法运算，符号：`/`，如：$x/y=z$
$$
x/y=z
$$
分式表示，符号：`\frac{分子}{分母}`，如：$\frac{x+y}{y+z}$
$$
\frac{x+y}{y+z}
$$
分式表示，符号：`{分子} \voer {分母}`，如：${x+y} \over {y+z}$
$$
{x+y} \over {y+z}
$$
绝对值表示，符号：`||`，如：$|x+y|$
$$
|x+y|
$$


### 高级运算

平均数运算，符号：`\overline{算式}`，如：$\overline{xyz}$
$$
\overline{xyz}
$$
开二次方运算，符号：`\sqrt`，如：$\sqrt x$
$$
\sqrt x
$$
开方运算，符号：`\sqrt[开方数]{被开方数}`，如：$\sqrt[3]{x+y}$
$$
\sqrt[3]{x+y}
$$
对数运算，符号：`\log`，如：$\log(x)$
$$
\log(x)
$$
极限运算，符号：`\lim`，如：$\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
$$
\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}
$$
极限运算，符号：`\displaystyle \lim`，如：$\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
$$
\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}
$$
求和运算，符号：`\sum`，如：$\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
$$
\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}
$$
求和运算，符号：`\displaystyle \sum`，如：$\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
$$
\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}
$$
积分运算，符号：`\int`，如：$\int^{\infty}_{0}{xdx}$
$$
\int^{\infty}_{0}{xdx}
$$
积分运算，符号：`\displaystyle \int`，如：$\displaystyle \int^{\infty}_{0}{xdx}$
$$
\displaystyle \int^{\infty}_{0}{xdx}
$$
微分运算，符号：`\partial`，如：$\frac{\partial x}{\partial y}$
$$
\frac{\partial x}{\partial y}
$$
矩阵表示，符号：`\begin{matrix} \end{matrix}`，如：$\left[ \begin{matrix} 1 &2 &\cdots &4\5 &6 &\cdots &8\\vdots &\vdots &\ddots &\vdots\13 &14 &\cdots &16\end{matrix} \right]$
$$
\left[ \begin{matrix} 1 &2 &\cdots &4\5 &6 &\cdots &8\\vdots &\vdots &\ddots &\vdots\13 &14 &\cdots &16\end{matrix} \right]
$$

### 逻辑运算

等于运算，符号：`=`，如：$x+y=z$
$$
x+y=z
$$
大于运算，符号：`>`，如：$x+y>z$
$$
x+y>z
$$
小于运算，符号：`<`，如：$x+y<z$
$$
x+y<z
$$
大于等于运算，符号：`\geq`，如：$x+y \geq z$
$$
x+y \geq z
$$
小于等于运算，符号：`\leq`，如：$x+y \leq z$
$$
x+y \leq z
$$
不等于运算，符号：`\neq`，如：$x+y \neq z$
$$
x+y \neq z
$$
不大于等于运算，符号：`\ngeq`，如：$x+y \ngeq z$
$$
x+y \ngeq z
$$
不大于等于运算，符号：`\not\geq`，如：$x+y \not\geq z$
$$
x+y \not\geq z
$$
不小于等于运算，符号：`\nleq`，如：$x+y \nleq z$
$$
x+y \nleq z
$$
不小于等于运算，符号：`\not\leq`，如：$x+y \not\leq z$
$$
x+y \not\leq z
$$
约等于运算，符号：`\approx`，如：$x+y \approx z$
$$
x+y \approx z
$$
恒定等于运算，符号：`\equiv`，如：$x+y \equiv z$
$$
x+y \equiv z
$$


### 集合运算

属于运算，符号：`\in`，如：$x \in y$
$$
x \in y
$$
不属于运算，符号：`\notin`，如：$x \notin y$
$$
x \notin y
$$
不属于运算，符号：`\not\in`，如：$x \not\in y$
$$
x \not\in y
$$
子集运算，符号：`\subset`，如：$x \subset y$
$$
x \subset y
$$
子集运算，符号：`\supset`，如：$x \supset y$
$$
x \supset y
$$
真子集运算，符号：`\subseteq`，如：$x \subseteq y$
$$
x \subseteq y
$$
非真子集运算，符号：`\subsetneq`，如：$x \subsetneq y$
$$
x \subsetneq y
$$
真子集运算，符号：`\supseteq`，如：$x \supseteq y$
$$
x \supseteq y
$$
非真子集运算，符号：`\supsetneq`，如：$x \supsetneq y$
$$
x \supsetneq y
$$
非子集运算，符号：`\not\subset`，如：$x \not\subset y$
$$
x \not\subset y
$$
非子集运算，符号：`\not\supset`，如：$x \not\supset y$
$$
x \not\supset y
$$
并集运算，符号：`\cup`，如：$x \cup y$
$$
x \cup y
$$
交集运算，符号：`\cap`，如：$x \cap y$
$$
x \cap y
$$
差集运算，符号：`\setminus`，如：$x \setminus y$
$$
x \setminus y
$$
同或运算，符号：`\bigodot`，如：$x \bigodot y$
$$
x \bigodot y
$$
同与运算，符号：`\bigotimes`，如：$x \bigotimes y$
$$
x \bigotimes y
$$
实数集合，符号：`\mathbb{R}`，如：`\mathbb{R}`
$$
\mathbb{R}
$$
自然数集合，符号：`\mathbb{Z}`，如：`\mathbb{Z}`
$$
\mathbb{Z}
$$
空集，符号：`\emptyset`，如：$\emptyset$
$$
\emptyset
$$


### 数学符号

无穷，符号：`\infty`，如：$\infty$
$$
\infty
$$
虚数，符号：`\imath`，如：$\imath$
$$
\imath
$$
虚数，符号：`\jmath`，如：$\jmath$
$$
\jmath
$$
数学符号，符号`\hat{a}`，如：$\hat{a}$
$$
\hat{a}
$$
数学符号，符号`\check{a}`，如：$\check{a}$
$$
\check{a}
$$
数学符号，符号`\breve{a}`，如：$\breve{a}$
$$
\breve{a}
$$
数学符号，符号`\tilde{a}`，如：$\tilde{a}$
$$
\tilde{a}
$$
数学符号，符号`\bar{a}`，如：$\bar{a}$
$$
\bar{a}
$$
矢量符号，符号`\vec{a}`，如：$\vec{a}$
$$
\vec{a}
$$
数学符号，符号`\acute{a}`，如：$\acute{a}$
$$
\acute{a}
$$
数学符号，符号`\grave{a}`，如：$\grave{a}$
$$
\grave{a}
$$
数学符号，符号`\mathring{a}`，如：$\mathring{a}$
$$
\mathring{a}
$$
一阶导数符号，符号`\dot{a}`，如：$\dot{a}$
$$
\dot{a}
$$
二阶导数符号，符号`\ddot{a}`，如：$\ddot{a}$
$$
\ddot{a}
$$
上箭头，符号：`\uparrow`，如：$\uparrow$
$$
\uparrow
$$
上箭头，符号：`\Uparrow`，如：$\Uparrow$
$$
\Uparrow
$$
下箭头，符号：`\downarrow`，如：$\downarrow$
$$
\downarrow
$$
下箭头，符号：`\Downarrow`，如：$\Downarrow$
$$
\Downarrow
$$
左箭头，符号：`\leftarrow`，如：$\leftarrow$
$$
\leftarrow
$$
左箭头，符号：`\Leftarrow`，如：$\Leftarrow$
$$
\Leftarrow
$$
右箭头，符号：`\rightarrow`，如：$\rightarrow$
$$
\rightarrow
$$
右箭头，符号：`\Rightarrow`，如：$\Rightarrow$
$$
\Rightarrow
$$
底端对齐的省略号，符号：`\ldots`，如：$1,2,\ldots,n$
$$
\ldots
$$
中线对齐的省略号，符号：`\cdots`，如：$x_1^2 + x_2^2 + \cdots + x_n^2$
$$
x_1^2 + x_2^2 + \cdots + x_n^2
$$
竖直对齐的省略号，符号：`\vdots`，如：$\vdots$
$$
\vdots
$$
斜对齐的省略号，符号：`\ddots`，如：$\ddots$
$$
\ddots
$$

[跳转到这]: 
