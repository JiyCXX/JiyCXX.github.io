---
title: MySql 数据处理增删改查 学习笔记
#文章创建日期
date:  2018-12-13 7:30:30
#文章分类 
categories: MySql
#文章关键字
keywords: MySql
#文章描述
description: MySql 笔记
#文章标籤	
tags: 
- MySql 
- 学习
#文章顶部图片
top_img: 
#文章缩略图
cover: /img/background/mysql.jpg
---

# 数据处理增删改查练习
# 综合案例
## 1、创建数据库test01_library
```bash
CREATE DATABASE test01_library CHARACTER SET 'utf8';
USE test01_library;
```
## 2、创建表 books，表结构如下：
```bash
CREATE TABLE IF NOT EXISTS books(
id INT,
`name` VARCHAR(50),
`authors` VARCHAR(100),
price FLOAT,
pubdate YEAR,
note VARCHAR(100),
num INT
);
SHOW TABLES FROM test01_library;
DESC books;
```
## 3、向books表中插入记录
### 1）不指定字段名称，插入第一条记录
```bash
INSERT INTO books
VALUE(1,'Tal of AAA','Dickes',23,1995,'novel',11);
```

### 2）指定所有字段名称，插入第二记录
```bash
INSERT INTO books (id,NAME,`authors`,price,pubdate,note,num)
VALUE(2,'EmmaT','Jane lura',35,1993,'Joke',22);

SELECT *FROM books;
```
### 3）同时插入多条记录（剩下的所有记录）
```bash
INSERT INTO books (id,NAME,`authors`,price,pubdate,note,num) VALUES
(3,'Story of Jane','Jane Tim',40,2001,'novel',0),
(4,'Lovey Day','George Byron',20,2005,'novel',30),
(5,'Old land','Honore Blade',30,2010,'Law',0),
(6,'The Battle','Upton Sara',30,1999,'medicine',40),
(7,'Rose Hood','Richard haggard',28,2008,'cartoon',28);
```
## 4、将小说类型(novel)的书的价格都增加5。
```bash
UPDATE books SET price=price+5 
WHERE note ='novel';
```

## 5、将名称为EmmaT的书的价格改为40，并将说明改为drama。
```bash
UPDATE books 
SET price=40,note='drama'
WHERE `name`='EmmaT';
```

## 6、删除库存为0的记录。
```bash
DELETE FROM books
WHERE num=0;
```

## 7、统计书名中包含a字母的书
```bash
SELECT NAME 
FROM books 
WHERE NAME LIKE '%a
```

## 8、统计书名中包含a字母的书的数量和库存总量
```bash
SELECT COUNT(*),SUM(num)
FROM books
WHERE NAME LIKE '%a%';
```

## 9、找出“novel”类型的书，按照价格降序排列
```bash
SELECT`name`,note,price
FROM books
WHERE note ='novel'
ORDER BY price DESC;
```

## 10、查询图书信息，按照库存量降序排列，如果库存量相同的按照note升序排列
```bash
SELECT *
FROM books
ORDER BY num DESC,note ASC;

```

## 11、按照note分类统计书的数量
```bash
SELECT note,COUNT(*)
FROM books
GROUP BY note;

```

## 12、按照note分类统计书的库存量，显示库存量超过30本的
```bash
SELECT note,SUM(num)
FROM books
GROUP BY note
HAVING SUM(num)>30;

```

## 13、查询所有图书，每页显示5本，显示第二页
```bash
SELECT *
FROM books
LIMIT 5,5;

```

## 14、按照note分类统计书的库存量，显示库存量最多的
```bash
SELECT note,SUM(num)sum_num
FROM books
GROUP BY note
ORDER BY sum_num DESC
LIMIT 0,1;

```

## 15、查询书名达到10个字符的书，不包括里面的空格
```bash
SELECT CHAR_LENGTH(REPLACE(NAME,' ',''))
FROM books

SELECT NAME 
FROM books
WHERE CHAR_LENGTH(REPLACE(NAME,' ','')) >=10

```


## 16、查询书名和类型，其中note值为novel显示小说，law显示法律，medicine显示医药，cartoon显示卡通，joke显示笑话
```bash
SELECT NAME,note, CASE note WHEN 'novel' THEN '小说'
			   WHEN 'law' THEN '法律'
			   WHEN 'medicine' THEN '医药'
			   WHEN 'cartoon' THEN '卡通'
			   WHEN 'joke' THEN '笑话'
			   ELSE '其他'
			   END AS "类型"
FROM books;
```


