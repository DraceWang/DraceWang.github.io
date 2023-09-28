---
title: ubuntu14.04 配置openssh  
categories: linux  
tags: 
	- ubuntu 
	- openssh
date: 2014-12-09
---

![post-cover](https://i.loli.net/2020/10/27/DwBuaAUOdTmpXyE.jpg)

*本文对本机内虚拟机上开启SSH服务，没有涉及外网桥接等方法，考虑后续更新。*

## 安装OpenSSH server
```
	sudo apt-get install openssh-sever
```
修改ssh配置文件（没有安装vim的，请自行替换文本编辑器：nano ；gedit；vi）  
```
	sudo vim /etc/ssh/ssh_config
```

修改以下四行（配置文件的代码意义请自行百度）
```
	#Port 22
	#Protocol 2,1 
	//取消注释
	GSSAPIAuthentication yes
	GSSAPIDelegateCredentials no  
	//加上注释
```
查看本机的ip
```
	ifconfig
```
可用Xshell等工具连接上述虚拟机等


