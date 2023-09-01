## 分组计算练习题

* Q: 现在运营想要对每个学校不同性别的用户活跃情况和发帖数量进行分析，请分别计算出每个学校每种性别的用户数、30天内平均活跃天数和平均发帖数量。
```sql
drop table if exists user_profile;
CREATE TABLE `user_profile` (
`id` int NOT NULL,
`device_id` int NOT NULL,
`gender` varchar(14) NOT NULL,
`age` int ,
`university` varchar(32) NOT NULL,
`gpa` float,
`active_days_within_30` float,
`question_cnt` float,
`answer_cnt` float
);
INSERT INTO user_profile VALUES(1,2138,'male',21,'北京大学',3.4,7,2,12);
INSERT INTO user_profile VALUES(2,3214,'male',null,'复旦大学',4.0,15,5,25);
INSERT INTO user_profile VALUES(3,6543,'female',20,'北京大学',3.2,12,3,30);
INSERT INTO user_profile VALUES(4,2315,'female',23,'浙江大学',3.6,5,1,2);
INSERT INTO user_profile VALUES(5,5432,'male',25,'山东大学',3.8,20,15,70);
INSERT INTO user_profile VALUES(6,2131,'male',28,'山东大学',3.3,15,7,13);
INSERT INTO user_profile VALUES(7,4321,'male',28,'复旦大学',3.6,9,6,52);
```
* A
```sql
SELECT 
gender,
university,
COUNT(*) user_num,
AVG(active_days_within_30) avg_active_day,
CAST(avg(question_cnt) AS DECIMAL(10,1)) avg_question_cnt
FROM user_profile
GROUP BY gender,university
```
* 收获
1. GROUP BY gender,university可锁定两个关键字相同的为同一个组,用来分组两个
2. CAST(avg(question_cnt) AS DECIMAL(10,1)) 可将int类型转换成DECIMAL之后保留一位小数

## 统计每个学校的答过题的用户的平均答题数(28.11%)
* Q:运营想要了解每个学校答过题的用户平均答题数量情况，请你取出数据。
## user_profile
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230831224526.png)
## question_practice_detail
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230831224610.png)
```sql
SELECT university,SUM(pc) / COUNT(*) avg_answer_cnt
FROM 
(SELECT COUNT(*) pc,q.device_id,university
FROM user_profile u,question_practice_detail q
WHERE u.device_id=q.device_id 
GROUP BY q.device_id,university) t
GROUP BY university
ORDER BY universit
```
##  统计每个学校各难度的用户平均刷题数
* Q: 运营想要计算一些参加了答题的不同学校、不同难度的用户平均答题量，请你写SQL取出相应数据
### user_profile
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901111131.png)
### question_practice_detail
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901111218.png)
### question_detail
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901111251.png)

* A
```sql
SELECT university,difficult_level,SUM(c) / COUNT(*)
FROM 
(SELECT university,difficult_level,q.device_id,COUNT(*) c
FROM user_profile u, question_practice_detail q,question_detail r
WHERE  u.device_id=q.device_id AND q.question_id=r.question_id
GROUP BY university,difficult_level,q.device_id) t
GROUP BY university,difficult_level
ORDER BY university
```
### **解体思路**
1. 先分析题目的要求:**参加了答题的不同学校、不同难度的用户平均答题量，请你写SQL取出相应数据**
2. 可以分析出来,是根据**不同学校、不同难度**分组,获取平均答题数量,分子是**每个学校每个题目的总数**,分子是**每个学校,每种难度学生的数量**
3. 写出一个子查询,细化分组(使用三个分组条件,获取答题的数量)
```sql
SELECT university,difficult_level,q.device_id,COUNT(*) c
FROM user_profile u, question_practice_detail q,question_detail r
WHERE  u.device_id=q.device_id AND q.question_id=r.question_id
GROUP BY university,difficult_level,q.device_id
```
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901111855.png)
4. 这张表里面的c为每个学校,每个难度,每个设备的答题数量
5. 再根据这个表分组将分组条件改为每个学校和每个难度,将新的分组的SUM(c)就是每个学校,每个难度的题目总数,COUNT(*)就是每个学校,每个难度的人数
```sql
SELECT university,difficult_level,SUM(c) / COUNT(*)
FROM 
(SELECT university,difficult_level,q.device_id,COUNT(*) c
FROM user_profile u, question_practice_detail q,question_detail r
WHERE  u.device_id=q.device_id AND q.question_id=r.question_id
GROUP BY university,difficult_level,q.device_id) t
GROUP BY university,difficult_level
ORDER BY university
```

