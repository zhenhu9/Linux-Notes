
------

## 软件包管理

为了满足应用的需求，我们通常会在系统上安装各种相应的软件，并在维护中会对软件进行升级和删除，甚至是退回到旧的版本。 CentOS 中主要有三种类型的软件管理方式，分别为源代码编译、RPM 包管理和 YUM 管理方式。其中 YUM 管理方式是对用户最友好的方式，其次为 RPM 方式，最不友好的是源代码编译方式。源代码编译也可生成 RPM 包，而 YUM 解决了 RPM 包的资源依赖问题。

“RPM Package Manager” 是 RPM 的递归缩写，也就是 RPM 软件包管理器。RMP 包管理器可以生成 RMP 软件包、安装、查询、升级、卸载、校验软件包和对 RPM 数据库进行管理。

RPM 软件包是以 .rpm 为扩展名，其命名规范为（NEVRA）：

name-[epoch:]version-release.os.arch.rpm

其中 name 为程序名称； version 为程序版本号； epoch 为程序版本号一部分； release 为发行号，用于标识 RPM 包本身的发行号，与源程序的 release 号无关； os 为 RPM 包支持的操作系统版本，如 el6、el7、el8 等； arch 为系统架构，如 i686、x86_64、amd64、ppc（power-pc）或 noarch（即不依赖平台）等。

例如：

```
openssh-server-8.0p1-4.el8_1.x86_64
|------1------|--2--|3|--4--|--5--|

1-->name  2--> version  3-->release  4-->os  5-->arch

openssh-server-0:8.0p1-4.el8_1.x86_64
              |6|
              |---2---|

6-->epoch
```

RPM 拥有一组工具来实现包管理器的管理功能。虽然打包系统定义了一种依赖模型，可以查询到所依赖的软件和库，但有时会陷入令人头痛的包依赖地狱（package dependency hell）。 高级包管理系统 YUM 此时就可以大放光彩了，其可以自动下载软件包并解决包依赖问题。在 CentOS 8 中开始启用 DNF 代替 YUM，这两个命令的大部分选项是兼容的，所以就不必太过担心了。

高级包管理系统 YUM 的两大基础设施，分别为软件包仓库和与软件包操作相关的命令集，以实现软件包的安装、查询、升级、卸载、校验软件包和对安装包数据库进行管理。


YUM/DNF 配置文件表

|   工具 |   YUM    |    DNF   |    描述    |
| ------ | -------- | -------- | ---------- |
| 主配置文件 | /etc/yum.conf   | /etc/dnf/dnf.conf | 全局配置文件。 |
|   库文件   | /etc/yum.repos.d  | /etc/yum.repos.d | 库配置文件。  |
| 插件配置文件 | /etc/yum/pluginconf.d/ | /etc/dnf/plugin/ | 插件配置文件。|
|  受保护的软件包列表 | /etc/yum/protected.d/  | /etc/dnf/protected.d | 防止被插件或作为 obsoletes 被删除。|
|  定义使用的变量 | /etc/yum/vars/  | /etc/dnf/vars/ | 变量。|
|  缓存文件  | /var/cache/yum	 | /var/cache/dnf   | 缓存文件路径。 |





CentOS 软件包仓库库配置文件位于 /etc/yum.repo.d 内。

```
$ yum repolist all			/* 列表系统所有仓库。

repo id /* 仓库 ID      repo name  /* 仓库名称        status    /* 仓库状态
AppStream               CentOS-8 - AppStream        enabled   /* 仓库开启
AppStream-source        CentOS-8 - AppStream Source disabled  /* 仓库关闭
BaseOS                  CentOS-8 - Base             enabled
BaseOS-source           CentOS-8 - BaseOS Sources   disabled
Devel                   CentOS-8 - Devel WARNING! F disabled
HighAvailability        CentOS-8 - HA               disabled
PowerTools              CentOS-8 - PowerTools       disabled
base-debuginfo          CentOS-8 - Debuginfo        disabled
c8-media-AppStream      CentOS-AppStream-8 - Media  disabled
c8-media-BaseOS         CentOS-BaseOS-8 - Media     disabled
centosplus              CentOS-8 - Plus             disabled
centosplus-source       CentOS-8 - Plus Sources     disabled
cr                      CentOS-8 - cr               disabled
...
```

列表的内容与 /etc/yum.repo.d 内的仓库库配置文件 .repo 相对应。仓库库配置文件常用参数如下：

