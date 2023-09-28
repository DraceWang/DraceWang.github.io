---
title: Android高通平台查找指定nv值   
categories: Android  
tags: 
	- nv 
	- qcom 
date: 2020-11-05
---

![post-cover](https://s2.loli.net/2023/09/21/UswxOvhH9yug4LT.jpg)

# 高通查找指定nv值

## 1.安装QXDM

使用高通账号登录https://createpoint.qti.qualcomm.com/tools/#，然后找到QXDM对应自己系统的安装包，或者先按照Qualcomm Package Manager，然后再安装QXDM。

因为后面查看nv需要，还需要安装Product Configuration Assistant Tool (PCAT)。

## 2.使用QXDM查询指定NV值

首先手机通过数据线连接电脑

![image-20201105163743538](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105163743538.png)

如果没有的话，可以通过ADB命令来设置diag口：

```shell
adb shell setprop sys.usb.config diag,serial_cdev,rmnet,adb
adb shell setprop persist.usb.config diag,serial_cdev,rmnet,adb
```

然后打开QSDM，点击连接：

![image-20201105164946696](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105164946696.png)

选择刚才看到的diag的com端口：

![image-20201105165022146](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105165022146.png)

选中后点击右侧的connect按钮，连接成功后软件后面会有数据显示，关闭这个对话框，软件主窗口上选择“show nv browser”：

![image-20201105170321383](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105170321383.png)

如果已经安装，就会自动打开界面并读取：

![image-20201105171342367](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105171342367.png)

读取完成后就会可以在id中输入需要查询的nvid，来查看：

![image-20201105171543336](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105171543336.png)

选中需要读取的项，点击read，如果失败则如下图：

![image-20201105171727278](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105171727278.png)如果成功则会提示success。

当然如果需要临时修改也可在input中输入你需要修改为的值，然后点击write，同样下面会有对应操作的提示：

![image-20201105171910420](C:%5CUsers%5Cwang_shunda%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201105171910420.png)