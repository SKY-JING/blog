<div style="text-align: center; font-weight: 700; font-size: 2em;">webpack入门</div>

# 一、什么是webpack？
webpack 是当下最热门的前端资源模块化管理和打包工具，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。

我们可以直接使用 require(XXX) 的形式来引入各模块，即使它们可能需要经过编译（比如JSX和sass），但我们无须在上面花费太多心思，因为 webpack 有着各种健全的加载器（loader）在默默处理这些事情。

# 二、为什么要使用webpack？
现今的很多网页其实可以看做是功能丰富的应用，它们拥有着复杂的JavaScript代码和一大堆依赖包。为了简化开发的复杂度，前端社区涌现出了很多好的实践方法。
* 模块化，让我们可以把复杂的程序细化为小的文件。
* 类似于TypeScript这种在JavaScript基础上拓展的开发语言：使我们能够实现目前版本的JavaScript不能直接使用的特性，并且之后还能能装换为JavaScript文件使浏览器可以识别。
* sass，less等CSS预处理器的出现。
* ...

这些改进确实大大的提高了我们的开发效率，但是利用它们开发的文件往往需要进行额外的处理才能让浏览器识别，而手动处理又是非常繁琐的，这就为webpack类的工具的出现提供了需求。

# 三、webpack的优势
* webpack 是以 commonJS 的形式来书写脚本的，但对 AMD/CMD 的支持也很全面，方便旧项目进行代码迁移。
* 能被模块化的不仅仅是 JS 了。
* 开发便捷，能替代部分 grunt/gulp 的工作，比如打包、压缩混淆、图片转base64等。
* 扩展性强，插件机制完善，特别是支持 React 热插拔（见 react-hot-loader ）的功能让人眼前一亮。

# 四、webpack基础使用
## 4.1 安装
安装webpack前的准备工作：
```
// 创建一个空的练习文件夹并切换到文件夹目录
mkdir test && cd test
// 创建package.json文件,这是一个标准的npm说明文件，里面蕴含了丰富的信息，包括当前项目的依赖模块，自定义的脚本任务等等
npm init
```
> 注意：创建package.json之后下载项目依赖时可能出现两个警告：

```shell
npm WARN test@1.0.0 No description
npm WARN test@1.0.0 No repository field  
```
解决办法: 打开package.json,在description字段中加入描述解决第一个警告，页面加入"private":true解决第二个警告。
```js
{
  "name": "test",
  "version": "1.0.0",
  "description": "A test", // 解决第一个警告
  "main": "index.js",
  "private": true, // 解决第二个警告
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^2.4.1"
  }
}
```
安装webpack作为依赖包（学习时推荐全局安装，实际项目中推荐局部安装）：
```
// 全局安装
npm install -g webpack
// 安装到你的项目目录
npm install --save-dev webpack
```
## 4.2 准备工作
开始学习webpack之前我们先在项目中创建几个额外的文件，文件目录结构如下：
```
|-- src                              // 需要处理的源文件路径
|   |-- other.js                     // 供main.js引用的其他文件
|   |-- main.js                      // 需编译的入口文件
|-- dist                             // 项目文件编译后文件路径
|   |-- index.html                   // 应用首页
|-- package.json                     // 项目基本信息
```
index.html文件只有最基础的html代码，它唯一的目的就是加载打包后的js文件。
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>A simple Webpack Project</title>
  </head>
  <body>
    <div id='hi'>
    </div>
    <script src="main.js"></script>
  </body>
