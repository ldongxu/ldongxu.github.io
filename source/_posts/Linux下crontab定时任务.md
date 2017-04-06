---
title: Linux下crontab定时任务
date: 2017-03-30 18:02:33
categories: Linux
tags: crontab
---
因为需要在Linux下定时对数据库做备份，所以对Linux的crontab服务做了一些简单的了解。
以下内容是在实际操作过程中遇到的一些问题及相关技术翻案，包括crontab简单了解、怎么用vi打开编辑crontab、crontab时间表达式、定时任务输出重定向等问题。
<!--more-->
### Linux下的定时任务设置

#### crontab简介

Linux下的任务调度分为两类，系统任务调度和用户任务调度。
系统任务调度：系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。在/etc目录下有一个crontab文件，这个就是系统任务调度的配置文件。
/etc/crontab文件包括下面几行：
  ```
  # /etc/crontab: system-wide crontab
  # Unlike any other crontab you don't have to run the `crontab'
  # command to install the new version when you edit this file
  # and files in /etc/cron.d. These files also have username fields,
  # that none of the other crontabs do.

  SHELL=/bin/sh
  PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  MAILTO=""
  HOME=/
  # m h dom mon dow user  command
  17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  ```
前四行是用来配置crond任务运行的环境变量，第一行SHELL变量指定了系统要使用哪个shell，这里是bash，第二行PATH变量指定了系统执行命令的路径，第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户，如果MAILTO变量的值为空，则表示不发送任务执行信息给用户，第四行的HOME变量指定了在执行命令或者脚本时使用的主目录。

用户任务调度：用户定期要执行的工作，比如用户数据备份、定时邮件提醒等。用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab 文件都被保存在 /var/spool/cron目录中。其文件名与用户名一致。

**使用者权限文件：**
/var/spool/cron：所有用户定义的crontab 文件都被保存在目录中，其文件名与用户名一致。
/etc/cron.deny：该文件中所列用户不允许使用crontab命令。
/etc/cron.allow：该文件中所列用户允许使用crontab命令，优先于/etc/cron.deny

设置crontab用vi打开编辑：
编辑.profile文件,增加EDITOR=vi;export EDITOR  或  直接在命令行输入 EDITOR=vi;export EDITOR
  判断方法：
  $ which $EDITOR
  $          如果该行显示为空，那么你就需要执行下面的两个语句了。
  $ EDITOR=vi
  $ export EDITOR

#### crontab的时间表达式

基本格式 :
```
*　　*　　*　　*　　*　　command
分　时　日　月　周　命令
```
![](https://sfault-image.b0.upaiyun.com/175/110/1751101486-55155eadab949_articlex)

几个例子：
```
1、每分钟执行一次            
*  *  *  *  * 

2、每隔一小时执行一次        
 00  *  *  *  * or * */1 * * *  (/表示频率)

3、每小时的15和30分各执行一次 
 15,45 * * * * （,表示并列）

4、在每天上午 8- 11时中间每小时 15 ，45分各执行一次
 15,45 8-11 * * * command （-表示范围）

5、每个星期一的上午8点到11点的第3和第15分钟执行
 3,15 8-11 * * 1 command

6、每隔两天的上午8点到11点的第3和第15分钟执行
 3,15 8-11 */2 * * command
```

#### crontab命令介绍

查看crontab服务状态：
  `service cron status`
手动启动crontab服务：
  `service cron start`
手动停止crontab服务：
  `service cron stop`

使用方式 :
```
crontab [-u user] file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。

crontab -u [user]：用来设定某个用户的crontab服务，例如，“-u ixdba”表示设定ixdba用户的crontab服务，此参数一般由root用户来运行。

crontab -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。

crontab -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。

crontab -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。

crontab -i：在删除用户的crontab文件时给确认提示。
```
常用crontab命令：
查看当前用户的定时任务：
  `[root@AY1404010929194851d3Z:~]# crontab -l`
查看指定用户的定时任务：
  `[root@AY1404010929194851d3Z:~]# crontab -uroot -l`
追加crontab定时任务：
  `[root@AY1404010929194851d3Z:~]# crontab -e`

**注**：定时任务结尾加 >/dev/null 2>&1(在调试好脚本程序后，应尽量把DEBUG及命令输出的内容信息屏蔽掉，如果确实需要输出日志，可定向到日志文件里，避免产生系统垃圾。)
如：
  ```
  [root@angelT ~]# crontab -l
  #backup www to /backup
  00 00 * * * /bin/sh /server/scripts/www_bak.sh >/dev/null  2>&1
  ```
有关/dev/null的说明：
*/dev/null为特殊的字符设备文件，表示黑洞设备或空设备*。
\>/dev/null 2>&1的作用：
如果定时任务规范结尾不加 >/dev/null 2>&1,很容易导致硬盘inode空间被占满，从而系统服务不正常（var/spool/clientmqueue邮件临时队列目录，垃圾文件存放于此，如果是centos 6.4系统，默认不装sendmail服务，所以不会有这个目录。）

有关重定向的说明：
```
>或1>   输出重定向：把前面输出的东西输入到后边的文件中，会删除文件原有内容。
>>或1>>追加重定向：把前面输出的东西追加到后边的文件中，不会删除文件原有内容。
<或<0   输入重定向：输入重定向用于改变命令的输入，指定输入内容，后跟文件名。
<<或<<0输入重定向：后跟字符串，用来表示“输入结束”，也可用ctrl+d来结束输入。
2>       错误重定向：把错误信息输入到后边的文件中，会删除文件原有内容。
2>>     错误追加重定向：把错误信息追加到后边的文件中，不会删除文件原有内容。
标准输入（stdin）：代码为0，使用<或<<。
标准输出（stdout）:代码为1，使用>或>>。正常的输出。
标准错误输出（sederr）：代码为2，使用2>或2>>。
特殊：
2>&1就是把标准错误重定向到标准输出（>&）。
>/dev/null 2>&1 等价于 1>/dev/null  2>/dev/null
```
