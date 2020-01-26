# 数据完整性约束

1、主键

2、外键

3、范式

## union :联合查询 

######   作用：查询多表字段的相同字段

```
SELECT country, name FROM Websites
WHERE country='CN'
UNION ALL
SELECT country, app_name FROM apps
WHERE country='CN'
ORDER BY country;

## 
```

## 函数（排序）正则表达

   降序：`SELECT * FROM user ORDER BY age desc`

   升序：`SELECT * FROM user ORDER BY age desc`

分组 ：

```
SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name;
```

  分组统计：

```
SELECT coalesce(name, '总数'), SUM(singin) as singin_count FROM  `user` GROUP BY name WITH ROLLUP;
```

作用：分组统计统计用户登陆情况

## INNER JOIN  连接查询

多表连接查询：join 提取相同部分输出

![](/home/yqf/图片/2019-09-22 11-06-00 的屏幕截图.png)

```
SELECT a.id, a.name,a.city, b.count FROM user a JOIN addrss b on a.city = b.city;
```

left join 左连接：把相同部分提取+左表的全部数据输出

```
SELECT a.id, a.name,a.city, b.count FROM user a LEFT JOIN addrss b on a.city = b.city;
```

![](/home/yqf/图片/2019-09-22 11-18-03 的屏幕截图.png)

right: 右连接同上

## 空值处理

```
 SELECT * FROM runoob_test_tbl WHERE runoob_count IS NULL;
```

```
 SELECT * FROM runoob_test_tbl WHERE runoob_count IS NOT NULL;
```

错误的处理方式

```
SELECT * FROM runoob_test_tbl WHERE runoob_count = NULL;

SELECT * FROM runoob_test_tbl WHERE runoob_count != NULL;
```

## 正则表达式

```
SELECT name FROM person_tbl WHERE name REGEXP '正则表达式';
```

## 事物

作用：

  MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

step1:事物初始配置

begin 开始事物

事物

commit 提交事物（end）

rollback 回滚

如果事物出错就会进行回滚（让整个事物失效）

### 事物的隔离级别

1、可读

2、提交课读  comod 之后可读（防止读脏数据）；

3、重复可读 可以读取原有数据不收其他事物数据影响

4、串行读取 只能有一个事物处理数据

# 数据库设计

   范式：要满足一定的设计规范，并不是强制要求，规范有助于数据的处理，减少数据冗余。。。

  第一范式：1、要有主键，2、字段不可再分；

  第二范式：第二范式建立在第一范试之上，非主键字段要完全依赖与主键，不能产生部分依赖。（不能使用联合主键，应该在生成一张关系表用来说明两个实体之间的关系）典型 的n对n设计

  第三范式：要求非主键字段不存在传递依赖于主键字段 典型 1 对  n 

1  对  1 加字段唯一性（只能被外键调用一次）；

# 存储过程

理解存储过程的优缺点

存储过程能够比普通语句更快执行

# 索引

建立索引能够更快查找到数据

其底层的原因是数据库在创建索引的同时会把这些字段排好序（类是于数据结构）能够更快的找出数据

二叉树

红黑树

。。。。

# 视图

一个视图是一个操作的集合，方便数据库测试时使用



# 触发器