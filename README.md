# linux

## Note

### 安装和更新

```shell
$ apt update //更新系统包索引或包列表
$ apt list --upgradable //查看需更新软件包
$ apt upgrade //升级软件包至最新版本;安装系统所需安全更新
$ apt --only-upgrade install package //指定升级软件包
$ apt install package //安装软件
$ apt list --installed //列出已安装软件
$ apt show package //显示软件包信息
$ apt remove package //卸载软件
```

### 终端命令

```shell
ctrl+l //清屏
ctrl+z //暂停
ctrl+c //中断
ctrl+d //退出shell
ctrl+a //光标切换至行首
ctrl+e //光标切换至行末 
ctrl+k //剪切光标处到行尾的字符
ctrl+u //剪切光标处到行首的字符
ctrl+y //将剪切的字符进行粘贴
$ sudo command //以root身份执行命令
$ su - user //切换用户
$ man command //命令手册;g:跳转到第一行;G:跳转到最后一行;/pattern:从当前往后查询;?pattern:从当前往前查询;n:跳转下一个查询内容;N:跳转上一个查询内容
$ command --help //命令解释
$ history //历史命令,终端关闭后将缓存存至文件~/.bash_history
$ history -c //删除所有历史命令
$ echo $HISTCONTROL //若为ignoreboth,不保存"空格+命令"和重复连续命令
```

### 文件系统

####文件结构

![Linux File System Hierarchy](images/linux-file-system-hierarchy.png)


| 目录                  | 描述                                                                                             |
|:----------------------|:-------------------------------------------------------------------------------------------------|
| `/`                   | 主层次 的根，也是整个文件系统层次结构的根目录。                                                  |
| `/bin`                | 存放在单用户模式可用的必要命令二进制文件，所有用户都可用，如 `cat`、`ls`、`cp`等等。             |
| `/boot`               | 存放引导加载程序文件，例如`kernels`、`initrd`等。                                                |
| `/dev`                | 存放必要的设备文件，例如`/dev/null`。                                                            |
| `/etc`                | 存放主机特定的系统级配置文件。其实这里有个关于它名字本身意义上的的争议。在贝尔实验室的`UNIX`实施文档的早期版本中，`/etc`表示是“其他（`etcetera`）目录”，因为从历史上看，这个目录是存放各种不属于其他目录的文件（然而，文件系统目录标准 `FSH` 限定 `/etc` 用于存放静态配置文件，这里不该存有二进制文件）。早期文档出版后，这个目录名又重新定义成不同的形式。近期的解释中包含着诸如“可编辑文本配置”或者“额外的工具箱”这样的重定义。 |
| `/etc/opt`            | 存储着新增包的配置文件 `/opt/`。                                                                 |
| `/etc/sgml`           | 存放配置文件，比如 `catalogs`，用于那些处理`SGML`(译者注：标准通用标记语言)的软件的配置文件。    |
| `/etc/X11`            | `X Window` 系统`11`版本的的配置文件。                                                            |
| `/etc/xml`            | 配置文件，比如`catalogs`，用于那些处理`XML`(译者注：可扩展标记语言)的软件的配置文件。            |
| `/home`               | 用户的主目录，包括保存的文件，个人配置，等等。                                                   |
| `/lib`                | `/bin/` 和 `/sbin/`中的二进制文件的必需的库文件。                                                |
| `/lib<32/64>`         | 备用格式的必要的库文件。 这样的目录是可选的，但如果他们存在的话肯定是有需要用到它们的程序。      |
| `/media`              | 可移动的多媒体(如`CD-ROMs`)的挂载点。                                                            |
| `/mnt`                | 临时挂载的文件系统。                                                                             |
| `/opt`                | 可选的应用程序软件包。                                                                           |
| `/proc`               | 以文件形式提供进程以及内核信息的虚拟文件系统，在`Linux`中，对应进程文件系统（`procfs`）的挂载点。|
| `/root`               | 根用户的主目录。                                                                                 |
| `/sbin`               | 必要的系统级二进制文件，比如， `init`, `ip`, `mount`。                                           |
| `/srv`                | 系统提供的站点特定数据。                                                                         |
| `/tmp`                | 临时文件 (另见 `/var/tmp`) 通常在系统重启后删除。                                                |
| `/usr`                | 二级层级存储用户的只读数据； 包含(多)用户主要的公共文件以及应用程序。                            |
| `/usr/bin`            | 非必要的命令二进制文件 (在单用户模式中不需要用到的)；用于所有用户。                              |
| `/usr/include`        | 标准的包含文件。                                                                                 |
| `/usr/lib`            | 库文件，用于`/usr/bin/` 和 `/usr/sbin/`中的二进制文件。                                          |
| `/usr/lib<32/64>`     | 备用格式库(可选的)。                                                                             |
| `/usr/local`          | 三级层次 用于本地数据，具体到该主机上的。通常会有下一个子目录, 比如, `bin/`, `lib/`, `share/`。  |
| `/usr/local/sbin`     | 非必要系统的二进制文件，比如用于不同网络服务的守护进程。                                         |
| `/usr/share`          | 架构无关的 (共享) 数据。                                                                         |
| `/usr/src`            | 源代码，比如内核源文件以及与它相关的头文件。                                                     |
| `/usr/X11R6`          | `X Window`系统，版本号:`11`，发行版本：`6`。                                                     |
| `/var`                | 各式各样的（`Variable`）文件，一些随着系统常规操作而持续改变的文件就放在这里，比如日志文件，脱机文件，还有临时的电子邮件文件。 |
| `/var/cache`          | 应用程序缓存数据`.` 这些数据是由耗时的`I/O`(输入`/`输出)的或者是运算本地生成的结果。这些应用程序是可以重新生成或者恢复数据的。当没有数据丢失的时候，可以删除缓存文件。 |
| `/var/lib`            | 状态信息。这些信息随着程序的运行而不停地改变，比如，数据库，软件包系统的元数据等等。             |
| `/var/lock`           | 锁文件。这些文件用于跟踪正在使用的资源。                                                         |
| `/var/log`            | 日志文件。包含各种日志。                                                                         |
| `/var/mail`           | 内含用户邮箱的相关文件。                                                                         |
| `/var/opt`            | 来自附加包的各种数据都会存储在 `/var/opt/`。                                                     |
| `/var/run`            | 存放当前系统上次启动以来的相关信息，例如当前登入的用户以及当前运行的`daemons`(守护进程)。        |
| `/var/spool`          | 该`spool`主要用于存放将要被处理的任务，比如打印队列以及邮件外发队列。                            |
| `/var/mail`           | 过时的位置，用于放置用户邮箱文件。                                                               |
| `/var/tmp`            | 存放重启后保留的临时文件。                                                                       |