</html>
```
other.js只包括一个用来返回包含问候信息的html元素的函数。
```js
module.exports = function() {
  var greet = document.createElement('div');
  greet.textContent = "hello world";
  return greet;
};
```
main.js用来把other模块返回的节点插入页面。
```js
var greeter = require('./other.js');
document.getElementById('hi').appendChild(greeter());
```
## 4.3 开始使用
### 4.3.1 直接使用：
```
// 如果安装了全局webpack
webpack {入口文件} {输出文件}
// 如果是在项目中加入的webpack依赖
node_modules/.bin/webpack {入口文件} {输出文件}
// 此次我们使用全局打包
webpack src/main.js dist/main.js
```
可以看到项目的dist文件夹下会出现一个main.js文件，webpack同时编译了main.js 和other,js,现在打开index.html,可以看到hello world的提示。

现在我们已经能通过webpack打包一个文件了，不过如果在终端中进行复杂的操作，还是不太方便且容易出错的，接下来看看Webpack的另一种使用方法。

### 4.3.2 通过配置文件使用：
webpack拥有很多其它的比较高级的功能（比如说本文后面会介绍的loaders和plugins），这些功能其实都可以通过命令行模式实现，但是正如已经提到的，这样不太方便且容易出错的，一个更好的办法是定义一个配置文件，这个配置文件其实也是一个简单的JavaScript模块，可以把所有的与构建相关的信息放在里面。

在当前练习文件夹的根目录下新建一个名为webpack.config.js的文件，并在其中进行最最简单的配置，如下所示，它包含入口文件路径和存放打包后文件的地方的路径。
```js
module.exports = {
  entry:  __dirname + "/src/main.js", // 入口文件
  output: {
    path: __dirname + "/dist", //打包后的文件存放的地方
    filename: "main.js" //打包后输出文件的文件名
  }
}
```
其中"__dirname"是node.js中的一个全局变量，它指向当前执行脚本所在的目录。

有了webpack.config.js文件之后我们可以直接在终端输入webpack命令，这条命令会自动参考webpack.config.js文件中的配置选项打包你的项目。

### 4.3.3 更高级的执行打包任务：
在npm中可以引导任务执行，对其进行配置后可以使用简单的npm start命令来代替这些繁琐的命令。在package.json中对npm的脚本部分进行相关设置即可，设置方法如下：
```js
{
  "name": "test",
  "version": "1.0.0",
  "description": "A test",
  "main": "index.js",
  "private": true,
  "scripts": {
    "start": "webpack" // 修改此处，把npm的start命令指向webpack命令
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^2.4.1"
  }
}
```
package.json中的脚本部分已经默认在命令前添加了node_modules/.bin路径，所以无论是全局还是局部安装的webpack，你都不需要写前面那指明详细的路径了。

npm的start是一个特殊的脚本名称，它的特殊性表现在，在命令行中使用npm start就可以执行相关命令，如果对应的此脚本名称不是start，想要在命令行中运行时，需要这样用npm run {script name}如npm run build。

# 五、webpack进阶
## 5.1 Source Maps（使调试更容易）
开发总是离不开调试，如果可以更加方便的调试当然就能提高开发效率，不过打包后的文件有时候你是不容易找到出错了的地方对应的源代码的位置的，Source Maps就是来帮我们解决这个问题的。

通过简单的配置后，Webpack在打包时可以为我们生成的source maps，这为我们提供了一种对应编译文件和源文件的方法，使得编译后的代码可读性更高，也更容易调试。

在webpack的配置文件中配置source maps，需要配置devtool，它有以下四种不同的配置选项，各具优缺点，描述如下：

| devtool选项        | 配置结果          |
|---------------|:-------------:|
|source-map|在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包文件的构建速度|
|cheap-module-source-map|在一个单独的文件中生成一个不带列映射的map，不带列映射提高项目构建速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便|
|eval-source-map|使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。不过在开发阶段这是一个非常好的选项，但是在生产阶段一定不要用这个选项|
|cheap-module-eval-source-map|这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点|
正如上表所述，上述选项由上到下打包速度越来越快，不过同时也具有越来越多的负面作用，较快的构建速度的后果就是对打包后的文件的的执行有一定影响。

在学习阶段以及在小到中性的项目上，eval-source-map是一个很好的选项，不过记得只在开发阶段使用它，继续上面的例子，进行如下配置：
```js
module.exports = {
  devtool: 'eval-source-map', // 配置生成Source Maps，选择合适的选项
  entry:  __dirname + "/src/main.js", // 入口文件
  output: {
    path: __dirname + "/dist", //打包后的文件存放的地方
    filename: "main.js" //打包后输出文件的文件名
  }
}
```
cheap-module-eval-source-map方法构建速度更快，但是不利于调试，推荐在大型项目考虑到时间成本时使用。
## 5.2 webpack-dev-server 构建本地服务器
想不想让你的浏览器监测你的代码的修改，并自动刷新修改后的结果，其实Webpack提供一个可选的本地开发服务器，这个本地服务器基于node.js构建，可以实现你想要的这些功能，不过它是一个单独的组件，在webpack中进行配置之前需要单独安装它作为项目依赖。
```
npm install --save-dev webpack-dev-server
```
devserver作为webpack配置选项中的一项，具有以下配置选项：

|devserver配置选项|功能描述|
|---------------|:-------------:|
|contentBase|默认webpack-dev-server会为根文件夹提供本地服务器，如果想为另外一个目录下的文件提供本地服务器，应该在这里设置其所在目录（本例设置到"dist"目录）|
|port|设置默认监听端口，如果省略，默认为"8080"|
|inline|设置为true，当源文件改变时会自动刷新页面|
|historyApiFallback|在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html|

继续把这些命令加到webpack的配置文件中，现在的配置文件如下所示:
```js
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  devServer: {
    contentBase: "./dist", // 本地服务器所加载的页面所在的目录
    historyApiFallback: true, // 不跳转
    inline: true // 实时刷新
  }
}
```
同时在package.json中给服务添加开启快捷方式：
```
"scripts": {
  "server": "webpack-dev-server"
}
```
## 5.3 loaders
Loaders是webpack中最让人激动人心的功能之一了。通过使用不同的loader，webpack通过调用外部的脚本或工具可以对各种各样的格式的文件进行处理，比如说分析JSON文件并把它转换为JavaScript文件，或者说把下一代的JS文件（ES6，ES7)转换为现代浏览器可以识别的JS文件。或者说对React的开发而言，合适的Loaders可以把React的JSX文件转换为JS文件。

Loaders需要单独安装并且需要在webpack.config.js下的modules关键字下进行配置，Loaders的配置选项包括以下几方面：
* test：一个匹配loaders所处理的文件的拓展名的正则表达式（必须）
* loader：loader的名称（必须）
* include/exclude:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
* query：为loaders提供额外的设置选项（可选）

继续上面的例子，我们把other.js里的问候消息放在一个单独的JSON文件里,并通过合适的配置使other.js可以读取该JSON文件的值，配置方法如下:
```js
//安装可以装换JSON的loader
npm install --save-dev json-loader
// 修改配置文件
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: { // 在配置文件里添加JSON loader
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
创建带有问候信息的JSON文件(命名为config.json)
```js
{
  "greetText": "hello world"
}
```
更新后的other.js
```js
var config = require('./config.json');
module.exports = function() {
  var greet = document.createElement('div');
  greet.textContent = config.greetText;
  return greet;
};
```
Loaders很好，不过有的Loaders使用起来比较复杂，比如说Babel。
## 5.4 Babel
Babel其实是一个编译JavaScript的平台，它的强大之处表现在可以通过编译帮你达到以下目的：
* 下一代的JavaScript标准（ES6，ES7），这些标准目前并未被当前的浏览器完全的支持；
* 使用基于JavaScript进行了拓展的语言，比如React的JSX

### 5.4.1 Babel的安装与配置
Babel其实是几个模块化的包，其核心功能位于称为babel-core的npm包中，不过webpack把它们整合在一起使用，但是对于每一个你需要的功能或拓展，你都需要安装单独的包（用得最多的是解析Es6的babel-preset-es2015包和解析JSX的babel-preset-react包），我们先来一次性安装这些依赖包。
```
// npm一次性安装多个依赖模块，模块之间用空格隔开
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
```
在webpack中配置Babel的方法如下：
```js
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader', // 在webpack的module部分的loaders里进行配置即可
        query: {
          presets: ['es2015']
        }
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
现在你的webpack的配置已经允许你使用ES6以及JSX的语法了。继续用上面的例子进行测试。

使用ES6的语法，更新other.js并返回一个组件。
```js
import config from './config.json';
class greeter {
  model() {
    let node = document.createElement('div');
    node.innerHTML = config.greetText;
    return node;
  }
}
var greeters = new greeter();
export default greeters;
```
更新main.js渲染模块
```js
import greeter from './other';
document.getElementById('hi').append(greeter.model());
```
### 5.4.2 Babel的配置选项
Babel其实可以完全在webpack.config.js中进行配置，但是考虑到babel具有非常多的配置选项，在单一的webpack.config.js文件中进行配置往往使得这个文件显得太复杂，因此一些开发者支持把babel的配置选项放在一个单独的名为 ".babelrc" 的配置文件中。我们现在的babel的配置并不算复杂，不过之后我们会再加一些东西，因此现在我们就提取出相关部分，分两个配置文件进行配置（webpack会自动调用.babelrc里的babel配置选项），如下：
```js
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader' // 留下基础的babel配置参数
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
//.babelrc
{
  "presets": ["es2015"]
}
```
到目前为止，我们已经知道了，对于模块，Webpack能提供非常强大的处理功能，那哪些是模块呢。
## 5.5 一切皆模块
Webpack有一个不可不说的优点，它把所有的文件都可以当做模块处理，包括你的JavaScript代码，也包括CSS和fonts以及图片等等等，只要通过合适的loaders，它们都可以被当做模块被处理。
## 5.6 css
webpack提供两个工具处理样式表，css-loader 和 style-loader，二者处理的任务不同，css-loader使你能够使用类似@import 和 url(...)的方法实现 require()的功能,style-loader将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。

继续上面的例子
```
//安装
npm install --save-dev style-loader css-loader
```
```js
// 修改配置
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: 'style-loader!css-loader' // 添加对样式表的处理
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
> 注：感叹号的作用在于使同一文件能够使用不同类型的loader

接下来，在src文件夹里创建一个名字为"main.css"的文件，对一些元素设置样式
```css
html {
  box-sizing: border-box;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
}
*, *:before, *:after {
  box-sizing: inherit;
}
body {
  margin: 0;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
h1, h2, h3, h4, h5, h6, p, ul {
  margin: 0;
  padding: 0;
}
```
你还记得吗？webpack只有单一的入口，其它的模块需要通过 import, require, url等导入相关位置，为了让webpack能找到”main.css“文件，我们把它导入”main.js “中，如下
```js
import greeter from './other';
import './main.css';
document.getElementById('hi').append(greeter.model());
```
通常情况下，css会和js打包到同一个文件中，并不会打包为一个单独的css文件，不过通过合适的配置webpack也可以把css打包为单独的文件的。

不过这也只是webpack把css当做模块而已，咱们继续看看一个真的CSS模块的实践。
## 5.7 CSS module
在过去的一些年里，JavaScript通过一些新的语言特性，更好的工具以及更好的实践方法（比如说模块化）发展得非常迅速。模块使得开发者把复杂的代码转化为小的，干净的，依赖声明明确的单元，且基于优化工具，依赖管理和加载管理可以自动完成。

不过前端的另外一部分，CSS发展就相对慢一些，大多的样式表却依旧是巨大且充满了全局类名，这使得维护和修改都非常困难和复杂。

最近有一个叫做 CSS modules 的技术就意在把JS的模块化思想带入CSS中来，通过CSS模块，所有的类名，动画名默认都只作用于当前模块。Webpack从一开始就对CSS模块化提供了支持，在CSS loader中进行配置后，你所需要做的一切就是把”modules“传递都所需要的地方，然后就可以直接把CSS的类名传递到组件的代码中，且这样做只对当前组件有效，不必担心在不同的模块中具有相同的类名可能会造成的问题。具体的代码如下:
```js
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules' // 此处添加了一个modules
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
创建一个other.css文件
```css
.root {
  background-color: #eee;
  padding: 10px;
  border: 3px solid #ccc;
}
```
导入.root到other.js中
```js
import config from './config.json';
import styles from './other.css'; // 导入.root
class greeter {
  model() {
    let node = document.createElement('div');
    node.className = styles.root; // 使用
    node.innerHTML = config.greetText;
    return node;
  }
}
var greeters = new greeter();
export default greeters;
```
如此设置就可以放心使用，相同的类名也不会造成不同组件之间的污染。 CSS modules 也是一个很大的主题，有兴趣的话可以去css-modules查看更多消息。
## 5.8 CSS预处理器
Sass 和 Less之类的预处理器是对原生CSS的拓展，它们允许你使用类似于variables, nesting, mixins, inheritance等不存在于CSS中的特性来写CSS，CSS预处理器可以这些特殊类型的语句转化为浏览器可识别的CSS语句，你现在可能都已经熟悉了，在webpack里使用相关loaders进行配置就可以使用了，以下是常用的CSS 处理loaders。
* Less Loader
* Sass Loader
* Stylus Loader

不过其实也存在一个CSS的处理平台-PostCSS，它可以帮助你的CSS实现更多的功能，在其CSS官方文档[postcss](https://github.com/postcss/postcss)可了解更多相关知识。

举例来说如何使用PostCSS，我们使用PostCSS来为CSS代码自动添加适应不同浏览器的CSS前缀。

首先安装postcss-loader 和 autoprefixer（自动添加前缀的插件）
```
npm install --save-dev postcss-loader autoprefixer
```
接下来，在webpack配置文件中进行设置，只需要新建一个postcss关键字，并在里面申明依赖的插件，如下，现在你写的css会自动根据不同浏览器添加不同前缀了。
```js
// webpack.config.js中引入webpack
var webpack = require('webpack');
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules!postcss-loader'
      }
    ]
  },
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
// 新建postcss.config.js中
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```
到现在，本文已经涉及到处理JS的Babel和处理CSS的PostCSS，它们其实也是两个单独的平台，配合Webpack可以很好的发挥它们的作用。接下来介绍Webpack中另一个非常重要的功能-Plugins，其实在刚使用autoprefixer时已经可以看到plugins参数使用了。
## 5.9 插件（Plugins）
插件（Plugins）是用来拓展Webpack功能的，它们会在整个构建过程中生效，执行相关的任务。Loaders和Plugins常常被弄混，但是他们其实是完全不同的东西，可以这么来说，loaders是在打包构建过程中用来处理源文件的（JSX，Scss，Less..），一次处理一个，插件并不直接操作单个文件，它直接对整个构建过程其作用。

Webpack有很多内置插件，同时也有很多第三方插件，可以让我们完成更加丰富的功能。
### 5.9.1 使用插件的方法
要使用某个插件，我们需要通过npm安装它，然后要做的就是在webpack配置中的plugins关键字部分添加该插件的一个实例（plugins是一个数组）继续看例子，我们添加了一个实现版权声明的插件。
```js
// 引入webpack
var webpack = require('webpack');
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules!postcss-loader'
      }
    ]
  },
  plugins: [
    new webpack.BannerPlugin("测试测试@版权所有.") //在这个数组中new一个就可以了
  ],
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
通过这个插件，打包后的JS文件会显示版权信息，如我们打包后的main.js开头显示
```
/*! 测试测试@版权所有. */
```
知道Webpack中的插件如何使用了，下面给大家推荐一个常用的插件
### 5.9.2 HtmlWebpackPlugin
这个插件的作用是依据一个简单的模板，帮你生成最终的Html5文件，这个文件中自动引用了你打包后的JS文件。每次编译都在文件名中插入一个不同的哈希值。

