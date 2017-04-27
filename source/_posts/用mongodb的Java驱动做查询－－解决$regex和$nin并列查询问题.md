---
title: 用mongodb的Java驱动做查询－－解决$regex和$nin并列查询问题
date: 2017-04-25 21:53:24
categories: Java
tags: [Java, mongodb]
---
用mongodb的Java驱动而非Spring-data-mongodb的封装，以db.runCommand方式做查询。
<!--more-->
开发中遇到一个需求是利用mongodb模糊查询查询除已选中的标签外的其它标签，比如：标签库中有‘Java，Javascript，Jade，Core Java，Node.js，Python，JavaEE’，已选中的标签有‘Java，Jade’，现在输入‘ja’需要查出包含‘ja’(不区分大小写)的标签并且排除已选中的‘Java，Jade’。

### 用Spring-data-mongodb做$regex和$nin模糊查询时的问题
```java
package com.ldongxu.dao;

import com.ldongxu.domain.Skill;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.regex.Pattern;

@Repository
public class SkillDao extends BaseDao {
 	@Autowired
    protected MongoTemplate mongoTemplate;

	public List<Skill> findByRegex(String keyWord, List<String> existTags, Integer limit){
        Pattern pattern = Pattern.compile("^.*" + keyWord + ".*$", Pattern.CASE_INSENSITIVE);//js正则忽略大小写
        Criteria criteria = Criteria.where("skillTag").regex(pattern);
        if (existTags != null && !existTags.isEmpty()){
            criteria.nin(existTags);
        }
        Query query = Query.query(criteria);
        query.limit(limit);
        return mongoTemplate.find(query,Skill.class);
    }
}
```
利用这种方式得出的mongodb查询语句是这样的：
```
db.skill.find({ "skillTag" : { "$regex" : "^.*ja.*$" , "$options" : "i"} , "$nin" : [ "Jade" , "Java"]})


{
    "skillTag": {
        "$regex": "^.*ja.*$", 
        "$options": "i"
    }, 
    "$nin": [
        "Jade", 
        "Java"
    ]
}

```
而且执行是报如下错误：
```
Error: error: {
	"$err" : "Can't canonicalize query: BadValue unknown top level operator: $nin",
	"code" : 17287
}
```
那么问题来了，$regex和$nin不能并列做查询条件么？

### mongodb的Java驱动做db.runCommand查询

在mongodb官方文档中找到一段话：
>To include a regular expression in a comma-separated list of query conditions for the field, use the $regex operator. For example:（为了字段在以逗号分隔的查询条件列表中包含正则表达式，可以使用$regex运算符。例如：）
```
{ name: { $regex: /acme.*corp/i, $nin: [ 'acmeblahcorp' ] } }
{ name: { $regex: /acme.*corp/, $options: 'i', $nin: [ 'acmeblahcorp' ] } }
{ name: { $regex: 'acme.*corp', $options: 'i', $nin: [ 'acmeblahcorp' ] } }
```
由此，需求中的mongodb查询语句应该是这样的：
```
db.skill.find({ "skillTag" : { "$regex" : "^.*ja.*$" , "$options" : "i" , "$nin" : [ "Jade" , "Java"]}})
```
#### db.runCommand()的方式一：

