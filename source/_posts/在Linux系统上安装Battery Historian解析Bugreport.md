---
title: 在Linux系统上安装Battery Historian解析Bugreport
categories: Android   
tags: 
  - battery historian 
  - android 
  - bugreport
  - linux
date: 2020-06-12 12:00:00
cover: https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg
---


由于工作需要功耗问题上总是需要使用Battery Historian来解析Bugrepot，这里呢就把这个过程整理下。而根据官网Docker是最方便的方式，但是由于公司网络大多有不少限制，因此我没能用成，只能一步步把运行环境全部搭建起来。

## 安装运行环境

### GO语言

首先到GO的官网上下载最新版本的go安装包，至于linux如何安装也有文档可以查询。

```shell
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```
在mac上使用官网下载的go语言安装包安装后
```shell
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

这里注意要设置`GOPATH`的路径，否则后续编译安装Battery Historian使用到的时候会找不到路径。

### 安装Python2.7

这个一般linux系统都有，如果是python3的话，记得设置切换到2.7。

如果没有呢，可以去https://python.org/downloads官网下载。

### 安装java运行环境

从官网安装 http://www.oracle.com/technetwork/java/javase/downloads/index.html.

## 安装Battery Historian

官网上又是很简单的

```go
go get -d -u github.com/google/battery-historian/...
```
但是非常不幸，我这里还是因为网络的问题无法安装。不要问我为什么github都不能访问。

需要安装三个东西

* [proto](http://github.com/golang/protobuf/proto)

* [closure-compiler](https://github.com/google/closure-compiler)

* [protobuf](https://github.com/golang/protobuf)

安装好之后先运行安装编译：

```go
cd $GOPATH/src/github.com/google/battery-historian
go run setup.go
```
基本应该是没啥问题了，如果还有报错就再看下，是不是少了`historian-optimized.js`文件
下载地址：`链接：pan.baidu.com/s/1kFdUVM6ICT_3Uh1ui14J3w 提取码：3fnn`
下载后放到`\go\src\github.com\google\battery-historian\compiled`目录下

如果是连接了可以上google的网络，则可以不用单独安装，在`go get -d -u github.com/google/battery-historian/...`之后就可以直接
```go
cd $GOPATH/src/github.com/google/battery-historian
go run setup.go
```
但是目前（2020-10-21）`closure-library`有问题，会导致`go run setup.go`报错，解决办法如下：
```go
//go run setup.go (this fails)
cd third_party/closure-library/
git reset --hard v20170409
cd -
go run setup.go //(this passes)
```
## 运行Battery Historian
```go
cd $GOPATH/src/github.com/google/battery-historian
go run cmd/battery-historian/battery-historian.go [--port <default:9999>]
```
这个时候大概率是无法运行的，或者是出现等了好久出现了界面，但是选择了需要解析的文件后没有submit提交按钮。

这个时候需要在`$GOPATH\src\github.com\google\battery-historian\templates\base.html`中替换js文件的加载地址。

可以从[https://www.bootcdn.cn/](https://www.bootcdn.cn/)网站上找到对应需要替换的js文件（尽量版本也可以对应）

替换后打开快了吗？不会！！

浏览器中开发者模式下，你会发现访问了[https://www.google.com/jsapi](https://www.google.com/jsapi)，这个是用来查询js的google的api，但是由于我们已经替换过了，并且国内无法使用google的域名，怎么办呢？嘿嘿，干掉它，注释掉。或者改为cn域名就可以访问了，但是如果你完全做了替换那也就没啥意义了。

附上`base.html`的前半部分有修改的地方。

```html
<html lang="en">
  <head>
    <link rel="stylesheet" href="//cdn.bootcdn.net/ajax/libs/jqueryui/1.11.4/jquery-ui.css">
    <script src="//cdn.bootcdn.net/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
    <script src="//cdn.bootcdn.net/ajax/libs/jqueryui/1.11.2/jquery-ui.min.js"></script>

    <link rel="stylesheet" href="//cdn.bootcdn.net/ajax/libs/select2/3.5.4/select2.css">
    <link rel="stylesheet" href="//cdn.bootcdn.net/ajax/libs/jquery-contextmenu/1.6.6/jquery.contextMenu.css">
    <link rel="stylesheet" href="//cdn.datatables.net/1.10.9/css/jquery.dataTables.css">
    <script src="//cdn.bootcdn.net/ajax/libs/select2/3.5.4/select2.js"></script>
    <script src="//cdn.bootcdn.net/ajax/libs/jquery-contextmenu/1.6.6/jquery.contextMenu.js"></script>
    <script src="//cdn.datatables.net/1.10.9/js/jquery.dataTables.js"></script>
    <script src="//cdn.bootcdn.net/ajax/libs/moment.js/2.13.0/moment.js"></script>
    <script src="//cdn.bootcdn.net/ajax/libs/moment-timezone/0.5.31/moment-timezone-with-data.js"></script>

    <!-- <script type="text/javascript" src="https://www.google.com/jsapi"></script> -->
    <!-- Loading fonts with https://www.gstatic.com/external_hosted/twitter_bootstrap_css/css/bootstrap.min.css is blocked by CORS policy, which means icons won't load. -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>

    <script src="//cdn.bootcdn.net/ajax/libs/flot/0.8.3/jquery.flot.min.js" type="text/javascript"></script>
    <script src="third_party/flot-axislabels/jquery.flot.axislabels.js?ver={{.ResVersion}}" type="text/javascript"></script>
    <script src="//www.benjaminbuffet.com/public/js/jquery.flot.orderBars.js" type="text/javascript"></script>
    <script src="//cdn.bootcdn.net/ajax/libs/flot/0.8.3/jquery.flot.pie.min.js" type="text/javascript"></script>

    <link type="text/css" rel="stylesheet" href="static/stylesheet.css?ver={{.ResVersion}}">
    <link type="text/css" rel="stylesheet" href="static/historian.css?ver={{.ResVersion}}">
    <link type="text/css" rel="stylesheet" href="static/histogram.css?ver={{.ResVersion}}">

    <script src="//www.gstatic.com/external_hosted/jquery_form/jquery.form.min.js" charset="utf-8"></script>

    <script type="text/javascript" src="//cdn.bootcdn.net/ajax/libs/d3/4.9.1/d3.min.js"></script>
    {{ if .IsOptimizedJs }}
    <script src="compiled/historian-optimized.js?ver={{.ResVersion}}"></script>
    {{ else }}
    <!-- Need to load Closure in header. -->
    <script>CLOSURE_NO_DEPS=true</script>
    <script type="text/javascript" src="third_party/closure-library/closure/goog/base.js"></script>
    <script type="text/javascript" src="compiled/historian_deps-runfiles.js"></script>
    {{ end }}

    <title>Battery Historian</title>
  </head>
