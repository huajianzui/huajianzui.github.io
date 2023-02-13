---
layout: post
title: JavaScript代码片段
date: 2020-03-21
Author: 念书
keyword: JavaScript代码片段
excerpt: JavaScript代码片段
categories: 
tags: [JavaScript]
css: style
comments: true
---

用","分隔数字,我们可以用正则表达式来处理.但是其实有现成的API

```js
Number(1234.12234).toLocaleString("zh",{minimumFractionDigits:4,useGrouping:true});
//minimumFractionDigits 指定小数的位数
//useGrouping:true  可以不写默认为true
```

