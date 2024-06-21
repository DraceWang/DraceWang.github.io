---
title: 用Obsidian记录学习Obsidian
categories: Learn
tags:
  - note
  - blog
  - obsidian
date: 2024-06-07 13:29
cover: https://s2.loli.net/2024/06/11/So9CcKAa5djMvQz.png
update: 2024-06-17 16:49
---
## 需求以及笔记软件的比较
首先markdown语法真的好用，我也是typora的付费用户，typora用来写一些文档确实不错，渲染效果各方面都好，就是不适合作为一个笔记软件，或者说，我其实更希望能有一个all in one的软件来对我自己的日常事物进行管理，类似于第二大脑，我也浅浅的用了下notion，很方便，但是没有类似于滴答清单的提醒功能，而滴答清单的日历视图是收费，所以我就找到了obsidian，其实我到写这篇随笔或者说学习笔记的时候都不确定obsidian是否可以达到我的期望，但是我决定试一下。从网上看到很多obsidian的视频以及微信公众号中很多介绍，至少乍一下，很符合我程序猿的习惯，似乎obsidian很多强大的功能都是通过插件来完成的，那我就一步步开始记录自己的折腾之路。

那么接下来先让我来整理下notion和obsidian之间的差异，感觉很有可能如果obsidian不能满足我期望的时候，两者公用可能是我最好的解决方案。

|              | <font color="#8064a2">**notion**</font> | <font color="#8064a2">**obsidian**</font> |
| :----------: | :--------: | :---------------------------------------: |
|    想法记录方式    |  有序，有逻辑的   |                  无序的，混沌的                  |
|  原生是否有提醒功能   |     无      |                     无                     |
| 是否支持markdown |     支持     |                    支持                     |
|   是否支持多端同步   | 免费用户也支持同步  |               本地文件，同步取决于自己                |
|   是否支持日历视图   |     支持     |                  通过插件实现                   |
|   是否支持知识图谱   |    不支持     |                    支持                     |
|      发布      |    官方自带    |               官方付费，可以联动hexo               |

暂时想到这些，后续想到再加吧。总的来看，notion更加的完善是一个相对完善完整的产品，而obsidian就好像还在进化，同时因为其开源以及三方插件（管中窥豹）的开放和自由，因此似乎其更能完成复杂需求，也就是潜力高。后面看看我能给它整成啥样吧。

ok，来整理下我的需求：
 + 日历视图
 + 手机端，电脑端多端多地同步
 + 可以通过系统（pc和ios）提醒代办事项
 + 有知识链接图谱
 + 可以联动hexo发布博客

## obsidian学习记录
先把视频教学看起来，我选择的是PKMer在B站发布的视频，这边就链接他们这个【从零开始学OB】系列的第一个视频，感谢作者的辛苦劳作，如果涉及了版权问题请联系我。
<iframe src="http://player.bilibili.com/player.html?isOutside=true&aid=574356314&bvid=BV19z4y1s7nk&cid=1223043822&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="800" height="600"></iframe>

观看视频记录的一些想法：
- tag似乎不错，不过不能有空格
- 可以和hexo联动发布博客，看着不错，回头折腾下 
	 - [x] 和hexo联动发布博客 📅 2024-06-18 ✅ 2024-06-17
