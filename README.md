# 2019InterviewSummary

## 2019 年面试总结

### this 变量提升 箭头函数 原型链
   1. this <br>
      Javascript解释器和执行上下文 <br>
      Javascript是一种脚本语言，这意味着代码执行中没有编译步骤。解释器读取代码并逐行执行。正在执行该行的环境（或范围）称为“执行上下文”。Javascript运行时维护这些执行上下文的堆栈，并且当前正在执行存在于该堆栈顶部的执行上下文。每次执行上下文更改时，“this”引用的对象都会更改。<br>
      - “this”指的是全局对象<br>
         默认情况下，执行的执行上下文是全局的，这意味着如果代码作为简单函数调用的一部分执行，则“this”指的是全局对象。“window”对象是浏览器中的全局对象，在Node.JS环境中，一个特殊对象“global”将是“this”的值。<br>
         ```javascript
         function foo(){
            console.log(this === window);
         }
         foo(); // true;
         console.log(this === window); //true
         // IIFE(立即执行函数)
         (function(){
            console.log(this === window);
         })();  // true
         ```
         如果为任何函数启用了严格模式，那么“this”的值将是“undefined”，如在严格模式下，global对象是指未定义的，而不是windows对象。<br>
         ```javascript
         function foo(){
            'use strict';
            console.log(this === window);
         }
         foo();  // false
         // 在控制台上打印为false，因为在“严格模式”中，全局执行上下文中的“this”值未定义。
         ```
      - “this”指的是新实例<br>
         当使用“new”关键字调用函数时，该函数称为构造函数并返回一个新实例。在这种情况下，“this”的值指的是新创建的实例。
         ```javascript
         function Person(fn, ln) {
            this.fname = fn; this.lname = ln;
            this.displayName = function() {
               console.log(`Name: ${this.fname} ${this.lname}`);
            }
         }
         let person = new Person("tom", "ha");
         person.displayName();  // Name: tom ha
         let person2 = new Person("xx", "ss");
         person2.displayName(); // Name: xx ss
         ```
      - “this”指的是调用者对象（父对象）<br>
         在Javascript中，对象的属性可以是方法或简单值。调用Object的方法时，“this”指的是包含被调用方法的对象。
         ```javascript
         function foo() {
            'use strict';
            console.log(this);
            console.log(this === window);
         }
         let user = {
            num: 10,
            foo: foo,
            foo1: function() {
               console.log(this);
               console.log(this === window);
            }
         }
         user.foo(); // {num:10,foo: f, foo1: f} false  此处this为user
         let fun1 = user.foo1;
         fun1(); // Window{.....}  true  普通函数调用 this为全局变量window;
         user.foo1();  // {num:10, foo: f, foo1: f} false 此处this为user;
         ```
      - “this”通过call，apply方法改变<br>
         javascript中的函数也是一种特殊类型的对象。每个函数都有调用(call)，绑定(bind)和应用(apply)方法。这些方法可用于将“this”的自定义值设置为函数的执行上下文。
         ```javascript
         function Person(fn, ln) {
            this.fname = fn; this.lname = ln;
            this.displayName = function() {
               console.log(`Name: ${this.fname} ${this.lname}`);
            }
         }
         let person = new Person("tom", "a");
         person.displayName(); // Name: tom a;
         let person2 = new Person("jack", "b");
         person2.displayName(); // Name: jack b;
         // call和apply方法之间的唯一区别是参数的传递方式。在apply的情况下，第二个参数是一个参数数组，在call方法的情况下，参数是单独传递的。
         person.displayName.call(person2, 11); // Name: jack b;
         person.displayName.apply(person2, [11]); //Name: jack b;
         var xx = person.displayName.bind(person2);
         xx(); // Name: jack b;
         ```
      - “this” 箭头函数中 this with fat arrow function<br>
         当使用胖箭头(fat来源 ->细箭头 =>胖箭头; 实际就是我们说的箭头函数)时，它不会为“this”创建新值。“this”继续引用它所指的同一个对象，在函数之外。<br>
         在箭头函数中，this与封闭词法环境的this保持一致。在全局代码中，它将被设置为全局对象：
         ```javascript
         var globalObject = this;
         var foo = (() => this);
         console.log(foo() === globalObject); // true
         // 作为对象的一个方法调用
         var obj = {foo: foo};
         console.log(obj.foo() === globalObject); // true
         // 尝试使用call来设定this
         console.log(foo.call(obj) === globalObject); // true
         // 尝试使用bind来设定this
         foo = foo.bind(obj);
         console.log(foo() === globalObject); // true
         ```
         无论如何，foo 的 this 被设置为他被创建时的环境（在上面的例子中，就是全局对象）。这同样适用于在其他函数内创建的箭头函数：这些箭头函数的this被设置为封闭的词法环境的。<br>
         ```javascript
         // 创建一个含有bar方法的obj对象，
         // bar返回一个函数，这个函数返回this，
         // 这个返回的函数是以箭头函数创建的，
         // 所以它的this被永久绑定到了它外层函数的this。
         // bar的值可以在调用中设置，这反过来又设置了返回函数的值。
         var obj = {
            bar: function() {
               var x = (() => this);
               return x;
            }
         };
         // 作为obj对象的一个方法来调用bar，把它的this绑定到obj。
         // 将返回的函数的引用赋值给fn。
         var fn = obj.bar();
         // 直接调用fn而不设置this，
         // 通常(即不使用箭头函数的情况)默认为全局对象
         // 若在严格模式则为undefined
         console.log(fn() === obj); // true
         // 但是注意，如果你只是引用obj的方法，
         // 而没有调用它
         var fn2 = obj.bar;
         // 那么调用箭头函数后，this指向window，因为它从 bar 继承了this。
         console.log(fn2()() == window); // true
         ```
      * 总结<br>
         * 默认情况下，“this”指的是全局对象，在NodeJS的情况下是全局的，在浏览器的情况下是窗口对象
         * 当一个方法被称为对象的属性时，那么“this”指的是父对象
         * 当使用“new”运算符调用函数时，“this”指的是新创建的实例。
         * 当使用call和apply方法调用函数时，“this”指的是作为call或apply方法的第一个参数传递的值。
         * 箭头函数不创建this新值 定义时就得到this的指向
         * this在运行时确定
      <br>
   2. 变量提升 <br>
      - 
   3. 箭头函数 <br>
   4. 原型链 <br>








### vue react 小程序 生命周期 store 相关

1. vue 生命周期 <br>

   1. init 阶段 <br>
      beforeCreate created
   2. compile create 阶段 <br>
      beforeMount mounted
   3. mounted 阶段 <br>
      beforeUpdate updated
   4. destroy 阶段 <br>
      beforeDestroy destroyed

   <div align="center"> <img src="./imgs/vuelifecycle.png" width="120" height="300" alt="vue生命周期图" /></div>

2. react 生命周期 <br>
   1.
