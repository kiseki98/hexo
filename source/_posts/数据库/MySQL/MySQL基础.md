---
title: MySQL基础
date: 2022/5/1 20:16:25
tags:
- MySQL
categories:
- [MySQL, 基础]
description: MySQL的锁简介
---

# MySQL基础

## MySQL常见命令

1. 显示数据库：`SHOW DATABASES`
2. 使用数据库：`USE DATABASE`
3. 显示库中表：`SHOW TABLES FROM DATABASE`
4. 查看表结构：`DESC TABLE`

## 常见知识

1. `+` 号只有运算功能，有一方是字符串(转为数字，失败转为0)按数字计算
2. 任何数据与 `NULL` 运算都是 `NULL` 

## 常见函数

### 字符函数

1. `CONCAT(f1，f2)`：连接字段，也可连接字符串
2. `LENGTH()`：可用于获取字段的字符长度
3. `UPPER()/LOWER()`：改大/小写函数
4. `SUBSTR(str，start，end)`：截取指定索引范围字符串，没有end表示全部，索引从1开始
5. `LEFT(str，size)/RIGHT(str，size)`：截取字符串左/右边几个
6. `INSTR(原，子)`：返回子串在原字符串起始索引
7. `TRIM(str)`：去除字符串的空格，`TRIM('a' FROM '达到a')`表示从字符串去除`a`，还有`LTRIM()/RTRIM()`
8. `LPAD()/RPAD(str，size，字符)`：表示以指定字符左/右填充字符串到指定长度
9. `REPLACE(str，target，replace)`：把`str`中的`target`替换为`replace`
10. `REVERSE(str)`：将字符串反转过来

### 数学函数

1. `ROUND()`：四舍五入，`ROUND(1.254，2)`表示保留两位小数
2. `CEIL()`：向上取整
3. `FLOOR()`：向下取整
4. `TRUNCATE()`：截断，`TRUNCATE(1.6354，1)`表示只留下小数点后一位
5. `MOD(a，b)`：取余，实质为`a-a/b*b`
6. `RAND()`：随机获取`0-1`之间的数字

### 日期函数

1. `NOW()`：返回系统当前日期和时间
2. `CUDATE()/CURTIME()`：返回系统当前日期/时间，两者其一
3. `YEAR()/MONTH()/MONTHNAME()`：获取时间的年/月/月名
4. `DATEDIFF(date1，date2)`：返回相差多少天
5. `TIMESTAMPDIFF(Unit，birth，CURRENT())`：获取两时间相差年/月/日
6. `STR_TO_DATE(date，"%Y-%m-%d")`：格式化日期，%Y(4位年)、&m(2位月)、%d(日)
7. `DATA_FORMATTER(date，pattern)`：%i(分)、%y(2位年)、%c(不补0月)、%H(24小时)、%h(12小时)、%s(秒)
8. `ADDDATE(d，n)`：在d日期加上n天

### 流程控制函数
![无标题](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%97%A0%E6%A0%87%E9%A2%98.png)
> IF(A，B，C)相当于三目运算符

### 分组函数

1. `SUM()/AVG()/MIN()/MAX()`：忽略NULL那一行，会影响其他聚合函数
2. `COUNT()`：`COUNT(*`)统计行数
3. 和分组函数一同查询的字段要求是`GROUP BY`后面的字段

### 其他函数

1. `IFNULL(字段，返回值)`：字段为`NULL`，设置一个值，类似oracle的nvl()函数
2. `IS NULL()`：判断某字段或者表达式是否为`NULL`

## 查询分类

1. 条件查询：`>、<、>=、<=、<>(相当于!=)、<=>(安全等于，可以判断NULL和普通值)`
2. 逻辑表达式：`&&、||、！、AND、OR、NOT`
3. 模糊查询：`LIKE、BETWEEN AND、IN、IS NULL、IS NOT NULL`，不能使用 `=` 判断 `NULL`

## 表相关操作

### 查询

1. 分组查询
   1. `GROUP BY`必须与`SELECT`中用到的字段一致
   2. `HAVING`用于分组后筛选，`WHERE`用于分组前筛选
   3. `HAVING `与 `GROUP BY` 组合可使用聚合函数。`SELECT score FROM a  GROUP BY score HAVING MIN(score) > 0`
   4. `HAVING  ` 单独使用不可以跟聚合函数，`SELECT score FROM a HAVING score > 0`
   
