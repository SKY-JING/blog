<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">buildFragment函数分析</div>

buildFragment顾名思义为构建片段，其中用到了一个关键方法createDocumentFragment。

DocumentFragments是DOM节点。它们不是主DOM树的一部分。通常的用例是创建文档片段，将元素附加到文档片段，然后将文档片段附加到DOM树。在DOM树中，文档片段被其所有的子元素所代替。

因为文档片段存在于内存中，并不在DOM树中，所以将子元素插入到文档片段时不会引起页面回流(对元素位置和几何上的计算)。因此，使用文档片段document fragments通常会起到优化性能的作用，如：
```js
// 传统写法：每次都将新建的元素添加到dom树，引起页面回流
var test = ['1', '2', '3', '4', '5'];
for (var i = 0; i < test.length; i++) {
  var cp = document.createElement('p');
  var ctext = document.createTextNode(test[i]);
  cp.appendChild(ctext);
  document.body.appendChild(cp);
}

// 优化后写法：多次操作内存中的文档片段，最后一次添加到dom树中，不会造成多次页面回流
var test = ['1', '2', '3', '4', '5'];
var cfrag = document.createDocumentFragment(); // 创建文档片段
for (var i = 0; i < test.length; i++) {
  var cp = document.createElement('p');
  var ctext = document.createTextNode(test[i]);
  cp.appendChild(ctext);
  cfrag.appendChild(cp); // 操作内存中的文档片段
}
document.body.appendChild(cfrag); // 将片段添加到dom树中
```
由此可以看出在目标节点存在的情况下先往片段（fragment）中添加内容然后将内容一次性添加到目标节点上比一次次将创建的内容直接添加到目标节点上效率更高。但我们仍然可以对createElement优化：
```js
// 先创建一个元素存储其他需要多次添加的子元素，然后一次性添加到dom中
var test = ['1', '2', '3', '4', '5'];
var cdiv = document.createElement('div');
for (var i = 0; i < test.length; i++) {
  var cp = document.createElement('p');
  var ctext = document.createTextNode(test[i]);
  cp.appendChild(ctext);
  cdiv.appendChild(cp);
}
document.body.appendChild(cdiv);
```
可以明显看出虽然此时进行了优化，但是多增加了元素，不如直接使用createDocumentFragment创建片段进行处理。 

