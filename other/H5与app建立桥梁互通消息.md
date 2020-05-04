<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">H5与app建立桥梁互通消息</div>

```js
// 实现H5与ios/android通信
setBridge: function() {
  // 注意ios获取数据是异步的
  if (!!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)) {
    function setupWebViewJavascriptBridge(callback) {
      if (window.WebViewJavascriptBridge) {
        return callback(WebViewJavascriptBridge);
      }
      if (window.WVJBCallbacks) {
        return window.WVJBCallbacks.push(callback);
      }
      window.WVJBCallbacks = [callback];
      var WVJBIframe = document.createElement('iframe');
      WVJBIframe.style.display = 'none';
      WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
      document.documentElement.appendChild(WVJBIframe);
      setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
    }
    setupWebViewJavascriptBridge(function(bridge) {
      // 定义和ios互通的桥梁函数
      bridge.registerHandler('IpanelJsInterface.getThemeColor();', function(data, responseCallback) {
        var responseData = { 'Javascript Says': 'Right back atcha!' }
        responseCallback(responseData)
      });
      // 回调获取数据
      bridge.callHandler('testObjcCallback', { 'foo': 'bar' }, function(response) {
        if (response) {
          // 获取数据
          userData.userTheme = response.split("|")[0];
          userData.accessToken = response.split("|")[1];
          localStorage.userName = response.split("|")[2];
          localStorage.devicetype = response.split("|")[3];
          localStorage.deviceno = response.split("|")[4];
          localStorage.homeName = response.split("|")[5];
          userData.flag = true;
        };
      })
    })
  } else if (/(Android)/i.test(navigator.userAgent)) { // 安卓为同步
    try { // 通过try-catch避免异常
      if (IpanelJsInterface) { // IpanelJsInterface是跟android协商好的桥梁字段
        // 获取数据
        userData.userTheme = IpanelJsInterface.getThemeColor();
        userData.accessToken = IpanelJsInterface.getToken();
        localStorage.userName = IpanelJsInterface.getUserName();
        localStorage.devicetype = IpanelJsInterface.getDevicetype();
        localStorage.deviceno = IpanelJsInterface.getDeviceno();
        localStorage.homeName = IpanelJsInterface.getHomeName();
        userData.flag = true;
      };
    } catch (e) {};
  };
}
```