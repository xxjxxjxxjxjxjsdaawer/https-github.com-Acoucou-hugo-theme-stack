+++
author = "coucou"
title = "开发工具——Mysql"
date = "2023-08-01"
description = "开发工具专题之Mysql"
categories = [
    "开发工具"
]
tags = [
    "开发工具","Mysql"
]
+++

![](1.jpg)

## mysql 命令集

### 1. 进入mysql

```mysql
mysql -u root -p
```

### 2. 数据库查看、创建、使用

```mysql
show databases;  # 查看
create database [name];  #创建
use [name];  # 使用
```

### 3. 创建表、查看表

```mysql
creat table [name] (name VARCHAR(20), id VARCHAR(20));  # 需要初始化表项以及数据类型
show tables;  # 查看表

# var()与varchar()的区别在于var()是定常的,哪怕存储的字符串没有达到"()"中数字的上限,var()依然会占用空格来填充空间.而        	varchar()则是不定长的,没有达到"()"中的上限则会自动去掉后面的空格;、
# 数据类型可以用小写.不过最好用大写来表示区分关键字,
```

### 4. 表的增删改查

#### 4.1 增

```mysql
insert into [table] values([name], [id]);
```

#### 4.2 删

```mysql
delete from [table] where 条件(字段1=值1);
```

#### 4.3 改

```mysql
update [table] set 字段1=值1,字段2=值2 ... WHERE 条件;
```

#### 4.4 查

```mysql
select * from [table] where 条件;  # * 指所有，查找时也可以是某字段
select * from [table] order by [value] desc;  # 降序，升序则为 "asc"
select count([value]) from [table] where 条件; # 计数，"avg" 可取平均

# 查找还有许多方式这里列出部分
```

### 5. 约束条件

#### 5.1 主键约束

```mysql
create table user(id int PRIMARY KEY,name VARCHAR(20));  # 所有字段都不同，允许重复
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+

CREATE TABLE user2(id INT,name VARCHAR(20),password VARCHAR(20),PRIMARY key(id,name));  # 复合主键，只要所有的字段都不是相同的情况下可以允许其中的字段重复:
+----+------+----------+
| id | name | password |
+----+------+----------+
|  1 | 老李 | 123456   |
|  1 | 老王 | 123456   |
|  2 | 老王 | 123456   |
+----+------+----------+

# 场景:表中有班级号以及学生座位号,我们可以用班级号+学生的座位号可以准确的定位一个学生,如:(1班5号可以准确的确定一个学生)
```

#### 5.2 自增约束

```mysql
CREATE TABLE user3(id INT PRIMARY KEY AUTO_INCREMENT,name VARCHAR(20));  # id会自动递增，无需赋值
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
INSERT INTO user3(name) VALUES('张三');
INSERT INTO user3(name) VALUES('李四');
+----+------+
| id | name |
+----+------+
|  1 | 张三 |
|  2 | 李四 |
+----+------+
```

#### 5.3 唯一约束

