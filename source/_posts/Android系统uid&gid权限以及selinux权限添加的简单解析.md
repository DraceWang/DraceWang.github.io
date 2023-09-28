---
title: Android系统权限以及selinux权限添加的简单解析
date: 2015-9-29
updated: 2016-4-21
categories: Android  
tags: 
    - linux
    - uid
    - gid
    - selinux
---
![post-cover](https://i.loli.net/2020/10/27/ogxvylKmiRqLtPQ.jpg)


在公司做项目发现/data/misc/dhcp这个目录的文件无法访问，抓log后又没有显示selinux权限报错的avcdeny，但是需求又依赖于dhcp协议的解析，所以一直在寻找这个问题的解决方案。现在系统整理一下并把相关知识记录下来，Android系统来源于Linux系统所以我们先来看linux系统。

## 1.linux系统中uid以及gid权限的学习

UID：用户id，UserID，简称UID。

GID：用户组id，GroupID，简称GID。

在linux系统中如何判别一个文件的拥有者？以及分辨可以对它进行操作的用户？就是利用uid和gid。每个文件都会有所谓的拥有者id（也叫文件所有者，owner）和拥有者组id，当我们有显示文件属性需求的时候，系统会根据`/etc/password`与`/etc/group`的内容，找到uid与gid对应的账号与组名再显示出来。

在 Linux 中，一个用户 UID 标示一个给定用户。Linux系统中的用户(UID)分为3类，即普通用户、根用户、系统用户。

普通用户是指所有使用Linux系统的真实用户，这类用户可以使用用户名及密码登录系统。Linux有着极为详细的权限设置，所以一般来说普通用户只能在其家目录、系统临时目录或其他经过授权的目录中操作，以及操作属于该用户的文件。通常普通用户的UID大于500，因为在添加普通用户时，系统默认用户ID从500开始编号。
根用户也就是root用户，它的ID是0，也被称为超级用户，root账户拥有对系统的完全控制权：可以修改、删除任何文件，运行任何命令。所以root用户也是系统里面最具危险性的用户，root用户甚至可以在系统正常运行时删除所有文件系统，造成无法挽回的灾难。所以一般情况下，使用root用户登录系统时需要十分小心。

组(GID)又是什么呢？事实上，在Linux下每个用户都至少属于一个组。举个例子：每个学生在学校使用学号来作为标识，而每个学生又都属于某一个班级，这里的学号就相当于UID，而班级就相当于GID。当然了，每个学生可能还会同时参加一些兴趣班，而每个兴趣班也是不同的组。也就是说，每个学生至少属于一个组，也可以同时属于多个组。在Linux下也是一样的道理。

### 1.1 linux账号与用户组权限扩展（euid，egid，fsuid，fsgid，suid，sguid）[^3]

每个进程都拥有真实的用户、组（uid、gid），有效的用户、组（euid、egid），保存的设置用户、组（suid、sgid），还有linux中专门用于文件存储存取的用户、组id（fsuid、fsgid对于unix系统没有这两个fields）。现说明进程中每种类型用户的功能：

+ 真实的用户、组（uid、gid）：进程的真正所有者。每当用户在shell终端登录时，都会将登录用户作为登录进程的真正所有者。通过getuid来获得进程的真正用户所有者，修改进程的真正用户所有者可以通过setuid、seteuid、setresuid、setreuid。

+ 有效的用户、组（euid、egid）：进程的有效用户、组。进程所执行各种操作所允许的权限（process  credentials）是依据进程的有效用户来判断的，（在linux系统中（内核2.4以上）又引入了一个新的进程权限管理模型process  capabilities，通过process capabilities来确定进程所允许的各种操作[可参看《深入理解linux内核》table  20-3]）。通过geteuid来获得进程的有效用户，修改进程的有效用户可以通过setuid、seteuid、setresuid、setreuid、seteuid。

+ 文件系统的用户、组（fsuid、fsgid）：用于进行文件访问的用户、组，这是linux系统中新引入的一类用户、组，对于unix系统文件的访问是通过euid来判断，没有函数获得进程的fsuid，用于修改有效用户的函数都会同时修改fsuid，如果要单独修改fsuid，而不修改euid，可以调用setfsuid。

+ 保存的设置用户、组（suid、sgid）：保存的设置用户、组。进程中该类型的用户、组主要的用处是用于还原有效用户，观察到对于非超级用户用于修改有效用户的各个函数setuid、seteuid、setresuid、setreuid、seteuid普遍有一个前提条件就是如果修改后的有效用户是原先的suid则允许修改，利用这一点，进程可以修改有效用户到一个新用户，然后还原到原来的值（原来的值保存在保存设置的用户）。通过getresuid来获得进程的真实用户、有效用户、保存的设置用户。

### 1.2 suid和sgid详解[^1]

#### 1.2.1 UNIX下关于文件权限的表示方法和解析

- SUID 是 Set User ID
- SGID 是 Set Group ID

UNIX下可以用`ls -l`命令来看到文件的权限。用ls命令所得到的表示法的格式是类似这样的：`-rwxr-xr-x` 。下面解析一下格式所表示的意思。这种表示方法一共有十位：

```shell
9 8 7 6 5 4 3 2 1 0

- r w x r - x r - x
```

第9位表示文件类型,可以为p、d、l、s、c、b和-：

-   p表示命名管道文件
-   d表示目录文件
-   l表示符号连接文件
-   -表示普通文件
-   s表示socket文件
-   c表示字符设备文件
-   b表示块设备文件

第8-6位、5-3位、2-0位分别表示文件所有者的权限，同组用户的权限，其他用户的权限，其形式为rwx：

-   r表示可读，可以读出文件的内容
-   w表示可写，可以修改文件的内容
-   x表示可执行，可运行这个程序

没有权限的位置用 `-`表示

例子：

`ls -l myfile`显示为：

```shell
-rwxr-x--- 1 foo staff 7734 Apr 05 17:07 myfile
```

表示文件myfile是普通文件，文件的所有者是foo用户，而foo用户属于staff组，文件只有1个硬连接，长度是7734个字节，最后修改时间4月5日17:07。

所有者foo对文件有读写执行权限，staff组的成员对文件有读和执行权限，其他的用户对这个文件没有权限。

如果一个文件被设置了SUID或SGID位，会分别表现在所有者或同组用户的权限的可执行位上。例如：

1、`-rwsr-xr-x` 表示SUID和所有者权限中可执行位被设置

2、`-rwSr--r--` 表示SUID被设置，但所有者权限中可执行位没有被设置

3、`-rwxr-sr-x` 表示SGID和同组用户权限中可执行位被设置

4、`-rw-r-Sr--` 表示SGID被设置，但同组用户权限中可执行位没有被设置

其实在UNIX的实现中，文件权限用12个二进制位表示，如果该位置上的值是1，表示有相应的权限：

```undefined
11 10 9 8 7 6 5 4 3 2 1 0

S  G  T r w x r w x r w x
```

第11位为SUID位，第10位为SGID位，第9位为sticky位，第8-0位对应于上面的三组rwx位。


上面的`-rwsr-xr-x`的值为： `1 0 0 1 1 1 1 0 1 1 0 1`

`-rw-r-Sr--`的值为： `0 1 0 1 1 0 1 0 0 1 0 0`

>   同样的，rwx可以对应我们熟知的数值：2，4，1。
>
>   那么suid和sgid也是可以对应数值的（隐藏权限：SUID（4）、SGID（2）、SBIT（1）），不过是在原先的位置前，因为suid和sgid属于隐藏权限。

给文件加SUID和SUID的命令如下：

```bash
chmod u+s  filename  # 设置SUID位
chmod 4770 filename

chmod u-s filename  # 去掉SUID设置
chmod 0770 filename

chmod g+s filename # 设置SGID位
chmod 2770 filename

chmod g-s filename # 去掉SGID设置
chmod 0770 filename
```

#### 1.2.2 SUID和SGID的详细解析

由于SUID和SGID是在执行程序（程序的可执行位被设置）时起作用，而可执行位只对普通文件和目录文件有意义，所以设置其他种类文件的SUID和SGID位是没有多大意义的。

首先讲普通文件的SUID和SGID的作用。例子：

如果普通文件myfile是属于foo用户的，是可执行的，现在没设SUID位，ls命令显示如下：

`-rwxr-xr-x 1 foo staff 7734 Apr 05 17:07 myfile`
 任何用户都可以执行这个程序。UNIX的内核是根据什么来确定一个进程对资源的访问权限的呢？是这个进程的运行用户的（有效）ID，包括user id和group id。用户可以用id命令来查到自己的或其他用户的user id和group id。

除了一般的user id 和group id外，还有两个称之为effective 的id，就是有效id，上面的四个id表示为：uid，gid，euid，egid。内核主要是根据euid和egid来确定进程对资源的访问权限。

一个进程如果没有SUID或SGID位，则euid=uid egid=gid，分别是运行这个程序的用户的uid和gid。例如kevin用户的uid和gid分别为204和202，foo用户的uid和gid为200，201，kevin运行myfile程序形成的进程的euid=uid=204，egid=gid=202，内核根据这些值来判断进程对资源访问的限制，其实就是kevin用户对资源访问的权限，和foo没关系。

如果一个程序设置了SUID，则euid和egid变成被运行的程序的所有者的uid和gid，例如kevin用户运行myfile，euid=200，egid=201，uid=204，gid=202，则这个进程具有它的属主foo的资源访问权限。

SUID的作用就是这样：让本来没有相应权限的用户运行这个程序时，可以访问他没有权限访问的资源。passwd就是一个很鲜明的例子。

SUID的优先级比SGID高，当一个可执行程序设置了SUID，则SGID会自动变成相应的egid。

下面讨论一个例子：

UNIX系统有一个/dev/kmem的设备文件，是一个字符设备文件，里面存储了核心程序要访问的数据，包括用户的口令。所以这个文件不能给一般的用户读写，权限设为：cr--r----- 1 root system 2, 1 May 25 1998 kmem

但ps等程序要读这个文件，而ps的权限设置如下：

```undefined
-r-xr-sr-x 1 bin system 59346 Apr 05 1998 ps
```

这是一个设置了SGID的程序，而ps的用户是bin，不是root，所以不能设置SUID来访问kmem，但大家注意了，bin和root都属于system组，而且ps设置了SGID，一般用户执行ps，就会获得system组用户的权限，而文件kmem的同组用户的权限是可读，所以一般用户执行ps就没问题了。但有些人说，为什么不把ps程序设置为root用户的程序，然后设置SUID位，不也行吗？这的确可以解决问题，但实际中为什么不这样做呢？因为SGID的风险比SUID小得多，所以出于系统安全的考虑，应该尽量用SGID代替SUID的程序，如果可能的话。下面来说明一下SGID对目录的影响。SUID对目录没有影响。如果一个目录设置了SGID位，那么如果任何一个用户对这个目录有写权限的话，他在这个目录所建立的文件的组都会自动转为这个目录的属主所在的组，而文件所有者不变，还是属于建立这个文件的用户。



#### 1.2.3 在Linux系统下suid和sgid的作用机制

进程在运行的时候，有一些属性，其中包括 实际用户ID,实际组ID,有效用户ID,有效组ID等。 实际用户ID和实际组ID标识我们是谁，谁在运行这个程序,一般这2个字段在登陆时决定，在一个登陆会话期间， 这些值基本上不改变。

而有效用户ID和有效组ID则决定了进程在运行时的权限。内核在决定进程是否有文件存取权限时，是采用了进程的有效用户ID来进行判断的。

知道了这点，我们来看看SUID的解决途径：

当一个程序设置了为SUID位时，内核就知道了运行这个程序的时候，应该认为是文件的所有者在运行这个程序。即该程序运行的时候，有效用户ID是该程序的所有者，SGID也是同理。

### 1.3 Linux系统uid和gid权限总结

创建一个文件，系统会根据该用户的uid和gid来设置这个文件的owner以及gid，同时root用户具有最高权限，不受权限管理限制。在系统中我们可以根据`ls -l`显示出的文件权限以及uid和gid来确认文件对应的权限。如果当前用户并不是文件或目录的创建者在linux系统中会根据最后三位`rwx`位的设置来控制是否可以控制该文件，因此为了安全考虑一般会将文件设置为770或者775等等，可以根据文件在场景中的实际需要来设置。当然除此以外也可以让用户加入该文件所在的用户组别，即加入gid，那么即使文件访问权限是770，也可以正常以完整读写执行的权限使用该文件。如果需要让非拥有者用户可以访问该文件可以设置suid以及sgid，鉴于安全考虑建议设置sgid，可以让系统在需要访问该文件时，临时将其他用户设置为该文件拥有者的gid，因此正常访问该文件。

+   uid相同，系统认为时所有者可以直接访问（suid位设置时，即使uid不同系统也会认为时文件所有者）
+   uid不同，系统认为非所有者继续确认是否属于同组
    -   同组，按照组权限控制访问（sgid位设置时，即使uid和gid与所有者不同系统也会认为时文件所有者所在的组，可以对应组权限访问）
    -   不同组，则继续根据其他用户的通用权限来控制访问

***SUID仅对二进制程序可用，shell脚本不可用，对目录也不可用***

## 2.Android系统的权限管理

### 2.1 Android 系统中的UID、GID、GIDS与PID

在 Android 上，一个用户 UID 标示一个应用程序。应用程序在安装时被分配用户 UID，应用程序在设备上的存续期间内，用户 UID 保持不变。对于普通的应用程序，GID即等于UID。

GIDS 是由框架在 Application 安装过程中生成，与 Application 申请的具体权限相关。 如果  Application 申请的相应的 permission 被 granted ，而且有对应的GIDS， 那么 这个Application 的  gids 中将 包含这个 gids。记住权限(GIDS)是关于允许或限制应用程序（而不是用户）访问设备资源。

![sandbox_model](https://i.loli.net/2020/11/02/xW5mITbqjucnHEh.gif)

Android  使用沙箱的概念来实现应用程序之间的分离和权限，以允许或拒绝一个应用程序访问设备的资源，比如说文件和目录、网络、传感器和  API。为此，Android 使用一些 Linux 实用工具（比如说进程级别的安全性、与应用程序相关的用户和组  ID，以及权限），来实现应用程序被允许执行的操作。

Android 应用程序运行在它们自己的 Linux 进程上，并被分配一个惟一的用户  ID。默认情况下，运行在基本沙箱进程中的应用程序没有被分配权限，因而此类应用程序访问系统或资源受到限制，Android  应用程序只能通过应用程序的 manifest 文件请求权限。

不同的应用程序可以运行在相同的进程中。对于此方法，首先必须使用相同的私钥签署这些应用程序，然后必须使用 manifest 文件给它们分配相同的  Linux 用户 ID，这通过用相同的值/名定义 manifest 属性 `android:sharedUserId`  来做到，从而共享对其数据和代码的访问。

![same UID](https://i.loli.net/2020/11/02/yZOkHGCpRKzc6Ld.gif)

基于用户id的安全机制，其实使用的是标准的Linux的权限控制的机制，本用户、本组的和其他用户各自有读、写、执行3中权限。系统在这方面的控制主要有：

>文件系统的各个文件具有Uid和Gid，并指定权限。
>每个进行具有自己的Uid和Gid，并指定它属于哪些组。
>每个进程可以根据本用户规则访问其Uid可以访问的文件。
>每个进程可以根据组规则访问其所属的所有组(Groups)可以访问的文件。
>如果文件定义了其他的用户可以访问的权限，可以被任何任何程序访问。
>任何进程都不可以访问不具有权限的文件。

### 2.2 共享UID（sharedUserId）

安装在设备中的每一个Android包文件（.apk）都会被分配到一个属于自己的统一的Linux用户ID，并且为它创建一个沙箱，以防止影响其他应用程序（或者其他应用程序影响它）。用户ID 在应用程序安装到设备中时被分配，并且在这个设备中保持它的永久性。

通过Shared User id,拥有同一个User id的多个APK可以配置成运行在同一个进程中.所以默认就是可以互相访问任意数据. 也可以配置成运行成不同的进程, 同时可以访问其他APK的数据目录下的数据库和文件.就像访问本程序的数据一样.

对于一个APK来说，如果要使用某个共享UID的话，必须做三步：

1. 在Manifest节点中增加android:sharedUserId属性。

2. 在`Android.mk`中增加LOCAL_CERTIFICATE的定义。

如果增加了上面的属性但没有定义与之对应的LOCAL_CERTIFICATE的话，APK是安装不上去的。提示错误是：`Package  com.test.MyTest has no signatures that match those in shared user  android.uid.system;  ignoring!`也就是说，仅有相同签名和相同sharedUserID标签的两个应用程序签名都会被分配相同的用户ID。例如所有和media/download相关的APK都使用android.media作为sharedUserId的话，那么它们必须有相同的签名media。

3. 把APK的源码放到`packages/apps/`目录下，用mm进行编译。

举例说明一下。

系统中所有使用`android.uid.system`作为共享UID的APK，都会首先在manifest节点中增加`android:sharedUserId="android.uid.system"`，然后在`Android.mk`中增加`LOCAL_CERTIFICATE := platform`。可以参见Settings等

系统中所有使用`android.uid.shared`作为共享UID的APK，都会在manifest节点中增加`android:sharedUserId="android.uid.shared"`，然后在`Android.mk`中增加`LOCAL_CERTIFICATE := shared`。可以参见Launcher等

系统中所有使用`android.media`作为共享UID的APK，都会在manifest节点中增加`android:sharedUserId="android.media"`，然后在`Android.mk`中增加`LOCAL_CERTIFICATE := media`。可以参见Gallery等。

 

另外，应用创建的任何文件都会被赋予应用的用户标识，并且正常情况下不能被其他包访问。当通过getSharedPreferences（String，int）、openFileOutput（String、int）或者openOrCreate  Database（String、int、SQLiteDatabase.CursorFactory）创建一个新文件时，开发者可以同时或分别使用MODE_WORLD_READABLE和MODE_WORLD_RITEABLE标志允许其他包读/写此文件。当设置了这些标志后，这个文件仍然属于自己的应用程序，但是它的全局读/写和读/写权限已经设置，所以其他任何应用程序可以看到它。

关于签名：

 

`build/target/product/security`目录中有四组默认签名供Android.mk在编译APK使用：

+ testkey：普通APK，默认情况下使用。

+ platform：该APK完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的APK所在进程的UID为system。

+ shared：该APK需要和home/contacts进程共享数据。

+ media：该APK是media/download系统中的一环。

应用程序的Android.mk中有一个LOCAL_CERTIFICATE字段，由它指定用哪个key签名，未指定的默认用testkey.

### 2.3 Android权限的细节[^2]

#### 2.3.1 **system app**

首先，什么时systemapp？在PackageManagerService中对是否是system app的判断： 
 **具有`ApplicationInfo.FLAG_SYSTEM`标记的，被视为System app**。

```java
    private static boolean isSystemApp(PackageParser.Package pkg) {
        return (pkg.applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) != 0;
    }

    private static boolean isSystemApp(PackageSetting ps) {
        return (ps.pkgFlags & ApplicationInfo.FLAG_SYSTEM) != 0;
    }
```

有两类app属于System app：

+ **特定的shared uid的app 属于system app**

例如：shared uid为`android.uid.system`，`android.uid.phone`，`android.uid.log`，`android.uid.nfc`，`android.uid.bluetooth`，`android.uid.shell`。这类app都被赋予了`ApplicationInfo.FLAG_SYSTEM`标志。

在PackageManagerService的构造方法中，代码如下：

```java
    mSettings.addSharedUserLPw("android.uid.system", Process.SYSTEM_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.phone", RADIO_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.log", LOG_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.nfc", NFC_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.bluetooth", BLUETOOTH_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.shell", SHELL_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
```

+  **特定目录中的app属于system app**

特定目录包括：`/vendor/overlay`，`/system/framework`，`/system/priv-app`，`/system/app`，`/vendor/app`，`/oem/app`。这些目录中的app，被视为system app。

在PackageManagerService的构造方法中，代码如下：

```java
    // /vendor/overlay folder
    File vendorOverlayDir = new File(VENDOR_OVERLAY_DIR);
    scanDirLI(vendorOverlayDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags | SCAN_TRUSTED_OVERLAY, 0);

    // Find base frameworks (resource packages without code). /system/framework folder
    File frameworkDir = new File(Environment.getRootDirectory(), "framework");
    scanDirLI(frameworkDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR
            | PackageParser.PARSE_IS_PRIVILEGED,
            scanFlags | SCAN_NO_DEX, 0);

    // Collected privileged system packages. /system/priv-app folder
    final File privilegedAppDir = new File(Environment.getRootDirectory(), "priv-app");
    scanDirLI(privilegedAppDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR
            | PackageParser.PARSE_IS_PRIVILEGED, scanFlags, 0);

    // Collect ordinary system packages. /system/app folder
    final File systemAppDir = new File(Environment.getRootDirectory(), "app");
    scanDirLI(systemAppDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);

    // Collect all vendor packages.
    File vendorAppDir = new File("/vendor/app");
    scanDirLI(vendorAppDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);

    // Collect all OEM packages. /oem/app folder
    final File oemAppDir = new File(Environment.getOemDirectory(), "app");
    scanDirLI(oemAppDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);
```

scanDirLI参数中的`PackageParser.PARSE_IS_SYSTEM`最终会被转换为Package的`ApplicationInfo.FLAG_SYSTEM`属性。这个过程相关的代码流程（简略）：

```java
scanDirLI(PackageParser.PARSE_IS_SYSTEM)

-> scanPackageLI(file, parseFlags | PackageParser.PARSE_MUST_BE_APK, scanFlags, currentTime, null);

-> scanPackageLI(pkg, parseFlags, scanFlags | SCAN_UPDATE_SIGNATURE, currentTime, user);

-> final PackageParser.Package res = scanPackageDirtyLI(pkg, parseFlags, scanFlags, currentTime, user);

-> scanPackageDirtyLI()中
    if ((parseFlags & PackageParser.PARSE_IS_SYSTEM) != 0) {
       pkg.applicationInfo.flags |= ApplicationInfo.FLAG_SYSTEM;
    }
```

#### 2.3.2 **privileged app**

*注：privileged app，在本文中称之为 **特权app**，主要原因是此类特权app可以使用protectionLevel为`signatureOrSystem`或者protectionLevel为`signature|privileged`的权限。*

从PackageManagerService的`isPrivilegedApp()`可以看出**特权app是具有`ApplicationInfo.PRIVATE_FLAG_PRIVILEGED`标志的一类app**。

```java
    private static boolean isPrivilegedApp(PackageParser.Package pkg) {
        return (pkg.applicationInfo.privateFlags & ApplicationInfo.PRIVATE_FLAG_PRIVILEGED) != 0;
    }
```

特权app首先必须是System app。也就是说 System app分为**普通的system app**和**特权的system app**。

```
System app = 普通的system app + 特权app
```

直观的（但不准确严谨）说，**普通的system app**就是`/system/app`目录中的app，**特权的system app**就是`/system/priv-app`目录中的app。 
 *BTW： `priv-app` 是`privileged app`的简写。*

**区分普通system app和特权app的目的是澄清这个概念：`"signatureOrSystem"`权限中的`System`不是为普通system app提供的，而是只有特权app能够使用。**

进一步说，`android:protectionLevel="signatureOrSystem"`中的`System`和`android:protectionLevel="signature|privileged"`中的`privileged`的含义是一样的。 
 可以从`PermissionInfo.java`中的`fixProtectionLevel()`看出来：

```java
    // PermissionInfo.java
    public static int fixProtectionLevel(int level) {
        if (level == PROTECTION_SIGNATURE_OR_SYSTEM) {
            // "signatureOrSystem"权限转换成了"signature|privileged"权限
            level = PROTECTION_SIGNATURE | PROTECTION_FLAG_PRIVILEGED;
        }
        return level;
    }
```

所以，当我们说起system app时，通常指的是前面提到的特定uid和特定目录中的app，包含了普通的system app和特权app。 
 当我们说起有访问`System`权限或者`privileged`权限的app时，通常指特权app。

**特权app首先是System app，然后要具有`ApplicationInfo.PRIVATE_FLAG_PRIVILEGED`标志。** 
 有两类app属于privileged app（特权app）：参考PackageManagerService的构造方法。

+  **特定uid的app**

    shared uid为`"android.uid.system"`，`"android.uid.phone"`，`"android.uid.log"`，`"android.uid.nfc"`，`"android.uid.bluetooth"`，`"android.uid.shell"`的app被赋予了privileged的权限。这些app同时也是system app。

```java
    // PackageManagerService的构造方法
    mSettings.addSharedUserLPw("android.uid.system", Process.SYSTEM_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.phone", RADIO_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.log", LOG_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.nfc", NFC_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.bluetooth", BLUETOOTH_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
    mSettings.addSharedUserLPw("android.uid.shell", SHELL_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
```

+ **`/system/framework`和`/system/priv-app`目录下的app**

`/system/framework`和`/system/priv-app`目录下的app被赋予了privileged的权限。 
 其中`/system/framework`目录中的apk，只是包含资源，不包含代码（dex）。

```java
    // PackageManagerService的构造方法
    // /system/framework
    File frameworkDir = new File(Environment.getRootDirectory(), "framework");
    scanDirLI(frameworkDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR
            | PackageParser.PARSE_IS_PRIVILEGED,
            scanFlags | SCAN_NO_DEX, 0);

    // /system/priv-app
    final File privilegedAppDir = new File(Environment.getRootDirectory(), "priv-app");
    scanDirLI(privilegedAppDir, PackageParser.PARSE_IS_SYSTEM
            | PackageParser.PARSE_IS_SYSTEM_DIR
            | PackageParser.PARSE_IS_PRIVILEGED, scanFlags, 0);
```

`PackageParser.PARSE_IS_PRIVILEGED`标志最终会转换为Package的`ApplicationInfo.PRIVATE_FLAG_PRIVILEGED`标志。大概的代码流程，如下：

```java
scanDirLI(PackageParser.PARSE_IS_PRIVILEGED)

-> scanPackageLI(file, parseFlags | PackageParser.PARSE_MUST_BE_APK, scanFlags, currentTime, null);

-> PackageParser.Package scannedPkg = scanPackageLI(pkg, parseFlags, scanFlags | SCAN_UPDATE_SIGNATURE, currentTime, user);

-> final PackageParser.Package res = scanPackageDirtyLI(pkg, parseFlags, scanFlags, currentTime, user);

-> scanPackageDirtyLI()中
    if ((parseFlags & PackageParser.PARSE_IS_PRIVILEGED) != 0) {
        pkg.applicationInfo.privateFlags |= ApplicationInfo.PRIVATE_FLAG_PRIVILEGED;
    }
```

#### 2.3.3 **Android app中的权限是必须先声明后使用吗？**

*在本文中，**声明权限**是指在AndroidManifest.xml中使用了`<permission>`，**使用权限**是指在AndroidManifest.xml中使用了`<uses-permission>`。**获得权限（或赋予权限）**是指真正的可以通过系统的权限检查，调用到权限保护的方法。*

场景：App A中声明了权限PermissionA，App B中使用了权限PermissionA。 
 那么App A必须比App B先安装，App B才能获取到权限吗？

答案是不一定，要看具体情况而定。这个**具体情况**就是**权限的保护级别**。

-   **情况一：PermissionA的保护级别是`normal`或者`dangerous`** 
     App B先安装，App A后安装，此时App B没有获取到PermissionA的权限。 
     即，此种情况下，权限必须先声明再使用。**即使App A和App B是相同的签名**。
-   **情况二：PermissionA的保护级别是`signature`或者`signatureOrSystem`** 
     App B先安装，App A后安装，**如果App A和App B是相同的签名，那么App B可以获取到PermissionA的权限。**如果App A和App B的签名不同，则App B获取不到PermissionA权限。
     即，**对于相同签名的app来说，不论安装先后，只要是声明了权限，请求该权限的app就会获得该权限。** 
     这也说明了对于具有相同签名的系统app来说，安装过程不会考虑权限依赖的情况。安装系统app时，按照某个顺序（例如名字排序，目录位置排序等）安装即可，等所有app安装完了，所有使用权限的app都会获得权限。

#### 2.3.4 **验证某个app是否获得了某个权限的方法**

可以用下面2个命令来验证：

```
adb shell dumpsys package permission <权限名>
adb shell dumpsys package <包名>
```

其中，`adb shell dumpsys package permission <权限名>`**可以查看某个权限是谁声明的，谁使用了**。 
 `adb shell dumpsys package <包名>`**可以看到某个app是否获得了某个权限**。

+   adb shell dumpsys package permission <权限名>

通过下面的命令，查看`com.package.a.PermissionA`权限：

```
adb shell dumpsys package permission com.package.a.PermissionA1
```

输出结果为：（注：这里只显示关键信息，省略了其他很多信息）

```
Permission [com.package.a.PermissionA] (a2930ef):
    sourcePackage=com.package.a            // 权限的声明者
    uid=10226 gids=null type=0 prot=normal //保护级别为normal

Packages:
  Package [com.package.b] (350d95):      // 权限的使用者
    requested permissions:
      com.package.a.PermissionA
    install permissions:
      com.package.a.PermissionA, granted=true, flags=0x012345678910
```

其中`granted=true`表明App B 获取到了`com.package.a.PermissionA`权限。 
 如果App B没有获取到`com.package.a.PermissionA`权限，则输出结果中只有权限的声明信息，如下：

```
Permission [com.package.a.PermissionA] (a2930ef):
    sourcePackage=com.package.a
    uid=10226 gids=null type=0 prot=normal 保护级别为normal
```

+   adb shell dumpsys package <包名>

查看某个app是否获得了某个权限。

查看App B（包名为com.package.b）是否获得了权限`com.package.a.PermissionA`：

```
adb shell dumpsys package com.package.b1
```

输出结果为：（注：省略了很多其他信息）

```
Packages:
  Package [com.package.b] (6611d16):
    requested permissions:         // 申请了哪些权限
      com.package.a.PermissionA
    install permissions:           // 获得了哪些权限
      com.package.a.PermissionA, granted=true, flags=0x0123456
```

上面的输出结果表明，App B获取到了`com.package.a.PermissionA`权限。 
 如果App B没有获取到 `com.package.a.PermissionA` 权限，那么在 `install permissions` 中**不会**出现`com.package.a.PermissionA, granted=true, flags=0x0`。

### 2.4  Android Permission权限控制机制[^4]

基于UID和GID的Android进程隔离机制， 这是利用 Linux 已有的权限管理机制，通过为每一个 Application 分配不同的  uid 和 gid ， 从而使得不同的 Application 之间的私有数据和访问（ native 以及 java 层通过这种 sandbox 机制，都可以）达到隔离的目的 。 与此同时， Android 还 在此基础上进行扩展，提供了 permission  机制，一个GIDS就是一个Permission的集合,它主要是用来对 Application  可以执行的某些具体操作进行权限细分和访问控制，同时提供了 per-URI permission 机制，用来提供对某些特定的数据块进行  ad-hoc 方式的访问.

#### 2.4.1 权限基本信息

  一个权限主要包含三个方面的信息：权限的名称；属于的权限组；保护级别。一个权限组是指把权限按照功能分成的不同的集合。每一个权限组包含若干具体权限，例如在 android.permission-group.CONTACTS 组中包含  android.permission.WRITE_CONTACTS ，  android.permission.GET_ACCOUNTS，android.permission.READ_CONTANTS 等和联系人相关的权限。可以通过pm list 命令去获取相应的permission信息。也可以通过查看/data/sytem/packages.xml文件来获取。

```bash
$ adb shell pm list permissions -f
All Permissions:
+ permission:android.permission.REAL_GET_TASKS
  package:android
  label:null
  description:null
  protectionLevel:signature|privileged
+ permission:android.permission.REMOTE_AUDIO_PLAYBACK
  package:android
  label:null
  description:null
  protectionLevel:signature
--snip--
```
```shell
$adb shell pm list permissions -g -d
Dangerous Permissions:
group:android.permission-group.CONTACTS
  permission:android.permission.WRITE_CONTACTS
  permission:android.permission.GET_ACCOUNTS
  permission:android.permission.READ_CONTACTS
group:android.permission-group.PHONE
  permission:android.permission.READ_CALL_LOG
  permission:android.permission.ANSWER_PHONE_CALLS
  permission:android.permission.READ_PHONE_NUMBERS
  permission:android.permission.READ_PHONE_STATE
  permission:android.permission.CALL_PHONE
  permission:android.permission.WRITE_CALL_LOG
  permission:android.permission.USE_SIP
  permission:android.permission.PROCESS_OUTGOING_CALLS
  permission:com.android.voicemail.permission.ADD_VOICEMAIL
--snip--
```

#### 2.4.2 权限等级

 Android权限等级划分为normal,dangerous,signature,signatureOrSystem,system,development，不同的保护级别代表了程序要使用此权限时的认证方式。

normal 的权限只要申请了就可以使用，dangerous  的权限在安装时需要用户确认才可以使用，signature需要签名才能赋予权限，signatureOrSystem需要签名或者系统级应用(放置在`/system/app`目录下)才能赋予权限，system系统级应用(放置在`/system/app`目录下)才能赋予权限，系统权限的描述在`frameworks/base/core/res/AndroidManifest.xml`当中。

android 6.0（api level  23)之后进行了一次权限大升级，其中出现了一个新的protectionLevel叫做"signature|privileged"，signatureOrSystem变成了deprecated。在这之后，系统分成了system app和priviledged  app，其中`/system/app`目录下的应用只有system权限，而`/system/priv-app/`目录下的应用才有privileged  app  权限。而"signature|privileged"的权限等同于android6.0之前的signatureOrSystem。大于等于此时的system权限。

#### 2.4.3 权限管理

Package 的权限信息主要 通过在 AndroidManifest.xml 中通过一些标签来指定。如 <permission>  标签， <permission-group> 标签 <permission-tree> 等标签。如果 package  需要申请使用某个权限，那么需要使用 <use-permission> 标签来指定。

#### 2.4.4 CheckPermission

下面这一组接口主要用来检查某个调用（或者是其它 package 或者是自己）是否拥有访问某个 permission 的权限。参数中 pid 和 uid 可以指定，如果没有指定，那么 framework 会通过 Binder 来获取调用者的 uid 和 pid  信息，加以填充。返回值为 PackageManager.PERMISSION_GRANTED 或者  PackageManager.PERMISSION_DENIED 

```java
public int checkPermission(String permission, int pid, int uid) // 检查某个 uid 和 pid 是否有 permission 权限
public int checkCallingPermission(String permission) // 检查调用者是否有 permission 权限，如果调用者是自己那么返回 PackageManager.PERMISSION_DENIED
public int checkCallingOrSelfPermission(String permission) // 检查自己或者其它调用者是否有 permission 权限
```
     下面这一组和上面类似，如果遇到检查不通过时，会抛出异常，打印消息 。
```java
 public void enforcePermission(String permission, int pid, int uid, String message)
 public void enforceCallingPermission(String permission, String message)
 public void enforceCallingOrSelfPermission(String permission, String message)
```
#### 2.4.5 CheckUriPermission

为某个 package 添加访问 content Uri 的读或者写权限。
```java
 public void grantUriPermission(String toPackage, Uri uri, int modeFlags)
 public void revokeUriPermission(Uri uri, int modeFlags)
```
检查某个 pid 和 uid 的 package 是否拥有 uri 的读写权限，返回值表示是否被 granted 。
```java
 public int checkUriPermission(Uri uri, int pid, int uid, int modeFlags)
 public int checkCallingUriPermission(Uri uri, int modeFlags)
 public int checkCallingOrSelfUriPermission(Uri uri, int modeFlags)
 public int checkUriPermission(Uri uri, String readPermission,String writePermission, int pid, int uid, int modeFlags)
```
检查某个 pid 和 uid 的 package 是否拥有 uri 的读写权限，如果失败则抛出异常，打印消息 。
```java
 public void enforceUriPermission(Uri uri, int pid, int uid, int modeFlags, String message)
 public void enforceCallingUriPermission(Uri uri, int modeFlags, String message)
 public void enforceCallingOrSelfUriPermission(Uri uri, int modeFlags, String message)
 public void enforceUriPermission(Uri uri, String readPermission, String writePermission,int pid, int uid, int modeFlags, String message)
```
其中check开头的，只做检查，enforce开头的，不单检查，没有权限的还会抛出异常。

#### 2.4.5 权限机制实现分析

+ **CheckPermission**

 1. 如果传入的 permission 名称为 null ，那么返回 PackageManager.PERMISSION_DENIED 。
 2. 判断调用者 uid 是否符合要求 。
 1 ） 如果 uid 为 0 ，说明是 root 权限的进程，对权限不作控制。
 2 ） 如果 uid 为 system server 进程的 uid ，说明是 system server ，对权限不作控制。
 3 ） 如果是 ActivityManager 进程本身，对权限不作控制。
 4 ）如果调用者 uid 与参数传入的 req uid 不一致，那么返回 PackageManager.PERMISSION_DENIED 。
 3. 如果通过 2 的检查后，再 调用 PackageManagerService.checkUidPermission ，判断 这个 uid 是否拥有相应的权限，分析如下 。
 1 ） 首先它通过调用 getUserIdLP ，去 PackageManagerService.Setting.mUserIds 数组中，根据 uid 查找 uid （也就是 package ）的权限列表。一旦找到，就表示有相应的权限。
 2 ） 如果没有找到，那么再去 PackageManagerService.mSystemPermissions 中找。这些信息是启动时，从  /system/etc/permissions/platform.xml 中读取的。这里记录了一些系统级的应用的 uid 对应的  permission 。
 3 ）返回结果 。

+ **CheckUriPermission**
 1. 如果 uid 为 0 ，说明是 root 用户，那么不控制权限。
 2. 否则，在 ActivityManagerService 维护的 mGrantedUriPermissions 这个表中查找这个 uid 是否含有这个权限，如果有再检查其请求的是读还是写权限。

### 2.5 Android下的权限管理解析

在Android中用户的概念已经被淡化，通常使用的是root用户和shell用户。

shell用户的进程effectiveUid为2000， 在system/core/include/private/android_filesystem_config.h中可以看到如下定义

```c
#define AID_ROOT 0 /* traditional unix root user */
/* The following are for LTP and should only be used for testing */
#define AID_DAEMON 1 /* traditional unix daemon owner */
#define AID_BIN 2    /* traditional unix binaries owner */

#define AID_SYSTEM 1000 /* system server */

#define AID_RADIO 1001           /* telephony subsystem, RIL */
#define AID_BLUETOOTH 1002       /* bluetooth subsystem */
...
#define AID_DHCP 1014            /* dhcp client */
#define AID_SDCARD_RW 1015       /* external storage write access */
...
#define AID_SHELL 2000 /* adb and debug shell user */
...
#define AID_APP 10000       /* TODO: switch users over to AID_APP_START */
#define AID_APP_START 10000 /* first app user */
#define AID_APP_END 19999   /* last app user */
```

所以当我们使用adb shell连接到手机调试时，就是使用shell用户在Android系统中，这个时候我们的euid就是2000。

同时我们可以看到app的uid是从10000开始的，相对的user的uid也是相同，因此我们可以看到Android系统对应用的大致的一个权限的划分，并且通过上面说过的沙盒模型把这些进程都相应的独立分开相互不影响。

由于是shell进程的子进程，它的uid为2000，我们cc文件只有sdcard_r这个组才可以读，那么为什么我们的进程可以读这个文件呢？只有一种可能，那就是当前进程它的gids里面包含了sdcard_r。

```shell
shell:/proc/471 # cat status
Name:   shell
Umask:  0000
State:  S (sleeping)
Tgid:   471
Ngid:   0
Pid:    471
PPid:   571
TracerPid:      0
Uid:    2000    2000    2000    2000
Gid:    2000    2000    2000    2000
FDSize: 256
Groups: 1004 1007 1011 1015 1028 3001 3002 3003 3006 3009 3011 
```

我们看到Groups里面有gid为1015,1015就是对应sdcard_r。当前进程时shell进程的子进程，当然gids里面就包含了1015.

#### 2.5.1 系统级权限配置文档[^6]

为了控制访问的安全，加入安全机制。alps\frameworks\base\data\etc\platform.xml 存放着一些权限配置，我们可以看到类似

 ```xml
 <permission ame="android.permission.READ_EXTERNAL_STORAGE" >
    <group gid="sdcard_r" />
  </permission>

 <permission name="android.permission.WRITE_EXTERNAL_STORAGE" >
    <group gid="sdcard_rw" />
  </permission>
 ```

的权限配置。

这里的配置为读取外部存储卡的权限 以及写入外部存储卡的权限。

>   <permission name="android.permission.READ_EXTERNAL_STORAGE" >

里面的<group gid="sdcard_r" >这里的意义为：如果apk申请读外部存储卡，则会在创建apk时，给此进程增加一个组，组名为sdcard_r，使用的接口为setgid();

`android\frameworks\base\core\res`里面的AndroidManifest.xml，这个里面可以看到更多的系统权限配置表。

我们来看下这个配置里面的详细信息：

 ```xml

 <permission android:name="android.permission.CALL_PHONE"
    android:permissionGroup="android.permission-group.COST_MONEY"
    android:protectionLevel="dangerous"
    android:label="@string/permlab_callPhone"
    android:description="@string/permdesc_callPhone" />
 ```

这里面的内容：

-   android:name 权限名称。
-   android:permissionGroup 权限组，这个是在安装应用时显示在哪个组下的。
-   android:protectionLevel 保护等级
-   android:label 权限图标
-   android:description 权限详细描述

#### 2.5.2 Android 自定义权限[^5]

当b应用启动a应用的组建（可以是Activity，Service，BroadcastReceiver等）时，如果没有设置权限，那么将无法使用。那么就需要给a应用制定的组建设置一个Permission就可以了。

首先得先自定义一个权限，才有自定义权限可以被使用。所以在AndroidManifest.xml文件中如下代码：同时自定义权限组和自定义权限树的用法与此相同。通常应该遵循Android的命名方案(*.permission.*)但非必须.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.test" >

<permission android:name="com.example.permission.test"
    android:protectionLevel="normal"
    android:description="@string/permission_description" />
<permission-group android:name="com.example.permission.test.group" android:protectionLevel="normal"/>
<permission-tree android:name="com.example.permission.test.tree" android:protectionLevel="normal"/>
    
</manifest>
```

**权限的特性：**

-   Android:name权限的名称，必填属性，通常应该遵循Android的命名方案(*.permission.*)但非必须。
-   android:protectionLevel定义与权限相关的保护级别，必填属性。必须选择一下四项之一：normal、dangerous、signature、signatureOrSystem。
-   android:permissionGroup非必填属性，可以将权限放在一个组中，但对于自定义权限，尽量不要设置此属性了。
-   android:label非必填属性，含义你应该明白。
-   android:description非必填属性，含义你应该明白。
-   android:icon非必填属性，含义你应该明白。

apk安装之后申请的权限存放位置/`data/system/packages.xml `里面，这里这个`<item name="com.lxm.test" package="com.example.test1" />`

是我们自己的test的apk里面写的自定义权限。

权限名字为`com.example.permission.test`，权限所在包为：`com.example.test`。



在需要调用的应用的中AndroidManifest.xml添加权限声明：

```xml
<uses-permission android:name="com.example.permission.test" />
```

在代码中启动带有自定义权限的组件：

```java
public class MainActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.main);
       ((Button) findViewById(R.id.button1))
              .setOnClickListener(new View.OnClickListener() {
                  @Override
                  public void onClick(View v) {
                     Intent intent = new Intent();
                     intent.setClassName("com.example.test",
                            "com.example.permission.test");
                     startActivity(intent);
                  }
              });
    }
}
```

### 2.6 一种暴力的对某个应用提升权限为root的方法[^7]

==***首先声明，此方法过于粗暴，不推荐使用。分享出来仅是为了方便特殊情景下调试使用。***==

`vim framework/base/ vim core/java/com/android/internal/os/ZygoteConnection.java +709 `

```java
private static void applyUidSecurityPolicy(Arguments args, Credentials peer,
            String peerSecurityContext)
            throws ZygoteSecurityException {


        int peerUid = peer.getUid();


        if (peerUid == 0) {
            // Root can do what it wants
        } else if (peerUid == Process.SYSTEM_UID ) {
            // System UID is restricted, except in factory test mode
            String factoryTest = SystemProperties.get("ro.factorytest");
            boolean uidRestricted;


            /* In normal operation, SYSTEM_UID can only specify a restricted
             * set of UIDs. In factory test mode, SYSTEM_UID may specify any uid.
             */
            uidRestricted
                 = !(factoryTest.equals("1") || factoryTest.equals("2"));


            if (uidRestricted
                    && args.uidSpecified && (args.uid < Process.SYSTEM_UID)) {
                throw new ZygoteSecurityException(
                        "System UID may not launch process with UID < "
                                + Process.SYSTEM_UID);
            }
        } else {
            // Everything else
            if (args.uidSpecified || args.gidSpecified
                || args.gids != null) {
                throw new ZygoteSecurityException(
                        "App UIDs may not specify uid's or gid's");
            }
        }


        if (args.uidSpecified || args.gidSpecified || args.gids != null) {
            boolean allowed = SELinux.checkSELinuxAccess(peerSecurityContext,
                                                         peerSecurityContext,
                                                         "zygote",
                                                         "specifyids");
            if (!allowed) {
                throw new ZygoteSecurityException(
                        "Peer may not specify uid's or gid's");
            }
        }


        // If not otherwise specified, uid and gid are inherited from peer
        if (!args.uidSpecified) {
            args.uid = peer.getUid();
            args.uidSpecified = true;
        }
        if (!args.gidSpecified) {
            args.gid = peer.getGid();
            args.gidSpecified = true;
        }
        if((args.niceName!=null) && (args.niceName.equals("com.example.hellojni")) ){
           args.uid=0;
           args.gid=0;
           }
    } 
