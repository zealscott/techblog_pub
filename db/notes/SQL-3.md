<p align="center">
  <b>介绍数据修改的SQL语句以及视图与索引</b>
</p>

# 数据修改

三种修改方式：

1. 修改某一个元组的值

   ```sql
   /* 将学生201215121的年龄改为22岁 */
   UPDATE  Student
   SET Sage=22
   WHERE  Sno=' 201215121 '; 
   ```

2. 修改多个元组的值

   ```sql
   /* 将所有学生的年龄增加1岁 */
   UPDATE Student
   SET Sage= Sage+1;
   ```

3. 带子查询的修改语句

   ```sql
   /* 将计算机科学系全体学生的成绩置零 */
   UPDATE  SC
   SET  Grade=0
   WHERE  Sno  IN
   (SELECT Sno
   FROM     Student
   WHERE  Sdept= 'CS' );
   ```


## 要求

- 关系数据库管理系统在执行修改语句时会检查修改操作是否破坏表上已定义的完整性规则
  - 实体完整性
  - 主码不允许修改（如果被外码参照）
  - 用户定义的完整性
    - `NOT NULL`约束
    - `UNIQUE`约束
    - 值域约束

## 删除数据

- 语句格式

  - ```sql
    DELETE
    FROM     <表名>
    [WHERE <条件>];
    ```

- 功能

  - 删除指定表中满足WHERE子句条件的元组

- WHERE子句

  - 指定要删除的元组
  - 缺省表示要删除表中的全部元组，表的定义仍在字典中

### 删除方式

1. 删除某一个元组的值

    ```sql
     /* 删除学号为201215128的学生记录。 */
     DELETE
     FROM Student
     WHERE Sno= '201215128';
    ```

2. 删除多个元组的值

   ```sql
     /* 删除所有的学生选课记录 */
     DELETE
     FROM SC;
   ```

3. 带子查询的删除语句

   ```sql
     /* 删除计算机科学系所有学生的选课记录。*/
     DELETE
     FROM  SC
     WHERE  Sno  IN
     (SELETE  Sno
     FROM   Student
     WHERE  Sdept= 'CS') ;
   ```

## 空值

空值是一个很特殊的值，含有不确定性。对关系运算带来特殊的问题，需要做特殊的处理。

### 空值的产生

向SC表中插入一个元组，学生号是”201215126”，课程号是”1”，成绩为空。


 ```sql
    INSERT INTO SC(Sno,Cno,Grade)
     VALUES('201215126 ','1',NULL);   
    /*该学生还没有考试成绩，取空值*/
    或
     INSERT INTO SC(Sno,Cno)
     VALUES(' 201215126 ','1');             
    /*没有赋值的属性，其值为空值*/
 ```


将Student表中学生号为”201215200”的学生所属的系改为空值。

  ```sql
    UPDATE Student
    SET Sdept = NULL
    WHERE Sno='201215200';
  ```

### 空值的判断

判断一个属性的值是否为空值，用`IS NULL`或`IS NOT NULL`来表示。

```sql
    /* 从Student表中找出漏填了数据的学生信息*/
    SELECT  *
    FROM Student
    WHERE Sname IS NULL OR Ssex IS NULL OR Sage IS NULL OR Sdept IS NULL;
```

### 空值的约束条件

属性定义（或者域定义）中

- 有`NOT NULL`约束条件的不能取空值
  - `Primary Key`不能为空值
- 加了`UNIQUE`限制的属性不能取空值
- 码属性不能取空值

### 空值的运算

- 空值与另一个值（包括另一个空值）的算术运算的结果为空值
- 空值与另一个值（包括另一个空值）的比较运算的结果为`UNKNOWN`。
- 有`UNKNOWN`后，传统二值（TRUE，FALSE）逻辑就扩展成了三值逻辑
  - 无论判断结果是否为True，都不会返回空值

找出选修1号课程的不及格的学生。

 ```sql
    SELECT Sno
    FROM SC
    WHERE Grade < 60 AND Cno='1';
 ```

查询结果不包括缺考的学生，因为他们的Grade值为null。

# 视图

- 虚表，是从一个或几个基本表（或视图）导出的表
- 只存放视图的定义，不存放视图对应的数据
- 基表中的数据发生变化，从视图中查询出的数据也随之改变

