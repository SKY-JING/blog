<div style="text-align: center; font-weight: 700; font-size: 2em;">webpack项目搭建</div>

# 一、准备工作
创建如下文件目录结构：
```
|-- webpack3
|  |-- build                            // 项目后台服务文件
|  |-- config                           // 项目配置文件
|  |-- src                              // 项目资源文件（工作区域）
|  |-- static                           // 项目最后生成图片、视频、字库等资源文件存储目录
|  |-- package.json                     // 项目基本信息
```

# 二、使用express构建服务器
Express 是一个自身功能极简，完全是由路由和中间件构成一个的 web 开发框架：从本质上来说，一个Express 应用就是在调用各种中间件。更多信息请查看[express](http://www.expressjs.com.cn/)
## 2.1 下载
```
npm i -D express
```
## 2.2 使用
在build文件下新建一个dev-server.js
```js
// 引入express
var express = require('express')
// 实例化express
var app = express()
// 创建一个简单的路由，当访问服务器首页时显示hello world
app.get('/', function(req, res){
  res.send('hello world')
})
// 服务监听8080端口
var server = app.listen(8080, function () {
  var host = server.address().address
  var port = server.address().port
  console.log('Example app listening at http://%s:%s', host, port)
})
```
在package.json中加入执行文件入口
```js
"scripts": {
  "dev": "node build/dev-server.js",
  "start": "node build/dev-server.js"
}
```
现在我们直接在终端输入npm start,然后在浏览器输入localhost:8080便可看到浏览器输出hello world。

# 三、express引入webpack
通过使用webpack-dev-middleware插件可以直接将webpack引入到express中，更多相关资料可查看[webpack-dev-middleware](https://www.npmjs.com/package/webpack-dev-middleware)
## 3.1 下载
由于我们需要使用webpack，此处将webpack和webpack-dev-middleware一起下载
```
npm i -D webpack webpack-dev-middleware
```
## 3.2 使用
使用webpack前，我们先在src下新建一个main.js文件用于测试
```js
// main.js
console.log('hello world')
```
现在我们来修改上面创建的dev-server.js
```js
// 引入path
var path = require('path')
// 引入express
var express = require('express')
// 引入webpack
var webpack = require('webpack')
// 引入webpack-dev-middleware
var webpackMiddleware = require('webpack-dev-middleware')
// 实例化epress
var app = express()
// express使用webpack
app.use(webpackMiddleware(webpack({
  entry: './src/main.js', // 入口文件
  output: {
    path: path.resolve(__dirname, '../dist'), // 出口文件本地目录
    filename: '[name].js', // 出口文件名称
    publicPath: '/' // 出口文件公共目录
  }
}),{ // webpack-dev-middleware配置信息，核心配置为服务器访问路径publicPath，其他可有可无
  noInfo: false, // 是否控制台显示警告和错误
  quiet: true, // 是否停止显示控制台信息
  lazy: true, // 是否开启懒惰编译（每次请求页面时编译而不是一次性全部编译）
  watchOptions: { // lazy为false时可用
    aggregateTimeout: 300,
    poll: true
  },
  publicPath: '/', // 公共访问路径
  // Web服务器的索引路径默认为index.html,如果index.html没有定义服务器无法响应请求的根URL
  index: 'index.html',
  headers: { 'X-Custom-Header': 'yes' }, // 自定义头
  mimeType: { 'text/html': [ 'phtml' ] }, // 添加自定义MIME /扩展映射
  stats: { // 格式化数据选项
    colors: true
  },
  reporter: null, // 提供一个定制的报表来改变日志显示的方式。
  serverSideRender: false // 关闭服务器端呈现模式。有关更多信息，请参见服务器端渲染部分。
}))
var server = app.listen(8080, function () {
  var host = server.address().address
  var port = server.address().port
  console.log('Example app listening at http://%s:%s', host, port)
})
```
现在再次执行npm start,现在我们浏览器输入localhost:8080/main.js可以看到webpack对main.js进行了打包

# 四、分离webpack和express
为了使代码看上去更加清晰，我们先将dev-server.js中的webpack配置部分提取出来，在build中新建webpack.base.conf.js
```js
// webpack.base.conf.js
var path = require('path')
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js'] // 配置此项可使之后require引入js文件时直接使用文件名便可
  }
}
```
修改后的dev-server文件
```js
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.base.conf')
var app = express()
var compiler = webpack(webpackConfig)
// 此处去掉了前面说明的很多不需要的参数
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
app.use(devMiddleware)
// 去掉了打印的信息，单纯只监听端口
var server = app.listen(8080)
```
现在运行npm start，浏览器输入localhost:8080/main.js和前面效果一致

# 五、webpack引入常用loader
## 5.1 url-loader
由于url-loader依赖file-loader，所以需下载此两个插件。
```
npm i -D url-loader file-loader
```
下载插件后编辑webpack.base.conf.js
```js
var path = require('path')
var utils = require('./utils')
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js']
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000, // 限制资源大小为10000字节
          name: utils.assetsPath('img/[name].[hash:7].[ext]') // 设置静态资源存储路径
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|acc)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
// 上面的正则表达式/\.(png|jpg?g|gif|svg)(\?.*)?$/为了识别.png、.jpg、.jpeg、.gif、.svg格式的文件和带有版本控制信息的图片资源，如:.png?v=1.0
```
由于图片、视频、字库等静态资源文件都需要设置存储路径，故而在build中新建一个utils.js文件用于存放此静态路径拼接的公共方法
```js
// utils.js
var path = require('path')
exports.assetsPath = function (_path) {
  // 由于windows和linux文件路径连接符不一样，所以采用posix以兼容的方式拼接路径
  return path.posix.join('static', _path)
}
```
## 5.2 babel-loader
babel-loader用于解析es6等功能，在webpack常见加载器和插件一文有说明。
```
npm i -D babel-core babel-loader babel-plugin-transform-runtime babel-preset-env babel-preset-stage-2 babel-register
```
修改webpack.base.conf.js
```js
var path = require('path')
var utils = require('./utils')
// 自定义地址拼接方法
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js']
  },
  module: {
    rules: [
      { // 给js文件配置babel-loader
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')] // 必须转化的文件夹有src和test
      },
      {
        test: /\.(png|jpg?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|acc)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```
根目录下新建.babelrc文件用于bebel的其他配置
```js
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "stage-2"
  ],
  "plugins": ["transform-runtime"],
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": ["istanbul"]
    }
  }
}
```
## 5.3 eslint-loader
eslint-loader主要用于监控代码格式，用于统一代码风格。由于在使用eslint时对babel也需要监听，所以将babel-eslint插件也一起下载,更多资料请查看[eslint-loader](https://eslint.org/docs/user-guide/configuring)
```
npm i -D babel-eslint eslint eslint-friendly-formatter eslint-loader eslint-plugin-html eslint-config-standard eslint-plugin-promise eslint-plugin-standard eslint-plugin-import eslint-plugin-node
```
webpack.base.conf.js中配置eslint-loader
```js
var path = require('path')
var utils = require('./utils')
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js']
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'eslint-loader',
        enforce: 'pre', // 强制执行
        include: [resolve('src', resolve('test'))],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpg?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|acc)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```
根目录新建.eslintignore忽略不需要监听的文件
```
build/*.js
config/*.js
```
根目录新建.eslintrc.js用于配置监听的其他参数
```js
module.exports = {
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  extends: 'standard',
  plugins: [
    'html'
  ],
  'rules': {
    'arrow-parens': 0,
    'generator-star-spacing': 0,
    'no-debugger': 0
  }
}
```

# 六、webpack开发环境引入常用插件
由于开发环境和生产环境用到的插件有所区别，所以我们应单独配置，新建一个webpack.dev.conf.js用于配置开发环境插件。
## 6.1 webpack-merge
合并基础部分配置和插件部分配置。
```
npm i -D webpack-merge
```
修改webpack.dev.conf.js
```js
var merge = require('webpack-merge') // 引入插件
var baseWebpackConfig = require('./webpack.base.conf') // 引入基础配置
module.exports = merge(baseWebpackConfig, { // 合并配置
  devtool: '#cheap-module-eval-source-map' // 设置资源打包方式
})

```
修改dev-server.js
```js
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf') // 此处改为引入完整的配置
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
app.use(devMiddleware)
var server = app.listen(8080)
```
## 6.2 html-webpack-plugin
配置页面模版。
```
npm i -D html-webpack-plugin
```
为了使用html-webpack-plugin我们先在根目录新建一个模版文件index.html
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>测试</title>
</head>
<body>
</body>
</html>
```
修改webpack.dev.conf.js
```js
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
// 引入html-webpack-plugin
var HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = merge(baseWebpackConfig, {
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    // 使用插件
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    })
  ]
})
```
此时执行npm strat,然后在浏览器输入localhost:8080便可看到模版文件，main.js也自动装载进入index.html，由于我们main.js写的内容是一个控制台打印，所以F12唤起可看到打印信息。
## 6.3 friendly-errors-webpack-plugin
创建友好错误提示。
```
npm i -D friendly-errors-webpack-plugin
```
修改webpack.dev.conf.js
```js
var webpack = require('webpack')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin') // 引入友好提示插件
module.exports = merge(baseWebpackConfig, {
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    // 在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。这样可以确保输出资源不会包含错误。对于所有资源，统计资料(stat)的 emitted 标识都是 false。
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin() // 使用插件
  ]
})
```

# 七、express使用webpack-hot-middleware实现热加载
下载：
```
npm i -D webpack-hot-middleware
```
为了给所有入口文件配置热加载，我们先修改一下webpack.base.conf.js的entry部分
```js
entry: { // 将入口改为对象形式，方便之后遍历
  app: './src/main.js'
}
```
修改一下index.html
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>测试</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```
然后修改一下main.js
```js
document.getElementById('app').innerText = 'hello world'
```
在webpack.dev.conf.js中给入口文件捆绑热加载相关代码
```js
var webpack = require('webpack')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
// 在此处绑定热加载相关代码
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
module.exports = merge(baseWebpackConfig, {
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```
在build中新建dev-client.js用于监听热加载模块，下载eventsource-polyfill模块用于处理事件资源
```
npm i -D eventsource-polyfill
```
修改dev-client.js
```js
require('eventsource-polyfill')
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true')
hotClient.subscribe(function (event) {
  if (event.action === 'reload') { // 当前事件为reload的时候重新加载页面
    window.location.reload()
  }
})
```
最后在express中使用webpack-hot-middleware模块
```js
// dev-server.js
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf')
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
// 使用热加载模块
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false, //  A function used to log lines, pass false to disable. Defaults to console.log
  // path: "/__what", // The path which the middleware will serve the event stream on, must match the client setting
  heartbeat: 2000 // How often to send heartbeat updates to the client to keep the connection alive. Should be less than the client's timeout setting - usually set to half its value.
})
// 在模版页面发生改变时强制产生reload事件，前面的dev-client.js收到reload事件后便会重载页面
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
// 此处用到了html-webpack-plugin的相关事件，其他类似事件如下：
// 事件名称                                       时机                                   同步/异步
// html-webapck-plugin-before-html-generation  生成htmlPluginData之前触发              async
// html-webpack-plugin-before-html-processing  htmlPluginData插入到html模板之前触发     async
// html-webpack-plugin-alert-asset-tags          验证资源，以及为资源做标记时触发         async
// html-webpack-plugin-after-html-processing   htmlPluginData插入到html模板之后触发    async
// html-webpack-plugin-after-emit              生成html目标文件后触发                    async
// html-webpack-plugin-alert-chunks              验证资源块信息                           sync
app.use(devMiddleware)
app.use(hotMiddleware)
var server = app.listen(8080)
```
现在我们开启服务器后，刚开始打开浏览器会看到hello world,如果此时改变main.js的内容，保存后浏览器内容会跟着改变<hr>

