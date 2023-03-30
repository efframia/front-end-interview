# front-end-interview

# JavaScript
## this的指向
- 箭头函数：无视其它规则，this指向箭头函数创建时的作用域<br>
- 关键字new绑定：
  1. 创建一个空对象
  2. 空对象链接到构造函数的原型上
  3. 新对象被绑定为this
  4. 有返回值，且返回值类型为对象，则返回；如果函数不返回任何东西，默认返回这个新对象
- 显式绑定：call、apply或者bind，this指向第一个传参
- 隐式绑定/默认绑定
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
原型：prototype，每个对象、构造函数都有自己的原型对象<br>
原型链：prototype chain，原型对象也可能拥有原型，并从中继承方法和属性
一切对象都是继承自Object对象，Object.prototype.__proto__ === null，null为原型链最顶端
![image](https://user-images.githubusercontent.com/60378935/228116525-221853e5-f748-42f9-bdca-13ca91df1c5b.png)

## var、let和const的区别？
- 共同点：都会被变量提升
- 不同点：
  - let和const定义的变量都会被提升，但是不会被初始化，不能被引用
  - let和const只在块级作用域中有效
  - let和const不允许重复声明
  - let和const不会绑定全局作用域
  - 暂时性死区：指声明之前使用变量，就会报错

## bind、call和apply的区别？分别如何实现？
- 共同点：显式绑定，让this指向第一个传参。如果第一个传参是null或者undefined，this默认是全局对象，浏览器中就是window
- 不同点：
  - bind:返回一个函数，可分开传入参数
    ```
    Function.prototype.myBind = function() {
        var thatFunc = this, 
            thatArg = arguments[0];
        var args = Array.prototype.slice.call(arguments, 1)
        if (typeof thatFunc !== 'function') {
            throw new TypeError('Function.prototype.bind - ' +
                 'what is trying to be bound is not callable');
        }
        var fBound  = function() {
            return thatFunc.apply(this instanceof fBound
                     ? this
                     : thatArg,
                     args.concat(Array.prototype.slice.call(arguments)));
            };
        var fNOP = function() {};       // 讲道理看得不是很懂，先跳过了。。。
        if (thatFunc.prototype) {
          fNOP.prototype = thatFunc.prototype; 
        }
        fBound.prototype = new fNOP();
        return fBound;
    }
    ```
  - call:剩余传参须单独传入，返回函数执行结果
    ```
    Function.prototype.myOwnCall = function(context) {
      context = context || window;
      var uniqueID = "00" + Math.random();                  // 防止覆盖原有函数
      while (context.hasOwnProperty(uniqueID)) {
        uniqueID = "00" + Math.random();
      }
      context[uniqueID] = this;

      var args = [];                                        // 这一步有待商榷，call跟apply一样传入的是入参数组，有问题，待我跟博主battle一下
      for (var i = 1; i < arguments.length; i++) {  
        args.push("arguments[" + i + "]");
      }
      var result = eval("context[uniqueID](" + args + ")"); // 不允许使用...时
      delete context[uniqueID];                             // 只是临时改变this指向，因此要把这个方法在运行之后删掉
      return result;
    }
    ```
  - apply:剩余传参放在一个数组中传入，返回函数执行结果
    ```
    Function.prototype.myOwnApply = function(context, arr) {
      context = context || window
      var uniqueID = "00" + Math.random();
      while (context.hasOwnProperty(uniqueID)) {
        uniqueID = "00" + Math.random();
      }
      context[uniqueID] = this;

      var args = [];
      var result = null;

      if (!arr) {                     // 比起call，只是多了传参为空的判断逻辑
        result = context[uniqueID]();
      } else {
        for (var i = 0; i < arr.length; i++) { 
          args.push("arr[" + i + "]");
        }
        result = eval("context[uniqueID](" + args + ")");
      }
      delete context[uniqueID];
      return result;
    }
    ```

## new操作符具体干了什么？如何实现？
1. 创建一个空对象
2. 空对象链接到构造函数的原型上
3. 新对象被绑定为this
4. 有返回值，且返回值类型为对象，则返回；如果函数不返回任何东西，默认返回这个新对象
```
function myNew() {
  var constr = Array.prototype.shift.call(arguments);
  var obj = Object.create(constr.prototype);
  var result = constr.apply(obj, arguments);
  return result instanceof Object? result : obj;
}
```
