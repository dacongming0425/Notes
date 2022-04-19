## 周末总结最近的js原生，每日学习更新
****javascript原生知识点****
1. 基本类型有哪几种？null 是对象吗？基本数据类型和复杂数据类型存储有什么区别？
基本类型有6种，分别是undefined,null,bool,string,number,symbol(ES6新增)。
虽然 typeof null 返回的值是 object,但是null不是对象，而是基本数据类型的一种。
基本数据类型存储在栈内存，存储的是值。
复杂数据类型的值存储在堆内存，地址（指向堆中的值）存储在栈内存。当我们把对象赋值给另外一个变量的时候，复制的是地址，指向同一块内存空间，当其中一个对象改变时，另一个对象也会变化。

2、三种类型判断
typeof：能判断原始值，比如string number boolean symbol undefined。 会把null判断成object（之前看到文章说这是一个javascript的bug）。 不能判断引用类型的值，比如array，Date等类型都会判断为object，但是唯独能判断function。
instanceof 比如A instanceof B 通过B的原型链层层查找，一直找到null为止。注意instanceof不能正确判断原始类型的值，可以试一试 console.log(1 instanceof Number);
instanceof的实现代码:


```js
// L instanceof R
function instance_of(L, R) {//L 表示左表达式，R 表示右表达式
    var O = R.prototype;// 取 R 的显式原型
    L = L.__proto__;    // 取 L 的隐式原型
    while (true) { 
        if (L === null) //已经找到顶层
            return false;  
        if (O === L)   //当 O 严格等于 L 时，返回 true
            return true; 
        L = L.__proto__;  //继续向上一层原型链查找
    } 
}


Object.prototype.toString     任何类型都能判断
`function type(object){
    var toString = Object.prototype.toString.call(object);
    return /s+(S+)]/.exec(toString)[1].toLowerCase();
}`

var x = [];
console.log(type(x));
```
3、浅拷贝与深拷贝
浅拷贝：
for循环遍历key
Object.assign
ES6的对象展开运算符 ...
深拷贝
递归遍历（DFS，环形问题，边界处理
JSON.parse(JSON.stringify())
该方法存在一些局限性，会忽略 undefined，会忽略 symbol，不能序列化函数，不能解决循环引用的对象。但是它还是能解决大部分深度拷贝的问题。

4、for...of for...in forEach map的区别
for...of 只能遍历 ''iterator对象'' 一个对象如果要具备可被for...of循环调用的 Iterator 接口，就必须在Symbol.iterator的属性上部署遍历器生成方法，而原生为iterator对象的比如数组，generator对象，string等。
for...in 遍历对象的自身或者继承的可枚举的属性
forEach 只能遍历数组 ，不能阻断循环
map 遍历数组
forEach和map都是数组的原型方法

4、类数组和数组的区别
类数组只是一个普通对象，它拥有length属性，但是没有数组的原型方法，比如pop push 等
数组是 构造函数Array的实例。
常见的类数组有：函数的参数arguments，DOM 对象列表(比如通过 document.querySelectorAll 得到的列表), jQuery 对象 (比如 $("div")).
转换方法有：

//第一种方法
Array.prototype.slice.call(arrayLike, start);
//第二种方法
[...arrayLike];
//第三种方法:
Array.from(arrayLike);
5、== 和 === 的区别
== 会先进行隐式转换然后再进行比对，下面是转换步骤
1、先比对类型是否相同，如果相同则比较值
2、比较两边是否是 null 和 undefined ，若是返回true
3、判断两边是否是原始值，若是转换成 number 再进行比较。（这里比如bool number string都统一转成number，注意是在两边类型不同的情况下）
4、一遍是object，一遍是原始值， 现将object转成原始值，再比较 （关于object到底以什么样的规则转换成原始值，需要思考）

思考: [] == ![] [] == []
这个问题就是 []会转换成一个什么原始值
那么将object也转换成Number,空数组转换成数字，对应的值是0.(空数组转换成数字，对应的值是0，如果数组中只有一个数字，那么转成number就是这个数字，其它情况，均为NaN)
而 ![] 转成 0 所以返回 true [] == 0 true [1] == 1 [2] == 2
而 [] == [] 两边类型相同，不转换类型，因为是引用类型，地址不同 所以返回false
感觉还有点滑稽

6、 ES6中的class和ES5的类有什么区别？
ES6 class 内部所有定义的方法都是不可枚举的;
ES6 class 不存在变量提升;
ES6 class 子类必须在父类的构造函数中调用super()，这样才有this对象;ES5中类继承的关系是相反的，先有子类的this，然后用父类的方法应用在this上。

