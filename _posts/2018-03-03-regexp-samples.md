# 常见正则表达式

JavaScript 中用到正则表达式的方法包括

- `String`: search, match, split, replace
- `RegExp`: test, exec

## 常用的匹配方法

### 去除首尾的'/'

```javascript
// 例：//ad/df/// -> ad/df
input.replace(/^\/*|\/*$/g, '')
```

### 匹配一些字符

```javascript
const str = 'asdf html-webpack-plugin for "index/index.html" asdfasdf'
str.match(/html-webpack-plugin for\"(.*)\"/ig)
console.log(RegExp.$1) // index/index.html
```

```javascript
const str = "access_token=dcb90862-29fb-4b03-93ff-5f0a8f546250; refresh_token=702f4815-a0ff-456c-82ce-24e4d7d619e6; account_uid=1361177947320160506170322436"
str.match(/account_uid=([^\=]+(\;)|(.*))/ig) // ["account_uid=1361177947320160506170322436"]
```

### 关键字符串的替换

```javascript
'css/[hash:8].index-index.css'.replace(/\[(?:(\w+):)?(contenthash|hash)(?::([a-z]+\d*))?(?::(\d+))?\]/ig,'(.*)') // css/(.*).index-index.css
```

### 匹配括号中的内容

```javascript
'max_length(12)'.match(/^(.+?)\((.+)\)$/)
// ["max_length(12)", "max_length", "12", index: 0, input: "max_length(12)"]
```

### 调换

```javascript
const name = "Doe, John"
name.replace(/(\w+)\s*, \s*(\w+)/, "$2 $1")
// "John Doe"
```

### 字符串截取

```javascript
const str = 'asfdf===sdfaf ##'
str.match(/[^===]+(?=[===])/g) // 截取 ===之前的内容

str.replace(/\n/g,'')  // 替换字符串中的 \n 换行字符
```

## 验证

### 小数点后几位的验证

```javascript
// 精确到小数点后两位
/^\d+\.\d{2}$/.test(1.234) // false
```

### 密码强度

```javascript
// 必须是包含大小写字母和数字的组合，包含特殊字符，长度在8-10之间。
/^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])(?=.{8,10})/.test('123a@Zhe99') // true

// 中等强度
/^(((?=.*[a-z])(?=.*[A-Z]))|((?=.*[a-z])(?=.*[0-9]))|((?=.*[A-Z])(?=.*[0-9])))(?=.{6,}/.test('asdfgQWERd') // true
```

### 中文检测

```javascript
/^[\u4e00-\u9fa5]{0,}$/.test("但是d") //false
/^[\u4e00-\u9fa5]{0,}$/.test("但是") //true
```

### 身份证号正则

```javascript
/^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/.test("32112319980929371X") // true
```

### 日期检验

```javascript
const re = /^(?:(?!0000)[0-9]{4}-(?:(?:0[1-9]|1[0-2])-(?:0[1-9]|1[0-9]|2[0-8])|(?:0[13-9]|1[0-2])-(?:29|30)|(?:0[13578]|1[02])-31)|(?:[0-9]{2}(?:0[48]|[2468][048]|[13579][26])|(?:0[48]|[2468][048]|[13579][26])00)-02-29)$/;
//输出 true
console.log(re.test("2017-02-11"))
//输出 false
console.log(re.test("2017-15-11"))
//输出 false
console.log(re.test("2017-02-29"))
// true
```

### 检验文件后缀

```javascript
/(.jpg|.gif)+(\?|\#|$)/.test('a/b/c.jpgsss') // false
/(.jpg|.gif)+(\?|\#|$)/.test('a/b/c.jpg?') // true
```

### 用户名正则

```javascript
// 用户名正则，4到16位（字母、数字、下划线、减号）
/^[\d\-]{4,16}$/.test('use_jack') // true
```

### 数字正则

```javascript
/^-?\d+$/.test(12) // true
/^-?\d+$/.test(-12)

// 浮点数
/^(?:[-+])?(?:[0-9]+)?(?:\.[0-9]*)?(?:[eE][\+\-]?(?:[0-9]+))?$/.test(0.2)
```

### Email正则

```javascript
/^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/.test("wowohoo@qq.com");
//输出 true

// 1.邮箱以a-z、A-Z、0-9开头，最小长度为1.
// 2.如果左侧部分包含-、_、.则这些特殊符号的前面必须包一位数字或字母。
// 3.@符号是必填项
// 4.右则部分可分为两部分，第一部分为邮件提供商域名地址，第二部分为域名后缀，现已知的最短为2位。
//   最长的为6为。
// 5.邮件提供商域可以包含特殊字符-、_、.
/^[a-z0-9]+([._\\-]*[a-z0-9])*@([a-z0-9]+[-a-z0-9]*[a-z0-9]+.){1,63}[a-z0-9]+$/.test("wowohoo@qq.com")
```

### URL 正则

```javascript
//URL正则
/^((https?|ftp|file):\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/.test("http://wangchujiang.com");
//输出 true

//获取url中域名、协议正则 'http://xxx.xx/xxx','https://xxx.xx/xxx','//xxx.xx/xxx'
/^(http(?:|s)\:)*\/\/([^\/]+)/.test("http://www.baidu.com");

/^((http|https):\/\/(\w+:{0,1}\w*@)?(\S+)|)(:[0-9]+)?(\/|\/([\w#!:.?+=&%@!\-\/]))?$/.test('https://www.baidu.com/s?wd=@#%$^&%$#')

// 必须有协议 
/^[a-zA-Z]+:\/\//.test("http://www.baidu.com")
```

### 域名表达式

```javascript
/^([a-zA-Z0-9]([a-zA-Z0-9\-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,6}$/.test('blog.csdn.net');
// 输出 true
```

### mac 地址

```javascript
/^([0-9a-fA-F][0-9a-fA-F]:){5}([0-9a-fA-F][0-9a-fA-F])$/.test('dc:a9:04:77:37:20');
// 输出 true
```

### IPv4 正则

```javascript
//ipv4地址正则
/^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test("192.168.130.199");
//输出 true
```

### 16进制颜色

```javascript
//RGB Hex颜色正则
/^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$/.test("#b8b8b8");
//输出 true
```

### 颜色值校验

```javascript
// HEX 颜色正则
/^#?([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$/.test("#ccb2b2")
```