# 创建开发项目

1.   创建一个新的空项目文件夹 plugin-demo

2.   在新文件夹下执行 create-umi

3.   选择plugin

     ![image-20220704170430944](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220704170438.png)

​	4.同时开启umi-ui插件开发

![image-20220704170725211](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220704170725.png)

# 文件结构

![image-20220704173031742](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220704173031.png)



# 基础启动

1.下载依赖

2.执行 yarn build -w 进行打包，使用 -w 对变更进行增量编译

​	为什么要先打包？在src/index.ts中可以看到

![image-20220706160903252](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220706160910.png)

3.另外开一个终端启动

这里启动的实际上是example文件夹的示例项目

![image-20220706165743759](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220706165743.png)

# 开发示例

在 Umi 中，插件实际上就是一个 JS 模块，需要定义一个插件的初始化方法并默认导出

```typescript
export default (api) => {
  // your plugin code here
};
```

看一下plugin-model的index.ts示例

![image-20220706173510857](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220706173510.png)

该初始化方法会收到 `api` 参数，Umi 提供给插件的接口都是通过它暴露出来的。

如果插件需要发布为 npm 包，那么需要发布之前做编译，确保发布的代码里面是 ES5 的代码。

# 进阶示例

通过一个完整的路由控制器插件示例来进行学习

这个示例的功能简要如下：

+   检索src/.umirc.ts文件
+   获取文件中导出的路由并显示在umi-ui中
+   umi-ui中对应显示的路由更改时，文件中对应路由也更改

## 获取路径

通过umi的插件开发API与node的path模块获取.umirc.ts的绝对路径

```typescript
const {
    paths,
} = api;
/**
 * 获取src/constants的绝对路径
 */
function getConstantsPath() {
    return join(paths.absSrcPath!, './constants.ts');
}
api.logger.info('path:', getConstantsPath())
```

![image-20220711171856071](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220711171903.png)

## 读取内容

使用node的fs模块读取文件的内容

```typescript
fs.readFile(getConstantsPath(), (err: any, data: any) => {
    if (err) {
        return console.log("读取文件失败！" + err.message)
    }
    console.log("读取文件成功！" + data)
})
```

![image-20220711171944824](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220711171944.png)

## 渲染内容

对前面获取的内容进行进一步的处理，然后将其传递给ui面板进行展示

与ui面板的通信函数 

**服务端接口**

**onUISocket**

```typescript
api.onUISocket(({ action: { type, payload, lang }, log, send, success, failure, progress }) => {
    if (type === 'routes') {
        send({ type: `${type}/success`, payload: { routes } });
    }
});
```

参数

![image-20220729084704235](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220729084711.png)

+   send({type,payload}) 向客户端发送信息

1.  按约定，如果客户端用 `api.callRemote` 调用服务端接口，处理完数据需 `send` 加 `/success` 或 `/failure` 后缀的数据表示成功和失败。

+   success(payload)  等同send({ type: `${type}/success` })
+   failure(payload)  等同 send({ type: `${type}/failure` })
+   progress(payload)  等同 send({ type:${type}/progress}) 
+   log(level, message) 在控制台和客户端同时打印日志。

上面的例子展示了完整的参数，而下面这个例子和上面的例子实现的效果是一样的

```typescript
api.onUISocket(({ action, failure, success }) => {
    if (action.type === 'routes') {
        success({
            routes: routes,

        })
    }
});
```

**客户端接口**

**callRemote**

```typescript
api.callRemote({
  type: string;
  payload: object;
  onProgress: (data) => void;
  keep: boolean;
})
```

+   type 链接名/服务端接口名
+   payload 参数
+   onProgress 监听服务端传的数据
+   keep 是否建立持久性链接

示例

![image-20220729085742835](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220729085742.png)

### 效果

![image-20220729090028775](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220729090028.png)

## 嵌套路由

将渲染函数改为递归，对子路由进行渲染，并保存路径数组作为路由项的索引

```typescript
const renderList = (list: UmiRoute.RoutesList, indexArr: Array<number> = []) => {
        const _indexArr: Array<number> = [...indexArr]
        return <List
            style={{ padding: 20 }}
            dataSource={list}
            renderItem={(item, index) => {
                _indexArr.push(index);
                return <React.Fragment>
                    <List.Item
                        actions={[<a key="list-loadmore-edit" onClick={() => handleEdit(item, _indexArr)}>edit</a>, <a key="list-loadmore-more">more</a>]}
                    >
                        <List.Item.Meta
                            title={<a href={`http://localhost:8000${item.path}`} target='_blank'>{item.name}</a>}
                            description={`页面源：${JSON.stringify(item.component)}`}
                        />
                    </List.Item>
                    {
                        !!item.routes && renderList(item.routes, _indexArr)
                    }
                </React.Fragment>
            }}
        />
    }
```

### 效果

![image-20220729111911215](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220729111911.png)

## 文件修改

使用node的fs模块对文件进行重写与重新读取

```typescript
/**
 * 写入文件
 * @param str 
 */
function overWriteRoutes(str: string) {
    fs.writeFileSync(getConstantsPath(), str, { mode: 0o666 });
}
function saveRoutes(fileData: string, oldRoutesStr: string, newRoutes: UmiRoute.RoutesList, success: Function, failure: Function) {
    try {
        const newRoutesStr = 'routes:' + addEnter(removeQuotes(JSON.stringify(newRoutes)))
        overWriteRoutes(fileData.replace(oldRoutesStr, newRoutesStr))
        console.log('写入成功');
        readRoutes()
        success({
            routes
        })
    } catch (e) {
        failure({
            message: '路由重写失败',
        })
    }
}
```

![image-20220729163359033](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220729163359.png)

# 附

umi3插件API地址 https://v3.umijs.org/zh-CN/plugins/api

demo仓库地址 https://gitee.com/lianganqing/umi-route-plug