2. 连接查询：`ON` 连接条件

   ```sql
   SELECT * FROM t1 FULL/LEFT/RIGHT JOIN t2 ON t1.sno=t2.sno WHERE t1.no>1 # 外连接
   SELECT * FROM t1 INNER JOIN t2 ON t1.sno=t2.sno WHERE t1.no>1 # 内连接
   SELECT salary,level FROM emp,lel WHERE salary BETWEEN lel.low AND lel.high # 非等值连接
   ```

3. 分页查询：`SELECT * FROM a LIMIT offset，size`,`offset`表示起始(0对应第一行)，`size`表示条目

4. 联合查询：`SELECT * FROM UNION SELECT * FROM`；要求两个查询结果列数一致即可

5. 子查询
   1. 分类：`ANY、SOME、ALL`表示和子查询项目比较，`ALL` 表示全部满足
      1. 标量子查询(1行1列)
      2. 列子查询(1列)
      3. 行子查询(1行)
      4. 表子查询(n行n列)
   2. 按出现位置分类：
      1. `SELECT`：只能时标量子查询
      2. `FROM`：表子查询
      3. `WHERE/HAVING/IN`：标量/列/行子查询
      4. `EXISTS`：表子查询

### 表管理

`auto_increment`表示设置自动增长(自动增长列称为标识列，标志列只能有一个，必须为主键/唯一键)

1. 创建表

   ```sql
   CREATE TABLE tb4 (
       id int(10),
       `name` varchar(30),
       age int(10),
       gender tinyint(1),
       PRIMARY KEY(id，age),
       CONSTRAINT 约束名 UNIQUE(name)
   ) ENGINE INNODB;
   # CONSTRAINT可以省略，约束名也可以省略，创建主键和唯一键同时会创建索引
   # 外键：外键关联主表字段必须是key(主键/唯一键)
   ```

2. 修改表：
   1. 修改表名：`ALTER TABLE tb RENAME TO tb1`
   2. 添加/删除/改变/修改列：`ALTER TABLE tb1 ADD/DROP/CHANGE/MODIFY COLUMN 列名 int(10)`
   3. 只有 `ADD` 才可以添加列同时添加约束/索引，其他修改(`CHANGE，MODIFY`)可以添加约束
   
3. 删除表：`DROP TABLE tb`

4. 复制表：
   1. 仅复制结构：`CREATE TABLE tb2 LIKE tb1`
   2. 复制结构和数据：`CREATE TABLE tb3 SELECT * FROM tb1`

### 约束(列级约束操作对表级约束无效)

1. 六大约束：`NOT NULL、DEFAULT、PRIMARY KEY、UNIQUE、CHECK(MySQL不支持，指定范围值)、FOREIFN KEY(外键)`
2. 主键和唯一键：都保证唯一性，主键不能为`NULL`，唯一键只能有一个`NULL`，主键唯一，唯一键可以多个，都可以组合
3. 表级约束
   1. ```sql
      CREATE TABLE tb4 (
          id int(10),
          `name` varchar(30),
          age int(10),
          gender tinyint(1),
          PRIMARY KEY(id，age),
          CONSTRAINT 约束名 UNIQUE(name),
          CONSTRAINT fk_表1_表2 FOREIGN KEY (NAME) REFERENCES major(id)
      ) ENGINE INNODB;
      # CONSTRAINT可以省略，约束名也可以省略，创建主键和唯一键同时会创建索引
      # 外键：外键关联主表字段必须是key(主键/唯一键)
      ```
   
   2. 修改表时添加/修改约束：
      1. 主键增删：`ALTER TABLE tb1 ADD/DROP PRIMARY KEY(id)`
      2. 唯一键(索引)增删：`ALTER TABLE tb1 ADD UNIQUE(name)/DROP INDEX 索引名`
      3. 外键增删：`ALTER TABLE tb1 ADD/DROP FOREIGN KEY(age) REFERENCES major(id)`
4. **列级约束(不支持外键)：**`ALTER TABLE tb1 MODIFY COLUMN 列名 int(10) UNIQUE`

### 数据类型