安装:
```
npm install --save-dev html-webpack-plugin
```
这个插件自动完成了我们之前手动做的一些事情，在正式使用之前需要对一直以来的项目结构做一些改变：
* 移除dist文件夹，利用此插件，HTML5文件会自动生成，此外CSS已经通过前面的操作打包到JS中了，dist文件夹里。
* 在src目录下，创建一个html文件模板，这个模板包含title等其它你需要的元素，在编译过程中，本插件会依据此模板生成最终的html页面，会自动添加所依赖的 css， js，favicon等文件，在本例中我们命名模板文件名称为index.tmpl.html，模板源代码如下

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>A simple Webpack Project</title>
  </head>
  <body>
    <div id='hi'></div>
  </body>
</html>
```
* 更新webpack的配置文件,执行npm start即可看到自动生成了dist文件夹以及index.html和main.js

```js
var webpack = require('webpack');
// 引入html-webpack-plugin插件
var HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules!postcss-loader'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      //new 一个HtmlWebpackPlugin插件的实例，并传入相关的参数
      template: __dirname + "/src/index.tmpl.html"
    }),
    new webpack.BannerPlugin("测试测试@版权所有.")
  ],
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
## 5.10 产品阶段的构建
目前为止，我们已经使用webpack构建了一个完整的开发环境。但是在产品阶段，可能还需要对打包的文件进行额外的处理，比如说优化，压缩，缓存以及分离CSS和JS。

