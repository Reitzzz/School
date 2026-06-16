# 第 03-3 模块作业整理笔记

> **本节作业对应 `chapt3重点.md` 的知识点说明：**
> * **属于全方位的综合练习，几乎覆盖了重点笔记的整个流程**：
>   * **“1 第一阶段：DDL”** 与 **“2 第二阶段：DML更新”**：包含建表 (`CREATE TABLE`) 及约束、数据的增删改 (`INSERT/UPDATE/DELETE`)。
>   * **“3 第三阶段：核心查询”** 与 **“4 第四阶段：高级扩展”**：基础查询、聚合统计、多表外连接 (`LEFT JOIN`)、复杂子查询及 `EXISTS` 谓词。
>   * **“5 第五阶段：视图”**：考察了创建视图 (`CREATE VIEW`) 保存复杂查询结果。
> * *(未涉及：集合运算及权限控制。)*

### 7.1 简单查询

#### 题干

给定酒店数据库关系：

英文版关系：

```text
Hotel(hotelNo, hotelName, city)
Room(roomNo, hotelNo, type, price)
Booking(hotelNo, guestNo, dateFrom, dateTo, roomNo)
Guest(guestNo, guestName, guestAddress)
```

中文版关系：

```text
酒店(酒店号, 酒店名称, 城市)
房间(房间号, 酒店号, 房间类型, 价格)
预订(酒店号, 客人号, 入住日期, 离店日期, 房间号)
客人(客人号, 客人姓名, 客人地址)
```

完成以下简单查询：

1. 列出所有酒店的完整信息。

2. 列出 London 所有酒店的完整信息。

3. 按姓名字母序列出住在 London 的所有客人姓名和地址。

4. 按价格升序列出所有每晚价格低于 40 美元的 double 或 family 房间。

5. 列出没有指定 dataTo 的预订记录。

#### 答案

```sql
SELECT * FROM Hotel;

SELECT * FROM Hotel
WHERE city = 'London';

SELECT guestName, guestAddress
FROM Guest
WHERE guestAddress LIKE '%London%'
ORDER BY guestName ASC;

SELECT *
FROM Room
WHERE type IN ('Double', 'Family') AND price < 40.00
ORDER BY price ASC;

SELECT *
FROM Booking
WHERE dateTo IS NULL;
```

#### 解析

简单查询通常只需要 `SELECT ... FROM ... WHERE ...`。排序用 `ORDER BY`，空值判断必须用 `IS NULL`。

### 7.2 聚合函数

#### 题干

使用聚合函数完成以下查询：

6. 一共有多少家酒店？

7. 房间的平均价格是多少？

8. 所有 double 房间每晚总收入是多少？

9. 有多少不同的客人在 8 月有预订？

#### 答案

```sql
SELECT COUNT(*) AS totalHotels FROM Hotel;

SELECT AVG(price) AS averagePrice FROM Room;

SELECT SUM(price) AS totalRevenue
FROM Room
WHERE type = 'Double';

SELECT COUNT(DISTINCT guestNo) AS guestsInAugust
FROM Booking
WHERE MONTH(dateFrom) = 8 OR MONTH(dateTo) = 8;
```

#### 解析

`COUNT(*)` 统计行数，`AVG` 求平均，`SUM` 求总和，`COUNT(DISTINCT ...)` 去重统计。

### 7.3 连接与子查询

#### 题干

使用连接和子查询完成以下查询：

10. 列出 Grosvenor Hotel 所有房间的价格和类型。

11. 列出当前住在 Grosvenor Hotel 的所有客人。

12. 列出 Grosvenor Hotel 所有房间的详细信息；如果房间已入住，还要包含入住客人的姓名。

13. 查询 Grosvenor Hotel 今天来自预订的总收入。

14. 列出 Grosvenor Hotel 当前未入住房间。

15. 查询 Grosvenor Hotel 当前未入住房间造成的收入损失。

16. 查询每家酒店的房间数量。

17. 查询 London 每家酒店的房间数量。

18. 查询 8 月每家酒店的平均预订数量。（选做）

19. 查询 London 每家酒店最常被预订的房间类型。（选做）

20. 查询今天每家酒店未入住房间造成的收入损失。

21. 查询预订过 London 所有酒店的客人姓名。

22. 查询被每位客人都预订过的酒店。

23. 查询被每位来自 China 的客人都预订过的酒店。

#### 答案

