# linux

## Note

### 安装和更新

```shell
$ apt update # 更新系统包索引或包列表
$ apt list --upgradable # 查看需更新软件包
$ apt upgrade # 升级软件包至最新版本;安装系统所需安全更新
$ apt --only-upgrade install package # 指定升级软件包
$ apt install package # 安装软件
$ apt list --installed # 列出已安装软件
$ apt show package # 显示软件包信息
$ apt remove package # 卸载软件
```

### 终端命令

```shell
ctrl+l # 清屏
ctrl+z # 暂停
ctrl+c # 中断
ctrl+d # 退出shell
ctrl+a # 光标切换至行首
ctrl+e # 光标切换至行末 
ctrl+k # 剪切光标处到行尾的字符
ctrl+u # 剪切光标处到行首的字符
ctrl+y # 将剪切的字符进行粘贴
$ sudo command # 以root身份执行命令
$ su - user # 切换用户
$ man command # 命令手册;g:跳转到第一行;G:跳转到最后一行;/pattern:从当前往后查询;?pattern:从当前往前查询;n:跳转下一个查询内容;N:跳转上一个查询内容
$ command --help # 命令解释
$ history # 历史命令,终端关闭后将缓存存至文件~/.bash_history
$ history -c # 删除所有历史命令
$ echo $HISTCONTROL # 若为ignoreboth,不保存"空格+命令"和重复连续命令
```

### 重定向(>)和管道(|)

#### 文件描述符

Linux的宗旨是一切皆文件，对于进程、IO等等都是通过文件的形式存在，这些文件都通过文件描述符的形式来表示。Linux的文件描述符可以理解为Linux为了跟踪一个打开的文件而分配的唯一标号，可以通过这个标号对文件实现读写操作。Linux系统的文件描述符一般都有最大的限制，可以通过`ulimit -n`这条命令来查看。如果想改变这个限制，需要修改/etc/security/limits.conf 文件，如下：

```shell
$ vi /etc/security/limits.conf
* hard nofile 102400
* soft nofile 102400
```

Linux启动时，最开始会创建init进程，其余的程序都是这个进程的子进程。而init进程默认打开3个文件描述符：

- 标准输入：standard input(0)
- 标准输出：standard output(1)
- 标准错误：standard error(2)

其中标准输入就是键盘，标准输出和标准错误都是屏幕，后面的数字分别是他们的文件描述符。

对于每个Linux进程，其都是init的子进程，包括bash命令窗口，而其中执行的shell命令，则更是如此。在Linux中，子进程会继承父进程的文件描述符，所以说，Linux中每个程序，执行的每个shell命令，拥有这三个文件描述符，而程序后续打开的文件，其文件描述符则（从3开始）依次增加。

对于一条shell命令，其从标准输入（键盘）中获得输入，如果执行成功，则将输出打印在标准输出（屏幕）上；如果执行出错，将结果打印在标准错误（屏幕）上。

#### 重定向

根据前文，命令的输入输出默认的文件描述符，如果不使用默认的文件描述符，而想自己指定，则需要用到重定向。

##### 输出重定向

```
command [1-n] > file或者文件描述符或者设备
```

下面示例输出重定向的操作，假设当前目录下只存在一个文件exists.txt。

- 不重定向输出

  执行命令:

  ```shell
  $ ls exists.txt no-exists.txt
  ```

  因为exists.txt文件存在，而no-exists.txt文件不存在，因此这个命令即具有成功执行的结果（标准输出），又具有出错的结果（标准错误）。由于这个命令没有进行重定向，因此标准输出和标准错误都将打印在屏幕上：

  ```shell
  ls: no-exists.txt: No such file or directory  # 执行错误，标准错误
  exists.txt  # 成功执行，标准输出
  ```

- 重定向输出

  执行命令：

  ```shell
  $ ls exists.txt no-exists.txt 1 > success.txt 2 > fail.txt
  ```

  命令执行，屏幕上将不显示任何信息，但是多了两个文件，其中succcess.txt中是执行成功的结果，标准输出重定向的文件，内容为`exists.txt`，而fail.txt是执行出错的结果，标准错误重定向的结果，内容为`ls: no-exists.txt: No such file or directory`。

- 只重定向标准输出

  执行命令：

  ```shell
  $ ls exists.txt no-exists.txt  > result.txt
  ```

  这个应该是我们平时用的最多的形式了，其意义就是将命令执行成功的结果输出到result.txt中，因此屏幕上没有命令执行成功的结果，只有出错的结果。值得注意的是，**标准输出的文件描述符1可以省略**

  同理，我们也可以只重定向标准错误的结果，不过文件描述符2不能省略。

