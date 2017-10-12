---
layout:     post
title:      "React 进阶"
subtitle:   "高阶函数"
date:       2017-08-01 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-03.jpg"
---
# 高阶组件

高阶组件是一个概念上很简单，却十分实用的东西，被大量的 React 相关的第三方库频繁使用。在前端业务开发当中，灵活使用高阶组件可以让你的代码更为优雅，灵活性和复用性也会更强。同时，了解高阶组件对我们理解各种 React 第三方库也有很大的帮助。

## [什么是高阶组件](https://reactjs.org/docs/higher-order-components.html)

>A higher-order component is a function that takes a component and returns a new component.
>高阶组件是一个函数，接收一个组件并返回一个新的组件。

```JavaScript
const EnhancedComponent = highOrderComponent(WrappedComponent)
```

高阶组件是一个函数而非组件，就像一个组件将 `props` 转换为 `UI`，高阶组件接收一个组件作为参数，返回一个新的组件。

进一步学习之前，我们先回顾一下 `new` 和构造函数之间的关系。

```JavaScript
var Person = function(name, age) {
  this.name = name;
  this.age = age;
  this.getName = function(){
    return this.name;
  }
}

function New(fn) {
  var res = {};
  if (fn.prototype !== null) {
    res.__proto__ = fn.prototype;
  }

  var ret = fn.apply(res, Array.prototype.slice.call(arguments, 1));

  if ((typeof ret === 'object || typeof ret === 'function) && ret !== null) {
    return ret;
  }

  return res;
}

var p1 = New(Person, 'tom', 20);
console.log(p1.getName());

console.log(p1 instanceof Person); // true
```

在以上的例子中，我定义了一个本质上与普通函数没区别的构造函数，然后将该构造函数以参数形式传入 New 函数中。我在 New 函数中进行了一些的逻辑处理，让 New 函数的返回值为一个实例，正因为 New 的内部逻辑，让构造函数中的 `this` 能够指向返回的实例。

## 高阶组件的实例

同理，我来写一个简单的高阶组件。

```JavaScript
import React, { Component } from 'react'

export default (WrappedComponent) => {
  class NewComponent extends Component {
    // 可以做很多自定义逻辑
    render () {
      return <WrappedComponent />
    }
  }
  return NewComponent
}
```

上例简单构建了一个新的组件类 `NewComponent`，然后把传入的 `WrappedComponent` 组件渲染出来。接下来我给 `NewComponent` 添加一些功能。

```JavaScript
import React, { Component } from 'react'

export default (WrappedComponent, name) => {
  class NewComponent extends Component {
    constructor () {
      super()
      this.state = { data: null }
    }

    componentWillMount () {
      let data = localStorage.getItem(name)
      this.setState({ data })
    }

    render () {
      return <WrappedComponent data={this.state.data} />
    }
  }
  return NewComponent
}
```

增加功能之后，`NewComponent` 会根据第二个参数 `name` 在挂载阶段从 LocaleStorage 加载数据，并且添加到自己的 `state.data` 中，而渲染的时候将数据通过 `props` 传送给 `WrappedComponent` 中。

我们来复用这段代码，假设上述代码在 `src/wrapLoadData.js` 文件中。

```JavaScript
import React, {Component} from 'react'
import wrapLoadData from './wrapLoadData'

class InputWithUserName extends Component {
  render() {
    return <input value={this.props.data} />
  }
}

InputWithUserName = wrapLoadData(InputWithUserName, 'username')

export default InputWithUserName
```

假如 `InputWithUserName` 的功能需求是挂载的时候从 LocalStorage 里面加载 `username` 字段作为 `<input />` 的 `value` 值，现在有了 `wrapLoadData`，我们可以很容易地做到这件事情。

根据 `wrapLoadData` 的代码我们可以知道，这个新的组件挂载的时候会先去 LocalStorage 加载数据，渲染的时候再通过 `props.data` 传给真正的 `InputWithUserName`。

如果现在我们需要另外一个文本输入框组件，它也需要 LocalStorage 加载 'content' 字段的数据。我们只需要定义一个新的 `TextareaWithContent`：

```JavaScript
import wrapLoadData from './wrapLoadData'

class TextareaWithContent extends Component {
  render () {
    return <textarea value={this.props.data} />
  }
}

TextareaWithContent = wrapWoadData(TextareaWithContent, 'content')
export default TextareaWithContent
```

这样做起来会非常轻松，我们根本不需要重复写从 LocalStorage 加载数据字段的逻辑，直接用 `wrapWithLoadData` 包装一下就可以了。

## 总结

上述两个组件 `InputWithValue` 和 `TextareaWithContent` 的需求都有一个相同的逻辑，即**挂载阶段从 LocalStorage 中加载特定的字段数据**。

高阶组件的作用，其实就是为了**组件之间的代码复用**。组件可能有着某些相同的逻辑，把这些逻辑抽离出来，放到高阶组件中进行复用。**高阶组件内部的包装组件和被包装组件之间通过 `props` 传递数据**。