---
title: Gson解析空字符串异常的处理
date: 2016-12-30 10:16
categories: Java
tags: [Java,Gson]
---
面对一些不规范的json,我们用Gson解析经常会抛出各种异常导致程序崩溃,这里可以采取一些措施来避免。
<!--more-->

### Json异常情况

先来看一个后台返回的json
正常情况下json:
``` 
{
    "code":0,
    "msg":"ok",
    "data":{
        "id":5638,
        "newsId":5638
    }
}
```
data部分对应的实体类:

``` java
public class JsonBean {
    private int id;
    private int newsId;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getNewsId() {
        return newsId;
    }

    public void setNewsId(int newsId) {
        this.newsId = newsId;
    }
}
```
异常情况json(后台数据库newsId字段未查询到对应数据):
```
{
    "code":0,
    "msg":"ok",
    "data":{
        "id":5638,
        "newsId":""
    }
}
```
这样Gson在解析时就会抛出解析错误的异常,程序崩溃,原因是无法将""转化为int。

### json异常的处理

我们期望在后台返回的json异常时,也能解析成功,空值对应的转换为默认值,如:newsId=0;
这里排除掉后台开发人员输出时给你做矫正,还是得靠自己。
我们写一个针对int值的类型转换器,需要实现Gson的JsonSerializer<T>接口和JsonDeserializer<T>,即序列化和反序列化接口：
``` java
public class IntegerDefault0Adapter implements JsonSerializer<Integer>, JsonDeserializer<Integer> {
    @Override
    public Integer deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
            throws JsonParseException {
        try {
            if (json.getAsString().equals("") || json.getAsString().equals("null")) {//定义为int类型,如果后台返回""或者null,则返回0
                return 0;
            }
        } catch (Exception ignore) {
        }
        try {
            return json.getAsInt();
        } catch (NumberFormatException e) {
            throw new JsonSyntaxException(e);
        }
    }

    @Override
    public JsonElement serialize(Integer src, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(src);
    }
}
```
### 同理Long及Double类型

double=>
``` java
public class DoubleDefault0Adapter implements JsonSerializer<Double>, JsonDeserializer<Double> {
    @Override
    public Double deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        try {
            if (json.getAsString().equals("") || json.getAsString().equals("null")) {//定义为double类型,如果后台返回""或者null,则返回0.00
                return 0.00;
        }
            } catch (Exception ignore) {
        }
        try {
            return json.getAsDouble();
        } catch (NumberFormatException e) {
            throw new JsonSyntaxException(e);
        }
    }

    @Override
    public JsonElement serialize(Double src, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(src);
    }
}
```
long=>
``` java
public class LongDefault0Adapter implements JsonSerializer<Long>, JsonDeserializer<Long> {
    @Override
    public Long deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
        throws JsonParseException {
        try {
            if (json.getAsString().equals("") || json.getAsString().equals("null")) {//定义为long类型,如果后台返回""或者null,则返回0
                    return 0l;
                }
            } catch (Exception ignore) {
        }
        try {
            return json.getAsLong();
        } catch (NumberFormatException e) {
            throw new JsonSyntaxException(e);
        }
    }

    @Override
    public JsonElement serialize(Long src, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(src);
    }
}
```
``` java
/**
 * 增加后台返回""和"null"的处理
 * 1.int=>0
 * 2.double=>0.00
 * 3.long=>0L
 *
 * @return
 */
public static Gson buildGson() {
    if (gson == null) {
        gson = new GsonBuilder()
                .registerTypeAdapter(Integer.class, new IntegerDefault0Adapter())
                .registerTypeAdapter(int.class, new IntegerDefault0Adapter())
                .registerTypeAdapter(Double.class, new DoubleDefault0Adapter())
                .registerTypeAdapter(double.class, new DoubleDefault0Adapter())
                .registerTypeAdapter(Long.class, new LongDefault0Adapter())
                .registerTypeAdapter(long.class, new LongDefault0Adapter())
                .create();
    }
    return gson;
}
```
再也不会因为后台json字段为空的情况崩溃了。

### <font color=red>数组类型的字段解析异常 </font>
关于数组类型的字段解析异常,尝试了一些方案,但最后都存在问题。
异常示例=>正常json:
```
{
    "code":0,
    "msg":"ok",
    "data":[    //约定为数组
      {
        "id":5638,
        "newsId":5638
      }
      ]
}
``` 
异常json:
``` 
{
    "code":0,
    "msg":"ok",
    "data":{}    //返回为对象或者空字符串
}
``` 
