# DataFactory 使用笔记
[toc]
## 下载安装
下载DataFactory后使用注册机激活即可
## 导入文件数据
### 编写数据文件
dataFactory仅支持导入TXT文件，格式要求如下
```
item_name:varchar(20),item_date:date(20)
汽水,2021-02-05
苹果,2022-03-22
```
首行为  字段名:类型
字段间以‘,’分割，记录间以换行分割
TXT文件可以使用代码的方式自动爬取数据生成，但注意编码
### 导入数据源
![image-20210728152634035](D:\MarkDownFile\IMG\image-20210728152634035.png)
![image-20210728152659957](D:\MarkDownFile\IMG\image-20210728152659957.png)

之后在资源管理器中选择对应的TXT文件即可
### 使用数据源
![image-20210728152718183](D:\MarkDownFile\IMG\image-20210728152718183.png)

注意：datafactory中新生成的数据源在列表最上方，所以在datatable中选择时新建的数据源在最上方。