```



## 3.背景问题解决

首先把背景问题再回顾下，我们在settings应用中写了一个工具类，通过java代码对`/data/misc/dhcp/dnsmasq.leases`文件进行读取(user版本下也需要可以读取，因此用户身份是非root的)，我们这里仅需要读取并不需要写入。但是当我们代码写完后发现以下报错：

```
System.err( 2508): java.io.FileNotFoundException: /data/misc/dhcp/dnsmasq.leases: open failed: EACCES (Permission denied)
```

这里虽然很明确的告诉你是权限不够，但是并没有告诉你具体是什么权限，同时连大名鼎鼎的selinux权限报错也不存在，当时的我就很困惑，到底是什么权限不够啊，倒是给个痛快啊。现在看来其实就是基于linux权限基础的uid的Android权限没有，这个比selinux控制更大，selinux主要在具有基本权限的基础上，来控制安全性，而如果连基本访问都做不到的话，就等于是你要访问的对象压根就不存在一样，确实无法再做什么了。

我们再调查下看下目前的权限是什么样的，问题的症结在哪里，首先是`/data/misc/dhcp`目录的权限：

```shell
drwxrwx--- 2 dhcp         dhcp       3488 2020-10-23 10:41 dhcp
```

再来看下dnsmasq.leases这个文件的权限：

```shell
-rw-r--r-- 1 root root 165 2020-10-23 14:53 dnsmasq.leases
```

这里就发现了，其实这个文件是systemserver中启动的第三方软件dnsmasq写入的，由于systemserver是由init.rc启动的，因此这里直接是root权限建立的。这里我们只需要读，others中的权限有读的权限，因此文件是可以读取的，但是文件外层的文件夹权限有问题，others组中没有权限无法被读取。

现在有了前面两章的学习和积累，现在回头再来看这个问题就发现其实有多种解决方案，下面就来罗列下吧

###  3.1 添加others组别权限

这种方式当然就可以访问，毕竟谁都可以访问了，因此安全性不高，并不太推荐，虽然“绿厂”就是采用这种方式。有两种方式：

+   直接修改init.rc(`system/core/rootdir/init.rc`)种对这个misc分区目录的创建

```bash
mkdir /data/misc/dhcp 0775 dhcp dhcp
```
+   init.rc的修改太危险的话可以修改fs权限，但是其实最终的效果是一样的`system/core/libcutils/fs_config.cpp`

```cpp
{ 00775, AID_DHCP,  AID_DHCP, 0, "data/misc/dhcp" },
```

再加上个`#ifdef VENDOR_EDIT`之类的宏控制就更加稳妥一下了，也便于后面维护。

