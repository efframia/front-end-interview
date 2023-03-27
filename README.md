# front-end-interview

# JavaScript
## this的指向
- 箭头函数：无视其它规则，this指向箭头函数创建时的作用域<br>
- 关键字new绑定：
  1. 创建一个空对象
  2. 空对象链接到原型对象上
  3. 原型对象被绑定为this
  4. 如果函数不返回任何东西，默认return this
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
  - 默认绑定：无调用对象，非严格模式下，this是全局对象，浏览器中就是window。严格模式下，this是undefined<br>
## bind、call和apply的区别？分别如何实现？
  - 共同点：显式绑定，让this指向第一个传参。如果第一个传参是null或者undefined，this默认是全局对象，浏览器中就是window
  - 不同点：
    - bind:返回一个函数，可分开传入参数
      
    - call:剩余传参须单独传入，返回函数执行结果
      ```
      Function.prototype.myOwnCall = function(context) {
        context = context || window;
        var uniqueID = "00" + Math.random();                  // 防止覆盖原有函数
        while (context.hasOwnProperty(uniqueID)) {
          uniqueID = "00" + Math.random();
        }
        context[uniqueID] = this;

        var args = [];
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
