

## 简述

​	这是一个简单的微信小程序，主要使用了云数据库、云存储和云函数。小程序名为医药小助手，Medical-Assistant压缩包内的是源码。

## 功能

​	小程序的功能为：

* 展示四类常见药品的图片、详细说明和使用指南。点击药品名进入详细内容页面。
* 展示四个地区的三甲医院的图片、介绍和就医指南。点击医院名进入详细内容页面。
* 使用云数据库向开发者提出建议。可以输入意见建议，点击提交至云数据库。

## 页面

小程序有三个主要页面构成

* **安全用药**
* **安心就医**
* **意见建议**

安全用药和安心就医页面下还包括了多个页面，对应不同的药品和医院。

页面信息如下

```javascript
"pages": [
    "pages/medicine/medicine",
    "pages/medicine/ajhf/ajhf",
    "pages/medicine/999/999",
    "pages/medicine/lhqw/lhqw",
    "pages/medicine/prxt/prxt",
    "pages/medicine/hxzqy/hxzqy",
    "pages/medicine/mdn/mdn",
    "pages/medicine/bhw/bhw",
    "pages/medicine/jwxsp/jwxsp",
    "pages/medicine/aqmsjn/aqmsjn",
    "pages/medicine/tbkwjn/tbkwjn",
    "pages/medicine/hmsjn/hmsjn",
    "pages/medicine/lhmsjn/lhmsjn",
    "pages/medicine/pyp/pyp",
    "pages/medicine/dkn/dkn",
    "pages/medicine/dndrg/dndrg",
    "pages/medicine/prs/prs",
    "pages/hospitical/hospitical",
    "pages/hospitical/scdxhxyy/scdxhxyy",
    "pages/hospitical/scsrmyy/scsrmyy",
    "pages/hospitical/bjxhyy/bjxhyy",
    "pages/hospitical/bjdtyy/bjdtyy",
    "pages/hospitical/301yy/301yy",
    "pages/hospitical/tjyy/tjyy",
    "pages/hospitical/xyyy/xyyy",
    "pages/hospitical/whdxrmyy/whdxrmyy",
    "pages/thanks/thanks"
  ],
```

## 实现

* 小程序的三个主要页面使用**tabbar设置**在app.json中

  ```javascript
    "tabBar": {
      "color": "#dddddd",
      "selectedColor": "#3cc51f",
      "borderStyle": "black",
      "backgroundColor": "#ffffff",
      "list": [
        {
          "pagePath": "pages/medicine/medicine",
          "iconPath": "images/wechat.png",
          "selectedIconPath": "images/wechatHL.png",
          "text": "安全用药"
        },
        {
          "pagePath": "pages/hospitical/hospitical",
          "iconPath": "images/wechat.png",
          "selectedIconPath": "images/wechatHL.png",
          "text": "安心就医"
        },
        {
          "pagePath": "pages/thanks/thanks",
          "iconPath": "images/wechat.png",
          "selectedIconPath": "images/wechatHL.png",
          "text": "意见建议"
        }
      ]
    },
  ```

  

* 对于安全用药和安心就医主界面，使用了WeUI中的**Panel组件**。

  ```javascript
  <view class="top-text-view" style="text-align:center">
  	<text class="top-text"></text>
  </view>
  
  <view class="weui-panel weui-panel_access">
  	<view class="weui-panel__hd"></view>
  	<view class="weui-panel__bd">
  		<navigator url="./ajhf/ajhf">
  			<a class="weui-media-box weui-media-box_appmsg">
  				<view class="weui-media-box__hd">
  					<image class="weui-media-box__thumb" src="cloud://menofletters-51imr.6d65-menofletters-51imr-1302633453/medicine/d7a2675145397d67d2f377521a076d39.jpg" alt></image>
  				</view>
  				<view class="weui-media-box__bd">
  					<h4 class="weui-media-box__title"></h4>
  					<view class="weui-media-box__desc"></view>
  				</view>
  			</a>
  		</navigator>
  	</view>
  </view>
  ```

  

* 对于每一张图片，都使用云存储保存，云数据库保存文件信息的方式渲染到小程序上，以求小程序的包袱最轻化。渲染图片时使用的是cloud链接，直接从云存储请求图片进行渲染。

  ```javascript
      imgUrls: [
        'cloud://menofletters-51imr.6d65-menofletters-51imr-1302633453/medicine/7787ef71c6f8faca8e1bebe086adc955.jpeg',
        'cloud://menofletters-51imr.6d65-menofletters-51imr-1302633453/medicine/80b16062a569d12bd21159508c694604.jpeg',
        'cloud://menofletters-51imr.6d65-menofletters-51imr-1302633453/medicine/0c492f3a27eaaec5c860aaa5b9d43aa9.jpg'
      ],
  ```

  