对于复杂的项目来说，需要复杂的配置，这时候分解配置文件为多个小的文件可以使得事情井井有条，以上面的例子来说，我们可以创建一个“webpack.production.config.js”的文件，在里面加上基本的配置,可专门用于生产项目时使用。具体复杂项目配置可参考vue-cli脚手架搭建项目的结构自行理解。

## 5.11 优化插件
webpack提供了一些在发布阶段非常有用的优化插件，它们大多来自于webpack社区，可以通过npm安装，通过以下插件可以完成产品发布阶段所需的功能。
* OccurrenceOrderPlugin :为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID
* UglifyJsPlugin：压缩JS代码；
* ExtractTextPlugin：分离CSS和JS文件

我们继续用例子来看看如何添加它们，OccurenceOrder 和 UglifyJS plugins 都是内置插件，你需要做的只是使用它们
```
npm install --save-dev extract-text-webpack-plugin
```
在配置文件的plugins后引用它们
```js
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
// 引入extract-text-webpack-plugin插件
var ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "main.js"
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      // 使用extract-text-webpack-plugin包装css加载器
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: 'css-loader?modules!postcss-loader'
        })
      }
    ]
  },
  plugins: [
    new webpack.optimize.OccurrenceOrderPlugin(), // 使用OccurrenceOrderPlugin
    new webpack.optimize.UglifyJsPlugin(), // 使用UglifyJsPlugin压缩js
    new ExtractTextPlugin("style.css"), // 使用extract-text-webpack-plugin
    new HtmlWebpackPlugin({
      template: __dirname + "/src/index.tmpl.html"
    }),
    new webpack.BannerPlugin("测试测试@版权所有.")
  ],
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
## 5.12 缓存
缓存无处不在，使用缓存的最好方法是保证你的文件名和文件内容是匹配的（内容改变，名称相应改变） webpack可以把一个哈希值添加到打包的文件名中，使用方法如下,添加特殊的字符串混合体（[name], [id] and [hash]）到输出文件名前。
```js
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/src/main.js",
  output: {
    path: __dirname + "/dist",
    filename: "[name]-[hash].js" // 使用hash值
  },
  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: 'css-loader?modules!postcss-loader'
        })
      }
    ]
  },
  plugins: [
    new webpack.optimize.OccurrenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin(),
    new ExtractTextPlugin("[name]-[hash].css"), // 使用hash值
    new HtmlWebpackPlugin({
      template: __dirname + "/src/index.tmpl.html"
    }),
    new webpack.BannerPlugin("测试测试@版权所有.")
  ],
  devServer: {
    contentBase: "./dist",
    colors: true,
    historyApiFallback: true,
    inline: true
  }
}
```
现在用户会有合理的缓存了。到这里我们可以说已经对webpack使用比较熟悉了。