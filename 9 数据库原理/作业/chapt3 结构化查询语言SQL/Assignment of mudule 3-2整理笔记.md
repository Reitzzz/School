# 第 03-2 模块作业整理笔记

### 6.1 前 4 题：RA 与 SQL

#### 题干

给定关系：student、lecturer、course、contact、enrol。分别用关系代数和 SQL 表达以下查询：

1. 查询选修 ACSC7101 课程的所有学生学号、姓名和院系。

2. 查询 CS 系且出生日期早于 1997-01-01 的学生学号和姓名。

3. 查询来自 Illinois 或 California 的学生学号、姓名和院系。

4. 查询没有讲授任何课程的教师编号。

#### 答案

```text
1. 选修 ACSC7101 的学生学号、姓名、系别
RA:  πSno,Sname,Sdept(σCno='ACSC7101'(student ⋈ enrol))
SQL:
SELECT s.Sno, s.Sname, s.Sdept
FROM student s JOIN enrol e ON s.Sno = e.Sno
WHERE e.Cno = 'ACSC7101';

2. CS 系且 1997-01-01 前出生学生学号、姓名
RA:  πSno,Sname(σSdept='CS' AND DOB<'1997-01-01'(student))
SQL:
SELECT Sno, Sname
FROM student
WHERE Sdept = 'CS' AND DOB < '1997-01-01';

3. 来自 Illinois 或 California 的学生学号、姓名、系别
RA:  πSno,Sname,Sdept(σAddress LIKE '%Illinois%' OR Address LIKE '%California%'(student))
SQL:
SELECT Sno, Sname, Sdept
FROM student
WHERE Address LIKE '%Illinois%' OR Address LIKE '%California%';

4. 没有教任何课的教师编号
RA:  πTno(lecturer) - πTno(contact)
SQL:
SELECT Tno
FROM lecturer
WHERE Tno NOT IN (SELECT Tno FROM contact);
```

#### 解析

“选修课程”要连 `student` 和 `enrol`；“没有教课”是典型的差集/反查询，可以用 `NOT IN` 或 `NOT EXISTS`。

### 6.2 统计与复杂 SQL

#### 题干

用 SQL 表达以下查询：

1. 本学期有多少位教师授课？

2. 查询课程号为 ACSC7102 的课程最高分、最低分和平均分。

3. 查询课程平均分大于 85 的学生学号。

4. 查询攻读博士学位的学生人数。

5. 查询各学位项目的学生人数。

6. 查询共有多少门课程被学生选修。

7. 查询每种职称的教师人数。

8. 查询选修人数少于 20 的课程号。

9. 查询学生人数超过 150 的系别。

10. 查询本学期每个系有多少位教师授课。

11. 查询 Database Theory 课程的最高分、最低分和平均分。

12. 查询课程平均分大于 85 的学生学号和姓名。

13. 查询选修 Data Science 且取得最高成绩的学生学号和姓名。

14. 查询尚未讲授任何课程的教师编号和姓名。

15. 查询讲授课程超过 3 门的教师编号和姓名。

16. 查询选修过 Tom 选修过的某些课程的学生姓名。

17. 查询讲授 Database Design 课程的教师姓名和职称。

22. 查询选修了大学中所有课程的学生姓名。

23. 查询选修了 Database Principles、College English 和 Advanced Mathematics 全部课程的 CS 学生姓名。

24. 查询被大学中每个学生都选修过的所有课程。

25. 查询被大学中每个 CS 学生都选修过的所有课程。

26. 查询至少选修了学生 202311101 所选全部课程的学生姓名。

#### 答案