* 对于药品和医院的说明，将所有文本信息写入到云数据库集合medicine和hospitical中，在每个页面的wxml使用几乎相同的代码，在页面的js文件中根据不同的药品名向云数据库发送查询请求，得到的数据渲染到页面中。这样做，当药品信息出现改变时，只需在云数据库中进行修改即可，不必修改源码。

  连接数据库并发送查询请求：

  ```javascript
    onLoad: function (options) {
      const db = wx.cloud.database() //获取数据库的引用
      const _ = db.command //获取数据库查询及更新指令
      db.collection("medicine") 
        .where({ //查询的条件指令where
          name: _.eq(""),
        })
        .skip(0) //跳过多少个记录（常用于分页），0表示这里不跳过
        .limit(10) //限制显示多少条记录，这里为10
        .get() //获取根据查询条件筛选后的集合数据
        .then(res => {
          console.log(res.data)
          this.setData({
            name: res.data[0].name,
            kind: res.data[0].kind,
            indication: res.data[0].guide.适应症,
            Usage: res.data[0].guide.用法用量,
            Adverse_Reactions: res.data[0].guide.不良反应,
            matters: res.data[0].guide.注意事项
          })
        })
        .catch(err => {
          console.error(err)
        })
    },
  ```

  得到数据后，使用data内的数据进行修改和渲染

  ```javascript
    data: {
      name: "name",
      kind: "kind",
      indication: "indication",
      Usage: "Usage and dosage",
      Adverse_Reactions: "Adverse reactions",
      matters: "matters",
    },
  ```

* 在药品和医院的详细数据页面，数据来源于js文件中的data，整体采用WeUI中的article组件。

  ```javascript
  <image src="cloud://menofletters-51imr.6d65-menofletters-51imr-1302633453/medicine/d7a2675145397d67d2f377521a076d39.jpg" mode="widthFix"></image>
  
  <view class="page" data-weui-theme="{{theme}}">
  	<view class="page__bd">
  		<view class="weui-article">
  			<view class="weui-article__h1" class="text-h1">{{name}}</view>
  			<view class="weui-article__section">
  				<view class="weui-article__h2" class="text-h2">{{kind}}</view>
  				<view class="weui-article__section">
  					<view class="weui-article__h3">适应症</view>
  					<view class="weui-article__p">
  						{{indication}}
  					</view>
  				</view>
  				<view class="weui-article__section">
  					<view class="weui-article__h3">用法用量</view>
  					<view class="weui-article__p">
  						{{Usage}}
  					</view>
  				</view>
  				<view class="weui-article__section">
  					<view class="weui-article__h3">不良反应</view>
  					<view class="weui-article__p">
  						{{Adverse_Reactions}}
  					</view>
  				</view>
  				<view class="weui-article__section">
  					<view class="weui-article__h3">注意事项</view>
  					<view class="weui-article__p">
  						{{matters}}
  					</view>
  				</view>
  			</view>
  		</view>
  	</view>
  </view>
  ```

* 在意见建议页面，使用了**textarea组件**并且在js文件中设置为点击按钮后进行聚焦和上传

wxml片段

````javascript
<view class="weui-cells__title">给出您宝贵的建议！</view>
<view class="section">
  <form bindsubmit="bindFormSubmit">
    <textarea placeholder="请输入您的宝贵建议！完成后点击提交" name="textarea"/>
    <button form-type="submit"> 提交 </button>
  </form>
</view>
````

js片段

````javascript
  bindFormSubmit: function (e) {
    console.log(e.detail.value.textarea)
    var suggestion = e.detail.value.textarea
    const db = wx.cloud.database() //获取数据库的引用
    const _ = db.command //获取数据库查询及更新指令
    db.collection('opinion').add({
        data: {
          suggestions: suggestion
        }
      })
      .then(res => {
        console.log(res)
        wx.showToast({
          title: '提交成功！谢谢您的意见！',
          icon: 'success',
          duration: 1000
        })
      })
  },
````

这里原本是使用了两个函数，但是遇到了一些问题，详细见后续内容。

* 在云数据库的数据采用medicine和hospitical两个集合。每一个字段由string类型name、string类型kind和对象类型guide组成，guide类型包含了药品和医院的属性。

## 错误纠正与改进方案

* 在意见建议页面原本使用了两个函数进行处理，一个函数进行提取文本，一个函数进行上传。然而在调试时出现问题，点击提交后提交的上一个文本。改为使用一个函数触发完成。
* 在云开发指南中[https://cloudbase.net/community/guides/handbook/tcb23.html](https://cloudbase.net/community/guides/handbook/tcb23.html) 的添加记录部分，省略了连接数据库、获取数据库的引用和获取数据库查询及更新指令内容，导致开发时出现问题。
* 在让文字居中的操作中，必须使用

```javascript
<view class="top-text-view" style="text-align:center">
```

在使用css选择器进行设置时不成功。

* 使用云数据库进行数据设置时，感觉不方便。在后台云控制端无法批量删除字段，无法复制格式。录入时不能拉大文本框进行格式设置。可以在js页面进行上传。
* 对**swiper轮播组件**的理解不够深入，没为每一个轮播图片插入链接。

## 致谢

​	经过十天的训练营学习，把我从纯粹的前端零基础带入了前端世界的大门。学习到了使用云函数、云数据库和云存储进行开发。使用云函数、云数据库和云存储进行开发可以节省大量的时间和空间，将本地端的包袱降到最小。

​	感谢吴博群老师和白宦成老师的耐心指导和帮助。

