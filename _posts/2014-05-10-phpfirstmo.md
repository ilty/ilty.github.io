---
layout: post
title:  "PHP总结<1>"
date:   2014-05-10
desc: "我的php"
keywords: "php,mysql,pdo,session,cookie,navicat操作,搜索"
categories: [php]
tags: [epel]
icon: icon-centos
---

PDO:

PDO常用方法：

PDO::query()主要用于有记录结果返回的操作（PDOStatement），特别是select操作。

PDO::exec()主要是针对没有结果集合返回的操作。如insert,update等操作。返回影响行数。

PDO::lastInsertId()返回上次插入操作最后一条ID，但要注意：如果用insert into tb(col1,col2) values(v1,v2),(v11,v22)..的方式一次插入多条记录，lastinsertid()返回的只是第一条(v1,v2)插入时的ID,而不是最后一条记录插入的记录ID。

PDOStatement::fetch()是用来获取一条记录。配合while来遍历。

PDOStatement::fetchAll()是获取所有记录集到一个中。

PDOStatement::fetchcolumn([int column_indexnum])用于直接访问列，参数column_indexnum是该列在行中的从0开始索引值，但是，这个方法一次只能取得同一行的一列，只要执行一次，就跳到下一行。因此，用于直接访问某一列时较好用，但要遍历多列就用不上。

PDOStatement::rowcount()适用于当用query("select ...")方法时，获取记录的条数。也可以用于预处理中。$stmt->rowcount();

PDOStatement::columncount()适用于当用query("select ...")方法时，获取记录的列数。

```
<?php
/*连接*/
$dbh = new PDO('mysql:host=localhost;dbname=access_control', 'root', '');
$dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$dbh->exec('set names utf8');
/*添加*/
//$sql = "INSERT INTO `user` SET `login`=:login AND `password`=:password";
$sql = "INSERT INTO `user` (`login` ,`password`)VALUES (:login, :password)"; $stmt = $dbh->prepare($sql); $stmt->execute(array(':login'=>'kevin2',':password'=>''));
echo $dbh->lastinsertid();
/*修改*
$sql = "UPDATE `user` SET `password`=:password WHERE `user_id`=:userId";
$stmt = $dbh->prepare($sql);
$stmt->execute(array(':userId'=>'7', ':password'=>'4607e782c4d86fd5364d7e4508bb10d9'));
echo $stmt->rowCount();
/*删除*/
$sql = "DELETE FROM `user` WHERE `login` LIKE 'kevin_'"; //kevin%
$stmt = $dbh->prepare($sql);
$stmt->execute();
echo $stmt->rowCount();
/*查询*/
$login = 'kevin%';
$sql = "SELECT * FROM `user` WHERE `login` LIKE :login";
$stmt = $dbh->prepare($sql);
$stmt->execute(array(':login'=>$login));
while($row = $stmt->fetch(PDO::FETCH_ASSOC)){
print_r($row);
}
print_r( $stmt->fetchAll(PDO::FETCH_ASSOC));
?>
```

搜索高量显示：
这里提供下实现思路，具体实践，自己遇到了就会明白了
首先,要有关键字,通过关键字搜索,在搜索的最后一部可以通过循环吧所显示的某一个字段替换,就是把我们的遍历的字段在加入一个替换颜色的标签就OK啦！

Mysqli:

从php5.0开始增加mysql(i)支持 ,从而mysql连接就不支持了, 新加的功能都以对象的形式添加

i表示改进的意思 功能多、效率高、稳定

相对于mysql有很多新的特性和优势
（1）支持本地绑定、准备（prepare）等语法
（2）执行sql语句的错误代码
（3）同时执行多个sql
（4）另外提供了面向对象的调用接口的方法。

mysqli需要php.ini的配置文件，开启扩展编译时参数：
;extension=php_mysqli.dll将其修改为：extension=php_mysqli.dll即可。

使用传统的面向过程的方法 php代码如下：

```
    <?php
    $connect = mysqli_connect('localhost','root','','volunteer') or die('Unale to connect');
    $sql = "select * from vol_msg";
    $result = mysqli_query($connect,$sql);
    while($row = mysqli_fetch_row($result)){
    echo $row[0];
    }
    ?>
```

使用面向对象的方法调用接口（推荐使用）看php代码如下：

```
    <?php
    //创建对象并打开连接，最后一个参数是选择的数据库名称
    $mysqli = new mysqli('localhost','root','','volunteer');
    //检查连接是否成功
    if (mysqli_connect_errno()){
    //注意mysqli_connect_error()新特性
    die('Unable to connect!'). mysqli_connect_error();
    }
    $sql = "select * from vol_msg";
    //执行sql语句，完全面向对象的
    $result = $mysqli->query($sql);
    while($row = $result->fetch_array()){
    echo $row[0];
    }
    ?>
```

navicat 操作数据库：

Navicat Premium是一个功能强大的第三方数据库管理工具，可以连接管理MySQL、Oracle、PostgreSQL、SQLite 及 SQL Server数据库。
提供了简单数据库管理的基本和必需功能.它支持最新的功能包括触发器、函数、视图；而且还包含导入或导出工具,让用户导入和导出数据为纯文本文件格式,包括 TXT、CSV 和 XML.

Session与cookie:

