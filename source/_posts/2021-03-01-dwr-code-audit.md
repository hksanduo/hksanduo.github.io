---
title: "dwr框架代码审计"
date:   2021-03-01  21:30:00 +0800
layout: post
tag:
- Code
categories:
- Security
- Code Audit
---

dwr框架代码审计

-------

## 简述
在做测试的过程中发现一个老系统提交的请求包比较奇怪，通过检索，发现是早期java web研发人员常用的DWR框架，dwr框架比较古老，可以帮助用户实现Ajax网站，可以让你在浏览器中的Javascript代码调用Web服务器上的Java，就像在Java代码就在浏览器中一样。</br>
![20210301-01](/images/20210301-01.png)

DWR 主要包括两部分：

    - 在服务器上运行的 Servlet 来处理请求并把结果返回浏览器。
    - 运行在浏览器上的 Javascript，可以发送请求，并动态改变页面。

DWR 会根据你的 Java 类动态的生成Javascript代码。这些代码让你感觉整个Ajax调用都是在浏览器上发生的，但事实上是服务器执行了这些代码，DWR负责数据的传递和转换。DWR有些过时了，目前可以直接使用jquery来实现ajax请求。

## DWR框架介绍
使用maven或者直接从官网获取jar包，将jar包放入WEB-INF的lib文件夹下。同时，dwr依赖于commons-logging.jar
web.xml配置文件如下：
```
<servlet>
  <servlet-name>dwr-invoker</servlet-name>
  <servlet-class>uk.ltd.getahead.dwr.DWRServlet</servlet-class>
  <init-param>
    <param-name>debug</param-name>
    <param-value>true</param-value>
  </init-param>
</servlet>
 
<servlet-mapping>
  <servlet-name>dwr-invoker</servlet-name>
  <url-pattern>/dwr/*</url-pattern>
</servlet-mapping>
```
dwr配置如下：
```
<!DOCTYPE dwr PUBLIC
    "-//GetAhead Limited//DTD Direct Web Remoting 3.0//EN"
    "http://getahead.org/dwr/dwr30.dtd">

<dwr>
  <allow>
    <create creator="new" javascript="JDate">
      <param name="class" value="java.util.Date"/>
    </create>
    <create creator="new" javascript="Demo">
      <param name="class" value="your.java.Bean"/>
    </create>
  </allow>
</dwr>
```
dwr.xml 是 dwr 的核心配置文件，主要的标签有：\<converter\>、\<convert\>、\<create\>这三个标签。\<create\> 标签是 dwr 中重要的标签，用来描述 java（服务器端） 与 javascript （客户端）的交互方式。其基本格式如下：
```
<allow>
  <create creator="..." javascript="..." scope="...">
    <param name="..." value="..."/>
    <auth method="..." role="..."/>
    <exclude method="..."/>
    <include method="..."/>
  </create>
  ...
</allow>
```
其中，creator 和 javascript 是必须属性，其他可以忽略。creator 包含有以下几个值：
　　
- new：Java用“new”关键字创造对象
- none：它不创建对象  (v1.1+)
- scripted：通过BSF使用脚本语言创建对象，例如BeanShell或Groovy
- spring：通过Spring框架访问Bean
- struts：使用Struts的FormBean  (v1.1+)
- jsf：使用JSF的Bean  (v1.1+)
- pageflow：访问Weblogic或Beehive的PageFlow  (v1.1+)
- ejb3：使用EJB3 session bean  (v2.0+)

这里初学，实用java new创建对象。
页面配置
页面需要引入3个JS
```
<script src="<%=ctxPath%>/dwr/interface/Chat.js" type="text/javascript"></script>
<script src="<%=ctxPath%>/dwr/engine.js" type="text/javascript"></script>
<script src="<%=ctxPath%>/dwr/util.js" type="text/javascript"></script> 
```
其中 engine.js 必须要，如果需要用到dwr提供的一些方便的工具要引用util.js ，然后是dwr自动生成的js文件。名字和 dwr.xml 中 create 标签的 javascript 属性值一样，且必须是 dwr/interface 开头的目录，详细请查阅官方文档，这里不过多说明。

## 审计
DWR框架对于审计人员比较友好，通过分析web.xml和dwr配置文件，我们可以迅速定位到源代码的位置。
首先分析web.xml
![20210301-02](/images/20210301-02.png)
获取dwr配置文件目录
![20210301-03](/images/20210301-03.png)
这里查找scriptName为：CaUsermanAjax，方法名为：getTysfyhbList的具体实现的代码。
本次审计的dwr框架配置文件使用spring模式，使用spring框架访问bean,并且使用dwr-signatures的配置模式，signatures段使DWR能确定集合中存放的数据类型。
signatures段允许我们暗示DWR应该用什么类型去处理。格式对以了解JDK5的泛型的人来说很容易理解。

```
	<signatures>
	  <![CDATA[
	    import java.util.Map;
	    import com.zfsoft.cacommon.username.ajax.UsermanAjax;
	    CaUsermanAjax.getYrdgxxbList(Map<String,String> condition);
	    CaUsermanAjax.getTysfyhbList(Map<String,String> condition);
	    CaUsermanAjax.getOnlineUserList(Map<String,String> condition);
	    CaUsermanAjax.getZhmmxxbList(Map<String,String> condition);
	    ]]>
	</signatures> 
```
<signatures>标签是用来声明java方法中List、Set或者Map参数所包含的确切类，以便java代码作出判断。
通过查找com.zfsoft.cacommon.username.ajax.UsermanAjax对象，我们找到getTysfyhbList方法。
![20210301-04](/images/20210301-04.png)
通过对比捕获的数据包我们可以了解到数据包的格式
![20210301-05](/images/20210301-05.png)

请求地址为:
```
http://ipaddress/dwr/call/plaincall/{{ scriptName }}.{{ methodName }}.dwr
```
http body:
```
callCount=1
page={{ 任意JSP地址均可 }}
httpSessionId= {{ 可留空 }}
scriptSessionId={{Cookie中的 DWRSESSION}}
c0-scriptName={{ scriptName }}
c0-methodName={{ methodName }}
c0-id=0
c0-e1=number:0  设置参数
c0-e2=number:15
c0-e3=string:
c0-param0=string(类型): 值 {{ 参数 1}}
也可以写成：c0-param0=Object_Object:{start:reference:c0-e1,...}start是参数名，后面是参数值的来源。
batchId=0
```

回过头，我们分析本次审计的源代码，其中TysfyhbDAO对象中的list方法会返回用户的数据信息，未经过处理，直接返回数据库中存储的用户敏感信息。在请求调用的整个过程中，未发现有相关的权限校验，存在未授权访问的风险。

## 参考
- [http://directwebremoting.org/dwr/index.html](http://directwebremoting.org/dwr/index.html)【dwr wiki】
- [https://xz.aliyun.com/t/8431](https://xz.aliyun.com/t/8431)
- [https://xz.aliyun.com/t/2147](https://xz.aliyun.com/t/2147)
- [https://blog.csdn.net/smileyan9/article/details/80545795](https://blog.csdn.net/smileyan9/article/details/80545795)
- [https://bbs.csdn.net/topics/390981028](https://bbs.csdn.net/topics/390981028)