```
$ cat /etc/yum.repos.d/CentOS-Base.repo

# 井号起始的行为注释内容。
# 远程仓库系统会根据您的 IP 地址自动选择相匹配的地理区域的仓库镜像。

[BaseOS]				/* 仓库 ID。必须唯一。
name=CentOS-$releasever - Base		/* 仓库名称。
/* 仓库镜像列表地址。
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=BaseOS&infra=$infra
/* 仓库地址。
#baseurl=http://mirror.centos.org/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1				/* gpg 测试，以验证包的完整。
enabled=1				/* 是否开启此仓库。
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial	/* gpg 公钥位置。
```

除了 CentOS 官方提供的仓库外，还可以使用第三方仓库。

CentOS 官方仓库及其推荐的第三方仓库介绍：

https://wiki.centos.org/action/show/AdditionalResources/Repositories?action=show&redirect=Repositories。

CentOS 软件仓库在各地理区域的镜像（含国内）：

```
CentOS 6	http://isoredirect.centos.org/centos/6/isos/
CentOS 7	http://isoredirect.centos.org/centos/7/isos/
CentOS 8	http://isoredirect.centos.org/centos/8/isos/
```

直接取 CentOS 软件仓库镜像的主域名，就可以看到各自网站提供的所有软件仓库镜像。

```
中国科技大学镜像源：https://mirrors.ustc.edu.cn/
清华大学镜像源： https://mirrors.tuna.tsinghua.edu.cn/
阿里云镜像源：https://developer.aliyun.com/mirror/
华为镜像源：https://mirrors.huaweicloud.com/
网易镜像源：https://mirrors.163.com/
```

为了提高软件包下载速度，我们还可以直接设置仓库在国内镜像的地址。

示例：

设置远程仓库：

```
/* 添加 docker-ce 仓库网易镜像。
$ sudo wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.163.com/docker-ce/linux/centos/docker-ce.repo
 
$ cat /etc/yum.repos.d/docker-ce.repo

[docker-ce-stable]
name=Docker CE Stable - $basearch
#baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
baseurl=https://mirrors.163.com/docker-ce/linux/centos/$releasever/$basearch/stable

/* 注意 baseurl 两行的对应关系与仓库网址对应关系。可以很方便的设置为任何仓库源。

enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
...
```

在本地创建远程仓库镜像：

```
/* 在系统更新后，同步一个远程仓库到本地，再设置本地仓库。

/* centos 7 
$ sudo yum install yum-utils createrepo		/* 安装 yum 插件。
$ sudo mkdir /data/repos			
$ sudo reposync -r docker-ce-stable -p /data/repos
/* reposync 同步仓库到本地， -r 指定同步的仓库 id，-p 同步数据到指定位置。
$ sudo createrepo /data/repos/docker-ce-stable
/* 创建仓库 xml 索引，也就是 metadata。

/* centos 8 
$ sudo dnf install createrepo_c
$ sudo mkdir /data/repos
$ sudo dnf reposync --repo docker-ce-stable -p /data/repos
$ sudo createrepo_c /data/repos/docker-ce-stable
```

设置直接使用本地仓库：

```
$ sudo cp  /etc/yum.repos.d/docker-ce.repo  /etc/yum.repos.d/local-docker-ce.repo
$ sudo vim  /etc/yum.repos.d/local-docker-ce.repo

[docker-ce-stable]
name=Docker CE Stable - $basearch
#baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
baseurl=file:///data/repos/docker-ce-stable/linux/centos/$releasever/$basearch/stable
...
```

CentOS 的其它方提供的各种软件仓库（非官方仓库）：

https://wiki.centos.org/action/show/AdditionalResources/Repositories?action=show&redirect=Repositories

------

dnf 命令

dnf [options] command [package ...]

通用选项：

```
--security	/* 指定有关于安全更新，应用于安装、查询或升级相关的命令。
--bugfix	/* 指定有关于 BUG 修复。

-v				/* 繁琐模式。
[-y | --assumeyes]		/* 命令执行时的交互都选择 Y。
-h				/* 帮助。
```

与库相关的命令和选项：

```
repolist [all | enabled | disabled]			/* 列表所有/开启/关闭的仓库。
repoinfo [all | enabled | disabled | <repo_id> ...]	/* 查询所有/开启/关闭/指定的仓库的信息。

clean [packages | metadata | dbcache | expire-cache | all]
/* 清理系统下载的软件包/仓库元数据/仓库 sqlite 库数据/使缓存数据过期/清理所有数据。缓存目录位于 /var/cache/[yum | dnf]。

makecache [fast]		/* 下载所有已启用仓库的元数据，如果使用 fast，则相当于执行 yum clean expire-cache。
```

