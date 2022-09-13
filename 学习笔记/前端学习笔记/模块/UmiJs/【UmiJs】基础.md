>   可扩展的企业级前端应用框架。Umi 以路由为基础的，同时支持配置式路由和约定式路由，保证路由的功能完备，并以此进行功能扩展。然后配以生命周期完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求。

umiJs集成了非常多的内容模块，使得开发者可以轻易使用，这一点大大降低了网页应用的开发门槛。开发者无需了解太多，只需要了解一点点的基础语法，就能完成开发。当然，当前的脚手架大多如此，只是umiJs尤其突出罢了。

用一个词概括umiJS：简单粗暴

也正是umiJS的这些特点，umiJS项目的依赖普遍很大。

# 下载安装

下载安装

```shell
yarn global add umi
```

查看版本

```shell
umi -v
```

如果出现 <font color="red">'umi' 不是内部或外部命令，也不是可运行的程序或批处理文件。</font> 执行命令yarn global bin 获取文件路径并将文件路径添加到Path环境变量中

如果出现 <font color="red">文件名、目录名或卷标语法不正确。</font> 打开上面yarn global bin命令输出的文件夹中的umi.cmd，将“%~dp0\”删除并保存。

![image-20220607172546633](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220607172554.png)

如果不行，执行yarn global dir 发现输出的文件路径与yarn global bin输出的文件路径不在一个盘中， 

执行

```shell
yarn config set global-folder "D:\yarn\global"
yarn config set cache-folder "D:\yarn\cache"
```

将yarn的全局目录和缓存目录设置到同一个文件夹下

# 创建应用

```shell
yarn create @umijs/umi-app
```

如果使用此命令，需要先新建一个项目文件夹，这个命令会在当前路径下直接创建项目结构 这个命令将使用最新的umi版本

或者

```shell
yarn create umi
```

如果出现<font color="red">文件名、目录名或卷标语法不正确。</font>则处理方式同上文

+   选择模板
+   是否使用TS
+   选择插件
+   安装依赖
+   启动

![image-20220607180103394](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220607180103.png)

## 文件结构

![image-20220607173650242](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220607173650.png)

### mock

mock文件，基于express

### layouts基础布局文件

![image-20220607180154982](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220607180155.png)

### locales国际化文件（多语言）

![image-20220607180311725](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220607180311.png)

使用方式 

```typescript
import { formatMessage } from 'umi-plugin-locale';
formatMessage({ id: 'index.start' })
```

### .umirc.ts配置文件

如果是自建目录或者antdpro模板，则也可以在config/config.ts中进行配置

注意：如果配置了 routes，则优先使用配置式路由，且约定式路由会不生效

**umi2和umi3版本在配置文件上的写法有所不同，umi3的配置文件结构是是打平了的**

umi2例：

![image-20220620190619956](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220620190620.png)

![image-20220620190604058](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220620191015.png)

umi3例：

![image-20220620190704051](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220620190704.png)

![image-20220620190652481](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220620190652.png)

### config/config.ts配置文件

作用同.umirc.ts,但是.umirc.ts的优先级更高

### app.ts

运行时配置写在这里

![image-20220615105251493](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220615105259.png)

### .editorconfig编辑器配置

用于配置编辑规则 如缩进，编码等规则

### .prettierignore美化格式忽略文件

用于忽略那些不需要格式化的文件 

# 基础

## ant-design-pro

**一套基础功能完善的开发模板**

执行yarn create umi后选择ant-design-pro

相比基础demo，这个模板有很多不同的文件，这些文件在基础demo上同样是可以创建并使用的

文件目录

![image-20220608165920560](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220608165948.png)

+   **src/access.ts**

权限数据文件

+   **src/.umi**

临时文件夹

+   **src/manifest.json**

项目信息文件

+   **jest.config.js**

jest测试框架配置文件

+   **.stylelintrc.js**

样式表语法检查文件

## 开发

为了一步步的使用学习，使用一个空白的模板作为demo

使用命令 yarn create @umijs/umi-app 创建一个干净项目模板

![image-20220617164109158](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220617164117.png)

![image-20220617164213089](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220617164213.png)

