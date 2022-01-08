---
layout: post
title: 多媒体
date: 2021-09-16
Author: 念书
keyword: 多媒体
excerpt: Audio,Video,Image,Map
categories: 
tags: [html,javaScript]
comments: true
---

HTML5的登场,带来了对音频,视频,canvas,svg,Map等等API的支持,取代了Flash在网页中的地位,使得网页的内容变得更加丰富多彩.



### Canvas

canvas在网页图形,动画,包括图片,视频等等处理上都占有一席之地,现在更是可以通过webGL来支持3D.

canvas以标签的形式在网页上占位,可以通过css指定宽高,但是同时也要通过js指定宽高,否则会使用默认的宽高(替换元素的默认宽高为300*150)来进行缩放,这也是有时canvas显得模糊的原因,当然亦可以利用这个特性通过逻辑像素 * dpr的形式来解决高倍屏的问题

```javascript
let dpr = res.pixelRatio;
canvas.width = res.windowWidth* dpr;
canvas.height = res.windowWidth* dpr;
ctx.scale(dpr, dpr);
```

用canvas绘制图形,绘制的先后顺序会对结果有一定的影响,canvas亦可以通过组合的方式解决重叠图形问题ctx.globalCompositeOperation



canvas能够对图片进行像素级处理,来实现一些特殊效果,又可以转化为图片,

```javascript
ctx.drawImage(image,0,0,this.width,this.height);//把图片添加到画布
imageData = ctx.getImageData(x,y,1,1).data;//获取像素数组,4个为一组,可遍历修改
ctx.putImageData(imagedata, x, y, dx, dy, width, height);//把修改的设置回画布
canvas.toDataURL(type, encoderOptions);//转化为图片,需要注意这个方法是canvasdom的方法
```

甚至可以与video事件配合来操作视频,换掉视屏的背景色,处理弹幕等等,track元素只能生成静态字幕,当然现在很多网站都是用普通的div来实现的弹幕..

```javascript
  processor.computeFrame = function computeFrame() {
    this.ctx1.drawImage(this.video, 0, 0, this.width, this.height);
    let frame = this.ctx1.getImageData(0, 0, this.width, this.height);
    let l = frame.data.length / 4;

    for (let i = 0; i < l; i++) {
      let r = frame.data[i * 4 + 0];
      let g = frame.data[i * 4 + 1];
      let b = frame.data[i * 4 + 2];
      if (g > 100 && r > 100 && b < 43)
        frame.data[i * 4 + 3] = 0;
    }
    this.ctx2.putImageData(frame, 0, 0);
    return;
  }
```





File

usb



