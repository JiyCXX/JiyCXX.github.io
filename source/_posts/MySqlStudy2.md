---
title: MySql 创建和管理表语句 学习笔记
#文章创建日期
date:  2018-12-11 8:00:00
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
# 创建和管理表的课后练习 
## 练习一
### 1 创建数据库test01_office 指明字符集为utf-8并在此数据库执行下列操作
```bash
CREATE DATABASE IF NOT EXISTS test01_office CHARACTER SET 'utf8';

USE test01_office;

#创建表 dept01
#字段  类型
#id     int7
#name   varchar25
CREATE TABLE IF NOT EXISTS dept01(
id INT(7),
`name` VARCHAR(25)
);  


SHOW TABLES
SHOW TABLES FROM test01_office
```

### 2将表departments中的数据插入新表dept02
```bash
CREATE TABLE dept02
AS
SELECT * 
FROM atguigudb.`departments`;

SELECT *
FROM dept02;
```

### 创建表emp01
```bash
#字段        类型
#id            int7
#first_name     varchar25
#last_nmae       varchar25
#dept_id  	int7


CREATE TABLE IF NOT EXISTS emp01(
id      INT(7),
first_name VARCHAR(25),
last_name VARCHAR(25),
dept_id INT(7)
);
```

### 将last_name 列长度增加到50
```bash
DESC emp01;

ALTER TABLE emp01
MODIFY last_name VARCHAR(50);
```

### 根据表employees创建emp02
```bash
CREATE TABLE emp02
AS 
SELECT *
FROM atguigudb.`employees`;

SELECT *FROM emp02;
```

### 删除表emp01
```bash
DROP TABLE emp01;
```

### 将表emp02重命名为emp01
```bash
#alter table emp02 rename to emp01

RENAME TABLE emp02 TO emp01;
```

### 在表dept02和emp01中添加新列test_column 并检查所作的操作
```bash
ALTER TABLE emp01 ADD test_column VARCHAR(10);

DESC emp01;

ALTER TABLE dept02 ADD test_column VARCHAR(10);

DESC dept02;
```

### 直接删除表emp01中的列department_id
```bash
ALTER TABLE emp01
DROP COLUMN department_id;
```
## 练习二

### 创建数据库test02_market
```bash
CREATE DATABASE IF NOT EXISTS test02_markets CHARACTER SET 'utf8'
USE test02_markets;

SHOW CREATE DATABASE test02_markets;
```

### 创建数据表customers
```bash
CREATE TABLE customers(
c_num INT,
c_name VARCHAR(50),
c_contacr VARCHAR(50),
c_city VARCHAR(50),
c_birth DATE
);

SHOW TABLES FROM test02_markets;
```

