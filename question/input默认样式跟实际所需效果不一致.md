<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">input默认样式跟实际所需效果不一致</div>

重新设置placeholder的样式（颜色，字体大小等）。
```css
input::-webkit-input-placeholder {/*WebKit browsers*/color:white;font-size:16px;}
input:-moz-placeholder {/*Mozilla Firefox 4 to 18*/color:white;font-size:16px;}
input::-moz-placeholder {/*Mozilla Firefox 19+*/color:white;font-size:16px;}
input:-ms-input-placeholder {/*Internet Explorer 10+*/color:white;font-size:16px;}
```