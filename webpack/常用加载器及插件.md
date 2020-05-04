<div style="text-align: center; font-weight: 700; font-size: 2em;">webpack常用加载器及插件</div>

# 一、加载器
## 1.1 json-loader
说明：

解析.json格式文件，将文件内容解析为json对象。

下载：
```
npm i -D json-loader
```
配置：
```js
module: {
  loaders: [
    {
      test: /\.json$/,
      loader: "json-loader"
    }
  ]
}
```

## 1.2 babel-loader
说明：

Babel其实是几个模块化的包，其核心功能位于称为babel-core的npm包中，不过webpack把它们整合在一起使用，但是对于每一个你需要的功能或拓展，你都需要安装单独的包（用得最多的是解析Es6的babel-preset-es2015包和解析JSX的babel-preset-react包）。

下载：
```
npm i -D babel-core babel-loader babel-preset-es2015 babel-preset-react
```
配置：
```js
// webpack.config.js文件中
module: {
  loader: [
    {
      test: /\.js$/,
      exclude: /node_modules/, // node_modules中文件不需要解析
      loader: "babel-loader"
    }
  ]
}
//.babelrc文件中
{
  "presets": ["es2015"]
}
```

## 1.3 style-loader & css-loader
说明：

css-loader使你能够使用类似@import 和 url(...)的方法实现 require()的功能,style-loader将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。

下载：
```
npm i -D css-loader style-loader
```
配置：
```js
module: {
  loaders: [
    {
      test: /\.css$/,
      loader: "style-loader!css-loader?modules" // 其中modules可以使css代码模块化
    }
  ]
}
```

## 1.4 postcss-loader
说明：

PostCSS，它可以帮助你的CSS实现更多的功能,举例来说如何使用PostCSS，我们使用PostCSS来为CSS代码自动添加适应不同浏览器的CSS前缀。

下载：
```
npm i -D postcss-loader autoprefixer
```
配置：
```js
// webpack.config.js文件中
module: {
  loaders: [
    {
      test: /\.css$/,
      loader: "style-loader!css-loader?modules!postcss-loader" // 
    }
  ]
}
// postcss.config.js文件中
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```

## 1.5 url-loader
说明：

解析图片资源，使用前请确保file-loader已下载。

下载：
```
npm i -D url-loader
```
配置：
```js
module: {
  loaders: [
    {
      test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
      loader: 'url-loader',
      options: {
        limit: 10000,
        name: 'img/[name].[hash:7].[ext]'
      }
    }
  ]
}
```

## 1.6 sass-loader
说明：

解析scss语法，使用前确保css-loader和style-loader已下载。

下载：
```
npm i -D sass-loader node-sass
```
配置：
```js
module: {
  loaders: [
    {
      test: /\.scss$/,
      // 此处采用loaders和用loader然后用！拼接效果相同
      loaders: ["style-loader", "css-loader", "sass-loader"]
    }
  ]
}
```

## 1.7 zepto
说明：

如需使用zepto需要安装exports-loader和script-loader对zepto处理，其他未模块化的插件也是同样处理逻辑，如果不直接使用zepto可以使用已经模块化的插件webpack-zepto。

下载：
```
npm i -D zepto exports-loader script-loader
```
配置：
```js
module: {
  loaders: [
    {
      test: require.resolve('zepto'),
      loader: 'exports-loader?window.Zepto!script-loader'
    }  
  ]
}
```

## 1.8 jquery
说明：

使webpack环境使用jQuery。

下载：
```
npm i -D jquery
```
配置：
```
// 直接import $ from 'jquery'便可使用
// 如需全局引入直接在ProvidePlugin插件配置全局变量即可
```

# 二、插件
## 2.1 BannerPlugin
说明：

