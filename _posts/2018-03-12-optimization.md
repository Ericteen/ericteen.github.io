---
layout:     post
title:      "webpack 资源加载优化"
subtitle:   "optimization"
date:       2018-03-12 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-02.jpg"
---
# 资源加载优化

## Tree Shaking

[tree shaking](https://webpack.js.org/guides/tree-shaking/)是在 JavaScript 用来描述消除 dead-code 的一个术语。它依赖于 ES2015 的静态结构的模块语法，例如 import 和 export。

自 webpack 2 时就内置了对 ES2015 模块的支持也包括对未使用的导出模块 (export module) 的检测。webpack 4进一步扩展了实用性，在 package.json 文件中添加 `sideEffects` 字段来告知 compiler 项目中的哪一个文件是纯的，可以被安全移除。

> `side effects` 是指当引入时代码表现出的特殊行为，而非多次暴露出 export。对应的一个例子是 polyfills，影响全局作用域并且通常不提供一个 export。[详情](https://github.com/webpack/webpack/tree/master/examples/side-effects)

在100%的 ESM 模块下，辨识出 side effects 是很直接的。可惜的是，现阶段的开发环境中并非如此，所以我们需要给 webpack compiler 提供字段来表示代码的纯净与否。这就是 package.json 的 `side effects` 字段。

package.json

```javascript
{
  "name": "project-name",
  "sideEffects": false
}
```

同时，我们也可以在 `module.rule` 里设置 `side effects`。

webpack.config.js

```javascript
{
  module: {
    rules: [
      {
        include: path.resolve('node_modules', 'lodash'),
        sideEffects: false
      }
    ]
  }
}
```

我们已经通过 ESM 的 `import` 和 `export` 语法剥离了 dead code，但我们依然需要将其从打包文件中删除。

在 webpack 4 中，可以通过设置 `mode` 为 production，webpack 就会自动启用 `UglifyJSPlugin` 来对代码进行压缩。

综合，要启用 tree shaking 我们需要：

- 使用 ES2015 模块语法 (import 和 export)
- 在 package.json 文件中添加 sideEffects 字段
- 包含一个支持消除 dead code 的压缩器 (如：UglifyJSPlugin)

## Bundle 分离(Bundle Spliting)

应用被打包成一个单一的 JavaScript 文件。如果应用发生改变，客户端也必须下载 vendor 依赖。如果只下载变化的部分，将会大大减少加载量。这就是 Bundle 分离想要达到的目的，它可以用通过设置 `optimization.splitChunks.cacheGroups` 属性来实现。这样可以充分利用客户端的缓存。

在 webpack 4 之前， Bundle 分离是通过 `CommonsChunkPlugin` 来实现的。到了 webpack 4 我们可以通过配置 `optimization` 来实现。

```javascript
// wepack.config.js
modules.exports = {
  // ...
  optimization: {
    splitChunks: {
      chunks: 'initial'
    }
  }
  // ...
}
```

或者使用一种更为显性的描述

```javascript
// wepack.config.js
modules.exports = {
  // ...
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          chunks: 'initial'
        }
      }
      chunks: 'initial'
    }
  }
  // ...
}
```

如果不想使用自动化的配置，可以用上述的配置格式，这样可以对整个控制流程有更多的控制权。

### Splitting and Merging Chunks

`AggressiveSplittingPlugin` 和 `AggressiveMergingPlugin`。这两个插件作用于两个截然相反的方面。前一个可以产生更多的小块 bundles。但同时也会增加客户端请求数量。后一个作用相反，产生更少的 bundle。

### webpack 中的 chunk 种类

webpack 将 chunk 划分为三类：

- **入口 chunk**。入口 chunk 包含 webpack runtime 和将要加载的模块。
- **普通 chunk**。普通 chunk 不包含 webpack runtime，这些 chunk 可以在应用运行时动态加载。
- **初始 chunk**。初始 chunk 也是一个普通 chunk，用来计算应用的加载时间。作为普通用户，需要考虑的是前两个。

## 代码分离(Code Spliting)

代码分离是 webpack 很强有力的一个特性。这个特性可以将代码分离到不同的包，因此这些文件可以按需加载或者并行加载。它可以减少打包文件的体积并且控制资源加载的优先顺序。运用得当的话，对加载时间可以有很大的影响。

通常有三种方法实现代码分离：

- **入口 (Entry Points)**：用配置文件中的 entry 字段手动分离代码
- **防止重复**
- **动态引入**：通过模块内的行内函数调用来分离代码

第一点比较简单，就不再赘述。

对于第二点，在 webpack 3 中可以通过使用 CommonsChunkPlugin 来分离重复的块。在 webpack 4 中，这个插件已经弃用，需要设置 config.optimizarion.splitChunks 来实现。

```javascript
{
  // ...
  optimization: {
    splitChunks: { chunks: 'all }
  }
}
```

在社区中还有一些其他有用的 loader 和插件来实现代码分离。如：`ExtractTextPlugin` 可以对 CSS 代码进行分离。还有 `bundle-loader` 和 `promise-loader`。

### 动态引入

`import()` 语法符合 ECMAScript 对动态引入的提案。需要搭配`babel-plugin-syntax-dynamic-import`。

.babelrc

```javascript
{
  plugins: ["syntax-dynamic-import"]
}
```

webpack.config.js

```javascript
{
  entry: './src/index.js',
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
    chunkFileName: '[name].bundle.js'
  }
}
```

chunkFileName 决定了非入口 chunk 的名称。这些文件名需要在 runtime 根据 chunk 发送的请求生成。

由于 `import()` 会返回一个 promise，因此可以和 `async 函数`一起使用。

```javascript
async function getComponent() {
  const element = document.createElement('div')
  const _ = await import(/* webpackChunkName: 'lodash' */ 'lodash')
  element.innerHTML = _.join(['Hello', 'code', 'spliting'], ' ')
  return element
}

getComponent().then(component => {
  document.body.appendChild(component)
})
```

在注释中使用了 `webpackChunkName`。这样做会导致我们的 bundle 被命名为 lodash.bundle.js ，而不是 [id].bundle.js。

### 懒加载(Lazy Loading)

懒加载或者叫按需加载，对应用或网站的优化有极大的作用。其中包括按逻辑对代码进行分割，请求的时候再加载模块。这可以极大提升应用的首次加载时间，并且降低加载量。

```javascript
import _ from 'lodash'

function component() {
  const element = document.createElement('div')
  const btn = document.createElement('button')
  const br = document.createElement('br')

  btn.innerHTML = 'Click me and see the console.'
  element.appendChild(br)
  element.appendChild(btn)

  btn.onclick = e => import(/* webpackChunkName: "print" */ './print').then(module => {
    const print = module.default
    print()
  })
  return element
}

document.body.appendChild(component())
```

如以上的例子，主要是运用 ESM 的 `import()` ，对按钮操作进行按需加载。

## 提取 manifest 文件

当 webpack 生成 bundle 时， 它同时维护一个 manifest 文件。你可以在生成的 vendor bundle 中找到它。manifest 文件描述了哪些文件需要 webpack 加载。

如果 webpack 生成的 hash 发生改变，manifest 文件也会发生改变。因此，vendor bundle 的内容也会发生改变，并且失效。所以，我们需要将 manifest 文件提取出来。

大部分工作都已经在 bundle splitting 中完成。为了提取 manifest 文件，需要用以下的方式定义 `optimization.runtimeChunk`

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      // ...
    },
    runtimeChunk: {
      name: 'manifest'
    }
  }
}
```

## 缓存(Caching)

当 `/dist` 文件里的内容被上传到服务器上，客户端发起请求获取资源，这是个非常耗时的操作。对此，浏览器通常会采用缓存的技术。这使得网站在加载的时候可以减少很多不必要的网络流量，同时这也对新资源的获取产生了挑战。

一种简单的解决方案是将配置文件中的 `output.filename` 字段加上 `[hash]`(每一次打包都会生成一个唯一的 hash ) 或者是 `[chunkhash]`(根据每个 chunk 的内容来生成)。推荐使用 `[chunkhash]`。

webpack 的每一个 chunk 里面都包括很多样板文件(boilerplate)，特别是运行时和 manifest 文件。这会导致每次打包后的 `output.filename` 都会发生变化。所以我们需要将样板文件(boilerplate)提取出来分开打包。

在 webpack 3 中，可以用 `CommonsChunkPlugin` 来取出 manifest 文件和样板文件。

```javascript
{
  entry: {
    main: './src/index.js',
    vendor: ['lodash']
  },
  //...
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor' // 取出 vendor 文件
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest' // 取出 manifest 文件，另外打包
    })
  ]
}
```

而在 webpack 4 中，CommonsChunkPlugin 已经被弃用。4.0 的文档不全，暂时请参考[webpack 4: mode and optimization](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)

```javascript
{
  optimization: {
    splitChunks: {
      chunks: 'initial'
    },
    runtimeChunk: {
      name: 'manifest'
    }
  },
}
```

另外，在打包过程中 vendor 文件的 `module.id` 每一次都会增加，所以每次输出的文件名也会有差异。为了修复这个，我们需要用到 `NamedModulesPlugin`，它会使用文件相对模块的路径来命名而非数字标识符(numerical identifier)。适用于开发环境。在生产环境中，适合用 [`HashedModuleIdsPlugin`](https://webpack.js.org/plugins/hashed-module-ids-plugin/)。

```javascript
{
  plugins: [
    new webpack.HashedModuleIdsPlugin()
  ]
}
```

修改文件再次打包后，会发现分理出的 vender 文件保持不变。
