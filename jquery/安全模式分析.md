<div style="text-align: center; font-weight: 700; font-size: 2em;">安全模式分析</div>

# jQuery的"XSS"漏洞：
现在jQuery框架中没有已知的直接xss漏洞（不包括jQuery插件）。但对于不受信任的内容引用到jQuery时，DOM（innerHTML，document.write()等）可能会被修改。

低版本中最常见的例子：
```html
<html>
  <head></head>
  <body>
    <script src="https://code.jquery.com/jquery-1.6.1.js"></script>
    <script>
      $(location.hash).appendTo('body');
    </script>
  </body>
</html>
```
在浏览器中打开此页面，地址栏后输入以下内容会弹出内容为1的对话框
```
#<script>alert(1)</script>
```
因为1.6.1中对传入jQuery对象的字符串验证方式为：
```
/^(?:[^<]*(<[\w\W]+>)[^>]*$|#([\w\-]*)$)/
```
可以验证通过如下#后直接跟html元素的字符串，此时使用location.hash可能出现xss漏洞（bug 9521）。
```
#<script>alert(1)</script>
#<img src=1 onerror="confirm(1)" />
```
在高版本中jQuery对验证字符串进行了修改，如3.2版本：
```
/^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]+))$/
```
此时地址栏#后面再跟html元素，使用location.hash做参数会直接报错，认证不通过，但是如果将location.hash使用方法改一下：
```html
<html>
  <head></head>
  <body>
    <script src="https://code.jquery.com/jquery-1.6.1.js"></script>
    <script>
      $(location.hash.split('#')[1]).appendTo('body');
    </script>
  </body>
</html>
```
此时浏览器输入#跟html元素，只会将html元素传给jQuery，如果此元素为可执行html元素，仍然存在xss漏洞。所以我们在使用时应该特别注意不要随便将不可信内容引入到jQuery，如：
```
#<script>alert(1)</script>
```
> 注：其他修改DOM的方法使用时也建议不要往页面插入不可信内容，因为不确定jQuery所有方法都做了避免xss漏洞处理。