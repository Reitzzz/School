# Assignment 2-2 整理笔记

给定：

```text
student(Sno, Sname, Sdept, DOB, Address, Telephone, Gender, Degree)
lecturer(Tno, Tname, Department, Address, Telephone, Title)
course(Cno, Cname, Cpno, Credit)
contact(Tno, Cno, Hours)
enrol(Sno, Cno, Mark)
```

### 3.1 查询的关系代数

#### 答案

```text
1. πSno,Sname,Sdept(σDOB<'2000-01-01'(student))

2. πSno,Sname(σSdept='CS' AND Gender='female'(student))

3. πTno(lecturer) - πTno(contact)

4. πSno,Sname,Sdept(σCno='c02' AND Mark>85(student ⋈ enrol))

5. πCno,Cname(σTitle='Professor'(lecturer ⋈ contact ⋈ course))
```

#### 讲解过程

前两题是单表筛选；第 3 题是“不教课”的差集；第 4、5 题需要连接选课/授课表，再筛选课程或职称。

### 3.2 关系代数计算题

#### 答案

原文已给出的结果：

```text
1) πA,B(R ∩ S) = {(3,5)}
2) σR.B=S.B=5(R × S) = R ⋈R.B=S.B=5 S
3) πA,C(R) ∩ πA,C(S) = {(3,7), (4,8)}
4) πB,C(R) ∪ πB,C(S) = {(4,6), (5,7), (6,8), (5,8), (5,9)}
```

第 5 小题 `R - S` 需要原表 R、S 的完整元组，当前抽取文本中没有表格数据，无法唯一确定。

#### 讲解过程

投影会去重；交集保留两个关系共有的元组；并集要求两个关系并相容；差集 `R - S` 表示属于 R 但不属于 S 的元组。

---