## 查找山东大学或者性别为男生的信息
* Q:现在运营想要分别查看学校为山东大学或者性别为男性的用户的device_id、gender、age和gpa数据，请取出相应结果，结果不去重。(使用UNION)
```sql
SELECT device_id,gender,age,gpa
FROM  user_profile
WHERE university="山东大学" 
UNION ALL
SELECT device_id,gender,age,gpa
FROM  user_profile
WHERE gender="male" 
```
* 总结
1. UNION: 已经提到过的关键字，用于合并两个或多个查询的结果集，并去除重复的行。

2. UNION ALL: 类似于UNION，但不会去除重复的行，它会将所有行都包含在结果集中。

3. INTERSECT: 用于获取同时存在于两个查询结果集中的行，类似于取交集。与UNION不同，INTERSECT不会去除重复的行。

4. EXCEPT / MINUS: 用于从第一个查询结果集中减去在第二个查询结果集中出现的行，类似于差集操作。在不同的数据库系统中，可能会使用EXCEPT或MINUS来表示这个操作。

## 计算25岁以上和以下的用户数量
* Q: 现在运营想要将用户划分为25岁以下和25岁及以上两个年龄段，分别查看这两个年龄段用户数量(使用分支语句)
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901140327.png)
```sql
SELECT 
CASE 
	WHEN AGE<25 OR AGE IS NULL
	THEN
		'25岁以下'
	ELSE
		'25岁及以上'
END AS age_cut,
COUNT(*) number
FROM user_profile
GROUP BY age_cut
ORDER BY age_cut
```
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901140350.png)

* 总结
1. 使用分支可以将字段的值作为一个集合作为分组,AS age_cut,转换字段名
2. AGE<25的时候如果AGE是null,判断情况是UNKNOWN,不为真也不为假

## 查看不同年龄段的用户明细
* Q:现在运营想要将用户划分为20岁以下，20-24岁，25岁及以上三个年龄段，分别查看不同年龄段用户的明细情况，请取出相应数据。（注：若年龄为空请返回其他。）
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901143909.png)
```sql
SELECT device_id,gender,
CASE 
	WHEN age<20 THEN "20岁以下"
	WHEN age < 25 AND age>=20 THEN "20-24岁"
	WHEN age>=25 THEN "25岁及以上"
	ELSE "其他"
END age_cut
FROM user_profile
```
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901144001.png)
* 总结
1. 使用CASE WHEN 语句来将不同年纪的人年纪改为标签
2. 将null字段放在其他里面("其他")

## 计算用户8月每天的练题数量(日期函数)
* Q:现在运营想要计算出2021年8月每天用户练习题目的数量，请取出相应数据。
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901152604.png)
```sql
SELECT DAY(date) day,COUNT(*) question_cnt
FROM question_practice_detail
WHERE YEAR(date)=2021 AND MONTH(date)=8
GROUP BY day 
```
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901152556.png)
* 总结
1. 使用YEAR(date),MONTH(date),DAY(date)来获取日期的年月日

## 计算用户的平均次日留存率
* Q:现在运营想要查看用户在某天刷题后第二天还会再来刷题的平均概率。请你取出相应数据。
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901155757.png)
```sql
SELECT COUNT(d2)/COUNT(*) avg_ret
FROM
(SELECT DISTINCT q1.device_id,q1.date d1,q2.date d2
FROM question_practice_detail q1
LEFT JOIN
question_practice_detail q2
ON 
YEAR(q1.date) = YEAR(q2.date) 
AND MONTH(q1.date) = MONTH(q2.date)
AND DAY(q1.date)+1 = DAY(q2.date)
AND q1.device_id=q2.device_id) t
```
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901155819.png)

 * 总结
 1. 使用


 ##  统计每种性别的人数
 * Q:现在运营举办了一场比赛，收到了一些参赛申请，表数据记录形式如下所示，现在运营想要统计每个性别的用户分别有多少参赛者，请取出相应结果
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901163005.png)
 ```sql
 SELECT SUBSTRING_INDEX(profile,",",-1) gender,COUNT(*) number
 FROM user_submit
 GROUP BY gender
 ```
 ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230901163026.png)
* 总结
1. 使用SUBSTRING_INDEX(filed,split,index) index这个是从1开始获取,保留前面的所有
2. 使用-1就是从后往前获取
3. 根据,分割字符串获取














































































