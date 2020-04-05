# MySQL 练习51题
## 练习数据
1. 学生表 Student(SId,Sname,Sage,Ssex)：  
SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别
2. 课程表 Course(CId,Cname,TId)：  
CId 课程编号,Cname 课程名称,TId 教师编号
3. 教师表 Teacher(TId,Tname)：   
TId 教师编号,Tname 教师姓名
4. 成绩表 SC(SId,CId,score)：  
SId 学生编号,CId 课程编号,score 分数

## 创建测试数据
### 学生表 student
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));  
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');  
insert into Student values('02' , '钱电' , '1990-12-21' , '男');  
insert into Student values('03' , '孙风' , '1990-05-20' , '男');  
insert into Student values('04' , '李云' , '1990-08-06' , '男');  
insert into Student values('05' , '周梅' , '1991-12-01' , '女');  
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');  
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');  
insert into Student values('09' , '张三' , '2017-12-20' , '女');  
insert into Student values('10' , '李四' , '2017-12-25' , '女');  
insert into Student values('11' , '李四' , '2017-12-30' , '女');  
insert into Student values('12' , '赵六' , '2017-01-01' , '女');  
insert into Student values('13' , '孙七' , '2018-01-01' , '女');  

### 科目表 Course
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));  
insert into Course values('01' , '语文' , '02');  
insert into Course values('02' , '数学' , '01');  
insert into Course values('03' , '英语' , '03');  

### 教师表 Teacher
create table Teacher(TId varchar(10),Tname varchar(10));  
insert into Teacher values('01' , '张三');  
insert into Teacher values('02' , '李四');  
insert into Teacher values('03' , '王五');  

## 成绩表 SC
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));  
insert into SC values('01' , '01' , 80);  
insert into SC values('01' , '02' , 90);  
insert into SC values('01' , '03' , 99);  
insert into SC values('02' , '01' , 70);  
insert into SC values('02' , '02' , 60);  
insert into SC values('02' , '03' , 80);  
insert into SC values('03' , '01' , 80);  
insert into SC values('03' , '02' , 80);  
insert into SC values('03' , '03' , 80);  
insert into SC values('04' , '01' , 50);  
insert into SC values('04' , '02' , 30);  
insert into SC values('04' , '03' , 20);  
insert into SC values('05' , '01' , 76);  
insert into SC values('05' , '02' , 87);  
insert into SC values('06' , '01' , 31);  
insert into SC values('06' , '03' , 34);  
insert into SC values('07' , '02' , 89);  
insert into SC values('07' , '03' , 98);

## 查询语句
1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

SELECT *  
FROM sc sca  
INNER JOIN sc scb  
ON sca.SID = scb.SID AND sca.CID = '01' AND scb.CID = '02'  
LEFT JOIN student st  
ON sca.SID = st.SID  
WHERE sca.score > scb.score;  

SELECT st.*, sca.score, scb.score  
FROM (SELECT * FROM sc WHERE CID = 01) sca  
INNER JOIN (SELECT * FROM sc WHERE CID = 02) scb  
ON sca.SId = scb.SID  
LEFT JOIN student st  
ON sca.SID = st.SID  
WHERE sca.score > scb.score  

2. 查询同时存在" 01 "课程和" 02 "课程的情况

SELECT *  
FROM sc sca
INNER JOIN sc scb    
ON sca.SID = scb.SID AND sca.CID = 01 AND scb.CID = 02  

3. 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )

SELECT *  
FROM sc sca  
LEFT JOIN sc scb  
ON sca.SID = scb.SID AND scb.CID = 02  
WHERE sca.CID = 01  

4. 查询不存在" 01 "课程但存在" 02 "课程的情况

SELECT *  
FROM sc sca  
LEFT JOIN sc scb  
ON sca.SID = scb.SID AND sca.CID = 02 AND scb.CID = 01  
WHERE sca.CID = 02 AND scb.CID IS NULL;  

SELECT *  
FROM sc    
WHERE sc.SID NOT IN (SELECT DISTINCT SID FROM sc WHERE sc.CID = 02)  
AND sc.CID = 01  

5. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩

SELECT st.SID, st.Sname, AVG(sc.score) AS 平均分  
FROM sc  
LEFT JOIN student st  
ON sc.SID = st.SID  
GROUP BY sc.SID  
HAVING AVG(sc.score) >=60  

