[【篇一在这里，点击链接查看~】](https://blog.csdn.net/2503_90543222/article/details/149499217?ops_request_misc=elastic_search_misc&request_id=9e22f7bcca1847f2a5f94a9b8f05dcf8&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~ElasticSearch~search_v2-1-149499217-null-null.142%5Ev102%5Epc_search_result_base9&utm_term=sql%E8%B6%85%E5%85%A8%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E7%82%B9&spm=1018.2226.3001.4187)
@[TOC](目录)

# 排序与分页
## 1. 排序
（1）默认情况下，没有使用排序操作时，返回顺序按照添加的顺序展现。

（2）使用 `ORDER BY` 排序，默认升序。
在 ORDER BY...  后加 `ASC` 或 `DESC`，指明升序或降序，
分别是 ascending（升） 和 descending（降）的缩写。

升序写法：
```sql
SELECT last_name, salary FROM employees ORDER BY salary;
SELECT last_name, salary FROM employees ORDER BY salary ASC;
```

降序写法：
```sql
SELECT last_name, salary FROM employees ORDER BY salary DESC;
```

降序“民间写法”，加一个负号哈哈：
```sql
SELECT last_name, salary FROM employees ORDER BY -salary;
```

（3）顺序：WHERE 和 ORDER BY 同时出现时，*WHERE 永远紧跟在 FROM 后面*（“从哪里”）

```sql
SELECT last_name, salary, department_id FROM employees WHERE department_id IN (50, 60, 70) ORDER BY department_id DESC;
```

（4）列的别名可以用在 ORDER BY 中，但不能用在 WHERE 中：

```sql
SELECT last_name, salary * 12 annual_sal FROM employees ORDER BY annual_sal;
```

> 原理：
> 第一步：从哪个表里按什么条件筛选出哪一部分数据（最大程度过滤掉不需要的数据）；
> 第二步：筛选出的数据中需要哪几列，若需要起别名就在这里起个别名；
> 第三步：这些列里是否有需要排序的。
> 因此，起别名是在筛选条件之后的，WHERE 中也就自然不能用别名了。

（5）二级排序：

```sql
SELECT last_name, salary, department_id FROM employees ORDER BY department_id ASC, salary DESC;
```
department_id 是一级，salary是二级。

## 2. 分页
（1）分页需求与优点：

+ 查询结果太多，客户不需要全部显示，全部在一页查看起来很不方便，想要比较不同结果难翻到；
+ 第一次请求网络，只用加载出第一页的数据，点第二页时会再次请求网络。约束返回结果的数量可以减少数据表的网络传输量，同时提高效率。

（2）用`LIMIT`实现分页（分段显示）：

```sql
LIMIT 位置偏移量, 行数
```
位置偏移量：从哪里开始，默认为0，为0时可省略

（3）公式：每页显示pageSize条记录，此时显示第pageNo页
```sql
LIMIT (PageNo - 1) * pageSize, PageNo
```

（4）顺序：LIMIT 放在整个SELECT语句的**最后**！
```sql
SELECT last_name, salary, department_id FROM employees WHERE department_id IN (50, 60, 70) ORDER BY department_id DESC LIMIT 0, 10;
```

（5）应用：

1）显示部分数据：表里有107条数据，只想显示第32、33条数据，也可以用LIMIT

```sql
SELECT employee_id, last_name FROM employees LIMIT 31, 2;
```

2）显示最大最小值：

```sql
SELECT employee_id, last_name, salary FROM employees ORDER BY salary DESC LIMIT 0, 1;
```
相当于：
```sql
SELECT MIN(salary) AS least_salary FROM employees;
```

（6）MYSQL8.0 新特性：`LIMIT... OFFSET...`
例如，LIMIT 2 OFFSET 31 等同于 LIMIT 31, 2（反过来）
<br>
# 多表查询

多表查询，也称为关联查询，指两个或更多个表一起完成查询操作。

前提条件：表间有关系（一对一、一对多、多对多），有**关联字段**，这个关联字段可能建立了`外键`，也可能没有建立外键。

> 外键：比如员工表和部门表，关联字段为部门id。部门表里部门id是主键，我们声明员工表里的部门id为外键，保证员工表里的部门id必须在部门表里存在，相当于加了一个数据库级约束。

## 1. 一个案例引发的多表查询
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bc2fb2d38513435abdd77a96f364d018.png)
关联字段是 department_id 和 location_id ，
（1）想知道员工 'Abel' 在哪个城市工作，可以进行多表查询，我们想到最直接的方法就是查三次：

