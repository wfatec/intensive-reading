# what the heck is the event loop anyway

这是Philip Roberts于2014年做的一次关于时间轮询的讲座，内容原文来自[这里](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html) 。此外，还实现了一个[可视化的事件轮询流程](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)，帮助我们更直观的进行理解。

### JS是单线程吗？

作为一个JS程序员，我们都知道**单线程**，因此经常使用**callbacks**，那么callbacks是如何工作的呢？JS既然是单线程的，那么它是如何实现非阻塞异步调用的呢？

首先，以V8为例介绍一下什么是JS的Runtime，先介绍其中的几个重要概念：

* 堆\(heap\)，在这里进行内存分配。
* 调用堆栈\(call stack\)， 是一个方法列表，按调用顺序保存所有在运行期被调用的方法，stack frames就在这里。
* 堆栈帧\(stack frames\)， 是一个为函数保留的区域，用来存储关于参数、局部变量和返回地址的信息。堆栈帧通常是在新的函数调用的时候创建，并在函数返回的时候销毁。

但是，通过查看V8的源码我们可以惊讶的发现，像setTimeout，DOM或是HTTP请求这类事物却并不在V8中，观察它们的共同点可以得出结论，异步事物不在V8源码中，而是由浏览器或是node.js这类的宿主环境提供的。这是一个非常有趣的现象，由此我们就能更加全面的去看待事件循环机制。

因此，我们可以明确以下结论：

* JS是单线程程序语言
* JS是单线程Runtime
* JS有单个调用堆栈

### JS如何实现异步调用

而尽管JS的Runtime本身是单线程的，但其宿主环境却不是，通常都拥有多个线程，而JS通过调用这些API就能实现异步调用。举个例子：

```javascript
$.on('button', 'click', function onClick() {
    setTimeout(function timer() {
        console.log('You clicked the button!');    
    }, 2000);
});

console.log("Hi!");

setTimeout(function timeout() {
    console.log("Click the button!");
}, 5000);

console.log("Welcome to loupe.");
```

首先为button绑定一个点击事件，然后在控制台输出`Hi!` ，下面调用宿主环境的setTimeout接口，继续向下执行，输出`Welcome to loupe.`，现在我们的调用堆栈执行完了，此时Runtime会对其进行clear操作，而此前的定时器的执行实际上是在宿主环境\(浏览器或Node.js\)中。假设该程序运行在浏览器中，当5s到达后，Web API不能直接开始修改你的代码，即不能像堆栈中插入执行内容，因为如果直接进行插入，这些插入内容可能出现在代码的任意位置。那么这时应该怎么办呢？这里引入了任务队列，或者也可能称为回调队列。宿主会将定时器内的执行函数推送到回调队列\(callback queue\)，但这时，该函数并不会立刻推送到调用堆栈，那么Runtime如何判断何时将回调队列中的函数推送到调用堆栈呢？答案就是今天的主角 ----- **事件轮询**。

**事件轮询的工作**是巡视调用堆栈以及任务队列。如果调用堆栈是空的，则将任务队列中的第一项推送到调用堆栈中，并由调用堆栈运行它。

### 调用堆栈和浏览器的重绘

浏览器每16.6毫秒可以进行重绘，即理想情况下每秒60个frame，这也是浏览器重绘频率的极限。但由于JS调用堆栈中还存在未执行完的代码，浏览器可能无法立刻执行render进行重绘。和回调的情况类似，render必须等到调用堆栈清空时才会执行。两者的区别仅仅是render拥有比callback更高的优先级，它每16毫秒就会将一个rend放入队列，等待调用堆栈清空后执行相应的render。因此，如果我们的JS代码发生阻塞，且阻塞的时长超过16毫秒，就会极大影响浏览器的重绘，导致出现**卡顿**的现象。

因此，当我们发现自己的同步代码中有一些耗时严重的逻辑时，需要仔细考虑是否能够使用异步函数来代替，从而避免页面卡顿的发生，影响用户体验。

