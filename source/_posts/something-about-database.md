---
title: 木犀后端分享——关系型数据库
comments: true
date: 2017-10-30 15:53:25
updated: 2017-11-16 15:53:25
tags: [database,python,sqlalchemy,flask-sqlalchemy]
categories: Database
permalink:
---
# 基本概念
`数据库(DataBase, DB）`：是存放数据的仓库，只不过这些数据存在一定的关联，并按一定的格式存放在计算机上。

`数据库管理系统(DataBase Management System, DBMS)`:是管理数据库的系统，它按一定的数据模型组织数据。DBMS应提供如下功能：
（1）数据定义功能可定义数据库中的数据对象。
（2）数据操纵功能可对数据库表进行基本操作，如插入、删除、修改、查询。
（3）数据的完整性检查功能保证用户输入的数据应满足相应的约束条件。
（4）数据库的安全保护功能保证只有赋予权限的用户才能访问数据库中的数据。
（5）数据库的并发控制功能使多个应用程序可在同一时刻并发地访问数据库的数据。
（6）数据库系统的故障恢复功能使数据库运行出现故障时进行数据库恢复，以保证数据库可靠运行。
（7）在网络环境下访问数据库的功能。
（8）方便、有效地存取数据库信息的接口和工具。编程人员通过程序开发工具与数据库的接口编写数据库应用程序。`数据库管理员（DataBase　Adminitrator，DBA）`通过提供的工具对数据库进行管理。

`数据库系统(Database System,DBS)`:数据、数据库、数据库管理系统与操作数据库的应用程序，加上支撑它们的硬件平台、软件平台和与数据库有关的人员一起构成了一个完整的数据库系统.

![数据库系统](/images/db.png)

从上面的这些概念中，我们可以知道平时我们讲的SQL Server、Oracle、MySQL、DB2、SyBase等，本身并不是数据库，他们是数据库管理系统。我们通过这个数据库管理系统创建的那个可以存储表的东西才是数据库。数据库中存储这一个或多个表，表之间存在着特定的关系。一个表由行和列组成，每一列代表一个字段(Field),每一行则代表着一条具有某种特别意义的记录。

![表的结构](/images/db2.jpg)

作为程序员，我们所要知道的就是要怎样使用数据库管理系统？怎样通过数据库管理系统去创建和删除数据库？怎样在数据库中创建和删除表，怎样在表中实现数据的增删改查？下面我就以mysql这一个关系型数据库管理系统为例来讲解这些操作。

# mysql客户端工具基本操作
## 连接并登录
``` bash
$ mysql -u <NAME> -h <HOSTNAME> -P <PORT> -p
$ mysql -u <NAME> -p
#不指明主机和端口，则默认访问本地的3306端口
```
在终端中输入上述命令，接着输入密码即可连接并登录指定主机指定端口的mysql服务器。(注意密码不回显)

## 编码的修改

在Mac或Linux上，需要编辑MySQL的配置文件，把数据库默认的编码全部改为UTF-8。MySQL的配置文件默认存放在/etc/my.cnf或者/etc/mysql/my.cnf：
```
[client]
default-character-set = utf8
[mysqld]
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
```
`重启MySQL`后，可以通过MySQL的客户端命令行检查编码：
``` bash
$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor...
...
mysql> show variables like '%char%';
+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_set_client     | utf8                                                   |
| character_set_connection | utf8                                                   |
| character_set_database   | utf8                                                   |
| character_set_filesystem | binary                                                 |
| character_set_results    | utf8                                                   |
| character_set_server     | utf8                                                   |
| character_set_system     | utf8                                                   |
| character_sets_dir       | /usr/local/mysql-5.1.65-osx10.6-x86_64/share/charsets/ |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.00 sec)
```
看到utf8字样就表示编码设置正确。

## 基本操作

### 查看当前所有的数据库：

``` sql
$ show databases;
```

### 切换数据库：

``` sql
$ use <DB NAME>;
```

### 创建数据库：
``` sql
$ CREATE DATABASE 数据库名;
```

### 建立数据表：
``` sql
$ USE 库名;
$ CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));
```
### 删除数据库：
``` sql
$ DROP DATABASE 库名;
```
### 删除数据表：
``` sql
$ DROP TABLE 表名;
```

### 查看当前数据库的所有的表：
``` sql
$ show tables;
```
### 查看某一个表的结构：
``` sql
$ desc <Table name>;
```
### 查看某一张表的前10条数据：
``` sql
$ select * from <Table name> limit 10;
```
基本上上面的几条命令会是大家以后在mysql客户端中使用的最多的命令了。

### 创建和管理用户

