# 命名捕获组

## 历史
js正则表达式可以返回一个匹配对象（一个包含匹配字符串的类数组），如：
```js
const reDate = /([0-9]{4})-([0-9]{2})-([0-9]{2})/;
const match = reDate.exec('2020-05-07');
// 不便于阅读
const year = match[1]; // 2020
const month = match[2]; // 05
const day = match[3]; // 07
```

## 新增
允许命名捕获组使用符?\<name\>，在打开捕获括号(后立即命名，如：
```js
const reDate = /(?<year>[0-9]{4})-(?<month>[0-9]{2})-(?<day>[0-9]{2})/;
const match = reDate.exec('2020-05-07');
const year = match.groups.year; // 2020
const month = match.groups.month; // 05
const day = match.groups.day; // 07

// replace中使用，将时间转化为美国的MM-DD-YYYY格式
const reDate = /(?<year>[0-9]{4})-(?<month>[0-9]{2})-(?<day>[0-9]{2})/;
const d = '2020-05-07';
const result = d.replace(reDate, '$<month>-$<day>-$<year>');
```