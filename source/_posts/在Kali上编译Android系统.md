---
title: 在Kali上编译Android系统    
categories: Android   
tags: 
	- kali 
	- android 
date: 2020-06-12
---

![post-cover](https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg)



## 0.准备工作

* 最新的kali镜像
* AOSP
* 以及一颗耐心，能慢慢解决冒出问题的心

## 1.安装Kali

新版的kali系统可以说真的是很神仙了，首先用这个系统的人大部分都是对linux有一定了解，并且想要学习或者会使用系统内自带的各种网络工具。

我是目前在从事网络方面的工作，所以就选择了kali，为啥会需要Android系统呢，主要是工作相关这里就不细说了，网上有ubuntu以及arch编译Android系统的，但是没有kali的，我也只是记录下自己折腾这些的过程，以后需要可以找到。

大家都知道Android源码就是在ubuntu上编译的，所以ubuntu肯定是可以的，而kali使用的是和ubuntu一样的包管理器，其实都是使用的debain。所以理论上肯定也是可以的，但是ubuntu编译Android源码就有各种问题，系统版本不同需要安装的依赖包都不一样，而且还有32位包的问题，而作为一个搞网络的为啥不在kali上搞Android呢？于是就有了这次折腾。

### 1.1制作U盘启动盘

好了废话不多说，安装kali系统，我用的是最新的kali系统（202006）然后制作一个安装盘，windows系统可以使用win32diskmager或者USBwriter，linux就dd就完事了。

### 1.2安装kali

使用刚制作好的U盘启动盘，启动需要安装的电脑，按照引导，一步步走就好了。

这里说几个容易弯路的点：

1. 硬盘分区，其实很简单，就按照引导给你分的也行。非要自己分区的，那至少有一个boot分区，其他只要别瞎分就行了，然后就是一定要做好数据备份，装系统基本都会格式化。
2. 双系统引导，kali会引导你写入MBR，自己注意点，真的被改写了无法启动也不要急毕竟GRUB是可以用的，可以重新设定好引导就行了，但是如果一直不行，就上网找找双系统启动的办法。
3. 联网，新版kali是安装时候希望你联网的，可以帮你把完整的最新的一些软件包安装上，如果因为什么原因就是没有网也是可以安装一个不依赖网络的系统。

安装好之后启动就可以看到kali的龙标记，自带的Xfce真好看，暗系风格。看到kali龙的标记出来，并且带着光晕动画，真的是爱了爱了。

### 1.3 安装环境

安装必要的一些软件啥的比如后面要用的Android Studio，ssh，virtualbox等等。

#### 1.3.1 修改软件源

首先先修改软件源：

`sudo mousepad /etc/apt/sources.list`

```
#阿里云
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
 
#清华大学
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
 
#中科大
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
     
#浙江大学
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
     
#kali官方
deb http://http.kali.org/kali kali-rolling main non-free contrib
deb-src http://http.kali.org/kali kali-rolling main non-free contrib
```

将上面的源添加到文件中保存即可，然后运行下面的命令更新：

`sudo apt-get update`

然后安装大部分软件都是可以的了。

#### 1.3.2 安装中文输入法

安装ibus拼音，linux发行版一般都没有中文输入法，有人会问为啥不装搜狗，emmm，我公司网络限制。

`apt-get install ibus ibus-pinyin  `

这里需要提到添加PPA源，刚装好的系统会出现` add-apt-repository:command not found`

`sudo apt-get install software-properties-common`

使用上面这个命令可以解决。

#### 1.3.3 调整系统显示时间

不知道是公司网络问题还是别的原因，安装系统后时间显示一直不正确。

`sudo apt-get install ntpdate`

`sudo hwclock -r`和`date`将硬件时间和系统时间都输出看下，如果只是系统时间不正确就可以使用同步把时间调整，而我是两者时间都不对，于是将Shanghai设置到`etc/localtime`

`rm -rf /etc/localtime`

`cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`

但是无论使用ntp的哪个命令都无法正确更新时间

`ntpdate -u ntp.api.bz`

`ntpdate time.windows.com `

结果都是找不到服务器，无法使用ntp同步时间，只好手动修改系统时间

`sudo date -s "20200610 16:00:00"`

为了重启后还能读取到这个时间，把它写入硬件时间

`sudo hwclock -w`

完成后过一会儿时间就变过来了，如果一直没有改变，那么重启下电脑。

#### 1.3.4 安装VirtualBox

对于我们大多数人来说就算是办公也是需要微信之类的，网页当然也行，但是还是会有用到windows的地方，因此会需要安装虚拟机，这里能提一下虚拟机的一个坑

安装完成后启动virtualbox，提示`Kernel driver not installed(rc=-1908)`并且同时提示你使用`modprobe vboxdrv`尝试解决问题。但是当时你使用了这个命令后，又会提示你`modprobe: FATAL: Module vboxdrv not found in directory /lib/modules/***********`这个时候最先考虑是该整体更新系统了或者是BISO安全启动的问题，然后才是是不是virtualbox的服务没有启动。

* `sudo apt install virtualbox-dkms`重新安装virtualbox-dkms或`sudo apt upgrade`更新整个系统

* 引导到BIOS并进入> advanced (f7) > boot > scroll down to" secure boot" > 更改"Windows EUFI mode" to" other OS" > advanced (f7) > boot  > scroll down to" secure boot" > 更改"Windows EUFI mode" to" other  OS"    

## 2.编译Android源码

首先是下载源码，这个推荐清华的镜像

### 2.1 安装编译依赖库

解压之后先别着急开始编译，需要先安装依赖库，不过依赖库里有很多都是32位的，需要先开启32位兼容模式：`sudo dpkg --add-architecture i386`

`sudo apt-get install libx11-dev:i386 libreadline-dev:i386 libgl1-mesa-dev g++-multilib lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip m4 lib32ncurses5-dev x11proto-core-dev libx11-dev  libc6-dev-i386  zip curl zlib1g-dev gcc-multilib g++-multilib  git-core gnupg flex bison gperf build-essential  dpkg-dev libsdl1.2-dev libesd0-dev tofrodos python3-markdown libxml2-utils xsltproc zlib1g-dev:i386  git flex bison gperf build-essential libncurses5-dev:i386  libswitch-perl libxml2-utils `

其中有一些库提示已经有兼容库了，那就直接替换。然后还有些库是找不到的，比如：`libesd0-dev`。那就上debain包的官网上去找https://packages.debian.org/jessie/libesd0-dev，记得把依赖也安装上。

`dpkg -i libesd0-dev_0.2.41-11_amd64.deb`

### 2.2 编译

`source build/envsetup.sh`

`lunch`

`make -j8 2>&1 |tee build.log`

根据线程数自己定义线程数，比如你电脑是4核8线程的cpu就可以用j8，但是会很卡，可以保留2-4个线程来干点别的事情。

然后就是漫长的等待

### 2.3 导入AndroidStudio

这个呢就看是否需要了，我其实就是用来看看源码，找找代码用。

在刚才source lunch过的bash shell中继续操作

`mmm development/tools/idegen/`

`development/tools/idegen/idegen.sh`

修改Android.iml文件把不需要的文件夹排除出去

然后导入AndroidStudio