7、let、const 以及 var 的区别是什么？
let const有块级作用域， var不存在块级作用域
let const 不存在变量提升，拥有暂时性死区和不允许重复声明， var有变量提升
let const 在全局作用域下不会绑定在window对象下，而var会，这是因为let const的声明是在词法环境下。

8、对JS执行上下文栈和作用域链的理解。
首次运行代码时，会创建一个全局的执行上下文，push进当前的执行栈，每当发生函数调用时，都会生成一个该函数的执行上下文，并push进当前执行栈的栈顶

9、闭包
在A函数中生成或者返回B函数，B函数可以继续使用A函数的局部变量或者方法，那么B函数就称为闭包。
那么B函数为什么可以使用A函数的局部变量呢？
普通情况下A函数执行完后其执行上下文就会弹出执行=执行上下文栈，并由js引擎清除A函数的变量对象，返还内存空间。
而如果A函数生成/返回了B函数，A函数的变量对象将会被保留在内存空间中，供B函数继续使用，所以闭包容易造成内存泄漏。
闭包的使用场景：
1、封装私有变量
2、模仿块级作用域
3、实现js模块
柯里化 防震动 节流等

10、call apply和bind的实现
call的简单实现，利用this的隐式绑定
```js
Function.prototype.call = function (context) {
    /** 如果第一个参数传入的是 null 或者是 undefined, 那么指向this指向 window/global */
    /** 如果第一个参数传入的不是null或者是undefined, 那么必须是一个对象 */
    if (!context) {
        //context为null或者是undefined
        context = typeof window === 'undefined' ? global : window;
    }
    context.fn = this; //this指向的是当前的函数(Function的实例)
    let rest = [...arguments].slice(1);//获取除了this指向对象以外的参数, 空数组slice后返回的仍然是空数组
    let result = context.fn(...rest); //隐式绑定,当前函数的this指向了context.
    delete context.fn;
    return result;
}
```
```js
//测试代码
var foo = {
    name: 'Selina'
}
var name = 'Chirs';
function bar(job, age) {
    console.log(this.name);
    console.log(job, age);
}
bar.call(foo, 'programmer', 20);
// Selina programmer 20
bar.call(null, 'teacher', 25);
// 浏览器环境: Chirs teacher 25; node 环境: undefined teacher 25
apply的实现，只是apply接受的参数是数组的形式

Function.prototype.apply = function (context, rest) {
    if (!context) {
        //context为null或者是undefined时,设置默认值
        context = typeof window === 'undefined' ? global : window;
    }
    context.fn = this;
    let result;
    if(rest === undefined || rest === null) {
        //undefined 或者 是 null 不是 Iterator 对象，不能被 ...
        result = context.fn(rest);
    }else if(typeof rest === 'object') {
        result = context.fn(...rest);
    }
    delete context.fn;
    return result;
}
var foo = {
    name: 'Selina'
}
var name = 'Chirs';
function bar(job, age) {
    console.log(this.name);
    console.log(job, age);
}
bar.apply(foo, ['programmer', 20]);
// Selina programmer 20
bar.apply(null, ['teacher', 25]);
// 浏览器环境: Chirs programmer 20; node 环境: undefined teacher 25
bind的实现 闭包返回和参数的拼接

`Function.prototype.bind = function (context){
    if(typeof this !== "function"){
        throw new TypeError("not a function");
    }`

`    var args = [...arguments].slice(1);//参数
    var _this = this;
    var binded = function (){
        args = args.concat([...arguments]);//参数的合并
        return _this.apply(context, args)
    };

    //如果是构造函数，还需要实现prototype的继承
    binded.prototype = Object.create(this.prototype);
    return binded;
}

