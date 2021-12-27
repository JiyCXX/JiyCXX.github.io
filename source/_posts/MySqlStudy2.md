---
title: MySql 创建和管理表语句 学习笔记
#文章创建日期
date:  2018-12-12 7:30:30
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

### 修改字段名
```bash
ALTER TABLE customers
CHANGE c_contacr c_contact VARCHAR(50);

DESC customers;
```

### 将字段c_contact字段移动到c_birth字段后面
```bash
ALTER TABLE customers
MODIFY c_contact VARCHAR(50) AFTER c_birth;
```

### 将c_name 字段数据类型改为varchar70
```bash
ALTER TABLE customers
MODIFY c_name VARCHAR(70);


```

### 将c_contact字段改名为c_phone
```bash
ALTER TABLE customers
CHANGE c_contact c_phone VARCHAR(70);

ALTER TABLE customers 
MODIFY c_phone VARCHAR(50);
```

### 增加 c_gender char1 字段到c_name 后面
```bash
ALTER TABLE customers 
ADD c_gender CHAR(1) AFTER c_name;
```

### 将表名改为customers_info
```bash
RENAME TABLE customers TO customers_info;

DESC customers_info;
```

### 删除字段 c_city
```bash
ALTER TABLE customers_info
DROP COLUMN c_city;
```

## 练习三

### 创建数据库
```bash
CREATE DATABASE test02_company CHARACTER SET 'utf8';

#创建表offices
USE test02_company;

CREATE TABLE offices(
officeCode  INT,
city  VARCHAR(30),
address VARCHAR(50),
country VARCHAR(50),
postalCode VARCHAR(25)
);

SHOW TABLES FROM test02_company;
DESC offices;

#创建表 employees
CREATE TABLE employees(
empNum INT,
lastName VARCHAR(50),
firstNmae VARCHAR(50),
mobile VARCHAR(25),
`code` INT,
jobTitle VARCHAR(50),
birth DATE,
note VARCHAR(255),
sex VARCHAR(5)
);

```

### 将表employees中字段mobile修改到code后面
```bash
ALTER TABLE employees 
MODIFY mobile VARCHAR(20) AFTER `code`;
```

### 将表employees中字段birth 改名为birthday
```bash
ALTER TABLE employees
CHANGE birth birthday DATE;
```

### 修改sex字段 数据类型为	char1
```bash
ALTER TABLE employees
MODIFY sex CHAR(1);
```

### 删除字段note
```bash
ALTER TABLE employees
DROP COLUMN note;
```

### 增加字段名favoriate_activity 数据类型为varchar100
```bash
ALTER TABLE employees
ADD favoriate_activity VARCHAR(100);
```

### 将表employees 的名称改为employees_info
```bash
RENAME TABLE employees TO employees_info;
DESC employees_info;
```