给生成出来的文件以注释的形式添加标语，如添加版权信息。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.BannerPlugin("@版权所有，盗版必究")
]
```

## 2.2 html-webpack-plugin
说明：

通过模版html文件自动生成html代码。

下载：
```
npm i -D html-webpack-plugin
```
配置：
```js
// 引入插件
var HtmlWebpackPlugin = require('html-webpack-plugin')
plugins: [
  new HtmlWebpackPlugin({
    template: __dirname + "/src/index.html", // 模板文件路径，支持加载器，比如 html!./index.html
    minify: { // 压缩生成后的html
      removeComments:true,    //移除HTML中的注释
      collapseWhitespace:true    //删除空白符与换行符
    }
  })
]
// 其他参数
// title: 用于生成的HTML文件的标题。
// filename: 用于生成的HTML文件的名称，默认是index.html。你可以在这里指定子目录。
// inject: true | 'head' | 'body' | false ,注入所有的资源到特定的 template 或者 templateContent 中，如果设置为 true 或者 body，所有的 javascript 资源将被放置到 body 元素的底部，'head' 将放置到 head 元素中。
// favicon: 添加特定的 favicon 路径到输出的 HTML 文件中。
// hash: true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS 文件，对于解除 cache 很有用。
// cache: true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
// showErrors: true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
// chunks: 允许只添加某些块 (比如，仅仅 unit test 块)
// chunksSortMode: 允许控制块在添加到页面之前的排序方式，支持的值：'none' | 'default' | {function}-default:'auto'
// excludeChunks: 允许跳过某些块，(比如，跳过单元测试的块)
```

## 2.3 OccurrenceOrderPlugin
说明：

为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin()
]
```

## 2.4 UglifyJsPlugin
说明：

压缩和混淆js代码。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false
    },
    except: ['$super', '$', 'exports', 'require'] // 排除不需要混淆的关键字
  })
]
```

## 2.5 extract-text-webpack-plugin
说明：

分离js和css。

下载：
```
npm i -D extract-text-webpack-plugin
```
配置：
```js
// 引入插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')
module: {
  loaders: [
    {
      test: /\.css$/,
      loader: ExtractTextPlugin.extract({ // 处理css
        fallback: 'style-loader',
        use: 'css-loader?modules!postcss-loader'
      })
    }
  ]
}
plugins: [
  new ExtractTextPlugin("[name]-[contenthash].css"), // 提js中的css到对于.css文件
]
```

## 2.6 webpack-dev-server
说明：

本地服务器。

下载：
```
npm i -D webpack-dev-server
```
配置：
```js
// webpack.config.js中
devServer: {
  contentBase: "./dist", // 服务器访问的目录
  // 在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html
  historyApiFallback: true,
  inline: true, // 热加载
  port:9090 //端口你可以自定义
}
// package.json中
"scripts": {
  "server": "webpack-dev-server" // 如此在控制台可直接使用npm run server开启服务
}
```

## 2.7 CommonsChunkPlugin
说明：

提取多个入口文件的公共脚本部分，然后生成一个common.js来方便多页面之间进行复用。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.optimize.CommonsChunkPlugin('common')
]
```

## 2.8 ProvidePlugin
说明：

把一个全局变量插入到所有的代码中,支持jQuery plugin的使用;使用ProvidePlugin加载使用频率高的模块。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.ProvidePlugin({
    $: "jquery",
    jQuery: "jquery",
    "window.jQuery": "jquery"
  })
]
```

## 2.9 NoErrorsPlugin
说明：

允许错误不打断程序。

配置：
```js
// 引入webpack
var webpack = require('webpack')
plugins: [
  new webpack.NoErrorsPlugin()
]
```

## 2.10 transfer-webpack-plugin
说明：

把指定文件夹下的文件复制到指定的目录。

下载：
```
npm i -D transfer-webpack-plugin
```
配置：
```js
// 引入插件
var path = require('path')
var TransferWebpackPlugin = require('transfer-webpack-plugin')
plugins: [
  new TransferWebpackPlugin([
    {from: 'www'}
  ], path.resolve(__dirname,"src"))
]
```

# 三、resolve
说明：

其他解决方案配置。
```js
resolve: {
  // root: 'E:/test/src', //绝对路径, 查找module的话从这里开始查找(可选)
  extensions: ['.js', '.html', '.css', '.scss'], //自动扩展文件后缀名，意味着我们require模块可以省略不写后缀名
  alias: { //模块别名定义，方便后续直接引用别名，无须多写长长的地址//后续直接 require('AppStore') 即可
    main : 'src/main.js'
  }
}
```