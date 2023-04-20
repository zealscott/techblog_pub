<p align="center">
  <b>关系数据库的查询结果都是一个结果表（也是关系）</b>
</p>

# 集聚函数

## 基本语法

- 统计元组个数
  - COUNT(*)
- 统计一列中值的个数
  - COUNT([DISTINCT|ALL]<列名>)
- 计算一列值的总和（此列必须为数值型）
  - SUM([DISTINCT|ALL]<列名>)  
- 计算一列值的平均值（此列必须为数值型）
  - AVG([DISTINCT|ALL]<列名>)
- 求一列中的最大值和最小值
  - MAX([DISTINCT|ALL]<列名>)
  - MIN([DISTINCT|ALL]<列名>)

## 例子

- 查询选修1号课程的学生最高分数

  - ```sql
    SELECTMAX(Grade)
       FROM SC
       WHERE Cno='1';
    ```

- 查询学生201215012选修课程的总学分数

  - ```sql
    SELECT SUM(Ccredit)
        FROM  SC,Course
        WHERE Sno='201215012'                
        AND SC.Cno=Course.Cno; 
    ```


## GROUP BY 子句

细化聚集函数的作用对象

- 如果未对查询结果分组，聚集函数将作用于整个查询结果
- 对查询结果分组后，聚集函数将分别作用于每个组
- 按指定的一列或多列值分组，值相等的为一组

`HAVING`短语与`WHERE`子句的区别：

- 作用对象不同
- `WHERE`子句作用于基表或视图，从中选择满足条件的元组
- `HAVING`**短语作用于组，从中选择满足条件的组**
- `WHERE`**子句不能使用聚合函数！**

### 例子

1. 求各个课程号及相应的选课人数

   ```sql
      SELECT Cno, COUNT(Sno)
      FROM    SC
      GROUP BY Cno; 
   ```

2. 查询选修了3门以上课程的学生学号

   ```sql
   	SELECT Sno
        FROM  SC
        GROUP BY Sno
        HAVING  COUNT(*) >3;      
   ```

3. 查询平均成绩大于等于90分的学生学号和平均成绩

   ```sql
       SELECT  Sno, AVG(Grade)
       FROM  SC
       GROUP BY Sno
       HAVING AVG(Grade)>=90;
   ```

   这里只能使用`HAVING`，不能使用`WHERE`。

## ORDER BY子句

- 可以按一个或多个属性列排序
  - 优先级逐渐降低
- 升序：ASC;
- 降序：DESC; 
- 缺省值为升序
- 对于空值，排序时显示的次序由具体系统实现来决定

### 例子

1. 查询选修了3号课程的学生的学号及其成绩，查询结果按分数降序排列

   ```sql
       SELECT Sno, Grade
       FROM    SC
       WHERE  Cno= ' 3 '
       ORDER BY Grade DESC;
   ```

2. 查询全体学生情况，查询结果按所在系的系号升序排列，同一系中的学生按年龄降序排列

   ```sql
       SELECT  *
       FROM  Student
       ORDER BY Sdept, Sage DESC;  
   ```

# 连接查询

- 连接查询：同时涉及两个以上的表的查询

- 连接条件或连接谓词：用来连接两个表的条件

- 一般格式：

  > [<表名1>.]<列名1>  <比较运算符> [<表名2>.]<列名2>
  >
  > [<表名1>.]<列名1>BETWEEN [<表名2>.]<列名2>AND[<表名2>.]<列名3>

- 连接字段：连接谓词中的列名称

  - 连接条件中的各连接字段类型必须是可比的，但名字不必相同

## （非）等值连接查询

等值连接：连接运算符为`=`，这里与`Join`操作等价。

### 例子

1. 查询每个学生及其选修课程的情况

   ```sql
       SELECT  Student.*, SC.*
       FROM     Student, SC
       WHERE  Student.Sno = SC.Sno;
   ```

2. 一条SQL语句可以同时完成选择和连接查询，这时WHERE子句是由连接谓词和选择谓词组成的复合条件。

   查询选修2号课程且成绩在90分以上的所有学生的学号和姓名

   ```sql
       SELECT  Student.Sno, Sname
       FROM     Student, SC
       WHERE  Student.Sno=SC.Sno  AND    		               
              SC.Cno=' 2 ' AND SC.Grade>90;
   ```

### 执行过程

嵌套循环法（NESTED-LOOP）

- 首先在表1中找到第一个元组，然后从头开始扫描表2，逐一查找满足连接件的元组，找到后就将表1中的第一个元组与该元组拼接起来，形成结果表中一个元组。
- 表2全部查找完后，再找表1中第二个元组，然后再从头开始扫描表2，逐一查找满足连接条件的元组，找到后就将表1中的第二个元组与该元组拼接起来，形成结果表中一个元组。
- 重复上述操作，直到表1中的全部元组都处理完毕

