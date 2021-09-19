### 1、连接方式
#### 1.1、命令行方式

```shell
mysql [-h 主机名 -P 端口号] -u 用户名 -p密码
```
注意：-p 和 密码之间不能有空格，其他参数可以有也可以没有
#### 1.2、客户端方式

* Navicat
* SQLyog
* MySQL Workbench
### 2、SQL语言分类
1. 数据定义语言(DDL: Data Definition Language)
2. 数据操作语言(DML: Data Manipulation Language)
3. 数据查询语言(DQL: Data Query Language)
4. 事务控制语言(TCL: Transaction Control Language)
### 3、MySQL数据类型
#### 3.1、数值型

* 整型

| 名称        | 字节数 |
| ----------- | ------ |
| tinyint     | 1      |
| smallint    | 2      |
| mediumint   | 3      |
| int/integer | 4      |
| bigint      | 8      |

​	特点： 

​    ①都可以设置无符号和有符号，默认有符号，通过unsigned设置无符号。
​    ②如果超出了范围，会报out or range异常，插入临界值。
​    ③长度可以不指定，默认会有一个长度。

**注意：长度代表显示的最大宽度，如果不够则左边用0填充，但需要搭配zerofill，并且默认变为无符号整型。**

* 小数型

| 名称         | 字节数 | 类型                 |
| ------------ | ------ | -------------------- |
| float(M,D)   | 4      | 浮点数               |
| double(M,D)  | 8      | 浮点数               |
| decimal(M,D) | M+2    | 定点数（精度比较高） |

​	特点：
​	①M代表整数部位+小数部位的个数，D代表小数部位
​	②如果超出范围，则报out or range异常，并且插入临界值
​	③M和D都可以省略，但对于定点数，M默认为10，D默认为0
​	④如果精度要求较高，则优先考虑使用定点数

#### 3.2、字符型

| 名称      | 描述                                   |
| --------- | -------------------------------------- |
| char      | 用来保存字符串，长度为0~255之间。      |
| varchar   | 用来保存字符串，长度为0~65535之间。    |
| binary    | 用来保存二进制字符串                   |
| varbinary | 用来保存二进制字符串                   |
| enum      | 枚举类型                               |
| set       | 和enum类型类似，里面可以保存0~64个成员 |
| text      | 用来保存字符串                         |
| blob      | 用来保存二进制数据                     |

注意：

* char：固定长度的字符，写法为char(M)，最大长度不能超过M，其中M可以省略，默认为1。
* varchar：可变长度的字符，写法为varchar(M)，最大长度不能超过M，其中M不可以省略。

#### 3.3、日期型

| 名称      | 描述      |
| --------- | --------- |
| year      | 年        |
| data      | 日期      |
| time      | 时间      |
| datatime  | 日期+时间 |
| timestamp | 日期+时间 |

注意：

* datetime 长度为8，范围为：1000-01-01 00:00:00 到 9999-12-31 23:59:59.
* timestamp 长度为4，比较容易受时区、语法模式、版本的影响，范围为：1970-01-01 00:00:00 到 2037-12-31 23:59:59.

### 4、MySQL约束

#### 4.1、主键约束（PRIMARY KEY）

一个表中至多有一个主键，标识为主键的列具有唯一性和不能为空的特性。

* 创建表时添加主键

```sql
create table 表名(
	字段名 字段类型 primary key,
    ......
)
```

* 修改表时添加主键

```sql
alter table 表名 add [constraint 约束名] primary key(字段名);
```

* 删除主键

```sql
alter table 表名 drop primary key;
```



#### 4.2、外键约束（FOREIGN KEY）

一个表中可以有多个外键。外键用于限制两个表的关系，从表的字段值引用了主表的某字段值。外键列和主表的被引用列要求类型一致，意义一样，名称无要求。主表的被引用列要求是一个key（一般就是主键）。

* 创建表时添加外键

```sql
create table 表名(
    ......
    constraint 约束名 foreign key(字段名) references 主表(被引用字段名)
)
```

* 修改表时添加外键

```sql
alter table 表名 add [constraint 约束名] foreign key(字段名) references 主表(被引用字段名);
```

* 删除外键

```sql
alter table 表名 drop foreign key 约束名;
```



#### 4.3、非空约束（NOT NULL）

表示该字段值不能为空。

* 创建表时添加非空约束

```sql
create table 表名(
	字段名 字段类型 not null,
    ......
)
```

* 修改表时添加非空约束

```sql
alter table 表名 modify column 字段名 字段类型 not null;
```

* 删除非空约束

