# Making Sense of React Hooks

本期文章出自Redux作者  [Dan Abramov](https://medium.com/@dan_abramov) ，原文在 [这里](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)，主要探讨了React Hooks\(钩子\)设计的初衷和使用方式，以及对于现有的开发模式所产生的影响。此外 [对React Hooks的一些思考](https://zhuanlan.zhihu.com/p/48264713) 一文也对其做出了非常独到的评述。

### 为什么使用Hooks？

事实上，React的开发模式随着React API的演变，也发生了不小的变化，从早期的 Mixin，再到后来的 HOC；在开发框架上也有Flux，Redux 以及后来的 Mobx 先后大型其道。但这些演变并没有完全解决一些开发上的痛点，主要包括以下三点：

* 超大组件，使得最终变得难以维护
* 重复的逻辑，主要存在于不同组件以及生命周期方法之间
* 复杂的模式，例如render props和高阶组件的出现

这三点其实也可以归结为一点，即**逻辑复用**，这是一个很重要的概念，尽管React通过组件的概念实现了UI元素级别的代码复用，但是对于更加细颗粒度的代码复用却无能为力，之前的方法普遍是通过HOC来不断嵌套，从而实现组件的组合和逻辑串联，但这里显然会增加代码的维护成本，在大量嵌套的情况下，会导致最终的代码难以阅读。

此外，由于组件必须是需要实现 render 方法的类或函数，对于一些无需渲染的纯逻辑片段，显然无能为力。可能有人会说将这些逻辑封装到函数中抽离出来即可，没错，此前的开发也多数是如此做的，但是如果这个函数里需要用到 state 等React内置API呢？此时我们只能在组件中定义 state ，而后引入该函数并绑定到组件上，在相应的生命周期或事件下处罚该函数，但是这样会导致这个函数与组件高度耦合。通过render props和高阶函数似乎可以解决代码耦合的问题，但是却又会带来设计模式的复杂化，增加开发成本。由此，Hooks应运而生，作为React官方推出的解决方案，Hooks的出现带来了设计模式的极大简化，提高了代码的复用性！

### Hooks是什么？

其实 **Hooks 就是普通的JavaScript函数**。这意味着我们此前的所有的基于函数的编程思想在这里都能够无缝使用，因此我们可以自由的对Hooks进行组合使用，并组成自定义的Hooks。由于Hooks的引入，使得我们的函数具有了更强的抽象能力和颗粒度，因而带来了更加高效的代码复用能力。

### 实例展示

话不多说，我们通过 Dan 提供的一个例子来感受一下 Hooks 的魅力吧！

我们需要实现一个组件，当视窗的宽度发生变化时，动态显示其宽度，组件定义如下：

```javascript

function MyResponsiveComponent() {
  const width = useWindowWidth(); // Our custom Hook
  return (
    <p>Window width is {width}</p>
  );
}
```

抽象逻辑定义如下：

```javascript
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
  
  return width;
}
```

是不是非常简介呢？通过 useEffect，我们将原本需要在多个生命周期中定义的方法，都统一到了一起，省却了很多重复代码。

事实上，React 一直都有统一状态变化相关 API 的倾向，从此前引入 getDerivedStateFromProps\(\) 方法就能看出端倪，对引起重新渲染的原因不再做区分。

### 关于Classes

我们似乎可以告别使用类来定义组件的时代了，Hooks的出现解决了函数式组件无法原生使用生命周期的问题，是时候真正拥抱函数式组件了！

当然，React 团队也承诺不会废弃 classes 相关 API ，去哦们似乎也不用为重构而苦恼了。

### Hooks实现原理

事实上和 state的使用类似，Hooks 的使用同样有一些约束条件，例如必去在顶层调用，不能在条件语句中定义。这些约束对于熟悉React的开发者来说都不是问题。

之所以有这些约束条件主要是由于Hooks的实现是基于数组的，React 需要根据声明的位置来对多个Hooks一一对应，其执行机制可以引用 Jamie 的[观点](https://mobile.twitter.com/jamiebuilds/status/1055538414538223616)，大意如下：

![](.gitbook/assets/image%20%283%29.png)

为每个组件保存一个Hooks队列，当Hook被使用时，将移动到队列的下一个元素。由于Hooks的规则约束，他们的顺序和每次渲染的顺序得以保持一致。

### 结论

Hooks的出现意味着 React 在 API 一致性方面的有一次重大尝试，必然会是今后 React 开发的潮流。即便从 Hooks 本身来看，对于优化代码结构，提高应用性能方面，也有着极大的优势。此外，Hooks 还第一次为 React 带来了 **“约定”** 的概念，虽然牺牲了一定的自由度，但是无疑也会带来更好的代码风格一致性，降低开发和维护成本。退一步讲，即便不喜欢 Hooks ，此前的开发方式一样可以用，不是么？

