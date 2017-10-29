<!--markdown--># 1.什么是文件上传漏洞
文件上传漏洞是说用户上传了一个可执行的脚本文件（就是一个能在服务器上被解析执行的文件），并通过这个脚本文件获得执行服务端命令的能力。  
这种漏洞往往是为直接和致命的。文件上传这个功能本身是没有问题的，问题出现在文件上传之后服务器怎么去出来文件，解释文件。如果服务器的处理逻辑做的不够安全，就会导致攻击者拿到他想拿到的东西。  
本文将会从文件上传的途中会遇到哪些阻拦开始讲，到最后的整体渗透思路介绍。
# 2.文件上传的危害
## 2.1 上传的文件时web脚本语言
服务器的web容器就会解释并执行用户上传的脚本，导致代码的执行。
## 2.2 上传的是 flash的策略文件crossdomain.xml
黑客用以控制flash在还域下的行为（其他通过类似方式控制策略文件的情况类似）。   
也就是说，我在你这里改掉你的安全策略，就相当于把你的防盗门换成了推拉门，安全性能自然下降，或者将别人访问你的web重定向到攻击者的钓鱼网站，则危害也是很大的。   
## 2.3上传文件是病毒、木马文件
黑客用以诱骗用户或者管理员下载执行。
## 2.4上传文件是钓鱼图片或为包含了脚本的图片
在某些版本的浏览器中会被作为脚本执行，被用于钓鱼和欺诈。除此之外，还有一些不常见的利用方法，比如将上传文件作为一个入口，溢出服务器的后台处理程序，如图片解析模块;或者上传一个合法的文本文件，其内容包含了PHP脚本，再通过"本地文件包含漏洞(Local File Include)"执行此脚本;等等。  
# 3.漏洞存在的条件
## 3.1 用户上传的文件能被web容器解释执行
无法通过web容器将其解释执行，就不会进一步的得到服务器的权限。
## 3.2 用户有访问上传的这个文件的权限。
如果攻击者没法访问上传的文件，则无法通过web容器将其解释执行，就不会进一步的得到服务器的权限。
## 3.3 有可以绕过的，或者没有文件安全监测
如果有，一则上传可能不成功，二则，文件解析不成功，都可以是攻击者攻击失败。
# 4.漏洞成因
## 4.1 IIS5.x-6.x解析漏洞
使用iis5.x-6.x版本的服务器，大多数厚实windows srver 2003的老古董，开发与剧一般为asp：该解析漏洞也只存在于asp文件，aspx都不行。  
目录解析（iis6.0）  
形式：www.xxx.com/xx.asp/xx,jpg  
服务器会默认吧.asp和.asp目录下的文件都解析为asp文件  
文件解析  
形式：www.xxx.com/xx.asp;.jpg  
服务器默认不解析;号后面的内容，因此xx.asp;.jpg便被解析成asp文件了  
解析文件类型   
Iis6.0 默认的可执行文件有.asp/.asa/.cer/.cdx   
## 4.2 apache 解析漏洞  
Apache 解析文件的规则是从右到左开始判断解析，如果后缀名为不可识别文件解析，就再往左判断一个。比如xx.php.qwe.asd  
.qwe和.asd这两种后缀是不可解析的，apache就会一直往前解析，解析到xx.php并执行。  
上传形式 xx.php,php123  
还有一些特殊配置造成的解析漏洞  
1.如果apache里面conf配置文件里面的配置AddHandler php5-script .php 这时只要文件名里包含.php 即使文件名是 test2.php.jpg 也会以 php 来执行。   
2.如果apache里面conf配置文件里面的配置AddType application/x-httpd-php .jpg 即使扩展名是 jpg，一样能以php 方式执行。   
## 4.3 nginx解析漏洞
nginx默认是以cgi的方式支持解析php的，普遍的做法实在nginx的配置文件中通过正则表达式匹配设置script_filename。  
当访问www.xx.com/phpinfo.jpg/1.php这个url时，$fastcgi_script_name会被设置为 “phpinfo.jpg/1.php” ，然后构造成SCRIPT_FILENAME传递给PHP CGI，但是PHP为什么会接受这样的参数，并将phpinfo.jpg作为PHP文件解析呢?这就要说到fix_pathinfo这个选项了。  
如果开启了这个选项，那么就会触发在PHP中的如下逻辑：PHP会认为SCRIPT_FILENAME是phpinfo.jpg，而1.php是PATH_INFO，所以就会将phpinfo.jpg作为PHP文件来解析了。  
漏洞形式  
www.xxxx.com/UploadFiles/image/1.jpg/1.php  
www.xxxx.com/UploadFiles/image/1.jpg%00.php  
www.xxxx.com/UploadFiles/image/1.jpg/%20\0.php  
或者直接上传名为test.jpg，内容是再同级目录下写一个shell.php的一句话木马,然后访问test.jpg/.php这样就是解析执行tesr.php生成shell
当然这个要看php的版本，5.5之后好像就已经不存在这个洞了  
## 4.4 iis7.5解析漏洞  
Iis7.5的漏洞和nginx的差不多，漏洞成因都是php的配置重开了  cgi.fix_pathinfo，而不是其本身的漏洞。  
## 4.5 配合操作系统文件命令规则  
如xx.asp./test.asp(空格)/总之一切能够不能出现在目录和文件名中的如/\:*?"<>  
这些符号来截断字符串，让服务器只保留前面的部分，而不会被过滤。  
![](http://i.imgur.com/VIqFkG1.png)  
![](http://i.imgur.com/GLiOY6y.png)  
然后前台，或者这个图片的任意输出位置都能得到它的物理路径。
然后连菜刀就能getshell  
# 5.绕过文件上传限制  
一般的文件文件上传限制都是用文件后缀名来判断文件是否为可上传文件。这时候根据语言的特性，可以选择不同的文件名截断方式来绕过对文件上传的限制。例如添加终止符0x00，比如xxx.php%00.jpg这里的.jpg被%00截断，传进服务器的还是xxx.php但是判断的时候后缀是jpg。所以，知道这里的校验方式对我们上传文件有很大的帮助  
## 5.1 客户端校验  
一般是再网页上写一段javascript脚本，校验上传文件的后缀，或白名单或黑名单。  
判断方法：在浏览加载文件时，但是还没有点击上传按钮的时候弹窗弹出错误，此时并没有真正地发出数据包再返回值。  
绕过方式：burp（burpsuit一个抓包工具，可百度）抓包，先上传一个能发出去的后缀类型的木马如gif,然后等他发包的时候再修改后缀。  
## 5.2 服务器端校验
 ### 5.2.1 content-type字段检测  
![](http://i.imgur.com/qqAN0UT.png)  
通过头信息里的content-tyoe字段检测  
头信息里面会附带文件类型，服务端对这个字段的内容进行检测获取文件类型，再决定文件是否能够上传。  
绕过方式：  
抓包修改content-type字段修改成允许上传的类型如image/gif  
### 5.2.2 文件头校验  
通过自己写的正则匹配，哦判断文件头内容是不是能够上传。  
> 1. .JPEG;.JPE;.JPG，”JPGGraphic File”  
> 2 .gif，”GIF 89A”  
> 3 .zip，”Zip Compressed”  
> 4 .doc;.xls;.xlt;.ppt;.apr，”MS Compound Document v1 or Lotus Approach APRfile”  

绕过方式：  
在木马内容基础上加一些文件信息，比如GIF  ->改GIF89a  
###5.2.3 扩展名校验
MIME验证  
MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。  
它是一个互联网标准，扩展了电子邮件标准，使其能够支持：  
非ASCII字符文本;非文本格式附件(二进制、声音、图像等);由多部分(multiple parts)组成的消息体;包含非ASCII字符的头信息(Header information)。  
这个标准被定义在RFC 2045、RFC 2046、RFC 2047、RFC 2048、RFC 2049等RFC中。 MIME改善了由RFC 822转变而来的RFC 2822，这些旧标准规定电子邮件标准并不允许在邮件消息中使用7位ASCII字符集以外的字符。正因如此，一些非英语字符消息和二进制文件，图像，声音等非文字消息原本都不能在电子邮件中传输(MIME可以)。MIME规定了用于表示各种各样的数据类型的符号化方法。 此外，在万维网中使用的HTTP协议中也使用了MIME的框架，标准被扩展为互联网媒体类型。  
MIME的作用  
使客户端软件区分不同种类的数据，例如web浏览器就是通过MIME类型来判断文件是GIF图片，还是可打印的PostScript文件。 Web服务器使用MIME来说明发送数据的种类，Web客户端使用MIME来说明希望接收到的数据种类。  
一个普通的文本邮件的信息包含一个头部分(To: From: Subject: 等等)和一个体部分(Hello Mr.,等等)。在一个符合MIME的信息中，也包含一个信息头并不奇怪，邮件的各个部分叫做MIME段，每段前也缀以一个特别的头。MIME邮件只是基于RFC 822邮件的一个扩展，然而它有着自己的RFC规范集。  
头字段：MIME头根据在邮件包中的位置，大体上分为MIME信息头和MIME段头。(MIME信息头指整个邮件的头，而MIME段头只每个MIME段的头。)
常见MIME类型  
![](http://i.imgur.com/KbS7imb.png)
mimntype判断  
一般先判断内容的前十个字节，来判断文件类型，然后再判断后缀名。  
文件扩展名绕过  
前提：黑名单校验  
黑名单检测：一般有个专门的 blacklist 文件，里面会包含常见的危险脚本文件。  
绕过方式：  
(1)找黑名单扩展名的漏网之鱼 - 比如 asa 和 cer 之类  
(2)可能存在大小写绕过漏洞 - 比如 aSp 和 pHp 之类  
能被解析的文件扩展名列表：  
jsp /jspx/ jspf  
Asp/ asa /cer/ aspx  
# 6.配合文件包含漏洞  
## 6.1 服务端只对传入的asp/php/jsp文件的内容做特性判断  
绕过方式  
比如php  
先上传一个一句话木马，后缀为txt，上传后不会被检测  
在上传一个php文件来包含这个txt文件，上传后检测，但是这里面没有一句话木马的特性函数，  
## 6.2 linux的后缀名大小写敏感特性  
linux下php和pHp的结果是不一样的，可用pHp绕过  
## 6.3 cms自身和编辑器漏洞  
cms 漏洞:cms自带漏洞，可能是过滤不严，可能是上传位置权限大。  
编辑器漏洞:编辑器插件的漏洞，这个需要大量的积累。  
配合其他规则  
(1)0x00截断：基于一个组合逻辑漏洞造成的，通常存在于构造上传文件路径的时候  
test.php(0x00).jpg  
test.php%00.jpg   
路径/upload/1.php(0x00)，文件名1.jpg，结合/upload/1.php(0x00)/1.jpg  
# 7.waf绕过  
## 7.1 垃圾数据  
这中检测是为了减少服务器负担，只对数据的都一段进行检测。攻击者可以先构造足够多的数据，再插入木马，就不会被检测到。这种过滤，多试几次，找到waf的规律就可以再合适的位置构造垃圾数据来绕过waf了。
## 7.2 filename  
这个是很早期的漏洞了，有一点HPP(HTTP Parameter Polution) 的感觉，就是上传的时候上传两个filename参数，服务器就只会判断一个，而解析另外一个。  
## 7.3 POST/GET/cookie  
这要看waf检测哪种传参方式，如果没有对所有传参方式进行过滤，那就给了攻击者攻击的方式。  
## 7.4 前文方式  
同样前文提到的解析漏洞，文件包含等漏洞都可以用过来。
#  8.文件上传引起的xss攻击  
## 8.1 触发点  
一个文件上传点是执行XSS应用程序的绝佳机会。很多网站都有用户权限上传个人资料图片的上传点，只要你能找到一个回显位置，你就有很多机会找到相关漏洞。  
## 8.2 漏洞成因  
显式输出文件名，上传安全策略文件，上传了可执行的javascript文件 
## 8.3 案例分析  
1.显式输出文件名  
文件名本身可能会显式反映在页面上，所以一个带有XSS明明的文件就足以进行XSS攻击   
![](http://i.imgur.com/OZRNmOF.png)  
![](http://i.imgur.com/4pduwoq.png)  
![](http://i.imgur.com/HjK45Cb.png)  
![](http://i.imgur.com/8RwcWHr.png)  
![](http://i.imgur.com/TWciEcz.png)  
像这种将上传的文件名显示在页面中的，我们就可以构造文件名使浏览器执行攻击者设定好的脚本命令。   
2.文件元数据   
漏洞点还是要有能地方能显示图片详情  
![](http://i.imgur.com/PgmK6Wl.png)  
这样修改某个元数据，再加上网站能找到这个数据的输出点，攻击着就可以实现脚本的插入，造成XSS攻击。  
3.VG格式的文件的Content  
如果应用允许上传SVG格式的文件（其实就是一个图像类型的），那么带有以下content的文件可以被用来触发XSS：
  
	<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>  
![](http://i.imgur.com/RfddLsZ.png)  
![](http://i.imgur.com/z1WelOw.png)  
这段代码只要存在于服务器中，用户访问就能造成XSS攻击，所以只要用户能上传一段这样内容的文件到服务器，在诱导其他用户或者管理点击就会获取他们的私密信息。  
4.源码引用，执行javascript脚本  
就像前文提到的，只要能引用，就能做我们想做的事儿，我们可以很容易的创建一张包含 javascript payload 的 GIF 图片，然后将这张图片当做源码加以引用。  
如果我们可以成功的注入相同的域名，如下所示，则这样可以有效的帮我们绕过 CSP（内容安全策略）防护（其不允许执行例如

	<script>alert(1)</script>  

）  
![](http://i.imgur.com/cEpNfwj.png)  
这里我们上传一个包含xss脚本的GIF图片，创建这样一张图片可以使用如下内容并将文件命名为 .gif 后缀：  

	 GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;  

GIF 文件标识 GIF89a 做为一个 javascript 的变量分配给 alert 函数。中间注释部分的 XSS 是为了以防图像被检索为 text/HTML MIME 类型时，通过请求文件来执行 payload。  
![](http://i.imgur.com/C01s98f.png)  
![](http://i.imgur.com/UqKVbD7.png)  
访问它，表示它已经上传成功  
![](http://i.imgur.com/6Nk8Qd3.png)  
![](http://i.imgur.com/hh5Y4He.png)  
这个的漏洞在于，如果对上传文件的过滤仅仅限于后缀，文件头等信息的过滤，我们可以利用这个文件本地包含（也就是利用这份代码来当源码），那么就可以造成xss漏洞。  