与检测相关的选项：

```
check-update			/* 检查是否有更新。

/* 检测本地软件包数据库及产品信息，来查看是否有问题。默认检测所有问题，也可指定问题类型。
check	[--dependencies] [--duplicates] [--obsoleted] [--provides]
```

与安装、重装、更新、更新到或移除相关的命令：

```
install [package ...]		/* 安装一个或多个软件包，用空格分隔软件包名称。
	/path/whatever/nvra.rpm		/* 安装资源的多种形式。
	name.version.release
	name
	networkpath/path/nvra.rpm
	'@group'			/* 安装包组。
	/path/command			/* 安装提供指定命令的软件包。
	--advisory=advisory-id \*	/* 安装属于指定建议的所有软件包。

reinstall [package ...]		/* 重新安装一个或多个软件包。
update [package ...]		/* 更新指定的一个、多个软件包或所有可以更新的软件包。
update-to [package ...]		/* 更新一个或多个指定的软件到指定版本的软件包。

remove | erase [package ...]	/* 删除一个或多个软件包。
	--duplicates		/* 重复的软件包。
	--oldinstallonly	/* 删除 installonly 软件包，仅保留最新版本的软件包和内核。
```

与软件包组相关的命令：

```
group [groups_option]		/* 软件包组的管理，类似软件包管理。
	list [installed|available|environment|language|package|hidden|ids]
				/* 列表软件包组，可以指定包组类型。默认不会列表隐藏包组，须指定 hidden 选项。
	install			/* 安装软件包组。
	remove			/* 删除软件包组，其会删除软件包组包含的所有软件包，无论其它软件包是否依赖所删除的。
	info			/* 查看软件包组信息。
```

列表系统中各类软件包信息：

```
list [list_option]		/* 列表指定软件包。
	[--all | glob_exp] [...]	/* 列表所有软件包或与 glob 表达式匹配的软件包。
	--installed [glob_exp] [...]	/* 列表已安装的软件包。
	--available [glob_exp] [...]	/* 列表可供安装的软件包。
	--upgrades [glob_exp] [...]	/* 列表可更新的软件包。
	--extras [glob_exp] [...]	/* 列表已安装，但不在已知库的软件包。
	--obsoletes [glob_exp] [...]	/* 列表已标识为废弃的软件包。
	--recent [glob_exp] [...]	/* 列表最近添加到仓库的软件包。
	--autoremove [glob_exp] [...]	/* 列表可以被 autoremove 的软件包。
```

查询软件包的详细信息：

```
info [list_option]			/* 查询指定软件包的信息
	/*info 可以使用与 list_option 同样的选项，但一般直接查询指定软件包的信息。
```

查询软件包管理事物信息:

```
history 			/* 软件包管理事务历史 
	[list] [<trans-id> ...]	/* 列表所有或指定管理事务历史，含事务 ID、命令行、日期、动作和所作的改动。 
	info <trans-id> ...	/* 查询指定事务详细信息。
	redo <trans-id>		/* 重做指定事务。
	rollback <trans-id>	/* 回滚到指定事务。
	undo <trans-id>		/* 撤销指定事务。
	userinstalled		/* 列表 installonly 软件包。不作为依赖安装的和 DNF 之外安装的软件包。
```

管理模组：

```
module				/* 模组是类似于软件包组的另一种形式。
	list			/* 列表模组。
		[--all] [module_name...]	/* 所有或指定模组。
		[--enabled] [module_name...]	/* 启用的及指定模组。
		[--disabled] [module_name...]	/* 关闭的及指定模组。
		[--installed] [module_name...]	/* 已安装的及指定模组。

	install <glob_exp>...	/* 安装模组。
	update <glob_exp>...	/* 更新模组。
	remove <glob_exp>...	/* 移除模组。
	enable <glob_exp>...	/* 开启模组。
	disable <glob_exp>...	/* 关闭模组。

	info <glob_exp>...	/* 模组信息。
		--profile <glob_exp>...		/* 节选 profile 信息。
```

查询 DNF 仓库中软件包信息：

