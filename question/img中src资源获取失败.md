<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">img中src资源获取失败</div>

通过设置img标签的onerror方法添加src资源加载失败时默认显示图片。
```html
<img src='...' onerror=javascript:this.src='...'>
```