```shell
$ pwd //print working directory 显示当前工作路径
$ cd /Dir //change directory ~:当前用户家目录; ..:上一级目录; -:返回上一次所在目录

$ ls [OPTION]... [FILE]...//list 列出目录内容 -l:显示详细信息; -a:显示所有文件,包括.开头的隐藏文件; -h:将大小表示为K M G 等; -S:按文件大小排序; -t:按修改时间排序; -r:排序是逆序排序; -R:递归显示所有子文件
$ ls -alh
total 3.9G
drwxr-xr-x  19 root root 4.0K 11月 23 22:47 .
drwxr-xr-x  19 root root 4.0K 11月 23 22:47 ..
lrwxrwxrwx   1 root root    7  8月 10 08:18 bin -> usr/bin
drwxr-xr-x   5 root root 4.0K 11月 23 23:29 boot
drwxr-xr-x  20 root root 4.1K 11月 30 23:11 dev
drwxr-xr-x 140 root root  12K 11月 28 22:09 etc
drwxr-xr-x   3 root root 4.0K 11月 23 22:50 home
lrwxrwxrwx   1 root root    7  8月 10 08:18 lib -> usr/lib
drwx------   2 root root  16K 11月 23 22:43 lost+found
drwxr-xr-x   3 root root 4.0K 11月 28 22:24 media
drwxr-xr-x   2 root root 4.0K  8月 10 08:18 mnt
drwxr-xr-x   2 root root 4.0K  8月 10 08:18 opt
dr-xr-xr-x 267 root root    0 11月 30 22:33 proc
drwx------   5 root root 4.0K 11月 24 00:23 root
drwxr-xr-x  40 root root 1.1K 11月 30 23:35 run
lrwxrwxrwx   1 root root    8  8月 10 08:18 sbin -> usr/sbin
drwxr-xr-x  14 root root 4.0K 11月 30 23:11 snap
drwxr-xr-x   2 root root 4.0K  8月 10 08:18 srv
-rw-------   1 root root 3.9G 11月 23 22:45 swap.img
dr-xr-xr-x  13 root root    0 11月 30 22:33 sys
drwxrwxrwt  19 root root 4.0K 11月 30 23:12 tmp
drwxr-xr-x  11 root root 4.0K  8月 10 08:18 usr
drwxr-xr-x  14 root root 4.0K 11月 23 23:28 var

$ stat [OPTION]... FILE... //status 显示文件状态信息
$ stat auth.log 
  文件：auth.log
  大小：90780     	块：192        IO 块大小：4096   普通文件
设备：fd00h/64768d	Inode：527843      硬链接：1
权限：(0640/-rw-r-----)  Uid: (  107/  syslog)   Gid: (    4/     adm)
访问时间：2023-11-23 22:50:33.196000002 +0800
修改时间：2023-11-30 23:18:15.926576196 +0800
变更时间：2023-11-30 23:18:15.926576196 +0800
创建时间：2023-11-23 22:50:33.196000002 +0800

$ file [OPTION]... FILE... //识别文件类型
$ file auth.log 
auth.log: data
```