mysql支持数据库的多用户管理，并且可以对每个用户的权限进行管理。下面我们来看看怎样创建用户和管理用户的权限。

#### 创建用户

命令：
``` sql
$ CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```
说明：

- username：你将创建的用户名
- host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符%
- password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器

例子：
``` sql
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'pig'@'192.168.1.101_' IDENDIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '';
CREATE USER 'pig'@'%';
```
#### 授权

命令：

``` sql
$ GRANT privileges ON databasename.tablename TO 'username'@'host'

GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER  ON 数据库名.* TO 用户名@localhost IDENTIFIED BY '密码';
```
说明：
- privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
- databasename：数据库名
- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*
例子：
``` sql
$ GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
$ GRANT ALL ON *.* TO 'pig'@'%';
```
注意：
用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

``` sql
$ GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```
#### 设置和更改用户密码

命令：
``` sql
$ SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
如果是当前登陆用户用:
$ SET PASSWORD = PASSWORD("newpassword");
```
例子：
``` sql
$ SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
```
#### 插销用户权限

命令:
``` sql
$ REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```
说明:
privilege, databasename, tablename：同授权部分

例子:

``` sql
$ REVOKE SELECT ON *.* FROM 'pig'@'%';
```
注意：
假如你在给用户'pig'@'%'授权的时候是这样的（或类似的）：GRANT SELECT ON test.user TO 'pig'@'%'，则在使用REVOKE SELECT ON *.* FROM 'pig'@'%';命令并不能撤销该用户对test数据库中user表的SELECT 操作。相反，如果授权使用的是GRANT SELECT ON *.* TO 'pig'@'%';则REVOKE SELECT ON test.user FROM 'pig'@'%';命令也不能撤销该用户对test数据库中user表的Select权限。

#### 删除用户

``` sql
$ DROP USER 'username'@'host';
```
# SQL
SQL 是一门 ANSI 的标准计算机语言，用来访问和操作数据库系统。SQL 语句用于取回和更新数据库中的数据。SQL 可与数据库程序协同工作。虽然是一门标准的语言，不幸地是，存在着很多不同版本的 SQL 语言，但是为了与 ANSI 标准相兼容，它们必须以相似的方式共同地来支持一些主要的关键词（比如 SELECT、UPDATE、DELETE、INSERT、WHERE 等等）。

## select

语法：SELECT 列名称 FROM 表名称;
SELECT 语句用于从表中选取数据。结果被存储在一个结果表中（称为结果集）。

``` sql
SELECT LastName,FirstName FROM Persons;
SELECT * FROM Persons;
```
## distinct

语法：SELECT DISTINCT 列名称 FROM 表名称;
在表中，可能会包含重复值。这并不成问题，不过，有时您也许希望仅仅列出不同（distinct）的值。
关键词 DISTINCT 用于返回唯一不同的值

``` sql
SELECT DISTINCT LastName FROM Persons;
```

## where

语法：SELECT 列名称 FROM 表名称 WHERE 列 运算符 值;
如需有条件地从表中选取数据，可将 WHERE 子句添加到 SELECT 语句。
下面的运算符可以在where子句中使用：

| 操作符 |	描述 |
| :-------: | :--------: |
| =	 | 等于 |
| <> |	不等于 |
| >	 | 大于 |
| <	 | 小于 |
| >= |	大于等于 |
| <= |	 小于等于 |
| BETWEEN |	在某个范围内 |
| LIKE |	搜索某种模式 |

```sql
SELECT * FROM Persons WHERE City='Beijing';
请注意，我们在例子中的条件值周围使用的是单引号。
SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值，请不要使用引号。
这是正确的：
SELECT * FROM Persons WHERE FirstName='Bush';
这是错误的：
SELECT * FROM Persons WHERE FirstName=Bush;
这是正确的：
SELECT * FROM Persons WHERE Year>1965;
这是错误的：
SELECT * FROM Persons WHERE Year>'1965';
```
## and & or

语法：AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

``` sql
SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter';
SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter';
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William') AND LastName='Carter';
```
## order by

语法：ORDER BY 语句用于根据指定的列对结果集进行排序。ORDER BY 语句默认按照升序对记录进行排序。如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。
``` sql
以字母顺序显示公司名称：
SELECT Company, OrderNumber FROM Orders ORDER BY Company;
以字母顺序显示公司名称（Company），并以数字顺序显示顺序号（OrderNumber）：
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber;
以逆字母顺序显示公司名称：
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC;
以逆字母顺序显示公司名称，并以数字顺序显示顺序号：
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```
## insert into

语法：INSERT INTO 表名称 VALUES (值1, 值2,....);
我们也可以指定所要插入数据的列：INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....);

