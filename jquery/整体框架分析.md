<div style="text-align: center; font-weight: 700; font-size: 2em;">整体框架分析</div>

# 一、前言
分析前先从3.2.0版本中抽取出核心框架。
```js
/*!
 * jQuery JavaScript Library v3.2.0
 * https://jquery.com/
 *
 * Includes Sizzle.js
 * https://sizzlejs.com/
 *
 * Copyright JS Foundation and other contributors
 * Released under the MIT license
 * https://jquery.org/license
 *
 * Date: 2017-03-16T21:26Z
 */
(function (global, factory) {
  "use strict";
  if (typeof module === "object" && typeof module.exports === "object") {
    // For CommonJS and CommonJS-like environments where a proper `window`
    // is present, execute the factory and get jQuery.
    // For environments that do not have a `window` with a `document`
    // (such as Node.js), expose a factory as module.exports.
    // This accentuates the need for the creation of a real `window`.
    // e.g. var jQuery = require("jquery")(window);
    // See ticket #14549 for more info.
    module.exports = global.document ?
      factory(global, true) :
      function (w) {
        if (!w.document) {
          throw new Error("jQuery requires a window with a document");
        }
        return factory(w);
      }
  } else {
    factory(global);
  }
// Pass this if window is not defined yet
})(typeof window !== "undefined" ? window : this, function (window, noGlobal) {
  "use strict";
  // Define a local copy of jQuery
  var jQuery = function (selector, context) {
    // The jQuery object is actually just the init constructor 'enhanced'
    // Need init if jQuery is called (just allow error to be thrown if not included)
    return new jQuery.fn.init(selector, context);
  };
  // 定义jQuery原型上的方法，并给原型添加命名
  jQuery.fn = jQuery.prototype = {...};
  jQuery.extend = jQuery.fn.extend = function () {...};
  // 属性或方法在jQuery自身上扩展
  jQuery.extend({});
  // javascript css选择器引擎
  var Sizzle = (function(window){...})(window);
  // 属性或方法在jQuery.fn上进行扩展
  jQuery.fn.extend({});
  // 创建jQuery初始化init方法
  var init = jQuery.fn.init = function (selector, context, root) {...};
  // 将init的原型指向jQuery.fn
  init.prototype = jQuery.fn;
  // 提供统一事件管理,jQuery内部使用,并不对外开放 
  jQuery.event = {...};
  //Event类似于Java的POJO类.传递事件的对象
  jQuery.Event = function (src, props) {...};
  // Expose jQuery and $ identifiers, even in AMD
  // (#7102#comment:10, https://github.com/jquery/jquery/pull/557)
  // and CommonJS for browser emulators (#13566)
  if (!noGlobal) {
    window.jQuery = window.$ = jQuery;
  }
  return jQuery;
})
```

# 二、Sizzle.js
从第五行注释可以看到，jQuery引入了Sizzle.js。

Sizzle是一个纯javascript CSS选择器引擎。jquery1.3开始使用Sizzle，Sizzle一反传统采取了相反的Right To Left的查询匹配方式提高效率。

Sizzle是jQuery作者John Resig写的DOM选择器引擎,速度号称业界第一。Sizzle完全独立于jQuery，若不想用jQuery，可单独使用Sizzle。

# 三、立即执行函数
我们知道有两种常见的立即执行函数写法：
* (function(){...})()
* (function(){...}())

它们的原理是使用()运算符将函数变成函数表达式，同理!、+、-甚至逗号都可以将函数变成函数表达式（如：!function(){...}()），之后后面的括号就能立即触发函数执行（如表达式var a = function(){...}，之后使用a()可调用函数，var a = function(){...}()也可立即调用函数）。

相反如果不是函数表达式或者表达式未赋值则触发不了函数（如：function a(){...}()，js引擎只解析函数声明不能解析后面的括号，从而发生语法错误。function(){...}()也会发生语法错误，因为匿名函数虽然属于函数表达式，但是未进行赋值操作）。

此处jQuery选择常见写法的第一种。

# 四、传入当前窗口对象（typeof window !== "undefined" ? window : this）
虽然所有浏览器窗口都支持了window对象，但是为了兼容一些不具有窗口（window对象）的上下文环境（如web workers环境，即运行在后台的javascript），当window未定义时传递this对象指向当前全局对象（vue.js源码也只传递了this对象）。

> 注：浏览器窗口中全局的window,self,this都表示window对象，所以像vue.js一样直接传this也是可以的

# 五、调整立即执行函数结构
为了让对环境处理部分代码和真正的插件部分代码分离，采用将插件核心代码执行函数当成参数的形式传递给立即执行函数的函数体，然后在立即执行函数体中进行调用的方式。

> 注：立即执行函数体和插件核心函数都使用严格模式控制源码。

# 六、环境处理
如果当前页面执行环境未定义module模块（即常规的浏览器环境），直接调用插件核心代码执行函数。

如果当前页面执行环境定义了module和module.exports，表示当前环境调用了一些特殊的js文件（如node.js或requireJS），分下面两种情况：

1. 当在真实的浏览器环境中使用时是不需要将jQuery对象作为全局变量的（如：vue.js中使用jQuery，虽然可能用node做为后台，但是由于当前页面执行环境还是浏览器环境，我们使用时也只需要在需要用到的地方require('jquery')即可，故不需要将jQuery当成全局变量绑定在window上），所以给插件核心代码函数传递参数noGlobal为true。
2. 当在模拟的浏览器环境中（如node后台或者遵从commonJS和AMD规范的模拟浏览器环境），我们期望引用了之后仍然可以直接使用jQuery，故而需要全局绑定，但是注意这些模拟环境是没有window对象也没有document对象的，所以在使用时需要手动引入（如：var jQuery = require("jquery")(window)），下面代码是node.js环境中效果：

