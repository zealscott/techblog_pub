建表：

```sql
drop table Student;
drop table Course;
drop table Teacher;
drop table SC;

-- 学生表 Student
create table Student(SId varchar(10),Sname varchar(10),Sage date,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');


-- 科目表 Course
create table Course(CId varchar(10),Cname varchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');


-- 教师表 Teacher
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');


-- 成绩表 SC
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
```

插入数据库

```
\i C:/Users/scott/Desktop/Class.sql
```

# 习题

1. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

   ```SQL
   SELECT sname,score
   FROM Student,
   (SELECT sid,score
   FROM SC,Course
   WHERE Course.cname = '数学' AND Course.cid = SC.cid
   AND SC.score < 60) t
   WHERE (t.sid = Student.sid);
   ```

2. 查询本月份过生日的同学名单

   ```SQL
   SELECT sname 
   FROM Student
   WHERE EXTRACT(month from sage) = EXTRACT(month FROM CURRENT_DATE);
   ```

3. 查询下个月过生日的学生名单

   ```sql
   SELECT sname 
   FROM Student
   WHERE EXTRACT(month from sage) = EXTRACT(month from (CURRENT_DATE + INTERVAL '1 MONTH'));
   ```

4. 查询有数学成绩的学生信息

   ```SQL
   SELECT sname,sage,ssex
   FROM Student,
   (SELECT sid
   FROM SC,Course
   WHERE Course.cname = '数学' AND Course.cid = SC.cid) t
   WHERE (t.sid = Student.sid);
   ```

5. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

   ```sql
   SELECT sname, sage,ssex,score
   FROM Student,
   (SELECT sid,score
   FROM Teacher,Course,SC
   WHERE Teacher.tname = '张三' AND Teacher.tid = Course.tid AND Course.cid = SC.cid
   ) t
   WHERE t.sid = Student.sid
   ORDER BY score
   DESC
   LIMIT 1;
   ```

6. 查询没有学全部课程（语数外）的学生的信息

   ```SQL
   SELECT sname, sage,ssex
   FROM Student,
   (SELECT sid,cname,cname
   FROM Course,SC
   WHERE Course.cid = SC.cid
   )t
   WHERE t.sid = Student.sid
   GROUP BY Student.sid,sname,sage,ssex
   HAVING COUNT(*) < (SELECT COUNT(*) from Course);
   ```

7. 统计各科成绩各分数段人数：课程名称，课程编号，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比

   ```sql
   SELECT cname,cid,
   a as "[100-85]",round(a::numeric/(a+b+c+d),2)*100 || '%' as ratio, 
   b as "[85-70]", round(b::numeric/(a+b+c+d),2)*100 || '%' as ratio, 
   c as "[70-60]", round(c::numeric/(a+b+c+d),2)*100 || '%' as ratio, 
   d as "[60-0]",round(d::numeric/(a+b+c+d),2)*100 || '%' as ratio
   FROM
   (SELECT cname,Course.cid as cid,
   count(case when score >=85 then 1 end) as a,
   count(case when score >=70 and score <85 then 1 end) as b,
   count(case when score >= 60 and score < 70then 1 end) as c,
   count(case when score < 60 then 1 end) as d
   FROM Course,SC
   WHERE Course.cid = SC.cid
   GROUP BY (cname,Course.cid)) t;
   ```

8. 查询平均成绩大于等于 80(且单科大于70) 的同学的学生编号和学生姓名和平均成绩

   ```SQL
   SELECT SC.sid,sname,round(AVG(score),2) as avgScore
   FROM SC,
   (SELECT sid,sname
   FROM Student
   WHERE NOT EXISTS
   (SELECT * FROM SC 
   WHERE SC.sid = Student.sid AND score <= 70)) t
   WHERE SC.sid = t.sid
   GROUP BY SC.sid,sname
   HAVING AVG(score) >= 80
   ```

   - 首先使用`NOT EXISTS`进行相关子查询，查询到单科成绩大于70分的同学id和名字（没有成绩的同学也在里面），然后再求平均数。

9. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

   ```SQL
   SELECT Student.sid,sname,round(AVG(score),2) as avgScore
   FROM Student,SC
   WHERE Student.sid = SC.sid
   GROUP BY Student.sid,sname
   HAVING COUNT(case when score < 60 then 1 end) >=2;
   ```

10. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

    ```sql
    SELECT SC.cid as cid,Course.cname as cname,round(AVG(score),2) as avgScore
    FROM Course,SC
    WHERE Course.cid = SC.cid
    GROUP BY SC.cid,Course.cname
    ORDER BY AVG(score) DESC, SC.cid;
    ```

11. 查询有一门成绩大于90(且没有挂科: >= 60) 的同学的学生编号和学生姓名和平均成绩

    ```sql
    SELECT Student.sid,sname,round(AVG(score),2) as avgScore
    FROM Student,SC
    WHERE Student.sid = SC.sid
    GROUP BY Student.sid,sname
    HAVING COUNT(case when score < 60 then 1 end) = 0 AND COUNT(case when score > 90 then 1 end) =1;
    ```

12. **查询各科成绩前三名的记录**

    ```sql
    select * from sc
    where (
    select count(*) from sc as a
    where sc.cid = a.cid and sc.score<a.score)< 3
    order by cid, sc.score DESC;
    ```

    这里使用了相关子查询，相当于两层循环
