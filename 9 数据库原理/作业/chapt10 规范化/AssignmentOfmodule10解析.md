# 第 10 模块作业整理笔记

### 1. 牙医预约关系

#### 题干

给定一个关于牙医和病人预约数据的关系。

一个病人在某个具体日期和具体时间，被安排与一名牙医进行预约；该牙医位于某个特定诊室。对于每天的病人预约，一名牙医在当天会被分配到一个特定诊室。

关系模式如下：

```text
Appointment(staffNo, dentistName, patNo, patName, date, time, surgery)
```

要求：

1. 找出上述关系中的所有函数依赖。

2. 找出该表的主键。

3. 判断该关系属于第几范式，并解释原因。

4. 对该关系进行规范化，直到所有关系都达到 BCNF，并说明每次分解的原因。

#### 答案

原关系：

```text
Appointment(staffNo, dentistName, patNo, patName, date, time, surgery)
```

函数依赖：

```text
staffNo -> dentistName
patNo -> patName
(staffNo, date) -> surgery
(patNo, date, time) -> staffNo
(staffNo, date, time) -> patNo
```

候选键：

```text
(patNo, date, time)
(staffNo, date, time)
```

主键可选：

```text
(patNo, date, time)
```

当前范式：

```text
1NF
```

原因：每个属性都是原子值，所以满足 1NF；但存在 `patNo -> patName`、`staffNo -> dentistName` 等依赖，非主属性只依赖组合键的一部分或依赖非候选键，因此不满足 2NF，也不满足 BCNF。

规范化到 BCNF 后的关系：

```text
Dentist(staffNo, dentistName)
Patient(patNo, patName)
DentistSurgery(staffNo, date, surgery)
Appointment(patNo, date, time, staffNo)
```

各关系主键：

```text
Dentist: staffNo
Patient: patNo
DentistSurgery: (staffNo, date)
Appointment: (patNo, date, time)
```

#### 解析

先找“谁决定谁”：

```text
staffNo -> dentistName
patNo -> patName
(staffNo, date) -> surgery
(patNo, date, time) -> staffNo
(staffNo, date, time) -> patNo
```

解释：

- 一个牙医编号对应一个牙医姓名，所以 `staffNo -> dentistName`。
- 一个病人编号对应一个病人姓名，所以 `patNo -> patName`。
- 一名牙医在某一天被分配到一个诊室，所以 `(staffNo, date) -> surgery`。
- 一个病人在某个日期和时间只能预约一名牙医，所以 `(patNo, date, time) -> staffNo`。
- 一名牙医在某个日期和时间只能接诊一个病人，所以 `(staffNo, date, time) -> patNo`。

由 `(patNo, date, time)` 可以推出 `staffNo`，再推出 `dentistName` 和 `surgery`，同时 `patNo` 可以推出 `patName`，所以 `(patNo, date, time)` 是候选键。

同理，`(staffNo, date, time)` 也可以推出 `patNo`，再推出病人姓名，所以它也是候选键。

该关系当前属于 1NF，因为属性值都是原子的；但不满足 2NF，因为存在：

```text
patNo -> patName
staffNo -> dentistName
```

这些依赖没有依赖完整候选键，会造成病人姓名、牙医姓名重复存储。

继续按 BCNF 分解：凡是函数依赖左边不是候选键的，都要拆出去。

`staffNo -> dentistName` 拆为：

```text
Dentist(staffNo, dentistName)
```

`patNo -> patName` 拆为：

```text
Patient(patNo, patName)
```

`(staffNo, date) -> surgery` 拆为：

```text
DentistSurgery(staffNo, date, surgery)
```

剩下预约本身：

```text
Appointment(patNo, date, time, staffNo)
```

这样每个关系中，函数依赖左边都是该关系的候选键，因此满足 BCNF。

---

### 2. 发票销售关系

#### 题干

考虑以下关系，它表示一次商品销售发票的信息：

```text
Invoice_data(I_number, I_date, C_number, C_name, C_city, Item_number, Item_name, Item_qty, Item_price, Total price)
```

