<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">获取url中参数</div>

```js
/**
 * [方法一：通过key获取url中单个参数]
 * @param  {[string]} _key [要获取的键值]
 * @param  {[string]} _url [要解析的url，不传默认location.href]
 * @return {[string]}      [对应的值]
 */
var getUrlParams = function(_key, _url) {
  if (typeof(_url) == 'object') {
    _url = _url.location.href;
  } else {
    _url = (typeof(_url) == 'undefined' || _url == null || _url == '') ? location.href : _url;
  }
  if (_url.indexOf('?') == -1) {
    return '';
  }
  var params = [];
  _url = _url.split('?')[1].split('&');
  for (var i = 0, len = _url.length; i < len; i++) {
    params = _url[i].split('=');
    if (params[0] == _key) {
      return params[1].split('#')[0];
    }
  }
  return '';
}

/**
 * [方法二：解析url中所有键值对]
 * @param  {[string]} str [浏览器地址,不传默认为location.href]
 * @return {[object]}     [参数键值对]
 */
var query = function(str) {
  var data = {},
    arr = [],
    uri = '',
    url = typeof str === 'undefined' ? location.href : str;
  uri = decodeURIComponent(url);
  if (uri.indexOf('?') != -1) {
    arr = (uri.split('?', 2)[1]);
    if (arr.length != 0) {
      arr = arr.split('&');
      var len = arr.length,
        nameValue; // 存储分解形如 name=value 的字符串为数组形式
      for (var i = 0; i < len; i++) {
        nameValue = arr[i].split('=');
        data[nameValue[0]] = nameValue[1].split('#')[0];
      }
      return data;
    }
  }
  return {};
}
```