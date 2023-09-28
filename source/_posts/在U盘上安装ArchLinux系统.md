---
title: 在U盘上安装ArchLinux系统   
categories: Linux  
tags: 
	- archlinux 
	- mobile system 
date: 2016-02-04
update: 2017-08-29
---

![post-cover](https://s2.loli.net/2023/09/21/UswxOvhH9yug4LT.jpg)

By Drace Wang  Feb 4 2016 Updated:Aug 29 2017
Contents
1. 在U盘上安装ArchLinux系统
1.1. 准备工作
1.1.1. U盘
1.1.2. ArchLinux安装镜像
1.1.3. 找个安装环境——VMware
1.2. 安装ArchLinux
1.2.1. 分区
1.2.1.1. 格式化U盘建立文件系统
1.2.2. 联网
1.2.2.1. 驱动
1.2.2.2. 网络部分的一些小命令
1.2.3. 挂载分区
1.2.4. 安装
1.2.5. 设置新系统
1.2.5.1. 设置U盘启动
1.2.5.2. 安装引导程序
1.2.5.3. 新建用户
1.2.5.4. 更新下系统
1.2.5.5. 配置WIFI
1.2.5.6. 安装声卡服务
1.2.6. 安装KDE桌面环境
1.2.7. 安装GNOME桌面环境
1.2.7.1. 美化gnome
在U盘上安装ArchLinux系统
其实在U盘上安装Linux系统基本都能装,也有针对U盘设计的系统像puppy之类的,选Arch主要是个人喜好,还有就是它的安装是手动的,不仅能学到不少linux的知识,也能更好的去适配不同的电脑,arch的可定制化程度非常高,所以更加的灵活也能适应更多的人去使用

## 0.准备工作
### 0.1U盘

要装系统U盘不能太差,我要求比较高,U盘建议16G以上,高速读写不用说最好100m/s以上,4K性能不能忽略(别问我为什么知道,到时候你发现几百大洋买回的U盘用的很不爽别来找我),SanDisk至尊系列亲测还行,如果你们能买到OWC的envy Pro mini最好,这是我到目前为止看的觉得最好的,但是我没有用过,4k性能不知道

### 0.2ArchLinux安装镜像

ArchLinux官方网站
自己下载最新的镜像,以下步骤为2016年1月尝试实践记录,请参考wikiArchLinux官方安装指导自行对比

ArchLinux的wiki非常丰富,要养成自行查找,解决问题的好习惯(说给自己听的)

 ### 0.3找个安装环境——VMware

安装到U盘的确可以直接用实体机启动,但是做live盘要多用一个U盘或者cd,还有很大的可能会不小心格错盘,导致追悔莫及,所以利用虚拟机安装方便快捷,VMware的安装教程自行寻找,用VirtualBox的也可以参考

## 1.安装ArchLinux
新建虚拟机,然后选择ArchLinux安装镜像启动,插上U盘,将U盘挂载到虚拟机中

### 1.1分区

```
lsblk
```
查看U盘是否挂载上去,不出意外的话,应该会是出现在`/dev/sdb`检查容量大小是否一致(以下就认为U盘挂载在`/dev/sdb`)

[fdisk](https://tldp.org/HOWTO/Partition/fdisk_partitioning.html) — Linux 自带的命令行分区工具

```
fdisk /dev/sdb
#建立boot区

–>n p 4096 +512M

#建立根目录/区

–> n p enter enter

#查看分区表

–> p

#将分区表写入U盘

–> w
```
cfdisk — 使用 ncurses 库编写的具有伪图形界面的命令行分区工具
https://www.kernel.org/
警告: cfdisk 创建的第一个分区起始位置是63扇区而不是通常的2048扇区这将会在 SSD 和使用高级格式化(4k 扇区)的设备上造成性能问题
GRUB2 也会受影响GRUB legacy 和 Syslinux不受影响
gdisk — GPT 版的 fdisk
http://www.rodsbooks.com/gdisk/ 
cgdisk — GPT 版的 cfdisk
http://www.rodsbooks.com/gdisk/ 
sgdisk — Scriptable version of gdisk.
http://www.rodsbooks.com/gdisk/sgdisk-walkthrough.html
GNU Parted — 命令行分区工具
http://www.gnu.org/software/parted/parted.html 
GParted — GTK 图形界面的分区工具
http://gparted.sourceforge.net/
Partitionmanager — QT 图形界面的分区工具
http://sourceforge.net/projects/partitionman/
QtParted — 与 Partitionmanager 类似,在 AUR 中可获取
http://qtparted.sourceforge.net/

### 1.2格式化U盘建立文件系统

```
mkfs.ext4 /dev/sdb1
以ext4格式,格式化sdb1


mkfs.ext4 /dev/sdb2
以ext4格式,格式化sdb2
```
### 1.3联网

1. 首先ping baidu.com如果能ping通说明已经联网了,就跳过这一部分
2. ping 8.8.8.8或者ping 114.114.114.114能ping通说明Dns解析服务器有问题编辑这个文件`/etc/resolv.conf`
3. 以上两步都不行的话,先查服务

```
systemctl --type=service
```
看看有没有fail的项,如果有就需要重启这项服务
比如

```
systemctl stop dhcpcd@enp2s0.service
```
先停止服务再重启

```
systemctl restart dhcpcd@enp2s0.service
```
这里失败是会有日志的,你可以
4. 一般的话这样就可以了,如果还不行,请换网络配置(例如虚拟机桥接网络的,改成NAT),一般虚拟机模拟出来的网卡之类的,不会有这些问题

### 1.4驱动

接下来说说驱动的事,ArchLinux开机会启动udev来检测驱动等,并在启动时自动载入必要的模块

```
lspci
```
可以查看所有pci借口上的硬件

```
lspci |grep Ethernet
```
会输出类似
```
02:00.0 Ethernet controller: Attansic Technology Corp. L1 Gigabit Ethernet Adapter (rev b0)
```
```
lspci -s 02:00.0 -v
```
查看02:00.0总线上的详细信息
```
02:00.0 Ethernet controller: Attansic Technology Corp. L1 Gigabit Ethernet Adapter (rev b0)
…
Kernel driver in use: atl1
Kernel modules: atl1
```
```
dmesg | grep module_name
```
会告诉你是否驱动加载成功
```
$ dmesg |grep atl1
…
atl1 0000:02:00.0: eth0 link is up 100 Mbps full duplex
```
没有驱动当然要装驱动啦,你问我怎么装？我很严肃的告诉你！要联网！这就是为啥用虚拟机装的事了,虚拟机一般虚拟出来的硬件,在ArchLinux的基本驱动都能通用,你要是你的电脑的网卡正好是Atheros AR8161 Gigabit和我一样,恭喜你,你联网一会儿就会断,根本不能好好玩耍,我到现在都没能整明白为啥,但是wiki上这个网卡有收录,告诉你要加载alx模块,可是我编译不通过,这个以后解决这个问题,暂时先跳过

#### 1.4.1网络部分的一些小命令
```
ip addr 查看现有网络连接及ip地址(这里查出来的会有个自动命名的名字比如enp2s0下面的***就应该用这个名字代替)
ip link 查看现有网络连接
ip link set *** up 设置***连接网络
ip link set *** down 设置***断网
dhcpcd *** 使***启动dhcp服务
ls /sys/class/net 查看现有网络接口名字
dhcpcd -k 释放dhcp获得的IP地址
systemctl status dhcpcd@enp2s0.service 查看enp2s0的dhcp服务状态
systemctl --type=service 查看系统已启动的服务
```
目前就学了这些,欢迎补充

### 1.5挂载分区

```
mount /dev/sdb2 /mnt
```
(挂载sdb2 到 /mnt)

```
mkdir -p /mnt/boot
```
(在mnt目录下创建boot目录)

```
mount /dev/sdb1 /mnt/boot
```
(挂载sdb1 到 /mnt/boot)

如果有建立了/home等目录的同理

如果想要用swap 请参考[wiki–swap](https://wiki.archlinux.org/index.php/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

## 2.安装

在确认联网的情况下,就可以安装了,不过在真正安装前,选择快速的镜像源很重要,国内163和清华的比较快

```
nano /etc/pacman.d/mirrorlist
```
按ctrl+K删除整行,去掉别的,就留163的和清华的即可

```
pacstrap -i /mnt base base-devel
```
安装系统,默认全装

> –>enter –>enter –>Y

等待安装完成

```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
生成fstab

设置新系统

```
arch-chroot /mnt
```
转入新系统中,这样就在我们新安装好的系统中进行操作了,可以看到root@archiso变为sh4.3

```
nano /etc/locale.gen
```
选择文字编码,找到en_US.UTF-8 UTF-8和zh开头的都打开注释

```
locale-gen
```
更新文字编码系统
在 `/etc/locale.conf` 里设置系统locale偏好；单个用户请设置`$HOME/.config/locale.conf`:

```
# echo LANG=zh_CN.UTF8 > /etc/locale.conf

ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
对于国内的基本可以用上海时间

```
hwclock --systohc
```
应该可以同步时间

```
echo **** > /etc/hostname
```
设置主机名****

### 2.1设置U盘启动

这个很重要,为了启动后能首先加载usb

```
nano /etc/mkinitcpio.conf
```
在udev 后面加入usb

> HOOKS=”base udev usb autodetect modconf block filesystems keyboard fsck”

```
mkinitcpio -p linux
```
重新初始化内存盘

### 2.2安装引导程序

```
pacman -S syslinux gptfdisk
```
#### 2.2.1安装syslinux

```
syslinux-install_update –iam
```
#### 2.2.2安装引导项

```
blkid
```
查看UUID,记录sdb2的UUID 目的是为了用UUID 做标识来启动操作系统,否则换了电脑硬盘标签变化就不能启动了

```
nano /boot/syslinux/syslinux.cfg
```
修改
```
# UI menu.c32 注释掉
UI vesamenu.c32 取消注释
```
把LABEL arch下面的root改成


> root=UUID="****************************" rw
完成了这步,在你开机选这个U盘启动后应该就能出现syslinux的引导界面了,如果不行,请自行wiki换grub之类的

### 2.3新建用户
```
passwd 
```
先设置root用户的密码

```
pacman -S zsh
```
### 2.4安装zsh

```
useradd -m -g users -G wheel -s /bin/zsh ****
```
创建一个名为****的用户,并使用zsh作默认shell

```
passwd ****
```
为****设置密码

### 2.5添加root权限
如果后面使用的时候出现不能sudo,提示is not in the sudoers file. This incident will be reported.
以root权限编辑 `/etc/sudoers`文件,在 `root ALL=(ALL) ALL`下面 加上 `[username] ALL=(ALL) ALL`

也可以去掉`#%wheel ALL=(ALL) ALL`这一行前面的#

### 2.6更新系统

```
pacman –Syu
```
更新系统

```
pacman-db-upgrade
```
更新数据库

基本配置就结束了,下面是锦上添花

### 2.7配置WIFI

把无线也写在这里吧,虚拟机中安装一般只会是有线状态,出问题也基本就是dhcp没获取到ip,手动重启一遍dhcp就会好

首先应该安装网卡驱动,ArchLinux在安装base-devel的时候一般都带着了,除非你的没有,那就要查一下资料自己装下了

首先需要安装无线相关的软件

```
pacman -S iw wpa_supplicant wireless_tools dialog networkmanager network-manager-applet

systemctl enable NetworkManager
```
这样就能开机自动启动网络配置。

### 2.8安装声卡服务

```
pacman -S alsa-utils
```
官方说法叫解除静音

### 2.9安装桌面系统

请自行根据喜欢的桌面环境去安装配置,先装X

```
pacman -S xorg xorg-xinit
```
#### 2.9.1安装KDE桌面环境

```
pacman -S plasma
```
这就能把最新的plasma5安装上,同时会找到很多依赖包,其中就包括显卡驱动,NVIDIA的显卡请选择开源的驱动,也就是默认选择,不要作死(不要问我为什么知道T_T)

安装完KDE后,如果你这时候启动X,你会发现怎么没有终端,我们熟悉的终端呢？！对！就是没有,要自己装！

```
pacman -S konsole dolphin
```
#### 2.9.2安装konsole和文件管理器

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```
拷贝X的启动文件到用户目录,针对用户修改

```
nano ~/.xinitrc
```
把exec之前的都注释掉,把exec后面的改为startkde

#### 2.9.3安装GNOME桌面环境

```
sudo pacman -S gnome gnome-extra gdm
```
将gdm设置为开机自启动，这样开机时会自动载入桌面

```
systemctl enable gdm
```
#### 2.9.4美化gnome

20170826-update
```
gnome-tweak-tool
```
如果你安装了gnome-extra，那么这个工具已经被安装了，否则的话

```
sudo pacman -S gnome-tweak-tool
```
##### 2.9.4.1图标包

这里我使用的numix-circle图标包，这个图标包在aur里，直接用yaourt即可

```
yaourt -S numix-circle-icon-theme-git
```
然后在gnome-tweak-tool里启用主题

##### 2.9.4.2主题

我们可以到下面这个网站上选自己喜欢的主体，安装方式也可以根据提示步骤来

https://www.gnome-look.org/

然后在gnome-tweak-tool里启用

##### 2.9.4.3gdm背景

输入以下指令

```
curl -L -O http://archibold.io/sh/archibold
chmod +x archibold
./archibold login-backgroung 你的背景的地址
```
重启后gdm就会变成你要的背景

##### 2.9.4.4gnome-shell拓展

shell拓展请进入https://extensions.gnome.org/自行按照说明安装

##### 2.9.4.5screenfetch

screenfetch可以在终端里输出你的系统logo和状态。

```
sudo pacman -S screenfetch
```
要让screenfetch在打开终端是自动输出，在`~/.zshrc`里加入

最后
`exit`然后`umount -R /mnt`关掉虚拟机,拔下U盘,去嗨吧

20160423-update

##### 2.9.4.6安装必要字体

```
pacman -S ttf-dejavu wqy-zenhei wqy-microhei
```
##### 2.9.4.7安装火狐

```
pacman -S firefox
```
##### 2.9.4.8安装net-tools（包含ifconfig）

```
pacman -S net-tools
```
修改`/boot/syslinux/syslinux.cfg`

> timeout = 0
注销所有菜单选项

##### 2.9.4.9启动sddm

```
systemctl enable sddm.service
```
##### 2.9.4.10安装wget，git

```
pacamn -S wget git
```
##### 2.9.4.11安装oh-my-zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
##### 2.9.4.12安装Yaourt

```
git clone https://aur.archlinux.org/package-query.git
cd package-query
makepkg -si
cd ..
git clone https://aur.archlinux.org/yaourt.git
cd yaourt
makepkg -si
cd ..
```

更新yaourt
```
yaourt -Syu --devel --aur
```
##### 2.9.4.13安装搜狗中文输入法

```
yaourt -S fcitx-sogoupinyin
```
由于设置的输出是UTF-8，所以终端中不做设置可能会显示为乱码，我们设置下nano ~/.zshrc
```
export LC_ALL=en_US.UTF-8  
export LANG=en_US.UTF-8
```
20170826-update

##### 2.9.4.14安装chrome

```
yaourt -S google-chrome
```
##### 2.9.4.15解压软件

需要图形化的解压软件可以这样：

```
sudo pacman -S p7zip file-roller unrar
```
##### 2.9.4.16文件系统支持

要支持制作fat文件系统，安装dosfstools，默认内核只能读取ntfs，要支持ntfs读写，安装ntfs-3g。
```
sudo pacman -S ntfs-3g dosfstools
```
##### 2.9.4.17无线AP

需要安装create-ap才能使用gnome3设置里的创建热点选项
```
sudo pacman -S create_ap
```