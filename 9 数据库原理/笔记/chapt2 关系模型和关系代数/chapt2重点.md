# Chapter 02 关系模型与关系代数：重点

本章分成两部分复习：

1. **概念部分**：关系模型术语、键、完整性、视图、关系代数符号。
2. **做题算法部分**：如何把文字题翻译成关系代数表达式。

## 目录

- [第一部分：概念](#第一部分概念)
  - [1. 关系模型术语](#1-关系模型术语)
  - [2. 关系的性质](#2-关系的性质)
  - [3. 键](#3-键)
  - [4. 完整性规则](#4-完整性规则)
  - [5. 视图与基本关系](#5-视图与基本关系)
  - [6. 关系代数符号](#6-关系代数符号)
  - [7. 关系代数运算注意点](#7-关系代数运算注意点)
- [第二部分：做题算法](#第二部分做题算法)
  - [1. 总算法：文字题翻译成关系代数](#1-总算法文字题翻译成关系代数)
  - [2. 算法 A：单表条件查询](#2-算法-a单表条件查询)
  - [3. 算法 B：多表连接查询](#3-算法-b多表连接查询)
  - [4. 算法 C：“没有/不/未”题用差集](#4-算法-c没有不未题用差集)
  - [5. 算法 D：“所有/全部/每个”题用除法](#5-算法-d所有全部每个题用除法)
  - [6. 算法 E：自连接与更名](#6-算法-e自连接与更名)
  - [7. 算法 F：外连接题](#7-算法-f外连接题)
  - [8. 算法 G：集合运算计算题](#8-算法-g集合运算计算题)
  - [9. Hotel 模式做题路径](#9-hotel-模式做题路径)
  - [10. 大学模式做题路径](#10-大学模式做题路径)
  - [11. 电影模式做题路径](#11-电影模式做题路径)
  - [12. 图书馆模式做题路径](#12-图书馆模式做题路径)
  - [13. 考前速记](#13-考前速记)

---

# 第一部分：概念

## 1. 关系模型术语

| 英文 | 中文 | 数据库直观对应 |
| --- | --- | --- |
| Relation | 关系 | 表 |
| Tuple | 元组 | 行/记录 |
| Attribute | 属性 | 列/字段 |
| Domain | 域 | 某列允许的取值集合 |
| Degree | 度 | 列数 |
| Cardinality | 基数 | 行数 |

关系是笛卡尔积的子集，直观表现为二维表。

## 2. 关系的性质

必须掌握：
- 关系名唯一。
- 属性名唯一。
- 同一属性的值来自同一域。
- 每个属性值必须是原子的。
- 行的顺序无关。
- 列的顺序无关。
- 不存在完全相同的两个元组。

易错点：
- 关系是集合，所以元组不能重复。
- 属性值原子性也是 1NF 的基础。

## 3. 键

| 键 | 定义 |
| --- | --- |
| Superkey 超码 | 能唯一标识元组的属性集，可以包含多余属性。 |
| Candidate Key 候选码 | 能唯一标识元组且没有多余属性的最小超码。 |
| Primary Key 主码 | 从候选码中选出的主要标识。 |
| Alternate Key 备用码 | 未被选为主码的候选码。 |
| Foreign Key 外码 | 本关系中的属性集，引用另一个关系或本关系的候选码/主码。 |

关键词：

```text
候选码 = 唯一 + 最小
主码 = 被选中的候选码
外码 = 用来引用其他关系的键
```

## 4. 完整性规则

| 完整性 | 规则 | 作用 |
| --- | --- | --- |
| 实体完整性 | 主码属性不能取空值 | 保证每一行可识别 |
| 参照完整性 | 外码要么为空，要么等于被参照关系中已有主码值 | 防止引用不存在的数据 |
| 用户定义完整性 | 业务约束，如价格大于 0、性别取值范围 | 保证业务合理性 |

找外键的方法：
1. 先标出每张表的主键。
2. 看哪些字段用于引用另一张表的主键。
3. 多属性主键被引用时，外键也常是多属性组合。

例：

```text
Room(roomNo, hotelNo) 的主键是 (roomNo, hotelNo)
Booking(roomNo, hotelNo) 可作为外键引用 Room(roomNo, hotelNo)
```

## 5. 视图与基本关系

| 对象 | 特点 |
| --- | --- |
| 基本关系 Base Relation | 实际存储数据的表。 |
| 视图 View | 由一个或多个基本关系导出的虚拟表，通常只保存定义。 |

视图作用：
- 简化复杂查询。
- 隐藏敏感字段。
- 给不同用户提供不同外模式。
- 提高逻辑数据独立性。

## 6. 关系代数符号

| 符号 | 名称 | SQL 对应 | 用法 |
| --- | --- | --- | --- |
| `σ` | 选择 Select | `WHERE` | 按行过滤 |
| `π` | 投影 Project | `SELECT 列` | 取指定列，结果去重 |
| `×` | 笛卡尔积 | 无直接常用写法 | 两表所有行组合 |
| `⋈` | 连接 Join | `JOIN ... ON` | 按条件合并表 |
| 自然连接 | Natural Join | `NATURAL JOIN` | 自动按同名属性连接 |
| `∪` | 并 | `UNION` | 合并两个并相容关系 |
| `∩` | 交 | `INTERSECT` | 取共有元组 |
| `-` | 差 | `EXCEPT` / `NOT EXISTS` | “没有/不属于” |
| `÷` | 除 | 双重 `NOT EXISTS` | “所有/全部” |
| `ρ` | 更名 | `AS` | 自连接或消除重名 |
| `⟕` | 左外连接 | `LEFT JOIN` | 保留左表全部元组 |
| `⋉` | 半连接 | `EXISTS` | 只保留左表匹配行 |

## 7. 关系代数运算注意点

集合运算 `∪`、`∩`、`-` 要求两个关系并相容：
- 属性个数相同。
- 对应属性的域兼容。

其他易错点：
- 投影 `π` 会自动去重。
- 自然连接会把同名属性合并；如果同名属性不是连接条件，容易误连。
- 外连接用于“即使没有匹配记录也要显示”。
- 半连接只返回左表中能匹配上的行，不带右表字段。

---

# 第二部分：做题算法

## 1. 总算法：文字题翻译成关系代数

看到关系代数题，按这个顺序做：

1. **找最终输出列**：题目问“列出什么”，最外层通常写 `π输出列(...)`。
2. **找筛选条件**：题目中的“CS 系”“价格大于 50”“2021 年”等，写成 `σ条件(...)`。
3. **找需要的表**：输出列和筛选条件分别在哪些表里。
4. **找连接路径**：按主键/外键把表连起来。
5. **判断特殊关键词**：
   - “没有/不/未”：优先用差集 `-`。
   - “所有/全部/每个”：优先用除法 `÷`。
   - “自己和自己比较”：用更名 `ρ` 做自连接。
   - “即使没有也显示”：用外连接 `⟕`。
6. **从内到外组装**：先筛选和连接，最后投影。

通用结构：

```text
π最终列(σ筛选条件(表1 ⋈ 表2 ⋈ 表3))
```

## 2. 算法 A：单表条件查询

适用特征：
- 输出列和条件都在同一张表。

做法：

```text
π输出列(σ条件(表))
```

例：列出 CS 系且性别为女的学生学号和姓名。

```text
πSno,Sname(σSdept='CS' AND Gender='female'(student))
```

例：列出价格低于 20 的单人间。

```text
σtype='Single' AND price<20(Room)
```

## 3. 算法 B：多表连接查询

适用特征：
- 输出列或条件分散在多张表。
- 题目中出现“某学生选修的课”“某酒店的房间”“某教师讲授的课程”。

做法：
1. 先列出涉及的表。
2. 用主键/外键连接。
3. 加 `σ` 筛选。
4. 最外层用 `π` 取结果列。

模板：

```text
π输出列(σ条件(表1 ⋈ 表2 ⋈ 表3))
```

例：列出选修 c02 且成绩大于 85 的学生学号、姓名和院系。

```text
πSno,Sname,Sdept(σCno='c02' AND Mark>85(student ⋈ enrol))
```

例：按课程号和课程名列出由教授讲授的课程。

```text
πCno,Cname(σTitle='Professor'(lecturer ⋈ contact ⋈ course))
```

## 4. 算法 C：“没有/不/未”题用差集

适用特征：
- 题目出现“没有”“不包含”“未参加”“没有任何记录”。

做法：

```text
全集 - 已经满足某条件的集合
```

例：没有讲授任何课程的教师编号。

```text
πTno(lecturer) - πTno(contact)
```

例：没有任何演员参演的电影标题。

```text
πTitle(Films) - πTitle(Films ⋈ Roles)
```

例：没有借书的读者姓名。

```text
πName(Borrower) - πName(Borrower ⋈ BookLoans)
```

## 5. 算法 D：“所有/全部/每个”题用除法

适用特征：
- 题目出现“所有”“全部”“每个”。
- 本质是：某个对象和目标集合里的每个元素都有关联。

做法：

```text
对象-目标关系 ÷ 目标全集
```

第一步：找“对象和目标的关联表”。

```text
π对象,目标(关联表)
```

第二步：找“必须全部覆盖的目标全集”。

```text
π目标(σ条件(目标表))
```

第三步：相除。

```text
π对象,目标(关联表) ÷ π目标(目标全集)
```

例：参演了所有 2021 年电影的演员。

```text
(πActorID,FilmID(Roles)) ÷ (πFilmID(σYear=2021(Films)))
```

再与 `Artists` 连接取演员姓名：

```text
πFirstName,Surname(((πActorID,FilmID(Roles)) ÷ (πFilmID(σYear=2021(Films)))) ⋈ Artists)
```

例：借过 Tom 借过的所有书的读者。

```text
(πCardNo,BookID(BookLoans)) ÷
πBookID(σName='Tom'(Borrower ⋈ BookLoans))
```

再与 `Borrower` 连接取姓名。

## 6. 算法 E：自连接与更名

适用特征：
- 同一张表要当成两张表比较。
- 题目出现“同一电影中是否存在不同性别演员”“员工管理另一名员工”等。

做法：
1. 用 `ρA(表)` 和 `ρB(表)` 复制两份逻辑表。
2. 在两份表之间写比较条件。
3. 必要时用差集排除不合格对象。

例：导演同时也是演员的电影标题。

```text
πTitle(σDirector=ActorID(Films ⋈ Roles))
```

例：演员性别全部相同的电影。

思路：
1. 先求所有电影。
2. 找出存在不同性别演员的电影。
3. 用所有电影减去这些电影。

```text
πTitle(Films) -
πTitle(σA1.FilmID=A2.FilmID AND A1.Sex<>A2.Sex
       ((Roles ⋈ActorID=ArtistID ρA1(Artists)) ×
        (Roles ⋈ActorID=ArtistID ρA2(Artists))) ⋈ Films)
```

## 7. 算法 F：外连接题

适用特征：
- 题目出现“列出所有 A；如果有 B，也显示 B”。
- 没有匹配时，A 也要保留。

做法：

```text
A ⟕ B
```

例：列出 Grosvenor Hotel 所有房间的详细信息；如果房间已入住，还要包含入住客人的姓名。

```text
πRoom.*,guestName(σhotelName='Grosvenor Hotel'(Hotel ⋈ Room ⟕ Booking ⟕ Guest))
```

## 8. 算法 G：集合运算计算题

适用特征：
- 题目给出具体关系 R、S，让你算表达式结果。

做法：
- `R ∪ S`：合并 R 和 S 的元组，去重。
- `R ∩ S`：只保留两边都有的元组。
- `R - S`：保留在 R 中但不在 S 中的元组。
- `πA,B(R)`：只取 A、B 列，重复行去掉。
- `σ条件(R)`：只保留满足条件的行。
- `R × S`：R 每行和 S 每行两两组合。

注意：
```text
投影后一定要去重。
并、交、差必须关系并相容。
```

## 9. Hotel 模式做题路径

```text
Hotel(hotelNo, hotelName, city)
Room(roomNo, hotelNo, type, price)
Booking(bookID, hotelNo, roomNo, guestNo, dateFrom, dateTo)
Guest(guestNo, guestName, guestAddress)
```

常用连接路径：
- 酒店找房间：`Hotel ⋈ Room`，按 `hotelNo`。
- 预订找客人：`Booking ⋈ Guest`，按 `guestNo`。
- 预订找房间：`Booking ⋈ Room`，按 `(roomNo, hotelNo)`。
- 酒店、房间、预订、客人全链路：`Hotel ⋈ Room ⋈ Booking ⋈ Guest`。

典型题：

```text
πhotelName,city(Hotel ⋈ σprice>50(Room))
```

表示至少有一间房价大于 50 的酒店名称和城市。

```text
πguestNo,hotelNo(Booking) ÷ πhotelNo(σcity='London'(Hotel))
```

表示预订过伦敦所有酒店的客人。

## 10. 大学模式做题路径

```text
student(Sno, ...)
lecturer(Tno, ...)
course(Cno, ...)
contact(Tno, Cno, Hours)
enrol(Sno, Cno, Mark)
```

常用连接路径：
- 学生选课：`student ⋈ enrol`，按 `Sno`。
- 课程选课：`course ⋈ enrol`，按 `Cno`。
- 教师授课：`lecturer ⋈ contact ⋈ course`，按 `Tno`、`Cno`。

典型题：

```text
πSno,Sname,Sdept(σCno='ACSC7101'(student ⋈ enrol))
```

```text
πCno,Cname(σTitle='Professor'(lecturer ⋈ contact ⋈ course))
```

## 11. 电影模式做题路径

```text
Films(FilmID, Title, Director, Year, ProductionCost)
Artists(ArtistID, Surname, FirstName, Sex, BirthDate, Nationality)
Roles(FilmID, ActorID, Character)
```

常用连接路径：
- 电影找演员：`Films ⋈ Roles ⋈ Artists`。
- 导演也是艺术家：`Films.Director` 对应 `Artists.ArtistID`。
- 演员编号：`Roles.ActorID` 对应 `Artists.ArtistID`。

典型题：

```text
πTitle(σFirstName='Henry' AND Surname='Fonda'(Artists)
       ⋈ArtistID=ActorID Roles ⋈ Films)
```

```text
πTitle(σDirector=ActorID(Films ⋈ Roles))
```

## 12. 图书馆模式做题路径

```text
Book(BookID, Title, PubID)
Author(BookID, AuthorName, order)
Publisher(PubID, PubName, Address, Phone)
BookCopies(BookID, BranchID, NCopies)
BookLoans(BookID, BranchID, CardNo, DateOut, DueDate, DateReturn)
Branch(BranchID, BranchName, Address)
Borrower(CardNo, Name, Address, Phone)
```

常用连接路径：
- 图书馆藏：`Book ⋈ BookCopies ⋈ Branch`。
- 借书读者：`Book ⋈ BookLoans ⋈ Borrower`。
- 作者图书馆藏：`Author ⋈ Book ⋈ BookCopies ⋈ Branch`。

典型题：

```text
πName(Borrower) - πName(Borrower ⋈ BookLoans)
```

```text
πTitle,NCopies(σAuthorName='Stephen King' AND BranchName='Central'
               (Author ⋈ Book ⋈ BookCopies ⋈ Branch))
```

## 13. 考前速记

```text
概念：
Relation=表，Tuple=行，Attribute=列，Domain=取值范围
Degree=列数，Cardinality=行数
候选键 = 唯一 + 最小
主键不能为空
外键必须引用已存在主键或为空

算法：
行过滤 σ
列筛选 π
表合并 ⋈
没有/不 -
所有/全部 ÷
自己比自己 ρ
保留无匹配 ⟕
投影会去重
```