- 关闭标准错误

  将标准错误信息关闭掉：

  ```shell
  $ ls exists.txt no-exists.txt 2 > /dev/null
  $ ls exists.txt no-exists.txt 2 > &-
  ```

  /dev/null 这个设备，是linux 中黑洞设备，什么信息只要输出给这个设备，都会给吃掉。&代表当前命令进程中是已经存在的文件描述符，&1代表标准输出，因为1可以省略，所以&也代表标准输出，&2代表标准错误，&-代表关闭与它绑定的描述符。重定向符号后面的文件描述符用&引用。

- 关闭所有输出

  和上面类似：

  ```shell
  $ ls exists.txt no-exists.txt 1 > /dev/null 2 > /dev/null
  $ ls exists.txt no-exists.txt 1 > &- 2 > &-
  ```

- 将标准错误绑定给标准输出

  ```shell
  $ ls exists.txt no-exists.txt 2 > &1 1 > /dev/null
  ```

  将标准错误绑定给标准输出，然后将标准输出发送给/dev/null设备，常用

**重定向输出还存在以下需要注意的地方：**

```
* shell遇到`>`操作符，会判断右边文件是否存在，如果存在就先删除，并且创建新文件。不存在直接创建。 无论左边命令执行是否成功。右边文件都会存在。

* `>>`操作符，判断右边文件，如果不存在，先创建。如果存在，以添加方式打开文件，会分配一个文件描述符[不特别指定，默认为1,2]然后，与左边的标准输出（1）或标准错误（2） 绑定。

* 一条命令在执行前，先会检查输出是否正确，如果输出设备错误，将不会进行命令执行
```

##### 输入重定向

```
command [n] < file或文件描述符或者设备
```

示例：

```shell
$ cat > output.txt  < input.txt
```

这条命令cat命令的输入重定向到input.txt文件，因此该文件的内容作为cat的输入。然后cat命令的输出重定向到output.txt，因此将内容输出到output.txt中。

与输出重定向类似，输入重定向的`<<`也表示追加。

##### 绑定重定向

上面的输出和输出绑定的文件或者设备只对该命令有效，如果需要一次绑定，接下来均有效的话，可以使用exec命令来绑定描述符。

```shell
$ exec 6 > &1
$ exec 1 > success.txt
$ exec 1 > &6
```

上述命令分别表示：

- 将标准输出与fd 6绑定。
- 将标准输出重定向到success.txt，接下来的指令执行成功的结果将不在屏幕上显示。
- 恢复标准输出。

**说明：**使用前先将标准输入保存到文件描述符6，这里说明下，文件描述符默认会打开0,1,2，还可以使用自定义描述符。然后对标准输出绑定到文件，接下来所有输出都会发生到文件。使用完后，恢复标准的输出，关闭打开文件描述符6。

#### 管道

管道的符号是`|`，它仅能处理经由前面一个指令传出的正确输出信息，也就是标准输出（standard output）的信息，对于标准错误（stdandard error）信息没有直接处理能力。然后，传递给下一个命令，作为其标准输入（standard input）。因此可以认为管道其实是重定向的一种常用形式。

**注意：**

- 管道命令只处理前一个命令正确输出，不处理错误输出
- 管道命令右边命令，必须能够接收标准输入流命令才行。

使用示例：

```shell
$ cat test.txt | grep -n 'test'
```

`cat test.txt`会将test.txt的内容作为标准输出，然后利用管道，将其作为`grep -n 'test'`命令的标准输入。

```shell
$ tee [OPTION]... [FILE]... # 读取标准输入的数据，并将其内容输出成文件 -a:追加

$ ls -l not_find_runoob 2>&1 | tee -a ls.log # 把标准报错也作为标准输出。写 crontab job 的时候常用
```



### 文件系统

#### 文件结构


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
$ pwd # print working directory 显示当前工作路径
$ cd /Dir # change directory ~:当前用户家目录; ..:上一级目录; -:返回上一次所在目录

$ ls [OPTION]... [FILE]...# list 列出目录内容 -l:显示详细信息; -a:显示所有文件,包括.开头的隐藏文件; -h:将大小表示为K M G 等; -S:按文件大小排序; -t:按修改时间排序; -r:排序是逆序排序; -R:递归显示所有子文件
$ ls -alh
total 3.9G
drwxr-xr-x  19 root root 4.0K 11月 23 22:47 .
drwxr-xr-x  19 root root 4.0K 11月 23 22:47 ..
lrwxrwxrwx   1 root root    7  8月 10 08:18 bin -> usr/bin
drwxr-xr-x   5 root root 4.0K 11月 23 23:29 boot
drwxr-xr-x  20 root root 4.1K 11月 30 23:11 dev
drwxr-xr-x 140 root root  12K 11月 28 22:09 etc
drwxr-xr-x   3 root root 4.0K 11月 23 22:50 home
。。。。