```sql
-- Grosvenor Hotel 的房价和类型
SELECT r.price, r.type
FROM Room r
JOIN Hotel h ON r.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel';

-- 当前住在 Grosvenor Hotel 的客人
SELECT DISTINCT g.*
FROM Guest g
JOIN Booking b ON g.guestNo = b.guestNo
JOIN Hotel h ON b.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel'
  AND b.dateFrom <= CURRENT_DATE
  AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL);

-- Grosvenor Hotel 所有房间及当前入住客人
SELECT r.*, g.guestName
FROM Room r
JOIN Hotel h ON r.hotelNo = h.hotelNo
LEFT JOIN Booking b ON r.roomNo = b.roomNo
    AND r.hotelNo = b.hotelNo
    AND b.dateFrom <= CURRENT_DATE
    AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL)
LEFT JOIN Guest g ON b.guestNo = g.guestNo
WHERE h.hotelName = 'Grosvenor Hotel';

-- Grosvenor Hotel 今日收入
SELECT SUM(r.price) AS totalIncome
FROM Booking b
JOIN Room r ON b.roomNo = r.roomNo AND b.hotelNo = r.hotelNo
JOIN Hotel h ON b.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel'
  AND b.dateFrom <= CURRENT_DATE
  AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL);

-- Grosvenor Hotel 当前空房
SELECT r.*
FROM Room r
JOIN Hotel h ON r.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel'
  AND NOT EXISTS (
      SELECT 1 FROM Booking b
      WHERE b.hotelNo = r.hotelNo
        AND b.roomNo = r.roomNo
        AND b.dateFrom <= CURRENT_DATE
        AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL)
  );

-- Grosvenor Hotel 空房损失收入
SELECT SUM(r.price) AS lostIncome
FROM Room r
JOIN Hotel h ON r.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel'
  AND NOT EXISTS (
      SELECT 1 FROM Booking b
      WHERE b.hotelNo = r.hotelNo
        AND b.roomNo = r.roomNo
        AND b.dateFrom <= CURRENT_DATE
        AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL)
  );

-- 每个酒店房间数量
SELECT hotelNo, COUNT(*) AS roomCount
FROM Room
GROUP BY hotelNo;

-- 伦敦每个酒店房间数量
SELECT r.hotelNo, COUNT(*) AS roomCount
FROM Room r
JOIN Hotel h ON r.hotelNo = h.hotelNo
WHERE h.city = 'London'
GROUP BY r.hotelNo;

-- 每个酒店今日空房损失收入
SELECT h.hotelNo, COALESCE(SUM(r.price), 0) AS lostIncome
FROM Hotel h
JOIN Room r ON h.hotelNo = r.hotelNo
WHERE NOT EXISTS (
    SELECT 1 FROM Booking b
    WHERE b.hotelNo = r.hotelNo
      AND b.roomNo = r.roomNo
      AND b.dateFrom <= CURRENT_DATE
      AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL)
)
GROUP BY h.hotelNo;

-- 预订过伦敦所有酒店的客人姓名
SELECT g.guestName
FROM Guest g
WHERE NOT EXISTS (
    SELECT h.hotelNo FROM Hotel h
    WHERE h.city = 'London'
      AND NOT EXISTS (
          SELECT 1 FROM Booking b
          WHERE b.guestNo = g.guestNo AND b.hotelNo = h.hotelNo
      )
);

-- 被每个客人都预订过的酒店
SELECT h.hotelNo, h.hotelName
FROM Hotel h
WHERE NOT EXISTS (
    SELECT g.guestNo FROM Guest g
    WHERE NOT EXISTS (
        SELECT 1 FROM Booking b
        WHERE b.hotelNo = h.hotelNo AND b.guestNo = g.guestNo
    )
);

-- 被每个中国客人都预订过的酒店
SELECT h.hotelNo, h.hotelName
FROM Hotel h
WHERE NOT EXISTS (
    SELECT g.guestNo FROM Guest g
    WHERE g.guestAddress LIKE '%China%'
      AND NOT EXISTS (
          SELECT 1 FROM Booking b
          WHERE b.hotelNo = h.hotelNo AND b.guestNo = g.guestNo
      )
);
```

#### 解析

当前入住条件是 `dateFrom <= CURRENT_DATE` 且 `dateTo >= CURRENT_DATE 或 dateTo IS NULL`。查询“空房”适合用 `NOT EXISTS`，查询“所有酒店/每个客人”也用双重 `NOT EXISTS`。

### 7.4 DML、DDL 与视图

#### 题干

完成以下 DML、DDL 和视图操作：

1. 向各表插入记录。（可选）

2. 将所有房间价格上调 5%。

3. 使用 SQL 完整性增强特性创建 Hotel 表。