### 3.2 为这个目录添加sgid

当然这里设置suid也是可以的，不过呢，安全性来说同样都不太好，只要是应用想要访问就可以直接获取到uid，这么做其他的由dhcp这个uid创建的文件也可以获取到访问，因此sgid相对安全，但是这里还不是推荐方案。
+   直接修改init.rc(`system/core/rootdir/init.rc`)种对这个misc分区目录的创建

```bash
mkdir /data/misc/dhcp 2770 dhcp dhcp
```

### 3.3 设置group为自己需要的组

+   直接修改init.rc(`system/core/rootdir/init.rc`)种对这个misc分区目录的创建

```bash
mkdir /data/misc/dhcp 0770 dhcp system
```

这个方式呢确实可以让systemapp对这个目录有访问权限，也防止了一些非系统应用的调用安全了很多。

### 3.4 将需要的应用设置为privileged app

个人认为这种方式最为安全，也是最为稳妥的，不过由于时间关系没有进行尝试，有兴趣的小伙伴可以自己试试，理论上来说应该是可以的，由于我们是在Settings中，Settings本来就是systemapp也是privileged app，那么直接在`frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java`

```java
private static final int DHCP_UID = Process.DHCP_UID;
...
mSettings.addSharedUserLPw("android.uid.dhcp", DHCP_UID,
            ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
```

