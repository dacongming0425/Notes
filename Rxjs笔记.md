## Rxjs学习笔记，每日学习更新

## 基础知识点

1.Observable

2.Observer

3.创建Observable

## Observer
观察者（Observer）是一个有三个方法的对象

|方法名|作用|
|-|-|
|next|当Observable 发出新的值时被调用，接收这个值作为参数|
|complete|当 Observable 完结，没有更多数据时被调用。**complete 之后，next 方法无效**|
|error|当 Observable 内部发生错误时被调用，之后不会调用 complete，next 方法无效|

**例子**

```js
const source = new Observable(observer => {
    observer.next(1);
    observer.next(2);
    observer.complete();
    observer.next(3);
})
const observer = {
    next: function(number: any) {
        console.log(number)
    },
    complete: function() {
        console.log('complete');
    }
}
console.log('start');
source.subscribe(observer);
console.log('end');
```
上面的代码会输出1、2、complete,不会输出3。因为在complete()处就已经结束了。

**简写**

Observer还有简单形式，即不用构造一个对象，而是直接把函数作为subscribe的参数传入。参数依次为
1.next
2.error
3.complete
```js
source.subscribe(
    (item:any) => {
        console.log(item);
    },
    (err) => {
        console.log(err);
    },
    () => {
        console.log('complete');
    }
)
```

## 退订(unsubscribe)
观察者想要退订，只要调用订阅返回的对象的```unsubscribe```方法，这样观察者就不会再接收到Observable的信息了。
```js
const source = new Observable(observer => {
    let number = 1;
    setInterval( () => {
        observer.next(number++);
    }, 1000);
});
const observer = {
    next: function(item : any) => {
        console.log(item);
    }
}
const subscription = source.subscribe(observer);
setTimeout( () => {
    subscription.unsubscribe();         // 5s之后退订
}, 5000)
```

## 操作符
在RxJs中，操作符是用来处理数据流的。往往需要对数据流做一系列处理，才交给Observer，这时操作符就像一个管道一样，数据进入管道，完成处理，流出管道。



```js
import {Observable} from 'rxjs';
const source = new Observable(observer => {
    let number = 1;
    setInterval(() => {
        observer.next(number++);
    }, 1000)
})
const observer = {
    next : function(item:any) {
        console.log(item)
    }
}
console.log('start1');
source.subscribe(observer);
console.log('start2');
```
**interval操作符**

interval 操作符创造了一个数据流，interval(1000) 会产生一个每隔 1000 ms 就发出一个从 0 开始递增的数据。

**map操作符**

map 操作符和数组的 map 方法类似，可以对数据流进行处理。
这个 map 和数组的 map 方法会产生新的数组类似，它会产生新的 Observable。每一个操作符都会产生一个新的 Observable，不会对上游的 Observable 做任何修改，这完全符合函数式编程“数据不可变”的要求。

**pipe操作符**

pipe 方法就是数据管道，会对数据流进行处理，上面的例子只有一个 map 操作符进行处理，可以添加更多的操作符作为参数。

## 创建Observable
注意和操作符不同，他们是从rxjs中导入，而不是rxjs/operators

### of方法
可以使用of方法对之前的方法进行简写。
```js
// 简写前
const source = new Observable(observer => {
    observer.next(1);
    observer.next(2);
    observer.next(3);
    observer.complete();
})
```

```js
// 简写后
import {of} from 'rxjs';
const source = of(1,2,3);
```

### from方法
使用from方法简写上面的代码
```js
// 简写后
import {from} from 'rxjs';
const source = from([1,2,3]);
```

from将可遍历对象（iterable）转化为一个Observable，字符串也有iterator接口，所以也支持。

from还可以根据promise创建一个Observable。用fetch或者axios等类库发送的请求都是一个promise对象，可以使用from将其处理为一个Observable对象。

### fromEvent方法
用DOM事件创建Observable，第一个参数为DOM对象，第二个参数为事件名称。
```js
import {fromEvent} from 'rxjs';
import {take} from 'rxjs/operators';
const eleBtn = document.querySelector('#btn');
const _click = fromEvent(eleBtn, 'click');      // 第一个参数为DOM对象，第二个参数为事件名称
_click.pipe(take(1)).subscribe(e => {
    console.log('只可点击一次');
    eleBtn.setAttribute('disabled', '');
})
```