```sql
SELECT department_id FROM employees WHERE last_name = 'Abel'; # 结果为80
SELECT location_id FROM departments WHERE department_id = 80; #结果为2500
SELECT city FROM locations WHERE location_id = 2500;
```
分析：需要查询三次，即向服务器发送三次请求，效率很低。

（2）Q：能否把这三张表合并为一张表？
A：可以，但不好。比如有的员工没有隶属的部门，department_id 为NULL，若合并为一张表，后面部门表和位置表的字段全都为空，出现很多冗余字段，还有很多类似的情况，会造成更多**冗余**。冗余字段变多会造成**IO**（输入输出）变多，降低效率。

>`第三范式`：不存在传递依赖。合并为一张表就不满足第三范式了。

## 2. 多表查询的实现

> `笛卡尔积`（交叉连接，CROSS JOIN）：
> 一种数学运算，X集与Y集中的每个元素都组合一遍
> 比如X = (a, b, c), Y = (x, y)
> 笛卡尔积得到的组合就是(a, x), (a, y), (b, x), (b, y), (c, x), (c, y)

（1）笛卡尔积的错误场景：
```sql
SELECT first_name, department_name FROM employees, departments;
```
从结果中看到，每位员工与每个部门都交叉连接了一次，都有27条记录，有107位员工，所以一共查询到了2889条记录。

（2）多表查询的正确方式：需要==有连接条件==
```sql
SELECT first_name, department_name FROM employees, departments WHERE employees.department_id = departments.department_id;
```

[ ✔️] 结果有106条记录，少了一条，原因是 Kimberely 没有部门，部门id是NULL。这里先暂且忽略，后面会讲到。

```sql
SELECT first_name, department_name, employees.department_id FROM employees, departments WHERE employees.department_id = departments.department_id;
```
（3）如果查询语句中出现了多个表中都存在的字段，要指明是哪个表：

```sql
SELECT first_name, department_name, department_id FROM employees, departments WHERE employees.department_id = departments.department_id;
```
报错，department_id模糊；

正确写法：

```sql
SELECT employees.first_name, departments.department_name, employees.department_id FROM employees, departments WHERE employees.department_id = departments.department_id;
```
从sql优化的角度，==多表查询时，建议每个字段都指明其所在表==。

（4）多表查询语句过长时，为了增强可读性，可以给表起别名。

> 后面（逻辑顺序的后面，SELECT、WHERE中）再使用表名时，必须用别名，不能用原表名，否则报错。

```sql
SELECT e.first_name, d.department_name, e.department_id FROM employees e, departments d WHERE e.department_id = d.department_id;
```

## 3. 多表查询的分类
### 3.1 等值连接 & 非等值连接
等值连接：顾名思义，表间连接条件是等价表达式

### 3.2 自连接 & 非自连接
自连接：自己和自己连接。

> 例如要查询员工id、员工姓名、其管理者id、管理者姓名，就是员工表自连接。
> 管理者id可以在员工表中用员工id查，而*管理者也是员工，因此管理者姓名又要在员工表中用管理者id查*。我们可以创建employees表的两个别名，看成两个表。**一个表是员工，一个表是管理者。关联字段就是员工表的管理者id和管理者表的员工id。**（理解）

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f963100af3e9400f81731832b13d0ba1.png)
本质上是同一张表，用取别名的方式虚拟成两张表以代表不同的意义。

```sql
SELECT e.employee_id, e.last_name, m.employee_id, m.last_name
FROM employees e, employees m
WHERE e.manager_id = m.employee_id;
```
### 3.3 内连接 & 外连接
#### 3.3.1 概念
==一句话就是：内连接按短筒来，外连接按长筒来，短的补全为NULL（短筒就是匹配的行）==

（1）`内连接`：多表具有相同的列，则合并相同列相匹配的行，结果集不包含不匹配的行；

（2）`外连接`：结果集包含左表/右表中不匹配的行 ，称为`左/右外连接`。没有匹配的行时，结果表中相应的列为空(NULL)；

> 何时一定要用外连接？
> 出现“所有”这种字眼时，例如所有员工，如果员工表在左那么就用左外连接。

（3）如果是左外连接，则连接条件中左边的表称为主表，右边的表称为从表，右外连接反之。还有一种叫`满外连接`，左右表中不匹配的行都查出来了。

