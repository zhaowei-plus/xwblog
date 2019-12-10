---
title: Rxjs学习入门
date: 2019-11-12 10:39:35
tags: [rxjs, 响应式编程]
categories: 响应式编程
---

**前言**

​	RxJS 是 Reactive Extensions for JavaScript 的缩写，起源于 Reactive Extensions，是一个基于可观测数据流在异步编程应用中的库，是基于观察者模式和迭代器模式并已函数式编程思维来实现。



使用Observables的响应式编程的库，可以更容易的编写异步货基于回调处理的代码，采用观察者模式、迭代器模式和函数编程思想，基于流的概念对数据进行处理

​    在介绍Rxjs之前，先来了解下两种设计模式：观察者模式和迭代器模式

**观察者模式**

观察者模式最常见的应用场景就是 Js Dom事件的监听和触发

- 订阅：通过 addEventListener 订阅 document.body 的 click 事件
- 发布：当body节点被点击时，body节点便会向订阅者发布这个消息

```
document.body.addEventListener('click', function listener(e) {
    console.log(e);
},false);

document.body.click(); // 模拟用户点击
```

**迭代器模式**

迭代器模式可以用JavaScript 提供的Iterable Protocol可迭代协议来表示

```
var iterable = [1, 2];
var iterator = iterable[Symbol.iterator]();

var iterator = iterable();

while(true) {
    try {
        let result = iterator.next();  // <= 获取下一个值
    } catch (err) {
        handleError(err);  // <= 错误处理
    }
    if (result.done) {
        handleCompleted();  // <= 无更多值（已完成）
        break;
    }
    doSomething(result.value);
}
```

迭代器模式主要对应三种情况：

- 获取下一个值：调用next 将元素一个一个返回，可支持多次调用
- 已完成：无更多值时，next返回元素中 done 为true
- 错误处理：当next方式执行时报错，则会抛出 error 事件，用 try catch 包裹进行错误处理



Rxjs中其他基本概念：

​	Observables  被观察者，是一个值或事件的流稽核

​	Observer  观察者，根据Observables进行处理

​	Observables 与 Observer 之间的订阅与发布关系（观察者模式）如下：

- 订阅：Observer 通过 Observable 提供的 subscribe() 方法订阅 Observable
- 发布：Observable 通过调用 next 方法向 Observer 发布事件
- 

Rxjs 的 Observer有三个方法接收 Observable 的事件，与迭代器模式一一对应：

```
var Observer = {
    next(value) { /* 处理值*/ },
    error(error) { /* 处理异常 */ },
    complete() { /* 处理已完成态 */ }
};
```





响应式编程：

Reactive

Lodash from events

Observable

Stream-based

使用Observables的响应式编程的库，可以更容易的编写异步货基于回调处理的代码，采用观察者模式、迭代器模式和函数编程思想，基于流的概念对数据进行处理

流：node stream， gulp中的管道流都是基于流数据进行处理

Rxjs Vs Promise

Promise之后能将一个数据的状态由Pending转换为resolved或者rejected，并且不可截取

Rxjs可以处理多个数据对应的complete和error状态，但是Rxjs同时拥有Next方法，可以发射多个值，是对Promise，callbacks，Web Workers，Web Sockets进行统一的优化，一旦我们统一了这些概念后，我们将能更好地进行开发

使用场景：

1、在线聊天室：https://baijiahao.baidu.com/s?id=1629682822857132150&wfr=spider&for=pc

2、页面滚动事件（原生实现做节流处理）

3、交互处理

4、异步请求

5、组件封装：自动搜索、拖拽实现

通常异步处理的方法：

1、回调函数

2、事件

3、Promise

4、Generator

需求分为两种：

- 分发
- 流程

处理分发需求的时候，回调、事件或者类似订阅发布这种模式比较合适

处理流程性质的需求时，Promise和Generator比较合适

Rxjs则结合了Promise和Generator两种模式，它的每个Observable都是可订阅的，而Observable之间的关系，则能够体现流程（Rxjs里面的流程的控制和处理，其直观性略强于Promise，但弱于Generator）

可以把一切输入作为数据流里进行处理，如：

- 用户操作
- 网络响应
- 定时器
- Worker

操作：

1、创建数据流的方法

- 单值：of, empty, never
- 多值：from
- 定时：interval, timer
- 从事件创建：fromEvent
- 从Promise创建：fromPromise
- 自定义创建：create

创建处理的数据流时一种可观测的序列，可以被订阅，也可以被用来做一些转换操作

2、操作符

转换操作符：对可观测之进行转换

- 改变数据形态：map, mapTo, pluck
- 过滤一些值：filter, skip, first, last, take
- 时间轴上的操作：delay, timeout, throttle, debounce, audit, bufferTime
- 累加：reduce, scan
- 异常处理：throw, catch, retry, finally
- 条件执行：takeUntil, delayWhen, retryWhen, subscribeOn, ObserveOn
- 转接：switch