1. 数值：
   1. 整型：`Tinyint(1)，smallint(2)，mediumint(3)，int(4)，bigint(8)；int(4) unsigned`表示无符号
   2. 浮点型：`float(M,D)/double(M,D)/dec(M,D)`。`M`表示小数总位数，`D`代表小数部分长度；`dec(M,D)`时定点型，精度较高，默认`M10`，`D0`，而`float`和`double`根据插入数据确定精度
   
2. 字符：

   1. 文本：`TinyText`(短文本)、`Text`(长文本)、`MediumText`(中等文本)、`LongText`(极大文本)

   2. 二进制文本：`Blob`(二进制长文本)、`MediumBlob`(二进制中等文本)、`LongBlob`(二进制极大文本)

   3. `char(30)/varchar(30)`：括号里面指定代表字符长度

   4. | 文本类型  |    长度     | M值(实际存储字节) |    占用空间     | 字符集 |   读取   | 默认值 |       支持索引        |   检索速度    |
      | :-------: | :---------: | :---------------: | :-------------: | :----: | :------: | :----: | :-------------------: | :-----------: |
      |  `char`   | 固定/可指定 |  `0 <= M <= 255`  | 小于`M`空格补齐 |  指定  |          |  支持  | 直接创建/指定前缀长度 | 快于`varchar` |
      | `varchar` | 可变/可指定 | `0 <= M <= 65535` |    `M+1/+2`     |  指定  |          |  支持  | 直接创建/指定前缀长度 |  快于`text`   |
      |  `text`   |  不可指定   |     `M<2^16`      |      `M+2`      |  指定  |          |  不能  |     指定前缀长度      | 慢于`varchar` |
      |  `blob`   |  不可指定   |     `M<2^16`      |      `M+2`      | 不指定 | 整体读取 |  不能  |     指定前缀长度      |               |

      

3. 时间：

   1. `TimeStamp/DATE/DATETIME`

### 数据操作

1. 新增数据：
   1. `INSERT INTO tablename (sno,class,score) VALUES(4,'数学',78)`
   2. `INSERT INTO table SET sno=5，class='数学'，score=63`
   3. `SELECT * INTO table1 FROM table2`
2. 修改数据：`UPDATE table SET score=60 WHERE sno=1`
3. 删除数据：
   1. `DELETE FROM table WHERE sno=1`，有返回值，删除最后一条数据，新增一条数据(id跟在删除的数据id后面)
   2. `TRUNCATE table`，只能删除全部数据，不能加`WHERE`条件，主键自增重置，无返回值
   3. 多表删除：`DELETE 要删除的表 FROM 表1 INNER JOIN 表2 ON 表1.no=  表2.no WHERE 表1.no=1`

### 事务(仅INNODB支持)

1. `set autocommit=0`
2. `start transaction`，可选
3. 执行的`SQL`
4. `commit/rollback`：提交或者回滚

### 事务ACID

1. 原子性 `Atomicity`：一个事务中的所有操作，要么全部完成，要么全部不完成
2. 一致性 `Consistency`：事务前后数据的完整性必须保持一致，这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作
3. 隔离性 `Isolation`：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致
4. 持久性 `Durability`：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失

|          隔离级别          |              读数据一致性              | 脏读 | 不可重复读 | 幻读 |
| :------------------------: | :------------------------------------: | :--: | :--------: | :--: |
|  读未提交 `Read Uncommit`  | 最低级别，只保证不读取物理上损坏的数据 |  √   |     √      |  √   |
|   读已提交 `Read Rommit`   |                 语句级                 |  ×   |     √      |  √   |
| 可重复读 `Repeatable read` |                 事物级                 |  ×   |     ×      |  √   |
|   串行化 `Serializable`    |             最高级，事物级             |  ×   |     ×      |  ×   |

**隔离性举例**：

1. 丢失修改数据：银行卡有100元，事务A取10元，事务B取10元，事务AB两人同时取钱，初始值都是100 
2. 读**脏**数据：数据库技术中，如果正常提交的事务A使用了事务B未提交的撤销数据，这种数据成为**脏数据**，会造成数据的**脏读**
3. 不一致分析：造成这种数据不一致的主要原因是并发执行的两个事务中，一个事务在读取数据时，另一个事务正在修改同一个数据。这样就可能导致两个事务的相互干扰及**读事务**的错误执行结果

**事物并发操作出现几种问题，事务ACID特性可能遭到破坏的因素有**：

1. 多个事务并发执行，不同事务的操作交叉执行
2. 事务在运行过程中被强行终止。 

