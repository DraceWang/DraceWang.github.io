---
title: ubuntu14.04 安装redmine2.5.3  
categories: ALM开发  
tags: 
	- redmine 
	- ubuntu 
date: 2014-12-15
---

![post-cover](https://i.loli.net/2020/10/27/Nv37KteIuWEq961.jpg)

## 安装redmine的环境
```
	sudo apt-get install -y apache2 php5 libapache2-mod-php5 mysql-server php5-mysql libapache2-mod-perl2 libcurl4-openssl-dev libssl-dev apache2-dev libapr1-dev libaprutil1-dev libmysqlclient-dev libmagickcore-dev libmagickwand-dev curl git-core patch build-essential bison zlib1g-dev libssl-dev libxml2-dev libxml2-dev sqlite3 libsqlite3-dev autotools-dev libxslt1-dev libyaml-0-2 autoconf automake libreadline6-dev libyaml-dev libtool imagemagick apache2-utils vim 
```
## 安装ruby
```
	sudo apt-get -y install ruby2.0 ruby2.0-dev ruby1.9.1-dev 
```

> 安装过程中数据库的管理员密码为：admin
```
	mysql -u root -p
```
### 创建空的数据库
```
	CREATE DATABASE redmine CHARACTER SET utf8;
	CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
	GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```





### 更新ruby的库（可能需要多更新几次，尽量更新过吧）
```
	sudo gem update 
```
## 安装budler
```
sudo gem install bundler 
```

## 下载redmine2.5.3
```
	wget http://www.redmine.org/releases/redmine-2.5.3.tar.gz
```
解压`redmine-2.5.3.tar.gz`
```
 	tar zxvf redmine-2.5.3.tar.gz
```
进入redmine目录修改配置文件，将其与数据库关联
```
	 cd redmine-2.5.3/config
	 cp database.yml.examle database.yml
	 sudo vim databse.yml
```
**将用户名改为redmine 将密码改为上面对应的my_password**

### bundler根据redmine的gemfile进行环境安装
```
	sudo bundle install
```

### 生成数据存储秘钥
```
	rake generate_secret_token
```
### 创建数据库结构
```
	RAILS_ENV=production rake db:migrate
```
### 设置默认数据库数据
```
	RAILS_ENV=production rake redmine:load_default_data
```
### 启动redmine服务
```
	ruby script/rails server webrick -e production
```
### 安装插件
将插件解压或者git下来的文件夹，拷贝到`/redmine-2.5.3/plugins/`  
```
	rake redmine:plugins:migrate RAILS_ENV=production
```