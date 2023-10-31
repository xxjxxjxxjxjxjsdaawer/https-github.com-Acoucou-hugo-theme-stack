+++
author = "coucou"
title = "微信小程序——echarts可视化"
date = "2023-08-01"
description = "微信小程序专题之echarts可视化"
categories = [
    "微信小程序"
]
tags = [
    "微信小程序","echarts可视化"
]
+++

![](1.jpg)

## echarts的使用

注：使用之前需到echarts官网下载 __ec-canvas__ 包

### 1. chart.wxml

```html
<view class="content1">
      <view class="box1">
       <ec-canvas id="mychart-dom-bar1" canvas-id="one" ec="{{ ec1}}"></ec-canvas>
      </view>
</view>

 <!-- <view bindtap="getTime" style="color:block;font-size: 15px;margin-left: 20%;">当前时间 {{timer}}</view>  -->
<view class="content2">
  <view class="sitdown">坐下时间(s)</view>
  <view class="box2">
  <ec-canvas id="mychart-dom-bar2" canvas-id="two" ec="{{ ec2}}"></ec-canvas>
</view>
</view>

<view class="content3">
  <view class="history">历史记录</view>
  <view class="box3" >
    <ec-canvas id="mychart-dom-bar3" canvas-id="three" ec="{{ ec3}}"></ec-canvas>
  </view>
</view>

```

### 2. chart.js