## 17、查询书名、库存，其中num值超过30本的，显示滞销，大于0并低于10的，显示畅销，为0的显示需要无货
```bash
SELECT NAME,num,CASE num WHEN num>30 THEN '滞销'
			 WHEN num>0 AND num<10 THEN '畅销'
			 WHEN num=0 THEN '无货'
			 ELSE '正常'
			 END AS "库存状态" 
FROM books;

```

## 18、统计每一种note的库存量，并合计总量
```bash
SELECT note,SUM(num)
FROM books
GROUP BY note WITH ROLLUP;

SELECT IFNULL(note,'合计库存总量') AS note,SUM(num)
FROM books
GROUP BY note WITH ROLLUP;
```

## 19、统计每一种note的数量，并合计总量
```bash
SELECT IFNULL(note,'合计总量') AS note,COUNT(*)
FROM books
GROUP BY note WITH ROLLUP;

```

## 20、统计库存量前三名的图书
```bash
SELECT *
FROM books
ORDER BY num DESC
LIMIT 0,3;

```

## 21、找出最早出版的一本书
```bash
SELECT *
FROM books
ORDER BY pubdate ASC
LIMIT 0,1;

```

## 22、找出novel中价格最高的一本书
```bash
SELECT *
FROM books
WHERE note='novel'
ORDER BY price DESC
LIMIT 0,1;
```

## 23、找出书名中字数最多的一本书，不含空格
```bash
SELECT REPLACE(NAME,' ','')
FROM books

SELECT *
FROM books
ORDER BY CHAR_LENGTH(REPLACE(NAME,' ','')) DESC
LIMIT 0,1;
```


# 数据处理之增删改的课后练习
# 练习1：
## 1. 创建数据库dbtest11
```bash
CREATE DATABASE IF NOT EXISTS dbtest11 CHARACTER SET 'utf8';
```

# 2. 运行以下脚本创建表my_employees
```bash
USE dbtest11;

CREATE TABLE my_employees(
	id INT(10),
	first_name VARCHAR(10),
	last_name VARCHAR(10),
	userid VARCHAR(10),
	salary DOUBLE(10,2)
);

CREATE TABLE users(
	id INT,
	userid VARCHAR(10),
	department_id INT
);
```

## 3.显示表my_employees的结构
```bash
DESC my_employees;

DESC users;
```

## 4.向my_employees表中插入下列数据
```bash
/*ID	FIRST_NAME	LAST_NAME	USERID		SALARY
1	patel		Ralph		Rpatel		895
2	Dancs		Betty		Bdancs		860
3	Biri		Ben		Bbiri		1100
4	Newman		Chad		Cnewman		750
5	Ropeburn	Audrey		Aropebur	1550
*/
INSERT INTO my_employees
VALUES(1,'patel','Ralph','Rpatel',895);

INSERT INTO my_employees VALUES
(2,'Dancs','Betty','Bdancs',860),
(3,'Biri','Ben','Bbiri',1100),
(4,'Newman','Chad','Cnewman',750),
(5,'Ropeburn','Audrey','Aropebur',1550);

SELECT * FROM my_employees;

DELETE FROM my_employees;

#方式2：
INSERT INTO my_employees
SELECT 1,'patel','Ralph','Rpatel',895 UNION ALL
SELECT 2,'Dancs','Betty','Bdancs',860 UNION ALL
SELECT 3,'Biri','Ben','Bbiri',1100 UNION ALL
SELECT 4,'Newman','Chad','Cnewman',750 UNION ALL
SELECT 5,'Ropeburn','Audrey','Aropebur',1550;
```

## 5.向users表中插入数据
```bash
/*1	Rpatel		10
2	Bdancs		10
3	Bbiri		20
4	Cnewman		30
5
	Aropebur	40
*/	
INSERT INTO users VALUES
(1,'Rpatel',10),
(2,'Bdancs',10),
(3,'Bbiri',20),
(4,'Cnewman',30),
(5,'Aropebur',40)

SELECT * FROM users;
```

## 6. 将3号员工的last_name修改为“drelxer”
```bash
UPDATE my_employees
SET last_name='drelxer'
WHERE id =3;

SELECT * FROM my_employees;
```

