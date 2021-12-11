---
title: MySql 语句 学习笔记
#文章创建日期
date:  2018-12-11 8:00:00
#文章分类
categories: MySql
#文章关键字
keywords: MySql
#文章描述
description: MySql 笔记
#文章顶部图片
top_img: 
#文章缩略图
cover: /img/background/background.jpg
---

# MySql 认真学习一下 记录一下学习的语句练习

``` bash
DESC departments;

SELECT * FROM employees 
WHERE last_name ='king';
```

## 单表练习
### 查询员工12个月工资总和和奖金 别名annual aslary
``` bash
SELECT employee_id,last_name,salary *12*(1+IFNULL(commission_pct,0)) "annual aslary"
FROM employees;
``` 

### 查询员工表中去除重复job_id后的数据
``` bash
SELECT  DISTINCT job_id FROM employees;
``` 

### 查询工资大于12000员工的名和工资
``` bash
SELECT last_name, salary FROM employees 
WHERE salary >12000;
``` 
### 查询员工工号为176的姓名和部门号
``` bash
SELECT last_name,department_id FROM employees
WHERE employee_id =176;
``` 

### 显示表departments 的结构 并查询其中全部数据
``` bash
DESC departments;
SELECT *FROM departments;
``` 

### 查询员工的姓名部门号 年薪 按年薪降序 姓名升序显示
``` bash
SELECT	last_name, department_id, salary * 12 annual_sal
FROM employees
ORDER BY annual_sal DESC ,last_name ASC;
``` 

### 选择工资不在8000到17000的员工的姓名和工资 按工资排序 显示21到40的数据
``` bash
SELECT last_name,salary 
FROM employees 
WHERE salary NOT BETWEEN 8000 AND 17000
ORDER BY salary DESC
LIMIT 20,20;
``` 

### 查询邮箱中包含e的员工的信息 并先按邮箱的字节数降序 再按部门号升序
``` bash
SELECT employee_id, last_name,email,department_id
FROM employees
WHERE email LIKE '%e%'
ORDER BY LENGTH(email) DESC, department_id ASC;
``` 

## 多表练习

### 显示所有员工的姓名部门号和部门名称
``` bash
SELECT e.last_name,e.employee_id, d.department_id, d.department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id`=d.department_id
``` 

### 查询90号部门员工的job_id 和90号部门的location_id
``` bash
SELECT e.job_id , d.location_id
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE d.`department_id`=90;
``` 

### 查询所有奖金的员工的last_name， department_name location_id city
``` bash
SELECT e.last_name ,e.`commission_pct`, d.department_name , d.location_id , l.city
FROM employees e LEFT JOIN departments d
ON e.`department_id`=d.`department_id`
LEFT JOIN locations l
ON d.`location_id`=l.`location_id`
WHERE e.`commission_pct` IS NOT NULL;
``` 

### 选择city 在toronto工作的员工的last_name job_id department_id department_name
``` bash
SELECT e.last_name,e.job_id,e.department_id,d.department_name,l.`city`
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`
JOIN locations l
ON d.`location_id`=l.`location_id`
WHERE l.`city`='toronto';
``` 

