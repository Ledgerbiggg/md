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


















































































