# Assignment of Module 03-1 整理笔记

### 答案

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

### 讲解过程

SQL 查询先判断是否只涉及单表；需要课程名、学生名、教师名时再连接表。统计类问题使用 `GROUP BY`，对分组结果筛选使用 `HAVING`，不是 `WHERE`。

---