6. 查询在 SC 表存在成绩的学生信息

SELECT st.*  
FROM sc  
LEFT JOIN student st  
ON sc.SID = st.SID   
GROUP BY sc.SID  

7. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )

SELECT st.SID, st.Sname, COUNT(sc.CID) AS 选课总数, SUM(sc.score) AS 总成绩   
FROM sc  
RIGHT JOIN student st  
ON sc.SID = st.SID  
GROUP BY sc.SID  

8. 查有成绩的学生信息

SELECT st.SID, st.Sname, COUNT(sc.CID) AS 选课总数, SUM(sc.score) AS 总成绩   
FROM sc  
RIGHT JOIN student st  
ON sc.SID = st.SID  
GROUP BY sc.SID   
HAVING COUNT(sc.CID) > 0  

9. 查询「李」姓老师的数量

SELECT COUNT(1)  
FROM teacher t  
WHERE t.Tname LIKE '李%'  

10. 查询学过「张三」老师授课的同学的信息

SELECT st.*  
FROM sc, student st, teacher t, course c  
WHERE t.TID = c.TID  
AND c.CID = sc.CID  
AND sc.SID = st.SID  
AND t.Tname = '张三'；  

SELECT *  
FROM student st  
WHERE st.SID IN   
	(SELECT SID  
    FROM sc  
    WHERE sc.CID IN  
		(SELECT CID  
        FROM course c  
        WHERE c.TID =   
			(SELECT TID  
            FROM teacher  
            WHERE Tname = '张三')))  

11. 查询没有学全所有课程的同学的信息

SELECT st.*, COUNT(sc.CID)  
FROM student st  
LEFT JOIN sc  
ON st.SID = sc.SID  
GROUP BY sc.SID  
HAVING COUNT(sc.CID) < (SELECT COUNT(DISTINCT sc.CID) FROM sc)  

12. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

SELECT st.*  
FROM sc  
LEFT JOIN student st  
ON sc.SID = st.SID  
WHERE sc.CID IN (SELECT CID FROM sc WHERE SID = 01) AND sc.SID <> 01    
GROUP BY sc.SID;  

13. 查询和" 01 "号的同学学习的课程 完全相同的其他同学的信息

SELECT st.*  
FROM sc  
LEFT JOIN student st  
ON sc.SID = st.SID  
WHERE sc.CID IN (SELECT CID FROM sc WHERE SID = 01) AND sc.SID <> 01  
GROUP BY sc.SID  
HAVING COUNT(sc.CID) = (SELECT COUNT(CID) FROM sc   WHERE SID = 01);  

14. 查询没学过"张三"老师讲授的任一门课程的学生姓名

SELECT st.sname  
FROM student st  
WHERE st.sid NOT IN	 
	(SELECT sid  
    FROM sc WHERE  
    sc.cid IN   
		(SELECT cid  
        FROM course  
        WHERE tid IN   
			(SELECT tid  
            FROM teacher  
            WHERE tname = '张三')));  

15. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

SELECT st.sid, st.sname, AVG(sc.score)  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
GROUP BY sc.sid  
HAVING SUM(IF(sc.score<60,1,0)) >= 2;  

16. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息

SELECT st.*, sc.cid, sc.score  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
WHERE sc.CID = 01 AND sc.score<60  
ORDER BY sc.score DESC  

17. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

SELECT st.*, sc.*, avg_score  
FROM sc  
RIGHT JOIN student st  
ON sc.sid = st.sid  
LEFT JOIN   
	(SELECT sid, AVG(score) AS avg_score  
    FROM sc  
    GROUP BY sid  
	) avgs  
ON sc.sid = avgs.sid  
ORDER BY avg_score DESC, st.sid DESC  

18. 查询各科成绩最高分、最低分和平均分： 以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率 
及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90 
要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

SELECT sc.cid, c.cname, COUNT(sc.sid), MAX(sc.score), MIN(sc.score), AVG(sc.score),   
SUM(IF(sc.score>=60,1,0))/count(1) AS 及格率,  
SUM(IF(sc.score>=70 AND sc.score<80,1,0))/count(1) AS 中等率,  
SUM(IF(sc.score>=80 AND sc.score<90,1,0))/count(1) AS 优良率,  
SUM(IF(sc.score>=90,1,0))/count(1) AS 优秀率  
FROM sc  
LEFT JOIN course c  
ON sc.cid = c.cid  
GROUP BY sc.cid  
ORDER BY COUNT(sc.sid) DESC, sc.cid ASC;  