首先在项目文件夹下新建一个config/config.ts作为配置文件，将.umirc.ts中的内容复制到config/config.ts中，删掉.umirc.ts

```typescript
import { defineConfig } from 'umi';

export default defineConfig({
    nodeModulesTransform: {
        type: 'none',
    },
    routes: [
        { path: '/', component: '@/pages/index' },
    ],
    fastRefresh: {},
});
```

在src下新建一个app.tsx运行时配置文件以待后用

![image-20220617170422658](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220617170422.png)

### @umijs/preset-ui

**一款可以提供可视化面板，对项目进行管理或创建的插件**

```shell
yarn add @umijs/preset-ui -D
```

打开面板

```shell
umi ui
```

如果要在启动项目的同时启动umi-ui，则可以修改或者添加启动命令

![image-20220620194227972](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220620194228.png)

导入项目

![image-20220608115049694](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220608115102.png)

同样，也可以使用面板进行项目创建

![image-20220615212337841](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220615212337.png)

==注意：使用ui面板创建的项目目前引用的是2.x的umi==

在启动项目的同时启动UI面板

![image-20220624105559638](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624105559.png)

### @umijs/plugin-initial-state

**数据初始化插件**

如何启用？

有 `src/app.ts`或者`src/app.tsx` 并且导出 `getInitialState` 方法时启用。getInitialState是一个异步函数，返回一个Promise对象。在整个应用最开始执行，返回值会作为全局共享的数据。Layout 插件、Access 插件以及用户都可以通过 `useModel('@@initialState')` 直接获取到这份数据。



在app.tsx中导出函数

```typescript
export async function getInitialState(): Promise<{
    text: string;
}> {
    const text: string = await new Promise((resolve) => setTimeout(() => resolve('demo'), 1500))
    return {
        text
    };
}
```

新增src/pages/init-state/index.tsx

```typescript
import styles from './index.less';
import { useModel } from 'umi';

export default function IndexPage() {
    const { initialState, loading, error, refresh, setInitialState } = useModel('@@initialState')
    console.log('initialState', initialState)
    console.log('loading', loading)
    return (
        <div>
            <h1 className={styles.title}>{initialState?.text}</h1>
        </div>
    );
}
```

在config/config.ts下配置路由

![image-20220617171903455](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220617171903.png)

启动

![image-20220617171739907](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220617171740.png)

### @umijs/plugin-access

**权限定义与控制管理插件**

如何启用？

存在有 `src/access.ts`文件 时启用。该文件需要默认导出一个方法，导出的方法会在项目初始化时被执行。该方法需要返回一个对象，对象的每一个值就对应定义了一条权限。

在src/access.ts中编辑

```typescript
export default function (initialState: any) {
    const { role } = initialState;

    return {
        admin: role === 'admin',
        ableRead:true,
    };
}
```

其中 `initialState` 是通过初始化状态插件 `@umijs/plugin-initial-state` 提供的数据，你可以使用该数据来初始化你的用户权限。

修改app.tsx 下的导出方法getInitialState

```typescript
export async function getInitialState(): Promise<{
    text: string;
    role:string;
}> {
    const text: string = await new Promise((resolve) => setTimeout(() => resolve('demo'), 1500))
    return {
        text,
        role:'admin'
    };
}
```

新增一个demo页面

```typescript
import { useAccess, Access } from 'umi';
export default function IndexPage() {
    const access = useAccess()
    return (
        <div>
            <Access
                accessible={access.ableRead}
                fallback={<h1>Can not read.</h1>}
            >
                <h1>ableRead</h1>
            </Access>
        </div>
    );
}
```

![image-20220624115207478](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624115207.png)

![image-20220624115231299](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624115231.png)

### @umijs/plugin-dva

 @umijs/preset-react插件集集成，无需额外下载，umi3中自动启用，但详细配置需要额外在配置文件中管理

符合以下规则的文件会被认为是 model 文件，

-   `src/models` 下的文件 **允许子文件夹嵌套**
-   `src/pages` 下，子目录中 models 目录下的文件
-   `src/pages` 下，所有 model.ts 文件(不区分任何字母大小写)

用例：

