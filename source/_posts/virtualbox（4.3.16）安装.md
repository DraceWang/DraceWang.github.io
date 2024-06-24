---
title: virtualbox（4.3.16）安装    
categories: Linux  
tags: 
    - ubuntu 
    - virtualbox 
date: 2014-12-09 12:00:00
cover: https://s2.loli.net/2023/09/21/UswxOvhH9yug4LT.jpg
---


## virtualbox（4.3.16）安装

```bash
	sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian trusty contrib" >> /etc/apt/sources.list'
	wget -q http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc -O- | sudo apt-key add -
	sudo apt-get update
	sudo apt-get install virtualbox-4.3
```
```bash
VBoxManage -v //查看版本号
```
------------------------------------------------------------------------------------------------------------------------------------------

### Oracle_VM_VirtualBox_Extension_Pack安装
[http://download.virtualbox.org/virtualbox](http://download.virtualbox.org/virtualbox)查看对应版本
```	bash
	mkdir ~/ISO
	cd ISO
	wget http://download.virtualbox.org/virtualbox/4.3.16/Oracle_VM_VirtualBox_Extension_Pack-4.3.16-95972.vbox-extpack
	sudo vboxmanage extpack install Oracle_VM_VirtualBox_Extension_Pack-4.3.16-95972.vbox-extpack
```

```bash
 #卸载指定扩展包
 VBoxManage extpack uninstall <name>

 #显示已安装的扩展包
 VBoxManage list extpacks

 #移除安装扩展包失败或卸载扩展包失败时可能遗留下来的文件和目录
 VBoxManage extpack cleanup
```

-------------------------------------------------------------------------------------------------------------------------------------------
### 创建Ubuntu Server虚拟机
```bash
	vboxmanage createvm --name "ubuntuserver" --ostype Ubuntu_64 --register //创建并注册一个虚拟机取名为“ubuntuserver”操作系统类型改为Ubuntu_64
```
Settings file: '/home/stack/VirtualBox VMs/ubuntuserver/ubuntuserver.vbox'            
```bash
	cd ~/VirtualBox\ VMs/ubuntuserver/
```
VirtualBox会自动创建一个名叫”VirtualBox VMs”的子目录，所有的虚拟机都会保存在这个目录内。进入对应于ubuntuserver的子目录。
```bash
	sudo vboxmanage modifyvm "ubuntuserver" --memory 2048 --acpi on --boot1 dvd --boot2 disk
```
指定ubunutserver使用2G内存、使用ACPI、启动顺序为先光盘再硬盘
```bash
	sudo vboxmanage createhd --filename ubuntuserver.vdi --size 50000
```
创建一个虚拟硬盘，最大容量50G（实际用量将随着文件增加而增加，上限为50G）
```	bash
	sudo vboxmanage storagectl "ubuntuserver" --name "SCSI Controller" --add scsi
	sudo vboxmanage storageattach "ubuntuserver" --storagectl "SCSI Controller" --port 0 --device 0 --type hdd --medium ubuntuserver.vdi
```
创建虚拟硬盘控制器，使用SCSI接口，并将刚刚创建的虚拟硬盘连接到这个控制器
```bash
	sudo vboxmanage storagectl "ubuntuserver" --name "IDE Controller" --add ide
	sudo vboxmanage storageattach "ubuntuserver" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium ~/ISO/ubuntu*
```
创建虚拟硬盘控制器，使用IDE接口，并把ISO文件连接到这个接口（虚拟光驱）
```bash
	sudo vboxmanage modifyvm "ubuntuserver" --nic1 bridged 
```
配置网卡1为桥接模式
```bash
	sudo vboxmanage modifyvm "ubuntuserver" --bridgeadapter1 em1 //具体查询本机网卡  
```
绑定桥接到物理网卡
```bash
	sudo vboxmanage modifyvm "ubuntuserver" --vrde on  
	sudo vboxmanage modifyvm "ubuntuserver" --vrdeport 5000                    //监听5000端口  
	sudo vboxmanage modifyvm "ubuntuserver" --vrdeaddress 192.168.2.200        //监听地址  
	sudo vboxmanage modifyvm "ubuntuserver" --vrdeauthtype external            //认证类型  
	sudo vboxmanage modifyvm "ubuntuserver" --vrdeauthlibrary default          //使用默认的认证库，也就是使用server的用户名和密码登陆  
```

```bash
	sudo vboxmanage startvm "ubuntuserver" --type headless                //10.10.3.16
	//sudo VBoxManage controlvm "ubuntuserver" poweroff //关闭虚拟机 
```
> 注意–type headless参数，这个是必须的，因为我们是在Ubuntu服务器的命令行环境下启动虚拟机，无法使用图形界面。  
> 通过VRDE虚拟桌面来安装虚拟机上的操作系统。

```bash
	VBoxManage snapshot "ubuntuserver" take "backup0918" 
```
为ubuntuserver建立快照取名
```bash
	VBoxManage snapshot "ubuntuserver" restore "backup0918base"
```
恢复到快照名为xx的快照

---

虚拟机快照管理命令
```bash
	VBoxManage snapshot         <uuid>|<name> 为指定的虚拟机拍快照
	                            take <name> 为快照取名
	                        [-desc <desc>]| 给快照添加描述
	                        delete <uuid>|<name> | 丢弃指定的快照                  
	                     restore <uuid|snapname> | 恢复到快照名为xx的快照
	                              restorecurrent | 恢复到最近的快照
	                        edit <uuid>|<name>| 编辑指定的快照
	                                   -current 编辑当前快照
	                          [-newname <name>] 修改快照名称
	                          [-newdesc <desc>] 修改快照描述
	                        showvminfo <uuid>|<name> 显示快照的虚拟机信息


```