function test(param1, param2){
    console.log(this.name)
    console.log(param1);
    console.log(param2);
}
var obj = {
    name: 1,
}
var binded = test.bind(obj, 2);
binded(3);
```
11、new的原理是什么？通过new的方式创建对象和通过字面量创建有什么区别？
new的原理，四个步骤
new:
创建一个新对象。
这个新对象会被执行[[原型]]连接。
属性和方法被加入到 this 引用的对象中。并执行了构造函数中的方法.
如果函数没有返回其他对象，那么this指向这个新对象，否则this指向构造函数中返回的对象。

```
function new(func) {
    let target = {};
    target.__proto__ = func.prototype;
    let res = func.call(target);
    if (typeof(res) == "object" || typeof(res) == "function") {
    	return res;
    }
    return target;
}
```
字面量创建对象，不会调用 Object构造函数, 简洁且性能更好;
new Object() 方式创建对象本质上是方法调用，涉及到在proto链中遍历该方法，当找到该方法后，又会生产方法调用必须的 堆栈信息，方法调用结束后，还要释放该堆栈，性能不如字面量的方式。
通过对象字面量定义对象时，不会调用Object构造函数。

12、 对原型的理解
在 JavaScript 中，每当定义一个对象（函数也是对象）时候，对象中都会包含一些预定义的属性。其中每个函数对象都有一个prototype 属性，这个属性指向函数的原型对象。使用原型对象的好处是所有对象实例共享它所包含的属性和方法。

13、 什么是原型链？【原型链解决的是什么问题？】
原型链解决的主要是继承问题。
每个对象拥有一个原型对象，通过 proto 指针指向其原型对象，并从中继承方法和属性，同时原型对象也可能拥有原型，这样一层一层，最终指向 null(Object.proptotype.__proto__指向的是null)。这种关系被称为原型链 (prototype chain)，通过原型链一个对象可以拥有定义在其他对象中的属性和方法。
构造函数 Parent、Parent.prototype 和 实例 p 的关系如下:(p.__proto__ === Parent.prototype)

14、prototype 和 __proto__ 区别是什么？
prototype是构造函数的属性。
__proto__ 是每个实例都有的属性，可以访问 [[prototype]] 属性。
实例的__proto__ 与其构造函数的prototype指向的是同一个对象。
```js
function Student(name) {
    this.name = name;
}
Student.prototype.setAge = function(){
    this.age=20;
}
let Jack = new Student('jack');
console.log(Jack.__proto__);
//console.log(Object.getPrototypeOf(Jack));;
console.log(Student.prototype);
console.log(Jack.__proto__ === Student.prototype);//true
```
15、用ES5实现一个继承
寄生组合

16、ES6新的特性有哪些？
新增了块级作用域(let,const)
提供了定义类的语法糖(class)
新增了一种基本数据类型(Symbol)
新增了变量的解构赋值
函数参数允许设置默认值，引入了rest参数，新增了箭头函数
数组新增了一些API，如 isArray / from / of 方法;数组实例新增了 entries()，keys() 和 values() 等方法
对象和数组新增了扩展运算符
ES6 新增了模块化(import/export)
ES6 新增了 Set 和 Map 数据结构
ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例
ES6 新增了生成器(Generator)和遍历器(Iterator)

17、setTimeout
setTimeout，时间不准确，因为不能保证主线程任务的执行时间
与requestAnimationFrame的区别？
requestAnimationFrame的特点
1、时间准确
2、不掉帧（关于掉帧：浏览器每16.7ms重绘一次，可以想象一台机器每16.7ms处理一个工件，而如果你非要每10ms给机器一个工件去处理，就会导致机器处理不过来，导致任务阻塞，会引起对电池和cpu的损耗）
3、节省CPU开销（当页面处于未激活状态，比如切出该页面，动画不会执行），用setTimeout向下兼容
4、动画平滑连贯（动画与UI同步渲染）

18、为什么 0.1 + 0.2 != 0.3 ?
十进制与二进制的转换
整数位：除2取余 小数位： 乘2取整 0.1转换成二进制是一个无限循环
进制转换和对阶运算中出现精度缺失导致了0.1 + 0.2 != 0.3
怎么解决精度问题？
1.将数字转成整数
这是最容易想到的方法，也相对简单
```js
function add(num1, num2) {
 const num1Digits = (num1.toString().split('.')[1] || '').length;
 const num2Digits = (num2.toString().split('.')[1] || '').length;
 const baseNum = Math.pow(10, Math.max(num1Digits, num2Digits));
 return (num1 * baseNum + num2 * baseNum) / baseNum;
}

```
但是这种方法对大数支持的依然不好
2.三方库
Math.js big.js
网上找到相关的方法参考，还没去学习：https://juejin.im/post/5b90e00e6fb9a05cf9080dff

19、js的异步发展史
1、回调函数
回调函数的使用会导致几个问题：
回调地狱
> 
> fs.readFile(A, 'utf-8', function(err, data) {  
>     fs.readFile(B, 'utf-8', function(err, data) {  
>         fs.readFile(C, 'utf-8', function(err, data) {  
>             fs.readFile(D, 'utf-8', function(err, data) {  
>                 //....  
>             });  
>         });  
>     });  
});
不能try catch获取错误，导致代码难以维护
2、Promise
Promise 主要解决了回调地狱的问题，Promise 最早由社区提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
那么我们看看Promise是如何解决回调地狱问题的，仍然以上文的readFile为例。

``` js
function read(url) {
    return new Promise((resolve, reject) => {
        fs.readFile(url, 'utf8', (err, data) => {
            if(err) reject(err);
            resolve(data);
        });
    });
}
read(A).then(data => {
    return read(B);
}).then(data => {
    return read(C);
}).then(data => {
    return read(D);
}).catch(reason => {
    console.log(reason);
});
```
3、async/await
async/await作为generator的语法糖，也建议了解其原理，里面最重要的知识点就是co模块，手写它一个。
http://es6.ruanyifeng.com/#docs/generator-async

20、对webWorker的理解
HTML5则提出了 Web Worker 标准，表示js允许多线程，但是子线程完全受主线程控制并且不能操作dom，只有主线程可以操作dom，所以js本质上依然是单线程语言。
简单示例：
```js
var worker = new Worker('./worker.js'); //创建一个子线程
worker.postMessage('Hello');
worker.onmessage = function (e) {
    console.log(e.data); //Hi
    worker.terminate(); //结束线程
};