### fromEventPattern方法
将添加事件处理器、删除事件处理器的API转化为Observable。（具体代码看RxJs教程，前端涉及较多，不过多理解）

### interval和timer
interval和JS的setInterval类似，参数为时间间隔，下面的代码每隔1000ms会发出一个递增的函数
```js
interval(1000).subscribe(console.log)       // 0,1,2,3....
```

timer则可以接收两个参数
参数列表

|参数名|说明|可省|类型|
|-|-|-|-|
|需要等待的时间|发出第一个值需要等待的时间|否|**数字**或**Date**|
|间隔时间|之后的间隔时间|是|any|

### range
操作符of产生较少的数据时可以直接写成```of(1,2,3)```，但是如果是100个数据时，我们就用range。
```js
range(1,100).subscribe(console.log);        // 产生1-100的正整数
```

### empty、throwError、never
|方法|作用|备注|
|-|-|-|
|empty|创建一个**立即完成**的Observable|
|throwError|创建一个**抛出错误**的Observable|
|never|创建一个**什么也不做**的Observable|不完结、不吐出数据、不抛出错误|

目前官方不推荐使用empty和never方法，而是推荐使用**常量EMPTY和NEVER**（不是方法，已经是一个Observable对象了）

### defer
defer创建的Observable只有**在订阅时才会去创建**真正想要操作的Observable。defer延迟了创建Observable，而又有一个Observable方便去订阅，这样也就**推迟了占用资源**。
```js
defer(() => ajax(ajaxUrl));     // 只有订阅了才会去发送ajax请求
```

## 操作符
操作符其实看作是处理数据流的管道，每个操作符实现了针对某个小的具体应用问题的功能，RxJS 编程最大的难点其实就是**如何去组合这些操作符从而解决的问题**。

RxJs有各种各样的操作符：

- 转化类
- 过滤类
- 合并类
- 多播类
- 错误处理类
- 辅助工具类
- ...

操作符是一个函数，一般不需要自己去实现操作符。但是实现的时候必须考虑以下功能：

> 1.返回一个全新的 Observable 对象
> 2.对上游和下游的订阅和退订处理
> 3.处理异常情况
> 4.及时释放资源
之前版本的RxJs各种操作符都挂载到了全局的Observable对象上，现在分离出来需要如下引入才能使用：

```js
import {filter, map} from 'rxjs/operators';
source.pipe(
    filter( x => x % 2 === 0),
    map( x => x * 2)
)
```

### pipe
pipe就是管道的意思，数据流通过操作符处理，流出然后交给下一个操作符。

### 几个类似数组方法的基础操作符
|操作符|说明|备注|
|-|-|-|
|map|和数组的map类似|-|
|filter|和数组的filter类似|-|
|scan|和reduce类似|-|
|mapTo|将所有发出的数据映射到一个给定的值|-|

**mapTo实例**


```js
import {interval} from 'rxjs';
import {mapTo} from 'rxjs/operators';
const observable = interval(1000).pipe(
    mapTo('Hi')
)
observable.subscribe(
    x => console.log(x)
)
```
每隔1000ms就会输出一个'Hi';

### 一些过滤的操作符
|操作符|说明|备注|
|-|-|-|
|take|从数据流中选取最先发出的若干数据|-|
|takeLast|从数据流中选取最后发出的若干数据|-|
|takeUntil|从数据流中选取知道发生某种情况前发出的若干数据|-|
|first|获得满足判断条件的第一个数据|-|
|last|获得满足判断条件的最后一个数据|-|
|skip|从数据流中忽略最先发出的若干数据|-|
|skipLast|从数据流中忽略最后发出的若干数据|-|

**用take举例**

```js
import {interval} from 'rxjs';
import {take} from 'rxjs/operators';
imterval(1000).pipe(
    take(3)
).subscribe(
    x => console.log(x),
    null,
    () => console.log('complete')
)
// 输出：0、1、2、complete
```
使用了take(3)，表示只取3个数据，Observable就进入完结状态。

```js
import { interval, fromEvent } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
interval(1000).pipe(
    takeUntil(fromEvent(document.querySelector('#btn'), 'click'))
).subscribe(
    x => { document.querySelector('#time').textContent = x + 1 },
    null,
    () => console.log('complete')
```
这里有一个interval一直发出数据，直到当用户点击了按钮时才停止。
