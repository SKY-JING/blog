<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">[1,2,3].map(parseInt)</div>

输入：
```
[1,2,3].map(parseInt);
```

输出：
```
[1,NaN,NaN]
```

原因：
1. parseInt接收两个参数string,radix，radix表示要解析的数字的基数（该值介于 2 ~ 36 之间）
  * 如果省略该参数或其值为 ‘0‘，则数字将以 10 为基础来解析。如果它以 ‘”0x”‘ 或 ‘”0X”‘ 开头，将以 16 为基数。
  * 如果该参数小于 2 或者大于 36，则 ‘parseInt()‘ 将返回 ‘NaN‘。
  * 如果字符串的第一个字符不能被转换为数字，那么 parseInt() 会返回 NaN。

2. map方法回调会传入三个参数value（当前循环的值）,index（索引）,array（原数组）
3. [1,2,3].map(parseInt)转化为parseInt(1,0),parseInt(2,1),parseInt(3,2)
4. parseInt(1,0)=1（radix为0按十进制解析）
5. parseInt(2,1)=NaN（radix不在2-36区间）
6. parseInt(3,2)=NaN（二进制只包含0，1。3不是二进制数，故不能进行二进制解析）