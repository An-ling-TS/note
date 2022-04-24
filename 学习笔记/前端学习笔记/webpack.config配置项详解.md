# webpack.config配置项详解

[toc]

```js
const path = require('path');

module.exports = {
  mode: "production", // "production" | "development" | "none"  // 告诉webpack是生产环境还是开发环境.
  entry: "./app/entry", // string | object | array  // 默认 ./src
  // 入口起点，可以指定多个入口起点
  output: {
    // 输出，只可指定一个输出配置
    path: path.resolve(__dirname, "dist"), // string
    //  所有输出文件所在的目录
    // 必须是绝对路径(use the Node.js path module)
    filename: "bundle.js", // string    // 输出文件的名称
    publicPath: "/assets/", // string    // 相对于HTML页面解析的输出目录的url
    library: "MyLibrary", // string,
    //导出库的名称
    libraryTarget: "umd", // universal module definition    // the type of the exported library
  module: { //如何处理项目中不同类型的模块
    rules: [ //用于规定在不同模块被创建时如何处理模块的规则数组
      {
        test: /\.jsx?$/, //匹配特定文件的正则表达式或正则表达式数组
        include: [ //规则适用的范围
          path.resolve(__dirname, "app")
        ],
        exclude: [ //规则排除的范围
          path.resolve(__dirname, "app/demo-files")
        ],
        issuer: { test, include, exclude },
        enforce: "pre",
        enforce: "post",
        loader: "babel-loader", //加载器
        options: { //转义
          presets: ["es2015"]
        },
      },
      {
        test: /\.html$/,
        use: [
          "htmllint-loader",
          {
            loader: "html-loader",
            options: {
              /* ... */
            }
          }
        ]
      },
    ],
  },
  resolve: {
    // 解决模块请求的选项
    modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    extensions: [".js", ".json", ".jsx", ".css"], // 用到的扩展
    alias: { //模块名称别名列表
      "module": path.resolve(__dirname, "app/third/module.js"),
    },
  },
  performance: {
    hints: "warning", // enum    maxAssetSize: 200000, // int (in bytes),
    maxEntrypointSize: 400000, // int (in bytes)
    assetFilter: function(assetFilename) {
      return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
  },
  devtool: "source-map", // 代码映射，增强调试，以构建速度为代价
  context: __dirname, // string (绝对路径) //webpack的主目录
  target: "web",
  externals: ["react", /^@angular\//],  //不要跟踪/捆绑这些模块，而是在运行时从环境中请求它们
  serve: { 
    port: 1337,
    content: './dist',
    // ...
  },
  stats: "errors-only",  // 让你精确地控制被显示的包信息
  devServer: {
    proxy: { // 代理url
      '/api': 'http://localhost:3000'
    },
    contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
    compress: true, //支持gzip压缩
    historyApiFallback: true, // html在404，对象为多个路径
    hot: true, // 热模块替换。取决于HotModuleReplacementPlugin
    https: false, 
    noInfo: true, 
    // ...
  },
  plugins: [ //webpack插件列表
    // ...
  ],
```

**1.entry**

entry可以是个字符串或数组或者是对象。 
当entry是个字符串的时候，用来定义入口文件：

```
entry: './js/main.js'
```

 

 

当entry是个数组的时候，里面同样包含入口js文件，另外一个参数可以是用来配置webpack提供的一个静态资源服务器，webpack-dev-server。webpack-dev-server会监控项目中每一个文件的变化，实时的进行构建，并且自动刷新页面：

```
entry: [
     'webpack/hot/only-dev-server',
     './js/app.js'
]
```

 

当entry是个对象的时候，我们可以将不同的文件构建成不同的文件，按需使用，比如在我的hello页面中只要\引入hello.js即可：

```
 entry: {
     hello: './js/hello.js',
     form: './js/form.js'
 }
```

 

------

**2.output** 
output参数是个对象，用于定义构建后的文件的输出。其中包含path和filename：

```
output: {
     path: './build',
     filename: 'bundle.js'
 }
```

 

当我们在entry中定义构建多个文件时，filename可以对应的更改为[name].js用于定义不同文件构建后的名字。

------

**3.module** 
关于模块的加载相关，我们就定义在module.loaders中。这里通过正则表达式去匹配不同后缀的文件名，然后给它们定义不同的加载器。比如说给less文件定义串联的三个加载器（！用来定义级联关系）：

```
module: {
    loaders: [
        { test: /\.js?$/, loaders: ['react-hot', 'babel'], exclude: /node_modules/ },
        { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader'},
        { test: /\.css$/, loader: "style!css" },
        { test: /\.less/, loader: 'style-loader!css-loader!less-loader'}
    ]
}
```

此外，还可以添加用来定义png、jpg这样的图片资源在小于10k时自动处理为base64图片的加载器：

```
{ test: /\.(png|jpg)$/,loader: 'url-loader?limit=10000'}
```

 

给css和less还有图片添加了loader之后，我们不仅可以像在node中那样require js文件了，我们还可以require css、less甚至图片文件：

```
require('./bootstrap.css');
 require('./myapp.less');
 var img = document.createElement('img');
 img.src = require('./glyph.png');
```

 

但是需要知道的是，这样require来的文件会内联到 js bundle中。如果我们需要把保留require的写法又想把css文件单独拿出来，可以使用下面提到的[extract-text-webpack-plugin]插件。 
在上面示例代码中配置的第一个loaders我们可以看到一个叫做[React](http://lib.csdn.net/base/react)-hot的加载器。我的项目是用来学习[react](http://lib.csdn.net/base/react)写相关代码的，所以配置了一个react-hot加载器，通过它，可以实现对react组件的热替换。我们已经在entry参数中配置了`webpack/hot/only-dev-server`,所以我们只要在启动webpack开发服务器时开启–hot参数，就可以使用react-hot-loader了。在package.json文件中这样定义：

```
"scripts": {
     "start": "webpack-dev-server --hot --progress --colors",
     "build": "webpack --progress --colors"
 }
```

 

------

**4.resolve** 
webpack在构建包的时候会按目录的进行文件的查找，resolve属性中的extensions数组中用于配置程序可以自行补全哪些文件后缀：

```
 resolve:{
     extensions:['','.js','.json']
 }
```

 

然后我们想要加载一个js文件时，只要require(‘common’)就可以加载common.js文件了。

------

**6.externals** 
当我们想在项目中require一些其他的类库或者API，而又不想让这些类库的源码被构建到运行时文件中，这在实际开发中很有必要。此时我们就可以通过配置externals参数来解决这个问题：

```
 externals: {
     "jquery": "jQuery"
 }
```

 

这样我们就可以放心的在项目中使用这些API了：var [jQuery](http://lib.csdn.net/base/jquery) = require(“[jquery](http://lib.csdn.net/base/jquery)”);

------

**7.context** 
当我们在require一个模块的时候，如果在require中包含变量，像这样：

```
require("./mods/" + name + ".js");
```

 

那么在编译的时候我们是不能知道具体的模块的。但这个时候，webpack也会为我们做些分析工作：

1.分析目录：’./mods’； 
2.提取正则表达式：’/^.*.js$/’；

于是这个时候为了更好地配合wenpack进行编译，我们可以给它指明路径，像在cake-webpack-config中所做的那样（我们在这里先忽略abcoption的作用）：

```
 var currentBase = process.cwd();
 var context = abcOptions.options.context ? abcOptions.options.context : 
 path.isAbsolute(entryDir) ? entryDir : path.join(currentBase, entryDir);
```