```mysql
CREATE TABLE user5(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20));
运行 DESCRIBE user5;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+

# 新增name为唯一约束:
ALTER TABLE user5 ADD UNIQUE(name);
运行 DESCRIBE user5;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  | UNI | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
# 测试:插入数据
INSERT INTO user5 (name) VALUES ('cc');
运行 SELECT * FROM user5; 查看结果:
+----+------+
| id | name |
+----+------+
|  1 | cc   |
+----+------+
再次插入INSERT INTO user5(name) VALUES ('cc');
出现:ERROR 1062 (23000): Duplicate entry 'cc' for key 'name'

换个试试 INSERT INTO user5(name) VALUES ('aa');
运行 SELECT * FROM user5; 查看结果:
+----+------+
| id | name |
+----+------+
|  3 | aa   |
|  1 | cc   |
+----+------+
总结一下:
    主键约束(primary key)中包含了唯一约束
# 场景:业务需求:设计一张用户注册表,用户姓名必须要用手机号来注册,而且手机号和用户名称都不能为空,那么:
CREATE TABLE user_test(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键id',
    name VARCHAR(20) NOT NULL COMMENT '用户姓名,不能为空',
    phone_number VARCHAR(20) UNIQUE NOT NULL COMMENT '用户手机,不能重复且不能为空'
);
运行 DESCRIBE user_test;
+--------------+-------------+------+-----+---------+----------------+
| Field        | Type        | Null | Key | Default | Extra          |
+--------------+-------------+------+-----+---------+----------------+
| id           | int(11)     | NO   | PRI | NULL    | auto_increment |
| name         | varchar(20) | NO   |     | NULL    |                |
| phone_number | int(11)     | NO   | UNI | NULL    |                |
+--------------+-------------+------+-----+---------+----------------+
这样的话就达到了每一个手机号都只能出现一次,达到了每个手机号只能被注册一次.
用户姓名可以重复,但是手机号码却不能重复,复合正常的逻辑需求
```

#### 5.4 非空约束

```mysql
CREATE TABLE user6(name VARCHAR(20) NOT NULL COMMENT '用户姓名,不能为空');  # 非空即输入不能留空，除非设置了默认约束
```

#### 5.5 默认约束

```mysql
CREATE TABLE user6(
    id int PRIMARY KEY AUTO_INCREMENT COMMENT '主键id',
    name VARCHAR(20) NOT NULL COMMENT '用户姓名不能为空',
    phone_number VARCHAR(20) NOT NULL COMMENT '用户手机号,不能为空',
    status INT DEFAULT 0 COMMENT '用户状态0:启用 1:禁封 默认:0'
);
运行DESCRIBE user6;
+--------------+-------------+------+-----+---------+----------------+
| Field        | Type        | Null | Key | Default | Extra          |
+--------------+-------------+------+-----+---------+----------------+
| id           | int(11)     | NO   | PRI | NULL    | auto_increment |
| name         | varchar(20) | NO   |     | NULL    |                |
| phone_number | varchar(20) | NO   |     | NULL    |                |
| status       | int(11)     | YES  |     | 0       |                |
+--------------+-------------+------+-----+---------+----------------+
插入数据:
INSERT INTO user6(name,phone_number) VALUES ('aa','123');
INSERT INTO user6(name,phone_number) VALUES('bb','1234');
INSERT INTO user6(name,phone_number) VALUES('cc','1263456');

查看数据:SELECT * FROM user6;
+----+------+--------------+--------+
| id | name | phone_number | status |
+----+------+--------------+--------+
|  1 | aa   | 123          |      0 |
|  2 | bb   | 1234         |      0 |
|  3 | cc   | 1263456      |      0 |
+----+------+--------------+--------+
我们没有设置status的值,但是给我们创建了默认值 0.

应用场景:
业务需求:找正常的用户,对这些正常用户进行发放优惠卷或者积分之类的东西,而被禁封的用户我们不让其参加多动.
我们想要封用户只要将status的值从0改为1就行了,当然我们取用户的时候必须要先判断status是否是0.若是1.说明该用户已经被禁封.
先封手机号为'1234'的用户:
UPDATE user6 SET status = 1 WHERE phone_number= '1234';
SELECT * FROM user6;
+----+------+--------------+--------+
| id | name | phone_number | status |
+----+------+--------------+--------+
|  1 | aa   | 123          |      0 |
|  2 | bb   | 1234         |      1 |
|  3 | cc   | 1263456      |      0 |
+----+------+--------------+--------+
status为1,说明用户已经被封,该用户不可以参加活动

我们取用户的时候加上status的判断,如:
SELECT * FROM user6 WHERE status = 0;
+----+------+--------------+--------+
| id | name | phone_number | status |
+----+------+--------------+--------+
|  1 | aa   | 123          |      0 |
|  3 | cc   | 1263456      |      0 |
+----+------+--------------+--------+
```