每张发票有：

- 发票编号 `I_number`
- 发票日期 `I_date`
- 一个顾客编号 `C_number`
- 所有商品的总价 `Total price`

每张发票可以列出一个或多个顾客购买的商品。发票中的每个商品有：

- 唯一商品编号 `Item_number`
- 商品名称 `Item_name`
- 商品数量 `Item_qty`
- 商品单价 `Item_price`

同一种商品可以被多个顾客购买。

每个顾客有：

- 顾客编号 `C_number`
- 顾客姓名 `C_name`
- 顾客所在城市 `C_city`

要求：

1. 找出上述关系中的所有函数依赖。

2. 找出该表的主键。

3. 判断该关系属于第几范式，并解释原因。

4. 对该关系进行规范化，直到所有关系都达到 BCNF，并说明每次分解的原因。

#### 答案

原关系：

```text
Invoice_data(I_number, I_date, C_number, C_name, C_city, Item_number, Item_name, Item_qty, Item_price, Total_price)
```

函数依赖：

```text
I_number -> I_date, C_number, Total_price
C_number -> C_name, C_city
Item_number -> Item_name, Item_price
(I_number, Item_number) -> Item_qty
```

传递依赖：

```text
I_number -> C_number -> C_name, C_city
```

主键：

```text
(I_number, Item_number)
```

当前范式：

```text
1NF
```

原因：每个属性都是原子值，所以满足 1NF；但存在 `I_number -> I_date, C_number, Total_price` 和 `Item_number -> Item_name, Item_price`，这些属性只依赖组合主键的一部分，因此不满足 2NF。

规范化到 BCNF 后的关系：

```text
Customer(C_number, C_name, C_city)
Invoice(I_number, I_date, C_number, Total_price)
Item(Item_number, Item_name, Item_price)
InvoiceLine(I_number, Item_number, Item_qty)
```

各关系主键：

```text
Customer: C_number
Invoice: I_number
Item: Item_number
InvoiceLine: (I_number, Item_number)
```

外键：

```text
Invoice.C_number references Customer.C_number
InvoiceLine.I_number references Invoice.I_number
InvoiceLine.Item_number references Item.Item_number
```

说明：`Total_price` 通常可以由发票明细中的 `Item_qty * Item_price` 汇总计算得到，因此实际数据库设计中可以不存储；但题目原关系中给出了该属性，所以这里保留在 `Invoice` 表中。

#### 解析

先找函数依赖：

```text
I_number -> I_date, C_number, Total_price
C_number -> C_name, C_city
Item_number -> Item_name, Item_price
(I_number, Item_number) -> Item_qty
```

解释：

- 一张发票编号可以确定发票日期、顾客编号和总价。
- 一个顾客编号可以确定顾客姓名和城市。
- 一个商品编号可以确定商品名称和单价。
- 某张发票中的某个商品，才能确定该商品购买数量。

主键是：

```text
(I_number, Item_number)
```

因为一张发票可以有多个商品，单独 `I_number` 不够；同一商品可以出现在多张发票中，单独 `Item_number` 也不够；两者合起来才能确定一条发票明细。

该关系当前属于 1NF，因为属性值都是原子的；但不满足 2NF，因为存在部分依赖：

```text
I_number -> I_date, C_number, Total_price
Item_number -> Item_name, Item_price
```

这些属性只依赖组合主键的一部分。

同时还存在传递依赖：

```text
I_number -> C_number -> C_name, C_city
```

所以需要继续拆表。

先把顾客信息拆出：

```text
Customer(C_number, C_name, C_city)
```

再把发票基本信息拆出：

```text
Invoice(I_number, I_date, C_number, Total_price)
```

再把商品信息拆出：

```text
Item(Item_number, Item_name, Item_price)
```

最后保留发票明细：

```text
InvoiceLine(I_number, Item_number, Item_qty)
```

分解后，每个关系中的函数依赖左边都是该关系的候选键，因此所有关系都满足 BCNF。