下面是私有方法buildFragment及相关源码：
```js
// We have to close these tags to support XHTML (#13200)
var wrapMap = {

  // Support: IE <=9 only
  option: [ 1, "<select multiple='multiple'>", "</select>" ],

  // XHTML parsers do not magically insert elements in the
  // same way that tag soup parsers do. So we cannot shorten
  // this by omitting <tbody> or other required elements.
  // XHTML 标签解析器不能自动添加<tbody>等其他标签，故此处不可省略。
  thead: [ 1, "<table>", "</table>" ],
  col: [ 2, "<table><colgroup>", "</colgroup></table>" ],
  tr: [ 2, "<table><tbody>", "</tbody></table>" ],
  td: [ 3, "<table><tbody><tr>", "</tr></tbody></table>" ],

  _default: [ 0, "", "" ]
};

// Support: IE <=9 only
wrapMap.optgroup = wrapMap.option;

wrapMap.tbody = wrapMap.tfoot = wrapMap.colgroup = wrapMap.caption = wrapMap.thead;
wrapMap.th = wrapMap.td;

var rtagName = ( /<([a-z][^\/\0>\x20\t\r\n\f]+)/i );
var rscriptType = ( /^$|\/(?:java|ecma)script/i );
var rhtml = /<|&#?\w+;/;

// 后面两个参数selection, ignored只在replaceWith方法中使用。
// 需要了解的是 replaceWith 只做节点替换，不会替换先前元素的所有数据（Data），比如绑定事件，$.data 都不会被新元素拥有。
function buildFragment( elems, context, scripts, selection, ignored ) {
  var elem, tmp, tag, wrap, contains, j,
    fragment = context.createDocumentFragment(), // 创建文档片段
    nodes = [],
    i = 0,
    l = elems.length;

  // 构建节点
  for ( ; i < l; i++ ) {
    elem = elems[ i ];

    if ( elem || elem === 0 ) {

      // Add nodes directly
      // 立即添加节点
      if ( jQuery.type( elem ) === "object" ) {

        // Support: Android <=4.0 only, PhantomJS 1 only
        // push.apply(_, arraylike) throws on ancient WebKit
        // 根据nodeType判断当前elem是否为dom元素
        jQuery.merge( nodes, elem.nodeType ? [ elem ] : elem );

      // Convert non-html into a text node
      // elem是字符串且不是html标签时创建文本节点对象放入nodes数组中
      } else if ( !rhtml.test( elem ) ) {
        nodes.push( context.createTextNode( elem ) );

      // Convert html into DOM nodes
      // elem是字符串且是html标签，将其转化为dom元素，放入nodes数组中
      } else {
        // 在文档片段中创建一个div元素用于添加之后解析的标签
        tmp = tmp || fragment.appendChild( context.createElement( "div" ) );

        // Deserialize a standard representation
        // 解析出标签名
        tag = ( rtagName.exec( elem ) || [ "", "" ] )[ 1 ].toLowerCase();
        // 标签名是否在wrapMap中有定义，未定义使用默认值
        wrap = wrapMap[ tag ] || wrapMap._default;
        // 将标签转化为闭合标签添加到文档片段中div内
        tmp.innerHTML = wrap[ 1 ] + jQuery.htmlPrefilter( elem ) + wrap[ 2 ];

        // Descend through wrappers to the right content
        // warp[0]表示了标签层级，遍历到标签最后一级
        j = wrap[ 0 ];
        while ( j-- ) {
          tmp = tmp.lastChild;
        }

        // Support: Android <=4.0 only, PhantomJS 1 only
        // push.apply(_, arraylike) throws on ancient WebKit
        // 将新建标签节点添加到nodes数组
        jQuery.merge( nodes, tmp.childNodes );

        // Remember the top-level container
        // 获取文档片段第一个子元素，即div元素
        tmp = fragment.firstChild;

        // Ensure the created nodes are orphaned (#12392)
        // 将元素内容置空
        tmp.textContent = "";
      }
    }
  }

  // Remove wrapper from fragment
  // 移除文档片段中内容
  fragment.textContent = "";

  i = 0;
  // 循环遍历节点
  while ( ( elem = nodes[ i++ ] ) ) {

    // Skip elements already in the context collection (trac-4087)
    // 跳过已经在selection数组中的元素
    if ( selection && jQuery.inArray( elem, selection ) > -1 ) {
      if ( ignored ) {
        ignored.push( elem );
      }
      continue;
    }

    // 检查元素的根元素是否包含此元素
    contains = jQuery.contains( elem.ownerDocument, elem );

    // Append to fragment
    // 将元素添加到文档片段，并获取片段中所有script标签
    tmp = getAll( fragment.appendChild( elem ), "script" );

    // Preserve script evaluation history
    // 全局执行tmp内的js
    if ( contains ) {
      setGlobalEval( tmp );
    }

    // Capture executables
    // 当传入的scripts标签为true时，捕获可执行文件，即把src引入的资源文件取下来
    if ( scripts ) {
      j = 0;
      while ( ( elem = tmp[ j++ ] ) ) {
        if ( rscriptType.test( elem.type || "" ) ) {
          scripts.push( elem );
        }
      }
    }
  }

  // 返回创建好的文档片段
  return fragment;
}

var rxhtmlTag = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([a-z][^\/\0>\x20\t\r\n\f]*)[^>]*)\/>/gi;

// 将标签转化为闭合标签
// 如：$.htmlPrefilter('111<table class="test"/>') -> 111<table class="tese"></table>
jQuery.extend({
  htmlPrefilter: function ( html ) {
    return html.replace( rxhtmlTag, "<$1></$2>" );
  }
})
```