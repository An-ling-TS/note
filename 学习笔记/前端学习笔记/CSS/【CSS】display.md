# display:flex|inline-flex

[参考](http://ruanyifeng.com/blog/2015/07/flex-grammar.html)

>    **Flexible Box 弹性布局**  任何一个容器都可以指定为 Flex 布局,行内元素也可以使用弹性布局 inline-flex

**注意：**设为 Flex 布局以后，子元素的float、clear和vertical-align属性将失效。

==采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。==

![image-20220422103819727](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220422103852.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。

项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

## 【容器属性】flex-direction 

**flex-direction: row | row-reverse | column | column-reverse**

>    决定主轴的方向（即项目的排列方向）

具有4个值

+   row 默认 主轴为水平方向，起点在左端。

 <div style="display:flex;flex-direction:row">
        <div style="width:50px;height:50px;background:red">1</div>
        <div style="width:50px;height:50px;background:yellow">2</div>
        <div style="width:50px;height:50px;background:blue">3</div>
        <div style="width:50px;height:50px;background:green">4</div>
        <div style="width:50px;height:50px;background:gray">5</div>
    </div>

+   row-reverse 主轴为水平方向，起点在右端。

<div style="display:flex;flex-direction:row-reverse">
    <div style="width:50px;height:50px;background:red">1</div>
    <div style="width:50px;height:50px;background:yellow">2</div>
    <div style="width:50px;height:50px;background:blue">3</div>
    <div style="width:50px;height:50px;background:green">4</div>
    <div style="width:50px;height:50px;background:gray">5</div>
</div>

+   column 主轴为垂直方向，起点在上沿。

<div style="display:flex;flex-direction:column">
    <div style="width:50px;height:50px;background:red">1</div>
    <div style="width:50px;height:50px;background:yellow">2</div>
    <div style="width:50px;height:50px;background:blue">3</div>
    <div style="width:50px;height:50px;background:green">4</div>
    <div style="width:50px;height:50px;background:gray">5</div>
</div>

+   column-reverse 主轴为垂直方向，起点在下沿。

<div style="display:flex;flex-direction:column-reverse">
    <div style="width:50px;height:50px;background:red">1</div>
    <div style="width:50px;height:50px;background:yellow">2</div>
    <div style="width:50px;height:50px;background:blue">3</div>
    <div style="width:50px;height:50px;background:green">4</div>
    <div style="width:50px;height:50px;background:gray">5</div>
</div>
## 【容器属性】flex-wrap

**flex-wrap: nowrap | wrap | wrap-reverse**

>   定义换行

具有3个值

+   nowrap 	默认 不换行
+   wrap 换行，向下换行，第一行在上方。
+   wrap-reverse 换行，向上换行，第一行在下方。

## 【容器属性】flex-flow

 **flex-flow: <flex-direction> || <flex-wrap>**

>   flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap

## 【容器属性】justify-content

**justify-content: flex-start | flex-end | center | space-between | space-around**

>   定义了项目在主轴上的对齐方式

+    flex-start 默认 左对齐
+   flex-end 右对齐
+   center 居中
+   space-between 两端对齐，项目之间的间隔都相等
+   space-around 每个项目两侧的间隔相等，项目之间的间隔比项目与边框的间隔大一倍。



## 【容器属性】align-items

**align-items: flex-start | flex-end | center | baseline | stretch**

>   定义项目在交叉轴上的对齐方式

+   flex-start  起点对齐

 <div style="display:flex;align-items:flex-start">
        <div style="width:50px;height:50px;background:red">1</div>
        <div style="width:50px;height:60px;background:yellow">2</div>
        <div style="width:50px;height:70px;background:blue">3</div>
        <div style="width:50px;height:80px;background:green">4</div>
        <div style="width:50px;height:90px;background:gray">5</div>
    </div>

+   flex-end 终点对齐

 <div style="display:flex;align-items:flex-end">
        <div style="width:50px;height:50px;background:red">1</div>
        <div style="width:50px;height:60px;background:yellow">2</div>
        <div style="width:50px;height:70px;background:blue">3</div>
        <div style="width:50px;height:80px;background:green">4</div>
        <div style="width:50px;height:90px;background:gray">5</div>
    </div>

+   center 居中

 <div style="display:flex;align-items:center">
        <div style="width:50px;height:50px;background:red">1</div>
        <div style="width:50px;height:60px;background:yellow">2</div>
        <div style="width:50px;height:70px;background:blue">3</div>
        <div style="width:50px;height:80px;background:green">4</div>
        <div style="width:50px;height:90px;background:gray">5</div>
    </div>

+   baseline  项目的第一行文字的基线对齐

 <div style="display:flex;align-items:baseline">
        <div style="width:50px;height:50px;background:red;line-height:100px">1</div>
        <div style="width:50px;height:60px;background:yellow;line-height:20px">2</div>
        <div style="width:50px;height:70px;background:blue;line-height:30px">3</div>
        <div style="width:50px;height:80px;background:green;line-height:40px">4</div>
        <div style="width:50px;height:90px;background:gray;line-height:50px">5</div>
    </div>

+   stretch 默认值 如果项目未设置高度或设为auto，将占满整个容器的高度

 <div style="display:flex;align-items:stretch">
        <div style="width:50px;background:red">1</div>
        <div style="width:50px;background:yellow">2</div>
        <div style="width:50px;background:blue">3</div>
        <div style="width:50px;height:80px;background:green">4</div>
        <div style="width:50px;height:90px;background:gray">5</div>
    </div>

## 【容器属性】align-content

**align-content: flex-start | flex-end | center | space-between | space-around | stretch**

>   定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用 （多行对应多根轴线）

+   flex-start 与交叉轴的起点对齐
+   flex-end 与交叉轴的终点对齐
+   center  与交叉轴的中点对齐
+    space-between 与交叉轴两端对齐，轴线之间的间隔平均分布
+   space-around 每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
+   stretch 默认 轴线占满整个交叉轴

## 【容器属性】column-gap

**column-gap: length|normal**

>   规定列之间的间隔

+   length 可以使用各单位 px % vh vw等

列之间的间隔是20px

 <div style="display:flex;column-gap:20px">
        <div style="width:50px;height:50px;background:red">1</div>
        <div style="width:50px;height:50px;background:yellow">2</div>
        <div style="width:50px;height:50px;background:blue">3</div>
        <div style="width:50px;height:50px;background:green">4</div>
        <div style="width:50px;height:50px;background:gray">5</div>
    </div>

+   normal 规定列间间隔为一个常规的间隔。W3C 建议的值是 1em。

## 【容器属性】row-gap

**row-gap: length|normal**

>   规定行之间的间隔



# display:grid

