---
layout:     post
title:      "CPU占用过高分析"
subtitle:   ""
date:       2017-11-28 
author:     "Qiby"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Android
---


#CPU占用过高分析
接手日历两个多月,感慨一下代码很烂.我的工作多数时间都是在接入广告,为了收入嘛,就是略感无聊.前两天测试报出一个日历静置耗电高的问题
拿到问题的时候我是蒙的
Http请求可能耗电,SD卡读写可能耗电,数据库读写,Timer,动画,后台Service 都有可能造成耗电量高.  
该从什么地方分析呢?

adb shell top -d 1 | grep com.android.calendar

日历在前台时 CPU占用竟然高达15%左右.当APP进程的CPU使用率超过1%的时候，都是耗电比较大的了，正常情况下应该处于0%的值，实际上应该是0点几.
好了就是这里.
之后我又验证了几种情况查看cpu使用率 ,例如进入应用时开始Http请求后看到cpu使用升高然后跌落,检查了Timer是否cancel

在这之后当我把日历切换到后台,CPU使用率掉到了0%~1%的正常值,由此猜想此问题可能页面在绘制的时候造成的问题.
接下来使用ddms查看了trace log

![Alt text](./img/2017-11-28 13-55-33-ScreenShot.png)

哦哦 应该就是这里 被标记红了
在绘制过程中执行事件过长
![Alt text](./img/2017-11-28 13-58-54-ScreenShot.png)

追到了具体代码发现是推啊sdk广告的问题...那就没办法再追下去了,是时候展现丢锅的技术了.
