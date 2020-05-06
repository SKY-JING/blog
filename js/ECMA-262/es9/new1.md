# 异步迭代

## 一、历史
在async/await的某些时刻，你可能尝试在同步循环中调用异步函数，但结果不是你想要的，如：
```js
// 循环本身依旧是同步，并在内部异步函数之前全部调用完成
async function process(array) {
  for (let i of array) {
    await doSomething(i);
  }
}
// 或者
async function process(array) {
  array.forEach(async i => {
    await doSomething(i);
  });
}
```

## 二、改进
引入异步迭代器（asynchronous iterators）,就像常规迭代器，除了next()方法返回一个Promise。因此await可以和for..of循环一起使用，以串行的方式运行异步操作。如：
```js
async function process(array) {
  for await (let i of array) {
    doSomething(i);
  }
}
```