Session与cookie的区别：
Cookie机制采用的是客户端保存状态的方案。
Session机制采用的是服务器端保存状态的方案。
Cookie:服务器通过http响应头中加一行指令来提示浏览器生成指定的cookie。Cookie的指令一般是由浏览器通过一定指令从后台发送到浏览器当中。Cookie的内容主要包括：名字、值、过期时间、路径和域。若不设置过期时间。则表示这个cookie的生命期为浏览器会话期间。关闭浏览器期间，cookie会自动消失，在这个时候，cookie是保存在内存中，而不是硬盘中。当然这种方法一般是不常用的。反之，如果我们设置了过期时间，浏览器就会把cookie的值保存到硬盘上，即使你关闭了浏览器再重新打开也是一样的。一直到你所设定的时间结束。
Session:是一种服务器端的机制，服务器使用的是一种散列表的结构。
当程序为某个客户端创建一个session的时候，服务器首先检测客户端这个请求中是否包含了session标识（session id）,如果已经包含则说明以前客户端中包含session标识，服务器就会通过session id 把这个session检索出来，如果没有，则会重新建立一个session。session类可以在我 们访问一个浏览器或网址的时候，对他进行一定的监控，对他们的状态进行跟踪。session类将每个用户的session进行信息序列化后存储到 cookie当中。并对他进行加密，当然你还可以进行把session存储到数据库当中以此来增加安全性。但是这是要求存储到cookie当中的 session id  与数据库当中的session id 进行匹配。程序默认只在cookie当中存储session.当页面进行载入的时候，session就会将cookie是否有效的存储到session当 中。如果seseion不存在或者已经过期，那么他就会生成一个新的session,并且把它绑定到cookie当中，如果session数据存在，那么 它就会对session进行更新，同时cookie也会被更新。每次更新都会生成session_id这个值。session一旦被初始化，他就会自动运行。

session :
启动会话函数：session_start();
通过调用 session_id()函数来访问会话ID，session_name()函数显示会话的名称，也就是php 返回给客户机的保存会话ID的cookie名称。
通过调用 session_id()函数来访问会话ID，session_name()函数显示会话的名称，也就是php 返回给客户机的保存会话ID的cookie名称。
session文件的名称是以sess_为前缀的，以session_id为结尾命名。
session_unset(); //释放当前在内存中已经创建的所有$_SESSION变量，但不删除session文件以及不释放对应的session id


Session-php.ini:
session.save_handler=file  //session的存储方式
session.save_path="/tmp"  (httpd等web守护进程的宿主)写权限。
session.use_cookies：默认的值是“1”，代表SessionID使用Cookie来传递，反之就是使用Query_String来传递；
session.name：这个就是SessionID储存的变量名称，可能是Cookie，也可能是Query_String来传递，默认值是“PHPSESSID”；
session.cookie_lifetime：这个代表SessionID在客户端Cookie储存的时间，默认是0，代表浏览器一关闭SessionID就作废……就是这个所以Session不能永久使用！
session.gc_maxlifetime：这个是Session数据在服务器端储存的时间，如果超过这个时间，那么Session数据就自动删除！

Session-回收机制:
Garbage Collector 简称GC)
由于PHP的工作机制，它并没有一个daemon线程来定期的扫描Session 信息并判断其是否失效，当一个有效的请求发生时，PHP 会根据全局变量 session.gc_probability 和session.gc_divisor的值，来决定是否启用一个GC, 在默认情况下， session.gc_probability=1, session.gc_divisor =100 也就是说有1%的可能性启动GC(也就是说100个请求中只有一个gc会伴随100个中的某个请求而启动).
GC 的工作就是扫描所有的Session信息，用当前时间减去session最后修改的时间，同session.gc_maxlifetime参数进行比较，如果生存时间超过gc_maxlifetime(默认24分钟) ,就将该session文件删除。


Mysql常用命令行操作:

一、连接MYSQL。
格式： mysql -h主机地址 -u用户名 －p用户密码
如果是连接本地的mysql
首先在打开DOS窗口，然后进入目录 mysqlbin，再键入命令mysql -uroot -p，回车后提示你输密码，如果刚安装好MYSQL，超级用户root是没有密码的，故直接回车即可进入到MYSQL中了，MYSQL的提示符是：mysql＞

退出MYSQL命令： exit （回车）

二、显示命令
1、显示数据库列表。
show databases;
2、显示库中的数据表：
use mysql； ／／打开库，学过FOXBASE的一定不会陌生吧
show tables;
3、显示数据表的结构：
describe 表名;
4、建库：
create database 库名;
5、建表：
use 库名；
create table 表名 (字段设定列表)；
6、删库和删表:
drop database 库名;
drop table 表名；
7、将表中记录清空：
delete from 表名;
8、显示表中的记录：
select * from 表名;

三、一个建库和建表以及插入数据的实例
drop database if exists school; //如果存在SCHOOL库则删除
create database school; //建立库SCHOOL
use school; //打开库SCHOOL
create table teacher //建立表TEACHER
(
id int(3) auto_increment not null primary key,
name char(10) not null,
address varchar(50) default '北京',
year date
); //建表结束
//以下为插入字段
insert into teacher values('','glchengang','北京二中','1976-10-10');
insert into teacher values('','jack','北京二中','1975-12-23');

注：在建表中

（1）将ID设为长度为3的数字字段:int(3)并让它每个记录自动加一:auto_increment并不能为空:not null而且让他成为主字段primary key

（2）将NAME设为长度为10的字符字段

（3）将ADDRESS设为长度50的字符字段，而且缺省值为深圳。varchar和char有什么区别呢，只有等以后的文章再说了。

（4）将YEAR设为日期字段。
如果你在mysql提示符键入上面的命令也可以，但不方便调试。你可以将以上命令原样写入一个文本文件中假设为school.sql，然后复制到c:\下，并在DOS状态进入目录\mysql\bin，然后键入以下命令：
mysql -uroot -p密码 < c:\school.sql
如果成功，空出一行无任何显示；如有错误，会有提示。（以上命令已经调试，你只要将//的注释去掉即可使用）。



EPEL版本不同，根据不同系统请自行安装。