# 目录
1. [SQL 语言的分类](#sql-语言的分类)<br>
   1.1 [DDL：数据定义语言](#ddl数据定义语言)  
   1.2 [DML：数据操作语言](#dml数据操作语言)  
   1.3 [DCL：数据控制语言](#dcl数据控制语言)  

2. [SQL 语言的规则和规范](#sql-语言的规则和规范)<br>
   2.1 [分号结尾](#分号结尾)  
   2.2 [大小写问题](#大小写问题)  
   2.3 [命名规则（暂时了解）](#命名规则暂时了解)  
   2.4 [注释](#注释)  
   2.5 [数据导入](#数据导入)  

3. [基本的 SELECT 语句](#基本的-select-语句)<br>
   3.1 [SELECT...](#select)  
   3.2 [SELECT... FROM](#select-from)  
   3.3 [列的别名](#列的别名)  
   3.4 [去除重复行](#去除重复行)  
   3.5 [空值 NULL](#空值-null)  
   3.6 [着重号](#着重号)  
   3.7 [查询常数](#查询常数)  

4. [显示表的结构](#显示表的结构)

5. [过滤数据](#过滤数据)

6. [运算符](#运算符)<br>
   6.1 [算术运算符](#算术运算符)  
   6.2 [比较运算符](#比较运算符)  
   6.3 [逻辑运算符](#逻辑运算符)  
   6.4 [位运算符](#位运算符)  
   6.5 [运算符的优先级](#运算符的优先级)  
<br><br>
   
#  SQL语言的分类
##  1. DDL：数据定义语言
DDL (data definition language)
定义了数据库、表、视图、索引等数据库对象

关键字：`CREATE` /  `ALTER` / `DROP` / `RENAME` / `TRUNCATE`

代表：创建 / 修改 / 删除 / 重命名 / 清空（清空表内容，表结构保留）

## 2. DML：数据操作语言
DML (data manipulation language)

对数据库记录进行==增删改查==，使用频率很高

关键字：`INSERT` / `DELETE` / `UPDATE` / `SELECT`

## 3. DCL：数据控制语言
DCL (data control language)

定义了数据库、表、字段、用户的访问权限和安全级别

关键字：`GRANT` / `REVOKE` / `COMMIT` / `ROLLBACK` / `SAVEPOINT`

代表：授予用户权限 / 撤销权限 / 提交事务保存修改 / 回滚 / 创建标记点，允许部分回滚
<br><br>

# SQL语言的规则和规范
##  1. 分号结尾
每句话以==分号结尾==，一句话太长可以换行。

## 2. 大小写问题
Windows系统本身就不区分大小写，因此在Windows环境下MYSQL大小写不敏感，但在Linux环境下大小写敏感。
(Oracle在Windows下是大小写敏感的，更严谨。SQL语言本身区分大小写，只是不同的DBMS严谨性有区别)

*我们总结出了一个规范：
数据库名、表名、字段名和一系列别名用小写；
SQL 关键字、函数名、绑定变量等用大写。*

## 3. 命名规则（暂时了解）
- *字符数限制*：数据库、表名不超过30个字符，变量名不超过29个字符，且只能包含 A–Z, a–z, 0–9, _共63个字符
- *不含空格*：数据库名、表名、字段名等对象名中间不要包含空格
- *不重名*：同一个MySQL软件中，数据库不能重名；同一个库中，表不能重名；同一个表中，字段不能重名
- *尽量不用关键字*：命名不能和关键字、数据库系统或常用方法同名。如果坚持使用，要用==着重号==（`）引起来，在键盘上，着重号在1的左边
- *字段名和类型的一致性*：命名字段并为其指定数据类型时要保证一致性。假如数据类型在一个表里是整数，那在另一个表里可就别变成字符型了

##  4. 注释
可以使用如下格式的注释：

单行注释：#注释文字(MySQL数据库特有的方式)

单行注释：-- 注释文字(--后面必须包含一个空格)

多行注释：/* 注释文字 */

##  5. 数据导入
法一. 在命令行登录MYSQL（或者在navicat中选工具--命令行界面）使用`source`指令导入：`mysql> source d:\mysqldb.sql`

法二. 在navicat图形化界面选文件--打开外部文件--查询，导入sql文件进行查询

法三：redis中直接把电脑里的文件拖到数据库里
<br><br>

# 基本的SELECT语句
##  1. SELECT...
可以用来检测数据库连接是否正常

```sql
SELECT 1; # 直接返回运算结果
SELECT 9/2 (FROM DUAL); # DUAL是伪表
```
关于`DUAL`伪表的解释：
+ 在SQL92标准中，每个SELECT语句必须指定一个FROM子句。因此，当SELECT不需要从实际表中选择数据，而只是返回某个常量或运算的结果集时，DUAL 就派上用场了。
+ 一般用于Oracle中，MYSQL中没有该标准，可以不写FROM DUAL。

## 2. SELECT... FROM
通配符 `*`：表示查询所有列
```sql
SELECT * FROM employees;
```
一般情况下：

```sql
SELECT 要查询的列 FROM 表;
```
## 3. 列的别名
结果集：查询结果的集合
起别名的例子：

```sql
SELECT employee_id id, last_name AS name, salary * 12 "annual salary"
FROM employees;
```
起别名的三种方式：
 1. 列名空格别名
 2. 列名`AS`别名（AS关键字可以省略，省略后就是1或3）
 3. 列名空格"别名"（加引号可以看得更清楚，特别地，当别名中含空格时，必须使用双引号）

建议别名简短，见名知意

## 4. 去除重复行
 （1）默认情况下，查询会返回全部行，包括重复行。
 （2）用`DISTINCT`关键字去除重复行：

例如，想看员工分布在哪些部门里，默认情况下一个部门的所有记录都会返回，看起来很乱，所以进行去重，每个部门只返回一次。

```sql
SELECT DISTINCT department_id FROM employees;
```
其中NULL代表该员工不在部门里。

 （3）DISTINCT放在所有列名前面，可以联合去重，但实际意义不大：

```sql
SELECT DISTINCT department_id, salary FROM employees;
```
## 5. 空值NULL
（1）空值不等同于0或空字符串
（2）==空值参与运算，结果都为NULL==

```sql
SELECT employee_id id, salary "月工资", salary * (1 + commission_pct) * 12 " 年工资", commission_pct FROM employees;
```
发现只要commission_pct为NULL的，年工资也为NULL。

（3）解决办法：引入`IFNULL`

```sql
SELECT employee_id id, salary "月工资", salary * (1 + IFNULL(commission_pct, 0)) * 12 " 年工资", commission_pct FROM employees;
```
其中`IFNULL(commission_pct, 0)`意思是如果commission_pct是NULL，就用0替换。
 
 ## 6. 着重号
 例如，有一个表名叫order，和关键字ORDER重名了。

```sql
 SELECT * FROM order;
```
这样运行会报错，识别为了关键字，而非表名。<br>
应该加上着重号（`）：
```sql
 SELECT * FROM `order`;
```
怎么知道是不是关键字？一般关键字会有一种特定的颜色。比如Navicat，关键字是蓝色，很好区分开来。

## 7. 查询常数
在 SELECT 查询结果中增加一列固定的常数列，这列的取值是我们指定的，而不是从数据表中动态取出的。<br>
用途：如果我们想整合不同的数据源，用常数列作为这个表的标记，就需要查询常数。<br>
例如：
```sql
SELECT '尚硅谷' AS corporation, last_name FROM employees;
```
除了查询last_name，还新增了一个常数列，这一列的值全都是“尚硅谷”，给列取了个别名叫corporation。
<br><br>
 
 # 显示表的结构
 
 `DESCRIBE`或者`DESC`

```sql
DESCRIBE employees;
```
```mysql
mysql> desc employees;
+----------------+-------------+------+-----+---------+-------+
| Field          | Type        | Null | Key | Default | Extra |
+----------------+-------------+------+-----+---------+-------+
| employee_id    | int(6)      | NO   | PRI | 0       |       |
| first_name     | varchar(20) | YES  |     | NULL    |       |
| last_name      | varchar(25) | NO   |     | NULL    |       |
| email          | varchar(25) | NO   | UNI | NULL    |       |
| phone_number   | varchar(20) | YES  |     | NULL    |       |
| hire_date      | date        | NO   |     | NULL    |       |
| job_id         | varchar(10) | NO   | MUL | NULL    |       |
| salary         | double(8,2) | YES  |     | NULL    |       |
| commission_pct | double(2,2) | YES  |     | NULL    |       |
| manager_id     | int(6)      | YES  | MUL | NULL    |       |
| department_id  | int(4)      | YES  | MUL | NULL    |       |
+----------------+-------------+------+-----+---------+-------+
11 rows in set (0.00 sec)
```
其中，各个字段的含义分别解释如下：

- Field：字段名称
- Type：字段数据类型
- Null：表示该列是否可以存储NULL值
- Key：表示该列是否已编制索引。PRI表示该列是表主键的一部分；UNI表示该列是UNIQUE索引的一部分；MUL表示在列中某个给定值允许出现多次。
- Default：表示该列是否有默认值，如果有，值是多少。
- Extra：表示可以获取的与给定列有关的附加信息，例如AUTO_INCREMENT等。
`AUTO_INCREMENT`：给表中的**主键**自动生成递增值，即自增列，主键通常是id，整数类型，默认从1开始，步长为1。
 <br><br>

# 过滤数据

只想返回满足*某些特定条件*的数据，用`where`关键字进行过滤：
`SELECT... FROM... WHERE...`，在FROM后面加上条件即可

```sql
SELECT * FROM employees WHERE department_id = 90;
```
<br><br>

# 运算符
## 1. 算术运算符
（1）加（+）减（-）
+ 加减一个浮点数，结果也是浮点数；
+ 在Java中，+的左右两边只要有一边字符串（" "），就表示*字符串的拼接*。在MySQL中，+只表示数值相加。
+ ==如果有一边为字符串，先尝试将其转成数值（隐式转换），如果转失败，比如是个字母，就按0计算。==（MySQL中字符串拼接要使用字符串函数CONCAT()实现）

```sql
SELECT 100 + '1' FROM DUAL; # '1'隐式转换为数值1，结果为101
SELECT 100 + 'a' FROM DUAL; # 'a'看成数值0计算，结果为100
SELECT 100 + NULL FROM DUAL; # 空值参与运算，结果为0
```
（2）乘（*）除（/ 或 DIV）
+ 乘除一个浮点数，结果一定是浮点数；
+ 除不尽时，保留到小数点后四位；
+ MYSQL除以0结果为NULL；

```sql
SELECT 100 / 3; # 33.3333
SELECT 100 DIV 3; # 33，DIV截断
```
（3）求模（%或MOD）

```sql
#筛选出employee_id是偶数的员工
SELECT * FROM employees
WHERE employee_id MOD 2 = 0;
```
结果和被模数符号一致
## 2. 比较运算符
结果要么是0（不相等），要么是1（相等）
### 第一扒：=, <=>, <>(!=), <, <=, >,  >=

#### （1）= 
+ 注意SQL中=表示比较，其它语言中==才是比较，=是赋值；
+ 左右两边都为字符串，就比较字符串是否相同；
+ 有一边是字符串，另一边是数值，就要将字符串转化为数值。先尝试隐式转换为数值，转换失败（是字母）就按0计算；
+ NULL参与比较，结果为NULL。
```sql
SELECT NULL = '1'; 
SELECT NULL = NULL; # 结果都是NULL
```
+ = 无法与NULL比较，因为只要NULL参与比较，结果就为NULL，查询不到任何东西
```sql
SELECT employee_id,salary FROM employees WHERE commission_pct = NULL; # 啥都不会返回，错误写法
```
#### （2）<=> 安全等于
用于实现和NULL比较，对=进行改进（==为NULL而生==）
```sql
SELECT NULL <=> '1'; 
SELECT NULL <=> NULL; 
```
```sql
SELECT employee_id,salary FROM employees WHERE commission_pct <=> NULL;
```
### 第二扒：非符号类型

#### （1）为空运算符
`IS NULL / ISNULL() `

```sql
SELECT employee_id,salary, commission_pct FROM employees WHERE commission_pct IS NULL;
SELECT employee_id,salary, commission_pct FROM employees WHERE ISNULL(commission_pct);
```
不为空运算符：`IS NOT NULL`
```sql
SELECT employee_id,salary, commission_pct FROM employees WHERE commission_pct IS NOT NULL;
```
*总结：
查询为空的三种方法：*
```sql
SELECT ... FROM ... WHERE ... IS NULL;
SELECT ... FROM ... WHERE ... <=> NULL;
SELECT ... FROM ... WHERE ISNULL(...);
```

*查询不为空的三种方法：*

```sql
SELECT ... FROM ... WHERE ... IS NOT NULL;
SELECT ... FROM ... WHERE NOT ... <=> NULL;
SELECT ... FROM ... WHERE NOT ISNULL(...);
```
#### （2）最大最小
`LEAST / GREATEST`

```sql
SELECT LEAST(first_name, last_name) FROM employees; # 字典序
```
只要有NULL参与，结果为NULL

#### （3）在 ... 区间内
`BETWEEN ... AND ...` （全闭区间）

```sql
SELECT employee_id, salary FROM employees WHERE salary BETWEEN 6000 AND 8000;
```
相当于：
```sql
SELECT employee_id, salary FROM employees WHERE salary >= 6000 && salary <= 8000;
```
#### （4）属于 / 不属于
`IN / NOT IN`

```sql
SELECT last_name, salary, department_id FROM employees WHERE department_id IN (10, 20, 30);
```
相当于：

```sql
SELECT last_name, salary, department_id FROM employees WHERE department_id = 10 OR department_id = 20 OR department_id = 30;
```
条件要写全，不能写成 OR 20 OR 30（这样所有非NULL记录都出来了）

#### （5）模糊匹配运算符
`LIKE`

> LIKE运算符通常使用如下**通配符**：
>  “%”：匹配0个或多个字符
>   “_”：只能匹配一个字符

1）名字里含a的：

```sql
SELECT last_name FROM employees WHERE last_name LIKE '%a%';
```

2）名字里含a和e的：

```sql
SELECT last_name FROM employees WHERE last_name LIKE '%a%e%' OR last_name LIKE '%e%a%';
```
3）名字的第三个字符是a：

```sql
SELECT last_name FROM employees WHERE last_name LIKE '__a%';
```
4）名字的第二个字符是 _ ， 第三个字符是a：

法一：==使用转义字符==：
```sql
SELECT last_name FROM employees WHERE last_name LIKE '_\_a%';
```
法二：（小众）`ESCAPE`
```sql
SELECT last_name FROM employees WHERE last_name LIKE ‘_$_a%‘ ESCAPE ‘$‘; # 跳过$，第二个字符是_
```
#### （6）正则表达式运算符
`REGEXP`运算符用来匹配字符串，语法格式为：`expr REGEXP 匹配条件`。如果expr满足匹配条件，返回1；如果不满足，则返回0。若expr或匹配条件任意一个为NULL，则结果为NULL。

REGEXP运算符在进行匹配时，常用的有下面几种通配符：
```
（1）‘^’匹配以该字符后面的字符开头的字符串。
（2）‘$’匹配以该字符前面的字符结尾的字符串。
（3）‘.’匹配任何一个单字符。
（4）“[...]”匹配在方括号内的任何字符。例如，“[abc]”匹配“a”或“b”或“c”。为了命名字符的范围，使用一个‘-’。“[a-z]”匹配任何字母，而“[0-9]”匹配任何数字。
（5）‘*’匹配零个或多个在它前面的字符。例如，“x*”匹配任何数量的‘x’字符，“[0-9]*”匹配任何数量的数字，而“*”匹配任何数量的任何字符。
```
## 3. 逻辑运算符
判断表达式真假，结果为1，0或NULL
四种：

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a1be8e5e718d46d685838011c5a26582.png)

AND优先级高于OR

异或：强调“异”，可以理解成A和B的真假性要不同，A、B只有一个成立时，返回真。==（A或B成立）==

## 4. 位运算符
用的不多，了解即可

“位”即为bit，在二进制上进行运算

（十进制-->二进制进行位运算-->十进制）

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/690c576df05d4e4da9b47d5067e3d15a.png)

**对象是两个数：**
（1）按位与
转化为二进制后，对应位置上的数值都为1，则返回1（类似“全真才为真”）
```sql
SELECT 1 & 10; # 结果为0
```
0001，0010，0011，0100，0101，0110，0111，1000，1001，1010

1转化为二进制：0001

10转化为二进制:1010

第一位到第四位上下比较，例如第一位分别是0和1，则返回0。

结果是0000，转化回十进制即为0。

（2）按位或
转化为二进制后，对应位置上的数值有一个为1，则返回1（类似“有真即为真”）

（3）按位异或
转化为二进制后，对应位置上的数值不同，则返回1；相同返回0。

**对象是某一个数：**
（4）按位取反运算符
转化为二进制后，每一位上的数值1变0，0变1

```sql
SELECT 10 & ~1;
```
按位取反运算符优先级更高，先对1（0001）按位取反得1110，再与10按位与得1010，即为10。

（5）按位左移
每一位上的数依次左移 n 位，相当于 × 2 的 n 次幂。

```sql
SELECT 4 << 2; # 结果为16
```
4 × 2 的 2 次幂 = 16

二进制转化为十进制的原理：最后一位 × 2 的 0 次幂，倒数第二位 × 2 的 1 次幂，以此类推最后相加
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f8f882351adb4f65b5b0cfd4b45a17ba.png)

（6）按位右移
每一位上的数依次左移一位，相当于➗2的 n 次幂。

## 5. 运算符的优先级
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3e4ddf68c4034fc481a43c52bfb83afe.png)
编号越大，优先级越高

---
