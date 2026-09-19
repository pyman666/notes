
- [数据库](#数据库)
- [insert](#insert)
- [update](#update)
- [delete](#delete)
- [select](#select)
  - [where](#where)
  - [order](#order)
  - [limit](#limit)
- [统计函数](#统计函数)
  - [count](#count)
  - [sum/avg/min/max](#sumavgminmax)
  - [group/having](#grouphaving)
- [字符串函数](#字符串函数)
- [数学函数](#数学函数)
- [日期函数](#日期函数)
- [加密和系统函数](#加密和系统函数)
- [流程控制函数](#流程控制函数)
- [子查询](#子查询)
  - [临时表](#临时表)
  - [all/any](#allany)
  - [多列子查询](#多列子查询)
  - [复制与去重](#复制与去重)
- [合并查询](#合并查询)
  - [纵向合并](#纵向合并)
  - [自连接](#自连接)
  - [外连接](#外连接)
- [约束](#约束)
- [默认](#默认)
- [自增](#自增)
- [隔离级别](#隔离级别)
- [ACID](#acid)
- [用户管理](#用户管理)
- [权限管理](#权限管理)

## 数据库

## insert
```sql
INSERT INTO tb_name [column, column...]
VALUES (value, value...)

-- 一次性插入多条
INSERT INTO students (name, age)VALUES ('tom', 12), ('kevin', 13);

-- 添加所有字段可以不写列名
INSERT INTO students VALUES ('tom', 12), ('kevin', 13);
```
## update
```sql
UPDATE tb_name
SET column1=expr1[, column2=expr2...]
[WHERE expr]
```
## delete

- delete仅删除记录，不删除表本身。删表用drop语句；

- 如果不使用where，将删除表中所有数据；

- Delete语句不能删除某一列的值（可使用update设为null或''）
```sql
DELETE FROM tb_name [WHERE expr]
```
## select
```sql
SELECT [DISTINCT] *|{col1|expr1 [AS name1], col2|expr2 [AS name2]}
FROM tb_name

SELECT name, (chinese+math) as score FROM student;
```
### where
```sql
SELECT column FROM tb_name [WHERE expr]
```
其中`BETWEEN.AND.`是闭区间

### order

- order by 指定的列既可以是表中的列，也可以是select语句后指定的列；

- ASC升序（默认），DESC降序；

- 顺序：group by - having - order by - limit
```sql
SELECT column FROM tb_name [ORDER BY col ASC|DESC]
```
### limit

常用于分页查询
```sql
SELECT ... LIMIT start, rows;  -- 从start+1行开始取，取出rows行，start从0开始计算

SELECT * FROM tb ORDER BY id LIMIT 0, 3;  -- 第一页
SELECT * FROM tb ORDER BY id LIMIT 3, 3;  -- 第二页
```
# 函数

## 统计函数

### count

- `count(*)`：满足条件的行数；

- `count(col)`：某列满足条件的行数，会排除`null`的情况
```sql
SELECT COUNT(*)|COUNT(column) FROM tb_name
```
### sum/avg/min/max
```sql
SELECT SUM(chinees+math) AS sum_score FROM tb_name;  -- 求和
SELECT AVE(chinees+math) AS avg_score FROM tb_name;  -- 平均数
SELECT MIN(chinees+math) AS min_score FROM tb_name;  -- 最小值
SELECT MAX(chinees+math) AS max_score FROM tb_name;  -- 最大值
```
### group/having

- group by用于对查询结果分组统计；

- having用于限制分组显示结果
```sql
SELECT col1, col2 FROM tb_name GROUP BY col [HAVING expr]

SELECT AVG(english) AS avg_eng, gender, age FROM student
GROUP BY gender, age  -- 根据性别年龄求平均分
HAVING avg_eng < 90  -- 平均分小于90
```
## 字符串函数
```sql
SELECT * FROM tb WHERE name LIKE '__O%'  -- 第三个字符为O的记录
```
## 数学函数

## 日期函数
```sql
-- 查询10分钟内的记录
SELECT * FROM tb
WHERE DATE_ADD(send_time, INTERVAL 10 MINUTE) >= NOW();

-- 转换UNIX时间戳
SELECT UNIX_TIMESTAMP() FROM DUAL;
SELECT FROM_UNIXTIME(1618483484, '%Y-%m-%d %H:%i:%s') FROM DUAL;
```
## 加密和系统函数

## 流程控制函数
```sql
SELECT IF(col IS NULL, 0, col) FROM tb_name;
SELECT IFNULL(col, 0) FROM tb_name;
SELECT CASE WHEN col IS NULL THEN 0 ELSE col END;
```
# 多表查询

多表查询的条件不能少于（表的个数-1），否则会出现笛卡尔积
```sql
SELECT * worker.ename AS '职员', boss.ename AS '上司'
FROM worker, boss
WHERE worker.mgr = boss.empno;
```
## 子查询

子查询也叫嵌套查询，是指嵌入在其他sql语句中的select语句
```sql
-- 单行子查询
SELECT * FROM emp WHERE deptno = (
  SELECT deptno FROM emp WHERE ename = 'jack'
)

-- 多行子查询
SELECT ename, job FROM emp WHERE job IN (
  SELECT DISTINCT job FROM emp WHERE deptno = 10
) AND deptno <> 10

-- 多列子查询
SELECT * FROM emp WHERE (deptno, job) = (
  SELECT deptno, job FROM emp WHERE ename = 'jack'
) AND ename != 'jack'
```
### 临时表

子查询当作临时表使用，某些时刻非常有用
```sql
SELECT goods_id, ecs_goods.cat_id, goods_name, shop_price FROM (
  SELECT cat_id, MAX(shap_price) as max_price
  FROM ecs_goods GROUP BY cat_id
) temp, ecs_goods
WHERE temp.cat_id = ecs_goods.cat_id AND temp.max_price = ecs_goods.shop_price
```
### all/any
```sql
-- 比30部门最高还要高，也可用MAX
SELECT ename, sal, deptno FROM emp
WHERE sal > ALL(SELECT sal FROM emp WHERE deptno = 30)

-- 比30部门其中一个高，也可用MIN
SELECT ename, sal, deptno FROM emp
WHERE sal > ANY(SELECT sal FROM emp WHERE deptno = 30)
```
### 多列子查询
```sql
SELECT * FROM emp
WHERE (age, job) = (SELECT age, job FROM emp WHERE ename='JACK')
AND ename != 'JACK'
```
### 复制与去重

有时为了对某个sql进行效率测试，需要海量数据，可以使用此法为表创建数据
```sql
-- 从其他表复制
INSERT INTO tb (ename, sal, deptno)
SELECT ename, sal, deptno FROM emp;

-- 自我复制
INSERT INTO tb
SELECT * FROM tb;

-- 复制结构（列 ）
CREATE TABLE tb LIKE emp;
```
通过临时表去重
```sql
CREATE TABLE temp LIKE tb;  -- 临时表
INSERT INTO temp SELECT DISTINCT * FROM tb;
DELETE FROM tb;
INSERT INTO tb SELECT * FROM temp;
DROP TABLE temp;
```
## 合并查询

### 纵向合并

取得多个select语句的结果的并集
```sql
-- UNION ALL 不去重
-- UNION 去重
SELECT ename, age FROM emp WHERE sal>5000
UNION ALL
SELECT ename, age FROM emp WHERE age<30;
```
### 自连接

自连接指在同一张表的连接查询

- 把同一张表当两张表使用；

- 需要给表取别名，列名不明确也可以给列取别名
```sql
SELECT * worker.ename AS '职员', boss.ename AS '上司'
FROM emp worker, emp boss
WHERE worker.mgr = boss.empno;
```
### 外连接

左表完全显示，左外连接；右表完全显示，右外连接；
```sql
SELECT name, stu.id, grade FROM stu
LEFT JOIN exam
ON stu.id = exam.id
```
# 修饰

## 约束

约束用于确保数据库数据满足特定的商业规则，mysql中约束主要有： `not null`、`unique`、`primary key`、`foreign key`、`check`五种。

**主键**（primary key）：

- 主键的值不能重复，且不能为null；

- 一张表最多只有一个主键，但是可以是复合主键；

- 使用`DESC 表名`，可以看到主键情况；

**非空**（not null）：

- 设置了非空，插入数据时，必须为列提供数据

**唯一**（unique）：

- 设置了唯一，该列的值不能重复

**外键**（foreign key）：

- 外键约束定义在从表上；

- 主表必须具有主键约束或是unique约束；

- 从表定义外键约束后，外键列数据必须在主表的主键列存在或者为null

**检查**（check）：

- 用于强制行数据必须满足的条件
```sql
CREATE TABLE tb
(id INT PRIMARY KEY,  -- 主键
name VARCHAR(32) UNIQUE NOT NULL);  -- 唯一，非空

CREATE TABLE tb
(id INT,
age INT CHECK (18<age AND age<35),  -- 检查
PRIMARY KEY (id, age));  -- 复合主键

CREATE TABLE tb1
(id INT,
tb_id INT,
FOREIGN KEY (tb_id) REFERENCES tb (id));  -- 外键
```
## 默认
```sql
CREATE TABLE tb (name VARCHAR(32) DEFAULT '');
```
## 自增

- 自增一般和primary key配合使用；

- 自增也可以单独使用，但需配合unique；

- 自增修饰的字段一般为整数类型；

- 自增默认从1开始，也可以通过命令修改：`alter table xx auto_increment = xxx`;

- 添加数据时，如果自增列有给定值，则以给定值为准
```sql
CREATE TABLE tb (id INT PRIMARY KEY AUTO_INCREMENT);
```
# 索引

索引类型：主键索引、唯一索引、普通索引、全文索引

- 创建索引后，只对创建了索引的列有效；

- 索引本身也占空间，空间换时间；

- 索引对DML（update delete insert）语句的效率有影响

标准：

- 较频繁作为查询条件的字段应该创建索引；

- 唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件；

- 更新非常频繁的字段不适合创建索引；

- 不会出现在WHERE子句中的字段不应该创建索引
```sql
-- 查询索引
SHOW INDEX FROM tb;
SHOW INDEXES FROM tb;
SHOW KEYS FROM tb;

-- 删除索引
DROP INDEX id_index ON tb;
ALTER TABLE tb DROP PRIMARY KEY;  -- 删除主键索引

-- 修改索引：先删除，再创建

-- 创建索引
CREATE INDEX id_index ON tb (id);  -- 普通索引，法一
ALTER TABLE tb ADD INDEX id_index (id);  -- 普通索引，法二
CREATE UNIQUE INDEX id_index ON tb (id);	-- 唯一索引
CREATE TABLE tb (id INT PRIMARY KEY, ...);  -- 主键索引，法一
ALTER TABLE tb ADD PRIMARY KEY (id);  -- 主键索引，法二
```
# 事务

**事务**用于保证数据的一致性看，它由一组相关的DML语句组成，该组的DML语句要么全部成功，要么全部失败。当执行事务操作时（DML语句），mysql会在表上加**锁**，防止其他用户更改表的数据。

注意事项：

- 如果不开始事务，默认DML操作自动提交，不能回滚；

- 可以在事务中创建多个保存点。如果开始事务没有创建保存点，执行rollback默认回退到事务开始时的状态；

- 开始一个事务 start transaction，或者 set autocommit=off；

- commit会结束事务，删除保存点，释放锁，不能回退。所有操作生效，其他会话可以查看到事务变化后的新数据

- mysql事务机制需要innodb存储引擎可以使用，myisam不行；
```sql
START TRANSACTION  -- 开始事务
SAVEPOINT a  -- 设置保存点
  INSERT INTO tb VALUES(100, 'Tom');
  SELECT * FROM tb;
SAVEPOINT b
  INSERT INTO tb VALUES(90, 'Jack');
  SELECT * FROM tb
ROLLBACK TO b  -- 回退事务
  SELECT * FROM tb;
ROLLBACK TO a  -- 不能再到b
  SELECT * FROM tb;
ROLLBACK  -- 回退全部事务
COMMIT  -- 提交事务
```
## 隔离级别

多个连接开启各自事务操作数据库中的数据时，数据库系统要负责隔离操作，以保证各个连接在获取数据时的准确性。如果不考虑隔离级别，可能引发：脏读、幻读、不可重复读。

- **脏读**（dirty read）：当一个事务读取另一个事务尚未提交的修改时，产生脏读；

- **幻读**（phantom read）：同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，发生幻读；

- **不可重复读**（norepeatable read）：同一查询在同一事务中多次修改，由于其他提交事务所做的修改或删除，每次返回不同的结果集，发生不可重复读

隔离级别（默认**可重复读**repeatable read）：
```sql
SELECT @@tx_isolation;  -- 查询当前会话隔离级别
SELECT @@global.tx_isolation;  -- 查询系统当前隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  -- 设置当前会话隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  -- 设置系统当前隔离级别
```
## ACID

事务的ACID特性：

- **原子性** Atomicity：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生；

- **一致性** Consistency：事务必须使数据库从一个一致性状态变换到另一个一致性状态；

- **隔离性** Isolation：多个用户并发访问数据库时，数据库为每一个用户开启的事务不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离；

- **持久性** Durability：一个事务一旦被提交，它对数据库中的数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

# 存储引擎

表类型由存储引擎（Storage Engines）决定，主要包括MyISAM、innoDB、Memory等；

mysql数据表主要支持六种类型：csv、memory、archive、mrg\_myisam、myisam、innodb；这六种又可以分为两类：事务安全型（transaction-safe，如innoDB）、非事务安全型（no-transaction-safe，如mysiam、memory）

- **MyISAM**不支持事务，也不支持外键，但是访问速度快，对事务完整性没有要求；

- **InnoDB**提供了具有提交、回滚、崩溃恢复能力的事务安全，但是比起myisam，处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引；

- **Memory**使用存于内存中的内容来创建表，每个memory表只实际对应一个磁盘文件。memory类型的表访问非常快，因为其数据存于内存中，并且默认使用hash索引。但是一旦服务关闭，表中的数据就会丢失掉，表的结构还在。

选择依据：

- 如果应用不需要事务，处理的只是基本的CRUD操作，选myisam；

- 如果需要支持事务，选innodb；

- memory经典用法：用户在线状态
```sql
CREATE TABLE tb (id INT PRIMARY KEY) ENGINE MYISAM;
ALTER TABLE tb ENGINE=MYISAM;  -- 修改引擎
```
# 视图

视图是一个虚拟表，其内容由查询定义。和真实表一样，视图包含列，其数据来自对应的真实表（基表，可以是多个）

- 创建视图后，数据库对应视图只有一个视图结构文件（xxx.frm），没有表文件；

- 通过视图可以修改基表的数据，视图的变化会影响基表，基表的改变也会影响到视图；

- 视图中可以再使用视图

应用场景：

- 安全。一些数据表有着重要信息，有些字段是保密的，不能让用户直接看到。可以建立一个视图，只保留非敏感字段；

- 性能。关系数据库的数据常常会分表存储，使用外键建立这些表之间的关系，这时数据库查询常会用到连接（JOIN），效率较低。可以建立一个视图，将相关的表和字段组合在一起，避免使用JOIN查询数据；

- 灵活。如果系统中有一张旧表即将被废弃，然而很多应用都基于这张表，不易修改。可以建立一个视图，视图中的数据直接映射到新表，这样就可以少做很多改动，也达到了升级数据表的目的
```sql
CREATE VIEW v1 AS SELECT id, name FROM tb;  -- 创建视图
ALTER VIEW v1 AS SELECT id, job FROM tb;  -- 修改视图
SHOW CREATE VIEW v1;  -- 查看创建视图的指令
DROP view v1, v2;  -- 删除视图
```
# MySQL管理

## 用户管理

MySQL中的用户，都存储在系统数据库mysql中的user表中

重要字段：

- host：可登录的位置，localhost表示该用户只能本机登录，也可指定ip地址。如果不指定host，则为%，表示所有IP都可以连接，也可以‘192.168.1.%’，表示192.168.1.\*的ip都可以登录；

- user：用户名（管理员root）；

- authentication\_string：密码，会通过mysql的password函数加密
```sql
CREATE USER 'usr'@'host' IDENTIFIED BY 'pwd';  -- 创建用户并指定密码
DROP USER 'usr'@'host';  -- 删除用户

SET PASSWORD = PASSWORD('pwd');  -- 修改自己的密码
SET PASSWORD FOR 'usr'@'host' = PASSWORD('pwd');  -- 修改他人密码
```
## 权限管理

不同数据库用户登陆到DBMS后，根据相应的权限，可以操作的数据库和数据对象（表、视图、触发器）都不一样
```sql
-- 给用户授权
GRANT SELECT,DELETE ON *.* TO 'usr'@'host';  -- 所有数据库的所有对象
GRANT ALL ON db.* TO 'usr'@'host';  -- db的所有对象
GRANT ALL ON db.* TO 'usr'@'host' IDENTIFIED BY 'pwd';  -- 用户存在就是改密码，不存在就是创建用户

-- 回收权限
REVOKE ALL ON db.* FROM 'usr'@'host';

-- 权限生效指令
FLUSH PRIVILEGES;  -- 如果权限没有生效，可以执行该指令
```
