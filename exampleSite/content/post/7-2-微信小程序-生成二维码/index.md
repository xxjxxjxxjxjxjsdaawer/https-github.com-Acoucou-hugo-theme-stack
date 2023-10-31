+++
author = "coucou"
title = "微信小程序——生成二维码"
date = "2023-08-01"
description = "微信小程序专题之生成二维码"
categories = [
    "微信小程序"
]
tags = [
    "微信小程序","生成二维码"
]
+++

![](1.jpg)

## 小程序生成二维码

注：使用之前需包含__weapp.qrcode.esm.js__ 文件，可到GitHub下载

### 1. qrcode.wxml

```html
<canvas type="2d" style="width: 260px; height: 260px;" id="myQrcode"></canvas>
```

### 2. qrcode.js

```js
// index.js
import drawQrcode from '../../utils/weapp.qrcode.esm.js'

// 获取应用实例
const app = getApp()

// const query = wx.createSelectorQuery()

Page({
  data: {
    qrcodeWidth: 0
  },

  onLoad: function () {
    const query = wx.createSelectorQuery()
    query.select('#myQrcode')
      .fields({
        node: true,
        size: true
      })
      .exec((res) => {
        var canvas = res[0].node

        // 调用方法drawQrcode生成二维码
        drawQrcode({
          canvas: canvas,
          canvasId: 'myQrcode',
          width: 260,
          padding: 30,
          background: '#ffffff',
          foreground: '#000000',
          text: 'hello world',
        })

        // 获取临时路径（得到之后，想干嘛就干嘛了）
        wx.canvasToTempFilePath({
          canvasId: 'myQrcode',
          canvas: canvas,
          x: 0,
          y: 0,
          width: 260,
          height: 260,
          destWidth: 260,
          destHeight: 260,
          success(res) {
            console.log('二维码临时路径：', res.tempFilePath)
          },
          fail(res) {
            console.error(res)
          }
        })
      })
  },
  // 生成二维码
})
```

