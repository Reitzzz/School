# 第 03-1 模块作业整理笔记

> **本节作业对应 `chapt3重点.md` 的知识点说明：**
> * **全面覆盖了“3 第三阶段：核心拼装（DML - 查询）”的基础语法（3.1~3.6节）**：包括 `SELECT`、`FROM`、`WHERE` 单表条件筛选，`GROUP BY` 及聚合函数（COUNT/MAX/MIN/AVG），`HAVING` 分组过滤，以及 `ORDER BY` 排序。
> * **涉及了“4 第四阶段：高级扩展拼装”的部分内容**：主要考察了 **4.1 多表连接 (JOIN)**。
> * *(未涉及：DDL、数据更新、嵌套子查询、EXISTS 除法查询、集合运算及视图权限等进阶知识点。)*

### 5.1 SQL 查询题

#### 题干

给定关系：

英文版关系：

```text
student(Sno, Sname, Sdept, DOB, Address, Telephone, Gender, Degree)
lecturer(Tno, Tname, Department, Address, Telephone, Title)
course(Cno, Cname, Cpno, Credit)
contact(Tno, Cno, Hours)
enrol(Sno, Cno, Mark)
```

中文版关系：

```text
学生(学号, 姓名, 院系, 出生日期, 地址, 电话, 性别, 学位)
教师(教师编号, 教师姓名, 所在部门, 地址, 电话, 职称)
课程(课程号, 课程名, 先修课程号, 学分)
授课(教师编号, 课程号, 课时数)
选课(学号, 课程号, 成绩)
```

用 SQL 表达以下查询：

1. 查询选修 ACSC7101 课程的所有学生学号。

2. 查询 CS 系且出生日期早于 1997-01-01 的学生学号和姓名。

3. 查询来自 Illinois 或 California 的学生学号、姓名和院系。

4. 查询正在讲授课程的教师编号。

5. 查询选修 ACSC7101 且成绩在 80 到 90 之间的学生学号。

6. 查询课时数超过 100 的课程编号和授课教师编号。

7. 查询 20210001 学生所选但还没有考试的课程编号。

8. 查询攻读博士学位的学生学号、姓名、系别和性别，并按系别升序排列。

9. 查询年龄大于 25 岁的本科生信息。

10. 查询数据库类课程的课程号、课程名和学分。

11. 查询本学期有多少位教师授课。

12. 查询 Data Structure 课程的最高分、最低分和平均分。

13. 查询平均成绩不低于 90 且最低分不低于 85 的学生学号。

14. 查询学生人数少于 100 的系别。

15. 查询选修人数超过 150 的课程号。

16. 查询选修人数少于 20 的课程名称。

17. 查询学生人数超过 150 的系别。

#### 答案

```sql
-- 1. 选修 ACSC7101 的学生学号
SELECT Sno
FROM enrol
WHERE Cno = 'ACSC7101';

-- 2. CS 系且 1997-01-01 前出生的学生学号、姓名
SELECT Sno, Sname
FROM student
WHERE Sdept = 'CS' AND DOB < '1997-01-01';

-- 3. 来自 Illinois 或 California 的学生学号、姓名、系别
SELECT Sno, Sname, Sdept
FROM student
WHERE Address LIKE '%Illinois%' OR Address LIKE '%California%';

-- 4. 正在讲授课程的教师编号
SELECT DISTINCT Tno
FROM contact;

-- 5. 选修 ACSC7101 且成绩 80 到 90 的学生学号
SELECT Sno
FROM enrol
WHERE Cno = 'ACSC7101' AND Mark BETWEEN 80 AND 90;

-- 6. 课时数超过 100 的课程编号和教师编号
SELECT Cno, Tno
FROM contact
WHERE Hours > 100;

-- 7. 20210001 学生未考试课程编号
SELECT Cno
FROM enrol
WHERE Sno = '20210001' AND Mark IS NULL;

-- 8. 博士生学号、姓名、系别、性别，按系别升序
SELECT Sno, Sname, Sdept, Gender
FROM student
WHERE Degree = 'Doctor'
ORDER BY Sdept ASC;

-- 9. 年龄大于 25 的本科生
SELECT *
FROM student
WHERE TIMESTAMPDIFF(YEAR, DOB, CURRENT_DATE) > 25
  AND Degree = 'Bachelor';

-- 10. 数据库类课程课程号、课程名、学分
SELECT Cno, Cname, Credit
FROM course
WHERE Cname LIKE '%Database%' OR Cname LIKE '%数据库%';

-- 11. 本学期授课教师人数
SELECT COUNT(DISTINCT Tno) AS TeacherCount
FROM contact;

-- 12. Data Structure 课程最高、最低、平均成绩
SELECT MAX(e.Mark) AS MaxMark, MIN(e.Mark) AS MinMark, AVG(e.Mark) AS AvgMark
FROM enrol e
JOIN course c ON e.Cno = c.Cno
WHERE c.Cname = 'Data Structure' OR c.Cname = 'data structure';

-- 13. 平均成绩 >= 90 且最低分 >= 85 的学生学号
SELECT Sno
FROM enrol
GROUP BY Sno
HAVING AVG(Mark) >= 90 AND MIN(Mark) >= 85;

-- 14. 学生人数少于 100 的系别
SELECT Sdept
FROM student
GROUP BY Sdept
HAVING COUNT(*) < 100;

-- 15. 选修人数超过 150 的课程号
SELECT Cno
FROM enrol
GROUP BY Cno
HAVING COUNT(*) > 150;

-- 16. 选修人数少于 20 的课程名
SELECT c.Cname
FROM course c
JOIN enrol e ON c.Cno = e.Cno
GROUP BY c.Cno, c.Cname
HAVING COUNT(e.Sno) < 20;

-- 17. 学生人数超过 150 的系别
SELECT Sdept
FROM student
GROUP BY Sdept
HAVING COUNT(*) > 150;
```

#### 解析

SQL 查询先判断是否只涉及单表；需要课程名、学生名、教师名时再连接表。统计类问题使用 `GROUP BY`，对分组结果筛选使用 `HAVING`，不是 `WHERE`。

---


