+++
author = "coucou"
title = "微信小程序——云开发"
date = "2023-08-01"
description = "微信小程序专题之云开发"
categories = [
    "微信小程序"
]
tags = [
    "微信小程序","云开发"
]
+++

![](1.jpg)

# 小程序云开发

```js
onLaunch: function () {
    wx.cloud.init({
        env:"cloud1-5gffyu7d7a7a5a08"  // 初始化，填写环境ID
    })
}

const db = wx.cloud.database().collection("test_db") // 创建数据库对象

	// 增
    add_data() {
        db.add({
            data: {
                name: "coucou"
            },
            success(res) {
                console.log("添加成功：", res)
            }
        })
    },
    // 查
    select_data() {
        db.get({
            success(res) {
                console.log("查询成功：", res)
            }
        })
    },
    // 删
    delete_data() {
        db.doc(this.data.db_id).remove({
            success(res) {
                console.log("删除成功", res)
            }
        })
    },
    // 改
    change_data() {
        db.doc(this.data.db_id).update({
            data: {
                name: "hhhhh"
            },
            success(res) {
                console.log("修改成功", res)
            }
        })
    },
    // 上传图片
    add_img() {
        wx.cloud.uploadFile({
            cloudPath: "my_test/1111.jpg",
            filePath: "/image/1.jpg",
            success(res){
                console.log("上传成功", res)
            }
        })
    },
```

