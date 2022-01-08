---
layout: post
title: 小程序
date: 2021-09-20
Author: 念书
keyword: 
excerpt: 微信小程序
categories: 
tags: []
comments: true
---

我们人类造物,一般往两极发展以解决各种问题. 一边往巨型化发展,一边往微型化发展,而往往又喜欢靠量变引起质变.当然量变与质变并不是简单的数量堆叠.要采用适当的方式方法,要做好整合也要做好解耦.像现在我们用微前端框架来开发巨石应用.我想当我们用纳米级机器人来组装成巨型机器人时.那么这个巨型机器人就不在拘泥于形状,可以千变万化. 在PC端我们一般采用微前端框架构建巨石应用,而在移动端则是以平台与小程序的形式存在

小程序基于微信,能够使用微信提供的接口,因此体量小,下载迅速,采用了离线存储增强了用户体验.

微信的公众号和小程序开发要先到开放平台申请账号然后下载微信开发者工具进行开发

小程序开发与网页相似只需要掌握一些API即可,所以阅读微信的开发指南和API就能很好的进行开发,这里记录一下遇到的问题

首先开发工具与运行环境有一定的差别

| 运行环境         | 逻辑层     | 渲染层           |
| ---------------- | ---------- | ---------------- |
| iOS              | JavaScript | WKWenView        |
| 安卓             | V8         | chromium定制内核 |
| 小程序开发者工具 | NWJS       | Chrome Webview   |

小程序每个page由四部分组成 与html类似wxml,css类似的wxss,js文件以及配置文件,与网页不同的是标签种类,css选择器,js的一些操作dom的方法. 渲染层与逻辑层互不影响.小程序的逻辑层大量的使用事件+回调函数的方式,包括获取Dom节点.

我的demo项目是一个狗狗展示的具有6,7个页面的小项目,使用的云服务.云服务操作数据库的方式与平常的sql略有不同.每个表格都可以进行权限配置,每个表都有默认的openId字段.通过配置权限给不同的用户展示不同的内容,新版本的小程序要求必须传递openid字段.云服务是以表单位进行连接

```javascript
const db = wx.cloud.database();  //数据库
const _ = db.command;//指令
const table = "image";
const conn = db.collection(table);//建立连接

function getUserImage(type=0){ //查询
   return  conn.where({
        _openid: "{openid}",
        type
     }).get();
}
function getUserImagePage(type=0,page=0,pageSize=5){//分页查询
    return  conn.where({
        _openid: "{openid}",
        type
     }).skip(page*pageSize).limit(pageSize).get();
}
function getUserImageByPath(path) { //in指令
    return conn.where({
        _openid: "{openid}",
        path: _.in(path)
    }).get();
}
function addUserImage(path,type) {//新增
    let data = {
        createTime: new Date(),
        path,
        type
    };
   return conn.add({data});
}
function deleteUserImage(path) {//删除
   return  conn.where({
        path
    }).remove();
}
```

请求网络接口

```javascript
        wx.request({
            url: this.data.url,
            method: "GET",
            success: (res)=> {
               
            },
            fail:(err)=> {
            }
        })
```





图片上传,下载,展示,图片上传返回id,然后要通过id去查询图片的临时路径,临时路径具有有效期

```javascript
function downloadImage(url) {//下载图片然后保存到相册
    wx.showModal({
        title:"提示",
        content:"是否下载图片?",
        success:(res)=> {    
            if(res.cancel) {
                return
            } ;
            wx.downloadFile({
                url: url,
                success: (res)=> {
                    wx.saveImageToPhotosAlbum({
                        filePath: res.tempFilePath,
                        success(res) { 
                            console.log(res);
                        }
                      })
                }
            })
        }
    })

}
 uploadFile() {
        try{
            wx.chooseImage({
                count: 1,
                sizeType: ['original', 'compressed'],
                sourceType: ['album', 'camera'],
                success (res) {
                    wx.cloud.uploadFile({
                        cloudPath: new Date().getTime()+".png",
                        filePath: res.tempFilePaths[0]// 文件路径
                    }).then(res => {
                        DBUtil.addUserImage(res.fileID,1).then(res=>{
                            wx.showToast({
                                icon:"success",
                                title: '上传成功',
                            })
                        })
                    })
                }
            })
        }catch{
            wx.showToast({
                icon:"error",
                title: '上传失败'
            })
        }
    }
wx.cloud.getTempFileURL({//根据fileid查询路径
    fileList:  imageList,
    success:res=>{
        let list = res.fileList;
        let urlList = list.map(n=>n.tempFileURL);
        console.log(urlList);
        this.setData({
            imageList: [...this.data.imageList,...imageList],
            loveList: [...this.data.loveList,...loveList],
            message: "fill"
        });
    }       
})
```

canvas作为原生组件,新版的小程序支持原生的语法,只是模拟器显示还有一定的问题,真机调试要采用新的调试选项不然会报错