``` sql
INSERT INTO Persons VALUES ('Gates', 'Bill', 'Xuanwumen 10', 'Beijing');
INSERT INTO Persons (LastName, Address) VALUES ('Wilson', 'Champs-Elysees');
```
## update

语法：UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值;
Update 语句用于修改表中的数据。

``` sql
UPDATE Person SET FirstName = 'Fred' WHERE LastName = 'Wilson' ;
UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing' WHERE LastName = 'Wilson';
```
## delete

DELETE 语句用于删除表中的行。
语法：DELETE FROM 表名称 WHERE 列名称 = 值;
``` sql
DELETE FROM Person WHERE LastName = 'Wilson' ;
可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的：
DELETE FROM table_name
或者：
DELETE * FROM table_name
```
## 原生SQL在python代码中的使用
要想在你的python代码中操作mysql，那么据需要使用python链接mysql的驱动程序。MysqlDB是最常用的mysql驱动，但是至今没有支持python3.下表列出了python3中链接mysql的几个驱动程序：

| 名称 |	安装	 | 导入 |
| :---------: | :-----------: | :----------: |
| [MYSQL Connector](https://dev.mysql.com/doc/connector-python/en/) |	pip install mysql-connector-python |	mysql.connctor |
| [PYMYSQL](https://github.com/PyMySQL/PyMySQL) |	pip install pymysql |	pymysql |

下面以MySQL Connector为例讲解：
``` python
import mysql.connector
# conn = mysql.connector.connect(user='root', password='password', database='mydb')
conn = mysql.connector.connect(user='root', password='password',
                               database='mydb',host="120.220.235.23",port=32777)
cursor = conn.cursor()
# 创建user表:
cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
# 插入一行记录，注意MySQL的占位符是%s:
cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Michael'])
print(cursor.rowcount)
# 输出　1
# 提交事务:
conn.commit()
cursor.close()
# 运行查询:
cursor = conn.cursor()
cursor.execute('select * from user where id = %s', ('1',))
values = cursor.fetchall()
# 输出：[('1', 'Michael')]
# 关闭Cursor和Connection:
cursor.close()
conn.close()
```

# ORM
SQLAlchemy是Python编程语言下的一款ORM框架，该框架建立在数据库API之上，使用关系对象映射进行数据库操作，简言之便是：将对象转换成SQL，然后使用数据库API执行SQL并获取执行结果。这是分了三层，上层和下面两层。ORM是用类来封装的，SQLALCHEMY Core 是用函数封装的，是核心层，不是执行数据库语句，是把写成的类，翻译成sql语句。DBAPI执行SQL从而获取执行结果或将数据持久化进磁盘。

![ORM](/images/orm.jpeg)

SQLAlchemy本身无法操作数据库，其必须依赖pymysql，mysql-connector等第三方插件.Dialect用于和数据库API进行交流，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作。SQLAlchemy为我们屏蔽了SQL语句的繁琐，统一了各种不同数据库的操作，极大的方便了数据库应用的开发。

## SQLAlchemy的基本使用

### 连接配置

`dialect+driver://user:password@host:port/dbname`
不同数据库和驱动条件下的连接配置：
``` python
MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>
pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]
MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>
cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]
```

### 创建表

``` python
import sqlalchemy
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String
# 创建实例，并连接test库，echo=True 显示SQL语句
engine = create_engine("mysql+pymysql://root:123456@localhost/test",
                                    encoding='utf-8', echo=True)
#create_engine方法返回一个Engine实例，Engine实例只有直到触发数据库事件时才真正去连接数据库。#echo=True是回显命令，sqlalchemy与数据库通信的命令都将打印出来
Base = declarative_base()  # 生成orm基类
#declarative_base类维持了一个从类到表的关系，通常一个应用使用一个base实例，
#所有实体类都应该继承此类对象
class User(Base):
    __tablename__ = 'users'  # 表名
    __table_args__ = {"mysql_charset": "utf8"}
    id = Column(Integer, primary_key=True,autoincrement=True)#主键，自增id
    name = Column(String(32))
    password = Column(String(64))
Base.metadata.create_all(engine) #创建表结构 （这里是父类调子类）
#Base.metadata返回sqlalchemy.schema.MetaData对象，它是所有Table对象的集合，
#调用create_all()该对象会触发CREATE TABLE语句，如果数据库还不存在这些表的话。
```
除了上面的创建方法之外，还有一种创建方式：
``` python
from sqlalchemy import Table, MetaData, Column, Integer, String
from sqlalchemy.orm import mapper
metadata = MetaData()
user = Table('user', metadata,
            Column('id', Integer, primary_key=True,autoincrement=True),
            Column('name', String(50)),
            Column('fullname', String(50)),
            Column('password', String(12))
        )
class User(object):
    def __init__(self, name, fullname, password):
        self.name = name
        self.fullname = fullname
        self.password = password
mapper(User, user)  # 类User 和 user关联起来
# the table metadata is created separately with the Table construct, 
# then associated with the User class via the mapper() function
# 如果数据库里有，就不会创建了。
```
在使用的时候建议使用第一种创建方式。

### Column构造函数常用参数

| 参数 |	含义 |	可选值	| 默认值 |
| :----------: | :----------: | :----------: | :------------: |
| autoincrement |	是否自增 |	Ture,False |	False |
| primary_key |	主键 |	Ture,False |	False |
| default |	指定默认值 |	自定义值	| 无 |
| index |	是否建索引	 | True,False |	False |
| nullable |	字段可否可空 |	Ture,False |	False |

### 插入一条数据
``` python
from sqlalchemy import create_engine
from sqlalchemy import Table, MetaData, Column, Integer, String
from sqlalchemy.orm import mapper, sessionmaker
# 创建实例，并连接test库
engine = create_engine("mysql+pymysql://root:123456@localhost/test",
                                    encoding='utf-8', echo=True)
metadata = MetaData()
user = Table('user', metadata,
            Column('id', Integer, primary_key=True,autoincrement=True),
            Column('name', String(50)),
            Column('password', String(12))
        )
class User(object):
    def __init__(self, name,  password):
        self.name = name
        self.password = password
# the table metadata is created separately with the Table construct, then associated with the User class via the mapper() function
mapper(User, user)
# 创建与数据库的会话session class ,注意,这里返回给session的是个class,不是实例.
Session_class = sessionmaker(bind=engine)  # 实例和engine绑定
Session = Session_class()  # 生成session实例，相当于游标
#Session是真正与数据库通信的handler，你还可以把他理解一个容器，add就是往容器中添加对象
user_obj = User(name="fgf",password="123456")  # 生成你要创建的数据对象
print(user_obj.name,user_obj.id)  # 此时还没创建对象呢，不信你打印一下id发现还是None
Session.add(user_obj)  # 把要创建的数据对象添加到这个session里， 一会统一创建
# Session.add_all([user1,user2,use3]) #add_all()添加多个对象，效率高
print(user_obj.name,user_obj.id) #此时也依然还没创建，id仍然为None
Session.commit() #现此才统一提交，创建数据
```
### 对象状态

对象实例有四种状态，分别是：

- Transient（瞬时的)：这个状态的对象还不在session中，也不会保存到数据库中，主键为None（不是绝对的，如果Persistent对象rollback后虽然主键id有值，但还是Transient状态的）。
- Pending（挂起的）：调用session.add()后，Transient对象就会变成Pending，这个时候它还是不会保存到数据库中，只有等到触发了flush动作才会存在数据库，比如query操作就可以出发flush。同样这个时候的实例的主键一样为None
- Persistent（持久的）：session中，数据库中都有对应的一条记录存在，主键有值了。
- Detached（游离的）：数据库中有记录，但是session中不存在，对这个状态的对象进行操作时，不会触发任何SQL语句。

### Session对象缓存清理

Session对象在commit()方法调用时,在执行查询时和显式调用session的flush方法时清理缓存，从而保证查询结果能反映对象的最新状态。

Session是真正与数据库通信的handler，你还可以把他理解一个容器，add就是往容器中添加对象
执行完add方法后，ed_user对象处于pending状态，不会触发INSERT语句，当然ed_uesr.id也为None，如果在add方后有查询(session.query)，那么会flush一下，把数据刷一遍，把所有的pending信息先flush再执行query。

### 查询

``` python
my_user = Session.query(User).filter_by(name="fgf").first()  # 查询
print(my_user) #输出 <__main__.User object at 0x7f0a5a3dea20>
print(my_user.id,my_user.name,my_user.password)
# 输出 1 fgf 123456
```
不过刚才显示的内存对象对址没办法分清返回的是什么数据的，除非打印具体字段看一下，如果想让它变的可读，只需在定义表的类下面加上这样的代码:
``` python
def __repr__(self):
    return "<User(name='%s',  password='%s')>" % ( self.name, self.password)
```
完整的查询代码：
``` python
from sqlalchemy import create_engine
from sqlalchemy import Table, MetaData, Column, Integer, String
from sqlalchemy.orm import mapper, sessionmaker
# 创建实例，并连接test库
engine = create_engine("mysql+pymysql://root:123456@localhost/test",
                                    encoding='utf-8', echo=True)
metadata = MetaData()
user = Table('user', metadata,
            Column('id', Integer, primary_key=True,autoincrement=True),
            Column('name', String(50)),
            Column('password', String(12))
        )
class User(object):
    def __init__(self, name, id, password):
        self.id = id
        self.name = name
        self.password = password
    def __repr__(self):
        return "<User(name='%s',  password='%s')>" % (self.name, self.password)
mapper(User, user)
# 创建与数据库的会话session class ,注意,这里返回给session的是个class,不是实例
Session_class = sessionmaker(bind=engine)
Session = Session_class()  # 生成session实例
my_user = Session.query(User).filter_by(name="fgf").first()  # 查询第一个
# my_user = Session.query(User).filter_by().all()  # 查询所有
print(my_user)
# print(my_user.id,my_user.name,my_user.password)
# Session.commit()  #查询不需要commit
```
### 多条件查询

filter_by与filter

``` python
my_user1 = Session.query(User).filter(User.id>2).all()
my_user2 = Session.query(User).filter_by(id=27).all()  # filter_by相等用‘=’
my_user3 = Session.query(User).filter(User.id==27).all()  # filter相等用‘==’
objs = Session.query(User).filter(User.id>0).filter(User.id<7).all()
print(my_user1,'\n',my_user2,'\n',my_user3,'\n',objs)
```
建立在SQLAlchemy上的几种常见的SQL查询实例：

``` python
几种常见sqlalchemy查询：
#简单查询    
print(session.query(User).all())
print(session.query(User.name, User.fullname).all())    
print(session.query(User, User.name).all())        
#带条件查询    
print(session.query(User).filter_by(name='user1').all())    
print(session.query(User).filter(User.name == "user").all())    
print(session.query(User).filter(User.name.like("user%")).all())      
#多条件查询  
from sqlalchemy import and_,or_  
print(session.query(User).filter(and_(User.name.like("user%"), User.fullname.like("first%"))).all())    
print(session.query(User).filter(or_(User.name.like("user%"), User.password != None)).all())        
#sql过滤    
print(session.query(User).filter("id>:id").params(id=1).all())        
#关联查询     
print(session.query(User, Address).filter(User.id == Address.user_id).all())    
print(session.query(User).join(User.addresses).all())    
print(session.query(User).outerjoin(User.addresses).all())        
#聚合查询    
print(session.query(User.name, func.count('*').label("user_count")).group_by(User.name).all())    
print(session.query(User.name, func.sum(User.id).label("user_id_sum")).group_by(User.name).all())        
#子查询    
stmt = session.query(Address.user_id, func.count('*').label("address_count")).group_by(Address.user_id).subquery()    
print(session.query(User, stmt.c.address_count).outerjoin((stmt, User.id == stmt.c.user_id)).order_by(User.id).all())     
   
#exists    
print(session.query(User).filter(exists().where(Address.user_id == User.id)))    
print(session.query(User).filter(User.addresses.any()))
限制返回字段查询
person = session.query(Person.name, Person.created_at,Person.updated_at).filter_by(name="zhongwei").order_by(Person.created_at).first()
记录总数查询：
from sqlalchemy import func
# count User records, without
# using a subquery.
session.query(func.count(User.id))
# return count of user "id" grouped
# by "name"
session.query(func.count(User.id)).\
        group_by(User.name)
from sqlalchemy import distinct
# count distinct "name" values
session.query(func.count(distinct(User.name)))
```
### 修改

``` python
my_user = Session.query(User).filter_by(name="fgf").first()
my_user.name = "fenggf"  # 查询出来之后直接赋值修改
my_user.passwork = "123qwe"
Session.commit() #注意提交
```

### 删除

``` python
query = Session.query(User).filter_by(name="hhh").delete()
Session.commit()
```

### 回滚

``` python
my_user = Session.query(User).filter_by(id=1).first()
my_user.name = "Jack"
fake_user = User(name='Rain', password='12345')
Session.add(fake_user)
print(Session.query(User).filter(User.name.in_(['Jack','rain'])).all() )  #这时看session里有你刚添加和修改的数据
Session.rollback() #此时你rollback一下
print(Session.query(User).filter(User.name.in_(['Jack','rain'])).all() ) #再查就发现刚才添加的数据没有了。
# Session
# Session.commit()
```
### 表与表之间的关系

SQLAlchemy中的映射关系有四种,分别是一对一,一对多,多对一,多对多。

### 一对多和多对一

因为外键(ForeignKey)始终定义在多的一方.如果relationship定义在多的一方,那就是多对一,一对多与多对一的区别在于其关联(relationship)的属性在多的一方还是一的一方，如果relationship定义在一的一方那就是一对多.

``` python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Enum,DATE,Integer, String,ForeignKey
from sqlalchemy.orm import sessionmaker,relationship
engine = create_engine("mysql+pymysql://root:123456@localhost/test",
                                    encoding='utf-8')
Base = declarative_base()  # 生成orm基类
class Stu2(Base):
    __tablename__ = "stu2"
    id = Column(Integer, primary_key=True,autoincrement=True)
    name = Column(String(32),nullable=False)
    register_date = Column(DATE,nullable=False)
    def __repr__(self):
        return "<%s name:%s>" % (self.id, self.name)
class StudyRecord(Base):
    __tablename__ = "study_record"
    id = Column(Integer, primary_key=True,autoincrement=True)
    day = Column(Integer,nullable=False)
    status = Column(String(32),nullable=False)
    stu_id = Column(Integer,ForeignKey("stu2.id"))  #------外键关联------
    #这个nb，允许你在user表里通过backref字段反向查出所有它在stu2表里的关联项数据
    stu2 = relationship("Stu2", backref="my_study_record")  # 添加关系，反查（在内存里）
    def __repr__(self):
        return "<%s day:%s status:%s>" % (self.stu2.name, self.day,self.status)
Base.metadata.create_all(engine)  # 创建表结构
Session_class = sessionmaker(bind=engine)  # 创建与数据库的会话session class ,注意,这里返回给session的是个class,不是实例
session = Session_class()  # 生成session实例 #cursor
s1 = Stu2(name="A",register_date="2014-05-21")
s2 = Stu2(name="J",register_date="2014-03-21")
s3 = Stu2(name="R",register_date="2014-02-21")
s4 = Stu2(name="E",register_date="2013-01-21")
study_obj1 = StudyRecord(day=1,status="YES", stu_id=1)
study_obj2 = StudyRecord(day=2,status="NO", stu_id=1)
study_obj3 = StudyRecord(day=3,status="YES", stu_id=1)
study_obj4 = StudyRecord(day=1,status="YES", stu_id=2)
session.add_all([s1,s2,s3,s4,study_obj1,study_obj2,study_obj3,study_obj4])  # 创建
session.commit()
stu_obj = session.query(Stu2).filter(Stu2.name=="a").first()  # 查询
# 在stu2表，查到StudyRecord表的记录
print(stu_obj.my_study_record)  # 查询A一共上了几节课
```
### 多对多关系

多对多关系需要一个中间关联表,通过参数secondary来指定,这个中间关联表只需要创建即可，不需要操作它，SQLAlchemy自己会管理这张表。
![多对多](/images/m2m.jpg)
``` python
post_keywords = Table('student_class',Base.metadata,
        Column('student_id',Integer,ForeignKey('students.id')),
        Column('class_id',Integer,ForeignKey('classes.id'))
)
class Student(Base):
    __tablename__ = 'students'
    id = Column(Integer,primary_key=True)
    name = Column(String(20))
    age = Column(Integer)
    keywords = relationship('Class',secondary=student_class,backref='students')
    # keywords = relationship('Class',secondary=student_class,backref=backref('students,lazy="dynamic"),lazy="dynamic")
class Class(Base):
    __tablename__ = 'classes'
    id = Column(Integer,primary_key = True)
    classname = Column(String(50),nullable=False,unique=True)
```

``` python
from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy import create_engine
Base = declarative_base()
class Customer(Base):
    __tablename__ = 'customer'
    id = Column(Integer, primary_key=True,autoincrement=True)
    name = Column(String(64))
    # 账单地址和邮寄地址, 都关联同一个地址表
    billing_address_id = Column(Integer, ForeignKey("address.id"))
    shipping_address_id = Column(Integer, ForeignKey("address.id"))
    billing_address = relationship("Address", foreign_keys=[billing_address_id])
    shipping_address = relationship("Address", foreign_keys=[shipping_address_id])
class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True,autoincrement=True)
    street = Column(String(64))
    city = Column(String(64))
    state = Column(String(64))
    def __repr__(self):
        return self.street
engine = create_engine("mysql+pymysql://root:123456@localhost/test",
                                    encoding='utf-8')
Base.metadata.create_all(engine)  # 创建表结构
```
## Flask-SQLAlchemy的使用

Flask-SQLAlchemy 在 SQLAlchemy 的基础上，提供了一些常用的工具，并预设了一些默认值，帮助你更轻松地完成常见任务。flask-sqlalchemy 用起来比直接用 sqlalchemy 方便、省事，不过有些高级一点的功能如果不了解 sqlalchemy 的话会用不好。

### 安装

``` bash
$ pip install flask-sqlalchemy
```
### 配置数据库

``` python
from flask_sqlalchemy import SQLAlchemy
from flask import Flask
app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"]="mysql+pymysql://root:pqc19960320@120.77.220.239:32777/mydb"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"]=True
db=SQLAlchemy(app)
```
db对象是SQLAlchemy类的实例，表示程序使用的数据库，同时还获得了Flask-SQLAlchemy提供的所有功能。

### 定义模型
``` python
class Role(db.Model):
    __tablename__="roles"
    id=db.Column(db.Integer,primary_key=True)
    name=db.Column(db.String(30),unique=True)
    def __repr__(self):
        return "<Role %s>" % self.name
class User(db.Model):
    __tablename__="users"
    id=db.Column(db.Integer,primary_key=True)
    username=db.Column(db.String(64),unique=True,index=True)
    def __repr__(self):
        return "<User %s>" % self.username
```
__tablename__定义在数据库中使用的表名，如果没有定义__tablename__,Flask-SQLAlchemy会使用一个默认的名字，但是默认的表名没有遵循复数形式进行命名的约定。

常使用的列类型

| 类型名称 |	python类型 |	描述 |
| :--------: | :----------: | :--------: |
| Integer |	int	 | 常规整形，通常为32位 |
| SmallInteger |	int	 | 短整形，通常为16位 |
| BigInteger |	int或long |	精度不受限整形 |
| Float |	float |	浮点数 |
| Numeric |	decimal.Decimal |	定点数 |
| String |	str |	可变长度字符串 |
| Text |	str |	可变长度字符串，适合大量文本 |
| Unicode |	unicode |	可变长度Unicode字符串 |
| UnicodeText |	unicode |	可变长度Unicode字符串,对较长或不限长度的字符串做了优化 |
| Boolean |	bool	| 布尔型 |
| Date |	datetime.date |	日期类型 |
| Time |	datetime.time	|  时间类型 |
| DataTime |	datetime.datatime |	日期和时间 |
| Interval |	datetime.timedelta |	时间间隔 |
| Enum |	str |	字符列表 |
| PickleType |	任意Python对象	| 自动Pickle序列化 |
| LargeBinary |	str |	二进制 |

常使用的列选项

| 参数 |	含义 |	可选值 |	默认值 |
| :----------: | :------------: | :----------: | :----------------: |
| primary_key |	主键	| True,False |	False |
| default |	指定默认值 |	自定义值 |	无 |
| index	 | 是否建索引 |	True,False |	False |
| nullable |	字段可否可空	| True,False |	False |
| unique |	是否允许出现重复值 |	True,False |	False |

### 关系

#### 一对多和多对一
``` python
class Role(db.Model):
    __tablename__="roles"
    id=db.Column(db.Integer,primary_key=True)
    name=db.Column(db.String(30),unique=True)
    users=db.relationship("User",backref="role")
    def __repr__(self):
        return "<Role %s>" % self.name
class User(db.Model):
    __tablename__="users"
    id=db.Column(db.Integer,primary_key=True)
    username=db.Column(db.String(64),unique=True,index=True)
    role_id=(db.Integer,db.ForeignKey("roles.id"))
    def __repr__(self):
        return "<User %s>" % self.username
```
两张表之间的关系事实上是由ForeignKey建立起来的，relationship在两张表之间建立了一个虚拟关系，帮助我们更方便的操作这种关系。注意：在一对多或多对一的关系中，ForeignKey只会出现在多的一方，并且它所关联的一定是另一张表的primary_key。 relationship的一个必须参数是要建立虚拟关系的表所对应类的名字，如果这个类还没有定义，可以用一个字符串代替，它的其他更多可选参数见下图：

![关系选项](/images/rela.png)

[lazy选项](http://blog.csdn.net/bestallen/article/details/52551579)

#### 数据库操作

##### 创建
``` python
#创建表
db.create_all()
#删除表
db.drop_all()
```
##### 插入行
``` python
from models import Role,User
admin_role=Role(name="Admin")
mod_role=Role(name="Moderator")
user_role=Role(name="User")
john=User(usename="John",role=admin_role)
susan=User(username="Susan",role=mod_role)
david=User(username="David",role=user_role)
```
注意这里创建Role和User实例时，都没有传id属性，这是因为primary_key是由Flask-SQLAlchemy管理的。relationship建立的虚拟字段role和users也是可以使用的,虽然他们不是真正的数据库列，但却是一对多关系的高级表示。此时这些对象的状态是暂时的，并没有写入数据库中，此时打印对象的id值：
``` python
print(admin_role.id)   #->None
print(john.id)   #->None
```
添加如会话：
``` python
db.session.add(admin_role)
db.session.add(mod_role)
db.session.add(user_role)
db.session.add(john)
db.session.add(susan)
db.session.add(david)
#或者一次性添加
db.session.add_all([admin_role,mod_role,user_role,john,susan,davia])
#此时这些数据对象的状态变成挂起的了，依然没有写入到数据库中，我们打印对象的id，依然是None
print(admin_role.id) #->None
#直到我们commit()之后，这些数据对象才持久化到数据库中
db.commit()
print(admin_role.id) #->1
```
##### 修改行

``` python
admin_role.name="Administrator"
db.session.add(admin_role)
db.session.commit()
```
##### 删除行

``` python
db.session.delete(mod_role)
db.session.commit()
```
##### 查询行
``` python
Role.query.all()
[<Role "Administrator">,<Role "User">]
```
Flask-SQLAlchemy为每个模型都提供了一个query对象，模型名.query就可以拿到这个query对象，在query对象的后面可以加查询过滤器，查询过滤器返回的仍然是query对象，这意味着我们还可以继续在后面追加查询过滤器，从而精确的筛选出我们需要的数据。query对象建立好之后，Flask-SQLAlchemy并没有执行SQL查询，要想真正的得到数据，我们还必须要在query对象的后面执行查询执行函数。下面是常用的查询过滤器和查询执行函数：

| 过滤器	| 说明 |
| :------------: | :--------------: |
| filter() |	把过滤器添加到原查询上，返回一个新查询 |
| filter_by() |	把等值过滤器添加到原查询上，返回一个新查询 |
| limit() |	使用指定的值限制原查询返回的结果数量，返回一个新查询 |
| offset() |	偏移原查询返回的结果，返回一个新查询 |
| order_by() |	根据指定条件对原查询结果进行排序，返回一个新查询 |
| group_by() |	根据指定条件对原查询结果进行分组，返回一个新查询 |

| 查询执行方法 |	说明 |
| :----------: | :------------: |
| all() |	以列表形式返回查询的所有结果 |
| first() | 	返回查询的第一个结果，如果没有结果，则返回None |
| first_or_404() |	返回查询的第一个结果，如果没有结果，则终止请求，返回404错误响应 |
| get() |	返回指定主键对应的行，如果没有对应的行，则返回None |
| get_or_404 |	返回指定主键对应的行，如果没有找到指定的主键，则终止请求，返回404错误响应 |
| count() |	返回查询结果的数量 |
| paginate() |	返回一个Paginate对象，它包含指定范围内的结果 |
下面我们看看relationship建立的虚拟字段role,users
``` python
print(admin_role.users)
#输出：[<User John>]
```
这里有一个小问题，我们用users虚拟字段的时候，默认我们就得到了当前角色的所有用户的列表，也就是得到了查询执行函数执行之后的结果，但是如果我们想要做更加精细的筛选呢？

事实上，我们更期待它返回的是一个query对象，而不是结果列表。这是可以做得到的。

##### 多对多

![多对多](/images/m2m2.png)
``` python
registrations=db.Table("registrations",
    db.Column("student_id",db.Integer,db.ForeignKey("students.id")),
    db.Column("class_id",db.Integer,db.ForeignKey("classes.id"))
)
class Student(db.Model):
    id=db.Column(db.Integer,primary_key=True)
    name=db.Column(db.String(20))
    classes=db.relationship("Class",
                            secondary=registrations,
                            backref=db.backref("students",lazy="dynamic"),
                            lazy="dynamic")
class Class(db.Model):
    id=db.Column(db.Integer,primary_key=True)
    name=db.Column(db.String(20))
c=Class(name="语文")
#学生s注册了课程c
s.classes.append(c)
db.session.add(s)
s.classes.remove(c)
db.session.add(s)
#学生s注册的所有课程
s.classes.all()
#注册c的所有的学生
c.students.all()
```
##### 自引用
![自引用](/images/zyy.png)
``` python
class Follow(db.Model):
    __tablename__="follows"
    follower_id=db.Column(db.Integer,db.ForeignKey("users.id"),primary_key=True)
    followed_id=db.Column(db.Integer,db.ForeignKey("user.id"),primary_key=True)
    timestamp=db.Column(db.DateTime,default=datetime.utcnow)
class User(db.Model):
    __tablename__="users"
    username=db.Column(db.String(20),index=True)
    followed=db.relationship("Follow",
                            foreign_key=[Follow.follower_id],
                            backref=db.backref("follower",lazy="joined"),
                            lazy="dynamic",
                            cascade="all,delete-orphan")
     followers=db.relationship("Follow",
                            foreign_key=[Follow.followed_id],
                            backref=db.backref("followed",lazy="joined"),
                            lazy="dynamic",
                            cascade="all,delete-orphan")
```
# 参考
参考内容过多，在此不一一列出，默默感谢那些走在前面的大佬。

