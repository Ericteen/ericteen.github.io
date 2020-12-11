---
layout:     post
title:      "TypeScript 简单工具类型"
subtitle:   "TS utils simple"
date:       2019-02-01 18:00:00
author:     "Eric"
header-img: "img/post-bg-03.jpg"
---

## Pick

封装方式

```ts
type MyPick<T, K extends keyof T> = {
    [P in K]: T[P]
}
```

使用方式

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

## readonly

封装方式

```ts
type MyReadonly<T> = {
    readonly [K in keyof T]: T[K]
}
```

使用方式

```ts
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};
```

## TupleToObject

```ts
type TupleToObject<T extends readonly string[]> = {
    [P in T[number]]: P
};
```

## First

获取数组的第一个元素

```ts
type First<T extends any[]> = T['length'] extends 0 ? never : T[0]
```

## Length

获取 tuple 的长度

```ts
type Length<T extends any[]> = T['length'];
```

## Exclude

```ts
type MyExclude<T, U> = T extends U ? never : T;
```

使用

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;
//    ^ = type T0 = "b" | "c"
```

## Extract

```ts
type MyExtract<T, U> = T extends U ? T : never;
```

使用 

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
//    ^ = type T0 = "a"
```

## Awaited

```ts
type Awaited<T> = T extends Promise<infer R> ? R : never;
```

## If

```ts
type If<C extends boolean, T, F> = C extends true ? T : F;
```