```
repoquery [<select-options>] [<query-options>] <glob_exp>...

repoquery --querytags		/* 提供可以被 --queryformat 识别的查询标签。
	  --queryformat "%{querytag} ..."

/* select-option
	-a, --all		/* 查询所有的软件包。
	--duplicates		/* 重复的软件包。
	--unneeded		/* 作为依赖安装，但不再需要的软件包。
	--extras		/* 不在可供使用的库中的软件包。
	--f <file>, --file <file>	/* 拥有指定文件的软件包。
	--installonly		/* installonly 软件包。
	--upgrades		/* 对已安装软件包提供更新的软件包。
	--userinstalled		/* 用户安装的软件包。

/* query-option
	--list			/* 查询软件包具有那些文件。
	--location		/* 可以从何处下载软件包。
	--pf <format>, --queryformat <format>		/* 查询指定标记。
	/* 大部分选项可以使用 --queryformats "%{querytags}" 替代。
```

搜索及其它命令：

```
search [keywords]		/* 查找软件包，可以指定部分字符串来查找。

deplist	[package] [...]		/* 列表指定软件包依赖。

autoremove <glob_exp>...	/* 删除那些不再需要的仅作为依赖的程序包。

provides | whatprovides	[command | package]	/* 查询指定的命令或软件包由谁提供。

/* 将所有或指定的已安装软件包与库中的最新版本软件包同步。
distro-sync [glob_exp...]
```


示例：

```
$ sudo yum --security check-update		/* 检查是否有关于安全的更新。

$ sudo yum update				/* 安装所有更新。

$ sudo yum history list				/* 查询安装历史。	

ID     | Command line              | Date and time    | Action(s)      | Altered
--------------------------------------------------------------------------------
    44 | install vsftpd            | 2020-12-15 21:54 | Install        |    1   
    43 |                           | 2020-12-15 17:30 | Upgrade        |    2   
    42 | update                    | 2020-12-12 16:19 | Upgrade        |    1   
...

$ sudo yum history info 44			/* 查询事务 ID 44 详细信息。

Transaction ID : 44
Begin time     : Tue 15 Dec 2020 09:54:33 PM CST
Begin rpmdb    : 2115:61986a56c82aec41b92fb2b3bc09ce57dd092e60
End time       : Tue 15 Dec 2020 09:54:34 PM CST (1 seconds)
End rpmdb      : 2116:146141955c021eb24181666635dc6b4293874964
User           :  <haojiang>
Return-Code    : Success
Releasever     : 8
Command Line   : install vsftpd
Comment        : 
Packages Altered:
    Install vsftpd-3.0.3-32.el8.x86_64 @appstream

$ sudo yum remove vsftpd			/* 删除 vsftpd。

Dependencies resolved.
================================================================================
 Package        Architecture   Version                 Repository          Size
================================================================================
Removing:
 vsftpd         x86_64         3.0.3-32.el8            @appstream         343 k

Transaction Summary
================================================================================
Remove  1 Package

Freed space: 343 k
Is this ok [y/N]: y
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Running scriptlet: vsftpd-3.0.3-32.el8.x86_64                             1/1 
  Erasing          : vsftpd-3.0.3-32.el8.x86_64                             1/1 
  Running scriptlet: vsftpd-3.0.3-32.el8.x86_64                             1/1 
  Verifying        : vsftpd-3.0.3-32.el8.x86_64                             1/1 
Installed products updated.

Removed:
  vsftpd-3.0.3-32.el8.x86_64                                                    

Complete!

$ sudo yum history redo 44			/* 重做事务 ID 44。

Last metadata expiration check: 0:23:16 ago on Tue 15 Dec 2020 09:43:53 PM CST.
Repeating transaction 44, from Tue 15 Dec 2020 09:54:33 PM CST
    Install vsftpd-3.0.3-32.el8.x86_64 @appstream
================================================================================
 Package         Architecture    Version               Repository          Size
================================================================================
Installing:
 vsftpd          x86_64          3.0.3-32.el8          appstream          180 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 180 k
Installed size: 343 k
Is this ok [y/N]: y
Downloading Packages:
vsftpd-3.0.3-32.el8.x86_64.rpm                  308 kB/s | 180 kB     00:00    
--------------------------------------------------------------------------------
Total                                           112 kB/s | 180 kB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : vsftpd-3.0.3-32.el8.x86_64                             1/1 
  Running scriptlet: vsftpd-3.0.3-32.el8.x86_64                             1/1 
  Verifying        : vsftpd-3.0.3-32.el8.x86_64                             1/1 
Installed products updated.

Installed:
  vsftpd-3.0.3-32.el8.x86_64     

$ whereis vsftpd		/* 查询 vsftpd 位置。

vsftpd: /usr/sbin/vsftpd /etc/vsftpd /usr/share/man/man8/vsftpd.8.gz

$ sudo yum provides /usr/sbin/vsftpd	/* 查询指定命令由那个软件包提供。

Last metadata expiration check: 6 days, 1:41:12 ago on Wed 09 Dec 2020 08:43:00 PM CST.
vsftpd-3.0.3-32.el8.x86_64 : Very Secure Ftp Daemon
Repo        : @System
Matched from:
Filename    : /usr/sbin/vsftpd

vsftpd-3.0.3-32.el8.x86_64 : Very Secure Ftp Daemon
Repo        : appstream
Matched from:
Filename    : /usr/sbin/vsftpd

$ sudo yum deplist vsftpd			/* 查询指定软件的依赖。

CentOS Linux 8 - AppStream                      3.8 kB/s | 4.3 kB     00:01    
CentOS Linux 8 - BaseOS                         4.7 kB/s | 3.9 kB     00:00    
CentOS Linux 8 - Extras                         1.5 kB/s | 1.5 kB     00:01    
CentOS Linux 8 - Extras                         7.6 kB/s | 8.6 kB     00:01    
...
package: vsftpd-3.0.3-32.el8.x86_64
  dependency: /bin/bash
   provider: bash-4.4.19-12.el8.x86_64
  dependency: /bin/sh
   provider: bash-4.4.19-12.el8.x86_64
  dependency: libc.so.6(GLIBC_2.28)(64bit)
   provider: glibc-2.28-127.el8.x86_64
  dependency: libcap.so.2()(64bit)
   provider: libcap-2.26-4.el8.x86_64
  dependency: libcrypto.so.1.1()(64bit)
   provider: openssl-libs-1:1.1.1g-11.el8.x86_64
...
```

