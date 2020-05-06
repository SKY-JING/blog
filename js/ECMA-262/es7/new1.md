# Array.prototype.includes()

## 历史
es7之前需要使用indexOf验证数组中是否存在某个元素，根据返回值是否为-1判断，如：
```js
let arr = ['angular', 'react', 'vue'];
if (arr.indexOf('vue') !== -1) {
  console.log('vue存在');
}
```

## 新增
使用includes更加直观
```js
let arr = ['angular', 'react', 'vue'];
if (arr.includes('vue')) {
  console.log('vue存在');
}
```

## 参考文档
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes