# 第 02-1 模块作业整理笔记

### 2.1 基本概念

#### 题干

在关系数据模型的语境下，讨论以下概念：relation、domain、attribute、tuple、degree 和 cardinality。

#### 答案

| 概念 | 答案 |
| --- | --- |
| relation | 笛卡尔积的子集，直观表现为二维表。 |
| domain | 属性可能取值的集合或范围。 |
| attribute | 关系中的一列，表示实体或联系的一个特征。 |
| tuple | 关系中的一行，表示一条记录。 |
| degree | 关系中属性的个数。 |
| cardinality | 关系中元组的个数。 |

#### 解析

关系模型把数据组织成二维表。表名对应 relation，列对应 attribute，行对应 tuple，每列的取值范围来自 domain。

### 2.2 关系的性质

#### 题干

讨论关系模型中“关系”的性质。

#### 答案

一个关系应满足：

```text
关系名唯一
属性名唯一
属性值原子
同一属性的值来自同一域
列的顺序无关
行的顺序无关
不存在完全相同的两个元组
```

#### 解析

关系是数学集合，所以元组不能重复；关系表中的每个单元格必须是不可再分的原子值，这也是 1NF 的基础。

### 2.3 候选键、主键、外键

#### 题干

讨论关系中候选键和主键的区别；解释什么是外键，以及外键如何与被参照关系的主键关联，并举例说明。

#### 答案

- 候选键：能唯一标识元组的最小属性集，一个关系可以有多个候选键。
- 主键：从候选键中选出的一个，用于主要标识元组。
- 外键：本关系中的属性集，引用另一个关系或本关系的主键。

示例：

```text
Student(Sno, Sname)
Enrol(Sno, Cno, Mark)
```

其中 `Student.Sno` 是 Student 的主键，`Enrol.Sno` 是引用 Student 的外键。

#### 解析

候选键强调“唯一 + 最小”；主键是被选中的候选键；外键用于建立关系之间的参照联系。

### 2.4 两条完整性规则

#### 题干

定义关系模型中的两条主要完整性规则，并讨论为什么需要强制执行这些规则。

#### 答案

```text
实体完整性：主键属性不能取空值。
参照完整性：外键值必须为空，或等于被参照关系中某个已有主键值。
```

#### 解析

实体完整性保证每条记录可识别；参照完整性避免出现“引用了不存在对象”的孤立数据。

### 2.5 视图与基本关系

#### 题干

什么是视图？讨论视图和基本关系之间的区别。

#### 答案

视图是由一个或多个基本关系导出的虚拟表；基本关系实际存储数据，视图通常只保存查询定义，查询时动态生成结果。

#### 解析

视图可以简化查询、隐藏敏感字段、提供不同用户的外模式，但通常不直接保存完整数据副本。

### 2.6 Hotel 模式外键与完整性

#### 题干

给定酒店数据库模式：Hotel(hoteNo, hoteName, city)，Room(roomNo, hoteNo, type, price)，Booking(bookID, hoteNo, roomNo, guestNo, dateFrom, dataTo)，Guest(guestNo, guestName, guestAddress)。其中 Hotel 的 hoteNo 是主键，Room 的 (roomNo, hoteNo) 是主键，Booking 的 bookID 是主键，Guest 的 guestNo 是主键。识别该模式中的外键，并说明实体完整性和参照完整性如何应用到这些关系中。

#### 答案

外键：

```text
Room.hotelNo -> Hotel.hotelNo
Booking.(roomNo, hotelNo) -> Room.(roomNo, hotelNo)
Booking.guestNo -> Guest.guestNo
Booking.hotelNo -> Hotel.hotelNo
```

实体完整性：

```text
Hotel.hotelNo、Room(roomNo, hotelNo)、Booking.bookID、Guest.guestNo 不能为空且唯一。
```

参照完整性：

```text
Room 中的 hotelNo 必须存在于 Hotel。
Booking 中的 guestNo 必须存在于 Guest。
Booking 中的房间必须存在于 Room。
```

#### 解析

先找每张表的主键，再看哪些属性用于引用其他表。房间属于酒店，预订关联房间和客人，所以这些字段自然成为外键。

### 2.7 关系代数结果描述

#### 题干

根据 Hotel、Room、Booking、Guest 关系，描述以下关系代数运算会产生什么关系结果：

(a) πhotelNo(σprice>50(Room))

(b) Hotel ⋈Hotel.hotelNo=Room.hotelNo Room

(c) πhotelName,City(Hotel ⋈Hotel.hotelNo=Room.hotelNo σprice>50(Room))

(d) σdataTo≥'1-Jan-2002'(Booking) ⋈ Guest

(e) Hotel ⋉Hotel.hotelNo=Room.hotelNo σprice>50(Room)，其中 ⋉ 表示半连接。

(f) (πguestNo,hotelNo(Booking ⋈ Guest)) ÷ πhotelNo(σcity='London'(Hotel))

#### 答案

```text
a) πhotelNo(σprice>50(Room))
   查询价格大于 50 的房间所在酒店编号。

b) σHotel.hotelNo=Room.hotelNo(Hotel × Room)
   Hotel 与 Room 按酒店编号连接。

c) πhotelName, city(Hotel ⋈Hotel.hotelNo=Room.hotelNo σprice>50(Room))
   查询至少有一间房价大于 50 的酒店名称和城市。

d) σdateTo>='1-Jan-2002'(Booking) ⋈ Guest
   查询结束日期不早于 2002-01-01 的预订对应客人信息。

e) Hotel ⋉Hotel.hotelNo=Room.hotelNo σprice>50(Room)
   查询至少有一间房价大于 50 的酒店完整信息。

f) πguestNo,hotelNo(Booking ⋈ Guest) ÷ πhotelNo(σcity='London'(Hotel))
   查询预订过伦敦所有酒店的客人编号。
```

#### 解析

选择 `σ` 负责筛选行，投影 `π` 负责取列，连接 `⋈` 负责合并相关表，除法 `÷` 常用于“全部/所有”类型查询。

### 2.8 Hotel 模式查询的关系代数

#### 题干

使用关系代数表达以下查询：

(a) 列出所有酒店的完整信息。

(b) 列出所有每晚价格低于 20 美元的单人间。

(c) 列出所有客人的姓名和城市。

(d) 列出 Grosvenor Hotel 所有房间的价格和类型。

(e) 列出当前住在 Grosvenor Hotel 的客人。

(f) 列出 Grosvenor Hotel 所有房间的详细信息；如果房间已入住，还要包含入住客人的姓名。

(g) 列出所有住在 Grosvenor Hotel 的客人的 guestNo、guestName 和 guestAddress。

#### 答案

```text
a) Hotel
b) σtype='Single' AND price<20(Room)
c) πguestName, city(Guest)
d) πprice,type(σhotelName='Grosvenor Hotel'(Hotel) ⋈ Room)
e) πGuest.*(σhotelName='Grosvenor Hotel'(Hotel) ⋈ Booking ⋈ Guest)
f) πRoom.*,guestName(σhotelName='Grosvenor Hotel'(Hotel) ⋈ Room ⟕ Booking ⟕ Guest)
g) πguestNo,guestName,guestAddress(σhotelName='Grosvenor Hotel'(Hotel) ⋈ Booking ⋈ Guest)
```

#### 解析

只涉及单表时直接选择/投影；涉及酒店名、房间、客人时，需要通过 `hotelNo`、`roomNo`、`guestNo` 做连接。

---