```js
// 保存app.js文件
var $ = require('jquery');
$('body').append('<div>test</div>');
console.log($('body').html());
// node app.js执行文件报错
Error: jQuery requires a window with a document

// 解决方法，使用jsdom模拟一个document便可正常使用
require('jsdom').env('', function(err, window) {
  if (err) {
    console.error(err);
    return;
  }
  var $ = require('jquery')(window);
  $('body').append('<div>test</div>');
  console.log($('body').html());
});
```

# 七、jQuery初始化
jQuery框架主要是对HTMl文档中元素进行快速匹配并执行某些操作，如：
```js
$('#test').hide()
$('#test').html('...').show()
```
虽然javaScript是函数式编程，但大家知道通过函数可以实现类的概念，从而达到通过其实例调用自身函数的目的，如：
```js
var jQuery = function (selector, context) {
  // 构造函数
}

jQuery.prototype = {
  // 原型
  a: function () {},
  b: function () {}
}

var jquery = new jQuery();
jquery.a();
```
上面是常规写法，但真正使用jQuery时发现，其并未采用new显示的进行实例化，而是直接采用$('#test').hide()调用自身方法，那么$()就应该已经是类的实例才对，为了达到目的我们对上面的代码进行修改：
```js
var jQuery = function (selector, context) {
  return new jQuery(selector, context);
}

jQuery.prototype = {
  a: function () {},
  b: function () {}
}
```
可以看到，我们通过new创建了一个实例，然后return出来，虽然现在实例创建了，但是很明显出现了死循环。我们对上面的方法再修改一下：
```js
var jQuery = function (selector, context) {
  return jQuery.prototype.init(selector, context);
}

jQuery.prototype = {
  init: function (selector, context) {
    return this;
  },
  a: function () {},
  b: function () {}
}
```
我们通过将实例方法放到jQuery的原型中，然后再通过此处this会指向jQuery原型达到实例化的目的。此时再执行jQuery()便可返回jQuery类的实例，不会再出现死循环。但是如果我们将init函数也当作一个构造器，又会出现新的问题：
```js
var jQuery = function (selector, context) {
  return jQuery.prototype.init(selector, context);
}

jQuery.prototype = {
  init: function (selector, context) {
    this.a = function () {
      console.log(11);
    }
    return this;
  },
  a: function () {
    console.log(22);
  },
  b: function () {}
}

jQuery().a() // 11
```
由于此时this指向的是jQuery的原型，所以在init中this.a会替换原型中a方法，从而输出11，所以我们需要给init设立独立的作用域才行：
```js
var jQuery = function (selector, context) {
  return new jQuery.prototype.init(selector, context);
}

jQuery.prototype = {
  init: function (selector, context) {
    this.a = function () {
      console.log(11);
    }
    return this;
  },
  a: function () {
    console.log(22);
  },
  b: function () {
    console.log(33);
  }
}

jQuery().a() // 11
jQuery().b() // Uncaught TypeError: jQuery(...).b is not a function
```
虽然此时输出仍然是11，但调用jQuery()时new每次构建了新的init实例对象，此时init中的this会指向jQuery.prototype.init的原型，拥有了独立作用域，也正是由于this指向了jQuery.prototype.init的原型，导致init中返回的this和jQuery类的this分离了，从而找不到b方法，出现异常。我们再对代码修改一下：
```js
var jQuery = function (selector, context) {
  return new jQuery.prototype.init(selector, context);
}

jQuery.prototype = {
  init: function (selector, context) {
    this.a = function () {
      console.log(11);
    }
    return this;
  },
  b: function () {
    console.log(33);
  }
}

jQuery.prototype.init.prototype = jQuery.prototype;
jQuery().a() // 11
jQuery().b() // 33
```
我们通过原型传递的方式将jquery.init方法的原型指向jQuery的原型，从而达到既能隔离作用域又能在实例中访问jQuery原型对象的目的。由于我们传递的是引用，所以也不需要担心循环引起的性能问题。然后我们观察jQuery框架中jQuery实例化过程正是如此（具体实例化过程请阅读jQuery实例化函数分析一文）：
```js
var jQuery = function (selector, context) {
  return new jQuery.fn.init(selector, context);
}

jQuery.fn = jQuery.prototype = {...}
var init = jQuery.fn.init = function (selector, context, root) {...};
init.prototype = jQuery.fn;
```
我们发现jQuery是支持链式调用的，链式调用比较简单，我们只需在每个支持链式调用的方法最后通过this返回当前实例的原型即可，如：
```js
var jQuery = function (selector, context) {
  return new jQuery.prototype.init(selector, context);
}

jQuery.prototype = {
  init: function (selector, context) {
    this.a = function () {
      console.log(11);
      return this;
    }
    return this;
  },
  b: function () {
    console.log(33);
    return this;
  }
}

jQuery.prototype.init.prototype = jQuery.prototype;

jQuery().a().b();
```

# 八、提供扩展接口
jquery提供给了开发者对jquery方法进行扩展，从封装角度来说，肯定是对扩展方法提供一个专用接口。

我们从核心框架可以看到有个jQuery.extend = jQuery.fn.extend = function () {...}定义就是对外提供的扩展接口，jQuery.extend和jQuery.fn.extend指向同一个处理扩展的方法，只是jQuery.extend是对jQuery本身的属性和方法进行扩展，jQuery.fn.extend是对jQuery.fn上的属性或方法扩展。

但是明明jQuery.extend和jQuery.fn.extend指向同一个方法为何能实现不同功能呢？答案请阅读jQuery扩展接口分析一文。