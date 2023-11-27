# linux

## Note

### 安装和更新

```java
apt update //更新系统包索引或包列表
apt upgrade //升级软件包至最新版本;安装系统所需安全更新
apt install package //安装软件
apt remove package //卸载软件
```

### 命令相关

```java
ctrl+l //清屏
ctrl+z //暂停
ctrl+c //中断
ctrl+d //退出shell
ctrl+a //光标切换至行首
ctrl+e //光标切换至行末 
ctrl+k //剪切光标处到行尾的字符
ctrl+u //剪切光标处到行首的字符
ctrl+y //将剪切的字符进行粘贴
sudo command //以root身份执行命令
su - user //切换用户
man command	//命令手册;g:跳转到第一行;G:跳转到最后一行;/pattern:从当前往后查询;?pattern:从当前往前查询;n:跳转下一个查询内容;N:跳转上一个查询内容
command --help 	//命令解释
history //历史命令,终端关闭后将缓存存至文件~/.bash_history
history -c //删除所有历史命令
echo $HISTCONTROL 	//若为ignoreboth,不保存"空格+命令"和重复连续命令
```

