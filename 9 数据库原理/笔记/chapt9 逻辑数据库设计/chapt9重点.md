# chapt9 逻辑数据库设计重点

## 闭卷客观题复习提示
- **ER转表规则**：强实体建表，1:N外键在N端，M:N单独建表，多值属性单独建表。`【闭卷重点·必背】`
- **1:1联系**：两边强制参与可合并；一边强制一边可选时，通常把可选参与端的主码作为外码放入强制参与端。`【闭卷重点·了解】`
- **规范化**：1NF无重复组，2NF消除部分依赖，3NF消除传递依赖，BCNF要求每个非平凡函数依赖的决定因素都是候选键。`【闭卷重点·常考】`
- **外键约束**：理解级联删除（CASCADE）和置空（SET NULL）的使用场景。`【闭卷重点·必背】`

本章考试最可能和 **ER 图转关系模式** 连在一起考。你可以把它理解成：

```text
chapt5：先画 ER 图
chapt8：整理概念数据库设计
chapt9：把 ER 图落地成关系表，并检查这些表能不能用
```

## 目录

- [1 本章到底在干什么](#1-本章到底在干什么)
- [2 逻辑数据库设计总流程](#2-逻辑数据库设计总流程)
- [3 Step 2.1：从 ER 模型导出关系模式](#3-step-21从-er-模型导出关系模式)
  - [3.1 强实体怎么变表](#31-强实体怎么变表)
  - [3.2 1:N 联系怎么变表](#32-1n-联系怎么变表)
  - [3.3 递归 1:N 联系怎么变表](#33-递归-1n-联系怎么变表)
  - [3.4 1:1 联系怎么变表](#34-11-联系怎么变表)
  - [3.5 M:N 联系怎么变表](#35-mn-联系怎么变表)
  - [3.6 复杂联系怎么变表](#36-复杂联系怎么变表)
  - [3.7 多值属性怎么变表](#37-多值属性怎么变表)
- [4 Step 2.2：用规范化验证关系模式](#4-step-22用规范化验证关系模式)
- [5 Step 2.3：用用户事务验证关系模式](#5-step-23用用户事务验证关系模式)
- [6 Step 2.4：检查完整性约束](#6-step-24检查完整性约束)
- [7 Step 2.5-2.7：用户复核、合并模型和未来增长](#7-step-25-27用户复核合并模型和未来增长)
- [8 考试速记模板](#8-考试速记模板)

---

## 1 本章到底在干什么

逻辑数据库设计是：

> 基于某种数据模型（本课程主要是关系模型），把企业中的数据结构设计出来，但暂时不考虑具体 DBMS 和物理存储细节。

也就是说，本章不是问：

```text
这个表存在哪个磁盘块？
这个索引用 B+ 树还是 Hash？
```

而是问：

```text
ER 图里的实体、联系、属性应该变成哪些表？
每张表的主键是什么？
外键放在哪里？
这些表有没有结构问题？
能不能支持用户要做的查询和更新？
```

一句话：

```text
逻辑数据库设计 = ER 图 -> 关系模式 + 主键外键 + 约束检查
```

---

## 2 逻辑数据库设计总流程

PPT 中给出的 Step 2 逻辑设计流程：

```text
Step 2.1  Derive relations for logical data model
Step 2.2  Validate relations using normalization
Step 2.3  Validate relations against user transactions
Step 2.4  Check integrity constraints
Step 2.5  Review logical data model with users
Step 2.6  Merge logical data models into global data model
Step 2.7  Check for future growth
```

翻译成做题语言：

```text
1. 先把 ER 图转成表。
2. 再用 1NF/2NF/3NF/BCNF 检查表有没有冗余和依赖问题。
3. 再检查这些表能不能完成题目中的业务操作。
4. 最后补齐主键、外键、非空、取值范围、删除更新规则等约束。
```

考试最常考的是：

```text
Step 2.1 ER 图转关系模式
Step 2.4 主键、外键、完整性约束
```

---

## 3 Step 2.1：从 ER 模型导出关系模式

目标：

> 根据 ER 图、数据字典和说明文档，把实体、联系、属性转成关系表。

### 3.1 强实体怎么变表

规则：

```text
每个强实体类型 -> 一个关系表
实体的简单属性 -> 表的列
实体的主键 -> 表的主键
复合属性 -> 通常只保留其简单组成部分
派生属性 -> 一般不存，必要时可通过计算得到 `【闭卷重点·易错】`
```

例：

```text
Student(Sno, Sname, Gender, Sdept)
主键：Sno
```

如果 ER 图中有：

```text
学生：学号、姓名、性别、院系
```

就转成：

```text
Student(Sno, Sname, Gender, Sdept)
```

### 3.2 1:N 联系怎么变表

规则：

```text
1:N 联系中，把 1 端实体的主键复制到 N 端实体表中，作为外键。
联系自己的属性也放到 N 端表中。
```

PPT 的说法是：

```text
A copy of primary key of parent entity is placed into table representing the child entity, to act as a foreign key.
```

通俗记法：

```text
一对多，外键放多端。 `【闭卷重点·必背】`
```

例：

```text
Department(DeptNo, DeptName)
Student(Sno, Sname, DeptNo)
```

其中：

```text
Student.DeptNo -> Department.DeptNo
```

因为一个院系有多个学生，所以外键放在学生表这一端。

### 3.3 递归 1:N 联系怎么变表

递归联系就是同一类实体和自己发生联系。

例：

```text
员工管理员工
Employee(EmpNo, EmpName, ManagerNo)
```

其中：

```text
ManagerNo -> Employee.EmpNo
```

解释：

```text
ManagerNo 存的是另一个员工的 EmpNo。
父实体和子实体都是 Employee，只是角色不同。
```

做题时可以写：

```text
Employee(EmpNo, EmpName, ManagerNo)
主键：EmpNo
外键：ManagerNo 引用 Employee(EmpNo)
```

### 3.4 1:1 联系怎么变表

1:1 联系要看参与约束。

#### 情况 A：两边都是强制参与

如果两边都必须参加这个联系，可以把两个实体合并成一个表。

例：

```text
Person(PersonID, Name)
Passport(PassportNo, IssueDate)
```

如果每个人必须有一本护照，每本护照也必须属于一个人，可以合并：

```text
PersonPassport(PersonID, Name, PassportNo, IssueDate)
```

主键可以选其中一个原主键，另一个可以作为候选键/备用键。

#### 情况 B：一边强制，一边可选

通常把强制参与一端作为子表，放入另一端主键作为外键，或者根据业务选择更自然的一侧。

做题简单记：

```text
1:1 不确定时，把一方主键放到另一方作为外键，并说明该外键最好加 UNIQUE。
```

例：

```text
Employee(EmpNo, EmpName)
ParkingSpace(SpaceNo, EmpNo)
```

如果一个车位最多分给一个员工：

```text
ParkingSpace.EmpNo -> Employee.EmpNo
并且 ParkingSpace.EmpNo 应设置 UNIQUE
```

### 3.5 M:N 联系怎么变表

规则：

```text
M:N 联系必须单独建一张联系表。
把两边实体的主键都复制进联系表，作为外键。
这两个外键通常共同组成联系表的主键。
联系自己的属性也放进联系表。
```

通俗记法：

```text
多对多，单独建表，两边主键拿来当外键。 `【闭卷重点·必背】`
```

例：

```text
Student(Sno, Sname)
Course(Cno, Cname)
Enrol(Sno, Cno, Mark)
```

主键和外键：

```text
Student: 主键 Sno
Course: 主键 Cno
Enrol: 主键 (Sno, Cno)
       外键 Sno -> Student(Sno)
       外键 Cno -> Course(Cno)
```

其中 `Mark` 是“选课”这个联系自己的属性。

### 3.6 复杂联系怎么变表

复杂联系指三元或更多元联系。

规则：

```text
复杂联系 -> 单独建一张联系表
把所有参与实体的主键复制进来作为外键
联系自己的属性也放进来
主键通常由若干外键组合而成，有时还要加联系自身属性
```

例：

```text
Supplier(SupNo, SupName)
Part(PartNo, PartName)
Project(ProNo, ProName)
Supply(SupNo, PartNo, ProNo, Quantity)
```

主键可为：

```text
(SupNo, PartNo, ProNo)
```

外键：

```text
SupNo -> Supplier
PartNo -> Part
ProNo -> Project
```

### 3.7 多值属性怎么变表

多值属性不能直接塞进一个格子里，否则不满足 1NF。

规则：

```text
多值属性 -> 单独建表
新表包含原实体主键 + 多值属性
原实体主键作为外键
通常用 (原实体主键, 多值属性) 作为新表主键
```

例：

```text
Student(Sno, Sname)
StudentPhone(Sno, Phone)
```

主键和外键：

```text
Student: 主键 Sno
StudentPhone: 主键 (Sno, Phone)
              外键 Sno -> Student(Sno)
```

解释：

```text
一个学生可以有多个电话，所以电话不能全部写在 Student.Phone 一个字段里。
```

---

## 4 Step 2.2：用规范化验证关系模式

PPT 里强调：

> 如果关系表不在 3NF，可能说明 ER 模型有问题，或者 ER 转表时引入了错误。

考试做法：

```text
1. 找每张表的主键。
2. 找函数依赖。
3. 判断有没有部分依赖、传递依赖、非候选键决定别人。
4. 如果不满足 3NF/BCNF，就拆表。
```

和 chapt10 对应：

```text
1NF：一个格子一个值。 `【闭卷重点·必背】`
2NF：非主属性不能只依赖组合主键的一部分。 `【闭卷重点·常考】`
3NF：非主属性不能传递依赖于主键。 `【闭卷重点·常考】`
BCNF：每个函数依赖左边都必须是候选键。 `【闭卷重点·必背】`
```

做 ER 转关系题时，如果你得到一张表：

```text
Enrol(Sno, Sname, Cno, Cname, Mark)
```

就要警惕：

```text
Sno -> Sname
Cno -> Cname
(Sno, Cno) -> Mark
```

这张表有部分依赖，不好。应该拆成：

```text
Student(Sno, Sname)
Course(Cno, Cname)
Enrol(Sno, Cno, Mark)
```

---

## 5 Step 2.3：用用户事务验证关系模式

目标：

> 检查关系模式是否能支持用户需要完成的查询、插入、删除、更新。

PPT 里的意思是：

```text
Check tables support the required transactions.
If a transaction requires data in more than one table, check these tables are linked through primary key/foreign key mechanism.
```

通俗理解：

```text
用户要查的东西，你的表里能不能查出来？
用户要更新的东西，你的表里有没有地方存？
如果一次操作涉及多张表，这些表之间有没有主键/外键能连起来？
```

例：

用户事务：

```text
查询某学生选修的所有课程名称和成绩。
```

需要表：

```text
Student(Sno, Sname)
Enrol(Sno, Cno, Mark)
Course(Cno, Cname)
```

连接路径：

```text
Student.Sno = Enrol.Sno
Enrol.Cno = Course.Cno
```

如果你的设计里没有 `Enrol(Sno, Cno, Mark)`，这个事务就很难支持。

---

## 6 Step 2.4：检查完整性约束

完整性约束用于防止数据库变得：

```text
不完整、不准确、不一致
```

常见约束：

| 约束类型 | 含义 | SQL 常见写法 |
|---|---|---|
| 属性域约束 | 限制列的取值范围和类型 | `CHECK`, 数据类型 |
| 实体完整性 | 主键不能为空且唯一 | `PRIMARY KEY` |
| 参照完整性 | 外键必须引用已有主键，或为空 | `FOREIGN KEY ... REFERENCES ...` |
| 一般业务约束 | 业务规则 | `CHECK`, 触发器等 |

### 6.1 外键要考虑两个问题

PPT 里提到外键有两个问题：

```text
1. Are nulls allowed for the foreign key?
2. How do you ensure referential integrity?
```

翻译：

```text
1. 外键能不能取 NULL？
2. 被引用的父表记录删除或更新时，子表怎么办？
```

### 6.2 父表删除时的处理策略

常见策略：

| 策略 | 含义 | SQL 关键词 |
|---|---|---|
| 禁止删除 | 如果还有子表引用，就不允许删父表记录 | `RESTRICT` / `NO ACTION` `【闭卷重点·常考】` |
| 级联删除 | 删除父表记录时，自动删除相关子表记录 | `ON DELETE CASCADE` `【闭卷重点·必背】` |
| 置空 | 删除父表记录时，把子表外键设为 NULL | `ON DELETE SET NULL` `【闭卷重点·了解】` |
| 设默认值 | 删除父表记录时，把子表外键设为默认值 | `ON DELETE SET DEFAULT` |
| 什么也不做 | 不自动维护，容易破坏参照完整性 | 通常不推荐 |

例：

```sql
CREATE TABLE Enrol (
    Sno CHAR(10),
    Cno CHAR(10),
    Mark INT,
    PRIMARY KEY (Sno, Cno),
    FOREIGN KEY (Sno) REFERENCES Student(Sno)
        ON DELETE CASCADE,
    FOREIGN KEY (Cno) REFERENCES Course(Cno)
        ON DELETE RESTRICT
);
```

解释：

```text
删除学生时，他的选课记录一起删除。
如果课程已经被选，不允许直接删除课程。
```

---

## 7 Step 2.5-2.7：用户复核、合并模型和未来增长

### Step 2.5 Review logical data model with users

把设计好的逻辑模型给用户确认：

```text
实体有没有漏？
属性有没有错？
联系和业务规则是否符合实际？
用户事务能不能支持？
```

### Step 2.6 Merge logical data models into global data model

如果不同用户视图分别做了局部逻辑模型，需要合并成全局模型。

合并时要处理：

```text
同名异义：同一个名字表示不同东西。
异名同义：不同名字表示同一个东西。
结构冲突：同一对象在不同模型中属性或主键设计不同。
约束冲突：不同用户视图给出的规则不一致。
```

### Step 2.7 Check for future growth

检查设计是否方便以后扩展：

```text
新增属性是否容易？
新增实体或联系是否容易？
主键是否稳定？
表之间耦合是否过高？
```

---

## 8 考试速记模板

### 8.1 ER 转关系模式总模板

```text
强实体：一个实体一张表，主键照抄。
1:N：外键放 N 端。
1:1：可合并，或一方主键放另一方作外键，通常加 UNIQUE。
M:N：单独建联系表，两边主键作为外键，通常共同组成主键。
复杂联系：单独建联系表，参与实体主键都放进来作外键。
多值属性：单独建表，原实体主键 + 多值属性组成主键。
```

### 8.2 写主键外键的模板

```text
表名(属性1, 属性2, ...)
主键：...
外键：本表属性 -> 被引用表(被引用属性)
```

例：

```text
Enrol(Sno, Cno, Mark)
主键：(Sno, Cno)
外键：Sno -> Student(Sno)
     Cno -> Course(Cno)
```

### 8.3 检查关系模式的模板

```text
1. 是否满足 1NF：属性是否原子？
2. 是否满足 2NF：组合主键下有没有部分依赖？
3. 是否满足 3NF：有没有传递依赖？
4. 是否满足 BCNF：每个函数依赖左边是不是候选键？
5. 是否支持用户事务：查询/插入/删除/更新能否完成？
6. 是否定义完整性约束：主键、外键、非空、CHECK、参照动作。
```

