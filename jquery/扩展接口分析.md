<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">扩展接口分析</div>

从jQuery整体框架一文我们知道jquery提供了一个扩展接口extend，支持在jquery上扩展和在jQuery.fn上扩展属性或方法，在看extend源码前我们先看看其具体能做哪些事情。

# 一、jQuery用法
## 1.1 扩展jQuery本身
```js
// 扩展
jQuery.extend({test: '11'});

// 执行
console.log(jQuery.test); // 11
// 注：不需要实例jQuery就可以使用
```
## 1.2 扩展jQuery.fn
```js
// 扩展
jQuery.fn.extend({
  test: function () {
    console.log('111');
    return this; // 支持链式操作
  }
})

// 执行
$().test(); // 111
```
## 1.3 扩展对象
```js
// 扩展
var ob1 = {test: '11'},
    ob2 = {test: '22', show: false};
var ob3 = jQuery.extend(ob1, ob2);

// 执行
console.log(JSON.stringify(ob1)); // {test: '22', show: false}
console.log(JSON.stringify(ob2)); // {test: '22', show: false}
// 注：ob1对象本身会被修改，返回值为修改后的ob1
```
## 1.4 浅拷贝
```js
// 扩展
var ob1 = {test: {name: 'a', age: 10}, show: false},
    ob2 = {test: {sex: 1}};
var ob3 = jQuery.extend(ob1, ob2);

// 执行
console.log(JSON.stringify(ob1)); // {test: {sex: 1}, show: false}
// 注：ob2中的test同名引用会覆盖ob1中test的引用
```
## 1.5 深拷贝
```js
// 扩展
var ob1 = {test: {name: 'a', age: 10}, show: false},
    ob2 = {test: {sex: 1}};
var ob3 = jQuery.extend(true, ob1, ob2);

// 执行
console.log(JSON.stringify(ob1)); // {test: {name: 'a', age: 10, sex: 1}, show: false}
```