4. 使用 SQL 完整性增强特性创建 Room、Booking 和 Guest 表，并满足约束：type 必须是 Single、Double 或 Family；price 必须在 10 到 100 美元之间；roomNo 必须在 1 到 100 之间；dateFrom 和 dateTo 必须晚于今天。

5. 创建一个与 Booking 表结构相同的归档表，使用 INSERT 将 2003-01-01 之前的预订复制到归档表，并从 Booking 表中删除这些历史预订。

6. 创建包含酒店名称和当前入住客人姓名的视图。

7. 创建包含 Grosvenor Hotel 每位客人账单金额的视图。

#### 答案

```sql
-- 插入样例数据
INSERT INTO Hotel(hotelNo, hotelName, city) VALUES ('H001', 'Grosvenor Hotel', 'London');
INSERT INTO Room(roomNo, hotelNo, type, price) VALUES (101, 'H001', 'Double', 50.00);
INSERT INTO Guest(guestNo, guestName, guestAddress) VALUES ('G01', 'John Doe', 'Beijing, China');
INSERT INTO Booking(hotelNo, guestNo, dateFrom, dateTo, roomNo)
VALUES ('H001', 'G01', '2026-05-01', '2026-05-10', 101);

-- 房价上涨 5%
UPDATE Room
SET price = price * 1.05;

-- Hotel 表
CREATE TABLE Hotel (
    hotelNo CHAR(5) PRIMARY KEY,
    hotelName VARCHAR(50) NOT NULL,
    city VARCHAR(50) NOT NULL
);

-- Room、Guest、Booking 表
CREATE TABLE Room (
    roomNo INT CHECK (roomNo BETWEEN 1 AND 100),
    hotelNo CHAR(5),
    type VARCHAR(10) CHECK (type IN ('Single', 'Double', 'Family')),
    price DECIMAL(5,2) CHECK (price BETWEEN 10 AND 100),
    PRIMARY KEY (roomNo, hotelNo),
    FOREIGN KEY (hotelNo) REFERENCES Hotel(hotelNo)
);

CREATE TABLE Guest (
    guestNo CHAR(5) PRIMARY KEY,
    guestName VARCHAR(50) NOT NULL,
    guestAddress VARCHAR(100)
);

CREATE TABLE Booking (
    hotelNo CHAR(5),
    guestNo CHAR(5),
    dateFrom DATE CHECK (dateFrom > CURRENT_DATE),
    dateTo DATE,
    roomNo INT,
    PRIMARY KEY (hotelNo, guestNo, dateFrom),
    FOREIGN KEY (hotelNo) REFERENCES Hotel(hotelNo),
    FOREIGN KEY (guestNo) REFERENCES Guest(guestNo),
    FOREIGN KEY (roomNo, hotelNo) REFERENCES Room(roomNo, hotelNo),
    CHECK (dateTo IS NULL OR (dateTo > CURRENT_DATE AND dateTo >= dateFrom))
);

-- 归档 2003-01-01 前预订
CREATE TABLE BookingArchive AS
SELECT * FROM Booking WHERE 1 = 0;

INSERT INTO BookingArchive
SELECT * FROM Booking
WHERE dateFrom < '2003-01-01';

DELETE FROM Booking
WHERE dateFrom < '2003-01-01';

-- 酒店当前入住客人视图
CREATE VIEW HotelGuests AS
SELECT DISTINCT h.hotelName, g.guestName
FROM Hotel h
JOIN Booking b ON h.hotelNo = b.hotelNo
JOIN Guest g ON b.guestNo = g.guestNo
WHERE b.dateFrom <= CURRENT_DATE
  AND (b.dateTo >= CURRENT_DATE OR b.dateTo IS NULL);

-- Grosvenor Hotel 客人账单视图
CREATE VIEW GrosvenorAccounts AS
SELECT g.guestNo, g.guestName,
       SUM(r.price * DATEDIFF(b.dateTo, b.dateFrom)) AS totalAccount
FROM Guest g
JOIN Booking b ON g.guestNo = b.guestNo
JOIN Room r ON b.roomNo = r.roomNo AND b.hotelNo = r.hotelNo
JOIN Hotel h ON b.hotelNo = h.hotelNo
WHERE h.hotelName = 'Grosvenor Hotel' AND b.dateTo IS NOT NULL
GROUP BY g.guestNo, g.guestName;
```

#### 解析

DDL 负责建表和约束，DML 负责插入、更新和删除，视图 `CREATE VIEW` 用于保存常用查询定义。

---




