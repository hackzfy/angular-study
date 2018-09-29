# 在centos7上安装mysql5.7

> 在centos上安装数据库对新手来说是很折腾人的。本文只说使用yum安装的步骤。假设你使用的root管理员账号。



> 下载yum源文件

```
# wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
```

> 这将在当前目录下载一个 rpm文件。使用 ls 命令查看一下是否存在。

```
# ls
mysql57-community-release-el7-11.noarch.rpm
```

> 添加yum源

```
# rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
```

> 查看是否添加成功

```
# yum list | grep mysql-community-server
mysql-community-server.x86_64  5.7.22-1.el7  @mysql57-community
```

> 安装

```
#  yum install mysql-community-server
```

> 启动 , 提示输入密码的时候，输入用户密码。

```
#  systemctl start mysqld
```

> 查看是否启动成功

```
# systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-04-24 12:56:56 CST; 2min 49s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 737 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid #MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 720 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 741 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─741 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

> 看到以上信息说明启动成功。去登录

```
# mysql -uroot -p
// 你不知道密码，无法登录。
```

> 获取密码

```
# grep 'temporary password' /var/log/mysqld.log
2018-04-24T03:16:32.226348Z 1 [Note] A temporary password is generated for root@localhost: Xc0lKqREzl!5
```

> 再次尝试登录

```
# mysql -uroot -p
Enter password:    // 这里输入上面的到的密码：Xc0lKqREzl!5

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.22 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> status;

--------------
mysql  Ver 14.14 Distrib 5.7.22, for Linux (x86_64) using  EditLine wrapper

Connection id:		3
Current database:	mysql
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.7.22 MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	latin1  # ----> 注意这里编码有问题
Db     characterset:	latin1 # ----> 注意这里编码有问题
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			2 min 3 sec

Threads: 1  Questions: 45  Slow queries: 0  Opens: 136  Flush tables: 1  Open tables: 129  Queries per second avg: 0.365
--------------


```

> 登录成功。但是还没完，先修改密码，初始密码不安全。

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码，要包含数字字母下划线';
```

> 使用新密码重新登录，现在开始修改字符编码。

```
#  vi /etc/my.cnf
```

> 加入下面

```
[client]
default-character-set=utf8

[mysqld] # 已经有mysqld标签，要将下面的内容加入标签下面
character-set-server=utf8
collation-server=utf8_general_ci
```

> 重启服务

```
#  systemctl restart mysqld
```

> 进入mysql 查看 status

```
--------------
mysql  Ver 14.14 Distrib 5.7.22, for Linux (x86_64) using  EditLine wrapper

Connection id:		2
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.7.22 MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			21 sec

Threads: 1  Questions: 5  Slow queries: 0  Opens: 105  Flush tables: 1  Open tables: 98  Queries per second avg: 0.238
--------------

```

> 新建一个数据库看看编码。

```
mysql> create database mydb;
Query OK, 1 row affected (0.00 sec)

mysql> show create database mydb;
+----------+---------------------------------------------------------------+
| Database | Create Database                                               |
+----------+---------------------------------------------------------------+
| mydb     | CREATE DATABASE `mydb` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)
```

> 用root登录数据库不安全，所以最后新建一个数据库用户。（这和操作系统用户不是一个概念）。

```
mysql> create user 'yourname'@'yourhost' identified by 'abc123';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

> mysql 报错，意思是我的密码达不到安全标准。所以你要创建一个够长，包含数字，字母，下划线的密码。

```
mysql> create user 'yourname'@'yourhost' identified by 'Ab#eng_1Tsfa';
Query OK, 0 rows affected (0.00 sec)
```

> 开始授权

```
 mysql> grant all on mydb.* to 'yourname'@'yourhost';
```

> 意思是授予 'yourname'@'yourhost'用户对 mydb 库下所有表的增删改查权利。查看是否授权成功。

```
mysql> SHOW GRANTS FOR 'yourname'@'yourhost';
+-------------------------------------------------------------+
| Grants for yourname@yourhost                                   |
+-------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'yourname'@'yourhost'                    |
| GRANT ALL PRIVILEGES ON `mydb`.`*` TO 'yourname'@'yourhost' |
+-------------------------------------------------------------+
```

> 退出mysql的root账号，使用新账号登录。

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
+--------------------+
2 rows in set (0.00 sec)
```

> 其他的数据库对这个用户来说是不可见不可操作的。
>
> 就到这里了，再见！👋
