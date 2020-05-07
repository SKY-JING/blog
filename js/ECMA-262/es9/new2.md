# Promise.finally()

## 一、历史
一个Promise调用链要么成功达到最后一个.then()，要么失败触发.catch()。

## 二、新增
在某些情况下，你想要在无论Promise运行成功还是失败，运行相同的代码，例如清除、删除对话、关闭数据库连接等。
```js
doSomething()
  .then(data => {})
  .catch(e => {})
  .finally(() => {})
```