#### 5.6 外键约束

```mysql
CREATE TABLE classes(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '班级表id',
    name VARCHAR(20) COMMENT '班级名称'
);
运行DESCRIBE classes;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+

CREATE TABLE student(
   id INT PRIMARY KEY AUTO_INCREMENT COMMENT '学生表id',
   name VARCHAR(20) COMMENT '学生姓名',
   class_id int COMMENT '教室id,这张表中的class_id是classes表中id的值',
   FOREIGN KEY (class_id) REFERENCES classes(id)
);
//FOREIGN :外来  REFERENCES:应用,参考
运行DESCRIBE student;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(20) | YES  |     | NULL    |                |
| class_id | int(11)     | YES  | MUL | NULL    |                |
+----------+-------------+------+-----+---------+----------------+

班级插入数据:
INSERT INTO CLASSES (name) VALUES ('一班');
INSERT INTO CLASSES (name) VALUES ('二班');
INSERT INTO CLASSES (name) VALUES ('三班');
INSERT INTO CLASSES (name) VALUES ('四班');
查看数据 SELECT * FROM classes;
+----+------+
| id | name |
+----+------+
|  1 | 一班 |
|  2 | 二班 |
|  3 | 三班 |
|  4 | 四班 |
+----+------+

学生插入数据:
INSERT INTO student (name,class_id) VALUES ('小赵',1);
INSERT INTO student (name,class_id) VALUES ('小钱',2);
INSERT INTO student (name,class_id) VALUES ('小孙',3);
INSERT INTO student (name,class_id) VALUES ('小李',4);
查看数据 SELECT * FROM student;
+----+------+----------+
| id | name | class_id |
+----+------+----------+
|  1 | 小赵 |        1 |
|  2 | 小钱 |        2 |
|  3 | 小孙 |        3 |
|  4 | 小李 |        4 |
+----+------+----------+
若是像插入班级为5的数据 如:
INSERT INTO student (name,class_id) VALUES ('小周',5);
报错: Cannot add or update a child row

我们删除正在被学生表引用的'四班'试试:
DELETE classes WHERE name = '四班';
出现:Cannot delete or update a parent row:不能删除主表中的行

我们先删除学生表中的 '小李'从而解除班级中'四班'的外键约束,再来删除'四班'(因为小李引用了四班)
DELETE FROM student WHERE name = '小李';
再次删除classes表中的'四班';
DELETE FROM classes WHERE name = '四班';
最后: SELECT * FROM classes;
+----+------+
| id | name |
+----+------+
|  1 | 一班 |
|  2 | 二班 |
|  3 | 三班 |
+----+------+
'四班'被成功删除!

总结:
1.主表中没有的数据,在附表中,是不可以使用的.
2.主表中记录的数据现在正在被附表所引用,那么主表中正在被引用的数据不可以被删除
3.若要想删除,先将附表中的数据删除在删除主表数据
4.对于外键约束大家可以联想 省,市 来进行联想 (市必须要依赖于省,只要省还有一个市在引用,那么就不可以删除省,要不然市就没有省了. 那么我们想删除省,必须要将该省下所有的市全部删除之后,才可以删除这个省)
```

### 6. 约束条件的增删改

```mysql
CREATE TABLE user4(
    id INT,
    name VARCHAR(20)
);
运行DESCRIBE user4;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

#### 6.1 加入主键约束

```mysql
ALTER TABLE user4 add PRIMARY KEY(id);
再次运行DESCRIBE user4;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

#### 6.2 删除主键约束

```mysql
ALERT TABLE user4 DROP PRIMARY KEY;
运行DESCRIBE user4查看表结构:
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

#### 6.3 修改约束

```mysql
ALTER TABLE user4 MODIFY id INT PRIMARY KEY;  # 使用modify 修改字段,添加约束
使用DESCRIBE user4 查看表结构:
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

### 7. 事务