## 建立视图

```sql
    CREATE  VIEW 
    <视图名>  [(<列名>  [,<列名>]…)]
    AS  <子查询>
    [WITH  CHECK  OPTION];
```

- 组成视图的属性列名：全部省略或全部指定

  - 全部省略 
    - 由子查询中SELECT目标列中的诸字段组成
  - 明确指定视图的所有列名
    - 某个目标列是聚集函数或列表达式
    - 多表连接时选出了几个同名列作为视图的字段
    - 需要在视图中为某个列启用新的更合适的名字

- 关系数据库管理系统执行`CREATE VIEW`语句时只是把视图定义存入数据字典，并不执行其中的`SELECT`语句。

- `WITHCHECK OPTION`

  - 对视图进行UPDATE，INSERT和DELETE操作时要保证更新、插入或删除的行满足视图定义中的谓词条件（即子查询中的条件表达式）

- 在对视图查询时，按视图的定义从基本表中将数据查出。

- 建立信息系学生的视图

  - ```sql
    CREATE VIEW IS_Student
    AS 
    SELECT Sno,Sname,Sage
    FROM     Student
    WHERE  Sdept= 'IS';
    ```


### 基于多个基表的视图

  - 建立信息系选修了1号课程的学生的视图（包括学号、姓名、成绩）

  - ```sql
    CREATE VIEW IS_S1(Sno,Sname,Grade)
    AS 
    SELECT Student.Sno,Sname,Grade
    FROM  Student,SC
    WHERE  Sdept= 'IS' AND
    Student.Sno=SC.Sno AND
    SC.Cno= '1';
    ```

### 带表达式的视图

  - 定义一个反映学生出生年份的视图。

  - ```sql
    CREATE  VIEW BT_S(Sno,Sname,Sbirth)
    AS 
    SELECT Sno,Sname,2014-Sage
    FROM  Student;
    ```

### 分组视图

  - 将学生的学号及平均成绩定义为一个视图

  - ```sql
    CREAT  VIEW S_G(Sno,Gavg)
    AS  
    SELECT Sno,AVG(Grade)
    FROM  SC
    GROUP BY Sno;
    ```


## 删除视图

- 语句的格式

  - ```sql
    DROP  VIEW <视图名>[CASCADE];
    ```

  - 该语句从数据字典中删除指定的视图定义

- 如果该视图上还导出了其他视图，使用`CASCADE`级联删除语句，把该视图和由它导出的所有视图一起删除

- 删除基表时，由该基表导出的所有视图定义都必须显式地使用`DROP VIEW`语句删除

## 查询视图

- 用户角度
  - 查询视图与查询基本表相同
- 关系数据库管理系统实现视图查询的方法
  - 视图消解法（View Resolution）
  - 进行有效性检查
  - 转换成等价的对基本表的查询
  - 执行修正后的查询

## 视图的作用

- **视图能够简化用户的操作**
- 视图对**重构数据库**提供了一定程度的逻辑独立性
  - 在新表上建立一个旧表视图。应用程序在旧表上的SQL可以保持不变。
- **视图能够对机密数据提供安全保护**
- 适当的利用视图可以更清晰的表达查询

# 索引

- 建立索引的目的：加快查询速度
- 关系数据库管理系统中常见索引：
  - 顺序文件上的索引
  - B+树索引
  - 散列（hash）索引
  - 位图索引
- 特点
  - B+树索引具有动态平衡的优点
  - HASH索引具有查找速度快的特点

## 建立索引

语句格式

```sql
    CREATE [UNIQUE] [CLUSTER] INDEX <索引名> 
    ON <表名>(<列名>[<次序>][,<列名>[<次序>] ]…);
```

- <表名>：要建索引的基本表的名字
- 索引：可以建立在该表的一列或多列上，各列名之间用逗号分隔
- <次序>：指定索引值的排列次序，升序：ASC，降序：DESC。缺省值：ASC
- UNIQUE：此索引的每一个索引值只对应唯一的数据记录
- CLUSTER：表示要建立的索引是聚簇索引


## 修改索引

```sql
    ALTER INDEX <旧索引名> RENAME TO <新索引名>
```

## 删除索引

删除索引时，系统会从数据字典中删去有关该索引的描述。


```sql

    DROP INDEX <索引名>;
```
