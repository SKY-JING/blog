<div style="text-align: center; font-weight: 700; font-size: 2em;">vue-cli2.x项目结构分析</div>

# 一、使用vue-cli创建项目

## 1.1 创建项目

安装vue-cli（要求已安装node.js）。

```shell
npm install -g vue-cli
```

创建模板
```shell
vue init <template-name> <project-name>
```

其中template-name是可选模版项，project-name是创建项目的名称。目前提供以下几种模板选项：

* webpack - A full-featured webpack + vue-loader setup with hot reload, linting, testing & css extraction
* webpack-simple - A simple webpack + vue-loader setup for quick prototyping
* browserify - A full-featured browserify + vueify setup with hot-reload, linting & unit testing
* browserify-simple - A simple browserify + vueify setup for quick prototyping
* simple - the simplest possible vue setup in single html file

当然也可以使用自定义的模板，模板可以来自远端托管仓库或本地。

此次我们选用webpack模板创建项目 `vue init webpack 项目名`

## 1.2 结构分析

项目整体结构如下:

```
|-- build                            // 项目构建(webpack)相关代码
|   |-- build.js                     // 生产环境构建代码
|   |-- check-version.js             // 检查node、npm等版本
|   |-- dev-client.js                // 热重载相关
|   |-- dev-server.js                // 构建本地服务器
|   |-- utils.js                     // 构建工具相关
|   |-- vue-loader.conf.js           // vue加载配置
|   |-- webpack.base.conf.js         // webpack基础配置
|   |-- webpack.dev.conf.js          // webpack开发环境配置
|   |-- webpack.prod.conf.js         // webpack生产环境配置
|-- config                           // 项目开发环境配置
|   |-- dev.env.js                   // 开发环境变量
|   |-- index.js                     // 项目一些配置变量
|   |-- prod.env.js                  // 生产环境变量
|-- src                              // 源码目录
|   |-- assets                       // 资源文件(图片)
|   |-- components                   // vue公共组件
|   |-- router                       // 路由
|   |-- App.vue                      // 页面入口文件
|   |-- main.js                      // 程序入口文件，加载各种公共组件
|-- static                           // 静态文件，比如一些图片，json数据等
|   |-- .gitkeep                     // git配置相关
|-- .babelrc                         // ES6语法编译配置
|-- .editorconfig                    // 定义代码格式
|-- .eslintignore                    // 忽视监听代码路径
|-- .eslintrc                        // 监听代码模块配置
|-- .gitignore                       // git上传需要忽略的文件格式
|-- .postcssrc.js                    // autoprefixer配置，用于自动补全兼容各浏览器的代码
|-- index.html                       // 入口页面
|-- package.json                     // 项目基本信息
|-- README.md                        // 项目说明
```

### 1.2.1 package.json
package.json文件是项目根目录下的一个文件，定义该项目开发所需的各种模块以及一些项目配置信息（如项目名称、版本、描述、作者等）。

package.json中几个重要字段： 

1）scripts字段
```js
"scripts": {
  "dev": "node build/dev-server.js",
  "start": "node build/dev-server.js",
  "build": "node build/build.js",
  "lint": "eslint --ext .js,.vue src"
}
```
在开发环境下，在命令行中运行npm run dev就相当于在执行node build/dev-server.js，所以scripts字段是用来指定npm相关命令的缩写。 

> 注意：scripts中定义的start字段可直接通过npm start运行，其他字段需要加run。

2）dependencies字段和devDependencies字段

dependencies字段指定了项目运行时所依赖的模块，devDependencies字段指定了项目开发时所依赖的模块。在命令行中运行npm install命令，会自动安装dependencies和devDependencies字段中的模块。 

