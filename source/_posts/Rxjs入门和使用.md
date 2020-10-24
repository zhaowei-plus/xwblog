---
title: Rxjs学习入门
date: 2019-11-12 10:39:35
tags: [Rxjs, 响应式编程]
categories: Rxjs
---
> by zhaowei 2019/12/19

[TOC]

## 前言

​	在学习Rxjs之前，先让我们先了解下**函数式编程**和**响应式编程**，**观察者模式**和**迭代器模式**，以及**拉取 (Pull)** 与**推送 (Push)**
协议的区别

### 函数式编程

 函数式编程（通常简称为 FP）是指通过复合纯函数来构建软件的过程，它避免了共享的状态（share state）、易变的数据(mutable data)、以及副作用(side-effects)。

### 响应式编程

响应式编程（通常简称为 RP）是一种从数据流和变化出发的解决问题的模式。

### 观察者模式

观察者模式最常见的应用场景就是 Js Dom 事件的监听和触发
订阅：通过 addEventListener 订阅 document.body 的 click 事件
发布：当 body 节点被点击时，body 节点便会向订阅者发布这个消息

```javascript
document.body.addEventListener('click', function listener(e) {
    console.log(e);
},false);

document.body.click(); // 模拟用户点击
```

### 迭代器模式

迭代器模式可以用JavaScript 提供的Iterable Protocol可迭代协议来表示

``` javascript
var iterable = [1, 2];
var iterator = iterable[Symbol.iterator]();

var iterator = iterable();

while(true) {
    try {
        let result = iterator.next(); // <= 获取下一个值
    } catch (err) {
        handleError(err); // <= 错误处理
    }

    if (result.done) {
        handleCompleted(); // <= 无更多值（已完成）
        break;
    }  
    doSomething(result.value);
}
```
主要对应三种情况：
-   获取下一个值：调用 next 将元素一个一个返回，可支持多次调用
-   已完成：无更多值时，next 返回元素中 done 为 true
-   错误处理：当 next 方式执行时报错，则会抛出 error 事件，用 try catch 包裹进行错误处理

### 拉取 (Pull) vs. 推送 (Push)

 拉取和推送是两种不同的协议，用来描述数据生产者 (Producer)如何与数据消费者 (Consumer)进行通信的。

![avatar](./images/1.jpg)

-   Function 是惰性的评估运算，调用时会同步地返回一个单一值。
-   Generator 是惰性的评估运算，调用时会同步地返回零到(有可能的)无限多个值。
-   Promise 是最终可能(或可能不)返回单个值的运算。
-   Observable 是惰性的评估运算，它可以从它被调用的时刻起同步或异步地返回零到(有可能的)无限多个值。

值得获取关系如下：

![avatar](./images/2.jpg)


## RxJS基本概念

​RxJS 是 Reactive Extensions for JavaScript 的缩写，起源于 Reactive Extensions，是一个基于**可观测数据流**在异步编程应用中的库（可以理解为异步的lodash），是基于观察者模式和迭代器模式并以函数式编程来实现的库。

Rxjs使用Observables的响应式编程的库，可以更容易的编写异步货基于回调处理的代码，采用观察者模式、迭代器模式和函数编程思想，基于流的概念对数据进行处理

Rxjs基本概念如下：

-   **Observable（可观察对象）**： 表示一个概念，这个概念是一个可调用的未来值或事件的集合。

-   **Observer（观察者）**： 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。

-   **Subscription（订阅）**：表示 Observable 的执行，它主要用于取消 Observable 的执行。

-   **Operator（操作）**： 采用函数式编程风格的纯函数，使用像 map、filter、concat、flatMap 等这样的操作符来处理集合。

-   **Subject（主体）**： 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。

-   **Schdulers（调度器）**：用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 setTimeout 或 requestAnimationFrame 或其他。

### Observable – 可观察对象

Observable 作为观察者，是一个值或事件的流集合，简单来说就是数据在 Observable 中流动，消费者可以使用各种 operator 对流进行处理，获取想要的结果。
observable 有三个方法：next，error，complete，分别发出不同类型的通知

创建 Observables
订阅 Observables
执行 Observables
清理 Observable 执行

### Observer – 观察者

观察者是由 Observable 发送的值的消费者。
观察者只是一组回调函数的集合，每个回调函数对应一种 Observable 发送的通知类型：next、error 和 complete 。

```javascript
var observer = {
    next: x => console.log('Observer got a next value: ' + x),
    error: err => console.error('Observer got an error: ' + err),
    complete: () => console.log('Observer got a complete notification'),
};

```
​
使用观察者，需要把它提供给 Observable 的 subscribe 方法：

```javascript
observable.subscribe(observer);
```

