# Rest/Spread

## 历史
es6引入了Rest参数和扩展运算符(...)，但仅用于数组。Rest参数允许我们将一个不定数量的参数表示为一个数组。
```js
restParam(1, 2, 3, 4, 5);
function restParam(p1, p2, ...p3) {}
```

## 新增
为对象解构提供了和数组一样的Rest参数和展开操作符。
```js
const myObj = {
  a: 1,
  b: 2,
  c: 3
}

const {a, ...x} = myObj;
// x = {b: 2, c: 3}
```