可以发现，等值连接的复杂度很高，为O(m* n)。

## 自身连接

- 自身连接：一个表与其自己进行连接
- 需要给表起别名以示区别
- 由于所有属性名都是同名属性，因此必须使用别名前缀

### 例子

1. 查询每一门课的间接先修课（即先修课的先修课）

   ```sql
   	SELECT  FIRST.Cno, SECOND.Cpno
        FROM  Course  FIRST, Course  SECOND
        WHERE FIRST.Cpno = SECOND.Cno;
   ```

## 外连接

外连接与普通连接的区别

- `普通连接`操作只输出满足连接条件的元组
- `外连接`操作以指定表为连接主体，将主体表中不满足连接条件的元组一并输出
- `左外连接`
  - 列出左边关系中所有的元组
- `右外连接`
  - 列出右边关系中所有的元组

### 例子

```sql
	SELECT Student.Sno, Sname, Ssex, Sage, Sdept, Cno, Grade
    FROM  Student  LEFT OUT JOIN SC ON    
                 (Student.Sno=SC.Sno); 
```

![table](/images/table1.png)

## 多表连接

两个以上的表进行连接。

MongoDB不提供这种操作：

- `JOIN`很慢
- 多级扩展能力差，代价太高

### 例子

1. 查询每个学生的学号、姓名、选修的课程名及成绩

   ```sql
     SELECT Student.Sno, Sname, Cname, Grade
     FROM   Student, SC, Course    /*多表连接*/
     WHERE Student.Sno = SC.Sno 
                  AND SC.Cno = Course.Cno;
   ```

## 嵌套查询

- 一个`SELECT-FROM-WHERE`语句称为一个查询块

- 将一个查询块嵌套在另一个查询块的`WHERE`子句或`HAVING`短语的条件中的查询称为嵌套查询

- 上层的查询块称为外层查询或父查询

- 下层查询块称为内层查询或子查询

- SQL语言允许多层嵌套查询

  - 即一个子查询中还可以嵌套其他子查询

- **子查询的限制**

  - 不能使用ORDERBY子句
  - 因为ORDER BY 结果为有序的，不满足关系的定义，只能作为最后的生成结果


### 带有IN谓词的子查询

1. 查询与“刘晨”在同一个系学习的学声

   ```sql
   SELECT Sno, Sname, Sdept
         	FROM Student
         	WHERE Sdept  IN
                     (SELECT Sdept
                      FROM Student
                      WHERE Sname= ' 刘晨 ');
   /*用自身连接表示*/
   	SELECT  S1.Sno, S1.Sname,S1.Sdept
         FROM     Student S1,Student S2
         WHERE  S1.Sdept = S2.Sdept  AND
              S2.Sname = '刘晨';
   ```

2. 查询选修了课程名为“信息系统”的学生学号和姓名

   ```sql
   SELECT Sno,Sname                
     	FROM    Student                         
    	WHERE Sno  IN
                (SELECT Sno                    
                 FROM    SC                         
                 WHERE  Cno IN
                        (SELECT Cno             
                          FROM Course           
                          WHERE Cname= '信息系统' )
                 );
   /*用连接查询表示*/
   SELECT Sno,Sname
         FROM    Student,SC,Course
         WHERE   Student.Sno = SC.Sno  AND
                 SC.Cno = Course.Cno AND
                 Course.Cname='信息系统';            
   ```

### 带有比较运算符的子查询

- 当能确切知道内层查询返回单值时，可用比较运算符`（>，<，=，>=，<=，!=或< >）`。

- 由于一个学生只可能在一个系学习，则可以用 = 代替IN ：

  ```sql
      SELECT Sno,Sname,Sdept
      FROM    Student
      WHERE Sdept   =
                     (SELECT Sdept
                      FROM    Student
                      WHERE Sname= '刘晨');
  ```

- 注意，用比较运算符取嵌套，只能`SELECT`一个属性，且为数值类型。

- ***不相关子查询***

  - 子查询的查询条件不依赖于父查询
  - 由里向外逐层处理。即每个子查询在上一级查询处理之前求解，子查询的结果用于建立其父查询的查找条件。

- ***相关子查询***

  - 子查询的查询条件依赖于父查询
  - 首先取外层查询中表的第一个元组，根据它与内层查询相关的属性值处理内层查询，若`WHERE`子句返回值为真，则取此元组放入结果表
  - 然后再取外层表的下一个元组
  - 重复这一过程，直至外层表全部检查完为止

#### 例子

