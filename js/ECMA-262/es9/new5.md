# 反向断言

## 历史
目前正则表达式中支持先行断言(lookahead)。这意味着匹配会发生，但不会有任何捕获，并且断言没有包含在整个匹配字段中。如：
```js
// 从价格中捕获货币符号
const reLookahead = /\D(?=\d+)/,
  match = reLookahead.exec('$123.45');
console.log(match[0]); // $
```

## 新增
引入以相同方式工作但是匹配前面的反向断言(lookbehind)，这样可以忽略货币符号，单纯捕获价格的数字。
```js
// 肯定反向断言，非数字\D必须存在
const reLookbehind = /(?<=\D)\d+/,
  match = reLookbehind.exec('$123.45');
console.log(match[0]); // 123

// 否定反向断言，非数字\D必须不存在（即必须以数字开头）
const reLookbehindNeg = /(?<!\D)\d+/,
  match = reLookbehindNeg.exec('$123.45');
console.log(match[0]); // 23
```