在app.tsx中进行运行时配置（动态配置初始状态）

```typescript
export const dva = {
    config: {
        onError(err: any) {
            err.preventDefault();
            alert(err.message);
        },
        initialState: {
            dvaModel: {
                text: 'hi umi + dva!!!!',
            },
        },
    },
};
```

在src下创建 models/dvaModel/dvaModel.ts

```typescript
import { Reducer, } from 'umi';

export interface DemoModelState {
    text: string;
}

export interface DemoModelType {
    namespace: 'dvaModel';
    state: DemoModelState;
    reducers: {
        save: Reducer<DemoModelState>;
    };
}

const DemoModel: DemoModelType = {
    namespace: 'dvaModel',
    state: {
        text: 'init',
    },
    reducers: {
        save(state, action) {
            return {
                ...state,
                ...action.payload,
            };
        },
    },
};

export default DemoModel;
```

函数式组件使用

```typescript
import React, { FC } from 'react';
import { DemoModelState, ConnectProps, connect } from 'umi';
interface PageProps extends ConnectProps {
    dvaModel: DemoModelState;
}

const IndexPage: FC<PageProps> = ({ dvaModel }) => {
    const { text } = dvaModel;
    return <div>text:{text}</div>;
};

export default connect(
    ({ dvaModel }: { dvaModel: DemoModelState }) => ({
        dvaModel,
    }),
)(IndexPage);
```

类组件使用

```typescript
import React, { Component } from 'react'
import { connect, DemoModelState, ConnectProps } from 'umi';
interface IProps extends ConnectProps {
    dvaModel: DemoModelState;
}
class IndexPage extends Component<any, IProps> {
    constructor(props: any) {
        super(props)
    }
    componentDidMount() {
        console.log(this.props)
    }
    render(): React.ReactNode {
        return <h1>text:{this.props?.dvaModel?.text}</h1>
    }
}
export default connect(
    ({ dvaModel }: { dvaModel: DemoModelState }) => ({
        dvaModel,
    }),
)(IndexPage);
```

![image-20220624105454102](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624105501.png)

### @umijs/plugin-layout

布局插件

两种方式进行配置

**构建时配置**

在config/config.ts中配置

```typescript
  layout: {
    name: 'Ant Design',
    locale: true,
    layout: 'side',
  },
```

**运行时配置**

在src.ts/app.ts中进行配置

官方示例：

```typescript
import React from 'react';
import {
  BasicLayoutProps,
  Settings as LayoutSettings,
} from '@ant-design/pro-layout';

export const layout = ({
  initialState,
}: {
  initialState: { settings?: LayoutSettings; currentUser?: API.CurrentUser };
}): BasicLayoutProps => {
  return {
    rightContentRender: () => <RightContent />,
    footerRender: () => <Footer />,
    onPageChange: () => {
      const { currentUser } = initialState;
      const { location } = history;
      // 如果没有登录，重定向到 login
      if (!currentUser && location.pathname !== '/user/login') {
        history.push('/user/login');
      }
    },
    menuHeaderRender: undefined,
    ...initialState?.settings,
  };
};
```



layout插件具有一系列的路由扩展属性

```typescript
//config/route.ts
export const routes: IBestAFSRoute[] = [
  {
    path: '/welcome',
    component: 'IndexPage',
    name: '欢迎', // 兼容此写法
    icon: 'testicon',
    // 更多功能查看
    // https://beta-pro.ant.design/docs/advanced-menu
    // ---
    // 新页面打开
    target: '_blank',
    // 不展示顶栏
    headerRender: false,
    // 不展示页脚
    footerRender: false,
    // 不展示菜单
    menuRender: false,
    // 不展示菜单顶栏
    menuHeaderRender: false,
    // 权限配置，需要与 plugin-access 插件配合使用
    access: 'canRead',
    // 隐藏子菜单
    hideChildrenInMenu: true,
    // 隐藏自己和子菜单
    hideInMenu: true,
    // 在面包屑中隐藏
    hideInBreadcrumb: true,
    // 子项往上提，仍旧展示,
    flatMenu: true,
  },
];
```

+   name