**如何保证在多个事务并发执行的过程中不发生上述的两种情况，是数据库管理系统并发控制的主要责任** 

### 视图

1. 创建：`CREATE VIEW 视图名 AS 查询语句`
2. 视图修改：
   1. `CREATE OR REPLACE VIEW 视图名 AS 查询语句`
   2. `ALTER VIEW 视图名 AS 查询语句`
3. 视图删除：`DROP VIEW 视图名`
4. 数据添加/修改：和表一样操作
5. 含义：和普通表一样使用，虚拟表，通过表动态生成数据
6. 作用：实现重用`SQL`、简化复杂`SQL`，不必知道查询细节、保护数据提高安全性

### 存储过程

1. 创建

   ```sql
   CREATE PROCEDURE pro(IN tb_no INTEGER)
   BEGIN 
   DELETE FROM tablea where a.sno = tb_no
   END;
   # 调用
   CALL pro(3)
   ```

2. 调用：`CALL 存储过程名(参数)`

3. 参数类型：`IN`(进)、`OUT`(出，作为返回值)、`INOUT`(进出，两个功能的结合)

### 函数

1. 创建

   ```sql
   CREATE FUNCTION func(p_con VARCHAR(400) ) RETURNS VARCHAR(400) 
   BEGIN 
   DECLARE v_con VARCHAR(400) 
   SET v_con = p_con
   SELECT p_con INTO v_con
   RETURN v_con
   END;
   # 调用
   SELECT func('你好')
   ```

2. 调用：`SELECT 函数名(参数列表)`

# 索引

1. 定义

   1. 索引是一种数据结构，相当于图书的目录，快速查找指定内容
   2. 数据库使用索引以找到特定值，然后顺指针找到包含该值的行
   3. 在表中建立索引，找到符合查询条件的索引值，最后通过保存在索引中的ROWID找到表中记录

2. 分类

   1. 唯一索引：`UNIQUE(PRAMARY KEY是特殊的UNIQUE)`，不允许重复值 `UNIQUE` 可以存在一个 `NULL`，`PRAMARY KEY  ` 不行
   2. 普通索引：允许重复值
   3. 单列索引：索引建在某个列上
   4. 组合索引：索引建在多个列上

3. 查看索引：`SHOW INDEX/KEYS FROM table_name`

4. 冗余索引：多个索引的前缀列相同，或在组合索引中包含了主键的索引(每个索引都把主键包含到最后，手动加`id`就是冗余索引)，`key(name，id)`是冗余索引

5. 重复索引：指相同的列以相同的顺序建立的同类型的索引

6. 创建索引

   1. 创建表创建索引

      ```sql
      CREATE TABLE tb4 (
          id int(10),
          `name` varchar(30),
          age int(10),
          gender tinyint(1),
          key/index idx_id_age(id,age) #这里是索引
      ) ENGINE INNODB
      ```

   2. 修改表时

      ```sql
      ALTER TABLE table_name ADD INDEX index_name (column_list) # 普通索引
      ALTER TABLE table_name ADD UNIQUE (column_list) # 唯一索引
      ALTER TABLE table_name ADD PRIMARY KEY (column_list) # 主键索引
      ```

   3. 直接创建索引

      ```sql
      CREATE INDEX index_name ON table_name (column_list) # 普通索引
      CREATE UNIQUE INDEX index_name ON table_name (column_list) # 唯一索引
      ```

7. 删除索引：

   ```sql
   DROP INDEX index_name ON talbe_name
   ALTER TABLE table_name DROP INDEX index_name
   ALTER TABLE table_name DROP PRIMARY KEY # 删除主键索引
   ```

# 数据库权限管理

```sql
REVOKE CREATE/DROP/ALTER/SELECT/INSERT/UPDATE/DELETE ON T FROM user2 # 回收建表、改表、删表、查表等权限
GRANT CREATE/SELECT/ALTER/DROP ON T FROM user2 # 授予表的权限
```

# 三大范式

## 三大范式

1. 第一范式`First Normal Form` ：每列都是不可分割单元，则属于第一范式。
   1. 数据库表中的字段都是单一属性的，不可再分
   2. 姓名字段，其中的姓和名必须作为一个整体，无法区分哪部分是姓，哪部分是名，如果要区分出姓和名，必须设计成两个独立的字段
