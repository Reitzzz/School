# 第 02-2 模块作业整理笔记

给定：

```text
student(Sno, Sname, Sdept, DOB, Address, Telephone, Gender, Degree)
lecturer(Tno, Tname, Department, Address, Telephone, Title)
course(Cno, Cname, Cpno, Credit)
contact(Tno, Cno, Hours)
enrol(Sno, Cno, Mark)
```

### 3.1 查询的关系代数

#### 题干

给定大学数据库模式：student(Sno, Sname, Sdept, DOB, Address, Telephone, Gender, Degree)，lecturer(Tno, Tname, Department, Address, Telephone, Title)，course(Cno, Cname, Cpno, Credit)，contact(Tno, Cno, Hours)，enrol(Sno, Cno, Mark)。用关系代数完成以下查询：

1. 列出 2000-01-01 之前出生的所有学生的学号、姓名和院系。

2. 列出 CS 系且性别为女的学生学号和姓名。

3. 列出没有讲授任何课程的教师编号。

4. 列出选修 c02 且成绩大于 85 的学生学号、姓名和院系。

5. 按课程号和课程名列出由教授讲授的课程信息。

#### 答案

```text
1. πSno,Sname,Sdept(σDOB<'2000-01-01'(student))

2. πSno,Sname(σSdept='CS' AND Gender='female'(student))

3. πTno(lecturer) - πTno(contact)

4. πSno,Sname,Sdept(σCno='c02' AND Mark>85(student ⋈ enrol))

5. πCno,Cname(σTitle='Professor'(lecturer ⋈ contact ⋈ course))
```

#### 解析

前两题是单表筛选；第 3 题是“不教课”的差集；第 4、5 题需要连接选课/授课表，再筛选课程或职称。

### 3.2 关系代数计算题

#### 题干

已知关系 R 和 S，求下列关系代数表达式的结果：

1. πA,B(R ∩ S)

2. σR.B=S.B=5(R × S)，等价于 R ⋈R.B=S.B=5 S

3. πA,C(R) ∩ πA,C(S)

4. πB,C(R) ∪ πB,C(S)

5. R - S

#### 答案

原文已给出的结果：

```text
1) πA,B(R ∩ S) = {(3,5)}
2) σR.B=S.B=5(R × S) = R ⋈R.B=S.B=5 S
3) πA,C(R) ∩ πA,C(S) = {(3,7), (4,8)}
4) πB,C(R) ∪ πB,C(S) = {(4,6), (5,7), (6,8), (5,8), (5,9)}
```

第 5 小题 `R - S` 需要原表 R、S 的完整元组，当前抽取文本中没有表格数据，无法唯一确定。

#### 解析

投影会去重；交集保留两个关系共有的元组；并集要求两个关系并相容；差集 `R - S` 表示属于 R 但不属于 S 的元组。

---


