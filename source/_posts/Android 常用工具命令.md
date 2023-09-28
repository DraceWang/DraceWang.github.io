---
title: Android 常用工具命令
date: 2015-9-29
updated: 2016-4-21
categories: Android  
tags: 
	- ADB
	- shell
---
![post-cover](https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg)
## ADB
ADB通过IP连接设备

```
adb connect ***.***.***.***
```
ADB指定IP断开连接

```
adb disconnect ***.***.***.***
```
ADB日志缓存清空

```
adb logcat -c
```
ADB日志抓取到指定路径

```
adb logcat -v time > c:\******.log
```
后台抓取日志到盒子

```
adb logcat -v time > /mnt/sdcard/******.log &
```
进入shell

```
adb shell
```
ADB获取系统高级权限

```
adb remount
```
ADB拉出文件

```
adb pull /data/data/***** c:\*****
```
ADB推入文件

```
adb push **** /****
```
获取属性值

```
getprop
```
设置属性值

```
setprop <key> <value>
```
### 网络抓包

将tcpdump文件push进设备

shell下

```
tcpdump -p -vv -s 0 -w /data/data/capture.pcap
```
kill小技巧

shell下

```
kill `ps |grep icntv | busybox awk '{print $2}'`
```
这个命令可以将grep检索的关键字的进程直接全部杀死

内存查看

shell下

```
procrank
```
或者

```
dumpsys meminfo
```
这个命令很强大，可以查看很多系统信息，比如surface的堆栈信息

```
dumpsys SurfaceFilnger
```