```

这个时候再运行就应该很快了。

### 点击error按钮不显示解析报错的问题
不知道为啥目前可以解析，但是如果读取一个AndroidQ的bugreport会报错，但是点击error按钮却无法显示报错信息。目前在mac上构建的可以正常显示报错信息。后期如果解决了该问题再更新。

## 解析bugreport

### 抓取Bugreport
由于版本问题不同方式抓取的bugreport会有不同的问题，所以抓取的时候，尽量按照官网的方式抓取，要重置汇总的电池统计信息和历史记录：

```shell
adb shell dumpsys batterystats --reset
```

#### 唤醒锁分析

默认情况下，Android不记录特定于应用程序的时间戳，即使在运行的基础上维护了汇总统计信息，用户空间唤醒锁的转换。 如果要Historian显示有关的详细信息，时间轴上的每个唤醒锁，都应启用完整唤醒锁，开始实验之前，请使用以下命令进行报告：

```
adb shell dumpsys batterystats --enable full-wake-history
```

请注意，通过启用完整的唤醒锁报告，电池历史记录日志将在几个小时后溢出。 使用此选项可以进行短时间的测试（3-4小时）。

#### 内核跟踪日志分析

生成跟踪文件来记录内核唤醒源和内核唤醒锁活动：

首先，将设备连接到台式机/笔记本电脑并启用内核跟踪日志记录：

```shell
$ adb root
$ adb shell

# Set the events to trace.
$ echo "power:wakeup_source_activate" >> /d/tracing/set_event
$ echo "power:wakeup_source_deactivate" >> /d/tracing/set_event

# The default trace size for most devices is 1MB, which is relatively low and might cause the logs to overflow.
# 8MB to 10MB should be a decent size for 5-6 hours of logging.

$ echo 8192 > /d/tracing/buffer_size_kb

$ echo 1 > /d/tracing/tracing_on
```

然后，将该设备用于预期的测试用例。

最后，提取日志：

```shell
$ echo 0 > /d/tracing/tracing_on
$ adb pull /d/tracing/trace <some path>

# Take a bug report at this time.
$ adb bugreport > bugreport.txt
```

Note:在输入完`adb shell dumpsys batterystats --reset`这个命令后bugreport就会清空了之前的信息，并且重新开始记录电量信息，所以在开始测试前就拔掉USB数据线，在测试步骤完成后，重新插上usb数据线，再输入`adb bugreport bugreport.zip`就会生成到你电脑的当前目录。



