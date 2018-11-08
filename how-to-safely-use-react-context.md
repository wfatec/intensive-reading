# How to safely use React context

这篇文章是 [Mobx](https://cn.mobx.js.org/) 的作者**Michel Weststrate**于2016年发表的一篇文章，原文出自 [这里](https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076)，主要探讨了context在React中的角色，以及如何安全的使用它。

 [Dan Abramov](https://medium.com/@dan_abramov) 对于context的使用规则进行了非常明智的概括，具体如下：

![](.gitbook/assets/image.png)

但有时候，即便遵循了以上规则，却还是可能遇到麻烦，例如组合使用一些用到context的第三方类库，比如 [react-router](https://github.com/ReactTraining/react-router) ，  [react-redux](https://github.com/reactjs/react-redux) ，或是  [mobx-react](https://github.com/mobxjs/mobx-react) ，甚至自定义的  _shouldComponentUpdate_  或者由  _React.PureComponent_ 提供的组件。

### 为什么Context + ShouldComponentUpdate易出现问题？

context的作用我们可以自行查阅 [官方文档](https://reactjs.org/docs/context.html#passing-info-automatically-through-a-tree) ，这里不再赘述。

 _shouldComponentUpdate_ \(SCU\) 可以缩短渲染的过程，例如当 _props_ 或者 _state_   没有发生有效变化时。但是这里不仅会阻断渲染，还会导致阻塞context的传播。

作者举了个相关的例子：

```javascript
const TODOS = ["Get coffee", "Eat cookies"]

class TodoList extends React.PureComponent {
  render() {
    return (<ul>
      {this.props.todos.map(todo => 
        <li key={todo}><ThemedText>{todo}</ThemedText></li>
      )}
    </ul>)
  }
}

class App extends React.Component {
  constructor(p, c) {
    super(p, c)
    this.state = { color: "blue" } 
  }

  render() {
    return <ThemeProvider color={this.state.color}>
      <button onClick={this.makeRed.bind(this)}>
      	<ThemedText>Red please!</ThemedText>
      </button>
      <TodoList todos={TODOS} />
    </ThemeProvider>
  }
  
  makeRed() {
    this.setState({ color: "red" })
  }
}

class ThemeProvider extends React.Component {
  getChildContext() {
    return {color: this.props.color}
  }

  render() {
    return <div>{this.props.children}</div>
  }
}
ThemeProvider.childContextTypes = {
  color: React.PropTypes.string
}

class ThemedText extends React.Component {
  render() {
    return <div style={{color: this.context.color}}>
      {this.props.children}
    </div>
  }
}
ThemedText.contextTypes = {
  color: React.PropTypes.string
}

ReactDOM.render(
  <App />,
  document.getElementById("container")
)
```