### Subscription - 订阅

Subscription 是表示可清理资源的对象，通常是 Observable 的执行。Subscription 有一个重要的方法，即 unsubscribe，它不需要任何参数，只是用来清理由 Subscription 占用的资源。

```javascript
var observable = Rx.Observable.interval(1000);
var subscription = observable.subscribe(x => console.log(x));
// 稍后：
// 这会取消正在进行中的 Observable 执行
// Observable 执行是通过使用观察者调用 subscribe 方法启动的
subscription.unsubscribe();
```

多个Subscription 还可以合在一起，这样一个 Subscription 调用 unsubscribe() 方法，可能会有多个 Subscription 取消订阅 。（涉及到多播的概念）

### Operators - 操作符

操作符是 Observable 类型上的方法，比如 .map(...)、.filter(...)、.merge(...)，等等。当操作符被调用时，它们不会改变已经存在的 Observable 实例。相反，它们返回一个新的 Observable ，它的 subscription 逻辑基于第一个 Observable 。
操作符是函数，它基于当前的 Observable 创建一个新的 Observable。这是一个无副作用的操作：前面的 Observable 保持不变。

-   1、创建数据流的操作符
    -   单值：of, empty, never
    -   多值：from
    -   定时：interval, timer
    -   从事件创建：fromEvent
    -   从 Promise 创建：fromPromise
    -   自定义创建：create

-   2、转换操作符
    -   改变数据形态：map, mapTo, pluck
    -   过滤一些值：filter, skip, first, last, take
    -   时间轴上的操作：delay, timeout, throttle, debounce, audit, bufferTime
    -   累加：reduce, scan
    -   异常处理：throw, catch, retry, finally
    -   条件执行：takeUntil, delayWhen, retryWhen, subscribeOn, ObserveOn
    -   转接：switchMap

-   3、合并操作符
    -   concat，保持原来的序列顺序连接两个数据流
    -   merge，合并序列
    -   race，预设条件为其中一个数据流完成
    -   forkJoin，预设条件为所有数据流都完成
    -   zip，取各来源数据流最后一个值合并为对象
    -   combineLatest，取各来源数据流最后一个值合并为数组

可以查看用例学习先关操作符：[案例
](https://www.jianshu.com/p/59d64562daa0
)

### Subject – 主体

RxJS Subject 是一种特殊类型的 Observable，它允许将值多播给多个观察者，所以 Subject 是多播的，而普通的 Observables 是单播的(每个已订阅的观察者都拥有 Observable 的独立执行)。
Subject 像是 Observable，但是可以多播给多个观察者。Subject 还像是 EventEmitters，维护着多个监听器的注册表。

**每个 Subject 都是 Observable**

-   对于 Subject，你可以提供一个观察者并使用 subscribe 方法，就可以开始正常接收值。从观察者的角度而言，它无法判断 Observable 执行是来自普通的 Observable 还是 Subject 。在 Subject 的内部，subscribe 不会调用发送值的新执行。它只是将给定的观察者注册到观察者列表中，类似于其他库或语言中的 addListener 的工作方式。

**每个 Subject 都是 Observer**

-  Subject 是一个有如下方法的对象： next(v)、error(e) 和 complete() 。要给 Subject 提供新值，只要调用 next(theValue)，它会将值多播给已注册监听该 Subject 的观察者们。

普通 Subject 没有缓存数据，订阅者在数据源发射数据之后订阅，是拿不到之前发射的值的，

```javascript
const subject = new Subject()

// 订阅之前发射的值时拿不到的
subject.next('100')

// 订阅者在数据源发射数据之后订阅，拿不到之前的数据
subject.subscribe(text => console.log('subscribeA:', text))
subject.subscribe(text => console.log('subscribeB:', text))

// 订阅之后发射的值才能拿到
subject.next('200')
subject.next('300')
```
![avatar](./images/3.jpg)

Subject 有几个子类，可以缓存部分或全部数据，在订阅时拿到数据执行处理，具体区别可以查看下面代码测试

**BehaviorSubject**

BehaviorSubject在创建时需要传递一个默认值，在订阅后会获取上一次发射的值

```javascript
// BehaviorSubject 会存储最后一次发射的数据
const subject1 = new BehaviorSubject(0)

subject1.next(1000)
subject1.next(2000)
subject1.subscribe(val => console.log('BehaviorSubject subscribeA:', val))
subject1.next(3000)
subject1.subscribe(val => console.log('BehaviorSubject subscribeB:', val))
subject1.next(4000)
```
![avatar](./images/4.jpg)

**ReplaySubject**

ReplaySubject会缓存所有数据，当有新的订阅者的时候，发射缓存的所有值