# 二、jQuery源码
从上面看到extend不止可以扩展jQuery本身和jQuery.fn上的属性或方法，还可以扩展对象，并提供了深拷贝和浅拷贝两种方式，现在我们直接看源码：
```js
jQuery.extend = jQuery.fn.extend = function() {
  var options, name, src, copy, copyIsArray, clone,
    target = arguments[ 0 ] || {}, // 设置target为第一个参数（|| {}避免参数不存在引起错误）
    i = 1, // 参数中扩展属性或方法开始位置
    length = arguments.length, // 提取参数长度为局部变量，避免循环时每次重新计算
    deep = false; // 是否支持深度拷贝（false: 不支持，true:支持）

  // 当第一个参数传入的值为boolean类型时，表示当前值设置是否需要深度拷贝
  // Handle a deep copy situation
  if ( typeof target === "boolean" ) {
    deep = target; // 直接将deep标签设置成第一个参数值

    // 跳过第一个参数，将target设置成第二个参数
    // Skip the boolean and the target
    target = arguments[ i ] || {};
    // 扩展属性或方法位置自加一
    i++;
  }

  // 当target既不是对象也不是函数时需特殊处理，将target设置成{}作为需要扩展的对象
  // 如：jQuery.extend('hello world', {test: '11'})中hello world为字符串，舍弃，使用{}替代
  // Handle case when target is a string or something (possible in deep copy)
  if ( typeof target !== "object" && !jQuery.isFunction( target ) ) {
    target = {};
  }

  // 扩展属性或方法位置正好等于参数长度，表示当前扩展的对象是jQuery本身或是jQuery.fn
  // Extend jQuery itself if only one argument is passed
  if ( i === length ) {
    // 将target指向当前this对象，通过this回答了jQuery整体框架一文中一个方法实现不同功能的问题。
    // this指向jQuery时，扩展jQuery本身、this指向jQuery.fn时，扩展jQuery.fn
    target = this;
    // 由于数组下标从0开始，故判断完成后准备遍历参数做扩展之前将i自减1
    i--;
  }
  
  // 前面已经准备好了target作为需要扩展的对象
  // 此处开始遍历需要扩展到target上的参数
  for ( ; i < length; i++ ) {
    
    // 将当前数组元素赋值给局部变量options，避免多次访问arguments数组
    // 仅扩展非null/undefined的元素
    // Only deal with non-null/undefined values
    if ( ( options = arguments[ i ] ) != null ) {
      
      // 遍历第i个元素所有可遍历属性
      // Extend the base object
      for ( name in options ) {
        // 根据被扩展对象的键获取源target对象中相应的值赋值给src
        src = target[ name ];
        // 获取被扩展对象的值
        copy = options[ name ];
        
        // 当被扩展元素和源target相同时跳过，避免无限循环
        // 如：var a = {test: '11'}; var ob = {test: a}; $.extend(a, ob); console.log(a);
        // 在没有此段时a = {test: a}，a是object类型，里面有字段test，值是a，a又是object类型...
        // 此时会出现无限循环，故jQuery中不操作直接跳过此字段
        // Prevent never-ending loop
        if ( target === copy ) {
          continue;
        }
        
        // 深度拷贝，copy存在并且是纯对象或数组时
        // Recurse if we're merging plain objects or arrays
        if ( deep && copy && ( jQuery.isPlainObject( copy ) ||
          ( copyIsArray = Array.isArray( copy ) ) ) ) {

          // 如果是数组
          if ( copyIsArray ) {
            // 将copyIsArray标志重新设置为false，为下次遍历做准备
            copyIsArray = false;
            // 检查源对象中src是否为数组，不是则直接将源src设置为[]
            clone = src && Array.isArray( src ) ? src : [];
          
          // 如果是纯对象
          } else {
            // 检查源对象中src是否为纯对象，不是则直接将源src设置为{}
            clone = src && jQuery.isPlainObject( src ) ? src : {};
          }

          // 递归调用extend方法，继续进行深度遍历
          // Never move original objects, clone them
          target[ name ] = jQuery.extend( deep, clone, copy );

        // 不是深拷贝，直接将copy的值赋值到对应target的属性上
        // Don't bring in undefined values
        } else if ( copy !== undefined ) {
          target[ name ] = copy;
        }
      }
    }
  }

  // 返回修改后的对象
  // 当target指向的源对象不是基础类型时，源对象会被改变，如果不想修改源对象，可将target传入{}
  // Return the modified object
  return target;
};
```
至此我们对源码已经熟悉通透了，对下面的结果表现应该能自行分析了：
```js
var ob1 = {test: '11'},
    ob2 = {test: ['22', '33'], show: false},
    ob3 = {test: ob1};
var ob4 = $.extend({}, ob1, ob2);
var ob5 = $.extend(ob1, ob3);

console.log(ob1);
// {test: "11"} 
// 因为target传入的{}故将扩展属性全扩展到了{}上，不会修改ob1

console.log(ob4);
// {"test":["22","33"],"show":false}
// 因为源test非数组，扩展时被设置为[]，11被舍弃，源test为对象同样理解

console.log(ob5);
// {test: "11"}
// 虽然ob3中test指向了ob1，但是为了避免无限循环，直接跳过不处理

// 切记扩展jQuery/jQuery.fn和扩展对象不能混用，否则会出错，如：
jQuery.extend({
  test: function () {
    console.log(11);
    return this;
  }
}, {
  aa: function () {
    console.log(22);
    return this;
  }
})

$.test();
// 报错，因为当传入了两个object时，扩展是对第一个参数扩展，故不会将test和aa扩展到期望的jQuery上
// 修改为
jQuery.extend({
  test: function () {
    console.log(11);
    return this;
  },
  aa: function () {
    console.log(22);
    return this;
  }
})
$.test().aa();
```