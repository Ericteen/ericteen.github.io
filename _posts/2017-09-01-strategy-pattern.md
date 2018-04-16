---
layout:     post
title:      "策略模式在表单验证中的应用"
subtitle:   "Strategy Pattern"
date:       2017-09-01 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-02.jpg"
---
# Strategy Pattern

## 问题

在做表单验证时大多都会写出如下的代码

```javascript
if (condition1) {
  // do something
} else if (condition2) {
  // do other things
} else {
  // do
}
```

这样的代码虽然可以实现需求，但同时也存在很多问题

1. dirty code
2. 验证规则不可复用
3. 可维护性较差

## 期望代码能得到的效果

```javascript
const form = document.getElementById('form');
// 创建表单验证实例
const validation = new Validation(policy);
// 编写校验配置
validation.add(form.username, 'isEmpty', '用户名不能为空');
validation.add(form.password, 'minLength: 6', '密码长度不能小于6个字符');
validation.add(form.code, 'isMobile', '请填写正确的手机号');

// 开始校验，并接收错误信息
var errorMsg = validation.start();

// 如果有错误信息输出，说明校验未通过
if(errorMsg){
    // 做一些其他的事
    return false;
}
```

## 策略模式

> 策略模式：
Encapsulates an algorithm inside a class separating the selection from the implementation.

做一件事你会有很多方法，也就是所谓的策略，它的核心思想是，将做什么和谁去做相分离。所以，一个完整的策略模式要有两个类，一个是策略类，一个是环境类(主要类)，环境类接收请求，但不处理请求，它会把请求委托给策略类，让策略类去处理，而策略类的扩展是很容易的，这样，使得我们的代码易于扩展。

在表单验证的例子中，各种验证的方法组成了策略类，比如：判断是否为空的方法(如：isNoEmpty)，判断最小长度的方法(如：minLength)，判断是否为手机号的方法(isMobule)等等，他们组成了策略类，供给环境类去委托请求。

**编写策略类**：策略类是由一组验证方法组成的对象。

```javascript
const VerifyPolicy = {
  isEmpty(val, errMsg) {
    if (val === '') {
      return errMsg;
    }
  },

  minLength(val, length, errMsg) {
    if (val.length < length) {
      return errMsg;
    }
  },

  isMobile(val, errMsg) {
    if (!/^1[3|5|8][0-9]{9}$/.test(val)) {
      return errMsg;
    }
  }
}
```

**编写环境类**: 使用 add 方法添加验证配置。

```javascript
/**
 * @ form.username 表单字段
 * @ isEmpaty 策略对象中策略方法的名字
 * @ 用户名不能为空 验证未通过的错误提示信息
 */
validation.add(form.username, 'isEmpty', '用户名不能为空');

/**
 * @ minLength 用': '分隔策略方法名称和方法的参数
 */
validation.add(form.password, 'minLength: 8', '最小长度为8个字符')
```

implementation

```javascript
class FormValidation {
  constructor(policy) {
    // 保存策略对象
    this.strategies = policy;
    // 验证缓存
    this.validationFns = [];
  }

  add(dom, rule, errMsg) {
    const arr = rule.split(': ');
    const arg = [];
    const self = this;
    this.validationFns.push(function() {
      const ruleName = arr[0];
      arg.push(dom.value);
      if (arr[1]) {
        arg.push(arr[1]);
      }
      arg.push(errMsg);
      return self.strategies[ruleName].apply(dom, arg);
    })
  }

  start() {
    this.validationFns.map(fn => {
      const result = fn();
      if (result) {
        return result;
      }
    })
  }
}
```
