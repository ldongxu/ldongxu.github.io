---
title: Linux同步工具：rsync
date: 2017-04-13 16:11:40
categories: Linux
tags: [Linux,rsync,Linux文件同步]
---
rsync是一款Linux上的文件远程同步工具。
rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。 rsync是一个功能非常强大的工具，其命令也有很多功能特色选项，我们下面就对它的选项一一进行分析说明。
<!--more-->
### 安装
一般的Linux系统都自带了这个工具，如果没有的话，可以用下面的命令安装
```
$ wget https://download.samba.org/pub/rsync/src/rsync-3.1.2.tar.gz
$ tar -xvzf rsync-3.1.2.tar.gz && cd rsync-3.1.2
$ ./configure && make && sudo make install
```
### 服务器端
假设我们有一个目录/home/ldongxu/backup/mysql需要同步，创建配置文件
```
$ sudo touch /etc/rsyncd.conf
$ sudo vi /etc/rsyncd.conf
```
加入下面的内容
```
# SYNC守护进程的用户
uid = root
# 运行RSYNC守护进程的组
gid = root
# 不使用chroot
use chroot = no
# 最大连接数是4
max connections = 4
# pid文件存放位置
pid file = /var/run/rsyncd.pid
# 锁文件存放位置
lock file = /var/run/rsync.lock
# 日志文件存放位置
log file = /var/log/rsyncd.log
[mysql]
# 要同步的目录
path = /home/ldongxu/backup/mysql
# 忽略无关的IO错误
ignore errors
# 只读，不能上传
read only = true
# 禁止查看文件列表
list = false
# 允许访问服务的ip
# hosts allow = 192.168.1.200
# 禁止访问服务的ip
# hosts deny = 0.0.0.0/32
# 认证的用户名，系统必须存在的用户，但是密码需要在secrets file 配置，不是系统的密码。
auth users = ldongxu
# 认证用户密码文件，配置auth users的密码
secrets file = /etc/backserver.pas
```
上面配置文件中的mysql可以自定义，客户端同步的时候要用到。上面的认证用户，必须是系统存在的用户。
接下来创建密码文件`/etc/backserver.pas`
```
$ sudo touch /etc/backserver.pas
$ sudo chown root:root /etc/backserver.pas
$ sudo chmod 600 /etc/backserver.pas
```
然后在里面加入
`ldongxu:ldongxu`
每个用户一行，冒号前面是用户名，后面是密码，然后启动服务
```
$ sudo rsync --daemon --config=/etc/rsyncd.conf
```
这样服务端就算搭建好了，可以把命令加入到开机启动
```
$ echo 'rsync --daemon --config=/etc/rsyncd.conf' >> /etc/rc.d/rc.local
```
### 客户端
假设我们要把刚刚服务器端的`/home/ldongxu/backup/mysql`同步下来。
先创建密码文件，用于同步时的验证
```
$ touch ~/rsyncd.secrets
$ chmod 600 ~/rsyncd.secrets
```
这个文件只要放密码就可以了，在这个例子中，则是放入
```
ldongxu
```
然后运行
```
$ rsync -avz --delete --password-file=/xxx/rsyncd.secrets clinyong@your_ip::mysql destination
```
把server_ip换成自己服务器的ip地址，mysql就是在服务器端的配置文件填写的字段，destination换成本地路径，这样子就能把文件同步下来了。

把命令加入到crontab，让其每天同步一次，运行crontab -e，在最后一行加入
```
00 00 * * * rsync -avz --delete --password-file=/etc/rsyncd.secrets ldongxu@server_ip::mysql destination
```


#### 语法
```
rsync [OPTION]... SRC DEST 
rsync [OPTION]... SRC [USER@]host:DEST 
rsync [OPTION]... [USER@]HOST:SRC DEST 
rsync [OPTION]... [USER@]HOST::SRC DEST 
rsync [OPTION]... SRC [USER@]HOST::DEST 
rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
```
对应于以上六种命令格式，rsync有六种不同的工作模式： 
1. 拷贝本地文件。当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。如：rsync -a /data /backup 
2. 使用一个远程shell程序(如rsh、ssh)来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。如：rsync -avz *.c foo:src 
3. 使用一个远程shell程序(如rsh、ssh)来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。如：rsync -avz foo:src/bar /data 
4. 从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。如：rsync -av root@192.168.78.192::www /databack 
5. 从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。如：rsync -av /databack root@192.168.78.192::www 
6. 列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。如：rsync -v rsync://192.168.78.192/www 
#### 选项
```
-v, --verbose 详细模式输出。 
-q, --quiet 精简输出模式。 
-c, --checksum 打开校验开关，强制对文件传输进行校验。 
-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD。 
-r, --recursive 对子目录以递归模式处理。 
-R, --relative 使用相对路径信息。 
-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir 将备份文件(如~filename)存放在在目录下。 -suffix=SUFFIX 定义备份文件前缀。 
-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件，不覆盖更新的文件。 
-l, --links 保留软链结。 
-L, --copy-links 想对待常规文件一样处理软链结。 
--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结。 
--safe-links 忽略指向SRC路径目录树以外的链结。 
-H, --hard-links 保留硬链结。 
-p, --perms 保持文件权限。 
-o, --owner 保持文件属主信息。 
-g, --group 保持文件属组信息。 
-D, --devices 保持设备文件信息。 
-t, --times 保持文件时间信息。 
-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间。 
-n, --dry-run现实哪些文件将被传输。 
-w, --whole-file 拷贝文件，不进行增量检测。 
-x, --one-file-system 不要跨越文件系统边界。 
-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节。 
-e, --rsh=command 指定使用rsh、ssh方式进行数据同步。 
--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息。 
-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件。 
--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件。 
--delete 删除那些DST中SRC没有的文件。 
--delete-excluded 同样删除接收端那些被该选项指定排除的文件。 
--delete-after 传输结束以后再删除。 
--ignore-errors 及时出现IO错误也进行删除。 
--max-delete=NUM 最多删除NUM个文件。 
--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输。 
--force 强制删除目录，即使不为空。 
--numeric-ids 不将数字的用户和组id匹配为用户名和组名。 
--timeout=time ip超时时间，单位为秒。 
-I, --ignore-times 不跳过那些有同样的时间和长度的文件。 
--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间。 
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0。 
-T --temp-dir=DIR 在DIR中创建临时文件。
--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份。 
-P 等同于 --partial。 
--progress 显示备份过程。 
-z, --compress 对备份的文件在传输时进行压缩处理。 
--exclude=PATTERN 指定排除不需要传输的文件模式。 
--include=PATTERN 指定不排除而需要传输的文件模式。 
--exclude-from=FILE 排除FILE中指定模式的文件。 
--include-from=FILE 不排除FILE指定模式匹配的文件。 
--version 打印版本信息。 
--address 绑定到特定的地址。 
--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件。 
--port=PORT 指定其他的rsync服务端口。 
--blocking-io 对远程shell使用阻塞IO。 
-stats 给出某些文件的传输状态。 
--log-format=formAT 指定日志文件格式。 
--password-file=FILE 从FILE中得到密码。 
--bwlimit=KBPS 限制I/O带宽，KBytes per second。 
-h, --help 显示帮助信息。
```
