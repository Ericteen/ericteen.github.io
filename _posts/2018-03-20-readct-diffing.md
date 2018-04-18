---
layout:     post
title:      "React 比较算法简述"
subtitle:   "React diffing algorithm"
date:       2018-03-20 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-05.jpg"
---
# React 比较算法

## Prerequisite

```javascript
const element = <div>Hello React</div>
```

它被称为 JSX， 一种 JavaScript 的语法扩展。 我们推荐在 React 中使用 JSX 来描述用户界面。JSX 乍看起来可能比较像是模版语言，但事实上它完全是在 JavaScript 内部实现的。

在编译之后，React 会将一个 JSX 元素转换成一个包含 type, props, children 三个元素的 JavaScript 对象。例如：

```html
<div class="root" style="color: green">
  <p class="text">hello react</p>
</div>
```

会变为

```javascript
"use strict";

React.createElement(
  "div",
  { "class": "root", style: "color: green" },
  React.createElement(
    "p",
    { "class": "text" },
    "hello react"
  )
);
```

React 比较算法基于两个假设

1. 两个不同类型的元素将产生不同的树
2. 通过渲染器附带 `key` 属性，开发者可以示意哪些元素可能是稳定的

## 比较算法

对比两颗组件树时，React 首先比较两个根节点。根节点 type 不同，其行为也不同。

## 元素类型不同的比较

每当根元素有不同类型，React将卸载旧树并重新构建新树。

当树被卸载，旧的 DOM 节点将会被销毁。组件实例会调用 `componentDidMount()`。当构建一个新树时，新的 DOM 节点将被插入到 DOM 中。组件实例将依次调用 `componentWillMount()` 和 `componentDidMount()`。任何与旧树相关的状态都将被丢掉。

```html
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

这将会销毁旧的 `<Counter />` 并重装新的 `<Counter />`。

## 相同类型的 DOM 元素

当比较两个相同类型的React DOM元素时，React则会观察二者的属性，保持相同的底层DOM节点，并仅更新变化的属性。例如：

```html
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较两个元素，React知道仅更改底层DOM元素的className。在处理完DOM元素后，React递归其子元素。

## 相同类型的组件元素

当组件更新时，实例仍保持一致，以让状态能够在渲染之间保留。React 通过更新底层组件实例的props来产生新元素，并在底层实例上依次调用 `componentWillReceiveProps()`  和 `componentWillUpdate()` 方法。

接下来，render()方法被调用，同时对比算法会递归处理之前的结果和新的结果。

## 递归子节点

默认时。当递归 DOM 节点的子节点，React仅在同一时间点递归两个子节点列表，并在有不同时产生一个变更。

React 支持了一个 `key` 属性。当子节点有 `key` 时，React 使用 `key` 来匹配原本树的子节点和新树的子节点。例如，增加一个 `key` 在之前效率不高的样例中能让树的转换变得高效：

```html
<ul>
  <li key="2016">apple</li>
  <li key="2017">pear</li>
</ul>

<ul>
  <li key="2016">apple</li>
  <li key="2017">pear</li>
  <li key="2018">strawberry</li>
</ul>
```

现在React知道带有'2018'的key的元素是新的，并仅移动带有'2016'和'2017'的 `key` 的元素。

## 权衡

牢记调和算法(reconciliation algorithm)的实现细节是非常重要的。React 可能会对每个动作都重新渲染整个应用，渲染结果可能是相同的。搞清楚，在上下文中重新渲染意味着对每一个组件都调用 `render`，这并不意味着 React 会卸载和重装他们。它会根据上述的规则来应用。

在目前实现中，可以表明一个事实，即子树在其兄弟节点中移动，但你无法告知其移动到哪。该算法会重渲整个子树。

由于React依赖于该启发式算法，若其背后的假设没得到满足，则其性能将会受到影响：

1. 算法无法尝试匹配不同组件类型的子元素。若你发现两个输出非常相似的组件类型交替出现，你可能希望使其成为相同类型。实践中，我们并非发现这是一个问题。
2. Keys应该是稳定的，可预测的，且唯一的。不稳定的key（类似由Math.random()生成的）将使得大量组件实例和DOM节点进行不必要的重建，使得性能下降并丢失子组件的状态。
