# [hexo-theme-Hexagon](https://dracewang.github.io/)
a hexo theme base on [hexo-theme-Claudia](https://haojen.github.io/Claudia-theme-blog/)

这里首先要感谢原主题作者的辛勤劳动，给我们带来一款非常棒的主题，我这个主题主要是基于这个原主题进行的修改。

## 使用

### 使用评论系统
> 在本主题的`_config.yml`配置文件

## 来比力

	use_livere: true
	livere_uid: 你注册的来比力 uid

## Disqus ##

	use_disqus: true
	disqus_url: 你的 Disqus 链接

### 配置博文封面图 ###

#### 方法一：使用MD语法插入图片
将 img 的 `alt` 属性设置为 `post-cover` 即可，如：
	
	![post-cover](https://<your img path>.jpg)
	![post-cover](/<your img path>.jpg)
#### 方法二：使用hexo的插件插入图片

现在升级到hexo3可以直接在`_config.yml`中设置`post_asset_folder: true`
然后安装hexo，asset-img的插件，`npm install hexo-asset-image --save`

***反正我没用成功你们愿意的话自己尝试吧，我只做到在文章内部正确插入了图片但是首页上的封面加载失败了***

新建文章后的目录结构应该是如下：

	本地图片测试
	├── apppicker.jpg
	├── logo.jpg
	└── rules.jpg
	本地图片测试.md

所以插入图片的代码大致如下：

	{% asset_img <your_img_name>.jpg post-cover %}

具体使用方法，我参考了这篇文章[https://www.jianshu.com/p/c2ba9533088a---“hexo博客图片问题”](https://www.jianshu.com/p/c2ba9533088a "hexo博客图片问题")
首先在这里感谢博文作者TSimeon这篇博文给我的帮助，如果对我的转载行为感到不满，请联系我，谢谢

### 配置右上角导航栏菜单项

注意: 必须在主题的 `_config.yml` 中配置以下信息, 而非博客根目录下的 `_config.yml`

    menu:
      Home: / #页面所在的路径
      My Works: /works
      About: /about
      #Tags: /tags


## 用户信息配置

注意: 必须在主题的 `_config.yml` 中配置以下信息, 而非博客根目录下的 `_config.yml`

	user_name: your name
	user_avatar: your avatar
	user_location: your location
	user_description: about you introduction
	
	// this info will show About page
	user_contact: 
	user_introduction
	
	// config you share info
	weibo_username: 
	zhihu_username: 
	github_username:
	twitter_username: 
	facebook_username:  
	linkedin_username: 
## 代码高亮功能
如果你对hexo原先的代码高亮或者说配合在主题中不满意的话，可以这么干，先把博客根目录中 `_config.yml`

	highlight:
  	  enable: false
  	  line_number: false
  	  auto_detect: false
  	  tab_replace: false	
原先的高亮功能关闭，然后在主题的`_config.yml`中找到

	block_highlight: highlight_rainbow

可以将后面的值修改为以下类型：

	highlight_default
	highlight_light
	highlight_github
	highlight_rainbow
	highlight_vs
	highlight_atom

***这部分功能需要感谢WizardMerlin在原主题上的pull request***
	
## 创建 About 页面
在博客根目录下的 `source` 文件夹里创建一个 `about` 文件夹, 然后打开该文件夹, 新建一个 `index.md`, 打开, 将下面这段文本复制到 `index.md` 里保存
	
	title: about
	date: 2017-05-31 10:05:56
	layout: about
	---

## 创建 Works 页面
创建的方式和上述创建 About 页面相同, 只不过是 `index.md` 内容略有不同, works 页面的 `index.md` 如下:

```
title: My Works
date: 2017-05-31 10:05:56
layout: works
---
```

然后再在博客根目录下的 `source` 文件夹下创建一个 `_data` 文件夹, 然后打开, 在里面新建一个 `project.json` 文件

project.json 文件格式范本:

	{
	  "Apple 官网临摹": {
	    "title": "Apple 官网临摹",
	    "subTitle": "根据美版apple官网临摹",
	    "img_link": "http://o7bkkhiex.bkt.clouddn.com/item-apple.jpg",
	    "use" : ["jQuery"],
	    "link": "http://haojen.github.io/apple-linmo/",
	    "data":"2016.3",
	    "direction": "临摹 2016 年三月份 Apple 美版单页面。"
	  },
	   "Anisina (阿尼丝娜)": {
	    "title": "Anisina",
	    "subTitle": "基于 Hexo 制作的个人博客主题",
	    "img_link": "http://o7bkkhiex.bkt.clouddn.com/Anisina.png",
	    "use" : ["jQuery","Bootstrap","Node.js","EJS","Hexo","SASS"],
	    "link": "http://haojen.github.io/",
	    "data": "2016.5",
	    "direction":
	        "Hexo 是某位台湾友人基于 Node.js 编写的博客框架"
	  }
	}
	
## 功能配置
***这些是原主题中挺牛的功能，我自己也没用起来，直接抄过来，希望原作者见谅***

可以依次在主题的根目录中执行终端命令, 根据自身需求分别安装依赖
	
	// 流程图功能
    npm install hexo-filter-flowchart --save
    
    // Emoji
    npm install hexo-filter-github-emojis --save
    
    // 搜索功能
    npm install hexo-generator-search --save
    
    // 数学公式
    npm install hexo-renderer-mathjax --save
    
具体的使用教程, 请参阅 [Claudia主题示例博客中对应的文章](https://haojen.github.io/Claudia-theme-blog/)    

## 最后

如果遇到任何问题, 可以提交 issue , 我看到会尽快回复!喜欢的话不妨给个 Star

另外如果原作者或此主题中涉及的作者对版权等觉得我的行为涉及版权问题的，请联系我，我会尽快处理，尽量不给大家添麻烦，谢谢。

## License

MIT © [DraceWang](https://dracewang.github.io/)