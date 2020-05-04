<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">文字超出div区域</div>

超出区域的内容用...代替。
```css
/* 备注：必须先控制内容显示区域宽高 */
/* 单行行末加... */
width:100px;
height:40px;
overflow:hidden;
white-space:nowrap;
text-overflow:ellipsis;

/* 第二行行末加...（可用于多行）*/
width:100px;
height:60px;
line-height:30px;
word-break:break-all;
display:-webkit-box;
-webkit-line-clamp:2;
-webkit-box-orient:vertical;
overflow:hidden;
white-space:normal
```