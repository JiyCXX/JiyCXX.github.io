---
title: MySql 创建和管理表语句 学习笔记
#文章创建日期
date:  2018-12-14 8:00:00
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
cover: /img/background/background.jpg
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