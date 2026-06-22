# Chapter 02 The Relational Model and Relational Algebra 关系模型和关系代数

## 闭卷客观题复习提示
- **关系特性**：理解度、基数、域的含义，关系是无序的集合。`【闭卷重点·常考】`
- **键的区分**：从超码到候选码再到主码的演变，外码的定义及允许为空的条件。`【闭卷重点·必背】`
- **完整性规则**：实体完整性与参照完整性的具体定义。`【闭卷重点·必背】`
- **集合运算**：并、交、差需要满足的相容条件。`【闭卷重点·易错】`
- **关系代数运算**：笛卡尔积、自然连接和外连接的行为差异，除法的应用场景。`【闭卷重点·常考】`

## 目录

- [Chapter 02 The Relational Model and Relational Algebra 关系模型和关系代数](#chapter-02-the-relational-model-and-relational-algebra-关系模型和关系代数)
  - [目录](#目录)
  - [2.1 Relational Data Model 关系数据库概述](#21-relational-data-model-关系数据库概述)
    - [2.1.1 Mathematical Relation 数学关系](#211-mathematical-relation-数学关系)
    - [2.1.2 Relations in the Relational Model 关系模型中的关系](#212-relations-in-the-relational-model-关系模型中的关系)
    - [2.1.3 Notation 符号](#213-notation-符号)
    - [2.1.4 RM Terminology 关系模型术语](#214-rm-terminology-关系模型术语)
    - [2.1.5 Characteristics of Relations 关系的特性](#215-characteristics-of-relations-关系的特性)
    - [2.1.6 Missing Values 空值](#216-missing-values-空值)
    - [2.1.7 Relational Keys 关系键](#217-relational-keys-关系键)
      - [超码（Superkey）](#超码superkey)
      - [候选码（Candidate Key）](#候选码candidate-key)
      - [主码（Primary Key）](#主码primary-key)
      - [备用码（Alternate Keys）](#备用码alternate-keys)
      - [外码（Foreign Key）](#外码foreign-key)
      - [2.1.7.1 Primary Key 主码](#2171-primary-key-主码)
      - [2.1.7.2 Where to Find Keys 如何确定键](#2172-where-to-find-keys-如何确定键)
      - [2.1.7.3 Foreign Keys 外码](#2173-foreign-keys-外码)
  - [2.3 Relational Integrity 关系的完整性](#23-relational-integrity-关系的完整性)
    - [实体完整性](#实体完整性)
    - [参照完整性](#参照完整性)
    - [业务规则](#业务规则)
    - [元组约束](#元组约束)
  - [2.4 Relational Algebra 关系代数](#24-relational-algebra-关系代数)
    - [2.4.1 Set Operations 集合运算](#241-set-operations-集合运算)
      - [并（Union）](#并union)
      - [差（Set Difference）](#差set-difference)
      - [交（Intersection）](#交intersection)
    - [2.4.2 Relational Algebra SELECT 选择运算](#242-relational-algebra-select-选择运算)
    - [2.4.3 Relational Algebra PROJECT 投影运算](#243-relational-algebra-project-投影运算)
    - [2.4.4 Relational Algebra JOIN 连接运算](#244-relational-algebra-join-连接运算)
      - [1. 笛卡尔积（Cartesian Product）](#1-笛卡尔积cartesian-product)
      - [2. Theta Join $\\theta$-连接](#2-theta-join-theta-连接)
      - [3. 等值连接（Equijoin）](#3-等值连接equijoin)
      - [4. 自然连接（Natural Join）](#4-自然连接natural-join)
      - [5. 外连接（Outer Joins）](#5-外连接outer-joins)
      - [6. 半连接（Semijoin）](#6-半连接semijoin)
    - [2.4.5 Division Operation 除运算](#245-division-operation-除运算)

---

## 2.1 Relational Data Model 关系数据库概述

数据模型是对现实世界的表示，如何把现实世界的对象抽象为计算机可识别、可操作、并且是正确的数据集合，这是数据模型的根本所在。数据模型通常由数据结构、数据操作和完整性约束三部分构成。关系模型是其中使用最广泛的数据模型之一。

---

### 2.1.1 Mathematical Relation 数学关系

数学中的关系是一组有序的 $n$ 元组 $(d_1, d_2, \ldots, d_n)$，其中 $d_1 \in D_1, d_2 \in D_2, \ldots, d_n \in D_n$。

关系是一组有序的 $n$ 元组。

**示例：足球比赛结果**

考虑足球比赛结果，它是 $A \times A \times B \times B$ 的子集，其中 $A = \{Malaysia, Australia, Italy, Greece\}$，$B$ 是所有非负整数的集合。

每个域被使用了两次（有两个角色），通过列位置来区分：
- 第1列：主队（Home team）
- 第2列：客队（Visiting team）
- 第3列：主队进球数
- 第4列：客队进球数

| HomeTeam | VisitingTeam | HomeGoals | VisitingGoals |
|----------|-------------|-----------|---------------|
| Malaysia | Australia | 3 | 2 |
| Australia | Italy | 0 | 2 |
| Malaysia | Greece | 1 | 2 |
| Greece | Italy | 1 | 0 |

---

### 2.1.2 Relations in the Relational Model 关系模型中的关系

| HomeTeam | VisitingTeam | HomeGoals | VisitingGoals |
|----------|-------------|-----------|---------------|
| Malaysia | Australia | 3 | 2 |
| Australia | Italy | 0 | 2 |
| Malaysia | Greece | 1 | 2 |
| Greece | Italy | 1 | 0 |

---

### 2.1.3 Notation 符号

---

### 2.1.4 RM Terminology 关系模型术语

属性（Attribute）是关系中的命名列，用于描述数据的某一特征。

**域（Domain）**：一组具有相同数据类型的值的集合，又称为值域（用 $D$ 表示）。

示例：
- 整数
- 实数
- 介于某个取值范围的整数
- 指定长度的字符串集合
- $\{'男', '女'\}$

域中所包含的值的个数称为域的基数（用 $m$ 表示）。

**单元格（Cell）**：表中的单个数据项。

**元组（Tuple）/ 行（Row）**：表中的一行数据。

**度（Degree）**：关系中属性的个数。

**基数（Cardinality）**：关系中元组的个数。

**替代术语对照表：**

| 正式术语 | 替代术语 |
|---------|---------|
| Relation | Table |
| Tuple | Row / Record |
| Attribute | Column / Field |

---

### 2.1.5 Characteristics of Relations 关系的特性

关系的度（degree）指的是关系中的**列数**（the number of columns）。

关系的特性：`【闭卷重点·必背】`
- 每个属性有唯一的名称（distinct name）
- 每个属性有特定的数据类型（distinct data type）
- 行的顺序没有意义（Order of rows has no significance）
- 列的顺序没有意义（Order of columns has no significance）

---

### 2.1.6 Missing Values 空值

空值（Null value）表示属性值当前未知或不适用（currently unknown or not applicable）。

以上表格中有些课程的 LECTURER 取空值，其含义是：该课程目前没有指定讲师，或讲师信息未知。

空值表示属性值当前未知或不适用，不是零、不是空集、也不是空格。

---

### 2.1.7 Relational Keys 关系键

#### 超码（Superkey）

超码可能包含多余的属性，但仍能满足唯一标识的要求。例如，$(customerID, name)$ 可能是超码，但仅 $customerID$ 就足以唯一标识一行。

#### 候选码（Candidate Key）

候选码是最小的超码。`【闭卷重点·必背】`

#### 主码（Primary Key）

候选码中被选中用于在表内唯一标识记录的键。`【闭卷重点·必背】`

#### 备用码（Alternate Keys）

未被选为主码的候选码。

#### 外码（Foreign Key）

一个表中的列或列集合，匹配某个（可能是同一个）表的主码。

**键约束满足示例：**

给定关系实例和键 studentID 和 emailID，关系满足键约束，因为元组具有不同的 studentID 值和不同的 emailID 值。

**键违反示例：**

给定不同的关系实例和键 studentID 和 emailID，关系违反了键约束。键可能以多种不同方式被违反（如输入错误）。

**提醒**：约束是"现实世界"的事实，对于数据库中每一个可能的实例都必须为真。

---

#### 2.1.7.1 Primary Key 主码

**简单主码（Simple Primary Key）**：由单个属性组成的主码。

**组合主码（Concatenated Primary Key）**：由多个属性组合而成的主码。

---

#### 2.1.7.2 Where to Find Keys 如何确定键

键应从应用程序中确定，而不是从表的数据实例中确定。

例如，在 Student 表中，我们知道每个学生有唯一的 ID 和不同的 email ID，因此 studentID 和 emailID 是两个候选码。相比之下，学生姓名和出生日期的组合不能作为键。

关系是集合——由不同的元组组成。整个属性集合总是一个超码。每个关系都有一个超码，因此也有一个候选码。要么整个属性集合是键（最小超码），要么存在一个更小的集合也是超码。重复这个论证，最终一定能找到最小超码。

键保证每条数据可以被唯一访问。

---

#### 2.1.7.3 Foreign Keys 外码

外码可以被视作是主码（键）的一个"副本"，这个副本从一个关系（表）中导出并导入到另一个关系（表）中，以此来表示它们之间存在的一种关系。

如果主键是由多个字段组合而成的复合码，那么外键也将包含码的全部字段。

**外码与参照完整性：**

外码只能取与其父级主码中的有效值相匹配的值，或者（可能）为空。

**外码与主码的区别：**

- 与主码不同，在大多数情况下，一个关系中可以有多个元组具有相同的外码值——外码值（通常）不必是唯一的
- 外码也可以（通常）为空——但是，组合外码不能有些属性为空而其他属性为非空

**示例关系模式：**

```
contact(Lecturer, Course, Hours)
enrol(Number, Code, Mark)
course(Code, Name, Lecturer)
student(Number, Name, DOB, Address, Telephone, Gender, Degree)
lecturer(Name, School, Address, Telephone, Title)
```

其中：
- student(Number, ...) 的 Number 是主码
- course(Code, ...) 的 Code 是主码
- enrol(Number, Code, Mark) 中 Number 和 Code 是组合主码，同时 Number 和 Code 分别是外码，引用 student 和 course
- contact(Lecturer, Course, Hours) 中 Lecturer 和 Course 是组合主码，同时是外码

**餐厅收据示例：**

考虑一个餐厅收据，包含：
- 收据编号和日期
- 购买的物品，每项由数量、描述和费用组成
- 总费用

表示为两个关系：
- Receipts(receiptNo, date, total)
- Details(receiptNo, quantity, description, cost)

其中 receiptNo 是 Receipts 的主码，也是 Details 的外码。

如果存在重复行，可以添加行号作为组合键的一部分：Details(receiptNo, line, quantity, description, cost)。

---

## 2.3 Relational Integrity 关系的完整性

完整性约束（Integrity constraint）是所有有意义的数据库实例都必须满足的属性。满足所有完整性约束的数据库实例是合法的（legal）。但合法的数据库中仍可能存在错误。

完整性约束的好处：
- 更详细地描述应用程序
- 有助于数据质量和更好的数据库设计（规范化）
- 帮助系统优化查询处理

三种完整性约束：
1. **实体完整性（Entity Integrity）**
2. **参照完整性（Referential Integrity）**
3. **业务规则（Business Rules）/ 用户自定义完整性**

### 实体完整性

在基本表中，主码的任何列都不能为空。主属性不能为空。`【闭卷重点·必背】`

### 参照完整性

如果表中存在外码，则外码值要么与主表中某个记录的候选码值匹配，要么为空。`【闭卷重点·易错】`

### 业务规则

定义或约束组织某些方面的规则。

**数据缺乏完整性的示例：**

假设在 Exams-Courses 数据库中：
- 有效成绩在 A 到 F 之间
- 荣誉只能在成绩为 A 时授予
- 学生必须有唯一的注册号
- 考试必须引用现有课程

缺乏完整性约束会导致一系列问题。

### 元组约束

表达单个元组内属性值的条件：
- 描述单个属性的域：$(Grade \geq 'A') \text{ AND } (Grade \leq 'F')$
- 描述属性之间的关系：$(Honours = 'no') \text{ OR } (Grade = 'A')$
- 数值计算：$Net = Amount - Deductions$

---

## 2.4 Relational Algebra 关系代数

关系代数是一种形式语言，它指定了当从关系中提取数据时可以对关系执行的操作。

---

### 2.4.1 Set Operations 集合运算

集合运算是关系代数的二元操作，包括四种：

#### 并（Union）

$R \cup S$

两个关系 $R$ 和 $S$ 的并定义了一个包含 $R$ 或 $S$ 或两者所有元组的关系，重复元组被消除。

$$R \cup S = \{t \mid t \in R \lor t \in S\}$$

$R$ 和 $S$ 必须是并兼容（Union-compatible）的：
- 两个关系必须具有相同数量的属性
- 每一对相对应的属性必须具有相同的域（即数据类型和格式相同）

#### 差（Set Difference）

$R - S$

集合差定义了一个由在关系 $R$ 中但不在 $S$ 中的元组组成的关系。

$R$ 和 $S$ 必须是并兼容的。`【闭卷重点·易错】`

$$R - S = \{t \mid t \in R \land t \notin S\}$$

#### 交（Intersection）

$R \cap S$

交定义了一个由同时在关系 $R$ 和 $S$ 中的元组组成的关系。

$R$ 和 $S$ 必须是并兼容的。

$$R \cap S = \{t \mid t \in R \land t \in S\}$$

**示例：**

City1:

| City_no | city |
|---------|------|
| 01 | 北京 |
| 02 | 天津 |

City2:

| City_no | city |
|---------|------|
| 02 | 天津 |
| 03 | 上海 |
| 06 | 西安 |

---

### 2.4.2 Relational Algebra SELECT 选择运算

选择运算符 $\sigma$ 根据选择条件筛选行，改变表的行数。

$$\sigma_{sdept="IS"}(S)$$

**示例：**

关系 $S$：

| sno | sname | sdept | ssex | sage |
|-----|-------|-------|------|------|
| 95001 | 李勇 | CS | 男 | 20 |
| 95002 | 刘晨 | IS | 女 | 19 |
| 95003 | 王名 | MA | 女 | 18 |
| 95004 | 张立 | IS | 男 | 19 |

$\sigma_{sdept="IS"}(S)$ 的结果：

| sno | sname | sdept | ssex | sage |
|-----|-------|-------|------|------|
| 95002 | 刘晨 | IS | 女 | 19 |
| 95004 | 张立 | IS | 男 | 19 |

**更多示例：**
- 列出工资大于 10000 的所有员工：$\sigma_{salary>10000}(Staff)$
- 列出分支 'B003' 的所有员工：$\sigma_{branchNo='B003'}(Staff)$

---

### 2.4.3 Relational Algebra PROJECT 投影运算

投影运算符 $\Pi$ 选择特定的列，改变表的列数。

$$\Pi_{sno,sname}(S)$$

**示例：**

$\Pi_{sno,sname}(S)$ 的结果：

| sno | sname |
|-----|-------|
| 95001 | 李勇 |
| 95002 | 刘晨 |
| 95003 | 王名 |
| 95004 | 张立 |

**注意**：在做投影运算时，去掉某些列的同时，可能会得到含有重复行的结果表。此时需删除重复的行（集合不允许重复值）。

**示例**：求已经开始招生的系的名称。

$$\Pi_{sdept}(S)$$

投影运算不仅改变表的列数，还可能改变行数。

**组合示例：**

1. 查询信息系的小于19岁的女生信息：
$$\Pi_{sno,sname}(\sigma_{sdept='IS' \land sage<19 \land ssex='女'}(SC))$$

2. 查询数据库这门课的先行课号：
$$\Pi_{cpno}(\sigma_{cname='数据库'}(course))$$

3. 查询没有选1号课程的学生的学号：
$$\Pi_{sno}(S) - \Pi_{sno}(\sigma_{cno='1'}(SC))$$

4. 列出工资大于12000的员工，只显示 staffNo, fName, lName, branchNo：
$$\Pi_{staffNo, fName, lName, branchNo}(\sigma_{salary>12000}(Staff))$$

5. 列出伦敦城市中房间数大于3的房产详情：
$$\Pi_{street, postcode, ownerNo}(\sigma_{city='London' \land rooms>3}(PropertyForRent))$$

---

### 2.4.4 Relational Algebra JOIN 连接运算

连接操作接受两个关系作为输入，将两个输入关系的元组组合在一起。连接操作是关系数据库模型背后的真正力量。

连接的形式包括：Theta join, Equijoin, Natural join, Outer join, Semijoin。

#### 1. 笛卡尔积（Cartesian Product）

笛卡尔积接受两个关系 $R$ 和 $S$ 作为输入，输出一个包含 $R$ 的每个元组与 $S$ 的每个元组所有可能拼接的关系。

$$R \times S$$

笛卡尔积本身并不是很有用！

**示例：**

$R$（SUPERVISOR）：

| SUPERVISOR |
|------------|
| 张清玫 |
| 刘逸 |

$S$（SPECIALITY）：

| SPECIALITY |
|------------|
| 计算机 |
| 信息 |

$K$（POSTGRADUATE）：

| POSTGRADUATE |
|--------------|
| 李勇 |
| 刘晨 |
| 王明 |

实际情况是 $R \times S \times K$，没有实际意义。

#### 2. Theta Join $\theta$-连接

先选择再连接。

#### 3. 等值连接（Equijoin）

当谓词 $F$ 只包含等号（$=$）时，使用等值连接。

#### 4. 自然连接（Natural Join）

自然连接是在公共属性上的等值连接，并去掉重复属性。自然连接的定义已经指明了连接条件。`【闭卷重点·常考】`

**示例：**

设关系 $R$ 和 $S$ 分别为：

$R$：

| A | B | C |
|---|---|---|
| a1 | b1 | 5 |
| a1 | b2 | 6 |
| a2 | b3 | 8 |
| a2 | b4 | 12 |

$S$：

| B | E |
|---|---|
| b1 | 3 |
| b2 | 7 |
| b3 | 10 |
| b3 | 2 |
| b5 | 2 |

Theta join $R \bowtie_{C<E} S$ 的结果（表 c）：

| A | R.B | C | S.B | E |
|---|-----|---|-----|---|
| a1 | b1 | 5 | b2 | 7 |
| a1 | b1 | 5 | b3 | 10 |
| a1 | b2 | 6 | b2 | 7 |
| a1 | b2 | 6 | b3 | 10 |
| a2 | b3 | 8 | b3 | 10 |

等值连接 $R \bowtie_{R.B=S.B} S$ 的结果（表 d）：

| A | R.B | C | S.B | E |
|---|-----|---|-----|---|
| a1 | b1 | 5 | b1 | 3 |
| a1 | b2 | 6 | b2 | 7 |
| a2 | b3 | 8 | b3 | 10 |
| a2 | b3 | 8 | b3 | 2 |

自然连接 $R \bowtie S$ 的结果（表 e）：

| A | B | C | E |
|---|---|---|---|
| a1 | b1 | 5 | 3 |
| a1 | b2 | 6 | 7 |
| a2 | b3 | 8 | 10 |
| a2 | b3 | 8 | 2 |

#### 5. 外连接（Outer Joins）

保留了在关系中没有匹配的元组。

**左外连接（Left Outer Join）**：左表的元组全部保留，右表相应属性用空值填充。

**右外连接（Right Outer Join）**：右表的元组全部保留。

**全外连接（Full Outer Join）**：左右两表未匹配元组全部保留。

#### 6. 半连接（Semijoin）

$R \ltimes_F S$

半连接定义了一个包含参与 $R$ 与 $S$ 连接的 $R$ 元组的关系。半连接的结果仅包含关系 $R$ 中那些能够与关系 $S$ 成功连接的元组。

作用：半连接不返回完整的连接结果，而是仅返回 $R$ 中符合条件的元组，通常用于优化查询或减少数据传输。

例如，如果 $R$ 是学生表，$S$ 是选课表，半连接操作会返回所有选过课的学生记录，但不包含选课的具体信息。

---

### 2.4.5 Division Operation 除运算

从关系 $R$ 中筛选出与关系 $S$ 中所有元组都匹配的元组。

$$R \div S$$

除操作是同时从行和列角度进行运算。

**示例**：查询选修了全部课程的学生号码。

设 $R(X)$ 为学号，$R(Y)$ 为选修的课程号，$S(Y)$ 为全部课程号：

$$\Pi_{Sno,Cno}(SC) \div \Pi_{Cno}(Course)$$

查询选修了全部课程的学生号码和姓名：

$$(\Pi_{Sno,Cno}(SC) \div \Pi_{Cno}(Course)) \bowtie \Pi_{Sno,Sname}(Student)$$

除的语义：满足某种条件下的集合之间的包含关系。