//worker.js           
onmessage = function (e) {
    console.log(e.data); //Hello
    postMessage("Hi"); //向主进程发送消息
};
```
webWorker的特点：
1、开启一个子进程
2、用postMessage和onMessage与主进程进行通信
3、不能操作DOM
webWorker的应用场景一般适用于一些较大计算量，耗时长的场景，比如某个算法的计算。

21、ES6 模块和commonJS的区别
ES6模块在编译时，就能确定模块的依赖关系，以及输入和输出的变量。
CommonJS 模块，运行时加载。
ES6 模块自动采用严格模式，无论模块头部是否写了 "use strict";
require可以做动态加载，import 语句做不到，import 语句必须位于顶层作用域中。
ES6 模块中顶层的 this 指向 undefined，CommonJS 模块的顶层 this 指向当前模块。
CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
CommonJS可以缓存。

22、跨域
1、jsonp
2、cors
http://www.ruanyifeng.com/blog/2016/04/cors.html
3、postMessage和onMessage 在window.open或者postMessage
4、nginx跨域代理
5、websocket 不受同源策略影响
6、node中间件 （vue中开发时使用的代理工具）
node中间件实现跨域的原理如下:
1.接受客户端请求
2.将请求 转发给服务器。
3.拿到服务器 响应 数据。
4.将 响应 转发给客户端。
以下三种跨域方式很少用，还未查阅相关资料。
window.name + iframe
location.hash + iframe
document.domain (主域需相同)

23、js异步加载 async和defer
相同点是 都是异步加载。
async是下载完立即执行，中断渲染引擎（js线程和UI渲染线程是互斥的），不保证执行顺序。
defer是渲染完再执行。 保证顺序。


24、Function.proto(getPrototypeOf)是什么？
获取一个对象的原型，在chrome中可以通过__proto__的形式，或者在ES6中可以通过Object.getPrototypeOf的形式。
那么Function.__proto__是什么？也就是说Function由什么对象继承而来，做乐如下判别。
```js
 Function.__proto__==Object.prototype //false
 Function.__proto__==Function.prototype//true
 ```
 发现Function的原型也是Function。
## 暂时性死区 4.19更新一、

暂时性死区
暂时性死区也叫临时性死区（Temporal Dead Zone），TDZ。let、const声明的变量不会进行变量的提升，如果在声明前访问就会报错：
```js
console.log(userName ) //报错ReferenceError: userName is not defined

let userName = 'Sofia';
```
报错的原因，就是因为userName这个变量此时还存在与暂时性死区里面。

二、变量提升的好坏
好处：

1、提高性能

在预编译阶段，会统计声明的变量、函数，分配空间。

2、增强容错性

在很久很久以前，js只是用来发送表单，那时候还没调试工具，也没eslint等代码检查工具，所以需要一定的容错性。

坏处：
如下代码

```js

       function foo() {
            console.log(userName) // undefined
            if (false) {
                var userName = 'Sofia';
                return userName
            } else {
                return userName
            }
        }

        var value =  foo();
        console.log(value) //undefined
```
吊，在false的情况下，变量userName被创建了

因为存在变量的提升，变量userName被提升到了函数顶部：

```
      function foo() {
            var userName;
            console.log(userName)
            if (false) {
                userName = 'Sofia';
                return userName
            } else {
                return userName
            }
        }
```
看这情况难受，再也不太想用用var了，准备拥抱let、const，没有变量的提升。

既然写到let、const，就总结下和var的区别，方便看看：

1、不存在变量的提升

2、暂时性死区

3、不可重复声明

4、let、const声明的变量不会被挂载到window下

5、块级作用域

  = =：那let和const的区别呢？

1、无论是非严格模式下还是严格模式下，都不可以为const声明的常量再赋值。常量，顾名思义，不变的值。

2、const声明时必须进行初始化赋值。
