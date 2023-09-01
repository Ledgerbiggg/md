## 库结构
```sh
# 查询所有数据库
SHOW DATABASES;

# 查询当前数据库
SELECT DATABASE();

# 创建数据库
CREATE DATABASE IF NOT EXISTS test CHARACTER SET utf8mb4;

# 删除数据库
DROP DATABASE IF EXISTS test

# 使用数据库
USE study;

```

## 表结构
```sh
# 展示数据库里面的所有表
SHOW TABLES

# 新建表
DROP TABLE IF EXISTS emploee;
CREATE TABLE emploee(
	ID INT COMMENT "员工编号",
	WORKID VARCHAR(255) COMMENT "员工编号",
	NAME VARCHAR(255) COMMENT "员工姓名",
	GENDER VARCHAR(1) COMMENT "姓名",
	AGE TINYINT UNSIGNED COMMENT "年纪",
	IDCARD VARCHAR(255) COMMENT "身份证号码",
	ENTRYDATE DATE COMMENT "入职时间"
	)ENGINE INNODB DEFAULT CHARSET = utf8mb4 COMMENT "练手2";

# 查询制定表的建表语句
SHOW CREATE TABLE TEST;

# 查询表的结构
DESC TEST;

```
## 列结构
```sh
# 改变一个列的结构
ALTER TABLE emploee2
MODIFY USERNAME VARCHAR(200) COMMENT "USERNAME";

# 增加一个列的结构
ALTER TABLE emploee
ADD COLUMN NIKCNAME VARCHAR(255) COMMENT "NICKNAME";

# 改变一个列的结构
ALTER TABLE emploee
CHANGE NIKCNAME USERNAME VARCHAR(300) COMMENT "USERNAME";

# 删除列
ALTER TABLE emploee2
DROP USERNAME;

# 改变表的名字
ALTER TABLE emploee
RENAME TO emploee2;

# 删除表
DROP TABLE emploee2;
```
## SML(增加,更新,删除)
```sh
# 增加
INSERT INTO emploee VALUES
(1,"123","LEDGER","男",15,"123456","2000-1-2"),
(1,"123","LEDGER","男",15,"123456","2000-1-2"),
(1,"123","LEDGER","男",15,"123456","2000-1-2"),
(1,"123","LEDGER","男",15,"123456","2000-1-2"),
(1,"123","LEDGER","男",15,"123456","2000-1-2"),
(1,"123","LEDGER","男",15,"123456","2000-1-2");

# 更新
UPDATE emploee 
SET NAME = "YB" , ENTRYDATE = "2010-12-12"
WHERE ID=1;

# 删除
DELETE FROM emploee WHERE NAME ="YB";
```
## DQL查询
```sh
# 简单查询
SELECT *
FROM emploee
WHERE ID=1;

SELECT DISTINCT GENDER KK
FROM emploee;

SELECT *
FROM emploee
WHERE NAME="LEDGER";

SELECT *
FROM emploee
WHERE AGE>10;


SELECT *
FROM emploee
WHERE AGE IS NOT NULL;


SELECT *
FROM emploee
WHERE NAME LIKE "L%"
```
## 聚合函数
* 查询结构要带有分组字段和聚合函数
```sh
SELECT GENDER, COUNT(*)
FROM emploee
WHERE AGE < 60
GROUP BY GENDER
```
## 分组
```sh
SELECT *
FROM emploee
WHERE AGE BETWEEN 20 AND 40
AND GENDER='男'
ORDER BY AGE,ENTRYDATE
LIMIT 5
```
## 排序
```sh
SELECT *
FROM emploee
WHERE AGE BETWEEN 20 AND 40
AND GENDER='男'
ORDER BY AGE,ENTRYDATE
LIMIT 5
```
## 分页查询
```sh
SELECT *
FROM emploee
WHERE AGE BETWEEN 20 AND 40
AND GENDER='男'
ORDER BY AGE,ENTRYDATE
LIMIT 5
```
## DCL(控制数据库的服务器)
```sh
# 查询用户
USE mysql

SHOW TABLES

SELECT *
FROM user

# 创建用户 在localhost主机可登录 用户名ledger,密码是LEDFER
CREATE USER 'ledger'@'localhost' IDENTIFIED BY 'LEDGER'

```

