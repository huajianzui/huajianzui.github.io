---
layout: post
title: TypeScript代码片段
date: 2020-03-21
Author: 念书
keyword: TypeScript代码片段
excerpt: TypeScript代码片段
categories: 
tags: [JavaScript]
css: style
comments: true
---



```ts
//数组转元组
function demo() {
    const name: string = 'ming';
    const age: number = 20;
    return [name,age] as const;   //<const> [name,age]
}
const people = demo();
const [name] = people;

function tuplify<T extends unknown[]>(...element:T):T{
	return elements;
}

const people1 = tuplify("ming",18);
```

```js
function get(o: object,name: string){
    return o[name];
}
function get1<T extends object,K extends keyof T>(o: T,name:K):T[K]{
    return o[name];
}
```

```ts
const textEl = <HTMLInputElement>document.querySelector('input');
```