## 7. 将所有工资少于900的员工的工资修改为1000
```bash
UPDATE my_employees
SET salary =1000
WHERE salary <900;
```

## 8. 将userid为Bbiri的users表和my_employees表的记录全部删除
```bash
#方式1：
DELETE FROM my_employees
WHERE userid = 'Bbiri';

DELETE FROM users
WHERE userid = 'Bbiri';

#方式2：

DELETE m,u
FROM my_employees m
JOIN users u
ON m.userid = u.userid
WHERE m.userid = 'Bbiri';

SELECT * FROM my_employees;
SELECT * FROM users;
```

## 9. 删除my_employees、users表所有数据
```bash
DELETE FROM my_employees;
DELETE FROM users;
```

## 10. 检查所作的修正
```bash
SELECT * FROM my_employees;
SELECT * FROM users;
```

## 11. 清空表my_employees
```bash
TRUNCATE TABLE my_employees;
```
# 练习2：
## 1. 使用现有数据库dbtest11
```bash
USE dbtest11;
```
## 2. 创建表格pet
```bash
CREATE TABLE pet(
NAME VARCHAR(20),
OWNER VARCHAR(20),
species VARCHAR(20),
sex CHAR(1),
birth YEAR,
death YEAR
);

DESC pet;
```
## 3. 添加记录
```bash
INSERT INTO pet VALUES
('Fluffy','harold','Cat','f','2003','2010'),
('Claws','gwen','Cat','m','2004',NULL),
('Buffy',NULL,'Dog','f','2009',NULL),
('Fang','benny','Dog','m','2000',NULL),
('bowser','diane','Dog','m','2003','2009'),
('Chirpy',NULL,'Bird','f','2008',NULL);

SELECT *
FROM pet;
```
## 4. 添加字段:主人的生日owner_birth DATE类型。
```bash
ALTER TABLE pet 
ADD COLUMN owner_birth DATE;
```

## 5. 将名称为Claws的猫的主人改为kevin
```bash
UPDATE pet
SET `owner`='kevin'
WHERE `name`='Claws' AND species ='Cat';
```

## 6. 将没有死的狗的主人改为duck
```bash
UPDATE pet 
SET OWNER ='duck'
WHERE death IS NULL AND species ='dog'
```

## 7. 查询没有主人的宠物的名字；
```bash
SELECT NAME
FROM pet
WHERE OWNER IS NULL;
```

## 8. 查询已经死了的cat的姓名，主人，以及去世时间；
```bash
SELECT NAME,OWNER,death
FROM pet
WHERE death IS NOT NULL;
```


## 9. 删除已经死亡的狗
```bash
DELETE FROM pet
WHERE death IS NOT NULL 
AND species = 'Dog';
```

## 10. 查询所有宠物信息
```bash
SELECT * 
FROM pet;
```

# 练习3：
## 1. 使用已有的数据库dbtest11
```bash
USE dbtest11;
```

## 2. 创建表employee，并添加记录
```bash
CREATE TABLE employee(
id INT,
NAME VARCHAR(15),
sex CHAR(1),
tel VARCHAR(25),
addr VARCHAR(35),
salary DOUBLE(10,2)

);

INSERT INTO employee VALUES
(10001,'张一一','男','13456789000','山东青岛',1001.58),
(10002,'刘小红','女','13454319000','河北保定',1201.21),
(10003,'李四','男','0751-1234567','广东佛山',1004.11),
(10004,'刘小强','男','0755-5555555','广东深圳',1501.23),
(10005,'王艳','男','020-1232133','广东广州',1405.16);

SELECT * FROM employee;
```

## 3. 查询出薪资在1200~1300之间的员工信息。

```bash
SELECT *
FROM employee
WHERE salary BETWEEN 1200 AND 1300;
```
## 4. 查询出姓“刘”的员工的工号，姓名，家庭住址。
```bash
SELECT id,NAME,addr
FROM employee
WHERE NAME LIKE '刘%';
```

## 5. 将“李四”的家庭住址改为“广东韶关”
```bash
UPDATE employee
SET addr = '广东韶关'
WHERE NAME = '李四';
```

## 6. 查询出名字中带“小”的员工
```bash
SELECT *
FROM employee
WHERE NAME LIKE '%小%';
```