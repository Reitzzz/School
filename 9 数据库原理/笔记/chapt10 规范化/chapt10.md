# Chapter 10 Normalization 规范化

## 目录
- [Chapter 10 Normalization 规范化](#chapter-10-normalization-规范化)
  - [目录](#目录)
  - [10.1 数据库设计目标](#101-数据库设计目标)
  - [10.2 规范化概述](#102-规范化概述)
  - [10.3 数据冗余与更新异常](#103-数据冗余与更新异常)
  - [10.4 函数依赖](#104-函数依赖)
    - [10.4.1 平凡依赖与非平凡依赖](#1041-平凡依赖与非平凡依赖)
    - [10.4.2 完全函数依赖与部分函数依赖](#1042-完全函数依赖与部分函数依赖)
    - [10.4.3 传递函数依赖](#1043-传递函数依赖)
  - [10.5 第一范式 1NF](#105-第一范式-1nf)
  - [10.6 第二范式 2NF](#106-第二范式-2nf)
  - [10.7 第三范式 3NF](#107-第三范式-3nf)
  - [10.8 Boyce-Codd 范式 BCNF](#108-boyce-codd-范式-bcnf)
  - [10.9 从 UNF 到 BCNF 的分解复习](#109-从-unf-到-bcnf-的分解复习)
  - [10.10 考试速记](#1010-考试速记)

---

## 10.1 数据库设计目标

数据库设计的目标是创建一个对数据、数据之间联系以及约束条件的准确表示。为了达到这个目标，需要识别一组合适的关系模式。

主要目标：
- 尽量减少数据冗余
- 减少基本关系所需的文件存储空间
- 降低插入、删除、修改时产生异常的可能性
- 让关系模式更清晰地表达业务语义和约束

---

## 10.2 规范化概述

规范化（Normalization）是一种把一个关系转换成一个或多个新关系的过程，目的是消除数据库中可能出现的冗余。

常用范式：
1. 第一范式（1NF）
2. 第二范式（2NF）
3. 第三范式（3NF）
4. Boyce-Codd 范式（BCNF）

规范化的基础是关系属性之间的函数依赖。随着规范化层次提高，关系模式的格式限制会越来越强，也会越来越不容易出现更新异常。

范式强弱关系：

```text
BCNF => 3NF => 2NF => 1NF
```

也就是说，满足 BCNF 的关系一定满足 3NF、2NF 和 1NF。

---

## 10.3 数据冗余与更新异常

如果一个关系把多个主题的数据混在一起，就容易产生数据冗余。冗余本身会浪费存储空间，更重要的是会带来更新异常。

常见更新异常：

1. **插入异常（Insertion Anomaly）**
   - 某些数据必须依赖其他数据才能插入。
   - 例如只想记录一个新系的信息，但表中主键包含学生或课程信息，导致没有学生或课程时无法单独插入该系。

2. **删除异常（Deletion Anomaly）**
   - 删除某条记录时，会意外删除其他仍然需要保留的信息。
   - 例如删除某学生最后一条选课记录时，把该学生所在系或系主任信息也一并丢失。

3. **修改异常（Modification Anomaly）**
   - 同一事实重复出现在多行中，修改时必须同时改多处。
   - 如果某个系主任变更，而表中多行都保存了该系主任姓名，漏改任何一行都会导致数据不一致。

课件中的学生关系示意：

```text
Student(Sno, Sname, Sdept, Mname, Cno, Grade)
```

其中学生、院系、系主任、课程成绩等信息放在同一张表中，容易产生冗余和更新异常。

---

## 10.4 函数依赖

函数依赖（Functional Dependency, FD）描述关系中属性之间的确定关系。

设关系模式中有属性集 `X` 和 `Y`，如果在任意合法关系实例中，两个元组只要 `X` 的值相同，则 `Y` 的值也必须相同，那么称 `Y` 函数依赖于 `X`，记作：

```text
X -> Y
```

其中：
- `X` 称为决定因素（Determinant）
- `Y` 称为被决定属性

示例：

```text
Sno -> Sname
Sno -> Sdept
Sdept -> Mname
(Sno, Cno) -> Grade
```

含义：
- 一个学号确定一个学生姓名
- 一个学号确定一个所在系
- 一个系确定一个系主任
- 一个学生和一门课程共同确定该学生该课程的成绩

### 10.4.1 平凡依赖与非平凡依赖

如果 `Y` 是 `X` 的子集，则 `X -> Y` 是平凡函数依赖。例如：

```text
(Sno, Cno) -> Sno
(Sno, Cno) -> Cno
```

如果 `Y` 不是 `X` 的子集，则 `X -> Y` 是非平凡函数依赖。例如：

```text
Sno -> Sname
(Sno, Cno) -> Grade
```

规范化讨论中更关注非平凡依赖。

### 10.4.2 完全函数依赖与部分函数依赖

设 `A -> B`。如果 `B` 依赖于整个属性集 `A`，并且不依赖于 `A` 的任何真子集，则称 `B` 完全函数依赖于 `A`。

如果 `B` 依赖于 `A`，同时也依赖于 `A` 的某个真子集，则称 `B` 部分函数依赖于 `A`。

部分函数依赖通常只在组合主键中出现。例如主键为 `(Sno, Cno)`：

```text
(Sno, Cno) -> Grade
Sno -> Sname
Sno -> Sdept
```

其中：
- `Grade` 完全依赖于 `(Sno, Cno)`
- `Sname` 和 `Sdept` 只依赖于主键的一部分 `Sno`，所以是部分依赖

### 10.4.3 传递函数依赖

第三范式基于传递依赖的概念。

若有属性 `A`、`B`、`C`，满足：

```text
A -> B
B -> C
```

并且 `B` 不能反向决定 `A`，则称 `C` 传递依赖于 `A`。

示例：

```text
Sno -> Sdept
Sdept -> Mname
```

因此：

```text
Sno -> Mname
```

这里 `Mname` 通过 `Sdept` 传递依赖于 `Sno`。如果把学生和系主任放在同一张学生表中，就会造成冗余和修改异常。

---

## 10.5 第一范式 1NF

第一范式要求关系中每一行和每一列的交叉位置只能包含一个值，也就是属性值必须是原子的，不能出现集合、重复组或嵌套表。

定义：

```text
一个关系属于 1NF，当且仅当每个属性在每个元组中都只含有一个原子值。
```

不符合 1NF 的例子：

| 学号 | 姓名 | 课程与成绩 |
|---|---|---|
| 99230 | Tom | OS:96, DB:88 |

转换为 1NF：

| 学号 | 姓名 | 课程 | 成绩 |
|---|---|---|---|
| 99230 | Tom | OS | 96 |
| 99230 | Tom | DB | 88 |

要点：
- 单元格中不能放多个值
- 多值属性要拆成多行或拆出新关系
- 1NF 是关系数据库最基本的要求

---

## 10.6 第二范式 2NF

第二范式要求关系先满足 1NF，并且每个非主属性都完全函数依赖于主键。

定义：

```text
2NF = 1NF + 消除非主属性对主键的部分函数依赖
```

适用场景：
- 当主键是单属性时，通常不会产生部分依赖
- 当主键是组合属性时，需要特别检查是否有非主属性只依赖主键的一部分

从 1NF 转换到 2NF 的步骤：

1. 确定 1NF 关系的主键
2. 找出关系中的函数依赖
3. 如果存在对主键的部分依赖，将部分依赖拆成新关系，并把决定因素复制到新关系中

示例：

```text
Student(Sno, Sname, Sdept, Mname, Cno, Grade)
主键: (Sno, Cno)

(Sno, Cno) -> Grade
Sno -> Sname
Sno -> Sdept
Sdept -> Mname
```

其中 `Sname`、`Sdept`、`Mname` 都不依赖完整主键 `(Sno, Cno)`，而是依赖 `Sno`，所以存在部分依赖。

分解为 2NF：

```text
SC(Sno, Cno, Grade)
Student(Sno, Sname, Sdept, Mname)
```

这样成绩由学生和课程共同决定，学生基本信息由学号决定。

---

## 10.7 第三范式 3NF

第三范式要求关系先满足 2NF，并且非主属性之间不能存在传递依赖。

定义：

```text
3NF = 2NF + 消除非主属性对主键的传递函数依赖
```

从 2NF 转换到 3NF 的步骤：

1. 确定 2NF 关系的主键
2. 找出关系中的函数依赖
3. 如果存在传递依赖，将传递依赖拆成新关系，并把决定因素复制到新关系中

示例：

```text
Student(Sno, Sname, Sdept, Mname)
Sno -> Sname
Sno -> Sdept
Sdept -> Mname
```

因为：

```text
Sno -> Sdept -> Mname
```

所以 `Mname` 传递依赖于 `Sno`。

分解为 3NF：

```text
Student(Sno, Sname, Sdept)
Department(Sdept, Mname)
```

或按照课件中的记法：

```text
SD(Sno, Sname, Sdept)
DL(Sdept, Sloc, Mname)
```

分解后，系主任、系位置等院系信息只在院系关系中存储一次。

---

## 10.8 Boyce-Codd 范式 BCNF

BCNF 是比 3NF 更严格的范式。它要求关系中每一个非平凡函数依赖的决定因素都必须是候选键。

定义：

```text
若关系 R 中每一个非平凡函数依赖 X -> Y 的决定因素 X 都是 R 的候选键，则 R 属于 BCNF。
```

核心判断：

```text
凡是能决定其他属性的属性集，都必须有资格唯一标识整个元组。
```

BCNF 与 3NF 的区别：
- 3NF 在某些情况下允许决定因素不是候选键
- BCNF 不允许任何非候选键作为决定因素

因此：

```text
BCNF 一定满足 3NF
3NF 不一定满足 BCNF
```

转换到 BCNF 的思路：

1. 找出所有函数依赖
2. 检查每个决定因素是否为候选键
3. 如果某个决定因素不是候选键，就按该依赖进行分解

课件要点：

```text
Boyce-Codd normal form does not allow determinants that are not candidate keys.
```

也就是 BCNF 不允许非候选键决定其他属性。

---

## 10.9 从 UNF 到 BCNF 的分解复习

课件给出的复习例子围绕房产检查记录展开，初始关系大致包含：

```text
propertyNo, iDate, iTime, pAddress, comments, staffNo, sName, carReg
```

函数依赖包括：

```text
(propertyNo, iDate) -> iTime, pAddress, comments, staffNo, sName, carReg
propertyNo -> pAddress
staffNo -> sName
(iDate, staffNo) -> carReg
(iDate, iTime, carReg) -> propertyNo, pAddress, comments, staffNo, sName
(iDate, iTime, staffNo) -> propertyNo, pAddress, comments, carReg, sName
```

### 转换到 2NF

2NF 不允许部分依赖。由于：

```text
propertyNo -> pAddress
```

而 `propertyNo` 是组合键的一部分，所以需要拆出房产关系：

```text
Property(propertyNo, pAddress)
Inspection(propertyNo, iDate, iTime, comments, staffNo, sName, carReg)
```

### 转换到 3NF

3NF 不允许传递依赖。由于：

```text
staffNo -> sName
```

检查记录中通过员工号可以确定员工姓名，因此拆出员工关系：

```text
Property(propertyNo, pAddress)
Staff(staffNo, sName)
Inspection(propertyNo, iDate, iTime, comments, staffNo, carReg)
```

### 转换到 BCNF

BCNF 不允许非候选键作为决定因素。由于：

```text
(iDate, staffNo) -> carReg
```

如果 `(iDate, staffNo)` 不是 `Inspection` 的候选键，就违反 BCNF，需要继续分解：

```text
Property(propertyNo, pAddress)
Staff(staffNo, sName)
Inspection(propertyNo, iDate, iTime, comments, staffNo)
StaffCar(staffNo, iDate, carReg)
```

这样车辆分配信息被独立出来，避免非候选键决定 `carReg` 所造成的异常。

---

## 10.10 考试速记

范式判断顺序：

```text
1NF: 属性值是否原子？
2NF: 是否消除了非主属性对组合主键的部分依赖？
3NF: 是否消除了非主属性之间的传递依赖？
BCNF: 每个非平凡依赖的决定因素是否都是候选键？
```

常见关键词：
- 冗余：同一事实重复存储
- 插入异常：没有其他相关数据时无法插入
- 删除异常：删除一条记录导致有用信息丢失
- 修改异常：同一事实需要多处同步修改
- 决定因素：函数依赖左边的属性集
- 部分依赖：依赖组合主键的一部分
- 完全依赖：依赖整个主键，不依赖任何真子集
- 传递依赖：`A -> B` 且 `B -> C`，导致 `A -> C`

做题步骤：

1. 写出关系模式和候选键
2. 列出题目给出的函数依赖
3. 判断是否满足 1NF
4. 检查是否有部分依赖，若有则分解到 2NF
5. 检查是否有传递依赖，若有则分解到 3NF
6. 检查每个决定因素是否都是候选键，若不是则继续分解到 BCNF

