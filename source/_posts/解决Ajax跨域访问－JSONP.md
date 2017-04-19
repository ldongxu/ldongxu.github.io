---
title: 解决Ajax跨域访问－JSONP
date: 2016-02-24 10:03:59
categories: JavaScript
tags: [Ajax,JSONP]
---
# 解决Ajax跨域访问问题－JSONP

------

### 同源策略
首先我们来了解一下浏览器的同源策略，浏览器为了安全都会遵循同源策略。
同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。
<!--more-->
>同源策略，它是由Netscape提出的一个著名的安全策略。
现在所有支持JavaScript 的浏览器都会使用这个策略。
所谓同源是指，域名，协议，端口相同。
当一个浏览器的两个tab页中分别打开来 百度和谷歌的页面
当浏览器的百度tab页执行一个脚本的时候会检查这个脚本是属于哪个页面的，
即检查是否同源，只有和百度同源的脚本才会被执行。
如果非同源，那么在请求数据时，浏览器会在控制台中报一个异常，提示拒绝访问。

会出现跨域问题的几种情况：
![](http://p1.pstatp.com/large/349000064ea1c1e1107)

### Ajax实现跨域

#### 1. 通过代理方案实现跨域

这种方式是通过后台(ASP、PHP、Java、ASP.NET)获取其他域名下的内容，然后再把获得内容返回到前端，这样因为在同一个域名下，所以就不会出现跨域的问题。

代理实现比较麻烦，但使用最广泛，任何支持AJAX的浏览器都可以使用这种方式。
#### 2. JSONP
>JSONP是JSON with Padding的略称。它是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式）。

模拟一个非同源的环境，假设A程序是localhost:20001,B程序是localhost:20002(域名相同但端口号不同)，做一个小示例。
在A程序添加sample.html和test.js，
sample.html代码如下：
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
<title>test</title>
<script type="text/javascript" src="test.js"></script>
</head>
<body>
</body>
</html>
```
test.js代码如下：
``` javascript
alert("success");
```
打开localhost:20001/sample.html后会跳出"success”这样的这样的信息框，这似乎并不能说明什么， 跨域问题到底怎么解决呢？好，现在我们模拟下非同源的环境,将我们的之前的test.js文件从A程序中移除然后拷贝到B程序中，因为test.js文件在B程序上了，所以testx.js在sample.html里的引用就应该是：
```
<script type="text/javascript" src="http://localhost:20002/test.js"></script>
```
请保持AB两个Web程序的运行状态，当你再次刷新localhost:20001/sample.html的时候，和原来一样跳出了"success"的对话框，是的，成功访问到了非同源的localhost:20002/test.js这个所谓的远程服务了。到了这一步，相信大家应该已经大概明白如何跨域访问了的原理了。

--------

`<script>`标签的src属性并不被同源策略所约束，所以可以获取任何服务器上脚本并执行。

JSONP的定义说明中讲到了javascript callback的形式。那我们就来修改下代码，如何实现JSONP的javascript callback的形式。
程序A中sample.html的代码:
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
<title>test</title>
<script type="text/javascript">
 //回调函数
 function callback(data) {
    alert(data.message);
 }
 </script>
 <script type="text/javascript" src="http://localhost:20002/test.js"></script>
</head>
<body>
</body>
</html>
```

程序B中test.js的代码：
```
//调用callback函数，并以json数据形式作为阐述传递，完成回调
callback({message:"success"}); 
```
这其实就是JSONP的简单实现模式，或者说是JSONP的原型：创建一个回调函数，然后在远程服务上调用这个函数并且将JSON 数据形式作为参数传递，完成回调。（也就是类似于返回一个带结果数据的js function）。

------

一般情况下，我们希望这个`<script>`标签引用能够动态的调用，而不是像上面因为固定在html里面所以没等页面显示就执行了，很不灵活。我们可以通过javascript动态的创建script标签，这样我们就可以灵活调用远程服务了。
比如程序A中sample.html的部分代码可以修改为：
```
<script type="text/javascript">
      function callback(data) {
          alert(data.message);
      }
      //添加<script>标签的方法
      function addScriptTag(src){
      var script = document.createElement('script');
          script.setAttribute("type","text/javascript");
          script.src = src;
         document.body.appendChild(script);
     }
     
     window.onload = function(){
         addScriptTag("http://localhost:20002/test.js");
     }
</script>
```
程序B的test.js代码不变，我们再执行下程序，是不是和原来的一样呢。如果我们再想调用另一个远程服务的话，只要添加addScriptTag方法，传入远程服务的src值就可以了。这里说明下为什么要将addScriptTag方法放入到window.onload的方法里，原因是addScriptTag方法中有句document.body.appendChild(script);，这个script标签是被添加到body里的，由于我们写的javascript代码是在head标签中，document.body还没有初始化完毕呢，所以我们要通过window.onload方法先初始化页面，这样才不会出错。

上面的例子是最简单的JSONP的实现模型，不过它还算不上一个真正的JSONP服务。我们来看一下真正的JSONP服务是怎么样的，比如Google的ajax搜索接口：http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=?&callback=?。
q=?这个问号是表示你要搜索的内容，最重要的是第二个callback=?这个是正如其名表示回调函数的名称，也就是将你自己在客户端定义的回调函数的函数名传送给服务端，服务端则会返回以你定义的回调函数名的方法，将获取的json数据传入这个方法完成回调。

看看实现代码吧：
```
<script type="text/javascript">
     //添加<script>标签的方法
     function addScriptTag(src){
          var script = document.createElement('script');
          script.setAttribute("type","text/javascript");
          script.src = src;
          document.body.appendChild(script);
      }
      
     window.onload = function(){
         //搜索apple，将自定义的回调函数名result传入callback参数中
         addScriptTag("http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=apple&callback=result");
         
     }
     //自定义的回调函数result
     function result(data) {
         //我们就简单的获取apple搜索结果的第一条记录中url数据
         alert(data.responseData.results[0].unescapedUrl);
     }
</script>
```
像Google这样的JSONP服务还有很多：
>Digg API：来自 Digg 的头条新闻：
http://services.digg.com/stories/top?appkey=http%3A%2F%2Fmashup.com&type=javascript&callback=?

>Geonames API：邮编的位置信息：
http://www.geonames.org/postalCodeLookupJSON?postalcode=10504&country=US&callback=?

>Flickr JSONP API：载入最新猫的图片：
http://api.flickr.com/services/feeds/photos_public.gne?tags=cat&tagmode=any&format=json&jsoncallback=?

>Yahoo Local Search API：在邮编为 10504 的地区搜索比萨：
http://local.yahooapis.com/LocalSearchService/V3/localSearch?appid=YahooDemo&query=pizza&zip=10504&results=2&output=json&callback=?

------

接下来我们自己来创建一个简单的远程服务，实现和上面一样的JSONP服务。还是利用Web程序A和程序B来做演示，这次我们在程序B上创建一个MyService.java文件。
``` java
@Controller
public class MyService {

  @RequestMapping("/myService")
  public String ProcessRequest(String callback){
    String jsonDataStr = "{\"name\":\"chopper\",\"sex\":\"man\"}";
    return callback+"("+jsonDataStr+")";
  }
  
}
```

程序A的sample.html代码中的调用：
```
<script type="text/javascript">
     function addScriptTag(src){
         var script = document.createElement('script');
         script.setAttribute("type","text/javascript");
         script.src = src;
         document.body.appendChild(script);
     }
     
     window.onload = function(){
         //调用远程服务
         addScriptTag("http://localhost:20002/myService?callback=person");
         
     }
     //回调函数person
     function person(data) {
         alert(data.name + " is a " + data.sex);
     }
</script>　
```
这就完成了一个最基本的JSONP服务调用了。

------

下面我们来了解下jQuery是如何调用JSONP的。
jQuery框架也当然支持JSONP，可以使用$.getJSON(url,[data],[callback])方法(详细可以参考<http://api.jquery.com/jQuery.getJSON/>)。
那我们就来修改下程序A的代码，改用jQuery的getJSON方法来实现(下面的例子没用用到向服务传参，所以只写了getJSON(url,[callback]))：
```
<script type="text/javascript" src="http://code.jquery.com/jquery-latest.js"></script>
<script type="text/javascript">
    $.getJSON("http://localhost:20002/myService?callback=?",function(data){
        alert(data.name + " is a a" + data.sex);
    });
</script>
```
　　结果是一样的，要注意的是在url的后面必须添加一个callback参数，这样getJSON方法才会知道是用JSONP方式去访问服务，callback后面的那个问号是内部自动生成的一个回调函数名。这个函数名大家可以debug一下看看，比如jQuery17207481773362960666_1332575486681。

　　当然，假如说我们想指定自己的回调函数名，或者说服务上规定了固定回调函数名该怎么办呢？我们可以使用$.ajax方法来实现(参数较多，详细可以参考http://api.jquery.com/jQuery.ajax)。先来看看如何实现吧：
```
<script type="text/javascript" src="http://code.jquery.com/jquery-latest.js"></script>
<script type="text/javascript">
   $.ajax({
        url:"http://localhost:20002/myService?callback=?",   
        dataType:"jsonp",
        jsonpCallback:"person",
        success:function(data){
            alert(data.name + " is a a" + data.sex);
        }
   });
</script>
```
　　没错，jsonpCallback就是可以指定我们自己的回调方法名person，远程服务接受callback参数的值就不再是自动生成的回调名，而是person。dataType是指定按照JSOPN方式访问远程服务。
　　利用jQuery可以很方便的实现JSONP来进行跨域访问。

### 下面给出个栗子

一.客户端

``` html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
<title>Insert title here</title>  
<script type="text/javascript" src="resource/js/jquery-1.7.2.js"></script>  
</head>  
<script type="text/javascript">  
$(function(){     
    /*  
    //简写形式，效果相同  
    $.getJSON("http://app.example.com/base/json.do?sid=1494&busiId=101&jsonpCallback=?",  
            function(data){  
                $("#showcontent").text("Result:"+data.result)  
    });  
    */  
    $.ajax({  
        type : "get",  
        async:false,  
        url : "http://app.example.com/base/json.do?sid=1494&busiId=101",  
        dataType : "jsonp",//数据类型为jsonp  
        jsonp: "jsonpCallback",//服务端用于接收callback调用的function名的参数  
        success : function(data){  
            $("#showcontent").text("Result:"+data.result)  
        },  
        error:function(){  
            alert('fail');  
        }  
    });   
});  
</script>  
<body>  
<div id="showcontent">Result:</div>  
</body>  
</html> 
```
 二.服务器端

Java代码 
``` java
import java.io.IOException;  
import java.io.PrintWriter;  
import java.util.HashMap;  
import java.util.Map;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import net.sf.json.JSONObject;  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
  
@Controller  
public class ExchangeJsonController {  
    @RequestMapping("/base/json.do")  
    public void exchangeJson(HttpServletRequest request,HttpServletResponse response) {  
       try {  
        response.setContentType("text/plain");  
        response.setHeader("Pragma", "No-cache");  
        response.setHeader("Cache-Control", "no-cache");  
        response.setDateHeader("Expires", 0);  
        Map<String,String> map = new HashMap<String,String>();   
        map.put("result", "content");  
        PrintWriter out = response.getWriter();       
        JSONObject resultJSON = JSONObject.fromObject(map); //根据需要拼装json  
        String jsonpCallback = request.getParameter("jsonpCallback");//客户端请求参数  
        out.println(jsonpCallback+"("+resultJSON.toString(1,1)+")");//返回jsonp格式数据  
        out.flush();  
        out.close();  
      } catch (IOException e) {  
       e.printStackTrace();  
      }  
    }  
}  
```

