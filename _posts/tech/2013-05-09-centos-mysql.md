---
layout: post
title: "cenos6 install mysql5.6"
tagline: "linux mysql install"
description: ""
category : tech
tags: [linux, mysql]
---
{% include JB/setup %}

1、MySQL免安装版/二进制版软件，不用编译，下载地址：
     http://dev.mysql.com/downloads/
     文件格式：MYSQL-VERSION-OS.tar.gz

2、创建mysql组，建立mysql用户并加入到mysql组中。
     （不同版本的Unix中，groupadd和useradd的语法可能会稍有不同。）
     #groupadd mysql
     #useradd -g mysql mysql

3、进入目录/usr/local，解压缩免安装版，并在此目录中建立名为mysql的软链接。
     #cd /usr/local
     #gunzip < /path/to/MYSQL-VERSION-OS.tar.gz | tar xvf -
     (该命令会在本目录下创建一个名为MYSQL-VERSION-OS的新目录。)
     (使用GNU tar，则不再需要gunzip。你可以直接用下面的命令来解包和提取分发：
          #> tar zxvf /path/to/mysql-VERSION-OS.tar.gz)

     #ln -s MYSQL-VERSION-OS mysql


4、添加MySQL配置文件。
     如果你想要设置一个选项文件，使用support-files目录中的一个作为模板。在这个目录中有4个模板文件，是根据不同机器的内存定制的。
     #cp support-files/my-medium.cnf /etc/my.cnf
     (可能你需要用root用户运行这些命令。)
5、安装依赖
     #yum -y install binutils compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel gcc gcc-c++ glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel make sysstat
6、设定目录访问权限，用mysql_install_db创建MySQL授权表初始化，并设置mysql,root帐号访问权限。
     #cd mysql
     #chown -R mysql .
     #chgrp -R mysql .
     #scripts/mysql_install_db --user=mysql
     #chown -R root .
     #chown -R mysql data
     (注意以上命令中的" . "符号不能少。)

7、运行mysql
     #bin/mysqld_safe --user=mysql &
      (如果没有问题的话,应该会出现类似这样的提示:
              [1] 42264
              # Starting mysqld daemon with databases from /usr/local/mysql/var
       如果出现 mysql ended这样的语句，表示Mysql没有正常启动，你可以到log中查找问题，Log文件的通常在/etc/my.cnf中配置。
       大多数问题是权限设置不正确引起的。 )

8、设置root密码。默认安装密码为空，为了安全你需要修改密码。
     #/usr/local/mysql/bin/mysqladmin -uroot password yourpassword

9、拷贝编译目录的一个脚本，设置开机自动启动。
     #cp  support-files/mysql.server /etc/rc.d/init.d/mysqld
     #chmod 700 /etc/init.d/mysqld
     #chkconfig --add mysqld
     #chkconfig --level 345 mysqld on

10、启动mysqld服务。
     #service mysqld start

11、查看3306端口是否打开。要注意在防火墙中开放该端口。
     #netstat -atln

免安装版/二进制版安装基本命令概述：

<pre class="prettyPrint">
# groupadd mysql<br />
# useradd -g mysql mysql<br />
# cd /usr/local<br />
# gunzip &lt; /PATH/TO/MYSQL-VERSION-OS.tar.gz | tar xvf -<br />
# ln -s FULL-PATH-TO-MYSQL-VERSION-OS mysql<br />
# cd mysql<br />
# chown -R mysql .<br />
# chgrp -R mysql .<br />
# yum -y install binutils compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel gcc gcc-c++ glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel make sysstat
# scripts/mysql_install_db --user=mysql<br />
# chown -R root .<br />
# chown -R mysql data<br />
# bin/mysqld_safe --user=mysql &amp;
</pre>