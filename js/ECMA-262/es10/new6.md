# Symbol.prototype.description
通过工厂函数Symbol()创建符号时，您可以选择通过参数提供字符串作为描述：
```js
const sym = Symbol('description');
```

## 历史
访问描述的唯一方法是将符号转化为字符串：
```js
assert.equal(String(sym), 'Symbol(description)');
```

## 新增
现在可以通过Symbol.prototype.description直接访问
```js
assert.equal(sym.description, 'description');
```

## 参考资料
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/description