# 第 02-3 模块作业整理笔记

### 4.1 电影、艺术家和角色关系模式

#### 题干

给定关系模式：

英文版关系：

```text
Films(FilmID, Title, Director, Year, ProductionCost)
Artists(ArtistID, Surname, FirstName, Sex, BirthDate, Nationality)
Roles(FilmID, ActorID, Character)
```

中文版关系：

```text
电影(电影编号, 标题, 导演编号, 年份, 制作成本)
艺术家(艺术家编号, 姓氏, 名字, 性别, 出生日期, 国籍)
角色(电影编号, 演员编号, 角色名)
```

Films 中的 Director 存放导演对应的 ArtistID。用关系代数表达以下查询：

(a) 列出 Henry Fonda 参演过的所有电影标题。

(b) 列出导演同时也是演员的所有电影标题。

(c) 列出演员性别全部相同的电影标题。

(d) 列出没有任何演员参演的电影标题。

(e) 列出演过 2021 年所有电影的演员。

(f) 列出电影 Transformer 的演员姓名。

#### 答案

```text
a. Henry Fonda 参演电影标题
πTitle(σFirstName='Henry' AND Surname='Fonda' AND ArtistID=ActorID AND Roles.FilmID=Films.FilmID
       (Artists ⋈ Roles ⋈ Films))

b. 导演也是演员的电影标题
πTitle(σFilms.FilmID=Roles.FilmID AND Director=ActorID(Films ⋈ Roles))

c. 演员性别全相同的电影标题
πTitle(Films) -
πTitle(σR1.ActorID=A1.ArtistID AND R2.ActorID=A2.ArtistID AND R1.FilmID=R2.FilmID
       AND A1.Sex<>A2.Sex AND R1.FilmID=Films.FilmID
       (ρR1(Roles) ⋈ ρA1(Artists) ⋈ ρR2(Roles) ⋈ ρA2(Artists) ⋈ Films))

d. 没有任何演员参演的电影标题
πTitle(Films) - πTitle(σFilms.FilmID=Roles.FilmID(Films ⋈ Roles))

e. 参演了所有 2021 年电影的演员
πActorID,FirstName,Surname(σActorID=ArtistID((πActorID,FilmID(Roles) ÷ πFilmID(σYear=2021(Films))) ⋈ Artists))

f. 电影 Transformer 的演员姓名
πFirstName,Surname(σTitle='Transformer' AND Films.FilmID=Roles.FilmID AND ActorID=ArtistID
                  (Films ⋈ Roles ⋈ Artists))
```

#### 解析

“所有”类查询用除法；“没有”类查询用全集减去已有集合；“导演也是演员”比较 `Films.Director` 与 `Roles.ActorID`。

### 4.2 图书馆关系模式

#### 题干

给定关系模式：

英文版关系：

```text
Book(BookID, Title, PubID)
Author(BookID, AuthorName, order)
Publisher(PubID, PubName, Address, Phone)
BookCopies(BookID, BranchID, NCopies)
BookLoans(BookID, BranchID, CardNo, DateOut, DueDate, DateReturn)
Branch(BranchID, BranchName, Address)
Borrower(CardNo, Name, Address, Phone)
```

中文版关系：

```text
图书(图书编号, 书名, 出版社编号)
作者(图书编号, 作者姓名, 作者顺序)
出版社(出版社编号, 出版社名称, 地址, 电话)
馆藏(图书编号, 分馆编号, 册数)
借阅(图书编号, 分馆编号, 借书证号, 借出日期, 应还日期, 归还日期)
分馆(分馆编号, 分馆名称, 地址)
读者(借书证号, 姓名, 地址, 电话)
```

用关系代数表达以下查询：

(a) Sharpstown 分馆拥有多少本题为 The Lost Tribe 的书？

(b) 查询所有当前没有借书的读者姓名。

(c) 对于从 Sharpstown 分馆借出且今天到期的每一本书，查询书名、读者姓名和读者地址。

(d) 对于 Stephen King 创作或合著的每本书，查询 Central 分馆拥有的书名和馆藏册数。

(e) 查询借过 The Lost Tribe 这本书的读者姓名和地址。

(f) 查询借过 Tom 所借全部图书的读者姓名。

#### 答案

```text
a. Sharpstown 分馆有多少本 The Lost Tribe
πNCopies(σTitle='The Lost Tribe' AND BranchName='Sharpstown' AND Book.BookID=BookCopies.BookID AND BookCopies.BranchID=Branch.BranchID
         (Book ⋈ BookCopies ⋈ Branch))

b. 没有借书的读者姓名
πName(Borrower) - πName(σBorrower.CardNo=BookLoans.CardNo(Borrower ⋈ BookLoans))

c. Sharpstown 分馆今天到期的外借图书标题、读者姓名和地址
πTitle,Name,Address(σBranchName='Sharpstown' AND DueDate=CURRENT_DATE AND Book.BookID=BookLoans.BookID AND BookLoans.CardNo=Borrower.CardNo AND BookLoans.BranchID=Branch.BranchID
                    (Book ⋈ BookLoans ⋈ Borrower ⋈ Branch))

d. Stephen King 作品在 Central 分馆的书名和册数
πTitle,NCopies(σAuthorName='Stephen King' AND BranchName='Central' AND Author.BookID=Book.BookID AND Book.BookID=BookCopies.BookID AND BookCopies.BranchID=Branch.BranchID
               (Author ⋈ Book ⋈ BookCopies ⋈ Branch))

e. 借过 The Lost Tribe 的读者姓名和地址
πName,Address(σTitle='The Lost Tribe' AND Book.BookID=BookLoans.BookID AND BookLoans.CardNo=Borrower.CardNo(Book ⋈ BookLoans ⋈ Borrower))

f. 借过 Tom 借过的所有书的读者姓名
πName(σQ.CardNo=Borrower.CardNo
     (ρQ(πCardNo,BookID(BookLoans) ÷
         πBookID(σName='Tom' AND Borrower.CardNo=BookLoans.CardNo(Borrower ⋈ BookLoans))) ⋈ Borrower))
```

#### 解析

图书馆题的核心路径是 `Book -> BookLoans -> Borrower` 和 `BookCopies -> Branch`。涉及“所有 Tom 借过的书”时，用除法表达。

---


