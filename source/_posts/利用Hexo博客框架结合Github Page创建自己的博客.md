---
title: 利用Hexo博客框架结合Github Page创建自己的博客
categories: Blog  
tags: 
    - Hexo
    - GitHub page 
date: 2024-06-14 17:00
cover: https://s2.loli.net/2023/10/09/WEFNigBfGyDOzdb.png
update: 2024-06-24 10:02
---


## 前言

其实我早在2020年就已经用Hexo搭建了自己的博客，并且使用github page发布了静态的博客。但是中间因为各种原因，差不多就算停更了。现在就是自己再回头记录下，不是说做任何事任何时候开始都不晚么~

so，那就来记录下吧。

首先，我这边的目的是记录下Hexo建立博客的过程，之前我都是在windows下弄的，现在想想还是用个虚拟机，系统就Kali 23.2了,主要后面也要常用的。Hexo也更新了好多代了。还记得当时18，19年的时候Hexo1升级到Hexo2，好多组件不兼容，改js改到半夜，也是头秃。不知道现在的Hexo对老版本是否支持，正好也再来把之前做的Hexo主图更新下。而这篇博客就记录下这个过程。

那就让我们开始把。

## 安装Hexo

### 什么是 Hexo？

Hexo 是一个快速、简洁且高效的博客框架。[Hexo](https://hexo.io/zh-cn/) 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

安装的这部分我也是参考[官方Doc](https://hexo.io/zh-cn/docs/), 不愿意看我废话的，可以自行参考官方Doc。

### 系统环境与Hexo依赖软件(删除kali部分，后续还是windows系统更新博客)

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

我们直接到node.js的官网上下载最新版本的安装包（[Download | Node.js (nodejs.org)](https://nodejs.org/en/download/current)）

根据自己的系统选择对应的32或64bit版本，然后双击运行，一路‘下一步’就完成了，中间建议勾选安装node.js依赖。

安装完成后可以在开始菜单中看到
![微信截图_20231008133554](https://s2.loli.net/2023/10/08/MeQwHWL2yRasS9F.jpg)

点击打开Node.js会弹出命令行，确认Node.js是否安装成功，且版本是多少

![微信截图_20231008135122](https://s2.loli.net/2023/10/08/1UJ2QLdNXwGIjer.jpg)

#### 安装git

到官网下载git程序包[Git - Downloads (git-scm.com)](https://git-scm.com/downloads)

同样也是一路‘下一步’来安装，建议添加系统环境变量和安装git bash。

### 安装Hexo

```shell
sudo npm install -g hexo-cli
```

```shell
hexo version
```

可以使用git bash或者powershell，git bash有右键菜单，可以省的切换目录，我个人觉得比较舒服，所以后面就以这个为例了。

```shell
npm install hexo
```

我这边因为之前在kali中进行部署发现hexo版本通过默认命令安装的不是最新的，因此，我就加上了版本号

```shell
npm install hexo@6.3.0
```
2024-06-24
```bash
npm install hexo@laster
```
---

如果担心后面组件不全，可以直接用官方的

```shell
npm install -g hexo-cli
```

然后确认下版本号即可

```shell
hexo version
```

如果你是刚装好node.js，git bash中似乎没有办法找到这个hexo的环境变量，使用下面的命令即可

```shell
npx hexo version
```

2024-06-23
这次安装发现版本查看不出来，我最后是到`C:\Users\<your name>\node_modules\hexo`下`package.json`中确认的
```
{
  "name": "hexo",
  "version": "7.2.0",
  "description": "A fast, simple & powerful blog framework, powered by Node.js.",
  "main": "dist/hexo",
  "bin": {
    "hexo": "./bin/hexo"
  },
```

然后使用
```shell
npm install hexo
```
现在可以安装到最新的hexo

---


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

我现在找到了一个新的主题，很喜欢，[Hexo-Theme-Async](https://async-docs.imalun.com/)推荐给大家，我就在这个主题基础上添加了一些个性化修改。

~~后期希望可以进一步为这个主题做出一些贡献：~~
2024-06-24
贡献不了一点，作者更新速度也好，修改都很快也很好，算了，我这个web外行就不要瞎掺和了。

---

#### 安装该主题依赖

```shell
npm install --save hexo-renderer-less hexo-renderer-ejs hexo-wordcount hexo-generator-feed katex hexo-generator-searchdb swup hexo-generator-category hexo-generator-tag hexo-generator-index
```

这边我因为几乎把所有功能都开了，因此安装的依赖比较多。**less和ejs是必须的渲染器。**

#### 安装该主题

```shell
npm i hexo-theme-async@latest
```
#### 启用主题
修改 Hexo 站点配置文件 `_config.yml`。
```yml
# 将主题设置为 hexo-theme-async 
theme: async
```
建议安装最新的版本，并且如果不是有无法满足的自定义个性化修改的话，建议就直接只使用原版，并使用该安装版，而**不要去自己clone下来放到`/themes`目录下**。***折腾是由成本的，别问我是怎么知道的😂***

如果你硬要改或者就是要clone，那么请看[原因](# github page部署后，主站仅显示背景)

2024-06-24
可以说目前作者提供的自定义以及个性化修改已经非常足够了，可以参照作者大大写好的文档[主题配置 | Hexo-Theme-Async](https://hexo-theme-async.imalun.com/guide/config.html)

---
### 博客文章

将之前备份的文章放入`source/_post`目录。

*啊~~，文章日期，封面又不匹配了，再改一遍😟*

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

## ~~生成~~
2024-06-24
在现在使用github pages来生成我们的博客网站的情况下，已经不需要再使用静态生成了。

---

## 部署

现在的hexo可以使用workflow中的action直接让github自动生成github pages来部署我们的博客，但是这种方式需要把源代码都提交上去，请自行斟酌是否包含**个人敏感信息**。

我这里就选择使用action来自动化部署，这样我自己博客的内容也完全托管在github上省的以后再找版本了。

在你自己的特殊储存库（`<yourname.github.io>`）中建立`.github/workflows/pages.yml`，内容：

```yaml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: "16"
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```



这里就需要我们之前记录下来的npm的版本号(我这边是18)，并修改到Hexo官方的page生成的action中（也就是上面的pages.yml）：

```diff
-     - name: Use Node.js 16.x
+     - name: Use Node.js 18.x
        uses: actions/setup-node@v2
        with:
-         node-version: "16"
+         node-version: "18"
```



参考官方的doc：[关于 GitHub Pages - GitHub 文档](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages)

### 老主题hexagon的问题

#### 首先是hexo版本与node.js不匹配

```shell
npm install hexo@指定版本
```

我们可以参照官方网站来做[hexo - npm (npmjs.com)](https://www.npmjs.com/package/hexo/v/6.3.0)

这个问题我这边参考了这篇[hexo博客网站主页空白或404](https://zhuanlan.zhihu.com/p/491881992)这里面有说如何降级node.js或者升级Hexo。

#### hexo官方给的pages的actions里面用的主分支叫main，我这边master

这个虽然在Hexo的文档中有说，老的仓库是主分支叫master，但是很容易漏掉，总之我们得确认下。

```diff
on:
  push:
    branches:
-     - main
+     - master
```

经历了这些后，我的老主题总算是可以显示了，但是发现代码以及markdown的大部分格式都不对，该主题也年久失修的状态了，还是换个主题吧。

---

### 新主题async的部署问题

2024-06-24
如果上面是按照`npm i hexo-theme-async@latest`方式使用主题的，那大概率是不会遇到这样的问题的。我这边的做法是先用hexo新版本新建一个本地目录，在本地目录中使用新主题，然后复制`_config.async.yml`过来，使用`hexo server`检查没有问题后，复制到对应`xxx.github.io`的仓库下，然后在这个仓库下用`hexo server`检查渲染等没有问题，然后`commit` 再`push`到远端，等待github pages的action动作完成，检查网页渲染。

---
#### github page部署后，主站仅显示背景

主要就这一个问题，我部署后去主站一看，怎么之后背景，以为是index.html没有内容，但是去gh-pages分支一瞅，这不都在么。就非常奇怪，然后跟着主题doc来来回回走了十多遍，都没有修复。然后去讨论区翻帖子，发现没有人有我这个问题，奇怪了还能成个例了？主要我自己`hexo server`在本地打开时，是好的啊，没有一点问题，不死心的我再去主站上打开开发者模式对比着看，最终发现是有个网页元素是

```html
<div id="trm-scroll-container" class="trm-scroll-container" style="opacity: 0">
```

这里的`opacity: 0`然后对应的css的样式`source/css/_components/base.less`中

```css
.trm-scroll-container {
    transition: opacity .6s;
}
```

意思是打开网页后0.6s时间内在这个容器中的内容从全透明变成不透明，emmmmm，但是我这边没有执行啊，啥情况？一阵懵逼之后直接在网页上修改元素

```diff
- <div id="trm-scroll-container" class="trm-scroll-container" style="opacity: 0">
+ <div id="trm-scroll-container" class="trm-scroll-container" style="opacity: 1">
```

网页立马有显示了，但是显示的不全，说明是这个问题，再次尝试了好几次之后还是不行，然后去作者主题讨论区去提了个问题（丢人😳）

一边等作者大大回复，一边写这个过程，结果突然注意到作者大大在doc中有说

> - 通过克隆本仓库安装（不推荐）
>
>> **DANGER**
>> 不推荐直接使用这种方式安装，会导致 bug 版本定位和后续升级比较麻烦。如果您需要自定义样式和页面模块，可以优先使用 [自定义样式](https://async-docs.imalun.com/guide/config.html#自定义样式-style) 和 [自定义模板](https://async-docs.imalun.com/guide/config.html#自定义模板-layout) 配置来个性话您的博客，如果以上方式无法满足您的需求时，且不在需要升级时可选择通过这种方式安装。
>
> <details class="details custom-block" open="" style="box-sizing: border-box; border-width: 1px; border-style: solid; border-color: var(--vp-custom-block-details-border); border-image: initial; border-radius: 8px; padding: 16px 16px 8px; line-height: 24px; font-size: var(--vp-custom-block-font-size); color: var(--vp-custom-block-details-text); background-color: var(--vp-custom-block-details-bg); margin: 16px 0px;"><summary style="box-sizing: border-box; touch-action: manipulation; margin: 0px 0px 8px; font-weight: 700; cursor: pointer;">v2.0.0 后版本</summary><p style="box-sizing: border-box; margin: 8px 0px; overflow-wrap: break-word; line-height: 24px;">从 v2.0.0 开始不在支持拉取后直接使用。新版本的脚本使用 TypeScript 进行重构，项目中不在提供打包压缩后的脚本。</p><p style="box-sizing: border-box; margin: 8px 0px; overflow-wrap: break-word; line-height: 24px;">如果您只想修改模板，您可以前往<span>&nbsp;</span><a href="https://github.com/MaLuns/hexo-theme-async/releases" target="_blank" rel="noreferrer" style="box-sizing: border-box; touch-action: manipulation; color: var(--vp-c-brand-1); text-decoration: underline; font-weight: 500; text-underline-offset: 2px; transition: color 0.25s ease 0s, opacity 0.25s ease 0s;">Github Releases</a><span>&nbsp;</span>的 Assets 下载打包文件<span>&nbsp;</span><code style="box-sizing: border-box; font-family: var(--vp-font-family-mono); font-size: var(--vp-code-font-size); color: var(--vp-code-color); border-radius: 4px; padding: 3px 6px; background-color: var(--vp-custom-block-details-code-bg); transition: color 0.25s ease 0s, background-color 0.5s ease 0s;">hexo-theme-async</code><span>&nbsp;</span>。</p><p style="box-sizing: border-box; margin: 8px 0px; overflow-wrap: break-word; line-height: 24px;">如果您仍然想要使用该方式，请 clone 项目后，手动执行 yarn &amp;&amp; yarn run lib:build 以构建压缩后的脚本。</p></details>

这不尴尬了，这么一找果然，作者大大在主题目录下添加了`.gitignore`

```
# dev esbuild
source/js/
source/plugins/
```

那么在clone下来的主题中执行

```shell
yarn && yarn run lib:build
```

把编译生成的js拷贝到自己的仓库对应目录下

```shell
IIFE ..\hexo-theme-async\source\js\plugins\local_search.js 2.03 KB
IIFE ..\hexo-theme-async\source\js\main.js                 18.51 KB
IIFE ..\hexo-theme-async\source\js\plugins\typing.js       583.00 B
IIFE ⚡️ Build success in 14ms
Done in 2.05s.
```

删除.gitignore文件后，重新上传部署，终于网站正常打开了。👏👏👏