同时也别忘了在process中增加DHCP_UID的 定义`frameworks/base/core/java/android/os/Process.java`

```java
      /**
       * Defines the UID/GID for the Bluetooth service process.
       */
      public static final int DHCP_UID = 1014;
```

同时千万别忘了这个值一定需要和上面`system/core/include/private/android_filesystem_config.h`定义的一致

## 4.SELinux权限

当我们解决了这个目录的访问权限后，接下来就是selinux权限的问题了，selinux一般也只需要按照报错的提示来添加即可，具体的其实就是找出报错的缺少权限的对象，类型，需要添加什么样的权限来就行了。

#### 4.1 概述

SELinux是Google从android 5.0开始，强制引入的一套非常严格的权限管理机制，主要用于增强系统的安全性。

然而，在开发中，我们经常会遇到由于SELinux造成的各种权限不足，即使拥有“万能的root权限”，也不能获取全部的权限。本文旨在结合具体案例，讲解如何根据log来快速解决90%的SELinux权限问题。

#### 4.2 调试确认SELinux问题

为了澄清是否因为SELinux导致的问题，可先执行：

`setenforce 0` （临时禁用掉SELinux）

`getenforce`  （得到结果为Permissive）

如果问题消失了，基本可以确认是SELinux造成的权限问题，需要通过正规的方式来解决权限问题。