```javascript
const subject2 = new ReplaySubject()

subject2.next(1000)
subject2.next(2000)
subject2.subscribe(val => console.log('ReplaySubject subscribeA:', val))
subject2.next(3000)
subject2.subscribe(val => console.log('ReplaySubject subscribeB:', val))
subject2.next(4000)
```

![avatar](./images/5.jpg)

**AsyncSubject**

AsyncSubject 和 BehaviorSubject 一样只会存储最后一次发出的数据，但是 AsyncSubject 只会在 complete 时把数据发射出去

```javascript
// AsyncSubject 和 BehaviorSubject 一样只会存储最后一次发出的数据，但是 AsyncSubject 只会在 complete 时把数据发射出去
const subject4 = new AsyncSubject()
subject4.next(1000)
subject4.next(2000)
subject4.subscribe(val => console.log('AsyncSubject subscribeA:', val))
subject4.next(3000)
subject4.subscribe(val => console.log('AsyncSubject subscribeB:', val))
subject4.next(4000)
subject4.complete(); // 只有在 complete 时把上一次缓存的值发射出去
// 会存储之前的值
subject4.subscribe(val => console.log('AsyncSubject subscribeC:', val))
subject4.next(5000) // 收不到 complete 之后的值
```
![avatar](./images/6.jpg)

四种 Subject 有各自的特性，可以根据下表来做详细区分

![avatar](./images/7.jpg)

Subject相关内容查看案例：[RxJS 源码解读之 Subject
](https://zhuanlan.zhihu.com/p/51493958)

### Schedulers – 调度器

调度器控制着何时启动 subscription 和何时发送通知。它由三部分组成：

-   调度器是一种数据结构。 它知道如何根据优先级或其他标准来存储任务和将任务进行排序。
-   调度器是执行上下文。 它表示在何时何地执行任务(举例来说，立即的，或另一种回调函数机制(比如 setTimeout 或 process.nextTick)，或动画帧)。

-   调度器有一个(虚拟的)时钟。 调度器功能通过它的 getter 方法 now() 提供了“时间”的概念。在具体调度器上安排的任务将严格遵循该时钟所表示的时间。

调度器可以让你规定 Observable 在什么样的执行上下文中发送通知给它的观察者。


## 总结

RxJS 是一个库，它通过使用 observable 序列来编写异步和基于事件的程序。可以把 RxJS 当做是用来处理事件的 Lodash 。RxJS 结合了 观察者模式、迭代器模式 和 使用集合的函数式编程，以满足以一种理想方式来管理事件序列所需要的一切。

Rxjs 可以处理多个数据对应的 complete 和 error 状态，但是 Rxjs 同时拥有 Next 方法，可以发射多个值，是对 Promise，callbacks，Web Workers，Web Sockets 进行统一的优化，一旦我们统一了这些概念后，将能更好地进行开发

**Rxjs生态**

Rxjs生态相对React，Vue，Angular等框架来说不算火，主要是因为学习成本比较高，但是各个框架都有对应的资源库支持，有兴趣可以详细研究

-   [Rxjs6官网](https://cn.rx.js.org/manual/usage.html
)

-   [Rsjx操作符文档](https://rxjs-cn.github.io/learn-rxjs-operators/
)

-   [Angular Rxjs运用](https://blog.csdn.net/Handsome_fan/article/details/82940875
)

-   [React rxjs-hook](https://juejin.im/entry/5bfdf27a6fb9a049a7118314
)

-   [redux-observable](https://juejin.im/post/5c9c79d0f265da610763090d
)

-   [探秘 vue-rx 2.0](https://juejin.im/entry/582678b68ac24700595fdc2d
)

-   [rxdb](https://github.com/pubkey/rxdb
)


**Rxjs使用场景**

-   异步请求
-   页面滚动事件（原生实现做节流处理）
-   交互处理（自动搜索、拖拽实现）
-   项目数据管理 实践案例[（DaoCloud 基于 RxJS 的前端数据层实践 ）](https://zhuanlan.zhihu.com/p/28958042
)


## 参考文章

-   [RxJS 介绍](https://www.jianshu.com/p/5d01341599e9)

-   [学习 RxJS 操作符](https://rxjs-cn.github.io/learn-rxjs-operators/)

-   [Rxjs 官网中文版](https://cn.rx.js.org/)

-   [DaoCloud 基于 RxJS 的前端数据层实践](https://zhuanlan.zhihu.com/p/28958042)

-   [RxJS 游戏之贪吃蛇](https://zhuanlan.zhihu.com/p/35457418)

-   [函数式和响应式的 JavaScript 框架，编写可观测代码 Cycle.js](http://cyclejs.cn/)