```sql
alter table 表名 modify column 字段名 字段类型;
```



#### 4.4、唯一性约束（UNIQUE）

表示该字段的值不可以重复，但是可以为空。

* 创建表时添加唯一性约束

```sql
create table 表名(
	字段名 字段类型 unique,
    ......
)
```

* 修改表时添加唯一性约束

```sql
alter table 表名 add [constraint 约束名] unique(字段名);
```

* 删除唯一性约束

```sql
alter table 表名 drop index 约束名
```



#### 4.5、检查约束（CHECK）

mysql不支持检查约束。



#### 4.6、默认值（DEFAULT）

表示该字段的值不用手动插入有默认值。

* 创建表时添加默认值

```sql
create table 表名(
	字段名 字段类型 default 默认值,
    ......
)
```

* 修改表时添加默认值

```sql
alter table 表名 modify column 字段名 字段类型 default 默认值;
```

* 删除默认值

```sql
alter table 表名 modify column 字段名 字段类型;
```



#### 4.7、自增长列

不需要手动插值，可以自动序列值，默认从1开始，步长为1；只支持数值型；一个表只能有一个自增长列；自增长列必须为一个key。

* 查看全局自增长列的初始值和步长

```sql
show global variables like 'auto_incr%';
```

* 修改全局自增长列的初始值

```sql
set global auto_increment_offset=初始值;
```

* 修改全局自增长列的步长

```sql
set global auto_increment_increment=步长;
```

* 创建表时设置自增长列

```sql
create table 表名(
	字段名 字段类型 约束 auto_increment,
    ......
)
```

* 修改表时设置自增长列

```sql
alter table 表名 modify column 字段名 字段类型 约束 auto_increment;
```

* 删除自增长列

```sql
alter table 表名 modify column 字段名 字段类型 约束;
```



### 5、数据定义语言 DDL
#### 5.1、数据库

* 查看所有数据库

```sql
show databases;
```
* 创建数据库

```sql
create database [if not exists] 库名 [character set 字符集名];
```
* 使用数据库

```sql
use 库名;
```
* 修改数据库

```sql
alter database 库名 character set 字符集名;
```
* 删除数据库

```sql
drop database [if exists] 库名;
```
#### 5.2、表

* 查看当前库中所有表

```sql
show tables;
```
* 创建表

```sql
create table [if not exists] 表名(
	字段名 字段类型 约束,
	...
);
```
* 查看表结构

```sql
desc 表名;
```
* 修改表

```sql
1. 修改表名
alter table 表名 rename [to] 新表名;
2. 添加列
alter table 表名 add column 列名 类型 约束 [first|after 列名];
3. 修改列名
alter table 表名 change column 旧列名 新列名 类型 约束 [first|after 列名];
4. 修改列类型或约束
alter table 表名 modify column 列名 新类型 [新约束];
5. 删除列
alter table 表名 drop column 列名;
```
* 删除表

```sql
drop table [if exists] 表名;
```
* 复制表

```sql
1. 复制表结构
create table 新表名 like 旧表名;
2. 复制表结构+数据 (注意：可以指定列进行复制)
create table 新表名 select 查询列表 from 旧表名 [where 查询条件];
```


#### 5.3、视图

MySQL从5.0.1版本开始提供视图功能。它是一种虚拟的表，行和列的数据来自定义视图的查询中使用的表。当我们使用视图时，视图才会动态生成。也就是说数据库中不会开辟空间存储视图中的数据，只是存储的sql逻辑。使用视图可以简化复杂sql语句，提高了sql的重用性，保护了基表数据，提高了安全性。

* 创建视图

```sql
create view 视图名 as 查询语句;
```

* 修改视图

```sql
create or replace view 视图名 as 查询语句;
或
alter view 视图名 as 查询语句;
```

* 删除视图

```sql
drop view 视图1,视图2,...;
```

* 查看视图

```sql
desc 视图名;
或
show create view 视图名;
```



注意：视图支持insert、update、delete、select。

但是一般只用于查询，而不是更新，所以具备以下特点的视图都不允许更新：

* 包含分组函数、group by、distinct、having、union或者union all。
* join。
* 常量视图。
* select 中包含子查询。
* from 一个不能更新的视图。
* where子句的子查询引用了from子句中的表。



### 6、数据操作语言 DML

#### 6.1、插入

