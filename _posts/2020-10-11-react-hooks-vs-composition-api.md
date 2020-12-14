---
layout:     post
title:      "React hooks 和 Vue Composition API 的比较"
subtitle:   "React hooks vs Vue Composition API"
date:       2020-11-01 18:00:00
author:     "Eric"
header-img: "img/post-bg-04.jpg"
---

React Hooks 的出现为我们提供了一种新的共享和复用逻辑的方式。而 Vue 3 Composition API 也是对这个概念的应用。这篇文章将会对这两种方式做一个对比。看一下 Vue 3 的响应式可变的(mutable reactivity)解决方案与 React 的不可变(immutability)的解决方案有哪些差别。

## 简单的组件状态

首先来看一下两者是如何来对 UI 状态来进行处理的

useState

```js
import React, { useState } from "react";

export function Search() {
  const [searchValue, setSearchValue] = useState("Batman");

  return (
    <input
      value={searchValue}
      onChange={(e) => setSearchValue(e.target.value)}
    />
  );
}
```

使用 `useState` 可以得到一个数值和一个改变这个数值的方法，当对数值进行修改时，需要调用这个方法来进行改变。这也体现了 React 函数式组件的心智模型：**函数式组件是 JavaScript 函数，每当有数据发生变更时，它们都将会执行**。每当状态发生变化，React 会再次调用当前函数，丢弃上一次执行时的所有变量，同时只保留组件的状态。通过这种方式，函数块中的变量可以用 `const` 来声明，因为每一次渲染都会重新进行声明。

这也表示，在 React 中状态值时不可变的(immutable)，它只能通过调用赋值函数来对状态来进行变更。

Vue 3

```html
<script>
import { ref } from "vue";

export default {
  setup(props) {
    const search = ref("");
    return { search };
  },
};
</script>

<template>
  <div>
    <input v-model="search" />
    <input :value="search" @input="search = $event.target.value" />
  </div>
</template>
```

而 vue 的处理方式是声明一个 `ref`，这个 `ref` 函数返回一个对象，这个对象有一个 `value` 属性。vue 使用 reactivity 的方式对这个值进行了包装，使其变成了一个响应式的值。所以，我们可以直接对这个状态来进行赋值，同样，这使得我们可以只执行一次 `setup`，这个初始函数可以对所有的响应式状态值进行初始化，并将其暴露给 template。在 template 中，vue 会利用 `@vue/compiler-sfc` 将模板编译为 render function，并自动对 ref 值来进行解封。

## Side Effects

在 React 中使用 `useEffect`

```js
import {useState, useEffect} from 'react'
const API_KEY = 'xxx';

export default function useMovieApi() {
  const [search, setSearch] = useState('BatMan')
  const [movies, setMovies] = useState([])
  const [isLoading, setIsLoading] = useState(false)
  const fetchMovies = () => {
    const MOVIE_API_URL = `https://www.omdbapi.com/?s=${search}&apikey=${API_KEY}`;
    setIsLoading(true)
    fetch(MOVIE_API_URL)
      .then(response => response.json())
      .then(jsonResponse => {
        setMovies(jsonResponse.Search);
        setIsLoading(false)
      });
  };

  useEffect(fetchMovies, [search])

  return {
    isLoading,
    search,
    setSearch,
    movies
  };
};
```

在 React 中使用 `useEffect` 时，需要手动添加依赖，每当这个依赖的值发生变化时，`useEffect` 中传入的第一个参数才会再次执行。

在 Vue 3 中使用 `watchEffect`

```js
import { reactive, watchEffect } from 'vue'
const API_KEY = 'xxx';

export const useMovieApi = () => {
  const state = reactive({
    search: 'Batman',
    loading: true,
    movies: []
  });

  watchEffect(() => {
    const MOVIE_API_URL = `https://www.omdbapi.com/?s=${state.search}&apikey=${API_KEY}`;

    fetch(MOVIE_API_URL)
      .then(response => response.json())
      .then(jsonResponse => {
        state.movies = jsonResponse.Search;
        state.loading = false;
      });
  });

  return state;
};
```

在 Vue 3 中，得益于响应式系统中内建的依赖收集功能，我们并不需要手动去设置依赖。

## 复用逻辑

如上面的例子，在 React 和 Vue 中，我们都可以将 hooks 抽离出来作为独立的文件来使用。相较于 mixins 和高阶组件等数据复用方式，数据的来源会变得给为清晰，而且也不会出现命名冲突等问题。但在 Vue 3 中，我们需要注意 `ref` 值，最好使用类型系统(TypeScript)来帮助我们来进行类型推断。

## 总结

React Hooks 和 Vue Composition API 的使用方式很好的体现了两者思维上的不同。一个是不可变的隐式进行数据修改，一个是可变的显式的响应式系统。

Vue 可以通过将数据包装成 `ref`，来使得我们可以直接对数据来进行修改。而 React 是通过重新触发 hooks 进而导致值的重新计算，所以每一次数据发生变更，hooks 都会执行一次。

关于两者的更多区别可以查看这个[例子](https://github.com/Ericteen/hooks-compare)。