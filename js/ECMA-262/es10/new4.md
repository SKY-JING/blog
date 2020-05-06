# String
新增trimStart()和trimEnd()用于去除字符串首尾空格。

## String.prototype.trimStart()
trimStart() 方法从字符串的开头删除空格。trimLeft() 是此方法的别名。
```js
const greeting = '   Hello world!   ';

console.log(greeting);
// expected output: "   Hello world!   ";

console.log(greeting.trimStart());
// expected output: "Hello world!   ";
```

## String.prototype.trimEnd()
trimEnd() 方法从一个字符串的末端移除空白字符。trimRight() 是这个方法的别名。
```js
const greeting = '   Hello world!   ';

console.log(greeting);
// expected output: "   Hello world!   ";

console.log(greeting.trimEnd());
// expected output: "   Hello world!";
```

## 参考资料
trimStart：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/TrimLeft

trimEnd：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/TrimRight