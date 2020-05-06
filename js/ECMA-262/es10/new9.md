# catch绑定

## 一、历史
try-catch语法中catch语句必须绑定异常变量，无论是否需要，如：
```js
// 必须绑定e
try {} catch (e) {}
```

## 二、改进
简化书写结构
```
try {} catch {}
```