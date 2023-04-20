
# SQL

- 1974 年诞生于 IBM System-R项目 前称SEQUEL （Structured English Query  Language）
- 声明式查询语言，易用，取代关系演算
  - Relational Calculus -> Relational Algebra
  - Relational  -> Relational Algebra 

## Function of SQL

- 数据查询（ DQL ）
- 数据定义（DDL）
- 数据增删改（ DML ）
- 数据访问控制（ DCL ）

![sql1](/images/sql1.png)

# Table

- 在定义表时，对数据格式的要求较高，可以让数据访问的效率更高

## Create

 ![create](/images/create.png)

### Key words

- `PRIMARY KEY`
  - 主码，列完整性约束条件
- `UNIQUE`
  - 取唯一值
  - 为了遵守独立性原则，由数据库来确定数据关系
- `FOREIGN KEY`
  - 设F是基本关系 R的一个或组属性 ，但不是关系 R的码。如果 F与基本关系 S的主码 Ks相对应 ，则称 F是R的外码
  - 基本关系 R称为 参照关系 （Referencing Relation）
  - 基本关系 S称为 被参照关系 （Referenced Relation）
    或目标关系 （Target Relation）
  - **当被参照表中的元组删除时，我们有两个选择：**
    -  连带删除参照表中对应的元组
    - 如果 参照表中对应的元组还存在，删除操作无效

## Alter

![ater](/images/alter.png)

- `<表名>`是要修改的基本表
- `ADD`子句用于增加新列、新的列级完整性约束条件和新的表级完整性约束条件
- `DROP COLUMN`子句用于删除表中的列
  - 如果指定了`CASCADE`短语，则自动删除引用了该列的其他对象
  - 如果指定了`RESTRICT`短语，则如果该列被其他对象引用，关系数据库管理系统将拒绝删除该列
- `DROP CONSTRAINT`子句用于删除指定的完整性约束条件
- `ALTER COLUMN`子句用于修改原有的列定义，包括修改列名和数据类型
- 修改表的代价很大，要么修改整张表，要么不修改。
  - 与MongoDB完全不同

## Drop

![drop](/images/drop.png)

## Select

![select](/images/select.png)

### Key words

- 使用列别名改变查询结果的列标题:

  - ```sql
    SELECT Sname NAME, 'Year of Birth:' BIRTH,
    2014-Sage BIRTHDAY, LOWER(Sdept) DEPARTMENT
    FROM Student;
    ```

  - ![template1](/images/template1.png)

- `DISTINCT`

  - 去掉表中重复的行 

- 确定范围

  - `BETWEEN … AND  …`
  - `NOT BETWEEN  …  AND  …`

- 确定集合

  - IN <值表>, 
  - NOT IN <值表>      

- 字符匹配

  - [NOT] LIKE  ‘<匹配串>’  [ESCAPE ‘ <换码字符>’]

  - 查询名字中第2个字为"阳"字的学生的姓名和学号

    - ```sql
      SELECT Sname,Sno
      FROM   Student
      WHERE  Sname LIKE '__阳%';
      ```

- 涉及空值的查询

  - `IS NULL `或 `IS NOT NULL`
  - IS不能用=代替

-  多重条件查询

  - AND和 OR来连接多个查询条件
  - AND的优先级高于OR


## Index

- 常用索引
  - B-Tree
  - Hash Index
- 通常情况下，主码会由系统自动创建索引
- 人工使用Create Index指令创建任意索引