注：如未特别指出，以上 yum 命令参数与 dnf 命令参数通用。

```
/* Centos 7 用 yum 下载软件包。如果加 --resolve，还将下载依赖。
$ yumdownloader --destdir /var/tmp openssh

/* Centos 8 用 yum/dnf 下载软件包。*如果加 --resolve，还将下载依赖。
$ [yum | dnf] download --destdir /var/tmp openssh
```

------

rpm 命令

```
GENERAL OPTIONS

-v			/* 繁琐模式。
-vv			/* 输出大量的调试信息。

INSTALL AND UPGRADE OPTIONS

{-i | --install}	/* 安装指定软件包。
{-U | --upgrade}	/* 更新或安装指定软件包。其更新过程其实是先删除再安装。
{-F | --freshen}	/* 仅升级已安装的软件包。
	-h		/* 使用 # 字符显示安装或更新进度。
	--percent	/* 解包文档时，输出百分比进度。
	--oldpackage	/* 使用指定旧版本软件包更新。
	--test		/* 仅作测试和报告潜在冲突。

ERASE OPTIONS

{-e | --erase}		/* 删除指定软件包。
	--test		/* 仅作测试，与 -vv 结合使用对调试很有用。

QUERY OPTIONS

{-q | --query}		/* 查询指定软件包信息。
	[package-select-option]
	{-a | --all}	/* 查询所有已安装的软件包。
	{-p | --package} package_file	/* 查询指定软件包。
	{-f | --file} file		/* 查询指定文件属于哪个软件包。

	--fileprovide	/* 列表指定文件由哪些文件提供。
	--filerequire	/* 列表指定文件依赖哪些文件。

	[package-query-option]
	{-i | --info}	/* 查询指定软件包的信息。

	{-l | --list}	/* 列表指定软件包内的文件。

	--provides	/* 列表指定软件包提供哪些功能。
	--whatrequires capability	/* 查询哪些软件包依赖指定的功能（软件包名称）。
	{-R | --requires}	/* 查询指定软件包的依赖功能。
	{-c | --configfiles}	/* 列表指定软件包的配置文件。

	--scripts	/* 查询指定软件包中用来安装和删除的脚本。

--querytags	/* 列表 rpm 可以查询的标记。
--qf, --queryformat <queryformat>	/* 指定查询的标记。

VERIFY OPTIONS

{-V | --verify}		/* 使用 rpm 数据库内的元数据来验证软件包。

/* 验证输出为 9 个字符的字符串，每个字符串含义如下：

c %config	配置文件。
d %doc		文档文件。
g %ghost	文件内容不是软件包包含的。
l %license	版权文件。
r %readme	说明文档。

. 		测试通过。 
S		文件大小不同。
M		文件权限和类型不同。
5		MD5 不匹配。
D		主次设备编号不匹配。
L		readlink(2) 路径不匹配。
U		用户属主不同。
G		组不同。
T		修改时间不同。
P		功能不匹配。

-K		/* 公钥验证。

```