package.json还有很多配置相关项，想进一步了解可参考[package.json](https://docs.npmjs.com/files/package.json)
### 1.2.2 webpack配置相关
上面说到在命令行中npm run dev就相当于在执行node build/dev-server.js，想必dev-server.js这个文件是十分重要的，它是在开发环境下构建时第一个要运行的文件，所以我们从这个文件入手来详细分析以下。 

dev-server.js：
```js
// 检查node和npm版本
require('./check-versions')()
// 获取config/index.js的默认配置
var config = require('../config')
// 如果node的环境无法判断当前是dev/product使用config.dev.env.NODE_ENV作为当前环境
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
// 使用open工具，用于在浏览器中打开地址
var opn = require('opn')
// 使用nodeJS自带的文件路径工具
var path = require('path')
// 使用express
var express = require('express')
// 使用webpack
var webpack = require('webpack')
// http-proxy可以实现转发所有请求代理到后端真实API地址，以实现前后端开发完全分离
// 在config/index.js中可以对proxyTable想进行配置
var proxyMiddleware = require('http-proxy-middleware')
// 使用dev环境的webpack配置
var webpackConfig = require('./webpack.dev.conf')
// 如果没有指定运行端口，使用config.dev.port作为运行端口
var port = process.env.PORT || config.dev.port
// 是否自动打开浏览器
var autoOpenBrowser = !!config.dev.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
// 使用config.dev.proxyTable的配置作为proxyTable的代理配置
var proxyTable = config.dev.proxyTable
// 启动一个express
var app = express()
// 启动webpack进行编译
var compiler = webpack(webpackConfig)
// 启动webpack-dev-middleware,将webpack编译后的文件暂存到内存中
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
// 启动webpack-hot-middleware,也就是我们常说的hot-reload
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})
// force page reload when html-webpack-plugin template changes
// 当html-webpack-plugin模板发生改变时，强制重新价值页面
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})
// proxy api requests
// 将proxyTable中的请求配置挂载到启动的express服务上
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})
// handle fallback for HTML5 history API
// 使用connect-history-api-fallback匹配资源，如果不匹配久可以重定向到指定地址
app.use(require('connect-history-api-fallback')())
// serve webpack bundle output
// 将缓存到内存中的webpack编译后的文件挂载到express服务上
app.use(devMiddleware)
// enable hot-reload and state-preserving
// compilation error display
// 将hot-reload挂载到express服务上
app.use(hotMiddleware)
// serve pure static assets
// 拼接static文件夹的静态资源路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
// 为服务器提供响应服务
app.use(staticPath, express.static('./static'))
// 拼接本地服务器url
var uri = 'http://localhost:' + port
// 定义暴露接口所需的字段
var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})
// 通过webpack-dev-middleware在服务器开启时做一些配置处理
console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  // when env is testing, don't need open it
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    // open后带第二个参数可设置用哪种浏览器打开,如：open("http://www.google.com", "chrome")
    opn(uri)
  }
  _resolve()
})
// 启动express，并且监听端口
var server = app.listen(port)
// 将此服务作为dev-server.js的接口暴露
module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```
刚刚我们在 dev-server.js 中用到了 webpack.dev.conf.js 和 index.js，我们先来看一下。

webpack.dev.conf.js：
```js
// 使用自己定义的一个工具模块
var utils = require('./utils')
// 使用webpack
var webpack = require('webpack')
// 使用config/index.js
var config = require('../config')
// 使用webpack配置合并插件
var merge = require('webpack-merge')
// 加载webpack.base.conf
var baseWebpackConfig = require('./webpack.base.conf')
// 使用html-webpack-plugin插件，这个插件可以帮我们自动生成 html 并且注入到 .html 文件中
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 使用错误友好提示插件
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
// 将hot-loader相对应路径添加到webpack.base.conf的对于entry前
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
// 将我们webpack.dev.conf.js的配置和webpack.base.conf.js的配置合并
module.exports = merge(baseWebpackConfig, {
  module: {
    // 使用自定义的styleLoaders
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
  // 使用#source-map模式作为开发工具，此模式有四个选项(source-map、cheap-module-source-map、eval-source-map、chep-module-eval-source-map)，四个选项具体详情请自行百度
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    // definePlugin接收字符串插入到代码当中，所以你需要的话可以写上JS的字符串
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    // hotMoudle插件再页面进行变更的时候只会重回对应的页面模块，不会重绘整个html文件
    new webpack.HotModuleReplacementPlugin(),
    // 使用NoEmitOnErrorsPlugin后页面中的报错不会阻塞，但会再编译结束后报错
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    // 将index.html作为入口,注入html代码后生成index.html
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    // 使用错误友好提示插件
    new FriendlyErrorsPlugin()
  ]
})
```
我们看到在webpack.dev.conf.js中又引入了webpack.base.conf.js，这个文件为webpack的基础配置文件，非常重要。

webpack.base.conf.js: 
```js
// 使用nodeJS自带的文件路径插件
var path = require('path')
// 使用自己编写的utils工具模块
var utils = require('./utils')
// 引入config/index.js
var config = require('../config')
// 引入vue-loader模块
var vueLoaderConfig = require('./vue-loader.conf')
// 定义一个resolve函数，不适用path自带的resolve方法
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
// 导出模块
module.exports = {
  entry: {
    // 编译入口文件
    app: './src/main.js'
  },
  output: {
    // 编辑输出路径
    path: config.build.assetsRoot,
    // 编译输出文件的文件名
    filename: '[name].js',
    // 正式发布环境下编译输出的发布路径
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    // 自动补全的扩展名
    extensions: ['.js', '.vue', '.json'],
    alias: {
      // 默认路径代理，例如import vue from 'vue'，会自动到'vue/dist/vue.esm.js'中寻找
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  },
  module: {
    // 各种不同类型文件加载器配置
    rules: [
      {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
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
我们看到在webpack.base.conf.js中引入了vue-loader.conf.js，我们来分析一下这个文件。

vue-loader.conf.js: 
```js
// 引入自定义的utils工具模块
var utils = require('./utils')
// 引入config/index.js
var config = require('../config')
// 判断当前环境是否为生产环境
var isProduction = process.env.NODE_ENV === 'production'
// 导出模块
module.exports = {
  // 根据不同生产环境调用utils中的cssLoaders对webpack.base.conf.js中模块规则所需参数进行配置
  loaders: utils.cssLoaders({
    sourceMap: isProduction
      ? config.build.productionSourceMap
      : config.dev.cssSourceMap,
    extract: isProduction
  })
```
在dev-server.js中用到了检查版本方法,我们也分析一下。

check-versions.js: 
```js
// 使用chalk插件，此插件用于给文字设置颜色
var chalk = require('chalk')
// 使用semver插件，此插件用于比较和处理版本数据
var semver = require('semver')
// 引入package.json
var packageConfig = require('../package.json')
// 使用shelljs插件，此插件用于对shell指令做一些处理
var shell = require('shelljs')
// 通过child_process执行指令
function exec (cmd) {
  return require('child_process').execSync(cmd).toString().trim()
}
// node版本参数
var versionRequirements = [
  {
    name: 'node',
    // semver.clean将版本号转化为数字，如semver.clean('v1.0.2')=>1.0.2
    currentVersion: semver.clean(process.version),
    // 配置文件中设置的版本号要求
    versionRequirement: packageConfig.engines.node
  },
]
// 如果系统中存在npm，需要检查npm版本
if (shell.which('npm')) {
  versionRequirements.push({
    name: 'npm',
    // 执行npm --version获取版本号
    currentVersion: exec('npm --version'),
    versionRequirement: packageConfig.engines.npm
  })
}
// 导出版本处理模块
module.exports = function () {
  // 定义警告数组
  var warnings = []
  for (var i = 0; i < versionRequirements.length; i++) {
    var mod = versionRequirements[i]
    // semver.satisfies检测当前版本是否符合配置信息中版本要求
    if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
      // 警告信息加入颜色标记后存入警告数组
      warnings.push(mod.name + ': ' +
        chalk.red(mod.currentVersion) + ' should be ' +
        chalk.green(mod.versionRequirement)
      )
    }
  }
  // 如果有某个版本不匹配，输出错误信息
  if (warnings.length) {
    console.log('')
    console.log(chalk.yellow('To use this template, you must update following to modules:'))
    console.log()
    for (var i = 0; i < warnings.length; i++) {
      var warning = warnings[i]
      console.log('  ' + warning)
    }
    console.log()
    // 存在错误时停止进程
    process.exit(1)
  }
}
```
在前面多个文件中都用到了utils.js这个自定义的模块，具体这个模块做什么用呢？

utils.js：
```js
// 引入nodeJS系统的path模块
var path = require('path')
// 引入config/index.js
var config = require('../config')
// 抽离文件插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')
// 拼接static模块路径
exports.assetsPath = function (_path) {
  var assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  return path.posix.join(assetsSubDirectory, _path)
}
// 导出css加载器
exports.cssLoaders = function (options) {
  options = options || {}
  // 基础css加载模块
  var cssLoader = {
    loader: 'css-loader',
    options: {
      minimize: process.env.NODE_ENV === 'production',
      sourceMap: options.sourceMap
    }
  }
  // generate loader string to be used with extract text plugin
  // 根据传入的加载器string动态创建加载模块配置
  function generateLoaders (loader, loaderOptions) {
    var loaders = [cssLoader]
    if (loader) {
      loaders.push({
        loader: loader + '-loader',
        options: Object.assign({}, loaderOptions, {
          sourceMap: options.sourceMap
        })
      })
    }
    // Extract CSS when that option is specified
    // (which is the case during production build)
    // 生产环境时使用extractTetPlugin插件分离代码
    if (options.extract) {
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'vue-style-loader'
      })
    } else {
      // 正常情况将构建好的loaders存入vue-style-loader
      return ['vue-style-loader'].concat(loaders)
    }
  }
  // https://vue-loader.vuejs.org/en/configurations/extract-css.html
  return {
    css: generateLoaders(),
    postcss: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus'),
    styl: generateLoaders('stylus')
  }
}
// Generate loaders for standalone style files (outside of .vue)
// 生成独立的样式文件装载机
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
dev-client.js:
```js
/* eslint-disable */
// 使用eventsource-polyfill,在浏览器不支持eventsource时提供一个解决方案
require('eventsource-polyfill')
// 使用热加载监听
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true')
// 当eventsource中出现reload的action时，重新加载页面
hotClient.subscribe(function (event) {
  if (event.action === 'reload') {
    window.location.reload()
  }
})
```
命令行中npm run build就相当于在执行node build/build.js,用于构建出浏览器中能直接运行的html,css,js资源文件。

