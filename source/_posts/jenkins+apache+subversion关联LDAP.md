---
title: jenkins+apache+subversion关联LDAP  
date: 2014-12-09
categories: ALM开发  
tags: 
    - LDAP 
    - jenkins 
    - apache 
    - subversion  
cover: https://s2.loli.net/2023/09/20/qORtrHv9IbkSilh.png
---


## 安装jenkins
```bash
	wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
```

添加以下信息到你的源`/etc/apt/sources.list`:
```bash
	deb http://pkg.jenkins-ci.org/debian binary/
	sudo apt-get update
	sudo apt-get install jenkins
```

## 下载安装SVN+apache
```bash
	sudo apt-get install subversion libapache2-svn apache2 apache2-utils
	
	sudo a2enmod ssl
	
	sudo a2ensite default-ssl
	
	sudo a2enmod dav_svn
```


### 设置apache2
```bash
	sudo vim /etc/apache2/mods-enabled/dav_svn.conf
```
编辑该文件使之如下：
```conf
	<Location /svn>
	
	DAV svn
	
	SVNPath /home/svn
	
	AuthType Basic
	
	AuthName "Subversion Repository"
	
	AuthUserFile /etc/apache2/dav_svn.passwd
	
	Require valid-user
	
	</Location>
```
更改 `/home/svn` 到任何你要保存仓库的地址。如果没有的话就新建。
```bash
	sudo mkdir /home/svn
```
### 开启svn服务
```bash
	sudo svnadmin  create /home/svn
```


### 是APache成为该仓库的所有者。
```bash
	sudo chown -R www-data /home/svn
```


### 建立密码文件     
```bash
	sudo htpasswd -cm /etc/apache2/dav_svn.passwd admin
```
admin是你想使用的用户名，然后输入两次密码。

### 重启Apache
```bash
	sudo /etc/init.d/apache2 restar
```
***


## 关联svn和LDAP


### a pache添加ldap组件


```bash
	sudo a2enmod ldap
	
	sudo a2enmod authnz_ldap
```
### 修改apache中的svn配置文件
```bash
	sudo vim /etc/apache2/mods-enabled/dav_svn.conf
```
---
```conf
	<Location /svn>
	DAV svn
	SVNParentPath /home/svn/
	
	AuthType Basic
	AuthName "Subversion Repository"
	
	#LDAP
	AuthBasicProvider ldap
	AuthLDAPBindDN "cn=admin,dc=example,dc=com"
	AuthLDAPBindPassword admin
	AuthLDAPURL "ldap://localhost:389/dc=example,dc=com?uid?sub?(objectClass=*)"
	
	AuthzSVNAccessFile /etc/apache2/dav_svn.authz
	require valid-user
	</Location>
```
### 重启Apache
```bash
	sudo /etc/init.d/apache2 restart
```
## jenkins关联LDAP

打开localhost：8080，jenkins默认安装在8080端口上

依次进入-->系统管理-->Configure Global Security

启用安全设置
```conf
	LDAP
	服务器	     ldap://localhost:389
	root DN      dc=example,dc=com
	Manager DN	 cn=admin,dc=example,dc=com
	管理密码      ****    //这是你之前安装ldap时输入的管理员密码    
```
点击保存应用即可