```javascript
let query = wx.createSelectorQuery(); 
query.select("#myCanvas").node((res)=>{
     let canvas = res.node;
     let ctx = canvas.getContext("2d");
     ctx.strokeStyle = '#AAAAAA';
     ctx.lineCap="round";
     ctx.lineJoin  = "round";
     ctx.lineWidth=1;
     wx.getSystemInfo().then((res)=>{//拿到宽高以适应不同的分辨率和dpr
         let dpr = res.pixelRatio;
         canvas.width = res.windowWidth* dpr;
         canvas.height = res.windowWidth* dpr;
         ctx.scale(dpr, dpr);
     });         
 }).exec();
query.select("#myCanvas").node((res)=>{
    let url = res.node.toDataURL("image/png");
    this.setData({
        "imageUrl":url
    })
}).exec();
```

由上边的的部分方法可以看出,大多数接口都是以回调的方式供我们使用,少部分支持了promise,并且需要执行方法调用,像上边的get方法,exec方法

小程序的事件绑定分为冒泡bind与禁止冒泡catch,组件传参可以使用自定义事件和prop

项目使用App()构造,页面使用Page()构造,组件使用Component,组件的方法要写在methods里边,另外注意一点使用1个以上的插槽需要修改配置.

```javascript
const common = require("../../lib/common.js");
// components/imageList/imageList.js

Component({
    options: {
        multipleSlots: true // 在组件定义时的选项中启用多slot支持
      },
    /**
     * 组件的属性列表
     */
    properties: {
        imageList: {
            type: Array,
            value: []
        },
        loveList: {
            type: Array,
            value: []
        },
        loveImage:{
            type: String,
            value:"/images/redHeart.png"
        },
        defaultimage:{
            type: String,
            value: "/images/myLove-sel.png"
        }
    },

    /**
     * 组件的初始数据
     */
    data: {

    },

    /**
     * 组件的方法列表
     */
    methods: {
        selLove: function(data){
            let index =data.currentTarget.dataset.index;
            this.triggerEvent("love",{index})
        },
        downloadImage: function(data) {
            let url =this.data.imageList[data.currentTarget.dataset.index];
            common.downloadImage(url);
        }
    }
})

```

关于路由,小程序有tabBar页面和普通页面的区别,也有普通跳转和重定向,小程序内可以维护十层页面栈

还有一些微信特有的分享,跳转来源(场景),转发,sitemap之类的并没有全部用到

另外一个比较有趣的是Behavior,这次也没有使用,它具有共享生命周期,修改自定义组件的功能

```javascript
    onShareAppMessage: function () {
        return {
            title: "可爱的狗狗",
            path: "pages/imagePage/imagePage",
            imageUrl: this.data.imageList[0]
        }
    },
    onShareTimeline: function() {
        return {
            title: "可爱的狗狗",
            path: "pages/imagePage/imagePage",
            imageUrl: this.data.imageList[0]
        }
    },
```



说一下配置文件,每个页面都有配置文件,整个项目又有一个总的page文件,路由信息写在配置文件中,第一个默认为主页,tabBar,还有小程序的一些窗口样式,以及引用的库,配置文件不允许加注释且要用"",

页面的配置文件可以配置引用的组件,当前页面的窗口样式

```json
{
    "pages": [
        "pages/dogRandom/dogRandom",
        "pages/myLove/myLove",
        "pages/myOwn/myOwn",
        "pages/imagePage/imagePage",
        "pages/drawDog/drawDog"
    ],
    "window": {
        "backgroundColor": "#F6F6F6",
        "backgroundTextStyle": "light",
        "navigationBarBackgroundColor": "#F6F6F6",
        "navigationBarTitleText": "Dog",
        "navigationBarTextStyle": "black"
    },
    "tabBar": {
        "list": [
            {
                "pagePath": "pages/dogRandom/dogRandom",
                "iconPath": "images/homepage-sel.png",
                "selectedIconPath": "images/homepage.png",
                "text": "首页"
            },
            {
                "pagePath": "pages/myLove/myLove",
                "iconPath": "images/myLove-sel.png",
                "selectedIconPath": "images/myLove.png",
                "text": "我喜欢"
            },
            {
                "pagePath": "pages/myOwn/myOwn",
                "iconPath": "images/myOwn-sel.png",
                "selectedIconPath": "images/myOwn.png",
                "text": "我的"
            }
        ]
    },
    "useExtendedLib": {
        "weui": true
    },
    "sitemapLocation": "sitemap.json",
    "style": "v2"
}

{
  "usingComponents": {
    "image-list": "/components/imageList/imageList",
    "mp-icon": "weui-miniprogram/icon/icon"
  }
}
```



最后说一下真机测试和体验版,必须在开放平台进行配置之后才能进行测试和体验