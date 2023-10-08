# 利用Hexo博客框架结合Github Page创建自己的博客

## 前言

其实我早在2020年就已经用Hexo搭建了自己的博客，并且使用github page发布了静态的博客。但是中间因为各种原因，差不多就算停更了。现在就是自己再回头记录下，不是说做任何事任何时候开始都不晚么~

so，那就来记录下吧。

首先，我这边的目的是记录下Hexo建立博客的过程，之前我都是在windows下弄的，现在想想还是用个虚拟机，系统就Kali 23.2了,主要后面也要常用的。Hexo也更新了好多代了。还记得当时18，19年的时候Hexo1升级到Hexo2，好多组件不兼容，改js改到半夜，也是头秃。不知道现在的Hexo对老版本是否支持，正好也再来把之前做的Hexo主图更新下。而这篇博客就记录下这个过程。

那就让我们开始把。

## 安装Hexo

### 什么是 Hexo？

Hexo 是一个快速、简洁且高效的博客框架。[Hexo](https://hexo.io/zh-cn/) 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

安装的这部分我也是参考[官方Doc](https://hexo.io/zh-cn/docs/), 不愿意看我废话的，可以自行参考官方Doc。

### 系统环境与Hexo依赖软件

~~由于我是虚拟机部署环境，想着是弄完这一次之后，后面要是长久不用，我也能直接用虚拟机恢复出写博客的环境。（反正想法总是一阵一阵的，就先这样吧）。~~

- [x] ~~Kali Linux 2023.2（有需要的改下国内源）~~
- [ ] ~~[Node.js](http://nodejs.org/) (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)~~
- [x] ~~[Git](http://git-scm.com/)~~



~~这里就不讲虚拟机怎么装Kali系统了，网上太多教程了。~~

这里可以说是非常的蛋疼，我后面其实已经把kali的环境都搭建好了，但是由于GFW的问题，导致无法再Linux下直接通过ssh上传到github，虽然本着有问题解决问题的态度，但是知道是因为代理导致的，那也就没啥好纠结的了，其次是根据Hexo官方现在都已经使用action来构建和部署github page，那原先想着的打包虚拟机整体环境就没有必要了。

因此最终决定，抛弃kali环境直接在windows环境下搭建博客环境。

- [x] windows 10
- [ ] Node.js
- [ ] Git

#### 安装Node.js

~~请根据你自己的系统版本来，我这边kali是基于debian~~

```shell
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt-get install -y nodejs
```

~~测试安装~~

```shell
curl -fsSL https://deb.nodesource.com/test | bash -
```
~~或者~~

```shell
npm -v 
```

我们直接到node.js的官网上下载最新版本的安装包（[Download | Node.js (nodejs.org)](https://nodejs.org/en/download/current)）

根据自己的系统选择对应的32或64bit版本，然后双击运行，一路‘下一步’就完成了，中间建议勾选安装node.js依赖。

安装完成后可以在开始菜单中看到

### 安装Hexo

```shell
sudo npm install -g hexo-cli
```

这里安装程序会创建文件等需要管理员权限。

安装以后，可以使用以下两种方式执行 Hexo：

1. `npx hexo <command>`
2. Linux 用户可以将 Hexo 所在的目录下的 `node_modules` 添加到环境变量之中即可直接使用 `hexo <command>`：

```shell
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
```

然后检查安装

```shell
hexo version
```

## 初始化

使用hexo的框架创建一个文件夹，用于部署和生成静态网站

```shell
hexo init <folder>
cd <folder>
npm install
```

操作完成之后，可以看到这个文件夹下面结构：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

### _config.yml

网站的 [配置](https://hexo.io/zh-cn/docs/configuration) 信息，您可以在此配置大部分的参数。

### package.json

应用程序的信息。[EJS](https://ejs.co/), [Stylus](http://learnboost.github.io/stylus/) 和 [Markdown](http://daringfireball.net/projects/markdown/) 渲染引擎 已默认安装，您可以自由移除。

### scaffolds

[模版](https://hexo.io/zh-cn/docs/writing#模版（Scaffold）) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来创建文件。

Hexo 的模板是指在新建的文章文件中默认填充的内容。例如，如果您修改 `scaffold/post.md` 中的 Front-matter 内容，那么每次新建一篇文章时都会包含这个修改。

### source

资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。

### themes

[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

## 恢复站点状态使用

### _config.yml

由于我之前已经用过了hexo并且已经建立过站点了，因此现在我可以通过之前备份的主题以及配置文件来恢复，主要是将自己使用的主题放进`themes`文件夹，并将`_config.yml`中的参数比对恢复。

***这里建议`_config.yml`中单独恢复把参数对应的值粘贴过来，因为hexo的版本更新，防止出现解析生成静态站的时候出现各种乱七八糟的问题。***

### 主题

我自己建立了一个hexo的主题[hexo-theme-Hexagon](https://github.com/DraceWang/hexo-theme-Hexagon)，是基于[hexo-theme-Claudia](https://haojen.github.io/Claudia-theme-blog/)这个主题修改得来的。因为之前就上传到github了，因此这里直接在`themes`目录下直接

```
git clone https://github.com/DraceWang/hexo-theme-Hexagon.git
```

然后在`_config.yml`中设置主题：

```xaml
theme: hexo-theme-Hexagon
```

### 博客文章

将之前备份的文章放入`source/_post`目录。

*啊~~，文章缺少目录，还缺少之前的格式，哎，下次完整打包备份吧。*

## 通过建立本地web服务进行预览

使用hexo服务器进行本地站的预览，Hexo 3.0 把服务器独立成了个别模块，必须先安装 [hexo-server](https://github.com/hexojs/hexo-server) 才能使用。

### 安装hexo-server

```shell
npm install hexo-server --save
```

安装完成后，输入以下命令以启动服务器，网站会在 `http://localhost:4000` 下启动。在服务器启动期间，Hexo 会监视文件变动并自动更新，无须重启服务器。

```
hexo server
```

## 生成

当我们在本地预览完成之后，可以使用hexo的生成器来生成静态页面。

```
hexo generate
```

生成的静态网站会在`/public`目录下

## 部署

现在的hexo可以使用workflow中的action直接让github自动生成github pages来部署我们的博客，但是这种方式需要把源代码都提交上去，请自行斟酌是否包含**个人敏感信息**。

这边说下我自己遇到的坑，首先是hexo版本与node.js不匹配，其次是hexo官方给的pages的actions里面用的主分支叫main，我这边master

npm install hexo@指定版本

https://zhuanlan.zhihu.com/p/491881992

