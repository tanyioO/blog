---
title: JSON序列化与嵌套字符串中引号的转义
tags: [JSON序列化,嵌套字符串的转义]
date: 2019-06-21 09:14:34
permalink:
categories: JavaScript
description: 
image: 

---

<p class="description"></p>

#### 问题

如果需要发给后台一个标准的`JSON数组`，通常情况下使用` JSON.stringify()`对数组进行序列化即可，如果数组比较简单，也可以使用引号手动拼接。在项目开发过程中遇到过这样一个问题，在手动拼接`JSON数组`后，经过[umi-request](<https://github.com/umijs/umi-request>)的自动序列化与反序列化，最终发送的请求参数却不是一个可识别的标准`JSON数组`，而是一个字符串。主要原因在于：对嵌套字符串进行序列化和反序列化时，有可能会将字符串中的引号进行转义，如果拼接的字符串格式不正确，就会出现无法转义的情况，最终只能得到一个字符串而不是`JSON数组`。

<!-- more -->

#### 字符串中的单/双引号

在讨论上面的问题前，先看一字符串的单引号和双引号的区别。在JavaScript中单引号和双引号可以同时使用，混合使用时要遵循一定的准则：

- 双引号`""`中可以嵌套单引号`''`
- 单引号`''`中可以嵌套双引号`""`
- 双引号中也可以嵌套双引号，但必须使用转义`\"`
- 单引号中也可以嵌套单引号，但必须使用转义`\'`

```javascript
var a = 'JavaScript "is" awesome'; 
var b = "JavaScript 'is' awesome";
var c = "JavaScript \"is\" awesome";
var d = "JavaScript "is" awesome"; // Uncaught SyntaxError: Unexpected identifier
var e = 'JavaScript \'is\' awesome'; 
var f = 'JavaScript 'is' awesome'; // Uncaught SyntaxError: Unexpected identifier
var g = "J\"a\'v\'\"aScript \'is\' awesome"; // "J"a'v'"aScript 'is' awesome"
```

理论上来说，不同类型引号之间可以通过转义无限嵌套，但由于可读性太差，一般不建议这么做。

#### JSON序列化和反序列化时出现的问题

回到一开始在项目中遇到的问题，如果使用嵌套字符串手动拼接`JSON数组`，应该如何使用？

```JavaScript
// 单引号内嵌套双引号
dispatch({
  type: 'camera/fetchCameraGroups',
  payload: '["salvo"]',
});
```

![request](https://github.com/tanyioO/image-lib/raw/master/blog/JSONAndString/request1.png)

这个结果正好是想要的标准`json数组`。

换另外一种形式，在双引号内嵌套单引号

```JavaScript
// 双引号内嵌套单引号
dispatch({
  type: 'camera/fetchCameraGroups',
  payload: "['salvo']",
});
```

request payload则是：

![request](https://github.com/tanyioO/image-lib/raw/master/blog/JSONAndString/request2.png)

从结果上看来，请求参数是一个`JSON字符串`而不是`JSON数组`。

通过上面讲述的字符串嵌套规则，在两种方式都支持的情况下，出现问题的原因应该在于umi-request的JSON自动序列化与反序列化中的某一过程。

#### 部分猜测及验证

根据以上猜测，直接使用`JSON.stringify()`和`JSON.parse()`进行验证：

~~~JavaScript
var a = JSON.stringify('["salvo"]'); // ""[\"salvo\"]""
JSON.parse(a); // "["salvo"]"

var b = JSON.stringify("['salvo']"); // ""['salvo']""
JSON.parse(b); // "['salvo']"
~~~

以上可以看出，JSON序列化`'["salvo"]'`时，首先将内层的双引号转义，然后将外层的单引号序列化为双引号，以便反序列化时正确解析；在序列化`"['salvo']"`时，由于内层使用单引号不符合标准的`JSON数组`格式，因此没有对其进行转义或者其他处理，然后将双引号包裹的内容当成了一个完整的字符串。至于在序列化和反序列化时，解析内外层的顺序，使用[umi内置的格式化标准](https://github.com/facebook/react/blob/master/scripts/prettier/index.js)以及[prettier](https://prettier.io/docs/en/api.html)工具验证如下：

~~~javascript
"[\"salvo\"]” // '["salvo"]'

'[\'salvo\']' // "['salvo']"
~~~

可以看出prettier在格式化时，会以内层的引号类型为准，然后在外层采用不同的类型。

#### 结论及疑问

JSON序列化与反序列化时，遵循的是[ECMA262的相关标准](<https://www.ecma-international.org/ecma-262/6.0/#sec-json-object>)，在操作嵌套字符串时是根据JavaScript来具体实现的。因此在使用字符串拼接`JSON数组`时，不仅要符合字符串的嵌套规则，更要遵循JSON的标准数据格式。

当然，为防止这种问题的出现，应当直接使用`JSON.stringify()`创建`JSON数组`，避免偷懒直接使用字符串进行拼接。

疑问在于：从结果上看，JSON的序列化/反序列化与嵌套字符串使用prettier格式化后的效果是相同的，至于是不是对于相同标准的不同实现，我暂时还没有找到相关的资料并验证。

<hr />

