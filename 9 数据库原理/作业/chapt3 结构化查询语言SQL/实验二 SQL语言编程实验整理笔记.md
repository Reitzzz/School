# 实验二 SQL语言编程实验整理笔记

给定关系：

英文版关系：

```text
DEPT(DNO, DNAME, CNAME)
EMP(ENO, ENAME, ESEX, ESALARY, DNO)
PROJ(PNO, PNAME, PCITY)
EP(ENO, PNO, RESPONSE)
```

中文版关系：

```text
部门(部门号, 部门名称, 部门负责人)
员工(员工号, 员工姓名, 性别, 工资, 部门号)
项目(项目号, 项目名称, 项目所在城市)
员工项目(员工号, 项目号, 职责)
```

### 8.1 建表、主键、外键和索引

#### 题干

有一个员工项目数据库，包含部门 DEPT、员工 EMP、项目 PROJ、员工参与项目情况 EP 四张表。<br>
**一个员工属于一个部门，一个部门有多个员工；一个员工可同时参加多个项目，一个项目也可由多个员工共同开发**。<br>
用 SQL 的 DDL 语言建立这四张表、表的主码、外码和索引。<br>
**其中：在部门表 DEPT 上按部门名称建立升序唯一索引；在员工表 EMP 上按部门建立索引，按职工工资降序建立索引；在员工项目表 EP 上按员工号升序、项目号降序建立唯一索引。**

#### 答案

```sql
CREATE TABLE DEPT (
    DNO CHAR(10) PRIMARY KEY,
    DNAME VARCHAR(50) NOT NULL,
    CNAME CHAR(10)
);

CREATE TABLE EMP (
    ENO CHAR(10) PRIMARY KEY,
    ENAME VARCHAR(50) NOT NULL,
    ESEX CHAR(2),
    ESALARY DECIMAL(10,2),
    DNO CHAR(10),
    FOREIGN KEY (DNO) REFERENCES DEPT(DNO)
);

CREATE TABLE PROJ (
    PNO CHAR(10) PRIMARY KEY,
    PNAME VARCHAR(50) NOT NULL,
    PCITY VARCHAR(50)
);

CREATE TABLE EP (
    ENO CHAR(10),
    PNO CHAR(10),
    RESPONSE VARCHAR(50),
    PRIMARY KEY (ENO, PNO),
    FOREIGN KEY (ENO) REFERENCES EMP(ENO),
    FOREIGN KEY (PNO) REFERENCES PROJ(PNO)
);

CREATE UNIQUE INDEX idx_dept_dname ON DEPT(DNAME ASC);
CREATE INDEX idx_emp_dno ON EMP(DNO);
CREATE INDEX idx_emp_salary_desc ON EMP(ESALARY DESC);
CREATE UNIQUE INDEX idx_ep_eno_pno ON EP(ENO ASC, PNO DESC);
```

#### 解析

部门、员工、项目分别用编号作主键；员工属于部门，所以 `EMP.DNO` 是外键；员工和项目是多对多关系，用 `EP` 作为联系表。

### 8.2 复杂查询

#### 题干

根据 DEPT、EMP、PROJ、EP 数据库，完成以下复杂查询：

1. 求**参加 J3 项目**的员工号、员工姓名以及所在部门名称。

2. 查询每个员工参加的项目数量，包括员工号、姓名和参加项目数量。

3. 找出参加项目数大于 3 的员工号。

4. 统计每个部门的最高工资、最低工资、平均工资，列出平均工资高于 1500 的部门编号和名称。

6. 查询本公司参加上海项目开发的员工姓名。

7. 查询每个部门的员工人数，列出员工人数大于 20 的部门号和部门名称。

8. 查询工资大于 1500 的员工人数。

9. 查询工程部的员工人数以及平均工资。

#### 答案

