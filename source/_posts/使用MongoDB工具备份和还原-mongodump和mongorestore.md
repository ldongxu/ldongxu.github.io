---
title: 使用MongoDB工具备份和还原(mongodump和mongorestore)
date: 2017-04-06 16:46:42
categories: Mongodb
tags: Mongodb备份还原
---
MongoDB version：3.4
之前已经搭建了mongodb副本集，主要结构是一主二从（其中的一个从节点为仲裁节点），mongodb复制集主要目的是为了故障转移，但是出于数据安全角度考虑，数据备份也是非常关键的。
MongoDB的备份其实算是一个基本操作。具体使用操作建议参看官方文档，官方文档的介绍比较全面也会有相关的建议。
<!-- more -->
### 常见的备份方式：
1、通过复制基础数据文件备份(使用文件系统快照备份数据文件)；
2、通过mongodump工具备份；
### 通过mongodump工具备份
mongodb官方文档关于‘通过mongodump工具备份’有一段描述：
>mongodump reads data from a MongoDB database and creates high fidelity BSON files which the mongorestore tool can use to populate a MongoDB database. mongodump and mongorestore are simple and efficient tools for backing up and restoring small MongoDB deployments, but are not ideal for capturing backups of larger systems.
mongodump从MongoDB数据库读取数据，并创建高保真的BSON文件，这些文件可以使用mongorestore将数据库还原到MongoDB数据库。 mongodump和mongorestore是用于备份和恢复小型MongoDB部署的简单有效的工具，但不是较大系统的备份的理想选择。

>mongodump only captures the documents in the database. The resulting backup is space efficient, but mongorestore or mongod must rebuild the indexes after restoring data.
mongodump只捕获数据库中的文档。 生成的备份是空间有效的，但是mongorestore或mongod必须在还原数据后重建索引。

>When connected to a MongoDB instance, mongodump can adversely affect mongod performance. If your data is larger than system memory, the queries will push the working set out of memory, causing page faults.
当连接到MongoDB实例时，mongodump可能会对mongod性能产生不利影响。 如果您的数据大于系统内存，查询会将工作集推出内存，导致页面错误。

>Applications can continue to modify data while mongodump captures the output. For replica sets, mongodump provides the --oplog option to include in its output oplog entries that occur during the mongodump operation. This allows the corresponding mongorestore operation to replay the captured oplog. To restore a backup created with --oplog, use mongorestore with the --oplogReplay option.
当mongodump捕获输出时，应用程序可以继续修改数据。 对于副本集，mongodump提供--oplog选项，以便在其输出的oplog条目中包含mongodump操作期间发生的数据库操作。 这允许相应的mongorestore操作重播捕获的oplog。 要恢复使用--oplog创建的备份，请使用带有--oplogReplay选项的mongorestore。