build.js: 
```js
// 检查版本
require('./check-versions')()
// 将当前环境设置为生产环境
process.env.NODE_ENV = 'production'
// 使用ora, 用于生产过程中加入动态提示效果
var ora = require('ora')
// 使用rimraf,node的一个深度删除模块,相当于rm -rf
var rm = require('rimraf')
// 使用nodeJS的path模块
var path = require('path')
// 使用chalk,用于给文字上色
var chalk = require('chalk')
// 使用webpack
var webpack = require('webpack')
// 引入config/index.js
var config = require('../config')
// 引入webpack.prod.conf模块
var webpackConfig = require('./webpack.prod.conf')
// 构建过程中开启动画
var spinner = ora('building for production...')
spinner.start()
// 构建完成后删除模块
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  // 启动webpack进行编译，并实时打印出编译信息
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
在build中使用到了生产所用的webpack配置文件，此文件内容如下。

webpack.prod.conf.js: 
```js
// 引入nodeJS的path模块
var path = require('path')
// 引入自定义的工具模块
var utils = require('./utils')
// 引入webpack
var webpack = require('webpack')
// 引入配置文件config/index.js
var config = require('../config')
// 引用webpack的合并模块
var merge = require('webpack-merge')
// 引入基础配置文件,此文件上面有说明
var baseWebpackConfig = require('./webpack.base.conf')
// 使用copy-webpack-plugin,用于拷贝文件和文件夹
var CopyWebpackPlugin = require('copy-webpack-plugin')
// 使用html-webpack-plugin,自动身成html文件
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 使用extract-text-webpack-plugin,用于提取html中的样式
var ExtractTextPlugin = require('extract-text-webpack-plugin')
// 使用optimize-css-assets-webpack-plugin,提取css代码
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
// 获取build的env
var env = config.build.env
// 合并基本webpack模块和新加入模块
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    // 使用自定义的styleLoaders
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  // source-map前面webpack.dev.conf.js有说明
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    path: config.build.assetsRoot,
    // 输出文件名
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // chunkname是未被列在entry中，却又需要被打包出来的文件命名配置,如在按需加载（异步）模块的时候，这样的文件是没有被列在entry中的
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    // definePlugin在webpack.dev.conf.js有说明
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // 上线时代码会经过uglifyJsPlugin进行压缩
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      // 这里的soucemap 不能少，可以在线上生成soucemap文件，便于调试
      sourceMap: true
    }),
    // extract css into its own file
    // 抽取页面css保存到一个文件
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    // 提取css,可以将重复的css从不同的组件中提取出来
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    // 生成html文件
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    // split vendor js into its own file
    // 提取js到单独文件
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        // 将node_modules中引入的模块js，提取插入到生成的应用
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    // 提取webpack runtime和module manifest到单独的文件，避免模版提供商更新文件时，应用程序更新
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    // 从static中拷贝文件到制定路径
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
// 使用compression-webpack-plugin,压缩配置中设置的文件
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
// 使用webpack-bundle-analyzer,生成分析图看下是不是编译了哪些不需要的文件，唱用于热加载很慢的时候
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
// 导出配置文件
module.exports = webpackConfig
```
至此，webpack的构建生产模块已基本学习完，前面文件中经常用到config/index.html，config文件夹下全是页面所需的配置信息，自己看一眼就知道了，不必提出来说。

### 1.2.3 其他配置相关
.babelrc：

Babel解释器的配置文件，存放在根目录下。Babel是一个转码器，项目里需要用它将ES6代码转为ES5代码。详情可参考[babel](http://www.ruanyifeng.com/blog/2016/01/babel.html)
```js
{
  // 设置转码规则
  "presets": [
    ["env", { "modules": false }],
    "stage-2"
  ],
  // 转码的一些插件
  "plugins": ["transform-runtime"],
  "comments": false,
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": [ "istanbul" ]
    }
  }
}
```

.editorconfig：

该文件定义项目的编码规范，编辑器的行为会与.editorconfig 文件中定义的一致，并且其优先级比编辑器自身的设置要高，这在多人合作开发项目时十分有用而且必要。
```
root = true
[*]                             // 对所有文件应用下面的规则
charset = utf-8                 // 编码采用utf-8
indent_style = space            // 缩进用空格
indent_size = 2                 // 缩进数量为两个空格
end_of_line = lf                // 换行符格式
insert_final_newline = true     // 是否在文件的最后插入一个空行
trim_trailing_whitespace = true // 是否删除行尾的空格
```

.eslintignore：

eslint不监听文件路径
```
build/*.js    // build下的所有js文件
config/*.js   // config下的所有js文件
```

.eslintrc.js：

监听器配置
```js
// http://eslint.org/docs/user-guide/configuring
module.exports = {
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  // https://github.com/feross/standard/blob/master/RULES.md#javascript-standard-style
  extends: 'standard',
  // required to lint *.vue files
  plugins: [
    'html'
  ],
  // add your custom rules here
  'rules': {
    // allow paren-less arrow functions
    'arrow-parens': 0,
    // allow async-await
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0
  }
}
```

.gitignore： 

git上传需要忽略的路径
```
.DS_Store
node_modules/
dist/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
```
.postcssrc.js： 

autoprefixer自动补全代码配置
```
// https://github.com/michael-ciniawsky/postcss-load-config
module.exports = {
  "plugins": {
    // to edit target browsers: use "browserlist" field in package.json
    "autoprefixer": {}
  }
}
```
# 二、vue中使用其他前端框架
## 2.1 mint-ui
下载mint-ui:
```shell
npm install mint-ui -d
```
使用： 在main.js中直接引入使用即可,如:
```js
import Vue from 'vue'
import App from './App'
import MintUI from 'mint-ui'
import 'mint-ui/lib/style.css'
import router from './router'
Vue.config.productionTip = false
Vue.use(MintUI)
MintUI.Toast({
  message: 'Upload Complete',
  position: 'bottom',
  duration: 5000
})
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
```