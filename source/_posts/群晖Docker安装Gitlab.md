---
title: 群晖Docker安装Gitlab
categories: Synology
tags:
  - Synology
  - Docker
  - Gitlab
date: 2024-06-23 11:00
cover: https://s2.loli.net/2024/06/25/xoktsdzuDGMFHL8.jpg
update: 2024-06-25 08:09
---
# 群晖Docker安装Gitlab



其实也没有什么特别的想法，就是突然觉得要是自己有个像gitlab或者github的私有服务还蛮酷的，于是，就有了这次的折腾。

首先声明这篇文章借鉴了[涂启标](https://www.zhihu.com/people/geescan)的[Nas码农篇：群晖Docker安装Gitlab](https://zhuanlan.zhihu.com/p/109834567)如果对原作者造成了侵权或者困扰，请联系我，我会第一时间删除，并且致以我最真诚的歉意。

同时由于安装完成后找不到管理员用户的密码，最后在官方doc上找到了，因此这里也把官方doc地址贴上[GitLab Docker images](https://docs.gitlab.com/ee/install/docker.html)

那么话不多说，就来说说我自己的折腾经历。

## 1.安装gitlab的前提

先说下我自己的群晖型号DS918+，扩展4G内存，使用两个nvme接口的ssd做了缓存，4块8T的硬盘组了raid10，系统目前已经升级了DSM7.0。

群晖上安装gitlab有两种方式：

+ 一种是直接安装套件中心的gitlab。***（这种方式我在DSM7.0的套件中心中没有找到）***
+ 另一种是在docker中自定义安装。

因为DSM7.0的套件中心中找不到gitlab，因此我们选择后者据说可以安装最新版本的gitlab。

注意前提条件：

+ 群晖必须是plus型号，这样才能支持docker。（我的是DS918+，所以没有问题）

+ 已经安装Docker套件。（没有安装的就自己在套件中心中搜索docker，安装下就可以了）

+ Gitlab官方推荐内存4G及以上，这里建议8G以上，因为gitlab很容易就会占用掉4g左右的内存。（我安装完成后确实稳定在4.72G左右）

## 2.安装gitlab

官方的都是在一个linux完整系统下的docker操作，基本都是命令行，我们这里由于是群晖的DSM系统下的docker，因此大部分都是图形界面，与官方还是有些不同的。但是我这里的方法是21年10月操作的，亲测可以正常安装运行，如果后期有问题请先查询官方doc，同时欢迎留言评论。

1.在套件中先安装Docker套件。

2.在docker中，注册页签下，搜索：gitlab，双击下载，选latest

3.下载完成后，在映像页签下，双击gitlab-ce镜像进行安装。

*3.3 【可选】设置环境变量（官方推荐，但是我实际没有操作）*

*Before setting everything else, configure a new environment variable `$GITLAB_HOME` pointing to the directory where the configuration, logs, and data files will reside. Ensure that the directory exists and appropriate permission have been granted.*

*For Linux users, set the path to `/srv/gitlab`:*

```
export GITLAB_HOME=/srv/gitlab
```

4.点击高级设置，在弹出的高级选项中，切换到卷页签，按照下面截图，设置目录。

这里需要添加对应的文件夹到docker目录下，可以使用filestation在docker目录下，创建gitlab目录，然后在gitlab目录下，分别创建logs，config，data来存储日志、配置和数据信息文件。装载路径手动填写。

| Local location        | Container location | Usage                                       |
| --------------------- | ------------------ | ------------------------------------------- |
| `$GITLAB_HOME/data`   | `/var/opt/gitlab`  | For storing application data.               |
| `$GITLAB_HOME/logs`   | `/var/log/gitlab`  | For storing logs.                           |
| `$GITLAB_HOME/config` | `/etc/gitlab`      | For storing the GitLab configuration files. |

5.切换端口设置页签，设置一个本地端口，这里指定80容器端口对应本地端口1080。当然也建议将其他本地端口的[自动]改为指定的端口，比如22端口对应的本地端口改为7022之类的，因为后续还要修改配置文件，让克隆地址可以正常显示端口，同时也避免自动获取而带来端口变化而导致的访问问题。

![image-20211019170521317](https://i.loli.net/2021/10/20/pEnGutOlVqZL48m.png)

这个端口就根据自己的爱好来设置了，注意不要与你已有的服务冲突了就行。比如ssh（22）和web服务（80）。

6.其他的暂时不用改，直接点击应用，并启动这个docker。正常需要等待一段启动时间，内存飙升到一个比较稳定的数值时，正常就可以访问gitlab的页面了。

![image-20211019171048060](https://i.loli.net/2021/10/20/u5GdVZ3nFBc62eg.png)

7.浏览器输入nas的ip地址+刚才配置的本地端口号，比如192.168.100.110:8080，这样来访问gitlab，如果此时出现502错误，但是看到了gitlab的图标，这表示服务还没起来，可以再等等。

8.首次登录，这里我遇到了一个小坑，直接附上官方doc的信息：

Visit the GitLab URL, and log in with username `root` and the password from the following command:

当我们首次访问了我们自己部署的gitlab的服务地址，就会要求我们登录，这时用户名请使用`root`，密码则使用下面的命令获取：

```
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```
我在尝试使用了这个密码之后发现并不可行，因为我们DSM虽说也是linux系统，但是并不能识别docker，exec命令。
因此，我们需要现在群晖中设置开启ssh访问：

![image-20211019171941845](https://i.loli.net/2021/10/20/TUkWFMa5lAh2vVe.png)

然后再局域网内使用电脑的终端，比如windows的powershell或者cmd都可以通过ssh连接到群晖：

```
ssh 你群晖上的用户名@你自己的群晖的ip -p 你ssh的端口
```
如果询问是否建立连接，请输入yes
然后输入自己群晖用户名对应的密码（你自己要是不是群晖nas的管理员就别折腾了）
通过`sudo -i`命令获取root权限，再次输入密码
然后通过下面的方法获取这个密码文件内容：
```
cat /etc/gitlab/initial_root_password
```

> The password file will be automatically deleted in the first reconfigure run after 24 hours.
>
> 这个初始化的密码会在首次配置启动后的24小时之后删除。

9.设置好root密码后，可以使用root账号登录。一般情况下，这就能正常登录到gitlab后台了。

![image-20211019173058249](https://i.loli.net/2021/10/20/LlqcmxTIAjKn6NX.png)

看到这里就大功告成了，已经可以去玩耍了。