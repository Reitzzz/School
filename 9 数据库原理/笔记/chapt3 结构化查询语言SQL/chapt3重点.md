# 结构化查询语言(SQL)全流程拼装演示

## 目录

- [结构化查询语言(SQL)全流程拼装演示](#结构化查询语言sql全流程拼装演示)
  - [目录](#目录)
  - [1 第一阶段：准备游乐场与底座（数据定义语言 DDL）](#1-第一阶段准备游乐场与底座数据定义语言-ddl)
    - [1.1 积木1：创建表(CREATE TABLE)](#11-积木1创建表create-table)
    - [1.2 积木2：删除表(DROP TABLE)](#12-积木2删除表drop-table)
    - [1.3 积木3：创建索引(CREATE INDEX)](#13-积木3创建索引create-index)
    - [1.4 积木4：删除索引(DROP INDEX)](#14-积木4删除索引drop-index)
  - [2 第二阶段：往底板上放积木（数据操纵语言 DML - 更新）](#2-第二阶段往底板上放积木数据操纵语言-dml---更新)
    - [2.1 积木1：插入数据(INSERT)](#21-积木1插入数据insert)
    - [2.2 积木2：更新数据(UPDATE)](#22-积木2更新数据update)
    - [2.3 积木3：删除数据(DELETE)](#23-积木3删除数据delete)
  - [3 第三阶段：核心拼装，一层层找积木（数据操纵语言 DML - 查询）](#3-第三阶段核心拼装一层层找积木数据操纵语言-dml---查询)
    - [3.1 基础底板：去哪个底板找？(FROM)](#31-基础底板去哪个底板找from)
    - [3.2 第一层拼装：按什么条件挑单块积木？(WHERE)](#32-第一层拼装按什么条件挑单块积木where)
      - [3.2.1 常用查询积木](#321-常用查询积木)
    - [3.3 第二层拼装：怎么分堆？(GROUP BY)](#33-第二层拼装怎么分堆group-by)
      - [3.3.1 COUNT 查询积木](#331-count-查询积木)
      - [3.3.2 GROUP BY 防坑积木](#332-group-by-防坑积木)
    - [3.4 第三层拼装：筛选哪些堆？(HAVING)](#34-第三层拼装筛选哪些堆having)
      - [3.4.1 HAVING COUNT(*) 题型积木](#341-having-count-题型积木)
    - [3.5 第四层拼装：挑出哪些部分展示？(SELECT)](#35-第四层拼装挑出哪些部分展示select)
      - [3.5.1 DISTINCT 去重查询积木](#351-distinct-去重查询积木)
    - [3.6 第五层拼装：按什么顺序排好交差？(ORDER BY)](#36-第五层拼装按什么顺序排好交差order-by)
  - [4 第四阶段：高级扩展拼装](#4-第四阶段高级扩展拼装)
    - [4.1 积木1：多表连接(JOIN)](#41-积木1多表连接join)
    - [4.2 积木2：子查询(Subquery)](#42-积木2子查询subquery)
    - [4.3 积木3：EXISTS 与除法查询](#43-积木3exists-与除法查询)
      - [4.3.1 Level 1：基础“没有记录”](#431-level-1基础没有记录)
      - [4.3.2 Level 2：带条件的“没有记录”](#432-level-2带条件的没有记录)
      - [4.3.3 Level 3：双层 NOT EXISTS 表达“所有”](#433-level-3双层-not-exists-表达所有)
      - [4.3.4 Level 4：限定集合里的“所有”](#434-level-4限定集合里的所有)
    - [4.4 积木4：集合运算(UNION/EXCEPT/INTERSECT)](#44-积木4集合运算unionexceptintersect)
  - [5 第五阶段：保存图纸与控制游乐场](#5-第五阶段保存图纸与控制游乐场)
    - [5.1 积木1：创建视图(CREATE VIEW)](#51-积木1创建视图create-view)
    - [5.2 积木2：查询、更新、删除视图](#52-积木2查询更新删除视图)
    - [5.3 积木3：授予权限(GRANT)](#53-积木3授予权限grant)
    - [5.4 积木4：收回权限(REVOKE)](#54-积木4收回权限revoke)
  - [本章速记](#本章速记)

---

## 1 第一阶段：准备游乐场与底座（数据定义语言 DDL）

DDL 就是在数据库里搭底板：建表、拆表、贴索引标签、撕掉索引标签。

### 1.1 积木1：创建表(CREATE TABLE)

先搭一块“学生成绩表”底板，规定每一格能放什么数据，并把 `学号` 设成主键。

```sql
CREATE TABLE 学生成绩表 (
    学号 CHAR(10) NOT NULL,
    姓名 CHAR(20),
    性别 CHAR(2),
    所在系 CHAR(30),
    成绩 INT,
    PRIMARY KEY (学号)
);
```

建表时常见约束：

- `NOT NULL`：这一格不能为空。
- `UNIQUE`：这一列不能重复。
- `DEFAULT`：没有填写时使用默认值。
- `CHECK`：限制取值范围。
- `PRIMARY KEY`：主键，唯一标识一行。
- `FOREIGN KEY ... REFERENCES ...`：外键，连接另一张表。

怎么选外键：

```text
1. 先找“被引用”的主表：例如 学生表、课程表、部门表。
2. 再找当前表里“指向主表”的列：例如 选课表.学号 指向 学生表.学号。
3. 外键列的含义和数据类型要与主表主键一致。
4. 一对多关系里，外键通常放在“多”的一方。
5. 多对多关系里，通常单独建一张联系表，把两边主表的主键都放进来作为外键。
```

如果要建“选课表”，它通常用联合主键：

```sql
CREATE TABLE 选课表 (
    学号 CHAR(10) NOT NULL,
    课程编号 CHAR(8) NOT NULL,
    成绩 INT,
    PRIMARY KEY (学号, 课程编号),
    FOREIGN KEY (学号) REFERENCES 学生表(学号),
    FOREIGN KEY (课程编号) REFERENCES 课程表(课程编号)
);
```

这里 `选课表.学号` 是外键，因为它引用 `学生表.学号`；`选课表.课程编号` 也是外键，因为它引用 `课程表.课程编号`。选课表同时连接“学生”和“课程”，所以它适合放这两个外键。

### 1.2 积木2：删除表(DROP TABLE)

如果整块底板不要了，用 `DROP TABLE` 拆掉表结构和表中数据。

```sql
DROP TABLE 学生成绩表;
```

注意：`DELETE FROM 表名` 是清空或删除部分数据，表还在；<br>
`DROP TABLE 表名` 是连表本身一起删除。

### 1.3 积木3：创建索引(CREATE INDEX)

索引像给底板贴标签，让数据库更快找到某些积木。

```sql
CREATE INDEX 院系索引
ON 学生成绩表(所在系);
```

通用格式：

```sql
CREATE [UNIQUE] INDEX 索引名
ON 表名(列名 [ASC|DESC], ...);
```

常见写法：

```sql
-- 唯一索引：要求被索引列的值不能重复
CREATE UNIQUE INDEX idx_dept_dname ON DEPT(DNAME ASC);

-- 普通索引：加快按某一列查询的速度
CREATE INDEX idx_emp_dno ON EMP(DNO);

-- 降序索引：适合经常按工资从高到低排序的查询
CREATE INDEX idx_emp_salary_desc ON EMP(ESALARY DESC);

-- 复合唯一索引：多列组合起来不能重复，也可以分别指定排序方向
CREATE UNIQUE INDEX idx_ep_eno_pno ON EP(ENO ASC, PNO DESC);
```

索引的作用：提高查询速度，但会占空间，也会拖慢插入、删除、更新，因为数据变了索引也要维护。一般不要给一张表乱建太多索引。

### 1.4 积木4：删除索引(DROP INDEX)

不用某个索引时，把标签撕掉。

```sql
DROP INDEX 院系索引;
```

---

## 2 第二阶段：往底板上放积木（数据操纵语言 DML - 更新）

DML 更新就是操作表里的具体积木：放进去、改掉、拿走。

### 2.1 积木1：插入数据(INSERT)

放上一块张三的积木。

```sql
INSERT INTO 学生成绩表 (学号, 姓名, 性别, 所在系, 成绩)
VALUES ('001', '张三', '男', '计算机科学(CS)', 85);
```

也可以用查询结果批量插入：

```sql
INSERT INTO 优秀学生表 (学号, 姓名, 成绩)
SELECT 学号, 姓名, 成绩
FROM 学生成绩表
WHERE 成绩 >= 90;
```

### 2.2 积木2：更新数据(UPDATE)

发现张三的成绩登记错了，把他的成绩换成 90 分。

```sql
UPDATE 学生成绩表
SET 成绩 = 90
WHERE 学号 = '001';
```

更新可以带子查询。例如把选修“数据库系统导论”的成绩提高 5%：

```sql
UPDATE 选课表
SET 成绩 = 成绩 * 1.05
WHERE 课程编号 = (
    SELECT 课程编号
    FROM 课程表
    WHERE 课程名称 = '数据库系统导论'
);
```

### 2.3 积木3：删除数据(DELETE)

如果张三退学，把他的积木拿走。

```sql
DELETE FROM 学生成绩表
WHERE 学号 = '001';
```

危险点：不写 `WHERE` 就会拿走整张表里的所有积木。

```sql
DELETE FROM 学生成绩表;
```

---

## 3 第三阶段：核心拼装，一层层找积木（数据操纵语言 DML - 查询）

查询的执行逻辑可以记成一条流水线：

```text
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```

我们的目标是：找出所有男生平均成绩大于 80 分的院系，并按平均成绩从高到低排列。

### 3.1 基础底板：去哪个底板找？(FROM)

先决定翻哪块底板。

```sql
SELECT *
FROM 学生成绩表;
```

`SELECT *` 表示把所有列都展示出来。真实做题时，最好写清楚列名，避免拿出不需要的列。

```sql
SELECT 学号, 姓名
FROM 学生成绩表;
```

### 3.2 第一层拼装：按什么条件挑单块积木？(WHERE)

`where`是把**不达标的个人先踢出去**，再进入后续分组和展示。

```sql
SELECT *
FROM 学生成绩表
WHERE 性别 = '男';
```

#### 3.2.1 常用查询积木

```sql
-- 比较运算
WHERE 成绩 >= 80

-- 多条件
WHERE 所在系 = '信息系统(IS)' AND 年龄 < 20

-- 范围
WHERE 成绩 BETWEEN 80 AND 90

-- 集合成员
WHERE 所在系 IN ('信息系统(IS)', '计算机科学(CS)')

-- 模糊匹配：% 任意长度，_ 单个字符
WHERE 姓名 LIKE '刘%'

-- 空值判断，不能写 = NULL
WHERE 成绩 IS NULL
WHERE 成绩 IS NOT NULL
```


### 3.3 第二层拼装：怎么分堆？(GROUP BY)

`group by`是把挑出来的男生**按照院系**分成几堆。

```sql
SELECT 所在系
FROM 学生成绩表
WHERE 性别 = '男'
GROUP BY 所在系;
```

分组常常配合聚合函数：

| 函数 | 作用 |
|------|------|
| `COUNT(*)` | 数行数 |
| `COUNT(DISTINCT 列名)` | 数不同值个数 |
| `SUM(列名)` | 求和 |
| `AVG(列名)` | 平均值 |
| `MAX(列名)` | 最大值 |
| `MIN(列名)` | 最小值 |

例子：统计每个系有多少学生。

```sql
SELECT 所在系, COUNT(*) AS 人数
FROM 学生成绩表
GROUP BY 所在系;
```

#### 3.3.1 COUNT 查询积木

```sql
-- 统计整张表有多少行
SELECT COUNT(*) AS 总人数
FROM 学生成绩表;

-- 统计成绩不为空的记录数
SELECT COUNT(成绩) AS 有成绩人数
FROM 学生成绩表;

-- 统计有多少个不同院系
SELECT COUNT(DISTINCT 所在系) AS 院系数
FROM 学生成绩表;

-- 统计每个院系有多少人
SELECT 所在系, COUNT(*) AS 人数
FROM 学生成绩表
GROUP BY 所在系;
```

注意：`COUNT(*)` 数的是行数，**包括某些列为空的行**；`COUNT(列名)` **只数这一列不为空的行**；`COUNT(DISTINCT 列名)` **会先去重再计数**。

#### 3.3.2 GROUP BY 防坑积木

看到题目要求“每个……的平均值 / 总数 / 最大值 / 最小值”，通常就要想到 `GROUP BY`。

硬规则：`SELECT` 里如果同时出现**普通列**和**聚合函数**，普通列必须写进 `GROUP BY`。

```sql
-- 错：cno 是普通列，AVG(mark) 是聚合函数，但没有 GROUP BY
SELECT cno, AVG(mark)
FROM enrol
HAVING AVG(mark) > 80;

-- 对：按课程号分组，再算每门课的平均分
SELECT cno, AVG(mark)
FROM enrol
GROUP BY cno
HAVING AVG(mark) > 80;
```

做题时先问自己：平均分、人数、最高分是“整张表一个结果”，还是“每个课程 / 每个院系 / 每个老师一个结果”？如果是“每个……”，就把这个“每个”的对象放进 `GROUP BY`。

### 3.4 第三层拼装：筛选哪些堆？(HAVING)

`HAVING` 用来挑分组后的**整堆积木哪些符合要求**。

```sql
SELECT 所在系
FROM 学生成绩表
WHERE 性别 = '男'
GROUP BY 所在系
HAVING AVG(成绩) > 80;
```

`WHERE` 和 `HAVING` 的区别：

```text
WHERE：分组前筛选单行，例如 性别 = '男'
HAVING：分组后筛选一组，例如 AVG(成绩) > 80
```

#### 3.4.1 HAVING COUNT(*) 题型积木

看到“至少 N 个 / 多于 N 个 / 少于 N 个 / 选修人数达到 N 人”这类题，通常是：

```sql
SELECT 分组列
FROM 表名
GROUP BY 分组列
HAVING COUNT(*) 比较符 N;
```

例子：查询至少有 5 名学生选修的课程号。

```sql
SELECT cno
FROM enrol
GROUP BY cno
HAVING COUNT(*) >= 5;
```

拆开理解：

```text
GROUP BY cno：先按课程号分堆
COUNT(*)：数每一堆里有几条选课记录
HAVING COUNT(*) >= 5：只留下人数至少 5 的课程堆
```

注意：`WHERE` 不能直接写 `WHERE COUNT(*) >= 5`，因为 `WHERE` 发生在分组前，那时还没有“每一堆的人数”。

### 3.5 第四层拼装：挑出哪些部分展示？(SELECT)

`SELECT` 决定最后展示哪些列、表达式、聚合结果。

```sql
SELECT 所在系, AVG(成绩) AS 平均分
FROM 学生成绩表
WHERE 性别 = '男'
GROUP BY 所在系
HAVING AVG(成绩) > 80;
```

常用展示积木：

```sql
-- 计算字段
SELECT 姓名, 2026 - 年龄 AS 出生年份
FROM 学生表;

-- 别名
SELECT AVG(成绩) AS 平均分
FROM 学生成绩表;
```

#### 3.5.1 DISTINCT 去重查询积木

`DISTINCT` 是把最后展示出来的结果里**重复的行合并成一行**。它放在 `SELECT` 后面、列名或表达式前面。

```sql
SELECT DISTINCT 列名
FROM 表名;
```

常用写法：

```sql
-- 1. 单列去重：有哪些院系
SELECT DISTINCT 所在系
FROM 学生表;

-- 2. 多列组合去重：有哪些“院系 + 性别”的组合
SELECT DISTINCT 所在系, 性别
FROM 学生表;

-- 3. 先 WHERE 筛选，再 DISTINCT 去重：男生来自哪些院系
SELECT DISTINCT 所在系
FROM 学生表
WHERE 性别 = '男';

-- 4. 去重后排序：按院系名展示
SELECT DISTINCT 所在系
FROM 学生表
ORDER BY 所在系 ASC;

-- 5. 去重计数：有多少个不同院系
SELECT COUNT(DISTINCT 所在系) AS 院系数
FROM 学生表;
```

容易错的点：

```text
SELECT DISTINCT 所在系, 性别
不是只对 所在系 去重，而是对“所在系 + 性别”这一整行组合去重。
```

如果只想看“不重复的院系”，就只写 `SELECT DISTINCT 所在系`；如果把 `姓名` 也放进去，因为每个学生姓名不同，结果可能看起来就“不怎么去重”了。

### 3.6 第五层拼装：按什么顺序排好交差？(ORDER BY)

最后排序。默认升序 `ASC`，降序写 `DESC`。

```sql
SELECT 所在系, AVG(成绩) AS 平均分
FROM 学生成绩表
WHERE 性别 = '男'
GROUP BY 所在系
HAVING AVG(成绩) > 80
ORDER BY 平均分 DESC;
```

可以按多列排序：

```sql
ORDER BY 所在系 ASC, 成绩 DESC;
```

---

## 4 第四阶段：高级扩展拼装

高级查询就是把多块底板、内部积木和集合拼到一起。

### 4.1 积木1：多表连接(JOIN)

如果成绩和奖学金信息不在一张表，就用连接把两块底板拼起来。

```sql
SELECT s.姓名, s.成绩, a.奖金金额
FROM 学生成绩表 AS s
JOIN 奖学金表 AS a ON s.学号 = a.学号;
```

常见连接写法：

```sql
-- 内连接：只保留能匹配上的行
FROM 学生表 s
JOIN 选课表 e ON s.学号 = e.学号

-- 左连接：左边学生都保留，没选课的右边为空
FROM 学生表 s
LEFT JOIN 选课表 e ON s.学号 = e.学号
```

多表查询时，同名列要加表名或别名限定，例如 `s.学号`、`e.学号`。

### 4.2 积木2：子查询(Subquery)

子查询就是先在内部拼一块小积木，再交给外层使用。

不相关子查询：内部查询可以独立运行。

```sql
SELECT 姓名, 成绩
FROM 学生成绩表
WHERE 成绩 > (
    SELECT AVG(成绩)
    FROM 学生成绩表
);
```

带 `IN` 的子查询：

```sql
SELECT 姓名
FROM 学生表
WHERE 学号 IN (
    SELECT 学号
    FROM 选课表
    WHERE 课程编号 = 'C001'
);
```

相关子查询：内部查询要引用外层当前行。

```sql
SELECT 姓名
FROM 学生表 s
WHERE EXISTS (
    SELECT *
    FROM 选课表 e
    WHERE e.学号 = s.学号
);
```

子查询规则：

- 普通比较子查询通常只能返回一列。
- `EXISTS` 子查询关注“有没有行”，不关心列内容。
- `ORDER BY` 通常只放在最外层查询中。

### 4.3 积木3：EXISTS 与除法查询

`EXISTS` 像问一句：内部这堆积木是否存在？

```sql
SELECT 姓名
FROM 学生表 s
WHERE EXISTS (
    SELECT *
    FROM 选课表 e
    WHERE e.学号 = s.学号
);
```

除法查询常见问法：“找出选修了所有课程的学生”。SQL 没有直接的“所有”，用两层 `NOT EXISTS` 表达：

```sql
SELECT 姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 课程表 c
    WHERE NOT EXISTS (
        SELECT *
        FROM 选课表 e
        WHERE e.学号 = s.学号
          AND e.课程编号 = c.课程编号
    )
);
```

记法：

```text
选了所有课
= 不存在一门课：这个学生没有选
= NOT EXISTS (课程 WHERE NOT EXISTS (选课记录))
```

#### 4.3.1 Level 1：基础“没有记录”

适合题型：没有选课的学生、没有授课的教师、没有被选的课程。

```sql
SELECT 外层表.要显示的列
FROM 外层表 x
WHERE NOT EXISTS (
    SELECT *
    FROM 关联表 y
    WHERE y.外键 = x.主键
);
```

例子：查询没有选任何课程的学生。

```sql
SELECT s.学号, s.姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 选课表 e
    WHERE e.学号 = s.学号
);
```

关键：`e.学号 = s.学号` 把内层选课记录限定为“当前这个学生”的记录。

#### 4.3.2 Level 2：带条件的“没有记录”

适合题型：没有选某门课、没有不及格记录、没有讲授某类课程。

```sql
SELECT 外层表.要显示的列
FROM 外层表 x
WHERE NOT EXISTS (
    SELECT *
    FROM 关联表 y
    WHERE y.外键 = x.主键
      AND 要排除的条件
);
```

例子：查询没有选修 `c02` 的学生。

```sql
SELECT s.学号, s.姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 选课表 e
    WHERE e.学号 = s.学号
      AND e.课程编号 = 'c02'
);
```

记法：不存在一条“属于当前对象，并且满足指定条件”的记录。

#### 4.3.3 Level 3：双层 NOT EXISTS 表达“所有”

适合题型：选修了所有课程、讲授了所有课程、借过所有图书。

```sql
SELECT 外层对象
FROM 外层表 x
WHERE NOT EXISTS (
    SELECT *
    FROM 必须全部覆盖的集合 y
    WHERE NOT EXISTS (
        SELECT *
        FROM 证明关系表 r
        WHERE r.外层对象编号 = x.编号
          AND r.集合对象编号 = y.编号
    )
);
```

例子：查询选修了所有课程的学生。

```sql
SELECT s.学号, s.姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 课程表 c
    WHERE NOT EXISTS (
        SELECT *
        FROM 选课表 e
        WHERE e.学号 = s.学号
          AND e.课程编号 = c.课程编号
    )
);
```

记法：选了所有课 = 不存在一门课，这个学生没选。

#### 4.3.4 Level 4：限定集合里的“所有”

适合题型：选修了 CS 系所有课程、选修了张三选过的所有课程、讲授了李老师讲过的所有课程。

```sql
SELECT 外层对象
FROM 外层表 x
WHERE NOT EXISTS (
    SELECT *
    FROM 限定后的目标集合 y
    WHERE 限定条件
      AND NOT EXISTS (
          SELECT *
          FROM 证明关系表 r
          WHERE r.外层对象编号 = x.编号
            AND r.目标对象编号 = y.编号
      )
);
```

例子：查询选修了 CS 系所有课程的学生。

```sql
SELECT s.学号, s.姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 课程表 c
    WHERE c.开课院系 = 'CS'
      AND NOT EXISTS (
          SELECT *
          FROM 选课表 e
          WHERE e.学号 = s.学号
            AND e.课程编号 = c.课程编号
      )
);
```

例子：查询选修了张三选过的所有课程的学生。

```sql
SELECT s.学号, s.姓名
FROM 学生表 s
WHERE NOT EXISTS (
    SELECT *
    FROM 选课表 ez
    JOIN 学生表 z ON ez.学号 = z.学号
    WHERE z.姓名 = '张三'
      AND NOT EXISTS (
          SELECT *
          FROM 选课表 e
          WHERE e.学号 = s.学号
            AND e.课程编号 = ez.课程编号
      )
);
```

关键：第一层子查询先圈定“所有”的范围，第二层子查询检查外层对象有没有漏掉其中任何一个。

### 4.4 积木4：集合运算(UNION/EXCEPT/INTERSECT)

集合运算是把两个查询结果当成两堆形状一样的积木来合并、相减或取交集。

要求：两边 `SELECT` 的列数和对应列类型要能匹配。

```sql
-- 并集：两堆合在一起
SELECT 学号, 姓名
FROM 学生表
WHERE 所在系 = '数学系(MA)'
UNION
SELECT 学号, 姓名
FROM 学生表
WHERE 年龄 > 18;
```

```sql
-- 差集：所有教师 - 已授课教师
SELECT 教师编号
FROM 教师表
EXCEPT
SELECT DISTINCT 教师编号
FROM 授课表;
```

```sql
-- 交集：既是教师又是学生的人
SELECT 姓名
FROM 教师表
INTERSECT
SELECT 姓名
FROM 学生表;
```

---

## 5 第五阶段：保存图纸与控制游乐场

视图是把常用拼法保存成图纸；权限控制决定谁能看、谁能改。

### 5.1 积木1：创建视图(CREATE VIEW)

把复杂查询保存成“优秀男系别视图”。

```sql
CREATE VIEW 优秀男系别视图
AS SELECT 所在系, AVG(成绩) AS 平均分
   FROM 学生成绩表
   WHERE 性别 = '男'
   GROUP BY 所在系
   HAVING AVG(成绩) > 80;
```

如果视图里有聚合函数、表达式或多表同名列，通常要显式写视图列名。

```sql
CREATE VIEW 系平均分视图(所在系, 平均分)
AS SELECT 所在系, AVG(成绩)
   FROM 学生成绩表
   GROUP BY 所在系;
```

`WITH CHECK OPTION` 可以限制通过视图插入或修改的数据仍然满足视图条件。

```sql
CREATE VIEW 信息系统学生视图
AS SELECT 学号, 姓名, 年龄, 所在系
   FROM 学生表
   WHERE 所在系 = '信息系统(IS)'
   WITH CHECK OPTION;
```

视图作用：

- 简化复杂查询。
- 让不同用户从不同角度看同一批数据。
- 隐藏敏感列，保护数据。
- 提高逻辑独立性，底层表变化时可以尽量不影响用户查询。

### 5.2 积木2：查询、更新、删除视图

查询视图像查询普通表一样。

```sql
SELECT *
FROM 优秀男系别视图;
```

更新视图有限制：一般只有从单个基本表导出、保留主键、没有分组和聚合的行列子集视图比较适合更新。

```sql
UPDATE 信息系统学生视图
SET 姓名 = '刘辰'
WHERE 学号 = '200215122';
```

删除视图只是删除保存的图纸，不会删除基本表。

```sql
DROP VIEW 优秀男系别视图;
```

### 5.3 积木3：授予权限(GRANT)

管理员把某张表或视图的操作权限交给用户。

```sql
GRANT SELECT
ON 优秀男系别视图
TO 教务处老师;
```

常见权限：

- `SELECT`：查询。
- `INSERT`：插入。
- `UPDATE`：更新。
- `DELETE`：删除。
- `REFERENCES`：允许被外键引用。

可以授权给所有用户：

```sql
GRANT SELECT
ON 部门表
TO PUBLIC;
```

可以允许对方继续转授权：

```sql
GRANT ALL PRIVILEGES
ON 员工表
TO 经理
WITH GRANT OPTION;
```

### 5.4 积木4：收回权限(REVOKE)

权限给错了，就把钥匙收回来。

```sql
REVOKE SELECT
ON 部门表
FROM PUBLIC;
```

收回全部权限：

```sql
REVOKE ALL PRIVILEGES
ON 员工表
FROM 总监;
```

只收回“继续转授权”的能力：

```sql
REVOKE GRANT OPTION FOR SELECT
ON 员工表
FROM 经理;
```

---

## 本章速记

```text
DDL：建/删底板和标签
CREATE TABLE, DROP TABLE, CREATE INDEX, DROP INDEX

DML 更新：放、改、拿走数据
INSERT, UPDATE, DELETE

DML 查询：按流水线拼查询
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY

查询细节：
DISTINCT 去重
BETWEEN 范围
IN 集合
LIKE 模糊
IS NULL 空值
COUNT/SUM/AVG/MAX/MIN 聚合

高级查询：
JOIN 拼多表
Subquery 套内部查询
EXISTS 判断存在
NOT EXISTS + NOT EXISTS 表达“所有”
UNION/EXCEPT/INTERSECT 做集合运算

视图与权限：
CREATE VIEW 保存查询
DROP VIEW 删除视图
GRANT 授权
REVOKE 收权
```