如果想让对应的页面显示在侧边栏中，则需要在路由中进行扩展配置，加入name属性

![image-20220624163026931](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624163026.png)

![image-20220624163108096](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220624163108.png)

top布局 layout:'top'

![image-20220627163434937](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627163435.png)

**layout运行时完整效果**：

layout运行时配置支持antd ProLayout的所有API

地址https://procomponents.ant.design/components/layout/

修改app.tsx

```typescript
```

#### title

**ReactNode**

左上角与页签 的 title

#### logo

**ReactNode | ()=> ReactNode**

左上角 logo 渲染

```typescript
设置 logo 为网络地址  logo="https://avatars1.githubusercontent.com/u/8186664?s=460&v=4"
设置 logo 为组件  logo={<img src="https://avatars1.githubusercontent.com/u/8186664?s=460&v=4"/>}
设置 logo 为 false 不显示 logo  logo={false}
设置 logo 为 方法  logo={()=> <img src="https://avatars1.githubusercontent.com/u/8186664?s=460&v=4"/> }
```



#### pure

**boolean**

是否删除掉所有的自带界面

简约模式 设置了之后不渲染的任何 layout 的东西，但是会有 context，可以获取到当前菜单。

#### loading

**boolean**

页面加载状态

![image-20220627171704754](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627171705.png)

#### menuHeaderRender

**ReactNode | (logo,title)=>ReactNode**

渲染 logo 和 title

![image-20220627174357929](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627174358.png)

#### menuFooterRender

**(menuProps)=>ReactNode**

在 layout 底部渲染一个块

**注意：仅在layout布局为侧边布局或混合布局时生效**

menuProps  左侧导航栏的属性

![image-20220627174041943](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627174042.png)

![](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627173302.png)

#### onMenuHeaderClick

**(e: React.MouseEvent<HTMLDivElement>) => void**

menu 菜单的头部点击事件

可以在这里实现点击返回首页的功能

#### menuExtraRender

**(menuProps)=>ReactNode**

在菜单标题的下面渲染一个区域

**注意：仅在layout布局为侧边布局或混合布局时生效**

![image-20220627174938403](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627174938.png)

#### onTopMixMenuHeaderClick

**(e: React.MouseEvent<HTMLDivElement>) => void**

mix 模式下顶部栏的头部点击事件

#### contentStyle

**CSSProperties**

layout 的内容区 style

#### layout

**side | top|mix**

layout 的菜单模式,side：右侧导航，top：顶部导航 mix 混合

mix模式

![image-20220627195442903](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627195443.png)

#### contentWidth

**Fluid | Fixed**

layout 的内容模式,Fluid：自适应，Fixed：定宽 1200px

#### navTheme

**light | dark**

导航栏主题

#### actionRef

**MutableRefObject<ActionType>**

layout 的常见的操作，比如刷新菜单

或者是添加一些自定义操作 

使用ProLayout组件时通过ref调用 actionRef.current.func();

#### headerTheme

**light | dark**

顶部导航主题 ，mix模式下生效

#### fixedHeader

**boolean**

是否固定 header 到顶部

#### fixSiderbar

**boolean**

是否固定导航

#### breakpoint

**Enum { 'xs', 'sm', 'md', 'lg', 'xl', 'xxl' }**

触发响应式布局的断点

#### menu

**menuConfig**

menu配置项

+   locale boolean menu 是否使用国际化，还需要 formatMessage 的配合
+   defaultOpenAll boolean 默认打开所有的菜单项，要注意只有 layout 挂载之前生效，异步加载菜单是不支持的
+   ignoreFlatMenu boolean 是否忽略手动折叠过的菜单状态，结合 defaultOpenAll 可实现折叠按钮切换后，同样可以展开所有子菜单
+   type    sub | group  菜单的类型
+   autoClose boolean 选中菜单是否自动关闭菜单
+   loading boolean 菜单是否正在加载中
+   onLoadingChange   (loading)=>void  菜单的加载状态变更
+   request  (params,defaultMenuDat) => Promise<MenuDataItem[]>   远程加载菜单的方法，会自动的修改 loading 状态

#### splitMenus

**boolean**

自动切割菜单,可以把第一级的菜单放置到顶栏中