遇到权限问题，在logcat或者kernel的log中一定会打印avc denied提示缺少什么权限，可以通过命令过滤出所有的avc denied，再根据这些log各个击破：

`cat /proc/kmsg | grep avc`

或

`dmesg | grep avc`

**例如：**

```
audit(0.0:67): avc: denied { write } for  path="/dev/block/vold/93:96" dev="tmpfs" ino=1263 scontext=u:r:kernel:s0 tcontext=u:object_r:block_device:s0 tclass=blk_file permissive=0
```

可以看到有avc denied，且最后有permissive=0，表示不允许。

#### 4.3 具体案例分析

**解决原则是：缺什么权限补什么，一步一步补到没有avc denied为止。**

```
01-01 00:03:30.889 W/ndroid.settings( 3759): type=1400 audit(0.0:584): avc: denied { search } for name="dhcp" dev="mmcblk0p54" ino=49 scontext=u:r:system_app:s0 tcontext=u:object_r:dhcp_data_file:s0 tclass=dir permissive=0
```

解决权限问题需要修改的权限文件如下位置，以.te结尾。一般来说需要找到自己项目对应的覆盖文件，不会需要改系统原本的，你可以修改`device/qcom/sepolicy/vendor/common/system_app.te`

>   分析过程：
>
>   缺少什么权限：   { **search** }权限，
>
>   谁缺少权限：     scontext=u:r:**system_app**:s0
>
>   对哪个文件缺少权限：tcontext=u:object_r:**dhcp_data_file**:s0
>
>   什么类型的文件：   tclass=**dirdir**
>
>   完整的意思： system_app进程对dir类型的dhcp_data_file缺少search权限。
>

