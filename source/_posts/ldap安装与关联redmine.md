---
title: LDAP安装与关联redmine  
categories: ALM开发 
tags: 
	- LDAP 
	- redmine
date: 2014-12-09 12:09:00
cover: https://i.loli.net/2020/10/27/Nv37KteIuWEq961.jpg
---

## 安装LDAP

```bash
	sudo vim /etc/hosts
```
 

修改 `127.0.1.1 hostname.example.com `hostname //hostname是本机的hostname不是“hostname”

 
```bash
	sudo apt-get install slapd ldap-utils //安装ldap
```
 

测试
```bash
	sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn

	ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn
```
 

---------------------------------------------------------------------------------


添加用户


```bash
	cd /etc/ldap
	
	sudo vim add_content.ldif   //创建add_content.ldif添加下列文字
```
---
```ldif
	dn: ou=People,dc=example,dc=com
	
	objectClass: organizationalUnit
	
	ou: People
	
	 
	
	dn: ou=Groups,dc=example,dc=com
	
	objectClass: organizationalUnit
	
	ou: Groups
	
	 
	
	dn: cn=miners,ou=Groups,dc=example,dc=com
	
	objectClass: posixGroup
	
	cn: miners
	
	gidNumber: 5000
	
	 
	
	dn: uid=john,ou=People,dc=example,dc=com
	
	objectClass: inetOrgPerson
	
	objectClass: posixAccount
	
	objectClass: shadowAccount
	
	uid: john
	
	sn: Doe
	
	givenName: John
	
	cn: John Doe
	
	mail: 385691567@qq.com
	
	displayName: John Doe
	
	uidNumber: 10000
	
	gidNumber: 5000
	
	userPassword: johnldap
	
	gecos: John Doe
	
	loginShell: /bin/bash
	
	homeDirectory: /home/john
```
---
```ldif
	ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_content.ldif //添加ldif文件
	
	ldapsearch -x -LLL -b dc=example,dc=com 'uid=john' cn gidNumber //查询john
```
 
{% note tip%}
 -x:“简单”绑定;不使用默认SASL方法


 -LLL:禁用打印无关的信息


 uid =john:一个“过滤器”找到john用户


 cn gidNumber:请求某些特定属性的显示(默认是显示所有属性)

 
{% endnote %}
 

 

 

 



## redmine设置

---

认证模式（LDAP）

名称：（随便填）

主机： ldap服务器IP

端口： 默认389

账号： cn=admin,dc=example,dc=com

密码： admin

Base DN: dc=example,dc=com

LDAP过滤器：

超时（秒）：

即时用户生成：打勾

登录名属性：uid

名字属性：sn

姓氏属性：givenName

邮件属性：mail

--------------------------------------------------------------------