#### iconfontUrl

**URL**

使用 IconFont 的图标配置

#### locale

**zh-CN | zh-TW | en-US**

当前 layout 的语言设置

#### settings

layout设置

#### siderWidth

**number**

侧边栏宽度

#### headerHeight

**number**

头部高度

![image-20220627210443960](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220627210444.png)

#### defaultCollapsed

**boolean**

默认的菜单的收起和展开，会受到 breakpoint 的影响，breakpoint=false 生效

#### collapsed

**boolean**

控制菜单的收起和展开

#### onCollapse

**(collapsed: boolean) => void**

菜单的折叠收起事件

#### onPageChange

**(location: Location) => void**

页面切换事件

#### headerRender

**(props: BasicLayoutProps) => ReactNode**

自定义头的 render 方法

![image-20220629154311426](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220629154318.png)

#### headerTitleRender

**(logo,title,props)=>ReactNode**

自定义头标题的方法,mix 模式下生效

#### headerContentRender

**(props: BasicLayoutProps) => ReactNode**

自定义头内容的方法

**注意：top布局下覆盖导航栏**

#### rightContentRender

**(props: HeaderViewProps) => ReactNode**

自定义头右部的 render 方法

#### collapsedButtonRender

**(collapsed: boolean) => ReactNode**

自定义 collapsed button 的方法

![image-20220629154939802](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220629154939.png)

#### footerRender

**(props: BasicLayoutProps) => JSX.Element | false**

 自定义页脚的 render 方法

#### pageTitleRender

**(props, defaultPageTitle?, info?) => string**

自定义页面标题的显示方法

#### menuRender

**(props: HeaderViewProps) => ReactNode**

自定义菜单的 render 方法

#### postMenuData

**(menuData: MenuDataItem[]) => MenuDataItem[]**

在显示前对菜单数据进行查看，修改不会触发重新渲染

#### menuItemRender

**(itemProps: MenuDataItem, defaultDom: React.ReactNode, props: BaseMenuProps) => ReactNode**

自定义菜单项的 render 方法

#### subMenuItemRender

**(itemProps: MenuDataItem) => ReactNode**

自定义拥有子菜单菜单项的 render 方法

#### menuDataRender

**(menuData: MenuDataItem[]) => MenuDataItem[]**

menuData 的 render 方法，用来自定义 menuData

#### breadcrumbRender

(route)=>route

自定义面包屑的数据

#### disableMobile

**boolean**

禁止自动切换到移动页面

#### ErrorBoundary

**ReactNode**

自带了错误处理功能，防止白屏，ErrorBoundary=false 关闭默认错误边界

#### links

**ReactNode[]**

显示在菜单右下角的快捷操作

top布局下不生效

![image-20220629160604036](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220629160604.png)

#### waterMarkProps

**waterMarkProps**

配置水印，水印是 PageContainer 的功能，layout 只是透传给 PageContainer

#### disableContentMargin

**boolean**

content的 margin

## 其它

### wrappers

导航守卫|页面容器

可以在已有的页面上添加一层包裹容器，进入该路由将优先展示此容器，原有的页面将作为children传入。

wrappers可以在加载原页面之前进行一系列的操作，比如当用户进入页面时，判断其有无登录，控制其是否可以进入原页面。对于中台项目，我们可以在app.ts的getInitialState中进行登录校验，控制其是否跳转登录页，但对于一些其它类型的网站，比如电商，或者博客网站之类的，它可能既有需要登录校验的页面，比如购物车或者个人中心页面，也有无需校验的页面，比如商品页，博文页等，这时候就可以使用wrappers对需要登录的页面进行包装。



或者当有某一部分页面具有相似的页面结构与样式时也可以将这一相似的部分提取到wrappers中



在src下新增wrappers/wrappers.tsx

```typescript
export default (props: any) => {
    const isLogin = false;
    if (isLogin) {
        return <div>{props.children}</div>;
    } else {
        return <div>未登录</div>;
    }
};
```



在routes中需要包裹的页面加上wrappers属性

![image-20220629183535367](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220629183535.png)

![image-20220629183447378](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20220629183447.png)