合并操作符：可以对若干个数据流进行合并

- concat，保持原来的序列顺序连接两个数据流
- merge，合并序列
- race，预设条件为其中一个数据流完成
- forkJoin，预设条件为所有数据流都完成
- zip，取各来源数据流最后一个值合并为对象
- combineLatest，取各来源数据流最后一个值合并为数组

Rxjs中的相关概念：

Observable 可观测序列，只发射出值，提供数据源，不接受任何响应

Observer 观察者，只接受悲观则数据源发出的值

Subject 既是可观测值，也是观察者

RelaySubject

BehaviourSubject

ReplaySubject

Subscription 订阅关系

管道

Subject：

Rxjs中的Subject一种特殊的Observable，允许将值多播给多个观察者，而普通的Observable时单播的，即每个已订阅的观察者都拥有Observable的独立执行，Subject本身既可以是Observable也可以是Observer，

Subject：普通Subject没有缓存数据，订阅者在数据源发射数据之后订阅，是拿不到之前发射的值的，

const subject = new Subject() // 订阅之前发射的值时拿不到的 subject.next('100') // 订阅者在数据源发射数据之后订阅，拿不到之前的数据 subject.subscribe(text => console.log('subscribeA:', text)) subject.subscribe(text => console.log('subscribeB:', text)) // 订阅之后发射的值才能拿到 subject.next('200') subject.next('300') 

打印数据

![img](C:/Users/Administrator/AppData/Local/YNote/data/weixinobU7VjgTbU4is0xFdNj_oMXBtYDI/6249bc78c00642b98ae486a11c170d10/clipboard.png)

Subject有几个子类，可以缓存部分或全部数据，在订阅时拿到数据执行处理

BehaviorSubject：创建时需要传递一个默认值，在订阅后会获取上一次发射的值

 // BehaviorSubject 会存储最后一次发射的数据 const subject1 = new BehaviorSubject(0) subject1.next(1000) subject1.next(2000) subject1.subscribe(val => console.log('BehaviorSubject subscribeA:', val)) subject1.next(3000) subject1.subscribe(val => console.log('BehaviorSubject subscribeB:', val)) subject1.next(4000) 

打印结果：

![img](C:/Users/Administrator/AppData/Local/YNote/data/weixinobU7VjgTbU4is0xFdNj_oMXBtYDI/be764ef22e7842088ce8b488a931c246/clipboard.png)

ReplaySubject：会缓存所有数据，当有新的订阅者的时候，发射缓存的所有值

const subject2 = new ReplaySubject() subject2.next(1000) subject2.next(2000) subject2.subscribe(val => console.log('ReplaySubject subscribeA:', val)) subject2.next(3000) subject2.subscribe(val => console.log('ReplaySubject subscribeB:', val)) subject2.next(4000) 

打印结果：

![img](C:/Users/Administrator/AppData/Local/YNote/data/weixinobU7VjgTbU4is0xFdNj_oMXBtYDI/d1f5e11fcd944862ac93d978649eee6f/clipboard.png)

AsyncSubject 和BehaviorSubject一样只会存储最后一次发出的数据，但是AsyncSubject 只会在complete时把数据发射出去

// AsyncSubject 和BehaviorSubject一样只会存储最后一次发出的数据，但是AsyncSubject 只会在complete时把数据发射出去 const subject4 = new AsyncSubject() subject4.next(1000) subject4.next(2000)     subject4.subscribe(val => console.log('AsyncSubject subscribeA:', val))     subject4.next(3000)     subject4.subscribe(val => console.log('AsyncSubject subscribeB:', val))     subject4.next(4000)     subject4.complete(); // 只有在 complete时把上一次缓存的值发射出去     // 会存储之前的值     subject4.subscribe(val => console.log('AsyncSubject subscribeC:', val))     subject4.next(5000) // 收不到complete 之后的值 

打印结果：

![img](C:/Users/Administrator/AppData/Local/YNote/data/weixinobU7VjgTbU4is0xFdNj_oMXBtYDI/f218588266c44cc4983c4dfd3a819954/clipboard.png)

总结：

四种Subject有各自的特性，可以根据下表来做详细区分

![img](C:/Users/Administrator/AppData/Local/YNote/data/weixinobU7VjgTbU4is0xFdNj_oMXBtYDI/8d8eb28944d247108f92074087bf287d/clipboard.png)

https://www.cnblogs.com/Unknw/p/6440808.html



Rxjs 与 Promise的比较：

​	Promise功能单一，只能将一个数据状态有pending转换成resolved或者rejected；

​	Promise内部状态不可控制，执行了就无法终止；



​	Rxjs可以处理多个数据对应的complete和error状态，并且Rxjs同时还有next方法，可以额外执行与状态无关的操作；

​	Rxjs中Observable可以定义如何取消异步方法，必要的时候终止操作



Rxjs Observable

​	Rxjs中Observable和函数一样是惰性的，即表示只有在subscribe的时候才输出值，并且Observable可以定义取消异步方法