# 八、使用公共配置文件
到上面为止我们已经简单的配置了一个项目，但是这还达不到我们的要求，为了方便后续开发，我们先将一些配置信息提取到config下新建的index.js中，其中配置数据分开成开发环境和生产环境。
```js
var path = require('path') // 引入path模块
module.exports = {
  build: { // 将来用于生产环境
    assetsRoot: path.resolve(__dirname, '../dist'), // 出口文件根目录
    assetsSubDirectory: 'static', // 静态资源路径
    assetsPublicPath: './' // 公共出口路径
  },
  dev: { // 将来用于开发环境
    assetsSubDirectory: 'static', // 静态资源路径
    assetsPublicPath: '/' // 公共出口路径
  }
}
```
修改webpack.base.conf.js
```js
var path = require('path')
var utils = require('./utils')
var config = require('../config') // 引入配置文件
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  entry: {
    app: './src/main.js'
  },
  output: {
    // 根路径是用于build文件时作为出口，所以此参数从config.build中取出
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js', '.json']
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src', resolve('test'))],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpg?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|acc)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```

# 九、配置dev-server.js为开发环境
在config下新建两个文件dev.env.js和prod.env.js用于标志开发环境还是生产环境。
```js
// prod.env.js
module.exports = {
  NODE_ENV: '"production"' // 设置环境变量为production
}
// dev.env.js
var merge = require('webpack-merge')
var prodEnv = require('./prod.env')
module.exports = merge(prodEnv, {
  NODE_ENV: '"development"' // 修改环境变量为development
})
```
修改config下index.js
```js
var path = require('path')
module.exports = {
  build: {
    env: require('./prod.env'), // 生产环境
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './'
  },
  dev: {
    env: require('./dev.env'), // 开发环境
    assetsSubDirectory: 'static',
    assetsPublicPath: '/'
  }
}
```
在webpack.dev.conf.js中配置全局标识process.env。
```js
var webpack = require('webpack')
var config = require('../config') // 引入配置文件
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
module.exports = merge(baseWebpackConfig, {
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({ // 定义全局参数process.env，此后当前环境下各文件都可使用此参数
      'process.env': config.dev.env
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```
修改dev-server.js标识当前环境为开发环境。
```js
var config = require('../config') // 引入config
if(!process.env.NODE_ENV) { // 如果环境变量未定义，设置当前环境为开发环境
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf')
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false,
  heartbeat: 2000
})
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
app.use(devMiddleware)
app.use(hotMiddleware)
var server = app.listen(8080)
```
修改webpack.base.conf.js的output部分
```js
output: {
  path: config.build.assetsRoot,
  filename: '[name].js',
  publicPath: process.env.NODE_ENV === 'production' // 根据环境不同设置不同的公共路径
    ? config.build.assetsPublicPath
    : config.dev.assetsPublicPath
}
```
修改utils.js
```js
var path = require('path')
var config = require('../config') // 使用配置文件
exports.assetsPath = function (_path) { // 根据当前环境设置静态资源路径
  var assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  return path.posix.join(assetsSubDirectory, _path)
}
```
修改.eslintrc.js配置文件
```js
module.exports = {
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  extends: 'standard',
  plugins: [
    'html'
  ],
  'rules': {
    'arrow-parens': 0,
    'generator-star-spacing': 0,
    // 根据环境不同设置是否打开debugger
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0
  }
}
```