* 方式一

  **语法：**

  ```sql
  insert into 表名(字段名1,字段名2, ......) values(值1,值2,......);
  ```

  **特点：**

  * 要求值的类型和字段的类型要一致或兼容。
  * 字段的个数和顺序不一定与原始表中的字段个数和顺序一致但必须保证值和字段一一对应。
  * 假如表中有可以为null的字段，注意可以通过以下两种方式插入null值
    1. 字段和值都省略。
    2. 字段写上，值使用null。

  * 字段和值的个数必须一致。
  * 字段名可以省略，默认所有列。
  * 字符和日期型数据应包含在**单引号**中。
  * 支持一次插入多行，语法如下：

  ```sql
  insert into 表名[(字段名,...)] values(值,...),(值,...),...;
  ```

  * 支持子查询，字段名要对应。语法如下：

  ```sql
  insert into 表名[(字段名,...)] 查询语句;
  ```

  

* 方式二

  **语法：**

  ```sql
  insert into 表名 set 字段=值, 字段=值,......;
  ```



#### 6.2、修改

**语法：**

* 修改单表记录

```sql
update 表名 set 字段名=值,字段名=值,...[where 筛选条件];
```

* 修改多表记录

```sql
update 表1 别名 left|right|inner join 表2 别名 on 连接条件 set 字段名=值,字段名=值,...[where 筛选条件];
```



#### 6.3、删除

##### 6.3.1、删除方式

* 方式一

  **语法：**

  ```sql
  delete from 表名 [where 筛选条件] [limit 条目数];
  ```

* 方式二

  语法：

  ```sql
  truncate table 表名;
  ```



##### 6.3.2、区别

| 问题                               | delete               | truncate |
| ---------------------------------- | -------------------- | -------- |
| 删除后，再插入，标识从什么地方开始 | 从断点处开始         | 从1开始  |
| 是否可以添加筛选条件               | 可以                 | 不可以   |
| 是否有返回值                       | 有，返回受影响的行数 | 没有     |
| 是否可以回滚                       | 可以                 | 不可以   |
| 执行效率                           | 低                   | 高       |



### 7、数据查询语言 DQL









### 8、事务控制语言 TCL

#### 8.1、概述

一条或多条sql语句组成一个执行单位，要么全部执行成功，要么全部不执行成功。

#### 8.2、特点（ACID）

* **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

* **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
* **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
* **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

#### 8.3、使用

* 隐式事务（自动事务）

  没有明显的开始和结束，本身就是一条事务可以自动提交，比如insert、update、delete。

  

* 显示事务

  具有明显的开始和结束。使用步骤如下：

  ```sql
  #禁止自动提交事务，只作用于当前会话
  set autocommit=0;
  #开启事务，可以省略，因为禁止自动提交事务就默认开启了事务
  start transaction;
  
  #执行的逻辑sql语句，
  #支持insert、update、delete
  
  #设置回滚点，即当回滚事务的时候，可以指定回滚到哪个回滚点
  savepoint 回滚点名;
  
  #结束事务
  #提交
  commit;
  #回滚
  rollback;
  #回滚到指定回滚点
  rollback to 回滚点名;
  ```

  

#### 8.4、并发事务

并发事务就是当多个事务同时操作同一个数据库中的相同数据。

##### 8.4.1、引发问题

* **脏读：**一个事务读取了其他事务还没有提交的数据，读到的是其他事务“更新”的数据。
* **不可重复读：**一个事务多次读取，结果不一样。
* **幻读：**一个事务读取了其他事务还没有提交的数据，读到的是其他事务“插入”的数据。



##### 8.4.2、解决方案

通过设置事务的隔离级别来解决并发事务问题。

* Read Uncommitted（读未提交）

  该隔离级别允许脏读取，其隔离级别最低。换句话说，如果一个事务正在处理某一数据，并对其进行了更新，但同时尚未完成事务，因此还没有进行事务提交；而与此同时，允许另一个事务也能够访问该数据。举个例子来说，事务A和事务B 同时进行，事务A在整个执行阶段，会将某数据项的值从1开始，做一系列加法操作（比如加1操作）直到变成10 之后进行事务提交，此时，事务B能够看到这个数据项在事务A操作过程中的所有中间值（如 1 变成 2、2 变成 3 等），而对这一系列的中间值的读取就是未授权读取。

* Read Committed（读已提交）

  该隔离级别只允许读取已经被提交的数据，因此禁止了脏读。

* Repeatable Read（可重复读）

  该隔离级别保证在事务处理过程中，多次读取同一个数据时，其值都和事务开始时刻是一致的。因此该事务级别禁止了不可重复读和脏读。

* Serializable（串行化）

  该隔离级别是最严格的，它要求所有事务都被串行执行，即事务只能一个接一个地进行处理，不能并发执行。



### 9、索引