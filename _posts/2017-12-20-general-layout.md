---
layout:     post
title:      "常见布局"
subtitle:   "general layout"
date:       2017-12-20 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-03.jpg"
---
# General Layout

## 骰子布局

## 网格布局

基本网格布局

```css
parent {
  display: flex;
}
child {
  flex: 1;
}
```

百分比布局。某个网格的宽度为固定的百分比，其余网格平均分配剩余的空间。

## 圣杯布局

参考文件 holly-grail.html

## 输入框布局

输入框前后两个 inline element 保持原来大小。输入框占据剩余空间。

```html
<div class="input-addon">
  <span class="input-addon-item">...</span>
  <input class="input-addon-field">
  <button class="input-addon-item">...</button>
</div>
```

```css
.input-addon {
  display: flex;
}

.input-addon-field {
  flex: 1;
}
```

## 悬挂式布局

media-body 占据剩余空间。

```html
<div class="media">
  <img class="media-figure" src="" alt="">
  <p class="media-body">...</p>
</div>
```

```css
.media {
  display: flex;
  align-itema: flex-start;
}
.media-figure {
  margin-right: 1em;
}

.media-body {
  flex: 1;
}
```

## 固定的底栏

有时，页面内容太少，无法占满一屏的高度，底栏就会抬高到页面的中间。

```html
<body class="site">
  <header>...</header>
  <main class="site-content">...</main>
  <footer>...</footer>
</body>
```

```css
.site {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.site-content {
  flex: 1;
}
```

header 和 footer 保持各自的高度。main 占据剩余空间。

## 流式布局

每行的项目数固定，会自动分行。

```css
.parent {
  width: 200px;
  height: 150px;
  background-color: black;
  display: flex;
  flex-flow: row wrap;
  align-content: flex-start;
}

.child {
  box-sizing: border-box;
  background-color: white;
  flex: 0 0 25%;
  height: 50px;
  border: 1px solid red;
}
```

---

## 文字垂直居中

### 单行文字

容器高度固定 height + line-height

```html
<div class="vertical-align">这是一行在固定高度容器中，垂直居中的文字</div>
```

```css
.verticl-align {
  height: 100px;
  line-height: 100px;
}
```

容器高度不固定 table + table-cell + vertical-align: middle。也适用于多行文字。

```html
<div>
  <p>p的高度随div高度变化而变化，单行文字垂直居中</p>
</div>
```

```css
div {
  display: table;
}

p {
  display: table-cell;
  vertical-align: middle;
}
```

### 多行文字

其实，vertical-align 的对齐，在非 display:table 下，是需要有参照物的，所以我们想到在容器里边添加一个高度等于容器高度、宽度为 0 的参照物，哎，对了，:before 要登场了。并将外层容器的 font-size 设置为 0。

```html
<div>
  <p>
    多行文字垂直居中
    多行文字垂直居中
    多行文字垂直居中
  </p>
</div>
```

```css
div:before {
    content: '';
    display: inline-block;
    vertical-align: middle;
    width: 0;
    height: 100%;
}
div {
    width: 400px;
    height: 140px;
    background: #369;
    font-size: 0;
}
p {
    display: inline-block;
    font-size: 16px;
    vertical-align: middle;
    text-align: left;
}
```

> vertical-align:
>  1. To vertically align an **inline element's** box inside its containing line box. For example, it could be used to vertically position an `<img>` in a line of text.
>  2. To vertically align the content of a **cell** in a **table**.

垂直居中

1.不知道自己高度和父容器高度的情况下, 利用绝对定位只需要以下三行：

```css
parentElement{
        position:relative;
    }

childElement{
      position: absolute;
      top: 50%;
      transform: translateY(-50%);

}
```

2.若父容器下只有一个元素，且父元素设置了高度，则只需要使用相对定位即可:

```css
parentElement{
        height:xxx;
    }

childElement {
  position: relative;
  top: 50%;
  transform: translateY(-50%);
}
```

Flex 布局(推荐)

```css
.container {
  display: flex;
  align-items: center;
  justify-content: center;
}
```