- obsidian的链接功能确实强大`#`对应标题，`^`对应文本块，`|`可以指定别名
- 日记，时间戳笔记真不错，其实应该可以增加 
	 - [x] 时间戳文本块，方便记录 [[#^e6b1fc]] 📅 2024-06-18
+ 白板类似思维导图好功能，可以增加笔记，那就是可以多联，应该有点东西

视频看到这里，突然觉得上面两个可以作为代办事项，于是用了下[Reminder]([PKMer_Obsidian 插件：Reminder 为待办任务增加提醒](https://pkmer.cn/Pkmer-Docs/10-obsidian/obsidian%E7%A4%BE%E5%8C%BA%E6%8F%92%E4%BB%B6/obsidian-reminder-plugin/))插件，可以方便的添加一个任务提示时间点，很不错，还可以用系统提示，nice~

2024-06-11
OB中可以和hexo一样，通过YAML增加文章属性，并且还渲染的很好，就做了一个博客的模板来方便后面通过hexo发布到自己的博客上，于是安装了picGo，以及安装上自动上传的obsidian插件。
同时为了方便记录当前时间下载了Natural Language Dates插件，~~通过`@`来快速插入时间。~~ ==事实证明还是用模板自带的时间更好(我设置了快捷键`alt+D`插入日期，还不错)==，因为自然语言插件插入的时间会自动生成一个双链，虽然不直接创建这个文件，但是关系图谱中有了。
下载日历插件显示日期以及周数，可以快速点击生成笔记。 ^e6b1fc

2024-06-12
安装了excalidraw和typewritescoll插件
安装了fileExpolercount和various-complements插件

2024-06-13
安装了remotely save插件，想着iphone手机上通过群晖的webdav来实现远程同步，既能保障本地化以及实时备份同步，又能有不输云服务的同步性能。
	在这一趴中遇到了几个小问题，先是ios11之后就禁止app使用http不安全的api，只能使用https，虽然群晖套件中的webdav可以开https，但是又有个新的问题就是ssl证书的问题，群晖自己的证书不是公有的，并不授信，所以，我先用阿里云申请了一个3个月的免费ssl证书测试了下，测试成功。同步的时候还需要设置成相同的根目录；好吧，总的来说，还是成功了。
那么 
- [x] 测试一下reminder是否可以在iphone上调用系统提醒功能 📅 2024-06-14
✅ 2024-06-14 需要手机开着obsidian，似乎只能在obsidian中提示没有系统提醒。

安装了kanban和emoji表情👍
安装了auto link title和editing toolbar
floating Toc因为官方有了大纲，暂时就不装了
安装了callout manager
wow~，[PKMer\_挂件集市](https://pkmer.cn/products/widget/widgetMarket/)这个可以哦🤩

2024-06-14
pkmer的教程看的差不多了，暂时就通过下载插件来完善obsidian功能，首先是日历视图，下载了一堆插件：
- full calendar ❌
- tasks ✔️
- day planner ❌
- task calendar wrap ✔️
- Time Ruler ✔️
- timeline view 🔰
- big calender ❌

安装了full calendar，拥有了日历视图。但是任务管理系统不够完善，下载tasks和day planner插件尝试关联。
full calendar必须建立一个笔记用它规定的元数据也就是属性来做，导致tasks插件以及其他插件是找不到这个任务的。暂时放弃❌
tasks可以直接找到带有`#task`标签的任务或者是markdown中todo列表格式的作为任务，可以在笔记中利用模板也能在看板中添加卡片，通过dataview可以显示出日历视图（借用task calendar wrap中整合的task calendar和task timeline）
day planner是基于日记规划的，有周视图但是更侧重这一天的规划，且该插件启动加载有问题❌
Time Ruler也可以显示出每天的task，也有每天的时间线
timeline view是以点的形式显示，是建立一个exclidraw来显示的，可以缩放
big calender也是需要使用daily note 来建立日历视图的，所以暂时pass❌

测试之后，总结出自己的一套日程管理使用方式：
1. 主要使用task来建立任务池，将其添加到看板卡片中。看板的卡片并不会生成一个笔记，所以我觉得还行，我并不是很喜欢多出很多笔记来做日程管理。
2. 使用笔记文章中使用todo格式（好像也能调用快捷键创建task模板来添加）来直接设置代办事项，很适合学习或者做事的时候突然想到要做个什么事情。

来看下联动hexo发布博客
[基于obsidian插件、hexo和github action的博客方案 - 经验分享 - Obsidian 中文论坛](https://forum-zh.obsidian.md/t/topic/32180)
这篇文章主要在说本地部署预览然后上传的方式，去年我折腾的时候hexo已经改用action来生成github page了，所以没多大用。
[Hexo + Obsidian + Git 完美的博客部署与编辑方案](https://segmentfault.com/a/1190000042111566)
这篇文章相对我来说更有价值一些，利用git插件上传github仓库，然后action会自动生成，似乎更合适，那么开始实践。
把这篇文章上传到github仓库，并显示到自己的博客上。试试看~
*这个git只是能在obsidian中调用git，并进行一个显示，主体还是需要用git来操作的，怎么说能便捷化一些。*

那我可能有两条路子走：
~~1.直接在obsidian仓库下面使用git来做远程仓库，然后写好的文章直接push到github上~~❌
2.借用文件同步软件，把写好的文章放到obsidian中指定的文件夹，然后单向同步到github在本机电脑中clone下来的仓库，然后用git push到远端github库上。
两种方式都试试吧。
git部分操作参考了[Obsidian +Obsidian Git插件 + Gitee 自动同步笔记-CSDN博客](https://blog.csdn.net/weixin_55688821/article/details/133849556)

2024-06-16
今天安装了file tree alternative和iconize插件，前者可以聚焦文件夹，后者可以添加图标，美化一下，明天操作下。

2024-06-17
今天把File tree设置下，设置完成之后确实就影响不大了，到时候直接复制到`_post`文件夹下，git push就好。
然后下载了几个图标，设置了下，但是貌似file tree不能显示自定义图标，有点尴尬。

对了，因为每次打开有视频链接的文章都会自动播放，有点烦人，试了下网上说的修改`<iframe>`标签为`<vedio>`标签，实测没有效果，增加`sandbox=""`也不行。最后算了，世上无难事，只要肯放弃~😜

两条同步路子的第一条，果然会把各种其他修改全都给过commit上去，算了，我还是用文件同步吧。
下载安装freefilesync，设置好单项同步，设置好路径，它会自动比较差异，点击更新就可以了。完成后可以保存成批处理命令，然后用实时监控方式，这也很美妙了。

## 总结
现在来回顾下，我自己的需求：

| <center>**需求**</center>               | <center>**实现方式**</center>       |
| ------------------------------------- | ------------------------------- |
| <center>日历视图</center>                 | tasks+task calendar wrap+kanban |
| <center>手机端，电脑端多端多地同步</center>        | remotely save中的webdav           |
| <center>可以通过系统（pc和ios）提醒代办事项</center> | reminder插件                      |
| <center>有知识链接图谱</center>              | obsidian自带的双链                   |
| <center>可以联动hexo发布博客</center>         | git插件上传github仓库                 |

ok，obsidian可以满足我的需求，而obsidian还在不断进化，类似dataview功能还可以用代码方式自定义更高程度的展示页面，目前这篇学习记录就算完成了，后面再慢慢进步改善吧。