# 十、设置编译成功后自动打开浏览器
在dev-server.js中通过opn插件设置浏览器自动打开。
```
npm i -D opn
```
config.js中添加配置
```js
var path = require('path')
module.exports = {
  build: {
    env: require('./prod.env'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './'
  },
  dev: {
    env: require('./dev.env'),
    port: 8080, // 添加端口设置
    autoOpenBrowser: true, // 是否自动打开浏览器
    assetsSubDirectory: 'static',
    assetsPublicPath: '/'
  }
}
```
dev-server.js中配置opn
```js
var config = require('../config')
if(!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var open = require('opn') // 添加opn插件
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf')
var port = process.env.PORT || config.dev.port // 获取端口
var autoOpenBrowser = !!config.dev.autoOpenBrowser // 获取是否自动打开浏览器
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false,
  heartbeat: 2000
})
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
app.use(devMiddleware)
app.use(hotMiddleware)
// 自动打开地址
var uri = 'http://localhost:' + port
// 控制台提示
console.log('> Starting dev server...')
// 使用webpack-dev-middleware当代码编译完成时调用open方法
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  if(autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    open(uri)
  }
})
var server = app.listen(port)
```

# 十一、express配置静态资源路径
由于我们引入url-loader时解析的静态资源全部放到了static文件夹下，所以我们配置静态资源路径直接配置到static便可,修改dev-server.js。
```js
var config = require('../config')
if(!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var open = require('opn')
var path = require('path') // 引入path
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf')
var port = process.env.PORT || config.dev.port
var autoOpenBrowser = !!config.dev.autoOpenBrowser
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false,
  heartbeat: 2000
})
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
app.use(devMiddleware)
app.use(hotMiddleware)
// posix：以兼容的方式拼接路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static')) // express设置静态资源路径
var uri = 'http://localhost:' + port
console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  if(autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    open(uri)
  }
})
var server = app.listen(port)
```