```sql
-- 1. 本学期授课教师人数
SELECT COUNT(DISTINCT Tno) AS teacher_count
FROM contact;

-- 2. ACSC7102 课程最高、最低、平均成绩
SELECT MAX(Mark), MIN(Mark), AVG(Mark)
FROM enrol
WHERE Cno = 'ACSC7102';

-- 3. 平均成绩大于 85 的学生学号
SELECT Sno
FROM enrol
GROUP BY Sno
HAVING AVG(Mark) > 85;

-- 4. 博士生人数
SELECT COUNT(*) AS doctor_count
FROM student
WHERE Degree = 'Doctor';

-- 5. 每种学位项目的学生人数
SELECT Degree, COUNT(*) AS student_count
FROM student
GROUP BY Degree;

-- 6. 被学生选修过的课程数量
SELECT COUNT(DISTINCT Cno) AS course_count
FROM enrol;

-- 7. 每种职称的教师人数
SELECT Title, COUNT(*) AS teacher_count
FROM lecturer
GROUP BY Title;

-- 8. 选修人数少于 20 的课程号
SELECT Cno
FROM enrol
GROUP BY Cno
HAVING COUNT(*) < 20;

-- 9. 学生人数超过 150 的系别
SELECT Sdept
FROM student
GROUP BY Sdept
HAVING COUNT(*) > 150;

-- 10. 每个系本学期授课教师人数
SELECT l.Department, COUNT(DISTINCT c.Tno) AS teacher_count
FROM lecturer l
JOIN contact c ON l.Tno = c.Tno
GROUP BY l.Department;

-- 11. Database Theory 课程最高、最低、平均成绩
SELECT MAX(e.Mark), MIN(e.Mark), AVG(e.Mark)
FROM enrol e
JOIN course c ON e.Cno = c.Cno
WHERE c.Cname = 'Database Theory';

-- 12. 平均成绩大于 85 的学生学号、姓名
SELECT s.Sno, s.Sname
FROM student s
JOIN enrol e ON s.Sno = e.Sno
GROUP BY s.Sno, s.Sname
HAVING AVG(e.Mark) > 85;

-- 13. Data Science 课程最高分学生学号、姓名
SELECT s.Sno, s.Sname
FROM student s
JOIN enrol e ON s.Sno = e.Sno
JOIN course c ON e.Cno = c.Cno
WHERE c.Cname = 'Data Science'
  AND e.Mark = (
      SELECT MAX(e2.Mark)
      FROM enrol e2 JOIN course c2 ON e2.Cno = c2.Cno
      WHERE c2.Cname = 'Data Science'
  );

-- 14. 没有教课的教师编号、姓名
SELECT Tno, Tname
FROM lecturer
WHERE Tno NOT IN (SELECT Tno FROM contact);

-- 15. 教授课程超过 3 门的教师编号、姓名
SELECT l.Tno, l.Tname
FROM lecturer l
JOIN contact c ON l.Tno = c.Tno
GROUP BY l.Tno, l.Tname
HAVING COUNT(DISTINCT c.Cno) > 3;

-- 16. 选修了 Tom 选修过的某些课程的学生姓名
SELECT DISTINCT s.Sname
FROM student s
JOIN enrol e ON s.Sno = e.Sno
WHERE e.Cno IN (
    SELECT e2.Cno
    FROM student t JOIN enrol e2 ON t.Sno = e2.Sno
    WHERE t.Sname = 'Tom'
);

-- 17. 教 Database Design 的教师姓名、职称
SELECT DISTINCT l.Tname, l.Title
FROM lecturer l
JOIN contact ct ON l.Tno = ct.Tno
JOIN course c ON ct.Cno = c.Cno
WHERE c.Cname = 'Database Design';

-- 22. 选修了大学所有课程的学生姓名
SELECT s.Sname
FROM student s
WHERE NOT EXISTS (
    SELECT c.Cno FROM course c
    WHERE NOT EXISTS (
        SELECT 1 FROM enrol e
        WHERE e.Sno = s.Sno AND e.Cno = c.Cno
    )
);

-- 23. CS 学生选修了三门指定课程
SELECT s.Sname
FROM student s
WHERE s.Sdept = 'CS'
  AND NOT EXISTS (
      SELECT c.Cno FROM course c
      WHERE c.Cname IN ('Database Principles', 'College English', 'Advanced Mathematics')
        AND NOT EXISTS (
            SELECT 1 FROM enrol e
            WHERE e.Sno = s.Sno AND e.Cno = c.Cno
        )
  );

-- 24. 被全校每个学生都选修的课程
SELECT c.Cno, c.Cname
FROM course c
WHERE NOT EXISTS (
    SELECT s.Sno FROM student s
    WHERE NOT EXISTS (
        SELECT 1 FROM enrol e
        WHERE e.Sno = s.Sno AND e.Cno = c.Cno
    )
);

-- 25. 被每个 CS 学生都选修的课程
SELECT c.Cno, c.Cname
FROM course c
WHERE NOT EXISTS (
    SELECT s.Sno FROM student s
    WHERE s.Sdept = 'CS'
      AND NOT EXISTS (
          SELECT 1 FROM enrol e
          WHERE e.Sno = s.Sno AND e.Cno = c.Cno
      )
);

-- 26. 至少选修了 202311101 所有课程的学生姓名
SELECT s.Sname
FROM student s
WHERE NOT EXISTS (
    SELECT e1.Cno FROM enrol e1
    WHERE e1.Sno = '202311101'
      AND NOT EXISTS (
          SELECT 1 FROM enrol e2
          WHERE e2.Sno = s.Sno AND e2.Cno = e1.Cno
      )
);
```

#### 解析

聚合题使用 `COUNT/MAX/MIN/AVG`；“没有”使用 `NOT IN` 或 `NOT EXISTS`；“所有课程/所有学生”使用双重 `NOT EXISTS` 表达除法语义。

---


