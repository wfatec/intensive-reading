# Comparison of state management solutions for React

本篇文章来自 [这里](https://medium.com/dailyjs/comparison-of-state-management-solutions-for-react-2161a0b4af7b) ，这是一篇关于现有 react 状态管理主流方案的对比分析文章，其中对于state复杂度的把控方面的论述尤为值得关注。可以结合上一篇文章《You are managing state? Think twice》来进行深入思考。

## State存在的意义

可以便于我们对页面逻辑进行抽象，多数情况下，当我们建立起完备的state结构时，接下来的业务结构也就水到渠成了。事实上，state的兴起也正是数据驱动业务思想下的产物，由此诞生了angular，react，vue等一系列优秀框架。正是由于state概念的重要性，涌现出了层出不穷的状态管理方案。


## 关于演示

这里提供了用于对比使用不同lib库进行state管理的实际效果，地址为：[statemanagement-comparison](https://codesandbox.io/s/135wv8zy33)。

## 组件本身的 state

也就是react为我们的组件类提供的state属性，可以通过 `setState` 进行更改。

使用场景：多用于组件本身的状态管理，高度聚合，且不宜向下进行多层传递。

## Context API

这是react 16.3.0版本提供的新的API，可以用于避免props深层注入的风险。

使用场景：需要将state传递到深层子组件时，且state复杂度不高的场景。

使用方法像这样：

**step1：** 创建Context对象

```js
const MyContext = React.createContext(defaultValue);
```
**step2：** 创建Provider

```js
<MyContext.Provider value={/* some value */}>
```

**step3：** 创建Consumer

```js
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

当然，除了用这种函数式方法获取最近的context值之外，也能够通过将此前创建的 MyContext 赋值给静态属性contextType，从而让该组件类内部可以使用`this.context`来获取最近的Context类型的值。具体写法如下：

```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```

## Unstate

基于Context API的状态管理类库，它将状态以及状态的变更方法都集中在同一个容器内，此外，容器内部的`setState`函数与React的`setState`函数几乎相同，区别仅仅是Unstate中的返回值是一个Promise对象。这也就意味着我们能够使用`await`等方法进行流程控制。

```js
import { Container } from 'unstated';
type CounterState = {
  count: number
};
class CounterContainer extends Container<CounterState> {
  state = {
    count: 0
  };

  increment() {
    await this.setState({ count: this.state.count + 1 });
    console.log("count",this.state.count) // this works with Unstated
  }

  decrement() {
    this.setState({ count: this.state.count - 1 });
  }
}
```

**注意**：这里使用了flow来进行类型校验。

对于这个容器的使用有些类似于`consumer`，这里使用的是`subscriber`：

```js
function Counter() {
  return (
    <Subscribe to={[CounterContainer]}>
      {counter => (
        <div>
          <button onClick={() => counter.decrement()}>-</button>
          <span>{counter.state.count}</span>
          <button onClick={() => counter.increment()}>+</button>
        </div>
      )}
    </Subscribe>
  );
}
```

通过 Unstated 我们可以轻松地实现UI逻辑和状态逻辑的分离。

**注意**：但是 React Hooks 的出现大大降低了该类库的作用，我们已经能够通过React Hooks原生实现状态逻辑的分离，并且更加的优雅。

## Redux

大名鼎鼎的React已无需赘言，对于大型项目的构建，它依然是我们的不二选择，同时其源码中大量出现的函数式编程思想也值得我们去反复斟酌。

## Mobx

另一个著名的状态管理库，用法非常的简洁，看起像这样：

```js
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import { observable } from "mobx"
import {observer} from 'mobx-react';


class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}

@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)

const store = new TodoList();
ReactDOM.render(<TodoListView todoList={store} />, document.getElementById('mount'));
```

Mobx在绝大多数场景都已经能够代替Redux进行开发，切编码速度会提高不少，但是维护性上相较Redux还是略有不足，尤其是社区生态上的差距还是存在的。

## 总结

我们在考虑状态管理的方案时，需要着重考虑方案的复杂度，根据项目状态的特点来考虑最适合的方案。适合的才是最好的！