### 查询员工所在的部门名称 部门地址 姓名工作工资 其中员工所在的部门的部门名称为Executive
``` bash
SELECT d.department_name,l.street_address,e.last_name,e.job_id,e.salary
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`
JOIN locations l
ON d.`location_id`=l.`location_id`
WHERE d.`department_name` = 'Executive';
``` 

### 选择指定员工的姓名 员工号 以及他的管理的姓名 员工号
``` bash
SELECT emp.last_name "employee", emp.employee_id "Emp#",mgr.last_name "manager",mgr.employee_id "Mgr#"
FROM employees emp LEFT JOIN employees mgr
ON emp.manager_id =mgr.employee_id;
``` 

### 查询那些员工没有部门
``` bash
SELECT e.`employee_id`,d.`department_id`,d.`department_name`
FROM  departments d LEFT JOIN  employees e 
ON e.`department_id`=d.`department_id`
WHERE e.`department_id` IS NULL;
``` 

### 查询那个城市没有部门
``` bash
SELECT l.location_id, l.city,d.`department_id`
FROM locations l LEFT JOIN departments d
ON l.`location_id`=d.`location_id`
WHERE d.`department_id` IS NULL;
``` 

### 查询部门名为sales或 it 的员工的信息
``` bash
SELECT e.`employee_id`,e.last_name,e.`department_id`,d.`department_name`
FROM departments d JOIN employees e
ON d.`department_id`=e.`department_id`
WHERE d.`department_name` IN('sales','it');
```

### 查看表结构 
``` 
DESC locations;
`employees`
DESC departments;
``` 

### 面试 分数降序
``` bash
SELECT salary 
FROM employees 
WHERE salary >6000
ORDER BY salary DESC;
``` 

### 平均分
``` bash
SELECT AVG(salary)
FROM employees
```
``` bash 
/*
学生表 S(学号 S#,姓名 Sn,所在系 SD,年龄 SA)
课程表 C(课程号 C#,课程名称 CN，先修课号 PC#)
学生选课表 SC(学号 S#，课程号 C#,成绩G)
问题一：画出该数据库系统的实体联系图（E-R图），注明联系类型
问题二：列出选课超过3门的学生学号，其所选课程数量及平均成绩
*/
SELECT CONNECTION_ID()
FROM DUAL
``` 
### 显示系统的时间
``` bash
SELECT CURDATE(),CURTIME(),SYSDATE() ,NOW(),LOCALTIME();
FROM DUAL;
``` 

### 查询员工工号 姓名 工资以及工资提高百分比20后的结果
``` bash
SELECT employee_id, last_name, salary, salary*(1+0.2)"new salary"
FROM employees;
``` 

### 将员工的姓名按首字母排序 并写出姓名长度
``` bash
SELECT last_name,LENGTH(last_name)
FROM employees
ORDER BY last_name ASC;
``` 

### 查询员工的id last_name salary 作为一列输出 别名为out_put
``` bash
SELECT CONCAT (employee_id ,'  ',last_name,'  ', salary)"OUT_PUT"
FROM employees;
``` 

### 查询公司各员工工作的年数，工作的天数，按工作年数降序
``` bash
SELECT employee_id,ROUND(DATEDIFF(NOW(),hire_date)/365) "worked_years",DATEDIFF(NOW(),hire_date)"worked_days"
FROM employees
ORDER BY worked_years DESC;
``` 

### 查询员工姓名 hire date department id 满足一下条件 
### 雇佣时间在97年之后 id 为80 或90或110 commission_pct不为空
``` bash
SELECT last_name,hire_date,department_id
FROM employees
WHERE department_id IN(80,90,110)
AND commission_pct IS NOT NULL
#and hire_date >='1997-01-01';隐式转换
#显示转换
AND hire_date >= STR_TO_DATE('1997-01-01','%Y-%m-%d')
``` 

### 查询公司中入职超过一万天的员工的姓名 入职时间
``` bash
SELECT last_name,hire_date
FROM employees
WHERE DATEDIFF(CURDATE(),hire_date) >=10000;	
``` 

### 输出指定结果
``` bash
SELECT CONCAT(last_name,' earns ',salary,' monthly but wants ',salary*3 )"dream salary"
FROM employees;
``` 

### 使用case when
``` bash
SELECT last_name "Last_name",job_id "Job_id",CASE job_id WHEN 'AD_PRES' THEN 'A'
                                                         WHEN 'ST_MAN' THEN 'B'
                                                         WHEN 'IT_PROG' THEN 'C'
                                                         WHEN 'SA_REP' THEN 'D'
                                                         WHEN 'ST_CLERK' THEN 'E' 
                                                         ELSE 'undefined' END "Grade"
FROM employees;
``` 

## 多行  聚合函数

### 查询各个部门的平均工资 最高工资
``` bash
SELECT department_id, AVG(salary),SUM(salary),MAX(salary)
FROM employees
GROUP BY department_id;

SELECT salary
FROM employees
WHERE department_id =20;
``` 

### 查询每个工种的平均工资
``` bash
SELECT AVG(salary)
FROM employees
GROUP BY job_id
``` 
### 查询各个job_id的员工工资的最大值 最小值 平均值 总和
``` bash
SELECT job_id ,MAX(salary) sala_max,MIN(salary) sala_min, AVG(salary) sala_avg, SUM(salary) sala_sum
FROM employees
GROUP BY job_id;
``` 

### 选着具有各个job_id的员工的人数
``` bash
SELECT job_id,COUNT(*)
FROM employees
GROUP BY job_id;
``` 

### 查询员工工资最高工资和最低工资的差距
``` bash
SELECT MAX(salary)-MIN(salary) "diffrence"
FROM employees
``` 
### 查询各个管理者手下员工的最低工资 其中最低工资不能低于6000没有管理者不计算在内
``` bash
SELECT manager_id,MIN(salary) sala_min
FROM employees
WHERE manager_id IS NOT NULL
GROUP BY manager_id
HAVING sala_min >=6000;
``` 
### 查询所有部门的名字 location_id 员工数量和平均工资 并按平均工资降序
``` bash
SELECT  d.department_name, d.location_id,COUNT(employee_id), AVG(e.salary) "sala_avg"
FROM departments d LEFT JOIN employees e
ON d.`department_id`=e.`department_id`
GROUP BY d.department_name, d.location_id
ORDER BY sala_avg DESC;
``` 

### 查询每个工种 每个部门的部门名 工种名和最低工资
``` bash
SELECT  d.department_name, e.job_id,MIN(e.salary)
FROM departments d LEFT JOIN employees e
ON d.`department_id`=e.`department_id`
GROUP BY d.department_name, e.job_id;
``` 

### 查询和 zlotkey相同部门的员工的姓名和工资
``` bash
SELECT last_name,salary
FROM employees
WHERE department_id =( SELECT department_id
		       FROM employees
		       WHERE last_name ='Zlotkey'                
                     );