# 十二、使用promise避免回调地狱并导出dev-server模块供其他地方使用
具体promise是啥这里不赘述，有兴趣的同学请查看[promise](https://www.jianshu.com/p/fe5f173276bd)
```js
var config = require('../config')
if(!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var open = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var webpackConfig = require('./webpack.dev.conf')
var port = process.env.PORT || config.dev.port
var autoOpenBrowser = !!config.dev.autoOpenBrowser
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false,
  heartbeat: 2000
})
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
app.use(devMiddleware)
app.use(hotMiddleware)
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))
var uri = 'http://localhost:' + port
// 定义promise方法
var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})
console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  if(autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    open(uri)
  }
  _resolve() // 此处只是对promise的resolve状态测试，没有做任何处理
})
var server = app.listen(port)
// 导出dev-server中的promise模块和服务器的关闭方法
module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```

# 十三、express使用connect-history-api-fallback
connect-history-api-fallback对于单页面处理历史API回退或刷新后访问路由地址变成404很有作用，更多相关请查看[connect-history-api-fallback](https://github.com/bripkens/connect-history-api-fallback)
下载：
```
npm i -D connect-history-api-fallback
```
dev-server.js中加入
```js
app.use(require('connect-history-api-fallback')())
```

# 十四、express使用http-proxy-middleware做代理
相关资料请查看[http-proxy-middleware](https://www.npmjs.com/package/http-proxy-middleware)
下载：
```
npm i -D http-proxy-middleware
```
修改dev-server.js
```js
var config = require('../config')
if(!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var open = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var proxyMiddleware = require('http-proxy-middleware') // 引入http-proxy-middleware
var webpackConfig = require('./webpack.dev.conf')
var port = process.env.PORT || config.dev.port
var autoOpenBrowser = !!config.dev.autoOpenBrowser
var proxyTable = config.dev.proxyTable // 获取需要代理的配置
var app = express()
var compiler = webpack(webpackConfig)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: false,
  heartbeat: 2000
})
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emot', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
// 遍历对象动态设置代理
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if(typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})
app.use(require('connect-history-api-fallback')())
app.use(devMiddleware)
app.use(hotMiddleware)
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))
var uri = 'http://localhost:' + port
var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})
console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  if(autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    open(uri)
  }
  _resolve()
})
var server = app.listen(port)
module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```
修改config中index.js
```js
var path = require('path')
module.exports = {
  build: {
    env: require('./prod.env'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './'
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {} // 暂时不需要代理
  }
}
```

# 十五、添加node和npm的版本检测
在package.json中添加版本信息。
```js
"engines": {
  "node": ">= 4.0.0",
  "npm": ">= 3.0.0"
}
```
webpack.base.conf.js中添加.json文件引入后缀
```js
resolve: {
  extensions: ['.js', '.json']
}
```
下载chalk、semver、shelljs插件，其中chalk插件用于给输出文字上色，semver插件用来处理或比较字符串等功能，shelljs用于处理命令。
```js
npm i -D chalk semver shelljs
```
build下新建版本检测文件check-versions.js
```js
var chalk = require('chalk') // 引入chalk
var semver = require('semver') // 引入semver
var packageConfig = require('../package') // 获取package.json文件
var shell = require('shelljs') // 引入shell
function exec (cmd) { // 通过开启子进程执行命令得到返回结果
  return require('child_process').execSync(cmd).toString().trim()
}
var versionRequirements = [
  {
    name: 'node',
    currentVersion: semver.clean(process.version), // semver.clean处理字符串
    versionRequirement: packageConfig.engines.node
  }
]
if (shell.which('npm')) { // 通过shelljs检测是否支持npm命令
  versionRequirements.push({
    name: 'npm',
    currentVersion: exec('npm --version'), // 效用exec执行npm --version获取npm版本号
    versionRequirement: packageConfig.engines.npm
  })
}
module.exports = function () {
  var warnings = []
  for (var i = 0; i < versionRequirements.length; i++) {
    var mod = versionRequirements[i]
    // 比较两个版本号是否相等
    if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
      warnings.push(mod.name + ': ' +
        chalk.red(mod.currentVersion) + ' should be ' +
        chalk.green(mod.versionRequirement) // 给错误消息上色
      )
    }
  }
  if (warnings.length) { // 存在版本不对的情况给出提示
    console.log('')
    console.log(chalk.yellow('To use this template, you must update following to modules:'))
    console.log()
    for (var i = 0; i < warnings.length; i++) {
      var warning = warnings[i]
      console.log('  ' + warning)
    }
    console.log()
    process.exit(1) // 结束当前进程
  }
}
```
dev-server.js最开头使用检查文件做一下检查
```js
require('./check-versions')()
```

# 十六、配置css样式加载器
此处之所以把css样式相关的loader单独提出来，是为了可以通过配置文件配置样式加载器的sourceMap，方便调试。并且可以把这一类型的加载器组合到一起，之后使用sass、less、stylus等的时候不需要每次都重复很多相同代码。目前我们只使用css-loader。
```
npm i -D css-loader style-loader
```
config的index.js中增加控制参数
```js
var path = require('path')
module.exports = {
  build: {
    env: require('./prod.env'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './'
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},
    cssSourceMap: false // 是否开启cssSourceMap
  }
}
```
webpack.dev.conf.js中调用我们即将在utils.js中合并的加载器模块组合到webpack基础配置中。
```js
var utils = require('./utils.js') // 引入utils
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
module.exports = merge(baseWebpackConfig, {
  module: { // 合并loader
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```
修改utils文件
```js
var path = require('path')
var config = require('../config')
exports.assetsPath = function (_path) {
  var assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  return path.posix.join(assetsSubDirectory, _path)
}
exports.cssLoaders = function (options) {
  options = options || {}
  var cssLoader = { // 默认使用css-loader
    loader: 'css-loader',
    options: {
      minimize: process.env.NODE_ENV === 'production',
      sourceMap: options.sourceMap
    }
  }
  function generateLoaders (loader, loaderOptions) {
    var loaders = [cssLoader]
    if(loader) { // 当传递了参数进来时，使用参数-loader拼接出加载器
      loaders.push({
        loader: loader + '-loader',
        options: Object.assign({}, loaderOptions, {
          sourseMap: options.sourceMap // 动态设置sourceMap
        })
      })
    }
    return ['style-loader'].concat(loaders)
  }
  return {
    css: generateLoaders(), // 默认使用css-loader
    postcss: generateLoaders(),
    less: generateLoaders('less'), // 其他这些加载器可以先定义，有用到时直接先下载后直接使用即可
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus'),
    styl: generateLoaders('stylus')
  }
}
// 数组形式导出各个加载器
exports.styleLoaders = function (options) {
  var output = []
  var loaders = exports.cssLoaders(options)
  for (var extension in loaders) {
    var loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      use: loader
    })
  }
  return output
}
```
上面预定义的postcss加载器需要添加一些配置信息，所以我们先配置好，之后只要下载postcss-loader便可使用。先下载autoprefixer组件
```
npm i -D autoprefixer cssnano
```
在根目录下新建.postcssrc.js用于postcss的配置
```js
module.exports = {
  "plugins": {
    "autoprefixer": {}
  }
}
```

# 十七、构建生产环境
到目前为止我们的开发环境已经部署完成了，但是仅仅有开发环境还不行，实际项目中我们需要有一个生产环境用于将项目打包成文件然后发布，为此我们先在package.json中加入生产命令。
```js
"scripts": {
  "dev": "node build/dev-server.js",
  "start": "node build/dev-server.js",
  "build": "node build/build.js" // 执行build.js文件用于生产
}
```
在build文件夹下新建一个build文件用于构建生产环境，在构建过程中我们使用到了ora、rimraf插件。ora插件用于实现node.js命令行环境的loading效果和显示各种状态的图标等。rimraf是node的一个深度删除模块,相当于rm -rf。先下载它们。
```
npm i -D ora rimraf
```
修改build.js
```js
require('./check-versions')() // 检测版本
process.env.NODE_ENV = 'production' // 标识当前环境为生产环境
var ora = require('ora') // 引入ora
var rm = require('rimraf') // 引入rimraf
var path = require('path')
var chalk = require('chalk')
var webpack = require('webpack')
var config = require('../config')
var webpackConfig = require('./webpack.prod.conf') // 用于生产的webpack配置
var spinner = ora('building for production...')
spinner.start()
// 打包完成后删除模块
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, function (err, stats) {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')
    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```

# 十八、制作用于生产环境的webpack.prod.conf.js
## 18.1 基础配置
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var webpackConfig = merge(baseWebpackConfig, {
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'), // 输出文件
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    })
  ]
})
module.exports = webpackConfig
```
## 18.2 extract-text-webpack-plugin分离css和js代码
下载extract-text-webpack-plugin插件。
```
npm i -D extract-text-webpack-plugin
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var ExtractTextPlugin = require('extract-text-webpack-plugin') // 引入插件
var webpackConfig = merge(baseWebpackConfig, {
  module: { // 引入css加载器
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new ExtractTextPlugin({ // 使用插件
      filename: utils.assetsPath('css/[name].[contenthash].css')
    })
  ]
})
module.exports = webpackConfig
```
修改config下的index.js
```js
var path = require('path')
module.exports = {
  build: {
    env: require('./prod.env'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './',
    productionSourceMap: true // 生产环境开启sourceMap
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},
    cssSourceMap: false
  }
}
```
修改utils.js的generateLoaders方法
```js
// 引入插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')
function generateLoaders (loader, loaderOptions) {
  var loaders = [cssLoader]
  if(loader) {
    loaders.push({
      loader: loader + '-loader',
      options: Object.assign({}, loaderOptions, {
        sourseMap: options.sourceMap
      })
    })
  }
  if(options.extract) { // 生产环境开启css和js分离
    return ExtractTextPlugin.extract({
      use: loaders,
      fallback: 'style-loader'
    })
  } else { // 否则正常显示
    return ['style-loader'].concat(loaders)
  }
}
```
## 18.3 使用html-webpack-plugin根据模版生成文件
config的index.js中build下添加模版文件参数。
```js
build: {
  env: require('./prod.env'),
  index: path.resolve(__dirname, '../dist/index.html'), // 模版文件地址
  assetsRoot: path.resolve(__dirname, '../dist'),
  assetsSubDirectory: 'static',
  assetsPublicPath: './',
  productionSourceMap: true
}
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin') // 引入模块
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new HtmlWebpackPlugin({ // 使用模版
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    })
  ]
})
module.exports = webpackConfig
```
## 18.4 添加资源打包方式和js/css代码压缩
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false, // 打包方式
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({ // 压缩js和css
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    })
  ]
})
module.exports = webpackConfig
```
## 18.5 使用optimize-css-assets-webpack-plugin优化css
下载：
```
npm i -D optimize-css-assets-webpack-plugin
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin') // 引入插件
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({ // 使用插件
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    })
  ]
})
module.exports = webpackConfig
```
## 18.6 使用CommonsChunkPlugin提取公共代码
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    }),
    new webpack.optimize.CommonsChunkPlugin({ // 提取公共代码先排除node_modules文件夹
      name: 'vendor',
      minChunks: function (module, count) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({ // 提取公共代码到manifest
      name: 'manifest',
      chunks: ['vendor']
    })
  ]
})
module.exports = webpackConfig
```
## 18.7 使用copy-webpack-plugin将静态资源拷贝到生产资源包中
下载：
```
npm i -D copy-webpack-plugin
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin') // 引入插件
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    new CopyWebpackPlugin([ // 使用插件
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
module.exports = webpackConfig
```
## 18.8 开启Gzip压缩
下载：
```
npm i -D compression-webpack-plugin
```
修改config的index.js中build
```js
build: {
  env: require('./prod.env'),
  index: path.resolve(__dirname, '../dist/index.html'),
  assetsRoot: path.resolve(__dirname, '../dist'),
  assetsSubDirectory: 'static',
  assetsPublicPath: './',
  productionSourceMap: true,
  productionGzip: true,
  productionGzipExtensions: ['js', 'css']
}
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')
  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}
module.exports = webpackConfig
```
## 18.9 使用可视化打包工具webpack-bundle-analyzer
下载：
```
npm i -D webpack-bundle-analyzer
```
修改config下index.js中build
```js
build: {
  env: require('./prod.env'),
  index: path.resolve(__dirname, '../dist/index.html'),
  assetsRoot: path.resolve(__dirname, '../dist'),
  assetsSubDirectory: 'static',
  assetsPublicPath: './',
  productionSourceMap: true,
  productionGzip: true,
  productionGzipExtensions: ['js', 'css'],
  bundleAnalyzerReport: process.env.npm_config_report
}
```
修改webpack.prod.conf.js
```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      chunksSortMode: 'dependency'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')
  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}
// 可视化打包
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
module.exports = webpackConfig
```

# 十九、添加编辑器配置
根目录下新建.editorconfig
```
root = true
[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```

# 二十、添加git上传时忽略文件
根目录下新建.gitignore
```
.DS_Store
node_modules/
dist/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Editor directories and files
.idea
*.suo
*.ntvs*
*.njsproj
*.sln
```