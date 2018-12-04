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