$ tree # 以树状图列出目录的内容 -a:显示所有文件和目录; -d:只显示目录; -L:level 限制目录显示层级
$ tree C-Program/
C-Program/
├── bin
│   └── Debug
│       └── C-Program
├── C-Program.cbp
├── main.c
└── obj
    └── Debug
        └── main.o

$ stat [OPTION]... FILE... # status 显示文件状态信息
$ stat auth.log 
  文件：auth.log
  大小：90780     	块：192        IO 块大小：4096   普通文件
设备：fd00h/64768d	Inode：527843      硬链接：1
权限：(0640/-rw-r-----)  Uid: (  107/  syslog)   Gid: (    4/     adm)
访问时间：2023-11-23 22:50:33.196000002 +0800
修改时间：2023-11-30 23:18:15.926576196 +0800
变更时间：2023-11-30 23:18:15.926576196 +0800
创建时间：2023-11-23 22:50:33.196000002 +0800

$ file [OPTION]... FILE... # 识别文件类型
$ file auth.log 
auth.log: data

$ wc [-lwc] # 显示文件行数、字数，以及字节数 -l:显示行数; -w:显示字数; -c:显示Bytes数
$ wc main.c 
 20  49 378 main.c
```

#### 硬连接、软连接

**硬连接:** linux下的文件是通过索引节点（Inode）来识别文件，硬链接可以认为是一个指针，指向文件索引节点的指针，系统并不为它重新分配inode。每添加一个一个硬链接，文件的链接数就加1。删除其中任何一个，每次只会删除一个指针，链接数同时减一，只有将所有指向文件内容的指针，也即链接数减为0时，内核才会把文件内容从磁盘上删除。不允许给目录创建硬链接。

```shell
$ ls -li
2232370 -rw-rw-r-- 1 wanyuhao wanyuhao 0 12月 16 00:24 file1
2232371 -rw-rw-r-- 1 wanyuhao wanyuhao 0 12月 16 00:24 file2
$ ln file2 file2hard  
$ ls –il  
ls -li
2232370 -rw-rw-r-- 1 wanyuhao wanyuhao 0 12月 16 00:24 file1
2232371 -rw-rw-r-- 2 wanyuhao wanyuhao 0 12月 16 00:24 file2
2232371 -rw-rw-r-- 2 wanyuhao wanyuhao 0 12月 16 00:24 file2hard
```

**软连接:** [链接名]可以是任何一个文件名，也可以是一个目录，并且允许它与“目标”不在同一个文件系统中。如果[链接名]是一个已经存在的目录，系统将在该目录下建立一个或多个与“目标”同名的文件，此新建的文件实际上是指向原“目标”的符号链接文件。

```shell
$ ln -s file1 file1soft
$ ls -li
2232370 -rw-rw-r-- 1 wanyuhao wanyuhao 0 12月 16 00:24 file1
2232372 lrwxrwxrwx 1 wanyuhao wanyuhao 5 12月 16 00:38 file1soft -> file1
2232371 -rw-rw-r-- 2 wanyuhao wanyuhao 0 12月 16 00:24 file2
2232371 -rw-rw-r-- 2 wanyuhao wanyuhao 0 12月 16 00:24 file2hard
```

#### 查看文件

```shell
$ cat [OPTION]... FILE # concatenate 连接文件并打印 -n:输出带行号
$ cat hosts host.conf > myhost # 将hosts host.conf两个文件合并保存为myhost

$ less [OPTION]... FILE # 浏览文件，支持翻页和搜索 g:跳转到第一行;G:跳转到最后一行;/pattern:从当前往后查询;?pattern:从当前往前查询;n:跳转下一个查询内容;N:跳转上一个查询内容

$ head [OPTION]... FILE # 查看文件开头部分,默认10行 -n number:显示前number行
$ tail [OPTION]... FILE # 查看文件末尾部分,默认10行 -n number:显示后number行
$ tail -f FILE # 将 FILE 文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要 FILE 更新就可以看到最新的文件内容。常用于查阅正在改变的日志文件。

$ cut OPTION... [FILE]... # 剪切字节、字符和字段 -b:以字节为单位进行分割; -c:以字符为单位进行分割; -d:自定义分隔符，默认为制表符; -f:与-d一起使用，指定显示区域