示例：

```
$ rpm -qa | grep openssh	/* 查询所有已安装的软件包，grep 匹配 openssh。

openssh-server-8.0p1-5.el8.x86_64
openssh-clients-8.0p1-5.el8.x86_64
openssh-8.0p1-5.el8.x86_64
openssh-askpass-8.0p1-5.el8.x86_64

$ rpm -q vsftpd			/* 查询是否安装了 vsftpd 软件包。

/* 验证软件包签名。
$ rpm -K  http://mirrors.aliyun.com/centos/8.3.2011/AppStream/x86_64/os/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm

http://mirrors.aliyun.com/centos/8.3.2011/AppStream/x86_64/os/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm: digests signatures OK

/* 安装相应远程仓库的 vsftpd 软件包。
$ sudo rpm http://mirrors.aliyun.com/centos/8.3.2011/AppStream/x86_64/os/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm 

$ sudo rpm -qi vsftpd		/* 查询相关信息。

Name        : vsftpd
Version     : 3.0.3
Release     : 32.el8
Architecture: x86_64
Install Date: Tue 15 Dec 2020 09:21:09 PM CST
Group       : System Environment/Daemons
Size        : 351530
License     : GPLv2 with exceptions
Signature   : RSA/SHA256, Wed 29 Apr 2020 12:08:42 AM CST, Key ID 05b555b38483c65d
Source RPM  : vsftpd-3.0.3-32.el8.src.rpm
Build Date  : Mon 27 Apr 2020 10:04:03 AM CST
Build Host  : x86-01.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : https://security.appspot.com/vsftpd.html
Summary     : Very Secure Ftp Daemon
Description :
vsftpd is a Very Secure FTP daemon. It was written completely from
scratch.

$sudo rpm -e vsftpd		/* 删除指定软件。

$sudo rpm -ql vsftpd		/* 查询指定的软件包含的文件。 

/etc/logrotate.d/vsftpd
/etc/pam.d/vsftpd
/etc/vsftpd
/etc/vsftpd/ftpusers
/etc/vsftpd/user_list
/etc/vsftpd/vsftpd.conf
/etc/vsftpd/vsftpd_conf_migrate.sh
/usr/lib/.build-id
...
```

------

编译安装

一般来说，当我们下载了程序源代码后，其内会提供编译安装所需的相应文件，如 README、INSTALL、configure、Makefile 等文件。README 自述文件介绍软件及其安装方式；INSTALL 一般是安装方式介绍；configure 是用于生成 Makefile 的脚本。Makefile 是编译和安装的特定格式文件。源代码编译所需的依赖文件，如头文件、库文件、所依赖的程序和编译套件就必须自行安装了。另外编译安装的软件并非比软件包安装的好，但有时软件安装包发行的比较慢，就可以自行编译安装了。

configure 脚本

```
--help		/* 帮助。 
--prefix	/* 指定安装目录。
--with-***	/* 指定可选模块。
```

示例：

```
/* 下载 git 源代码。
$ wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.xz
/* 解源代码压缩包。
$ tar -xvf git-2.9.5.tar.xz

$ cd git-2.9.5

$ less README		/* 查看自述文件。
$ less INSTALL		/* 查看安装文件。

$ sudo yum install make gcc -y		/* 安装编译套件。
/* 安装编译依赖。
$ sudo yum install sudo dnf install dh-autoreconf curl-devel expat-devel gettext-devel openssl-devel perl-devel zlib-devel asciidoc xmlto

/* 指定安装目录为 /usr/local/git，方便以后删除或升级。
$ make prefix=/usr/local/git all doc
$ sudo make prefix=/usr/local/git install install-doc install-html

/* 把编译安装的 git 路径放到环境变量里，修改 .bashrc 文件。
$ vim ~.bashrc
export PATH=/usr/local/git/bin:$PATH	/* 修改最后一行。

$ source ~.bashrc			/* 使修改立刻生效。
```

------
