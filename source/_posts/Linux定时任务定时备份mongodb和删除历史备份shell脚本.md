---
title: Linux定时任务定时备份mongodb和删除历史备份shell脚本
date: 2017-03-29 17:54:24
categories: mongodb
tags: [mongodb,crontab,shell]
english_title: linux_mongodb_backup
---
- Linux下mongodb数据备份shell脚本
- 常用crontab命令：
- mongodb的mongodump、mongorestore几种执行方式
<!--more-->

#### 一、在linux下面使用shell脚本执行Mongodb数据文件备份
代码如下：
```
#!/bin/bash
sourcepath='/app/mongodb'/bin
targetpath='/yoyodata/mongobak'
nowtime=$(date +%Y%m%d)
expiretime=$(date -d '-7 days' "+%Y%m%d")
 
start()
{
   ${sourcepath}/mongodump --host 127.0.0.1 --port 27017 --out ${targetpath}/${nowtime}
}
delete()
{
	if [ -d "${targetpath}/${expiretime}/" ]
	then
 	 	rm -rf "${targetpath}/${expiretime}/"
  		echo "=======${targetpath}/${expiretime}/===delete successfully!=="
	fi
	echo "===no $expiretime need to delete==="
}
execute()
{
  start
  if [ $? -eq 0 ]
  then
    echo "back successfully!"
    delete
  else
    echo "back failure!"
  fi
}
 
if [ ! -d "${targetpath}/${nowtime}/" ]
then
 mkdir ${targetpath}/${nowtime}
fi
execute
echo "============== back end ${nowtime} =============="
```

#### 二、常用crontab命令：
查看当前用户的定时任务：
[root@AY1404010929194851d3Z:~]# crontab -l
查看指定用户的定时任务：
[root@AY1404010929194851d3Z:~]# crontab -uroot -l
追加crontab定时任务：
[root@AY1404010929194851d3Z:~]# crontab -e
{% post_link Linux下crontab定时任务 Linux下crontab定时任务 %}

#### 三、mongodb的mongorestore数据还原

1. 几种mongodb数据备份方式：
```
无账号、密码
mongodump -o backup   #备份所有数据库到backup目录下，每个数据库一个文件，除local数据库外。
mongodump -d abc  -o backup #备份abc数据库到backup目录下。
mongodump -d abc -c ddd -o backup  #备份abc数据库下的ddd集合。

#有账号、密码
mongodump -udba -pdba -d abc -c ddd -o backup  #备份abc数据库下的ddd集合。
mongodump  --host=127.0.0.1 --port=27017 -udba -p  --db=abc --collection=ddd -o backup
```

2. 几种mongodb数据恢复方式
```
mongorestore -udba -pdba -d abc backup/abc #还原abc数据库。

mongorestore -udba -pdba -d abc --drop backup/abc #还原之前先删除原来数据库（集合）。

mongorestore -udba -pdba -d abc -c ddd --drop backup/abc/ddd.bson  #还原abc库中的ddd集合。

mongorestore --host=127.0.0.1 --port=27017 -udba -pdba -d abc -c test --drop backup/abc/test.bson #还原abc库中的test集合。

mongorestore --host=127.0.0.1 --port=27017 -udba -pdba -d abc -c ooo --drop backup/abc/test.bson #还原abc库中的test集合到ooo集合。
```