19. 按各科成绩进行排序，并显示排名， Score 重复时也继续排名

SELECT sc.*, ROW_NUMBER() OVER(ORDER BY score DESC) AS r  
FROM sc  
ORDER BY r;  

**select   
    sid,cid,score,  
    @rank:=@rank+1 as rn   
from sc ,(select @rank:=0) as t   
order by score desc;**

20. 按各科成绩进行排序，并显示排名， Score 重复时合并名次

SELECT sc.*, RANK() OVER(ORDER BY score DESC) AS r  
FROM sc  
ORDER BY r;  

**select   
\*,  
case when (@sco=score) then @rank else @rank:=@rank+1 end as rn,  
@sco:=score  -- 保存上一次的分数  
 from sc ,(select @rank:=0,@sco:=null) as t order by score desc**  

21. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺

SELECT total_score.*,  
CASE WHEN 总成绩 = @curscore THEN '' ELSE @rank:=@rank+1 END AS rn,  
@curscore = 总成绩  
FROM  
(SELECT sc.*, SUM(sc.score) AS 总成绩  
FROM sc, (select @rank:=0, @curscore:=NUll) v  
GROUP BY sc.sid) total_score  
ORDER BY total_score.总成绩 DESC  

22. 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺

SELECT total_score.*,  
CASE WHEN 总成绩 = @curscore THEN @rank ELSE @rank:=@rank+1 END AS rn,  
@curscore = 总成绩  
FROM 
(SELECT sc.*, SUM(sc.score) AS 总成绩  
FROM sc, (select @rank:=0, @curscore:=NUll) v  
GROUP BY sc.sid) total_score  
ORDER BY total_score.总成绩 DESC  

23. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比

SELECT *,   
SUM(CASE WHEN score>= 0 AND score<60 THEN 1 ELSE 0 END)/COUNT(1) AS '0-60',  
SUM(CASE WHEN score>= 60 AND score<70 THEN 1 ELSE 0 END)/COUNT(1) AS '60-70',  
SUM(CASE WHEN score>= 70 AND score<85 THEN 1 ELSE 0 END)/COUNT(1) AS '70-85',  
SUM(CASE WHEN score>= 85 AND score<=100 THEN 1 ELSE 0 END)/COUNT(1) AS '85-100'  
FROM sc  
GROUP BY cid  

24. 查询各科成绩前三名的记录

SELECT *  
FROM   
(SELECT *, DENSE_RANK() OVER(PARTITION BY cid ORDER BY score) AS rn  
FROM sc) rn_table  
WHERE rn <= 3  
ORDER BY cid ASC, rn ASC;

SELECT *  
FROM sc sca  
WHERE 3 <=   
(SELECT COUNT(1)  
FROM sc scb  
WHERE sca.CId = scb.cid AND sca.score > scb.score )  
ORDER BY cid ASC, score DESC;

25. 查询每门课程被选修的学生数

SELECT cid, COUNT(sc.sid)  
FROM sc  
GROUP BY sc.cid;  

26. 查询出只选修两门课程的学生学号和姓名

SELECT sc.sid, st.sname  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
GROUP BY sc.sid  
HAVING COUNT(sc.cid) = 2  

27. 查询男生、女生人数

SELECT ssex, count(1)  
FROM student st  
GROUP BY ssex  

28. 查询名字中含有「风」字的学生信息

SELECT *  
FROM student st  
WHERE sname LIKE '%风%'  

29. 查询同名同性学生名单，并统计同名同姓人数

SELECT *, COUNT(2)  
FROM student st  
GROUP BY sname, ssex  
HAVING COUNT(1) > 1;  

SELECT *  
FROM student sta  
INNER JOIN student stb  
ON sta.sname = stb.sname AND sta.ssex = stb.ssex AND sta.sid != stb.sid;  

30. 查询 1990 年出生的学生名单

SELECT *  
FROM student st  
WHERE YEAR(sage) =1990  

31. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

SELECT cid, AVG(score) as Avg_sc  
FROM sc  
GROUP BY cid  
ORDER BY Avg_sc DESC, cid ASC  

32. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩

SELECT sc.sid, st.sname, AVG(sc.score) AS Avg_sc  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
GROUP BY sc.sid  
HAVING Avg_sc > 85;  

33. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

SELECT st.sname, sc.score  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
WHERE sc.cid = (SELECT cid FROM course WHERE cname = '数学')  
AND sc.score >60  

34. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）

SELECT *  
FROM student st  
LEFT JOIN sc  
ON st.sid = sc.sid  

35. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数

SELECT *  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
LEFT JOIN course c 
ON sc.cid = c.cid  
WHERE sc.score > 70  

36. 查询存在不及格的课程

SELECT DISTINCT cid  
FROM sc  
WHERE sc.score < 60  

37. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名

SELECT *  
FROM sc  
LEFT JOIN student st  
ON sc.sid = st.sid  
WHERE sc.cid = 01 AND sc.score > 80;  

38. 求每门课程的学生人数

SELECT cid, COUNT(sid)  
FROM sc  
GROUP BY cid  

39. 假设成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

SELECT *  
FROM sc  
INNER JOIN student st  
WHERE sc.cid IN   
	(SELECT cid  
    FROM course  
    WHERE tid =   
		(SELECT tid  
        FROM teacher  
        WHERE tname = '张三'))  
AND sc.sid = st.sid  
ORDER BY sc.score DESC       
LIMIT 1    

40. 假设成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

SELECT *  
FROM student st  
INNER JOIN  
(SELECT *, RANK() OVER(ORDER BY score DESC) AS sc_rn  
FROM sc  
WHERE sc.cid IN   
	(SELECT cid  
    FROM course  
    WHERE tid =   
		(SELECT tid  
        FROM teacher  
        WHERE tname = '张三'))) rn  
WHERE rn.sid = st.sid AND sc_rn = 1  

41. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

SELECT *  
FROM sc sca  
INNER JOIN sc scb  
WHERE sca.cid != scb.cid   
AND sca.score = scb.score   
AND sca.sid != scb.sid  
GROUP BY sca.sid  

42. 查询每门功成绩最好的前两名

SELECT *  
FROM sc sca  
WHERE 2 > (SELECT COUNT(1)  
		   FROM sc scb  
           WHERE sca.cid = scb.cid AND sca.score < scb.score)  
ORDER BY sca.cid  

43. 统计每门课程的学生选修人数（超过 5 人的课程才统计）

SELECT cid, COUNT(1)  
FROM sc  
GROUP BY cid   
HAVING COUNT(1) > 5  

44. 检索至少选修两门课程的学生学号

SELECT sc.sid  
FROM sc  
GROUP BY sc.sid  
HAVING COUNT(sc.cid) >= 2  

45. 查询选修了全部课程的学生信息 

SELECT *  
FROM student st  
LEFT JOIN sc  
ON st.sid = sc.sid  
GROUP BY sc.sid  
HAVING COUNT(sc.cid) = (SELECT COUNT(DISTINCT cid)FROM sc)  

46. 查询各学生的年龄，只按年份来算

SELECT *, YEAR(NOW())-YEAR(sage)  
FROM student;  

47. 按照出生日期来算

SELECT *, TIMESTAMPDIFF(YEAR,sage,now())  
FROM student;  

48. 查询本周过生日的学生

SELECT *, sage, NOW()  
FROM student  
WHERE ABS(WEEk(NOW()) - WEEk(sage)) = 0;  

**select   
\*,  
YEARWEEK(student.Sage),  
substr(YEARWEEK(student.Sage),5,2) as birth_week  
from student   
where substr(YEARWEEK(student.Sage),5,2)=substr(YEARWEEK(CURDATE()),5,2);**  

49. 查询下周过生日的学生 

SELECT *, sage, NOW()  
FROM student  
WHERE WEEk(NOW()) - WEEk(sage) = -1;

**select   
    \*,  
    substr(YEARWEEK(student.Sage),5,2) as birth_week,  
    substr(YEARWEEK(CURDATE()),5,2) as now_week   
from student   
where substr(YEARWEEK(student.Sage),5,2)=  
        substr(YEARWEEK(CURDATE()),5,2)+1;**  

50. 查询本月过生日的学生

SELECT *, sage, NOW()  
FROM student  
WHERE MONTH(NOW()) - MONTH(sage) = 0

51. 查询下月过生日的学生

SELECT *, sage, NOW()  
FROM student  
WHERE  MONTH(sage) - MONTH(NOW())= 1  