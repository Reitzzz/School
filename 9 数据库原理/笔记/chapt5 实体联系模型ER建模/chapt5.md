# Chapter 05 Entity-Relationship (ER) Modelling 实体-联系模型

## 闭卷客观题复习提示
- **实体与属性**：辨析弱实体的条件、复合属性与多值属性的区别。`【闭卷重点·必背】`
- **弱实体连线**：弱实体的识别联系必须是双层菱形，且没有独立主码，只有部分键（虚下划线）。`【闭卷重点·易错】`
- **联系的度与属性**：多对多（M:N）联系的属性挂在菱形上，三元联系不等价于三个二元联系。`【闭卷重点·常考】`
- **ER图形符号**：必须熟记哪种图形代表哪种概念（双层矩形、双线椭圆、虚线椭圆等）。`【闭卷重点·必背】`

## 目录
- [Chapter 05 Entity-Relationship (ER) Modelling 实体-联系模型](#chapter-05-entity-relationship-er-modelling-实体-联系模型)
  - [目录](#目录)
  - [5.1 概念数据建模概述 (Conceptual Data Modelling)](#51-概念数据建模概述-conceptual-data-modelling)
  - [5.2 实体与实体集 (Entities and Entity Sets)](#52-实体与实体集-entities-and-entity-sets)
  - [5.3 属性的分类与特性 (Attributes Classification)](#53-属性的分类与特性-attributes-classification)
    - [5.3.1 简单属性 (Simple Attributes)](#531-简单属性-simple-attributes)
    - [5.3.2 复合属性 (Composite Attributes)](#532-复合属性-composite-attributes)
    - [5.3.3 单值与多值属性 (Single-valued & Multi-valued Attributes)](#533-单值与多值属性-single-valued--multi-valued-attributes)
    - [5.3.4 派生属性与空值 (Derived Attributes & Null Values)](#534-派生属性与空值-derived-attributes--null-values)
  - [5.4 联系与联系集 (Relationships and Relationship Sets)](#54-联系与联系集-relationships-and-relationship-sets)
    - [5.4.1 联系的度 (Degree of Relationship)](#541-联系的度-degree-of-relationship)
    - [5.4.2 三元联系 (Ternary Relationship)](#542-三元联系-ternary-relationship)
    - [5.4.3 联系的属性 (Attributes of Relationships)](#543-联系的属性-attributes-of-relationships)
  - [5.5 弱实体类型 (Weak Entity Types)](#55-弱实体类型-weak-entity-types)
  - [5.6 ER 图标准图形符号规范 (ER Diagram Notations)](#56-er-图标准图形符号规范-er-diagram-notations)

---

## 5.1 概念数据建模概述 (Conceptual Data Modelling)
概念建模是数据库设计的第一步，旨在跳过具体的物理存储细节，抽象地描述现实世界中的业务逻辑。ER模型（Entity-Relationship Model）是目前最主流的概念模型，它将现实世界抽象为“实体”、“属性”和“联系”。

---

## 5.2 实体与实体集 (Entities and Entity Sets)
* **实体 (Entity)**：现实世界中可以独立存在并相互区分的对象或事物（如一个具体的学生、一门课程）。
* **实体集 (Entity Set)**：具有相同类型和属性的实体的集合（如所有“学生”的集合）。在系统实现中通常对应底层的表。

---

## 5.3 属性的分类与特性 (Attributes Classification)
属性是实体或联系所具有的某种特性。根据其结构和取值特点，可以分为以下几类：

### 5.3.1 简单属性 (Simple Attributes)
* **定义**：不能再划分为更小、更简单的独立组件的属性。
* **示例**：身高（Height）。身高是一个原子值，在业务逻辑中无法再进一步拆分。

### 5.3.2 复合属性 (Composite Attributes)
* **定义**：可以进一步划分为更小组件的属性，每个组件都有其独立的含义。
* **示例**：出生日期（DOB, Date of Birth）。它可以被进一步拆分为年（Year）、月（Month）和日（Day）。通过这种层级划分，系统既可以整体访问该属性，也可以单独访问其中的某个组件。

### 5.3.3 单值与多值属性 (Single-valued & Multi-valued Attributes)
* **单值属性**：对一个特定的实体而言，该属性只能取一个值（如一个人只能有一个主身份证号）。
* **多值属性**：一个实体在该属性上可以同时拥有多个值（如一个员工可以有多个电话号码、多项特定技能）。在通用ER图中通常用双线椭圆表示。`【闭卷重点·必背】`

### 5.3.4 派生属性与空值 (Derived Attributes & Null Values)
* **派生属性**：无需直接存储，可以通过其他相关属性计算得出的属性（如通过“出生日期”可以动态计算出“年龄”）。在ER图中用虚线椭圆表示。`【闭卷重点·必背】`
* **空值 (Null Value)**：当实体在某个属性上没有值、或者对应的值未知、不存在时，使用空值进行占位填充。

---

## 5.4 联系与联系集 (Relationships and Relationship Sets)
联系（Relationship）是指多个实体集之间的相互关联关系。联系集则是同类联系的集合。

### 5.4.1 联系的度 (Degree of Relationship)
联系的度是指参与该联系的实体集的个数：
* **一元联系 (Unary/Recursive)**：同一个实体集内部实体之间的联系（如员工表中的经理与普通下属之间的管理联系）。
* **二元联系 (Binary)**：两个实体集之间的联系（如学生实体 `student` 与科目实体 `subject` 之间的修读联系 `enrols in`）。
* **三元联系 (Ternary)**：三个实体集之间同时发生、相互制约的联系。

### 5.4.2 三元联系 (Ternary Relationship)
* **定义**：三个实体集通过同一个关系菱形联合关联。
* **经典示例**：销售关系（Sale）。它同时涉及三个实体集：供应商（Vendor）、商品（Item）以及购买者（Purchaser）。这三者共同构成了一个不可分割的三方贸易事件。
* **核心结论**：在一般情况下，一个三元联系在逻辑上不等价于三个独立的二元联系。`【闭卷重点·易错】`

### 5.4.3 联系的属性 (Attributes of Relationships)
* 联系本身也可以拥有属性，用来记录关联发生时的附加信息。
* **绘图规范**：联系的属性需要用线画在代表联系的菱形符号 (Relationship diamond symbol) 上。
* **经典规则**：通常情况下，只有在多对多（M:N）联系中，将属性附加在联系上才具有实际业务意义；如果是 1:N 联系，该属性通常可以直接合并到 N 端实体的属性中。`【闭卷重点·常考】`

---

## 5.5 弱实体类型 (Weak Entity Types)
* **核心定义**：弱实体类型是指没有自身主码（Primary Key）、必须依赖于另一个强实体（称为父实体或标识实体 Owner/Identifying Entity Type）的存在而存在的实体。`【闭卷重点·必背】`
* **部分键 (Partial Key / Discriminator)**：弱实体虽然没有主码，但通常会有一个部分键（如家属实体中的姓名），用来在同一个父实体所关联的弱实体集合中进行区分。在图中用虚下划线表示。`【闭卷重点·常考】`
* **绘图连线规则**：弱实体除了必须与它所依赖的父实体建立标识联系外，它同样可以根据实际业务需求，与模型中的其他独立普通实体相连。

---

## 5.6 ER 图标准图形符号规范 (ER Diagram Notations)
在绘制 ER 图时，必须遵守严格的符号规范：
1. **矩形 (Rectangle)**：代表强实体类型 (Entity Type)。
2. **双层矩形 (Doubled Rectangle)**：代表弱实体类型 (Weak Entity Type)。
3. **菱形 (Diamond)**：代表普通联系类型 (Relationship Type)。
4. **双层菱形 (Doubled Diamond)**：代表弱实体的识别联系 (Identifying Relationship symbol)，用于明确该弱实体依赖于哪一个父实体。
5. **椭圆 (Ellipse)**：代表属性 (Attribute)。
6. **下划线椭圆**：代表主属性/主码 (Key Attribute)。
7. **虚下划线椭圆**：代表弱实体的部分键 (Partial Key)。
8. **双线椭圆**：代表多值属性 (Multi-valued Attribute)。
9. **虚线椭圆**：代表派生属性 (Derived Attribute)。