```js
import * as echarts from '../../ec-canvas/echarts';
var util=require('../../utils/util'); 
let{score,xdata,ydata,downcount,watercount}=getApp().globalData; //从app.js中接收到数据
const app = getApp().globalData;

Page({
    // 页面的初始数据
    data: {
        ec1: {
          lazyLoad:true     //仪表
        },
        ec2: {
          lazyLoad:true     //柱状
      },
        ec3: {
          lazyLoad:true   //多柱状
      },

      time:"",
      month:[], //月份
      // xdata:["6:30", "6:45", "9:30","10:00", "13:30", "14:40", "15:50"],  //小时
      alldata:[
        ['product', '久坐次数', '喝水次数', '坐下时长(h)'],  //历史记录
        ['3月1日', 3, 6, 7],
        ['3月2日', 6, 4, 8],
        ['3月3日', 5, 6, 4],
        ['3月4日', 4, 9, 3],
      ],

     downCount:downcount,
     waterCount:watercount,

    },
    /**
     * 生命周期函数--监听页面加载
     */
    //动态刷新
  onLoad: function (options) {
    //获取小时并传到xdata中
    var that = this;
    setInterval(function(){
        that.setData({
            // xdata: util.formatHour(new Date()),   //获取小时并传到xdata中
            month: util.formatMonth(new Date()), //获取月日并传到xdata中
            time: util.formatTime((new Date())),
        });    
    },1000);    
    
    //获取到数据后渲染到图表
   setTimeout(()=>{
    this.lazyComponent=this.selectComponent('#mychart-dom-bar2')  //获取到组件
    this.initone(xdata,ydata),  //坐下时间

    this.lazyComponent=this.selectComponent('#mychart-dom-bar1')  //获取到组件
    this.inittwo(score), //得分

    this.lazyComponent=this.selectComponent('#mychart-dom-bar3')  //获取到组件
    this.initthree(this.data.alldata)   //历史记录
   } ,1000)   //定时为1S

  },
  // 下拉刷新
  onRefresh:function(){
    //导航条加载动画
    wx.showNavigationBarLoading()
    //loading 提示框
    wx.showLoading({
      title: 'Loading...',
    })
    console.log("下拉刷新啦");
    setTimeout(function () {
      wx.hideLoading();
      wx.hideNavigationBarLoading();
      //停止下拉刷新
      wx.stopPullDownRefresh();
    }, 1500)
  },
  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh:function(){
    this.onRefresh();
    this.lazyComponent=this.selectComponent('#mychart-dom-bar2')  //获取到组件
    this.initone(xdata,ydata)  //坐下时间
  },

  //坐下时间
 initone(xdata,ydata){  //手动初始化
  this.lazyComponent.init((canvas,width,height,dpr)=>{
  let chart=echarts.init(canvas,null,{
    width:width,
    height:height,
    devicePixelRatio:dpr
  })

  let option=getOption(xdata,ydata)

  chart.setOption(option)

  this.chart=chart  //将图表实例绑定到this上，方便其他函数访问

  return chart
  })
},

// 得分
inittwo(Score){ //手动初始化
  this.lazyComponent.init((canvas,width,height,dpr)=>{
  let chart=echarts.init(canvas,null,{
    width:width,
    height:height,
    devicePixelRatio:dpr
  })

  let option=OnitChart(Score)

  chart.setOption(option)

  this.chart=chart  //将图表实例绑定到this上，方便其他函数访问

  return chart
  })
},

//历史记录
initthree(xydata){  //手动初始化
  this.lazyComponent.init((canvas,width,height,dpr)=>{
  let chart=echarts.init(canvas,null,{
    width:width,
    height:height,
    devicePixelRatio:dpr
  })

  let option=MoreinitChart(xydata)

  chart.setOption(option)

  this.chart=chart  //将图表实例绑定到this上，方便其他函数访问

  return chart
  })
},

    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {
      //对应的wxml中的ec-canvas id
      this.oneComponent = this.selectComponent('ec1');
      this.oneComponent = this.selectComponent('ec2');
      this.oneComponent = this.selectComponent('ec3');
    },

    /**
     * 生命周期函数--监听页面显示
     */
    // 异步加载数据
})


// 坐下时间
function getOption (xdata,ydata) {
  return{
    title: {
      text: ''
  },
  xAxis: {
    type: 'category',
    data: xdata
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      data: ydata,
      type: 'bar',
      showBackground: true,
      backgroundStyle: {
        color: 'rgba(180, 180, 180, 0.2)'
      }
    }
  ]
}
}

//得分
function OnitChart(Score) {  
  return{
    series: [
      {
        type: 'gauge',
        startAngle: 180,
        endAngle: 0,
        min: 0,
        max: 1,
        splitNumber: 8,
        axisLine: {
          lineStyle: {
            width: 6,
            color: [
              [0.25, '#FF6E76'],
              [0.5, '#FDDD60'],
              [0.75, '#58D9F9'],
              [1, '#7CFFB2']
            ]
          }
        },
        pointer: {
          icon: 'path://M12.8,0.7l12,40.1H0.7L12.8,0.7z',
          length: '12%',
          width: 20,
          offsetCenter: [0, '-60%'],
          itemStyle: {
            color: 'auto'
          }
        },
        axisTick: {
          length: 12,
          lineStyle: {
            color: 'auto',
            width: 2
          }
        },
        splitLine: {
          length: 20,
          lineStyle: {
            color: 'auto',
            width: 5
          }
        },
        axisLabel: {
          color: '#464646',
          fontSize: 20,
          distance: -60,
          formatter: function (value) {
            if (value === 0.875) {
              return 'A';
            } else if (value === 0.625) {
              return 'B';
            } else if (value === 0.375) {
              return 'C';
            } else if (value === 0.125) {
              return 'D';
            }
            return '';
          }
        },
        title: {
          offsetCenter: [0, '-20%'],
          fontSize: 10
        },
        detail: {
          fontSize: 15,
          offsetCenter: [0, '0%'],
          valueAnimation: true,
          formatter: function (value) {
            return Math.round(value * 100) + '分';
          },
          color: 'auto'
        },
        data: [
          {
            value:Score,
            name: '今日综合得分'
          }
        ]
      }
    ]
  }
}
//历史记录
function MoreinitChart(xydata) {  
  return{
    legend: {},
    tooltip: {},
    dataset: {
      source: xydata,
    },
    xAxis: { type: 'category' },
    yAxis: {},
    // Declare several bar series, each will be mapped
    // to a column of dataset.source by default.
    series: [{ type: 'bar' }, { type: 'bar' }, { type: 'bar' }]
  };
}
```

### 3. chart.wxss

```css
/* 得分 */
.content1{
    width: 95%;
    height: 400rpx;
    background-color: white;
    padding-top: 30rpx;
    margin-top: 30rpx;
    margin-left: 2.5%;
    border-radius: 20rpx;
    align-items: center;
}

.box1 {
    width: 120%;
    height: 120%;
    bottom: 0;
    margin-left: -10%;
}

/* 坐下时间 */
.content2 {
    width: 95%;
    height: 700rpx;
    margin-left: 2.5%;
    background-color: white;
    padding-top: 30rpx;
    margin-top: 30rpx;
    margin-left: 2.5%;
    border-radius: 20rpx;
    align-items: center;
  }

  .box2 {
    width: 100%;
    height: 90%;
    bottom: 0;
    left: 0;
    margin-top: 100rpx;
    right: 0rpx;
}

.sitdown{
    float: left;
    width: 300rpx;
    height: 60rpx;
    padding-top: 20rpx;
    border-radius: 20rpx;
    margin-left: 28%;
    margin-top: 30rpx;
    text-align: center;
    background-color: whitesmoke;
}

/* 历史记录 */
.content3 {
    width: 95%;
    height: 1000rpx;
    margin-left: 2.5%;
    background-color: white;
    padding-top: 30rpx;
    margin-top: 30rpx;
    margin-left: 2.5%;
    border-radius: 20rpx;
    align-items: center;
  }
  .box3{
    padding-top: 80rpx;
    margin-top: 70rpx;
    width: 100%;
    height: 80%;
    margin-top: 50rpx;
    bottom: 0;
    left: 0;
    right: 0rpx;
    
}
.history{
    width: 20%;
    padding-top: 20rpx;
    margin-top: 5rpx;
    margin-left: 41%;
    float: left;
    width: 300rpx;
    height: 60rpx;
    padding-top: 20rpx;
    border-radius: 20rpx;
    margin-left: 28%;
    margin-top: 30rpx;
    text-align: center;
    background-color: whitesmoke;
}
```