**mongodump的选项--oplog是一个值得一提的选项：**
注意这是replica set或者master/slave模式专用（standalone模式运行mongodb并不推荐）。
它的实际作用是在导出的同时生成一个oplog.bson文件，存放在你开始进行dump到dump结束之间所有的oplog。
![](http://images2015.cnblogs.com/blog/333151/201509/333151-20150914022238492-1981549003.png)
简单地说，在replica set中oplog是一个定容集合（capped collection），它的默认大小是磁盘空间的5%（可以通过--oplogSizeMB参数修改），位于local库的db.oplog.rs，有兴趣可以看看里面到底有些什么内容。其中记录的是整个mongod实例一段时间内数据库的所有变更（插入/更新/删除）操作。
oplog有一个非常重要的特性——幂等性（idempotent）。即对一个数据集合，使用oplog中记录的操作重放时，无论被重放多少次，其结果会是一样的。举例来说，如果oplog中记录的是一个插入操作，并不会因为你重放了两次，数据库中就得到两条相同的记录。这是一个很重要的特性，也是后面这些操作的基础。

回到主题上来，看看oplog.bson到底有什么作用。首先要明白的一个问题是数据之间互相有依赖性，比如集合A中存放了订单，集合B中存放了订单的所有明细，那么只有一个订单有完整的明细时才是正确的状态。假设在任意一个时间点，A和B集合的数据都是完整对应并且有意义的（对非关系型数据库要做到这点并不容易，且对于MongoDB来说这样的数据结构并非合理。但此处我们假设这个条件成立），那么如果A处于时间点x，而B处于x之后的一个时间点y时，可以想象A和B中的数据极有可能不对应而失去意义。

再回来看mongodump的操作。mongodump的进行过程中并不会把数据库锁死以保证整个库冻结在一个固定的时间点，这在业务上常常是不允许的。所以就有了dump的最终结果中A集合是10点整的状态，而B集合则是10点零1分的状态这种情况。这样的备份即使恢复回去，可以想象得到的结果恐怕意义有限。那么上面这个oplog.bson的意义就在这里体现出来了。如果在dump数据的基础上，再重做一遍oplog中记录的所有操作，这时的数据就可以代表dump结束时那个时间点（point-in-time）的数据库状态。
这个结论成立的重要条件就是幂等性：已存在的数据，重做oplog不会重复；不存在的数据重做oplog就可以进入数据库。所以当做完截止到某个时间点的oplog时，数据库就恢复到了截止那个时间点的状态。

**那么从别处而来的oplog呢？**
聪明如你可能已经想到，既然dump出的数据配合oplog就可以把数据库恢复到某个状态，那是不是拥有一份从某个时间点开始备份的dump数据，再加上从dump开始之后的oplog，如果oplog足够长，是不是就可以把数据库恢复到其后的任意状态了？是的！事实上replica set正是依赖oplog的重放机制在工作。当secondary第一次加入replica set时做的initial sync就相当于是在做mongodump，此后只需要不断地同步和重放oplog.rs中的数据，就达到了secondary与primary同步的目的。

既然oplog一直都在oplog.rs中存在，我们为什么还需要在mongodump时指定--oplog呢？需要的时候从oplog.rs中拿不就完了吗？答案是肯定的，你确实可以只dump数据，不需要oplog。在需要的时候可以再从oplog.rs中取。但前提是oplog时间窗口（忘了时间窗口概念的请往前翻）必须能够覆盖dump的开始时间。
[参考链接](http://www.cnblogs.com/yaoxing/p/mongodb-backup-rules.html):<http://www.cnblogs.com/yaoxing/p/mongodb-backup-rules.html>
### mongodump几个例子
数据库在本地27017端口上运行
`mongodump  --db test --collection collection`
	
需要认证，备份输出到指定目录
`mongodump --host mongodb1.example.net --port 37017 --username user --password pass --out /opt/backup/mongodump-2011-10-24`

数据库备份到归档文件test.20150715.archive。*不能将--archive选项与-out选项一起使用*
`mongodump --archive=test.20150715.archive --db test`

归档并压缩
`mongodump --archive=test.20150715.gz --gzip --db test`

副本集备份（以副本集名称为前缀，mongodump将默认从主节点成员读取）
`mongodump --host "replSet/rep1.example.net:27017,rep2.example.net:27017,rep3.example.net:27017"`

副本集备份（不包含副本集名称作为主机字符串的前缀，mongodump则默认从最近的节点读取。）
`mongodump --host "rep1.example.net:27017,rep2.example.net:27017,rep3.example.net:27017"`

`mongodump --host "replSet/rep1.example.net:27017,rep2.example.net:27017,rep3.example.net:27017" --username user --password --oplog --gzip --out  /home/data/mongobak`

`mongorestore --host "replSet/rep1.example.net:27017,rep2.example.net:27017,rep3.example.net:27017" --username user --password --gzip --db test  /home/data/mongobak`

### mongodb备份和恢复角色
admin数据库中包括了备份和还原数据的角色backup和restore。（只在admin库中存在）
backup：提供备份数据所需的权限。 该角色提供了使用MongoDB Cloud Manager备份代理，Ops Manager备份代理或使用mongodump的足够权限。
restore：提供使用mongorestore恢复数据所需的权限，而不使用--oplogReplay选项或没有system.profile集合数据。（如果使用--oplogReplay运行mongorestore，还原角色不足以重播oplog。 要重播oplog，请创建一个用户定义的角色。）

*为了安全考虑，建议在admin中创建备份和恢复角色来执行数据备份和恢复操作。*



### mongodump/mongorestore与mongoexport/mongoimport的区别
除了mongodump/mongorestore之外还有一对组合是mongoexport/mongoimport

区别在哪里？

mongoexport/mongoimport导入/导出的是JSON格式，而mongodump/mongorestore导入/导出的是BSON格式。
JSON可读性强但体积较大，BSON则是二进制文件，体积小但对人类几乎没有可读性。
在一些mongodb版本之间，BSON格式可能会随版本不同而有所不同，所以不同版本之间用mongodump/mongorestore可能不会成功，具体要看版本之间的兼容性。当无法使用BSON进行跨版本的数据迁移的时候，使用JSON格式即mongoexport/mongoimport是一个可选项。跨版本的mongodump/mongorestore个人并不推荐，实在要做请先检查文档看两个版本是否兼容（大部分时候是的）。
JSON虽然具有较好的跨版本通用性，但其只保留了数据部分，不保留索引，账户等其他基础信息。使用时应该注意。
总之，这两套工具在实际使用中各有优势，应该根据应用场景选择使用（好像跟没说一样）。但严格地说，mongoexport/mongoimport的主要作用还是导入/导出数据时使用，并不是一个真正意义上的备份工具。所以这里也不展开介绍了。





