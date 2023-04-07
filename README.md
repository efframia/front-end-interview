# front-end-interview

# JavaScript
## JavaScript数据类型
ECMAScript标准定义8种数据类型：
### 基本类型
Number（数字），String（字符串），Boolean（布尔），Null（空），Undefined（未定义），Symbol（es6），BigInt（es10）
### 引用类型
Object
### Symbol
es6特性，主要用于创建唯一标识符；不可被let in遍历
```
const s = Symbol('key')
const a = {
  [s]: 'value'
}
```
### BigInt
解决大整数运算
```
const a = 1n  //  声明方式：数字后面添加n
console.log(typeof a)
```

## JavaScript类型检测的方法
### typeof
返回类型字符串。typeof null会返回object，历史遗留问题；typeof [1,2,3]会返回object
### instanceof
基于原型链操作，原理：判断右边构造函数的prototype是否出现在左边实例对象的原型链上
### Object.prototype.toString
js中较为精确判断变量类型的方法，它能够区别对象，数组，函数
```
console.log(Object.prototype.toString.call({ a: 1 })); //[object Object]
console.log(Object.prototype.toString.call([1, 2, 3])); //[object Array]
console.log(Object.prototype.toString.call(function() {})); //[object Function]
```

## 堆内存和栈内存
### 栈（stack）
- 自动分配相对固定大小的内存空间，并由系统自动释放
- 栈数据结构遵循FILO（first in last out）先进后出的原则
- 基础数据类型存放在栈内存中，值与值之间相互独立
### 堆（heap）
- 动态分配内存，内存大小不固定，也不会自动释放
- 堆数据结构是一种无序的树状结构，存储方式是key-value键值对
- 引用数据类型存放在堆内存中，在堆内存中分配一个新的空间，栈内存中保存该空间的内存地址，因此引用类型的变量保存的是内存地址<br>
![image](https://user-images.githubusercontent.com/60378935/230064781-0e4608c0-981c-48e9-a10f-ff11a0d48e2c.png)


## this的指向
### 箭头函数
无视其它规则，this指向箭头函数创建时的作用域
### 关键字new绑定
1. 创建一个空对象
2. 空对象的__proto__指向构造函数的原型对象prototype上
3. 执行构造函数中的代码，给空对象添加新属性
4. 有返回值，且返回值类型为对象，则返回；如果函数不返回任何东西，默认返回这个新对象
### 显式绑定
call、apply或者bind，this指向第一个传参
### 隐式绑定/默认绑定
```
function foo() { 
  console.log(this.bar); 
} 
var bar = "bar1"; 
var o2 = {bar: "bar2", foo: foo}; 
var o3 = {bar: "bar3", foo: foo}; 
foo();            // "bar1" – 默认绑定
o2.foo();         // "bar2" – 隐式绑定
o3.foo();         // "bar3" – 隐式绑定
```
- 隐式绑定：this指向调用对象
- 默认绑定：无调用对象，非严格模式下，this是全局对象，浏览器中就是window。严格模式下，this是undefined
  
## 原型、原型链有什么特点？
### 原型
函数有显式原型对象prototype，包含指向构造函数指针constructor；引用类型（对象、数组、函数）有隐式对象__proto__
### 原型链
prototype chain，原型对象也可能拥有原型，并从中继承方法和属性。一切对象都是继承自Object对象，Object.prototype.__proto__ === null，null为原型链最顶端
![image](https://user-images.githubusercontent.com/60378935/228116525-221853e5-f748-42f9-bdca-13ca91df1c5b.png)

## var、let和const的区别？
### 共同点
都会被变量提升
### 不同点
- let和const定义的变量都会被提升，但是不会被初始化，不能被引用
- let和const只在块级作用域中有效
- let和const不允许重复声明
- let和const不会绑定全局作用域
- 暂时性死区：指声明之前使用变量，就会报错

## bind、call和apply的区别？分别如何实现？
### 共同点
显式绑定，让this指向第一个传参。如果第一个传参是null或者undefined，this默认是全局对象，浏览器中就是window
### 不同点
- bind:返回一个函数，可分开传入参数
```
// bind的实现
Function.prototype._bind = function(context) {
  // 对调用者进行判断，只有函数类型才可以调用
  if (typeof this !== 'function') {
    throw new Error();
  }
  // 保存被调用的方法
  var _this = this;
  // 保存实参
  var args = [...arguments].slice(1);
  // 返回新的函数
  return function newFn() {
    // 判断是否执行new操作，如果为new操作，通过apply绑定为实例的this，同时合并调用返回函数的实参
    if (this instanceof newFn) {
      return _this.apply(this, args.concat([...arguments]));
    } else {
      return _this.apply(context, args.concat([...arguments]));
    }
  }
}
```
- call:剩余传参须单独传入，返回函数执行结果
```
// call的实现
Function.prototype._call = function(context) {
  // 对调用者进行判断，只有函数类型才可以调用
  if (typeof this !== 'function') {
    throw new Error();
  }
  context = context || window;
  // 将被调用方法定义在context.fn上，通过对象调用的方式绑定this
  context.fn = this;
  var args = [...arguments].slice(1);
  var result = context.fn(...args);
  // 删除该方法，避免污染对象
  delete context.fn;
  return result;
}
```
- apply:剩余传参放在一个数组中传入，返回函数执行结果
```
// apply的实现
Function.prototype._apply = function(context) {
  // 对调用者进行判断，只有函数类型才可以调用
  if (typeof this !== 'function') {
    throw new Error();
  }
  context = context || window;
  // 将被调用方法定义在context.fn上，通过对象调用的方式绑定this
  context.fn = this;
  var result;
  if(arguments[1]) {
    result = context.fn(...arguments[1]);
  } else {
    result = context.fn();
  }
  delete context.fn;
  return result;
}
```

## new操作符具体干了什么？如何实现？
1. 创建一个空对象
2. 空对象的__proto__指向构造函数的原型对象prototype上
3. 执行构造函数中的代码，给空对象添加新属性
4. 有返回值，且返回值类型为对象，则返回；如果函数不返回任何东西，默认返回这个新对象
```
function myNew(Func) {
  var obj = {};
  obj.__proto__ = Func.prototype;
  var result = Func.apply(obj, [...arguments].slice(1))
  return result instanceof Object? result : obj;
}
```

## instanceof的作用？如何实现？
检测构造函数的prototype是否出现在某个实例的原型链上
```
function _instanceof(left, right) {
  const prototype = right.prototype;
  let proto = left.__proto__;
  while(true) {
    if(proto === null || proto === undefined) {
      return false;
    }
    if(prototype === proto) {
      return true;
    } 
    proto = proto.__proto__;
  }
}
```

## 对闭包的理解？
### 含义
可以在一个内层函数中访问到其外层函数的作用域
### 作用
- 阻止变量被GC垃圾回收，在函数执行完成，由于变量被引用，延长变量的生命周期
- 函数外部可以读取函数内部的数据，利用这一特性可以实现私有变量和私有方法
### 缺点
使用不当造成内存泄漏：由于闭包会使得函数内部变量长期保存在内存当中，不会被GC垃圾回收机制回收
### 应用
- 围绕2点作用：1、创建私有变量 2、延长变量的生命周期
- 计数器、延迟调用、回调
### 注意事项
创建新的对象或者类时，方法通常应该关联于对象的原型，而不是定义到对象的构造器中。原因是每个实例的创建，都会重新声明一遍方法，实际只在原型对象上声明一次就可以了

# CSS
## 对盒子模型的理解？
### 盒子模型是什么？
当浏览器对一个文档进行布局时，浏览器会依据盒模型原则将每一个元素渲染成为一个矩形盒子。这个矩形盒子会有四个属性，从内到外分别是content、padding、border、margin。
- content：实质内容，例如文本或图片。
- padding 表示的是内边距。
- border 表示的盒子的边框。
- margin 表示的是盒子的外边距。
### 标准盒模型和IE盒模型
- 标准盒模型：width、height代表的是content的宽高
- IE盒模型：width、height代表的是content + padding + border的宽高
可以使用一个css属性：box-sizing来切换盒模型模式：
- box-sizing: content-box; 对应 标准盒模型
- box-sizing: border-box; 对应 IE盒模型
### 浏览器渲染后盒模型最终大小是多少？
- 浏览器最终显示盒子的大小 = content + padding + border，这个公式是固定的，不管什么类型的盒子，计算宽高时都必须是这三个东西的结果。
- 我们自己定义的width属性的含义，在不同情况下是不同的：
  - 在标准盒模型中，width代表的是content的宽度，那么要计算其在浏览器渲染后盒模型最终大小，还要加上盒模型的padding和border；
  - 而在IE盒模型中，width代表的是content + padding + border，所以width就是盒模型最终在浏览器渲染后显示的宽大小。

## 元素水平垂直居中的方法有哪些？如果元素不定宽高呢？
### 行内元素居中布局
- 水平居中：行内元素可设置：text-align: center
- 垂直居中：单行文本父元素确认高度：height === line-height

### 块级元素居中布局
  - 定宽：margin auto
  - 绝对定位+left/top:50%+margin:负自身一半
  - flex布局设置父元素：display: flex; justify-content: center; align-items: center;

# Vue
## 父子组件生命周期顺序？

# 网络
## 对跨域的理解？
- 跨域：浏览器从一个“域”向另一个“域”的服务器发出请求，访问另一个“域”上的资源
- 同源策略：请求的文件可能会存在恶意攻击，浏览器并不允许直接访问另一个“域”上的资源，只能访问同一个“域”上的资源。所谓“同源”，指的是“协议（http、https）、域名、端口号”一致
- 解决跨域问题：JSONP、CORS、反向代理
### JSONP
利用script标签，向服务器请求一个js文件，把数据放在函数传参。只能处理GET请求
### CORS
新增一组 HTTP 首部字段，允许服务器声明哪些源站，可以通过浏览器有权限访问哪些资源。在老式浏览器上不可行
- 简单请求
  - 请求方法是HEAD、GET、POST，且头信息不超过Accept、Accept-Language、Content-Language、Last-Event-ID、Content-Type
  - 浏览器向服务器发起的request header中，origin字段表示当前的“源”。服务器返回的response header中，Access-Control-Allow-Origin字段，里面写明允许那些“源”，浏览器发现两者匹配，或者服务器允许所有的“源”，那么跨域成功
- 非简单请求
  - 不同时满足以上两个条件
  - 在通信前增加“预检”请求（OPTIONS请求）。通过预检后，后续通信返回规则同简单请求。
### 反向代理
利用代理服务器接收到请求之后，转发给真正的服务器，再把结果返回到浏览器上

# Webpack
## Webpack的构建流程？

# 参考链接
[web前端面试 - 面试官系列](https://github.com/febobo/web-interview)<br>
[Jason前端博客](https://my-bucket-1258873497.cos-website.ap-guangzhou.myqcloud.com/)
