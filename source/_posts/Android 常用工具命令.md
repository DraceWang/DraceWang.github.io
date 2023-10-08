---
title: Android 常用工具命令
date: 2015-9-29 09:29:00
update: 2016-4-21 09:29:00
categories: Android  
tags: 
	- ADB
	- shell
cover: https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg
---

## ADB
ADB通过IP连接设备

```shell
adb connect ***.***.***.***
```
ADB指定IP断开连接

```shellshell
adb disconnect ***.***.***.***
```
ADB日志缓存清空

```shell
adb logcat -c
```
ADB日志抓取到指定路径

```shell
adb logcat -v time > c:\******.log
```
后台抓取日志到盒子

```shell
adb logcat -v time > /mnt/sdcard/******.log &
```
进入shell

```shell
adb shell
```
ADB获取系统高级权限

```shell
adb remount
```
ADB拉出文件

```shell
adb pull /data/data/***** c:\*****
```
ADB推入文件

```shell
adb push **** /****
```
获取属性值

```shell
getprop
```
设置属性值

```shell
setprop <key> <value>
```
### 网络抓包

将tcpdump文件push进设备

shell下

```shelltcpdump -p -vv -s 0 -w /data/data/capture.pcap
```
kill小技巧

shell下

```shellkill `ps |grep icntv | busybox awk '{print $2}'`
```
这个命令可以将grep检索的关键字的进程直接全部杀死

内存查看

shell下

```shellprocrank
```
或者

```shell
dumpsys meminfo
```
这个命令很强大，可以查看很多系统信息，比如surface的堆栈信息

```shell
dumpsys SurfaceFilnger
```