利用Spring-data-mongodbAPI始终是组合不到这样的查询语句，所以选择用mongodb的Java驱动以db.runCommand()的方式实现，如下：
```
public List<Skill> findByRegex(String keyWord, List<String> existTags, Integer limit) {
        String collectionName = mongoTemplate.getCollectionName(Skill.class);
        DBObject filter = new BasicDBObject();
        DBObject condition = new BasicDBObject();
        Pattern pattern = Pattern.compile("^.*" + keyWord + ".*$", Pattern.CASE_INSENSITIVE);
        condition.put("$regex",pattern);
        if (existTags != null && existTags.size() > 0) {
            condition.put("$nin",existTags);
        }
        filter.put("skillTag",condition);
        DBCollection dbCollection = mongoTemplate.getCollection(collectionName);
        Cursor cursor = dbCollection.find(filter).limit(limit);
        List<Skill> skillList = new ArrayList<>();
        while (cursor.hasNext()){
            DBObject object = cursor.next();
            Skill skill = new Skill();
            skill.set_id(object.get("_id").toString());
            skill.setSkillTag(object.get("skillTag").toString());
            skill.setAddTime((Date) object.get("addTime"));
            skill.setStatus(Byte.valueOf(object.get("status").toString()));
            skillList.add(skill);
        }
        return skillList;
	}	
```
#### db.runCommand()的方式二：

也可以按mongodb的find原生定义命令进行查询。
mongodb官方文档中对find命令的定义如下：
>find
*New in version 3.2.*
Executes a query and returns the first batch of results and the cursor id, from which the client can construct a cursor.
The **find** command has the following form:
```
{
   "find": <string>,
   "filter": <document>,
   "sort": <document>,
   "projection": <document>,
   "hint": <document or string>,
   "skip": <int>,
   "limit": <int>,
   "batchSize": <int>,
   "singleBatch": <bool>,
   "comment": <string>,
   "maxScan": <int>,
   "maxTimeMS": <int>,
   "readConcern": <document>,
   "max": <document>,
   "min": <document>,
   "returnKey": <bool>,
   "showRecordId": <bool>,
   "snapshot": <bool>,
   "tailable": <bool>,
   "oplogReplay": <bool>,
   "noCursorTimeout": <bool>,
   "awaitData": <bool>,
   "allowPartialResults": <bool>,
   "collation": <document>
}
```
详细参考：<https://docs.mongodb.com/manual/reference/command/find/>
mongodb的Java驱动db.runCommand()的方式也可以写成：
```
public List<Skill> findByRegex(String keyWord, List<String> existTags, Integer limit) {
        String collectionName = mongoTemplate.getCollectionName(Skill.class);
        DBObject command = new BasicDBObject();
        command.put("find",collectionName);
        DBObject filter = new BasicDBObject();
        DBObject condition = new BasicDBObject();
        Pattern pattern = Pattern.compile("^.*" + keyWord + ".*$", Pattern.CASE_INSENSITIVE);
        condition.put("$regex",pattern);
        if (existTags != null && existTags.size() > 0) {
            condition.put("$nin",existTags);
        }
        filter.put("skillTag",condition);
        command.put("filter",filter);
        command.put("limit",limit);
        DBCollection dbCollection = mongoTemplate.getCollection(collectionName);
        CommandResult commandResult = dbCollection.getDB().command(command);
        List<Skill> skillList = new ArrayList<>();
        DBObject r = (DBObject) commandResult.get("cursor");
        BasicDBList dbObjectList = (BasicDBList) r.get("firstBatch");
        Iterator<Object> iterator = dbObjectList.iterator();
        while (iterator.hasNext()){
           DBObject object = (DBObject) iterator.next();
            Skill skill = new Skill();
            skill.set_id(object.get("_id").toString());
            skill.setSkillTag(object.get("skillTag").toString());
            skill.setAddTime((Date) object.get("addTime"));
            skill.setStatus(Byte.valueOf(object.get("status").toString()));
            skillList.add(skill);
        }
        return skillList;
    }
```
产生的**find** command格式：
```
{ "find" : "skill" , "filter" : { "skillTag" : { "$regex" : { "$regex" : "^.*ja.*$" , "$options" : "i"} , "$nin" : [ "Jade" , "Java"]}} , "limit" : 6}

{
    "find": "skill", 
    "filter": {
        "skillTag": {
            "$regex": {
                "$regex": "^.*ja.*$", 
                "$options": "i"
            }, 
            "$nin": [
                "Jade", 
                "Java"
            ]
        }
    }, 
    "limit": 6
}
```


