<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">rem布局</div>

```js
// 移动端适配，根据设备ratina不同先进行缩放，然后根据640屏幕标准设置rem所参考的基础字体大小
// 从而达到根据屏幕大小或分辨率不同自动调整H5页面中字体或组件大小
(function (doc, win) {
  var docEl = doc.documentElement;
  // 一物理像素在不同屏幕的显示效果不一样。要根据devicePixelRatio来修改meta标签的scale
  (function(){
    var dpr = scale = '',
      isIPhone = win.navigator.appVersion.match(/iphone/gi),
      devicePixelRatio = win.devicePixelRatio;
    if (isIPhone) {
      // iOS下，3倍屏用3的方案
      if (devicePixelRatio >= 3) {
        dpr = 3;
      } else if (devicePixelRatio >= 2){  // 2倍屏用2的方案
        dpr = 2;
      } else {  // 其余用1的方案
        dpr = 1;
      }
    } else {    // 其他设备下，仍旧使用1倍的方案
      dpr = 1;
    }
    scale = 1 / dpr;
    var metaEl = "";
    metaEl = doc.createElement('meta');
    metaEl.setAttribute('name', 'viewport');
    metaEl.setAttribute('content', 'width=device-width,initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
    if (docEl.firstElementChild) {
      docEl.firstElementChild.appendChild(metaEl);
    } else {
      var wrap = doc.createElement('div');
      wrap.appendChild(metaEl);
      doc.write(wrap.innerHTML);
    }
  })();
  var isMobile = navigator.userAgent.toLowerCase().match(/ipad|iphone|midp|rv:1.2.3.4|ucweb|android|windows ce|windows mobile|micromessenger/i);
    resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
    clientWidth = docEl.clientWidth,    // 此处放在外层防止input唤起页面重新计算font-size
    clientHeight = docEl.clientHeight,
    min = Math.min(clientWidth, clientHeight),
    recalc = function () {
      if(!!isMobile){
        // 移动端按640标准（以iphone5标准开发，iphone5上1rem == 100px）设置字体
        var fontSize = Math.ceil(100 * (min / 640));
        docEl.style.fontSize = fontSize + 'px';
      }else { // 此处为兼容pc端手动调整的，不影响手机上效果(pc上1px任然为1px)
        docEl.style.fontSize =  '100px';
      }
      // 页面横竖屏切换时让页面向上滚动到顶部，解决fixed定位浮动bug
      window.scrollTo(0, 0);
    };
  // 如果浏览器不支持addEventListener方法，终止运行
  if (!doc.addEventListener) return;
  win.addEventListener(resizeEvt, recalc, false); // 页面调试(横竖屏)时重新设置字体
  doc.addEventListener('DOMContentLoaded', recalc, false); // 页面dom结构加载完设置字体
})(document, window);
```