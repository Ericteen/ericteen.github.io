---
layout:     post
title:      "React 进阶"
subtitle:   "高阶组件 (2/2)"
date:       2017-08-16 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-02.jpg"
---
# 高阶组件

## 引入组件

引入一个简单的高阶组件

```JavaScript
import React, { Component } from 'react';

const HOC = (WrappedComponent) => {
  console.log('HOC');
  return class extends Component {
    render() {
      return <WrappedComponent {...this.props}/>
    }
  }
}
export default HOC;
```

```JavaScript
import React, { Component } from 'react';
import HOC from './HOC';

class App extends Component {
  componentDidMount() {
      console.log(this.props, 'props')
    }

  render() {
    return (
      <div>
        App
      </div>
    )
  }
}
export default HOC(App);
```

组件 `App` 通过 `HOC` 的包裹，输出了 `props`。高阶组件 `HOC` 接收组件 `App` 并返回一个新的组件。在高阶组件里我们可以做很多操作，并且返回的新组件也有自己的生命周期。也可以将 `props` 传给 `WrappedComponent`。

## 装饰器模式

高阶组件可以看做是装饰器模式 (Decorator Pattern) 在 React 中的实现。即允许向一个现有的组件添加新的功能，同时又不改变其结构。

ES7中添加了一个decorator的属性，使用@符表示，可以更精简的书写。那以上的例子就可以改成：

```JavaScript
import React, { Component } from 'react';
import HOC from './HOC';

@HOC
export default class App extends Component {
  render() {
    return (
      <div>
        App
      </div>
    )
  }
}
```

## HOC 的两种实现方式

### 属性代理（Props Proxy）

```JavaScript
import React, { Component } from 'react'

const ppHOC = (WrapperComponent) => (
  class extends Component {
    render() {
      return <WrappedComponent {this.props} />
    }
  }
)
```

这里主要是 HOC 在 render 方法中 返回 了一个 WrappedComponent 类型的 React Element。我们还传入了 HOC 接收到的 props，这就是名字 Props Proxy 的由来。

使用 Props Proxy 可以用来：

1. 操作 props
1. 通过 Refs 访问到组件实例
1. 提取 state
1. 用其他元素包裹 `WrappedComponent`

**操作 props**

可对传给 `WrappedComponent` 的 props 进行读取，编辑，添加，删除等操作。

例如，向 `WrappedComponent` 中添加新的属性。

```JavaScript
import React, { Component } from 'react'

const ppHOC = (WrapperComponent) => (
  class extends Component {
    render() {
      const newProps = {
        user: CurrentLoggedInUser
      }
      return <WrappedComponent {...this.props} {...newProps} />
    }
  }
)
```

**通过 Refs 访问到组件实例**

```JavaScript
import React, { Component } from 'react'

const refsHOC = (WrapperComponent) => (
  class extends Component {
    method(wrappedComponentInstance) {
      wrappedComponentInstance.method()
    }

    render() {
      const props = Object.assign({}, this.props, {ref: this.method.bind(this)})
      return <WrappedComponent {...props} />
    }
  }
)
```

Ref 的回调函数会在 WrappedComponent 渲染时执行，你就可以得到 WrappedComponent 的引用。这可以用来读取/添加实例的 props ，调用实例的方法。

**提取 state**

通过传入 props 和回调函数把 state 提取出来。

```JavaScript
import React, { Component } from 'react'

const ppHOC = (WrappedComponent) =>
  class extends Component {
    constructor(props) {
      super(props)
      this.state = {
        name: ''
      }

      this.onNameChange = this.onNameChange.bind(this)
    }
    onNameChange(event) {
      this.setState({
        name: event.target.value
      })
    }
    render() {
      const newProps = {
        name: {
          value: this.state.name,
          onChange: this.onNameChange
        }
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
```

也可以用 ES7 的装饰器

```JavaScript
@ppHOC
class Example extends React.Component {
  render() {
    return <input name="name" {...this.props.name}/>
  }
}
```

这个 input 会自动成为受控input。

**用其他元素包裹 WrappedComponent**

为了封装样式、布局或别的目的，你可以用其它组件和元素包裹 WrappedComponent。
包裹样式：

```JavaScript
const ppHOC = (WrappedComponent) => 
  class extends Component {
    render() {
      return (
        <div>
          <WrappedComponent {...this.props} />
        <div />
      )
    }
  }
```

### 反向继承（Inheritance Inversion）

未完待续...