### 4. chart.json

```json
{
  "usingComponents": {
    "ec-canvas": "../../ec-canvas/ec-canvas"
  },
  "backgroundTextStyle": "light",
  "navigationBarBackgroundColor": "#7cd0ce",
  "navigationBarTitleText": "可视化报告",
  "navigationBarTextStyle": "black",
  "enablePullDownRefresh": true
}
```

## 上传图片

### 1. img.wxml

```html
<view class="three">图片</view>
  <view class="weui-uploader">
    <view class='pics' wx:for="{{imgs}}" wx:for-item="item" wx:key="*this">
        <image class='weui-uploader__img' src="{{item}}" data-index="{{index}}" mode="aspectFill" bindtap="previewImg">
                  <icon type='cancel' class="delete-btn" data-index="{{index}}" catchtap="deleteImg"></icon>
        </image>
    </view>

    <view class="tp_cont {{tj_ycang?'':'hide'}}" bindtap="chooseImg">
      <view class="tp_add">+</view>
    </view>

</view>
```



### 2. img.js

```javascript
// pages/abc/abc.js
const app = getApp();

Page({

    data: {
        imgs: []
       },
      // 上传图片
       chooseImg: function (e) {
        var that = this;
        var imgs = this.data.imgs;
        if (imgs.length >= 9) {
         this.setData({
          lenMore: 1
         });
         setTimeout(function () {
          that.setData({
           lenMore: 0
          });
         }, 2500);
         return false;
        }
        wx.chooseImage({
         // count: 1, // 默认9
         sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
         sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
         success: function (res) {
          // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片
          var tempFilePaths = res.tempFilePaths;
          var imgs = that.data.imgs;
          // console.log(tempFilePaths + '----');
          for (var i = 0; i < tempFilePaths.length; i++) {
           if (imgs.length >= 9) {
            that.setData({
             imgs: imgs
            });
            return false;
           } else {
            imgs.push(tempFilePaths[i]);
           }
          }
          // console.log(imgs);
          that.setData({
           imgs: imgs
          });


          
            wx.uploadFile({
                url: getApp().globalData.url+'/home/login/uploadQuestionFile', //接受图片的接口地址
                filePath: tempFilePaths[0],
                name: 'file',
                formData: {
                    'user': 'test'
                },
                success (res){
                    console.log(res);
                    const data = res.data
                    //do something
                }
            })

         }
        });
       },
       // 删除图片
       deleteImg: function (e) {
        var imgs = this.data.imgs;
        var index = e.currentTarget.dataset.index;
        imgs.splice(index, 1);
        this.setData({
         imgs: imgs
        });
       },
       // 预览图片
       previewImg: function (e) {
         //获取当前图片的下标
        var index = e.currentTarget.dataset.index;
         //所有图片
        var imgs = this.data.imgs;
        wx.previewImage({
         //当前显示图片
         current: imgs[index],
         //所有图片
         urls: imgs
        })
       }
      
})

```

### 3. img.wxss

```css
/* 图片 */
.three{
  margin-top: 27rpx;
}
.weui-uploader{
  margin-top: 16rpx;
}
.tp_add{
  width: 152rpx;
  height: 152rpx;
  border-radius: 10rpx;
  opacity: 1;
  border: 2rpx dashed #999999;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 59rpx;
}

.pics {
  float:left;
  position:relative;
  margin-right:15px;
  margin-bottom:15px;
  }
  .pics image{
    width: 152rpx;
    height: 152rpx;
  }
  .delete-btn{
    width: 20rpx;
    height: 20rpx;
    position: absolute;
    top: -15rpx;
    right: -5rpx;
  }
  .weui-uploader{
    padding: 10rpx;
  }

```