```sql
-- 1. 参加 J3 项目的员工号、姓名、部门名
SELECT e.ENO, e.ENAME, d.DNAME
FROM EMP e
JOIN EP ep ON e.ENO = ep.ENO
JOIN DEPT d ON e.DNO = d.DNO
WHERE ep.PNO = 'J3';

-- 2. 每个员工参加的项目数量
SELECT e.ENO, e.ENAME, COUNT(ep.PNO) AS project_count
FROM EMP e
LEFT JOIN EP ep ON e.ENO = ep.ENO
GROUP BY e.ENO, e.ENAME;

-- 3. 参加项目数大于 3 的员工号
SELECT ENO
FROM EP
GROUP BY ENO
HAVING COUNT(PNO) > 3;

-- 4. 平均工资高于 1500 的部门工资统计
SELECT d.DNO, d.DNAME,
       MAX(e.ESALARY) AS max_salary,
       MIN(e.ESALARY) AS min_salary,
       AVG(e.ESALARY) AS avg_salary
FROM DEPT d
JOIN EMP e ON d.DNO = e.DNO
GROUP BY d.DNO, d.DNAME
HAVING AVG(e.ESALARY) > 1500;

-- 6. 参加上海项目开发的员工姓名
SELECT DISTINCT e.ENAME
FROM EMP e
JOIN EP ep ON e.ENO = ep.ENO
JOIN PROJ p ON ep.PNO = p.PNO
WHERE p.PCITY = '上海';

-- 7. 员工人数大于 20 的部门号和部门名
SELECT d.DNO, d.DNAME
FROM DEPT d
JOIN EMP e ON d.DNO = e.DNO
GROUP BY d.DNO, d.DNAME
HAVING COUNT(e.ENO) > 20;

-- 8. 工资大于 1500 的员工人数
SELECT COUNT(*) AS emp_count
FROM EMP
WHERE ESALARY > 1500;

-- 9. 工程部员工人数和平均工资
SELECT COUNT(e.ENO) AS emp_count, AVG(e.ESALARY) AS avg_salary
FROM DEPT d
JOIN EMP e ON d.DNO = e.DNO
WHERE d.DNAME = '工程部';
```

#### 解析

员工、部门、项目之间分别通过 `DNO`、`ENO`、`PNO` 连接。统计分组时，筛选聚合结果要用 `HAVING`。

### 8.3 数据库操作

#### 题干

根据 DEPT、EMP、PROJ、EP 数据库，完成以下数据库操作：

1. 给工程部所有人员的工资上涨 5%。

2. 删除上海的所有项目以及相关数据。

3. 找出没有参加任何项目的工程部人员姓名。

4. 统计每个部门的最高工资、最低工资、平均工资，将结果存入工资统计表 salarystatic。

5. 给 DEPT 表增加部门人数属性，并将部门人数统计出来填写完整。

6. 创建一个包含员工号、员工姓名和所在部门名称的视图。

7. 创建一个包含部门编号、部门名称、部门领导姓名的视图。

8. 查询参加了所有北京项目的员工姓名。

9. 查询参加了 1002 员工参加的所有项目的员工姓名。

10. 查询所有项目主管的平均工资、最高工资和最低工资。

#### 答案