2. 第二范式`Second Normal Form` ：若关系模式R属于第一范式，且每个非主属性都是完全函数依赖于主键，则R属于第二范式。
   1. 要求数据库表中的每个实例或行必须可以被惟一地区分，通常需要为表加上一个列，以存储各个实例的惟一标识。这个惟一属性列主键
   2. 要求实体的属性完全依赖于主关键字。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性，如果存在，那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体，新实体与原实体之间是一对多的关系。为实现区分通常需要为表加上一个列，以存储各个实例的惟一标识。简而言之，第二范式就是非主属性非部分依赖于主关键字。
3. 第三范式 `Third Normal Form` ：若关系模式R属于第一范式，且每个非主属性都不传递函数依赖于主键，则R属于第三范式。
   1. 要求一个数据库表中不包含已在其它表中已包含的非主关键字信息
   2. 所以第三范式具有如下特征
      1. 每一列都是单一属性不可分割
      2. 每一行都能根据唯一标识区分
      3. 每一个表都不包含其他表已经包含的非主关键字信息
   3. 例如：帖子表中只能出现发帖人的 `id`，不能出现发帖人的 `id` 同时出现发帖人姓名，否则，只要出现同一发帖人 `id` 的所有记录，它们中的姓名部分都必须严格保持一致，这就是数据冗余。

## 几种数据模型

1. 层次模型：采用的是树（二叉树）的结构来表达实体和实体间联系
2. 网状模型：采用的是图的结构来表达实体和实体间联系
3. 对象模型：就是用的面型对象的思想，用对象和其之间的联系来表达实体和实体间联系
4. 关系模型：就是用的二维表

## SQL四种语言

1. `DDL`：数据定义语言，用来维护存储数据的结构。如`create、drop、alter`不需要`commit`
2. `DML`：数据操作语言，用来对数据进行操作。如`Insert、select、delete、update`
3. `DCL`：数据控制语言，用来负责权限管理。`grant、revoke`
4. `TCL`：事务控制语言，用来对事务操作。如`savepoint、rollback、set transaction`

## 关系完整性约束

1. 域（列）完整性：域完整性是对数据表中字段属性的约束，通常指数据的有效性,它包括字段的值域、字段的类型及字段的有效规则等约束，它是由确定关系结构时所定义的字段的属性决定的。限制数据类型,缺省值,规则,约束,是否可以为空,域完整性可以确保不会输入无效的值.。
2. 实体（行）完整性：实体完整性是对关系中的记录唯一性，也就是主键的约束。准确地说，实体完整性是指关系中的主属性值不能为Null且不能有相同值。定义表中的所有行能唯一的标识,一般用主键,唯一索引 `unique`关键字,及`identity`属性比如说我们的身份证号码,可以唯一标识一个人. 
3. 参照完整性：参照完整性是对关系数据库中建立关联关系的数据表间数据参照引用的约束，也就是对外键的约束。准确地说，参照完整性是指关系中的外键必须是另一个关系的主键有效值，或者是`NULL`。参考完整性维护表间数据的有效性,完整性,通常通过建立外部键联系另一表的主键实现,还可以用触发器来维护参考完整性

# MySQL执行顺序

MySQL的语句一共分为11步，如下图所标注的那样，最先执行的总是FROM操作，最后执行的是LIMIT操作。其中每一个操作都会产生一张虚拟的表，这个虚拟的表作为一个处理的输入，只是这些虚拟的表对用户来说是透明的，但是只有最后一个虚拟的表才会被作为结果返回。如果没有在语句中指定某一个子句，那么将会跳过相应的步骤。

![Image](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/Image.png)

1. FORM: 对FROM的左边的表和右边的表计算笛卡尔积。产生虚表VT1
2. ON: 对虚表VT1进行ON筛选，只有那些符合`<join-condition>`的行才会被记录在虚表VT2中。
3. JOIN： 如果指定了OUTER JOIN（比如left join、 right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。
4. WHERE： 对虚拟表VT3进行WHERE条件过滤。只有符合`<where-condition>`的记录才会被插入到虚拟表VT4中。
5. GROUP BY: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.
6. CUBE | ROLLUP: 对表VT5进行cube或者rollup操作，产生表VT6.
7. HAVING： 对虚拟表VT6应用having过滤，只有符合`<having-condition>`的记录才会被 插入到虚拟表VT7中。
