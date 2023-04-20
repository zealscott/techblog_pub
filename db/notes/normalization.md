<p align="center">
  <b>如何设计关系数据库的各个表，减少数据冗余？</b>
</p>

# 数据库范式化

- **目的**
  - 减少数据中重复和冗余 
- **冗余带来的问题**
  - 额外的存储开销
  - 语义不清晰
  - 数据增删改的麻烦
    - 程序员必须知道容易的存在
    - 应用程序与数据库之间的关系复杂化
- **在关系数据库的理念中，好的模式设计应该避免冗余**
  - 性能问题应该交给物理层来解决
  - 但对于实际问题中，很难做到

## 第一范式

- 只要满足关系的定义（笛卡尔积的子集），则满足第一范式

### 函数依赖

设R(U) 为属性集 U上的关系模式。 X，Y是U的子集。若对于 R(U) 的 任意一个在现实世界中可能的关系 r，r中不可能存在 两个元组中X上的属性值相上的属性值相 等，而在Y上的属性值不等，则称 Y函数依赖于 X， 记作 X->Y。

### 完全函数依赖

对于X，必须由其全部属性才能确定Y，则称为Y完全依赖于X。

### 键

- 设R(U) 为属性集 U上的关系模式。 X是U的子集。  如果 X->U，并且不存在  X的子集 X’ 使得 X’ ->U， 那么 X为R(U) 的键/码（key）。
- 在R(U)中可能存在多个键，我们人为指定其中的一个键为主键（primary key）
- 包含在任意一个键中的属性，称为主属性（primary attribute）。

## 第二范式

- 关系模式R满足第二范式，当且仅当，R满足第一范式，并且，R的每一个非主属性都完全依赖于R的每一个键。
- 若不是键，则不能决定其他属性。

## 第三范式

- 关系模式R满足第三范式，当且仅当，R中不存在这样的键X，属性组Y和非主属性Z，使得X->Y，Y->Z成立，且Y->X不成立。
- 即：若X是键，Y不是键，但Y能决定Z，则不满足第三范式。
- 满足第三范式的模式必满足第二范式。

## BC范式

- 关系模式 R满足 BC 范式，当且仅对任意一个属性集 A，如果存在不属于A一个属性 X， 使得 X函数依赖于A，那么所有R的属性都函数依赖于A。
- 任何满足BC范式的关系模式都满足第三范式。
- BCNF与[第三范式](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%B8%89%E8%8C%83%E5%BC%8F)的不同之处在于：
  - 第三范式中不允许[非主属性](https://zh.wikipedia.org/w/index.php?title=%E9%9D%9E%E4%B8%BB%E5%B1%9E%E6%80%A7&action=edit&redlink=1)被另一个非主属性决定，但第三范式允许主属性被非主属性决定；
  - 而在BCNF中，任何属性（包括非主属性和主属性）都不能被非主属性所决定。