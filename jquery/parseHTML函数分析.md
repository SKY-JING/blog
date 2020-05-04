<div style="text-align: center; font-weight: 700; font-size: 2em;">parseHTML函数分析</div>

# 一、语法：
```js
$.parseHTML(htmlString [, context] [,keepScripts])
```

# 二、参数：
* htmlString: 需要解析并转化为DOM节点数组的HTML字符串
* context: 指定使用哪个Document创建元素，默认为当前文档的document
* keepScripts: 指定传入的HTML字符串转化时是否包含script脚本部分，默认false

# 三、用法：
将字符串转化为包含dom节点的数组
```js
// 解析script
$.parseHTML("<script src='test'>console.log(111)<\/script>11", true) // [script, text]
// 屏蔽script
$.parseHTML("<script src='test'>console.log(111)<\/script>11") // [text]
```

# 四、源码分析：
```js
var rsingleTag = ( /^<([a-z][^\/\0>:\x20\t\r\n\f]*)[\x20\t\r\n\f]*\/?>(?:<\/\1>|)$/i );
// 在safari 8中，通过document.implementation.createHTMLDocument创建document
// 连续两个form，第二个form会成为第一个form的子元素
// 由于这个原因，此安全措施在safari 8中不可用
// https://bugs.webkit.org/show_bug.cgi?id=137337
support.createHTMLDocument = ( function() {
  var body = document.implementation.createHTMLDocument( "" ).body;
  body.innerHTML = "<form></form><form></form>";
  return body.childNodes.length === 2;
} )();


// Argument "data" should be string of html
// context (optional): If specified, the fragment will be created in this context,
// defaults to document
// keepScripts (optional): If true, will include scripts passed in the html string
jQuery.parseHTML = function( data, context, keepScripts ) {
  // data不是string直接返回[]
  if ( typeof data !== "string" ) {
    return [];
  }
  
  // 第二个期望传入document对象，当第二个参数不传直接传第三个参数boolean类型时
  if ( typeof context === "boolean" ) {
    keepScripts = context; // 设置第三个参数为当前传入boolean
    context = false; // context设置为false供后面重新指向document使用
  }

  var base, parsed, scripts;

  //  context不存在时设置context指向document
  if ( !context ) {

    // Stop scripts or inline event handlers from being executed immediately
    // by using document.implementation
    // 安全措施可用时，通过document.implementation.createHTMLDocument创建一个新的document，而不影响当前document内容
    if ( support.createHTMLDocument ) {
      context = document.implementation.createHTMLDocument( "" );

      // Set the base href for the created document
      // so any parsed elements with URLs
      // are based on the document's URL (gh-2965)
      // 给用户操作的新document设置资源base url
      base = context.createElement( "base" );
      base.href = document.location.href;
      context.head.appendChild( base );
    } else { // 不支持时使用html的document
      context = document;
    }
  }

  // 解析是否为纯标签，即标签中不包含内容，如：<div></div>或<img />等
  parsed = rsingleTag.exec( data );
  // 是否移除script标志位
  // 按此写法，传入非boolean值也会根据转化结果控制是否移除script
  scripts = !keepScripts && [];

  // 单个标签时直接创建标签dom元素返回
  if ( parsed ) {
    return [ context.createElement( parsed[ 1 ] ) ];
  }

  // 否则通过buildFragment构建出片段
  parsed = buildFragment( [ data ], context, scripts );

  // 根据标识位选择是否移除script部分
  if ( scripts && scripts.length ) {
    // 移除script，具体remove方法请查看remove函数解析
    jQuery( scripts ).remove();
  }

  // 返回片段中的子节点，即片段中的dom元素
  return jQuery.merge( [], parsed.childNodes );
};
```
> 注：源码分析中使用到了buildFragment和remove方法，具体请查看buildFragment函数分析一文和remove函数分析一文。