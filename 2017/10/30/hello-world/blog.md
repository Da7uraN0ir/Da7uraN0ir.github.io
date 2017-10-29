<!--markdown--># **blog配置历程** #

----------
## 0x00　aliyun配置 ##
远连过去，我这是个centos的系统盘，配置起来贼麻烦所以我选择修改系统盘为ubuntu。

![](http://i.imgur.com/cPeIeYc.png)

![](http://i.imgur.com/ZdpaaMT.png)
换成了ubuntu系统。

设置快照策略，也就是自动快照，这点很重要，除非你保证你的站永远不被搞。
还有就是重置密码。

## 0x01　远连进去
远连进去是一个纯命令行无界面linux.ubuntu。

	1. 安装lamp环境
	0x00 安装Apache2
	sudo apt-get install apache2
	
	0x01 安装PHP
	sudo apt-get install php
	sudo apt-get install libapache2-mod-php
	
	0x02 安装MySql
	sudo apt-get install mysql-server
	（安装中会提示设置数据库root的密码）
	sudo apt-get install mysql-client
	mysql_secure_installation
	（输入数据库root的密码，没有什么特殊需要的全部回车就可以了）
	#mysql_secure_installation
	执行出错的话，可以启动下mysql服务
	#sudo service mysql start
 
	0x03 安装PHP一些必要插件
	sudo apt-get install php-mysql php-gd php-imap php-ldap php-mbstring php-odbc php-pear php-xml php-xmlrpc
 
	0x04 测试
	进入网站根目录 #cd /var/www/html
	编辑测试文件   #vim test.php   
	（vim Ubuntu16.04貌似需要安装 sudo apt-get install vim
	vi 用法我会新开一篇来记录）
	<?php
	echo "Hello World!"
	?>
	在编辑时可能出现的问题:vim不能写入文件，是因为html是root-root且权限是755其他用户没有写入权限
	#sudo chmod 777 html
	1.先测试PHP编译
	#php test.php
	2.测试Apache解析
	#curl 127.0.0.1/test.php
	返回 Hello World! 则安装成功。
## 0x02　Typecho搭建
下载源码
[http://typecho.org/download](http://typecho.org/download)
![](http://i.imgur.com/xY8ySTp.png)
下载下来是一个压缩包，解压后将下图这个目录用FTP工具上传到服务器的网站根目录
![](http://pic4.zhimg.com/v2-4584944e2bdbf15c2330c49fec4467e7_b.png)
![](http://i.imgur.com/Z8MqzUf.png)
这里注意用sftp模式
![](http://i.imgur.com/zrFoTnd.jpg)
开始我建立了独立的文件夹来保存
然后绑定域名
然后开始安装
![](http://pic3.zhimg.com/v2-66f34e4dd11be8919289082b3530103a_b.png)
填写信息
![](http://pic2.zhimg.com/v2-06f70381df0a39e8433db4ebe6107151_b.png)
这一步可能遇上数据库找不到的问题，可以自己安装扩展
然后大功告成
![](http://pic1.zhimg.com/v2-d8c6a61753018a16e896836fb15d9410_b.png)
然后访问http://120.24.64.234/50n9/

觉得这样不好看，想要直接域名访问

修改下apache的配置吧

	vi /etc/apache2/sites-available/000-default.conf
找到DocumentRoot /var/www/改成你自己的目录

重启apache

	/etc/init.d/apacge2 restart

修改默认网页

	vi /etc/apache2/mods-available/dir.conf

原来是：

	<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm 
	</IfModule>

添加上想要的/typecho就行啦~

	<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm /typecho
	</IfModule>
但是发现主页的样式丢失，懵逼了，没找到解决方法。

所以我直接把文件全部放在根目录，好了可以直接访问了。

到此，这个blog就完成了。