1. 找出每个学生超过他选修课程平均成绩的课程号

   ```sql
   SELECT Sno, Cno
       FROM    SC x
       WHERE Grade >= ( SELECT AVG(Grade) 
                        FROM  SC y
                        WHERE y.Sno = x.Sno );
   /*用连接查询表示*/
   SELECT First.Sno, First.Cno
   	FROM SC First JOIN (
       	SELECT Sno, AVG(Grade) as A_Grade
           FROM SC
           GROUP BY Sno) SA
           ON First.Sno = SA.Sno
        WHERE First.Grade > SA.A_Grade
   ```

### 带有ANY（SOME）或ALL谓词的子查询

使用ANY或ALL谓词时必须同时使用比较运算

若子查询中不是唯一的，使用ANY/ALL可以使用比较运算符

语义为：

> \> ANY  大于子查询结果中的某个值       
>
> \>ALL  大于子查询结果中的所有值
>
> \>=ANY  大于等于子查询结果中的某个值    
>
> <=ANY  小于等于子查询结果中的某个值 
>
> =ANY   等于子查询结果中的某个值        
>
> !=（或<>）ALL  不等于子查询结果中的任何一个值

#### 例子

1. 查询非计算机科学系中比计算机科学系**任意一个学生**年龄小的学生姓名和年龄

   ```sql
   SELECT Sname,Sage
       FROM    Student
       WHERE Sage < ANY ( SELECT  Sage
                          FROM    Student
                          WHERE Sdept= ' CS ')
        AND Sdept <> ‘CS ' ;           /*父查询块中的条件 */

   /*用聚集函数实现*/

   SELECT Sname,Sage
        FROM   Student
        WHERE Sage < 
                     ( SELECT MAX（Sage）
                        FROM Student
                        WHERE Sdept= 'CS ')
         AND Sdept <> ' CS ';
   ```

2. 查询非计算机科学系中比计算机科学系**所有**学生年龄都小的学生姓名及年龄

   ```sql
   SELECT Sname,Sage
       FROM Student
       WHERE Sage < ALL
                    (SELECT Sage
                      FROM Student
                      WHERE Sdept= ' CS ')
         AND Sdept <> ' CS ’;
   /*用聚集函数实现*/
   SELECT Sname,Sage
       FROM Student
       WHERE Sage < 
                  (SELECT MIN(Sage)
                   FROM Student
                   WHERE Sdept= ' CS ')
       AND Sdept <>' CS ';
   ```

### 带有EXISTS谓词的子查询

EXISTS谓词

- 存在量词
- 带有EXISTS谓词的子查询不返回任何数据，只产生逻辑真值“true”或逻辑假值“false”。
- 若内层查询结果非空，则外层的WHERE子句返回真值
- 若内层查询结果为空，则外层的WHERE子句返回假值
- 由EXISTS引出的子查询，其目标列表达式通常都用\* ，因为带EXISTS的子查询只返回真值或假值，给出列名无实际意义。

#### 例子

1. 查询所有选修了1号课程的学生姓名。

   思路

   - 本查询涉及`Student`和`SC`关系
   - 在`Student`中依次取每个元组的`Sno`值，用此值去检查`SC`表
   - 若`SC`中存在这样的元组，其`Sno`值等于此`Student.Sno`值，并且其`Cno=‘1’`，则取此`Student.Sname`送入结果表

   ```sql
   SELECT Sname
        FROM Student
        WHERE EXISTS
                      (SELECT *
                       FROM SC
                       WHERE Sno=Student.Sno AND Cno= ' 1 ');
   ```

2. 查询没有选修1号课程的学生姓名。

   ```sql
   SELECT Sname
        FROM     Student
        WHERE NOT EXISTS
                      (SELECT *
                       FROM SC
                       WHERE Sno = Student.Sno AND 	  Cno='1');
   ```


### 难点

- 不同形式的查询间的替换

  - 一些带EXISTS或NOT EXISTS谓词的子查询不能被其他形式的子查询等价替换

  - 所有带IN谓词、比较运算符、ANY和ALL谓词的子查询都能用带EXISTS谓词的子查询等价替换

  - 查询与“刘晨”在同一个系学习的学生

    - 可以用带EXISTS谓词的子查询替换

    - ```sql
      SELECT Sno,Sname,Sdept
           FROM Student S1
            WHERE EXISTS
                   　   (SELECT *
                           FROM Student S2
                           WHERE S2.Sdept = S1.Sdept AND
                                 S2.Sname = '刘晨');
      ```

- 用EXISTS/NOT EXISTS实现全称量词（难点）

  - 查询选修了全部课程的学生姓名

    - 不存在一门课,这个学生没有选

    - ```sql
      SELECT Sname
              FROM Student
              WHERE NOT EXISTS
                            (SELECT *
                              FROM Course
                              WHERE NOT EXISTS
                                            (SELECT *
                                             FROM SC
                                             WHERE Sno= Student.Sno
                                                   AND Cno= Course.Cno
                                            )
                             );
      ```
