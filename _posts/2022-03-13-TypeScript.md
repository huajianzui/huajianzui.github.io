---
layout: post
title: TypeScript
date: 2022-03-13
Author: 念书
keyword: TypeScript
excerpt: TypeScript
categories: 
tags: [JavaScript]
comments: true
---

TypeScript 是JavaScript的超集.可以编译成JavaScript. 

V8有一项预设类型的优化,通过假设类型不会改变来减少编译的时间

ts提供静态类型进行代码检查. 能够避免一些开发时的错误. 

标注类型提高了代码的可读性



##### 两个需要注意的库

+ `lib.dom.d.ts`        dom的类型定义
+ `lib.es5.d.ts`         es语法

##### 工具

+ [quicktype](https://app.quicktype.io/)  提供json到type的转换.  可以集成到项目中自动转换
+ ts-node-dev 编译执行ts的服务

##### tsc

我们可以通过tsc命令编译一整个项目的文件,tsc会去寻找项目中的tsconfig.json文件.

也可以 tsc filename 去编译单个文件,这时会忽略掉tsconfig.json. 我们可以通过类似于 target的一些修饰符来为单个文件指定配置  tsc -- target es2015 filename





##### 基础

+ string,boolean,number,null,undefined 基本类型不用多说

+ any any代表任意一种类型,但使用any就使得ts失去了意义,一般会通过noImplicitAny禁止无意义的any

+ 枚举 一组具有相同特征的常量
  + `enum Days {Sun, Mon,Tue, Wed,Thu, Fri,Sat}`

+ 数组 类型相同的一组数据

  + `let arr:number[] = [1,2,3]`
  + `let arr = new Array<number>(4)`
  + `interface NumberArray { [index: number]:number}`
  + `function sun() { let args:IArguments = arguments};` 类数组

+ function

    ```ts
    //当一个类型中的方法是匿名方法,这种type为funtion,而且可以拥有属性
    type DescribableFunction = {
      description: string;
      (someArg: number): boolean;
    };
    // 通过new关键字来表明是否是构造函数  函数内部可以通过new.target判断是否是构造函数
    interface CallOrConstruct {
      new (s: string): Date;
      (n?: number): number;
    }
    //限制this  通过this关键字限制this,如果不是此类型的实例调用会报错,此时的this不作为参数
    type IUser = {
       readonly id: number,
       getId(this:IUser):number,
      // getId:(this:IUser)=>number
    }
    
    const user:IUser = {
        id:1,
        getId(){
          return this.id;
        }
    }
    
    console.log(user.getId());
    let getId1 = user.getId;
    getId1();//报错
    
    ```

    

+ 元组 类型不相同的一组数据

    + `let arr:[number,string]=[1,"2"]`  长度受限,类型受限
    + `let arr:(number|string)[] =[1,2,"3"] `

+ 可选

    + ```js
        //作为参数时要放在后边
        function getName(firstName: string,lastName?:string):string {
            return firstName + lastName??"";
        }
        ```

    + ```ts
        interface Person = {
        	readonly id:  number,
            name: string,
            age?: number,
            [proName;string]:any
        }
        ```

+ type ,interface

    + type 定义类型 不能有同名的type

    + ```ts
      type Point = { //type定义的类型在气泡注释中会展开,方便观察
      	x: number,
          y: number
      }
      ```

    + interface 也可以定义类型,同名的interface会merge,所以他是可扩展的

+ 类型断言  把不具体的类型转为具体的类型,断言的基础是开发者比ts更加明确参数的类型

    ```ts
    /* 通过<type> 或 as type 指定类型,但是这是纯粹的ts行为 并不会发生强转*/
    function getLength(str:string|number) {
        if((<string> str).length) {
            return (str as string).length
        }else {
            return str.toString().length
        }
    }
    //非空断言
    function getLength(str?:string){
        return str!.length
    }
    ```

    

+ 文字类型

    ```ts
    function setSex(obj:object,sex:"男"|"女"){//限制选项
        obj.sex = sex;
    } 
    
    let xiaoMing = {name:"小明",sex:"男"}
    setSex(xiaoMing,1);//报错
    setSex(xiaoMing,"男");//通过
    setSex(xiaoMing,xiaoMing.sex);//报错 这里打的sex会被推断为string
    ```

+ 类型推断

  ```TS
  //ts具有类型推断,即使不设置类型,也会根据初始值推断出类型.
  //类型推断往往是推断出原始的类型,如果是自定义的类型往往需要与断言配合
  let a;  //any
  let a = 1; //number  此时不能改变类型  a="a";
  let a = true? 1:"a"; //尽管是1,但是还是会推断为number|string 
  ```

+ Narrowing  我们通过一些额外的判断(分支)可以把宽泛的类型具体化,然后分别处理

+ 泛型  不能确定类型,但是能够确定(约束)类型的关系的时候使用泛型

  ```ts
  function getFirst<T>(arr:T[]):T{
  	return arr[0]
  }
  //限制调用时只能取某个对象拥有的值,防止使用时出现错误
  function get<T extends object,K extends keyof T>(o: T,name:K):T[K]{
      return o[name];
  }
  //包括官网上 限定length属性
  function longest<Type extends { length: number }>(a: Type, b: Type) {
    if (a.length >= b.length) {
      return a;
    } else {
      return b;
    }
  }
  ```

+ 特有的类型

  + void: 没有返回值

  + any : 所有类型,代表任意类型,any属性拥有所有的属性和方法

  + unknown : 未知类型,可以表示任何值,但是与any不同,没有属性和方法

  + never: 不存在的类型,不会被观察到,不会应该发生的情况

    ```ts
    interface Circle {
      kind: "circle";
      radius: number;
    }
     
    interface Square {
      kind: "square";
      sideLength: number;
    }
    
    interface Triangle {
      kind: "triangle";
      sideLength: number;
    }
     
    type Shape = Circle | Square | Triangle;
     
    function getArea(shape: Shape) {
      switch (shape.kind) {
        case "circle":
          return Math.PI * shape.radius ** 2;
        case "square":
          return shape.sideLength ** 2;
        default:
          const _exhaustiveCheck: never = shape;
    //报错 Type 'Triangle' is not assignable to type 'never'.
          return _exhaustiveCheck;
      }
    }
    
    function fn(x: string | number) {
      if (typeof x === "string") {
        // do something
      } else if (typeof x === "number") {
        // do something else
      } else {
        x; // has type 'never'!
      }
    }
    ```

+ 函数重载  多个方法签名实现重载,重载是参数列表不同,方法明相同的一组函数,

  ```TS
  //重载声明
  function getDate(timestamp:number):Date;
  function getDate(y:number,m:number,d:number):Date;
  //实现声明
  function getDate(timestampOrY:number,m?:number,d?:number):Date {
      if(m===undefined){
          return new Date(timestampOrY);
      }else {
          return new Date(timestampOrY,m,d);
      }
  }
  //尽管实现签名中m,n并不是绑定的,但是重载签名中对实现签名做了限制,此时不能传递2个参数.
  
  interface CallableFunction extends Function {
  
      apply<T, R>(this: (this: T) => R, thisArg: T): R;
      apply<T, A extends any[], R>(this: (this: T, ...args: A) => R, thisArg: T, args: A): R;
  
      call<T, A extends any[], R>(this: (this: T, ...args: A) => R, thisArg: T, ...args: A): R;
  
      bind<T>(this: T, thisArg: ThisParameterType<T>): OmitThisParameter<T>;
      bind<T, A0, A extends any[], R>(this: (this: T, arg0: A0, ...args: A) => R, thisArg: T, arg0: A0): (...args: A) => R;
      bind<T, A0, A1, A extends any[], R>(this: (this: T, arg0: A0, arg1: A1, ...args: A) => R, thisArg: T, arg0: A0, arg1: A1): (...args: A) => R;
      bind<T, A0, A1, A2, A extends any[], R>(this: (this: T, arg0: A0, arg1: A1, arg2: A2, ...args: A) => R, thisArg: T, arg0: A0, arg1: A1, arg2: A2): (...args: A) => R;
      bind<T, A0, A1, A2, A3, A extends any[], R>(this: (this: T, arg0: A0, arg1: A1, arg2: A2, arg3: A3, ...args: A) => R, thisArg: T, arg0: A0, arg1: A1, arg2: A2, arg3: A3): (...args: A) => R;
      bind<T, AX, R>(this: (this: T, ...args: AX[]) => R, thisArg: T, ...args: AX[]): (...args: AX[]) => R;
  }
  ```

  