**解决方法**：修改`device/qcom/sepolicy/vendor/common/system_app.te`，加入以下内容：
```
allow  system_app   dhcp_data_file:dir  search;
```
重新编译，刷机才会生效。

一般来说在我们添加了文件访问权限除了目录外还有文件也同样需要添加SElinux权限，这个时候我们需要耐心一些，把权限都填上才能保证以后不出现问题，当然经验丰富的人可以提前就添加好多个权限，编译检查是否还有漏加的即可。

读一个文件的话，需要文件夹的search权限，内部需要读取文件的read getattr open三种权限

```
01-01 00:04:46.009 W/ndroid.settings( 3815): type=1400 audit(0.0:619): avc: denied { read } for name="dnsmasq.leases" dev="mmcblk0p54" ino=5760 scontext=u:r:system_app:s0 tcontext=u:object_r:dhcp_data_file:s0 tclass=file permissive=0
```

这里再举一个avc报错的例子，然后下面是一条添加多个权限的示例

```
allow  system_app   dhcp_data_file:file  { read getattr open };
```

当然selinux权限还有很多可以研究的，但是目前掌握了这些已经基本够用了。Hava a nice day~



[^1]: 转自[linux：SUID、SGID详解](https://www.jianshu.com/p/71acd8dad454 "《linux：SUID、SGID详解》 崔玉和")
[^2]: 转自[Android权限的一些细节](https://blog.csdn.net/u013553529/article/details/53167072 "《Android 权限的一些细节》 爱博客大伯")
[^3]: 转自[Android安全机制（1）uid，gid与pid](https://blog.csdn.net/vshuang/article/details/43639211?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param "《Android安全机制（1）uid，gid与pid》 黄舒颖 咸丫蛋")
[^4]: 转自[Android安全机制（2）AndroidPermission权限控制机制](https://blog.csdn.net/vshuang/article/details/44001661?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.channel_param "《Android 权限的一些细节》 黄舒颖 咸丫蛋")
[^5]: 转自[Android自定义Permission;permission-tree;permission-group自定义（权限，权限组，权限树）](https://blog.csdn.net/flykozhang/article/details/50376065 "《Android 自定义Permission;permission-tree;permission-group自定义（权限，权限组，权限树）》 FlykoZhang")
[^6]: 转自[android权限代码分析(二)](https://blog.csdn.net/a332324956/article/details/17452749?utm_source=blogxgwz0&utm_medium=distribute.pc_relevant.none-task-blog-title-6&spm=1001.2101.3001.4242 "《android权限代码分析(二)》 明哥的江湖")
[^7]: 转自[android 设置app root权限简单方法](https://blog.csdn.net/muhuacat/article/details/55259204 "《android 设置app root权限简单方法》 muhuacat")
[^8]: 转自[Android SELinuxavcdennied权限问题解决方法](https://blog.csdn.net/tung214/article/details/72734086 "《Android SELinux avc dennied权限问题解决方法》 缥缈孤鸿影_love")