```sql
-- 1. 工程部工资上涨 5%
UPDATE EMP
SET ESALARY = ESALARY * 1.05
WHERE DNO IN (SELECT DNO FROM DEPT WHERE DNAME = '工程部');

-- 2. 删除上海项目及相关数据
DELETE FROM EP
WHERE PNO IN (SELECT PNO FROM PROJ WHERE PCITY = '上海');

DELETE FROM PROJ
WHERE PCITY = '上海';

-- 3. 没有参加任何项目的工程部人员姓名
SELECT e.ENAME
FROM EMP e
JOIN DEPT d ON e.DNO = d.DNO
WHERE d.DNAME = '工程部'
  AND NOT EXISTS (
      SELECT 1 FROM EP ep WHERE ep.ENO = e.ENO
  );

-- 4. 工资统计表
CREATE TABLE salarystatic AS
SELECT d.DNO, d.DNAME,
       MAX(e.ESALARY) AS max_salary,
       MIN(e.ESALARY) AS min_salary,
       AVG(e.ESALARY) AS avg_salary
FROM DEPT d
JOIN EMP e ON d.DNO = e.DNO
GROUP BY d.DNO, d.DNAME;

-- 5. DEPT 增加部门人数并填写
ALTER TABLE DEPT ADD emp_count INT DEFAULT 0;

UPDATE DEPT d
SET emp_count = (
    SELECT COUNT(*) FROM EMP e WHERE e.DNO = d.DNO
);

-- 6. 员工号、员工姓名、部门名称视图
CREATE VIEW v_emp_dept AS
SELECT e.ENO, e.ENAME, d.DNAME
FROM EMP e
JOIN DEPT d ON e.DNO = d.DNO;

-- 7. 部门编号、部门名称、部门领导姓名视图
CREATE VIEW v_dept_leader AS
SELECT d.DNO, d.DNAME, e.ENAME AS leader_name
FROM DEPT d
LEFT JOIN EMP e ON d.CNAME = e.ENO;

-- 8. 参加了所有北京项目的员工姓名
SELECT e.ENAME
FROM EMP e
WHERE NOT EXISTS (
    SELECT p.PNO FROM PROJ p
    WHERE p.PCITY = '北京'
      AND NOT EXISTS (
          SELECT 1 FROM EP ep
          WHERE ep.ENO = e.ENO AND ep.PNO = p.PNO
      )
);

-- 9. 参加了 1002 员工参加的所有项目的员工姓名
SELECT e.ENAME
FROM EMP e
WHERE NOT EXISTS (
    SELECT ep1.PNO FROM EP ep1
    WHERE ep1.ENO = '1002'
      AND NOT EXISTS (
          SELECT 1 FROM EP ep2
          WHERE ep2.ENO = e.ENO AND ep2.PNO = ep1.PNO
      )
);

-- 10. 所有项目主管的平均、最高、最低工资
SELECT AVG(e.ESALARY) AS avg_salary,
       MAX(e.ESALARY) AS max_salary,
       MIN(e.ESALARY) AS min_salary
FROM EMP e
JOIN EP ep ON e.ENO = ep.ENO
WHERE ep.RESPONSE = '主管';
```

#### 解析

删除有外键关联的数据时，应先删联系表 `EP`，再删主表 `PROJ`。第 8、9 题都是“参加了所有……”类型，使用双重 `NOT EXISTS`。

### 8.4 用户与权限

#### 题干

根据以下要求完成用户创建和权限设置：

1. 为数据库创建 3 个用户 user1、user2、user3，并设置登录密码。

2. 给 user1 授权：将 DEPT 表和 EMP 表的 insert、update 权限授予 user1。

3. 将 PROJ 表的所有权限授予 user2，并允许 user2 继续授予这些权限。

4. 将 EP 表的 insert 权限、response 属性的 update 权限、select 权限授予 user3。

5. 回收 user2 对 PROJ 表的 delete 权限。

#### 答案

```sql
CREATE USER 'user1'@'localhost' IDENTIFIED BY 'password1';
CREATE USER 'user2'@'localhost' IDENTIFIED BY 'password2';
CREATE USER 'user3'@'localhost' IDENTIFIED BY 'password3';

GRANT INSERT, UPDATE ON DEPT TO 'user1'@'localhost';
GRANT INSERT, UPDATE ON EMP TO 'user1'@'localhost';

GRANT ALL PRIVILEGES ON PROJ TO 'user2'@'localhost' WITH GRANT OPTION;

GRANT INSERT, SELECT ON EP TO 'user3'@'localhost';
GRANT UPDATE(RESPONSE) ON EP TO 'user3'@'localhost';

REVOKE DELETE ON PROJ FROM 'user2'@'localhost';
```

#### 解析

`GRANT` 授权，`REVOKE` 回收权限。列级授权可写成 `UPDATE(列名)`，`WITH GRANT OPTION` 表示用户可以继续把权限授予别人。

---