$ watch [options] command # 以周期性的方式执行给定的指令并显示。用于监测一个命令的运行结果。-n:每间隔时间运行,默认2s; -d:高亮显示变化部分
```

#### 新建、拷贝、移动、删除

```shell
$ touch [OPTION]... FILE... # 新建文件
修改时间戳：
$ touch [-amr] [-t 202312060713.20] FILE # 默认修改访问时间和修改时间; a:改变访问时间; m:改变修改时间; r:指定文档或目录的日期时间，统统设成和参考文档或目录的日期时间相同; t:修改为指定时间
$ mkdir [OPTION]... DIRECTORY... # 创建目录 -p:根据需要创建父目录,不产生错误

$ cp [OPTION]... SOURCE... DIRECTORY # 复制文件或目录 -r:递归复制目录; -u:仅复制更新的文件; -p:保留源文件的权限、所有者和时间戳信息; -i:覆盖前提示; -f:覆盖前不提示,强制复制

$ mv [OPTION]... SOURCE... DIRECTORY # 移动文件或目录;可用于重命名 -n:不要覆盖已存在的文件或目录; -u:仅移动更新的文件; -i:覆盖前提示; -f:覆盖前不提示,强制复制

$ rm [OPTION]... [FILE]... # 删除文件或目录 -r:递归删除目录; -f:强制删除，无需确认 -i:删除前提示
```

#### 查找

```shell
$ which [-a] filename ... # 在环境变量$PATH设置的目录里查找符合条件的文件
$ which ls
/usr/bin/ls

$ locate [OPTION]... FILE... # 从/var/cache/locate/locatedb中查找符合的文件；手动更新库：sudo updatedb。-b:仅匹配路径名的基本名称; -i:忽略大小写; -r:使用基本正则表达式; -S:不搜索条目，打印有关每个数据库的统计信息

$ find [PATH] [OPTION] # 在指定目录下查找文件和目录.-name:按文件名查找; -type:按文件类型查找，可以是 f（普通文件）、d（目录）、l（符号链接）等; -size [+-]size[cwbkMG]：按文件大小查找，支持使用 + 或 - 表示大于或小于指定大小，单位可以是 c（字节）、w（字数）、b（块数）、k（KB）、M（MB）或 G（GB）; -atime -amin -ctime -cmin -mtime -mmin +n比n天前更早 -n在n天内 n 天前（指定那一天）:根据时间戳查找
$ sudo find /etc -name passwd 
/etc/passwd
/etc/pam.d/passwd

$ grep [OPTION...] PATTERNS [FILE...] # 查找文件里符合条件的字符串. -i:忽略大小写进行匹配。-v:反向查找; -n:显示匹配行的行号; -c:只打印匹配的行数; -w:只显示全字符合; -A:除了显示符合范本样式的那一行之外，并显示该行之后的内容; -B:除了显示符合样式的那一行之外，并显示该行之前的内容。-C:除了显示符合样式的那一行之外，并显示该行之前后的内容。
$ grep printf C-Program/main.c
    printf("Please enter your name,age,height:");
    printf("\nYour name is %s,%d years old,%.2f meters tall.\n", name, age, height);
```

#### 对比

```shell
$ cmp [OPTION]... FILE1 [FILE2 [SKIP1 [SKIP2]]] # 对比文件不同
cmp main.c C-Program/main.c 
main.c C-Program/main.c 不同：第 51 字节，第 3 行

$ diff [OPTION]... FILES # 对比文件不同. -w:忽略全部的空格字符; -y:以并列的方式显示文件的异同之处; -B:不检查空白行。
$ diff main.c C-Program/main.c 
3,4c3,4
< Purpose: This program prints out hello world.
< Date: 2023-12-15
---
> Purpose: This program prints out imputs.
> Date: 2023-12-11
12c12,17
<     printf("hello world!");
---
>     char name[10];
>     int age;
>     float height;
>     printf("Please enter your name,age,height:");
>     scanf("%s %d %f", name, &age, &height);
>     printf("\nYour name is %s,%d years old,%.2f meters tall.\n", name, age, height);
```

#### 归档、压缩

```shell
$ tar [-cxzjvf] filename [file/dir] # 打包压缩 -c:创建归档; -x:从归档解出文件; -v:详细地列出处理的文件; -f:指定文件名; -z:通过gzip处理; -j: 过bzip2处理,压缩更强; -t:查看内容
$ tar -czvf C-Program.tar.gz C-Program/
$ tar -cjvf C-Program.tar.bz2 C-Program/
$ tar -xzvf C-Program.tar.gz C-Program/
$ tar -xjvf C-Program.tar.bz2 C-Program/
```

