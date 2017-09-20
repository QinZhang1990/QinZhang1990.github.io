---
layout: post
title: "MySQL的安装与配置"
date: 2017-09-20 20:23:00.000000000
tags: 他山之石
---
### 1 下载

（1）下载地址：https://www.mysql.com/——DOWNLOADS——Community

 此处有几种版本供选择，大多情况下选择社区（community）版，该版本免费使用，但不提供技术支持。

（2）社区版有两种格式供下载即msi格式和ZIP压缩包格式；msi即通过常见的.exe文件来安装，ZIP压缩包则无需安装，解压即可，然后再进行配置。

  	![](/assets/images/2017/mysql_deploy/msq1.PNG)
  
  ### 2 配置
  
  解压后的安装包，选择合适的位置存放，如D盘，文件夹名为（mysql-5.7.18）
（1）配置环境变量
	我的电脑——属性——高级系统设置——环境变量——用户变量：在用户变量里的path的最后添加“D:\mysql-5.7.18\bin;”然后点击“确定”（此处注意分号）。配置成功后，效果如下：
	
	![](/assets/images/2017/mysql_deploy/msq2.PNG)

（2）配置初始化文件`my.ini`
	MySQL的安装包从版本5.7.6之后，不再包含data目录（该目录用于存放数据表中的数据），可以通过my.ini文件来进行初始化生成，将my.ini初始化文件直接保存在mysql的主目录下，简单的版本如下：
      
```swift
[mysqld]
# The TCP/IP Port the MySQL Server will listen on
port=3306
# Path to installation directory. All paths are usually resolved relative to this.
basedir="D:\mysql-5.7.18"
# Path to the database root
datadir="D:\mysql-5.7.18\data"
# The default character set that will be used when a new schema or table is
# created and no character set is defined
character-set-server=utf8
# The default storage engine that will be used when create new tables when
default-storage-engine=INNODB
# Set the SQL mode to strict
sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
# Enable Windows Authentication
# plugin-load=authentication_windows.dll
# General and Slow logging.
log-output=NONE
general-log=0
general_log_file="qzhang-PC.log"
slow-query-log=0
slow_query_log_file="qzhang-PC-slow.log"
long_query_time=10
# Binary Logging.
# log-bin
# Error Logging.
log-error="qzhang-PC.err"
```

（3）启动DOS窗口进行初始化（WIN+R；以管理员的形式运行），Dos界面打开之后，进入到bin目录下：键入mysqld –initialize

	![](/assets/images/2017/mysql_deploy/msq3.PNG)

初始化之后，mysql目录下便会产生data目录，并且会生成一个root用户和用于登录数据库的随机密码。

	![](/assets/images/2017/mysql_deploy/msq4.PNG)

随机密码的查看有两种方式：

1）打开data目录下的.err文件，找到“[Note] A temporary password is generated for root@localhost: M*-x>jLj1Lw9”这样一句话, root@localhost:之后即为密码；

2）“我的电脑”——管理——“事件查看器”——Windows日志——应用程序

	![](/assets/images/2017/mysql_deploy/msq5.PNG)

（4）将mysql注册到Windows系统服务，此处同样需要在bin目录下（否则下一步启动mysql服务时，会提示“报发生系统错误2”；
http://blog.csdn.net/zhouyufengqingyang/article/details/46291057）， 打开dos界面，然后键入“mysqld install mysql”，若删除服务则“mysqld remove mysql”（mysql为服务的名字，可以任意命名）。

（5）启动和关闭mysql服务，该项操作需要在bin目录下，操作dos界面，在dos中键入“net start mysql”；关闭服务即“net stop mysql”。

	![](/assets/images/2017/mysql_deploy/msq6.PNG)
 
（6）登录MySQL，在bin目录下通过dos界面登录MySQL，键入“mysql –u root –p”；

![](/assets/images/2017/mysql_deploy/msq7.PNG)

（7）更改初始密码，登录之后，在dos界面“mysql>”之后键入 “set password=‘123456’”；然后通过“flush privileges”进行权限更新

	![](/assets/images/2017/mysql_deploy/msq8.PNG)

（8）验证登录；重新打开dos界面（不需要在bin目录下），键入“mysql –u root -p”

	![](/assets/images/2017/mysql_deploy/msq9.PNG)







 

