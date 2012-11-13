---
layout: default
title: linux下如何安装mysql 5.5
---
故事是这样滴~这样滴~  
因为redis试完了，当然就想用nginx连上mysql嘛， 虽然原来的服务器上dba已经装了几个mysql，但是我不知道dba是谁，所以肯定不会厚颜无耻的去
找一个不知道是谁的人要一个用户和密码的对吧。所以我就打算自己安装一个，可是老子哟~百度了一下安装的教程，全都是说新建一个mysql的用户，
然后用mysql的用户来操作。可是涅~mysql这个用户已经被原来的dba拿去用了，我才不会厚颜无耻的去问他密码呢。所以涅~我就用我当前的
用户安装了。可是对一个从来没用过数据库的人来说，连基本的语句都不会，安装起来还真是蛋疼。（好吧，我承认我很无知），安装了半天，
结果老是安装不行，然后我就打算偷懒，先问同事，同事们呐，你们谁有mysql呀，给我一个做测试呀~结果同事尼玛一个两个都说没有。
╮(￣▽￣")╭ ...然后我就问，那你们谁会装啊~结果一个两个都说很麻烦，或者在忙呢...╮(￣▽￣")╭ ...然后我就屁颠屁颠的自己学怎么安装了。
昨天弄了一下午没成果，昨晚睡了个好觉以后，今早一来就搞定了。咳咳~果然是休息好才能干好活呀~  
旁白：够了啊喂！(#`O′)  这可是篇安装教程的帖子啊~不是吐槽贴啊喂！(#`O′) ！！！  
咳咳~好了，转正题.  
我这里说的是一个很随便的方法，严谨的话随便百度一下，得到的都是很严谨的答案。像[百度文库的这篇](http://wenku.baidu.com/view/fdb43e350b4c2e3f572763ab.html)也蛮言简意赅的。 
1，首先你需要下载一个[源码包](http://dev.mysql.com/downloads/mysql/#downloads)~（不是rpm包哟，如果你下的是rpm包那个点下这个[链接](http://www.2cto.com/database/201111/111980.html)，这是介绍rpm包安装的，我没试过，纯粹是友情推荐，^-^ ，我是不是很有爱涅？O(∩_∩)O哈哈哈~）  
(ps:其实我觉得源码包安装的话，这篇[文章](http://www.jb51.net/article/28751.htm)也不错,推荐之。“喂~两种你都推荐看其它教程了，那你这篇文章还有什么意义啊喂？”“额，这...算是补丁嘛~可能会出现的问题你可以参考一下嘛”“额~Pia!(ｏ ‵-′)ノ”(ノ﹏<。) ”)  
因为我机子上遗留dba君留下的源码包（mysql-5.5.15.tar.gz），所以我就不用下了，很大的说...
然后你要先下个cmake 
>
    安装cmake（mysql5.5以后是通过cmake来编译的）  
　　# wget http://www.cmake.org/files/v2.8/cmake-2.8.5.tar.gz   
　　# tar zxvf cmake-2.8.4.tar.gz 
　　# cd cmake-2.8.4  
　　#./configure  
　　# make && make install  
    然后下载解压mysql 5.5.15
　　wget http://mirrors.sohu.com/mysql/MySQL-5.5/mysql-5.5.15.tar.gz  
　　[root@localhost down]# tar zxvf mysql-5.5.15.tar.gz 
　　[root@localhost down]# cd mysql-5.5.15  
    接下来你可能会发现用户mysql是一个wheel的用户，这时候你要修改下权限，比如  
    chown -R root:wangbicheng mysql (印象中好像是这样写的，对吧~？) 
    接下来是一推乱七八糟的配置  
    cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/service/data/ -DMYSQL_UNIX_ADDR=/service/data/mysqld.sock -DWITH_INNOBASE_STORAGE_ENGINE=1 -DSYSCONFDIR=/etc -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_unicode_ci  -DWITH_DEBUG=0 
    前面三个参数我猜你知道怎么配置的啦，就是你数据库的路径呀，服务器数据目录，地址啊虾米的，具体我也不知道（呼~嘘声一片~）  

接下来就是make 和 make install了。  
等待的时间你可以出去找你室友玩一会，如果你不是用虚拟机的话，你还可以打一盘dota~   
好了，用6.67版本的小黑爽完以后，你可以继续回来安装了。接下来是要拷贝一些东西。  
>   
    cp support-files/my-medium.cnf /etc/my.cnf  
    cp support-files/mysql.server /etc/init.d/mysqld  
>
然后修改下my.conf里面的端口（因为原来的端口可能已经被人用了）
>
    [root@qrwefsdf software]# vim /etc/my.cnf   
    #password       = your_password
    port            = 8806
    socket          = /tmp/mysqld.sock
    
    # Here follows entries for some specific programs
    
    # The MySQL server
    [mysqld]
    port            = 8806
    socket          = /tmp/mysqld.sock
>
然后修改下服务器的配置:(不是全按下面的配置配啊，我是按我自己的路径来的啊喂~，你们也要一样按自己的路径来啊喂~如果你不知道你往下看
几行脚本你也知道路径是什么了啊喂~因为脚本里面有啊喂~)   
>
[root@qrwefsdf software]# vim /etc/init.d/mysqld 
 basedir=/usr/local  
 datadir=/home/server/mysqldata 
>
这时候你差不多就可以开开心心的运行mysql了。请运行：   
 /etc/init.d/mysqld start
如果你不幸的发现有这个提示：  
     Starting MySQL.. ERROR! The server quit without updating PID file (/usr/local/mysql/data/localhost.localdomain.pid) 
那么说明你权限不足，你可以试试解决方法：编辑/etc/init.d/mysql，找到start模块，添加--user=root到mysqld_safe。。。后面就可以  
例如：  
      $bindir/mysqld_safe --datadir="$datadir"  --user="bbc"   --pid-file="$mysqld_pid_file_path" $other_args >/dev/null 2>&1 & 
这样应该就差不多了。上面的bbc是我的用户名，你可以换成你的。就这样应该就差不多了吧。但是引起这个错误的可能还有其它方法，比如
你的文件夹权限不足什么的，具体请百度（因为18big召开，google你估计不能用了） 
        /etc/init.d/mysql start
    然后就可以了。  
接下来演示的是一些小实例:   
>
        [root@rac2 ~]# mysql -u root -p
        Enter password:
        Welcome to the MySQL monitor.  Commands end with ; or \g.
        Your MySQL connection id is 6
        Server version: 5.5.15 MySQL CommunityServer (GPL)
         
        Copyright (c) 2000, 2010, Oracle and/or itsaffiliates. All rights reserved.
         
        Oracle is a registered trademark of OracleCorporation and/or its
        affiliates. Other names may be trademarksof their respective
        owners.
         
        Type 'help;' or '\h' for help. Type '\c' toclear the current input statement.
         
        --查看已经存在的数据库，这几个是安装时自动创建的
        mysql> show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | test               |
        +--------------------+
        4 rows in set (0.00 sec)
         
        --使用mysql 数据库
        mysql> use mysql
        Reading table information for completion oftable and column names
        You can turn off this feature to get aquicker startup with -A
         
        Database changed
         
        --查看mysql 数据库下面的表
        mysql> show tables;
        +---------------------------+
        | Tables_in_mysql           |
        +---------------------------+
        | columns_priv              |
        | db                        |
        | event                     |
        | func                      |
        | general_log               |
        | help_category             |
        | help_keyword              |
        | help_relation             |
        | help_topic                |
        | host                      |
        | ndb_binlog_index          |
        | plugin                    |
        | proc                      |
        | procs_priv                |
        | proxies_priv              |
        | servers                   |
        | slow_log                  |
        | tables_priv               |
        | time_zone                 |
        | time_zone_leap_second     |
        | time_zone_name            |
        | time_zone_transition      |
        | time_zone_transition_type |
        | user                      |
        +---------------------------+
        24 rows in set (0.01 sec)
         
        --创建数据库Dave
        mysql> create database dave;
        Query OK, 1 row affected (0.01 sec)
         
        --在dave 数据库下面创建dave表：
        mysql> use dave;
        Database changed
        mysql> create table dave(id int,namechar(20));
        Query OK, 0 rows affected (0.01 sec)
         
        --查看dave 表结构：
        mysql> desc dave;
        +-------+----------+------+-----+---------+-------+
        | Field | Type     | Null | Key | Default | Extra |
        +-------+----------+------+-----+---------+-------+
        | id   | int(11)  | YES  |     | NULL   |       |
        | name | char(20) | YES  |     | NULL   |       |
        +-------+----------+------+-----+---------+-------+
        2 rows in set (0.00 sec)
         
        --向dave 表里插入2条记录：
        mysql> insert into davevalues(1,'dave');
        Query OK, 1 row affected (0.00 sec)
         
        mysql> insert into davevalues(2,'anqing');
        Query OK, 1 row affected (0.01 sec)
         
        mysql> select * from dave;
        +------+--------+
        | id  | name   |
        +------+--------+
        |   1 | dave   |
        |   2 | anqing |
        +------+--------+
        2 rows in set (0.00 sec)
         
        --update 表：
        mysql> update dave set name='david dai'where id=1;
        Query OK, 1 row affected (0.01 sec)
        Rows matched: 1  Changed: 1 Warnings: 0
         
        mysql> select * from dave;
        +------+-----------+
        | id  | name      |
        +------+-----------+
        |   1 | david dai |
        |   2 | anqing    |
        +------+-----------+
        2 rows in set (0.00 sec)
         
        --drop 表：
        mysql> drop table dave;
        Query OK, 0 rows affected (0.01 sec)
         
        --drop 数据库：
        mysql> drop database dave;
        Query OK, 0 rows affected (0.00 sec)
         
        mysql> show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | test               |
        +--------------------+
        4 rows in set (0.01 sec)
 
 ps:
>
        1.新建用户。
        
        //登录MYSQL
        @>mysql -u root -p
        @>密码
        //创建用户
        mysql> insert into mysql.user(Host,User,Password) values("localhost","phplamp",password("1234"));
        //刷新系统权限表
        mysql>flush privileges;
        这样就创建了一个名为：phplamp  密码为：1234  的用户。
        
        然后登录一下。
        
        mysql>exit;
        @>mysql -u phplamp -p
        @>输入密码
        mysql>登录成功
        
        2.为用户授权。
        
        //登录MYSQL（有ROOT权限）。我里我以ROOT身份登录.
        @>mysql -u root -p
        @>密码
        //首先为用户创建一个数据库(phplampDB)
        mysql>create database phplampDB;
        //授权phplamp用户拥有phplamp数据库的所有权限。
        >grant all privileges on phplampDB.* to phplamp@localhost identified by '1234';
        //刷新系统权限表
        mysql>flush privileges;
        mysql>其它操作
        
        /*
        如果想指定部分权限给一用户，可以这样来写:
        mysql>grant select,update on phplampDB.* to phplamp@localhost identified by '1234';
        //刷新系统权限表。
        mysql>flush privileges;
        */
        
        3.删除用户。
        @>mysql -u root -p
        @>密码
        mysql>DELETE FROM user WHERE User="phplamp" and Host="localhost";
        mysql>flush privileges;
        //删除用户的数据库
        mysql>drop database phplampDB;
        
        4.修改指定用户密码。
        @>mysql -u root -p
        @>密码
        mysql>update mysql.user set password=password('新密码') where User="phplamp" and Host="localhost";
        mysql>flush privileges;
ps2:
>   
        查询数据库所有用户:
        SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;        
        查看mysql的当前登陆用户 
        select user();
        使用mysql登录数据库后，如何查看当前数据库
        1. 通过函数database()实现
            mysql> SELECT database();
        2. 通过show tables实现
            mysql> show tables;
            注：注意查看列头形式, 格式为：Tables_in_[db_name]
            www.2cto.com  
        3. 通过status实现
            mysql> status;
            注：注意结果中的"Current database"信息
        4.查看端口
            mysql> show variables like 'port';