``` 

### 查询工资比公司平均工资高的员工的工号 姓名和工资
``` bash
SELECT employee_id,last_name,salary
FROM employees
WHERE salary > (SELECT AVG(salary)
		FROM employees		
		);
``` 
		
### 选着工资大于所有JOB_ID='SA_MAN'的员工的工资的员工的姓名 job_id salary
``` bash
SELECT last_name,job_id,salary
FROM employees
WHERE salary > ALL(
		SELECT salary
		FROM employees
		WHERE job_id='SA_MAN'
		);
``` 
### 查询  和姓名中包含字母u的员工在相同部门的   员工的工号和姓名
``` bash
SELECT employee_id, last_name
FROM employees e
WHERE department_id IN (
			SELECT DISTINCT department_id
			FROM employees 
			WHERE last_name LIKE '%u%'
			);
```

### 查询在部门location_id为1700的部门工作的员工的员工号
``` bash
SELECT employee_id
FROM employees
WHERE department_id IN(
		SELECT department_id
		FROM departments
		WHERE location_id =1700
		   );	
```		

### 查询管理者是King的员工的 工资 姓名
``` bash
SELECT last_name,salary,manager_id
FROM employees
WHERE manager_id IN (
			SELECT employee_id
			FROM employees
			WHERE last_name ='King'
			);
```
			
### 查询工资最低的员工的信息 
``` bash
SELECT last_name, salary
FROM employees
WHERE salary <=(
		SELECT MIN(salary)
		FROM employees
		);
```

### 查询平均工资最低的部门信息
``` bash
#方式一
SELECT *
FROM departments
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)=(
					SELECT MIN(avg_salary) 
					FROM (
						SELECT AVG(salary)"avg_salary"
						FROM employees
						GROUP BY department_id
						)t_dept_avg_sala
					    )
			);
			
#方式二
SELECT *
FROM departments
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)<= ALL(
						SELECT AVG(salary)
						FROM employees
						GROUP BY department_id
						)
			);

#方式三 limt 
SELECT *
FROM departments
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)=(
						SELECT AVG(salary) avg_sala
						FROM employees
						GROUP BY department_id
						ORDER BY avg_sala ASC
						LIMIT 0,1
					   )

			);

#方式 四
SELECT d.* 
FROM departments d,(
		  SELECT department_id,AVG(salary) avg_sala
		  FROM employees
		  GROUP BY department_id
		  ORDER BY avg_sala ASC
		  LIMIT 1
		  )t_emp_avg_sala
WHERE d.`department_id`=t_emp_avg_sala.department_id

```

### 查询平均工资最低的部门信息和该部门的平均工资 相关子查询
``` bash
#方式一
SELECT d.*,(SELECT AVG(salary) FROM employees WHERE department_id = d.`department_id`) avg_salary
FROM departments d
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)=(
					SELECT MIN(avg_salary) 
					FROM (
						SELECT AVG(salary)"avg_salary"
						FROM employees
						GROUP BY department_id
						)t_dept_avg_sala
					    )
			);

#方式二
SELECT d.*,(SELECT AVG(salary) FROM employees WHERE department_id = d.`department_id`) avg_salary
FROM departments d
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)<= ALL(
						SELECT AVG(salary)
						FROM employees
						GROUP BY department_id
						)
			);

#方式三 limt 
SELECT d.*,(SELECT AVG(salary) FROM employees WHERE department_id = d.`department_id`) avg_salary
FROM departments d
WHERE department_id = (
			SELECT department_id
			FROM employees
			GROUP BY department_id
			HAVING AVG(salary)=(
						SELECT AVG(salary) avg_sala
						FROM employees
						GROUP BY department_id
						ORDER BY avg_sala ASC
						LIMIT 0,1
					   )

			);

#方式 四
SELECT d.* ,(SELECT AVG(salary) FROM employees WHERE department_id = d.`department_id`) avg_salary
FROM departments d,(
		  SELECT department_id,AVG(salary) avg_sala
		  FROM employees
		  GROUP BY department_id
		  ORDER BY avg_sala ASC
		  LIMIT 1
		  )t_emp_avg_sala
WHERE d.`department_id`=t_emp_avg_sala.department_id
```