## 小结
```sh
# 结构顺序
SELECT 字段1,字段2,...(可能存在聚合函数)
FROM 表1 (LEFT / RIGHT) JOIN 表2 ON 多表的连接条件
(LEFT / RIGHT) JOIN 表2 ON 多表的连接条件2...
WHERE 不包含聚合函数的过滤条件
GROUP BY 分组字段1,分组字段2...
HAVING 包含聚合函数的过滤条件
ORDER BY 排序字段1,排序字段2...(ASC / DESC)
LIMIT 偏移量,条目数

# 执行顺序
# FROM
# (LEFT / RIGHT) 
# WHERE
# GROUP
# HAVING
# SELECT
# ORDER
# LIMIT
```

## 函数
```sh
字符串函数（String Functions）： 用于处理文本数据。

CONCAT(): 将多个字符串连接在一起。
SUBSTRING(): 提取子字符串。
UPPER(): 将字符串转换为大写。
LOWER(): 将字符串转换为小写。
LENGTH(): 获取字符串长度。
日期和时间函数（Date and Time Functions）： 用于处理日期和时间数据。

NOW(): 获取当前日期和时间。
DATE(): 提取日期部分。
TIME(): 提取时间部分。
YEAR(): 获取年份。
MONTH(): 获取月份。
条件函数（Conditional Functions）： 用于基于条件进行计算。

CASE: 执行条件语句。
类型转换函数（Type Conversion Functions）： 用于不同数据类型之间的转换。

CAST(): 将一个数据类型转换为另一个数据类型。
CONVERT(): 类似于 CAST()，在某些数据库系统中支持。
NULL 相关函数：

IS NULL: 判断是否为 NULL。
COALESCE(): 返回第一个非 NULL 值。
```
## 合并或者取交集
1. UNION: 已经提到过的关键字，用于合并两个或多个查询的结果集，并去除重复的行。

2. UNION ALL: 类似于UNION，但不会去除重复的行，它会将所有行都包含在结果集中。

3. INTERSECT: 用于获取同时存在于两个查询结果集中的行，类似于取交集。与UNION不同，INTERSECT不会去除重复的行。

4. EXCEPT / MINUS: 用于从第一个查询结果集中减去在第二个查询结果集中出现的行，类似于差集操作。在不同的数据库系统中，可能会使用EXCEPT或MINUS来表示这个操作。

## 日期函数
* CURRENT_DATE(): 返回当前日期（不包括时间）。

* CURRENT_TIME(): 返回当前时间（不包括日期）。

* CURRENT_TIMESTAMP() 或 NOW(): 返回当前日期和时间。

* DAY(date),MONTH(date),YEAR(date): 返回日,月,年

## 文本函数
SQL提供了一系列常用的文本函数，用于处理文本数据。这些函数可以用于搜索、提取、转换和操作文本字符串。以下是一些常见的SQL文本函数：

1. **CONCAT(str1, str2, ...)**：将多个文本字符串连接成一个字符串。

   ```sql
   CONCAT(first_name, ' ', last_name) AS full_name
   ```

2. **LENGTH(str)** 或 **LEN(str)**：返回文本字符串的长度。

   ```sql
   LENGTH(description) AS description_length
   ```

3. **UPPER(str)**：将文本字符串转换为大写。

   ```sql
   UPPER(city) AS upper_city
   ```

4. **LOWER(str)**：将文本字符串转换为小写。

   ```sql
   LOWER(username) AS lower_username
   ```

5. **SUBSTRING(str, start, length)** 或 **SUBSTR(str, start, length)**：从文本字符串中提取子字符串。

   ```sql
   SUBSTRING(title, 1, 5) AS first_five_chars
   ```

6. **TRIM([LEADING | TRAILING | BOTH] trim_character FROM str)**：去除文本字符串两端的空格或指定字符。

   ```sql
   TRIM(' ' FROM product_name) AS trimmed_name
   ```

7. **LEFT(str, length)**：从文本字符串的左侧提取指定长度的字符。

   ```sql
   LEFT(description, 50) AS left_description
   ```

8. **RIGHT(str, length)**：从文本字符串的右侧提取指定长度的字符。

   ```sql
   RIGHT(phone_number, 4) AS last_four_digits
   ```

9. **CHAR_LENGTH(str)** 或 **CHARACTER_LENGTH(str)**：返回文本字符串的字符数，考虑多字节字符。

   ```sql
   CHAR_LENGTH(text_content) AS char_count
   ```

10. **REPLACE(str, search, replace)**：将文本字符串中的所有匹配项替换为指定的新字符串。

    ```sql
    REPLACE(comment, 'bad', 'good') AS corrected_comment
    ```

































