---
layout:     post
title:      "函数式方程简介"
subtitle:   "Intro to Functional Programming"
date:       2018-01-15 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-04.jpg"
---
# Principles

函数式编程的特点

- 可预测性 (Predictable)
- 安全 (safe)
- 透明 (Transparent)
- 模块化 (modular)

## Predictable

- pure function 同样的输入一定会有同样的输出
- 易于测试

```javascript
const add = (x, y) => x + y;
add(3, 2); // 5
add(1, 2); // 3
```

```javascript
let name = 'Jack';
const getName = () => name; // 依赖全局变量
const setName = (newName) => { // 修改了全局变量
  name = newName;
}
const PrintUpperName = () => {
  console.log(name.toUpperCase()); // 修改了全局变量
}
```

## Imperative & Declarative

imperative

```javascript
function doubleNums(nums) {
  const doubled = [];
  const l = nums.length;

  for (let i = 0; i < l; i++) {
    doubled.push(nums[i] * 2);
  }

  return doubled;
}
```

declarative

```javascript
function doubleNums(nums) {
  return nums.map(num => num * 2);
}
```

## Immutable

不修改原始数据，生成新的数据

```javascript
const hobbies = [
  'programming',
  'reading',
  'music',
];

const firstTwo = hobbies.splice(0, 2);
console.log(firstTwo); // ['programming', 'reading']
console.log(hobbies); // ['music']
```

Object.freeze & immutable.js

```javascript
const hobbies = Object.freeze([
  'programming',
  'reading',
  'music',
])

const firstTwo = bobbies.splice(0, 2); // TypeError
```

```javascript
// 不推荐使用的方式
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  moveBy(dx, dy) {
    this.x += dx;
    this.y += dy;
  }
}

const point = new Potin(0, 0);
point.moveBy(5, 5);
point.moveBy(-2, 2);
console.log([point.x, point.y]); // [3, 7]

// 推荐
const createPoint = (x, y) => Object.freeze([x, y]);
const movePointBy = ([x, y], dx, dy) => {
  return Object.freeze([x + dx, y + dy]);
}

let point = createPoint(0, 0);
point = movePointBy(point, 5, 5);
point = movePointBy(point, -2, 2);
console.log(point); // [3, 7]
```

优点

- 安全性
- Free Undo/Redo logs - Redux
- 清晰的数据流
- 内存使用率低
- 并发安全

缺点(可以通过 Immutable.js 库减轻影响)

- 冗余代码
- 创建更多的对象
- 更多的垃圾回收(Garbage Collection)
- 更多的内存使用

## First Class Function JS 的最大优点

```javascript
const multiply = (x, y) => x * y;

function add (x, y) {
  return x + y;
}
const addAlias = add;

const evens = [1, 2, 3].map(x => x * 2);
```

偏函数

```javascript
const partilalFromBind = (fn, ...args) => {
  return fn.call(null, ...args);
};

const partial = (fn, ...args) => {
  return (...otherArgs) => {
    fn(...args, ...otherArgs);
  };
};
```

柯里化

```javascript
const add = x => y => x + y;

function add(x) {
  return function (y) {
    return x + y;
  }
}
```

## 合并以上知识点

```javascript
const map = fn => array => array.map(fn);
const multiply = x => y => x * y;
const pluck = key => object => object[key];

const discount = multiply(0.88);
const tax = multipy(1.0925);

const customRequest = request({
  headers: {'X-custom': 'myKey'}
});

customRequest({
  url: '/cart/items'
})
  .then(map(pluck('price')))
  .then(map(discount))
  .then(map(tax));

// OR 执行顺序从左到右
customRequest({
  url: '/cart/items'
})
  .then(map(
    compose(
      tax,
      discount,
      pluck('price'),
  )))
```

compose 函数

```javascript
const processWord = compose(hypenate, reverse, toUpperCase);
const words = ['hello', 'functional', 'programming'];
const newWords = words.map(processWord);
console.log(newWords); // ["OLL-EH", "LANOI-TCNUF", "GNI-MMARGORP"]
```

compose implementaion

```javascript
const compose = (...fns) => {
  if (fns.length === 0) {
    return arg => arg;
  }

  if (fns.length === 1) {
    return fns[0];
  }

  return fns.reduce((a, b) => (...args) => a(b(...args)));
}
```

用迭代的方法解决 需要用 for 循环来解决的问题

```javascript
// imperial way
const factorial = (n) => {
  let result = 1;

  while(n > 1) {
    result *= n;
    n--;
  }

  return result;
}

// declarative way
// 始终保持对上一个函数的引用，会使 call stack 不断积累
const factorial = (n) => {
  if (n < 2) return 1;
  return n * factorial(n - 1);
}

// 尾调用优化
// 返回值为函数，调用完之后出栈，call stack 中始终只有当前函数
const factorial = (n, accum = 1) => {
  if (n < 2) return accum;
  return factorial(n - 1, accum * n);
}
```

> [Mostly Adequated Guide](https://drboolean.gitbooks.io/mostly-adequate-guide-old)
>
> [Functional Programming in ES6](https://www.youtube.com/watch?v=FYXpOjwYzcs)
>
> [Learning Functional Programming with JavaScript](https://www.youtube.com/watch?v=e-5obm1G_FY)
