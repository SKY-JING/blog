# dotAll模式

## 历史
正则表达式中点(.)匹配除回车外的任何字符。
```js
/hello.world/.test('hello\nworld'); // false
```

## 调整
标记s改变这种行为，允许行终止符出现。
```js
/hello.world/s.test('hello\nworld'); // true
```