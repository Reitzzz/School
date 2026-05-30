# Assignment 2-3 整理笔记

### 4.1 Films / Artists / Roles

给定：

```text
Films(FilmID, Title, Director, Year, ProductionCost)
Artists(ArtistID, Surname, FirstName, Sex, BirthDate, Nationality)
Roles(FilmID, ActorID, Character)
```

#### 答案

```text
a. Henry Fonda 参演电影标题
πTitle(σFirstName='Henry' AND Surname='Fonda'(Artists)
       ⋈ArtistID=ActorID Roles ⋈ Films)

b. 导演也是演员的电影标题
πTitle(σDirector=ActorID(Films ⋈ Roles))

c. 演员性别全相同的电影标题
πTitle(Films) -
πTitle(σA1.FilmID=A2.FilmID AND A1.Sex<>A2.Sex
       ((Roles ⋈ActorID=ArtistID ρA1(Artists)) ×
        (Roles ⋈ActorID=ArtistID ρA2(Artists))) ⋈ Films)

d. 没有任何演员参演的电影标题
πTitle(Films) - πTitle(Films ⋈ Roles)

e. 参演了所有 2021 年电影的演员
πActorID,FirstName,Surname((πActorID,FilmID(Roles) ÷ πFilmID(σYear=2021(Films))) ⋈Artists)

f. 电影 Transformer 的演员姓名
πFirstName,Surname(σTitle='Transformer'(Films) ⋈ Roles ⋈ActorID=ArtistID Artists)
```

#### 讲解过程

“所有”类查询用除法；“没有”类查询用全集减去已有集合；“导演也是演员”比较 `Films.Director` 与 `Roles.ActorID`。

### 4.2 Library 模式

给定：

```text
Book(BookID, Title, PubID)
Author(BookID, AuthorName, order)
Publisher(PubID, PubName, Address, Phone)
BookCopies(BookID, BranchID, NCopies)
BookLoans(BookID, BranchID, CardNo, DateOut, DueDate, DateReturn)
Branch(BranchID, BranchName, Address)
Borrower(CardNo, Name, Address, Phone)
```

#### 答案

```text
a. Sharpstown 分馆有多少本 The Lost Tribe
πNCopies(σTitle='The Lost Tribe' AND BranchName='Sharpstown'
         (Book ⋈ BookCopies ⋈ Branch))

b. 没有借书的读者姓名
πName(Borrower) - πName(Borrower ⋈ BookLoans)

c. Sharpstown 分馆今天到期的外借图书标题、读者姓名和地址
πTitle,Name,Address(σBranchName='Sharpstown' AND DueDate=CURRENT_DATE
                    (Book ⋈ BookLoans ⋈ Borrower ⋈ Branch))

d. Stephen King 作品在 Central 分馆的书名和册数
πTitle,NCopies(σAuthorName='Stephen King' AND BranchName='Central'
               (Author ⋈ Book ⋈ BookCopies ⋈ Branch))

e. 借过 The Lost Tribe 的读者姓名和地址
πName,Address(σTitle='The Lost Tribe'(Book ⋈ BookLoans ⋈ Borrower))

f. 借过 Tom 借过的所有书的读者姓名
πName((πCardNo,BookID(BookLoans) ÷
      πBookID(σName='Tom'(Borrower ⋈ BookLoans))) ⋈ Borrower)
```

#### 讲解过程

图书馆题的核心路径是 `Book -> BookLoans -> Borrower` 和 `BookCopies -> Branch`。涉及“所有 Tom 借过的书”时，用除法表达。

---