例如，查询员工表和部门表时，关联字段是部门id，记住，Kimberely 这个员工没有部门，这一列是NULL。

如果使用内连接：

```sql
SELECT e.first_name, d.department_name, d.department_id FROM employees e, departments d WHERE e.department_id = d.department_id;
```
结果集少一个员工，少的就是 Kimberely，因为员工表的部门id这一列他是NULL，但部门表没有一个部门id是NULL，不匹配，因此不返回他这一行。

如果使用外连接，就会返回这一不匹配的行。

#### 3.3.2 如何实现内外连接（重点）
##### SQL92
内连接同上
外连接：用 + 表示从表的位置

```sql
#左外连接
SELECT e.first_name, d.department_name, d.department_id FROM employees e, departments d WHERE e.department_id = d.department_id(+);
#右外连接
SELECT e.first_name, d.department_name, d.department_id FROM employees e, departments d WHERE e.department_id(+) = d.department_id;
```
MySQL不支持SQL92，且没有满连接。
##### SQL99
###### 基本语法
（1）内连接：`JOIN...ON...`

```sql
SELECT table1.column, table2.column, table3.column
FROM table1
JOIN table2 ON table1 和 table2 的连接条件
JOIN table3 ON table2 和 table3 的连接条件
```
（2）左外连接：`LEFT  JOIN...ON...`
（3）右外连接：`RIGHT JOIN...ON...`
（4）满外连接：`FULL JOIN...ON...`（MySQL不支持FULL JOIN，用UNION代替）

###### UNION的使用
顾名思义，UNION用来合并查询结果

```sql
SELECT column,... FROM table1
UNION (ALL)
SELECT column,... FROM table2
```

`UNION`：返回两个查询结果集的并集，去重
`UNION ALL`：同上，但不去重

在两者有重复部分时，需要进行去重，使用UNION；
无重复部分时，去不去重都一样，但从sql优化的角度，去重是需要时间的，会降低效率，因此使用UNION ALL更好。

###### 7种JOINS
![](https://i-blog.csdnimg.cn/direct/595e040233064dce84874811baa89f07.png)
```sql
# 中图：内连接 A∩B
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e JOIN departments d
ON e.department_id = d.department_id;
```

```sql
# 左上图：左外连接
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e LEFT JOIN departments d
ON e.department_id = d.department_id;
```

```sql
# 右上图：右外连接
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e RIGHT JOIN departments d
ON e.department_id = d.department_id;
```

```sql
# 左中图：A - A∩B
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e LEFT JOIN departments d
ON e.department_id = d.department_id
WHERE d.department_id IS NULL; # 去掉交集（从表关联字段为NULL）
```

```sql
# 右中图：B-A∩B
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e RIGHT JOIN departments d
ON e.department_id = d.department_id
WHERE e.department_id IS NULL;
```

```sql
# 左下图：满外连接
# 左中图 + 右上图  A∪B
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e LEFT JOIN departments d
ON e.department_id = d.department_id;
WHERE d.department_id IS NULL
UNION ALL  #没有去重操作，效率高
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e RIGHT JOIN departments d
ON e.department_id = d.department_id;
```

```sql
# 右下图
# 左中图 + 右中图  A∪B- A∩B 或者 (A -  A∩B) ∪ （B - A∩B）
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e LEFT JOIN departments d
ON e.department_id = d.department_id;
WHERE d.department_id IS NULL
UNION ALL
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e RIGHT JOIN departments d
ON e.department_id = d.department_id
WHERE e.department_id IS NULL;
```

后面复杂的直接拆解成前面几个小问题

###### SQL99新特性
（1）自然连接

 `NATURAL JOIN` 表示自然连接，相当于等值连接（自动查询两张连接表中所有相同的字段，然后进行等值连接）

```sql
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e NATURAL JOIN departments d;
```
（2）USING
`USING`指定了具体的相同的字段名称

```sql
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
USING (department_id);
```

/小结一下/
三种表连接的约束条件：WHERE ON USING 
推荐用ON

WHERE：

```sql
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id;
```

ON：只能和JOIN一起用

```sql
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e JOIN departments d
ON e.department_id = d.department_id;
```

USING：只能和JOIN一起用

```sql
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
USING (department_id);
```


---
博主加班持续更新中
欢迎指正~

该笔记由博主学习尚硅谷MySQL课程整理所得，特此鸣谢。
