# 第3章 SQL
## 3.0 写sql遇到的一些note
![1776133048983](image/lec3SQL/1776133048983.png)
## 3.1 SQL 概述

SQL（Structured Query Language）是关系数据库的标准语言，集数据定义、查询、操纵、控制于一体。

**SQL 的特点：**
- 综合统一：DDL + DML + DCL 一体
- 高度**非过程化**：只说"做什么"，不说"怎么做"
- 面向**集合**的操作方式
- 以同一种语法结构提供两种使用方式（交互式 / 嵌入式）
- 语言简洁，易学易用

![1775723523015](image/lec3SQL/1775723523015.png)


**SQL 功能分类：**

| 类别 | 功能 | 关键字 |
|------|------|--------|
| DDL | 数据定义 | CREATE, DROP, ALTER |
| DML | 数据操纵（查询+更新） | SELECT, INSERT, UPDATE, DELETE |
| DCL | 数据控制 | GRANT, REVOKE |

一条完整SQL语句，通常以命令动词开始，以分号';'作为结束符
在交互式SQL执行窗口中，可以一次只执行一条SQL语句，也可以一次执行多条SQL语句（批处理）
在批处理执行方式下，分号既作为前一条SQL语句的结束符，也可
以看做是不同SQL语句之间的分隔符

除常量外，SQL语言中的其他语言成分仅支持西文字符，且（字母）不区分大小写

保留字（关键字）
是 SQL 里“官方规定有特殊意义”的词，不能随便改意思。
例如：create、select、insert、alter、table、view、primary key、if、while。
它们用来表示“你要做什么操作”。

标识符
是你自己起的名字。
比如表名、列名、视图名、存储过程名、变量名。
例如 student、score、order_detail 这种。

常量
就是固定不变的值。
例如数字 100、字符串 “Tom”、日期 2026-04-09。

![1775724069486](image/lec3SQL/1775724069486.png)

---

## 3.2 学生-课程数据库（示例库）

```sql
Student(Sno, Sname, Ssex, Sage, Sdept)
Course(Cno, Cname, Cpno, Ccredit)
SC(Sno, Cno, Grade)
```

- **Student**：学号、姓名、性别、年龄、所在系
- **Course**：课程号、课程名、先修课号、学分
- **SC**：学号、课程号、成绩（选课关系）

---

## 3.3 数据定义（DDL）
一个关系数据库管理系统的**实例**（Instance）中可以建立多个数据库
一个数据库中可以建立**多个模式**
一个模式下通常包括多个表、视图和索引等数据库对象
![1775750503330](image/lec3SQL/1775750503330.png)

### 数据定义
对数据库中“数据的结构”进行规定和管理，而不是操作具体数据内容。
![1775750648483](image/lec3SQL/1775750648483.png)

### **SQL 常用数据类型：**
SQL中域的概念用**数据类型**来实现，定义表的**属性**时需要指明其数据类型及长度

| 类型 | 说明 |
|------|------|
| CHAR(n) | 定长字符串 |
| VARCHAR(n) | 变长字符串 |
| INT / SMALLINT / BIGINT | 整数 |
| NUMERIC(p,d) / DECIMAL(p,d) | 精确小数，由p位数字组成，其中d位小数 |
| REAL| 单精度浮点数 |
| FLOAT(n) | 双精度浮点数 |
| DATE | 日期（YYYY-MM-DD） |
| TIME | 时间（HH:MM:SS） |
| TIMESTAMP | 日期时间（时间戳） |
| INTERVAL | 时间间隔 |
![1775784567946](image/lec3SQL/1775784567946.png)


### 3.3.1 模式（Schema）
![1775750657912](image/lec3SQL/1775750657912.png)
例子：
```sql
-- 为用户WANG定义一个学生-课程模式S-T
CREATE SCHEMA "S_T" AUTHORIZATION WANG;

-- 未定义模式名，默认使用授权用户的名字作为模式名
CREATE SCHEMA AUTHORIZATION WANG;

-- 为用户ZHANG创建了一个模式TEST，并且在其中定义一个表TAB1
CREATE SCHEMA TEST AUTHORIZATION ZHANG;
CREATE TABLE TAB1 ( COL1 SMALLINT,
COL2 INT,
COL3 CHAR(20),
COL4 NUMERIC(10,3),
COL5 DECIMAL(5,2),);

DROP SCHEMA <模式名> <CASCADE|RESTRICT>
-- CASCADE（级联）
-- ●删除模式的同时把该模式中**所有的数据库对象**全部删除
-- RESTRICT（限制）
-- ●如果该模式中定义了下属的数据库对象（如表、视图等），则**拒绝**该删除语句的执行。
-- ●仅当该模式中没有任何下属的对象时才能执行。
-- 删除模式
DROP SCHEMA ZHANG CASCADE;
```

### 3.3.2 基本表
![1775783721696](image/lec3SQL/1775783721696.png)
**创建表：**

```sql
CREATE TABLE Student (
    Sno   CHAR(9)  PRIMARY KEY, --Sno是主码
    Sname CHAR(20) UNIQUE, --Sname取值唯一，unique约束
    Ssex  CHAR(2),
    Sage  SMALLINT,
    Sdept CHAR(20)
);

CREATE TABLE Course (
    Cno    CHAR(4)  PRIMARY KEY,
    Cname  CHAR(40) NOT NULL, -- Coursename不能为空
    Cpno   CHAR(4), -- 先修课号，允许空值（无先修课）
    Ccredit SMALLINT, -- SMALLINT 取值范围 -32768~32767
    FOREIGN KEY (Cpno) REFERENCES Course(Cno)  -- 自参照，Cpno是外码，参照Cno这一列
);

-- 学生选课表（复合主键，外码参照 Student 和 Course）
CREATE TABLE SC (
    Sno   CHAR(9),
    Cno   CHAR(4),
    Grade SMALLINT,
    PRIMARY KEY (Sno, Cno),       -- 复合主键
    /* 表级完整性约束条件，Sno是外码，被参照表是Student */
    FOREIGN KEY (Sno) REFERENCES Student(Sno),
    /* 表级完整性约束条件，Cno是外码，被参照表是Course */
    FOREIGN KEY (Cno) REFERENCES Course(Cno)
);
```

**修改表：**

```sql
ALTER TABLE Student ADD S_entrance DATE;           -- 增加列
ALTER TABLE Student ALTER COLUMN Sage INT;         -- 修改列数据类型
ALTER TABLE Course ADD UNIQUE(Cname);              -- 增加约束，约束Cname取值唯一
```

**删除表：**

```sql
DROP TABLE Student CASCADE;   -- 级联删除（含相关视图、索引等）
DROP TABLE Student RESTRICT;  -- 若有依赖则拒绝
```

### 3.3.3 索引

索引加速查询，但增加存储开销及维护代价。
![1775785132754](image/lec3SQL/1775785132754.png)
```sql
-- 创建索引
CREATE UNIQUE INDEX Stusno ON Student(Sno); -- 指定唯一索引，Sno取值唯一，按学号升序建唯一索引
CREATE UNIQUE INDEX Coucno ON Course(Cno);
CREATE UNIQUE INDEX SCno   ON SC(Sno ASC, Cno DESC); -- 指定对这个表的多列索引，且指定升序/降序，ASC 升序（默认），DESC 降序
CREATE CLUSTER INDEX Stusname ON Student(Sname);  -- 聚簇索引

-- 删除索引
DROP INDEX Stusno;
```

- **UNIQUE**：唯一索引，不允许重复值
- **CLUSTER**：聚簇索引，按索引键顺序物理存储，每张表只能建一个

---

## 3.4 数据查询（SELECT）

### SELECT 通用语法

```sql
SELECT [ALL | DISTINCT] <目标列表达式>, ...
FROM   <表名或视图名>, ...
[WHERE <条件表达式>]
[GROUP BY <列名> [HAVING <条件>]]
[ORDER BY <列名> [ASC | DESC]];
```

**执行顺序：** FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY

---

### 3.4.1 目标子句
SELECT [distinct] column-name-list | expressions | *
定义生成结果表时目标列的**投影方式**（显示哪些列，以什么方式显示）
[distinct]可选的去重，表示重复行只保留一条
column-name-list 只选指定列
expressions 选表达式结果
* 选全部列

#### 基本查询

```sql
-- 查询所有列
SELECT * FROM Student;

-- 查询指定列
SELECT Sno, Sname FROM Student;

-- 消除重复行
SELECT DISTINCT Sno FROM SC;

-- 列别名
SELECT Sname AS 姓名, 2014-Sage AS 出生年份 FROM Student;
```

### 3.4.2 范围子句
FROM tablename { , tablename ...}
指定操作对象
可以在FROM子句中对一个表重新命名（即定义一个别名 alias），只在这一句语句里面有效：
`<table_name> <alias_name>`
```sql
SELECT Sname, Sdept FROM Student AS S; -- 把Student重命名为S
```

### 3.4.3 条件子句

WHERE search_condition
是可选成分，用于定义查询条件

在FROM子句中给出的表只是表明此次查询需要访问这些表，它们之间是通过笛卡儿积运算进行合并的；
如果需要执行它们之间的$\theta$-连接或自然连接运算，则需要在WHERE子句中显式地给出它们的连接条件。

```sql
-- LIKE 模糊查询（ESCAPE 定义转义字符）
SELECT Cno, Ccredit FROM Course WHERE Cname LIKE 'DB\_Design' ESCAPE '\';

-- 范围查询
SELECT Sname, Sdept, Sage FROM Student WHERE Sage BETWEEN 20 AND 23;

-- 集合查询
SELECT Sname, Ssex FROM Student WHERE Sdept IN ('CS','MA','IS');
```

### 3.4.4 单表查询

#### 选择表中的若干列
```sql
SELECT Sname, Sdept FROM Student;
SELECT * FROM Student; -- 选全部列
SELECT Sname, 2026 - Sage /*假设当时为2026年*/
FROM Student;
```
![1775986209776](image/lec3SQL/1775986209776.png)
![1775996551907](image/lec3SQL/1775996551907.png)
![1775997868069](image/lec3SQL/1775997868069.png)
给这个字符串列取了个别名叫出生年份，因为是常量，所以结果表中这一列的值都是一样的

#### 选择表中的若干元组
![1775997947184](image/lec3SQL/1775997947184.png)
![1775998475821](image/lec3SQL/1775998475821.png)
![1775998499259](image/lec3SQL/1775998499259.png)
![1775998513908](image/lec3SQL/1775998513908.png)
![1775998522191](image/lec3SQL/1775998522191.png)
![1775998676114](image/lec3SQL/1775998676114.png)
![1775998686092](image/lec3SQL/1775998686092.png)
![1775998704347](image/lec3SQL/1775998704347.png)
![1775998761862](image/lec3SQL/1775998761862.png)
![1775998804516](image/lec3SQL/1775998804516.png)
![1775998827434](image/lec3SQL/1775998827434.png)
![1776002199665](image/lec3SQL/1776002199665.png)

#### 聚集函数
![1776002279630](image/lec3SQL/1776002279630.png)
![1776002312806](image/lec3SQL/1776002312806.png)
![1776003319161](image/lec3SQL/1776003319161.png)
![1776006263586](image/lec3SQL/1776006263586.png)

#### 对查询结果分组
GROUP BY 子句将查询结果按照指定列进行分组，常与聚集函数一起使用
```sql
SELECT Sdept, COUNT(*) AS num_students
FROM Student
GROUP BY Sdept;
```
按指定的一列或多列值分组，值相等的为一‘组’
对查询结果分组后，**聚集函数**将分别作用于每个‘组’，在SELECT子句中对每一个‘组’分别进行统计计算，实现分类统计查询

HAVING 子句用于对分组后的结果进行过滤，常与 GROUP BY 子句一起使用
```sql
SELECT Sdept, COUNT(*) AS num_students
FROM Student
GROUP BY Sdept
HAVING COUNT(*) > 10; -- 只显示学生人数超过10的系
```
根据GROUP BY子句的分组结果，定义分组选择条件
● 在HAVING子句中给出的查询条件是定义在分组后的‘组’（元组集合）上的，通常是根据某种统计计算的结果来定义‘组’的选择条件
● 只有满足条件group_condition的‘组’才会被保留下来，用于生成最终的查询结果

聚集函数的使用：只能用在 SELECT子句 或 HAVING子句 中
![1776007514186](image/lec3SQL/1776007514186.png)

### 3.4.5 连接查询
连接查询：同时涉及两个或两个以上的表的查询
连接条件或连接谓词：用来连接两个表的条件

#### 等值连接
```sql
SELECT Student.*, SC.*
FROM Student, SC
WHERE Student.Sno = SC.Sno;
```
等值连接：连接条件是两个表中某些列的值相等
这里的=是连接运算符，连接条件是Student表中的Sno列与SC表中的Sno列的值相等

如果没有索引，连接查询的效率较低，对每一个Student表中的元组，都要扫描整个SC表来寻找满足连接条件的元组，时间复杂度为O(n*m)，其中n和m分别是两个表的大小

如果在Student表的Sno列（连接字段）上建立了索引，那么对于每一个Student表中的元组，可以通过索引快速定位到SC表中满足连接条件的元组，时间复杂度可以降低到O(n*log(m))

排序合并连接：首先对两个表按照连接字段进行**排序**，然后通过一次扫描来完成连接操作，时间复杂度为$O(n*\log(n) + m*\log(m) + n + m)$
![1776063059632](image/lec3SQL/1776063059632.png)

#### 自身连接
一个表和自己进行连接查询，称为自身连接，需要给表起别名以示区别
```sql
SELECT FIRST.Cno, SECOND.Cpno
FROM Course FIRST, Course SECOND
WHERE FIRST.Cpno = SECOND.Cno; -- 查询课程的先修课
```
![1776063570683](image/lec3SQL/1776063570683.png)

#### 外连接
普通连接操作只输出满足连接条件的元组，外连接操作以指定表为连接主体，将主体表中不满足连接条件的元组一并输出
```sql
SELECT Student.Sno,Sname,Ssex,Sage,Sdept,Cno,Grade
FROM Student LEFT OUTER JOIN SC ON (Student.Sno=SC.Sno);
```
![1776063872265](image/lec3SQL/1776063872265.png)

#### 多表连接
```sql
SELECT Student.Sno, Sname, Cname, Grade
FROM Student, SC, Course /*多表连接*/
WHERE Student.Sno = SC.Sno
AND SC.Cno = Course.Cno;
```

### 3.4.6 嵌套查询
一个 SELECT-FROM语句 称为一个**查询块**（query block），一个查询块中又包含另一个查询块，称为嵌套查询
```sql
SELECT Sname /*外层查询/父查询*/
FROM Student
WHERE Sno IN
( SELECT Sno /*内层查询/子查询*/
 FROM SC
 WHERE Cno = '2');
```
最常见的是在WHERE/FROM子句中嵌套查询块
子查询的限制:不能使用ORDER BY子句

不相关子查询：子查询中的查询块与外层查询块没有任何联系，子查询的结果是一个常量值，比如上面那个例子

相关子查询：子查询中的查询块与外层查询块有联系，子查询的结果依赖于外层查询块中当前处理的元组
```sql
SELECT Sname
FROM Student s
WHERE EXISTS (
    SELECT *
    FROM SC
    WHERE SC.Sno = s.Sno AND Grade >= 90
);
```
这里的子查询块中，SC.Sno = s.Sno 这个条件使得子查询块与外层查询块联系起来了，子查询块的结果依赖于外层查询块中当前处理的元组s
外层先取s的一个元组，然后把这个元组的Sno值传递给子查询块，子查询块根据这个Sno值来判断是否存在满足条件的选课记录，如果存在则返回true，外层查询块就会把这个学生的名字包含在结果中
![1776065502676](image/lec3SQL/1776065502676.png)
![1776065588776](image/lec3SQL/1776065588776.png)
![1776065628112](image/lec3SQL/1776065628112.png)
![1776065753513](image/lec3SQL/1776065753513.png)
![1776065768930](image/lec3SQL/1776065768930.png)
![1776065854722](image/lec3SQL/1776065854722.png)
如果想查询每个学生超过平均成绩的课程，可以使用相关子查询来实现：
```sql
SELECT Sno, Cno
FROM SC x
WHERE Grade >= ( SELECT AVG(Grade)
FROM SC y
WHERE y.Sno = x.Sno );
```
![1776065960179](image/lec3SQL/1776065960179.png)
![1776065995745](image/lec3SQL/1776065995745.png)

**带有EXISTS谓词的相关子查询**
不返回任何数据，只产生逻辑真值“true”或逻辑假值“false”
如果查询结果表中至少有一行满足WHERE子句中的条件，则EXISTS谓词返回true，否则返回false

由EXISTS引出的子查询，其SELECT的目标列表达式通常都用 *，因为带EXISTS的子查询只返回真值或假值，给出列名无实际意义。

例子：查询所有选修了1号课程的学生姓名。
```sql
SELECT Sname
FROM Student s
WHERE EXISTS ( -- 相关子查询，结果是true的就加入结果表
    SELECT * -- 这里的*没有实际意义
    FROM SC
    WHERE SC.Sno = s.Sno AND Cno = '1'
);
```
一些带EXISTS或NOT EXISTS谓词的子查询不能被其他形式的子查询等价替换

SQL语言中没有全称量词 （For all）
- 可以把带有全称量词的谓词转换为等价的带有存在量词的谓词
$\forall x P(x) \equiv \neg \exists x \neg P(x)$
![1776066208641](image/lec3SQL/1776066208641.png)
![1776066316116](image/lec3SQL/1776066316116.png)
![1776066462953](image/lec3SQL/1776066462953.png)
![1776066475017](image/lec3SQL/1776066475017.png)
![1776066482468](image/lec3SQL/1776066482468.png)
![1776066494585](image/lec3SQL/1776066494585.png)
x是SC的一个元组，，限定x.Sno = S4.Sno，表示x是S4选修的课程记录
```sql
SELECT S.sno
FROM S
WHERE NOT EXISTS (
SELECT *
FROM SC x
WHERE x.sno='S4' and NOT EXISTS (-- 可以在这里看成一个子查询块，检查S4选过的每一门课x.cno，是否有被漏掉的
SELECT *
FROM SC y
WHERE y.sno=S.sno and y.cno=x.cno ) );
```
中层：找 S4 选过的每一门课 x.cno
内层：检查当前学生 S 是否也选过这门 x.cno
如果存在某门 S4 选过但 S 没选过的课（漏掉的），中层 EXISTS 为真，外层就把这个学生排除。
只有当“找不到任何漏掉的课程”时，外层 NOT EXISTS 才成立，学生被保留

```sql
SELECT S.sno
FROM S
WHERE NOT EXISTS (
SELECT *
FROM SC x
WHERE x.sno='S4' and
x.cno NOT IN ( SELECT y.cno
FROM SC y
WHERE y.sno=S.sno) );
```
第二种表示方法

### 3.4.7 集合查询
参加集合操作的各个子查询需要满足
- 结果的列数必须相同
- 对应列的数据类型也必须相同

#### UNION
- UNION：将多个查询结果合并起来时，系统自动去掉重复元组
- UNION ALL：将多个查询结果合并起来时，保留重复元组
[例3.64]查询计算机科学系的学生及年龄不大于19岁的学生。
```sql
SELECT *
FROM Student
WHERE Sdept = 'CS'
UNION
SELECT *
FROM Student
WHERE Sage <= 19;
```

#### INTERSECT
[例3.66] 查询计算机科学系的学生与年龄不大于19岁的学生的交集。
```sql
SELECT *
FROM Student
WHERE Sdept='CS'
INTERSECT
SELECT *
FROM Student
WHERE Sage<=19;
```
实际上就是查询计算机科学系中年龄不大于19岁的学生。
```sql
SELECT *
FROM Student
WHERE Sdept = 'CS' AND Sage<=19
```

#### 差
查询计算机科学系的学生与年龄不大于19岁的学生的差集。
```sql
SELECT *
FROM Student
WHERE Sdept='CS'
EXCEPT -- 也可以写成 MINUS，差集
SELECT *
FROM Student
WHERE Sage <= 19
```

### 3.4.8 基于派生表的查询
![1776067334221](image/lec3SQL/1776067334221.png)
![1776067346017](image/lec3SQL/1776067346017.png)

## 3.5 数据更新（DML）

### 3.5.1 插入元组
```sql
INSERT
INTO <表名> [(列名1, 列名2, ...)]
VALUES (值1, 值2, ...);
```
INTO子句
- 指定要插入数据的表名及属性列
- 属性列的顺序可与表定义中的顺序不一致
- 没有指定属性列：表示要插入的是一条完整的元组，且属性列属性与表定义中的顺序一致
- 指定部分属性列：插入的元组在其余属性列上取空值
VALUES子句
- 提供的值必须与INTO子句匹配：值的个数 & 值的类型
![1778313876862](image/lec3SQL/1778313876862.png)
![1778314183855](image/lec3SQL/1778314183855.png)

还可以插入子查询，不一定要是VALUES子句
![1778314250468](image/lec3SQL/1778314250468.png)

### 3.5.2 修改数据
```sql
UPDATE  <表名>
SET  <列名> = <表达式>  [ , <列名> = <表达式> ] …
[WHERE  <条件>] ;
```
修改指定表中满足WHERE子句条件的元组
![1778314304047](image/lec3SQL/1778314304047.png)
![1778314316127](image/lec3SQL/1778314316127.png)
关系数据库管理系统在执行修改语句时会检查修改操作是否破坏表上已定义的完整性规则：
- 实体完整性
- 参照完整性
- 用户定义的完整性约束
    - NOT NULL 约束
    - UNIQUE 约束
    - 值域约束

### 3.5.3 删除数据
```sql
DELEYE
FROM <表名>
[WHERE <条件>];
```
删除指定表中满足WHERE子句条件的元组
![1778314540082](image/lec3SQL/1778314540082.png)

## 3.6 事务处理
### 3.6.1 事务概述
事务(Transaction)是用户定义的一个**数据库操作序列**，这些操作要么全做，要么全不做，是一个不可分割的工作单位。
在关系数据库中，一个事务可以由一条SQL语句或一组SQL语句组成，它们在数据库内的执行就构成了一个事务。
事务是恢复和并发控制的基本单位
![1778324086094](image/lec3SQL/1778324086094.png)
如果数据库中的所有数据都满足**完整性约束**，就称数据库处于**一致性状态**
事务的作用是把数据库从一个一致性状态安全地带到另一个一致性状态

破坏完整性的三类因素：
- concurrency（并发）: 多个事务同时访问数据库，可能会导致数据不一致
- abort（中止）: 事务执行过程中发生错误，导致事务无法完成
- crash（崩溃）: 系统崩溃导致事务无法完成

一个例子：银行的转账操作，涉及两个账户的余额更新，如果在转账过程中发生错误或系统崩溃，可能会导致一个账户的余额减少了，但另一个账户的余额没有增加，造成数据不一致
这个应用的一致性约束就是：两个账户的余额之和必须保持不变

事务的特性(ACID)：
- 原子性：事务中的所有操作要么全部执行成功，要么全部不执行，不能只执行其中的一部分
- 一致性：事务执行前后，数据库都必须处于**一致性状态**
- 隔离性：一个事务的执行不应该被其他事务干扰，多个事务之间应该**相互隔离**
- 持续性：一旦事务提交成功，对数据库的修改应该是**永久性**的，即使系统发生崩溃也不应该丢失
![1778331643857](image/lec3SQL/1778331643857.png)

### 3.6.2 定义事务
#### 事务启动
有三种方式：
- 数据定义命令DDL
    - 每一条数据定义命令都将被作为一个单独的事务来执行
    - 在此之前，用户当前正在运行的事务(active事务)将被自动提交
- 将系统设为自动提交方式
    - 在自动提交模式下，每一条SQL语句都被当作一个单独的事务来执行，语句执行成功后自动提交
- 数据操纵命令(DML)
    - 当DBMS接收到来自某个用户的数据操纵请求时，首先检查该用户当前有没有正在运行的active事务
    - 如果有，则把该SQL语句加入到这个active事务中继续执行
    - 如果没有，则新建一个active事务，该请求就被作为新事务的第一条数据访问请求去执行

#### 事务结束
事务的结束有两种方式：提交(commit)和回滚(rollback)
##### 提交
用于提交用户当前的active事务。如果能够成功完成提交，该事务在执行过程中对于数据库的所有修改操作结果都将**永久地**反映到数据库中，并且不可被取消。
事务的提交操作也可能失败，其原因包括：
●发生系统故障
●在提交阶段执行的数据完整性检查不满足要求
在事务提交失败后，用户可以通过回退(Rollback)操作来取消当前事务
由系统自动提交的事务，如果提交失败，系统将自动执行事务的
回退操作

##### 回滚
取消当前active事务在执行过程中的所有操作结果，将数据库回滚至该事务开始前的状态，以便重新执行或放弃（abort）该事务
事务的执行过程不可能被‘回滚’，执行rollback操作是为了撤销该事务在之前的执行过程中对数据库中的数据所作的修改

保存点 (savepoint)
- 在事务的执行过程中，用户可以为该事务设置若干个保存点
- 用户事务可以使用 Rollback 命令将当前事务回退到前面的某个保存点sp，放弃“在保存点sp之后，回退操作之前”执行过的对数据库的所有访问操作，并继续执行当前事务
- 不带保存点的回退操作将结束并放弃整个事务

### 3.6.3 有关事务的SQL语句
![1778326037910](image/lec3SQL/1778326037910.png)
```sql
BEGIN TRANSACTION;
```
```sql
SET  TRANSACTION  READONLY | READWRITE
```
定义新事务的类型
●READONLY：只读型事务，在事务的运行过程中只能执行对数据库的‘读’操作，而不能执行‘写’类型的操作，直到定义新的事务类型
●READWRITE：读/写型事务，在事务运行过程中可以执行对数据库的‘读/写’操作，这是事务的缺省类型定义

```sql
SET  TRANSACTION  ISOLATION  LEVEL
READUNCOMMITTED | READCOMMITTED | READREPEATABLE | SERIALIZABLE ;
```
定义新事务的隔离级别
有四种不同的隔离级别可供选择
事务的隔离级别与事务并发所采用的封锁策略紧密相关。选择不同的隔离级别，系统所采用的封锁策略则不同！
- READ UNCOMMITTED：读未提交级别，不加锁就读，允许一个事务读取另一个事务尚未提交的修改结果，可能会导致脏读（dirty read）
- READ COMMITTED：读已提交级别，读之前加个锁，读完就释放，禁止一个事务读取另一个事务尚未提交的修改结果，可能会导致不可重复读（non-repeatable read）
- REPEATABLE READ：可重复读级别，读之前加个锁，一直持有知道失误结束，保证同一事务内多次读结果一致，避免不可重复读，但可能会导致幻读（phantom read）
- SERIALIZABLE：可串行化级别，通过调度策略让并发事务效果等价于串行执行，隔离性最强，但并发性能最低。

```sql
SAVEPOINT <保存点名>;
```
定义一个保存点，用户可以在事务执行过程中设置多个保存点，以便在需要时回滚到这些保存点
```sql
ROLLBACK TO <保存点名>;
```
将当前事务回滚到指定的保存点，放弃自该保存点以来对数据库的所有修改操作，并继续执行当前事务


## 3.7 SQL中的空值
一般有以下几种情况：
- 该属性应该有一个值，但目前不知道它的具体值
- 该属性不应该有值
- 由于某种原因不便于填写

### 3.7.1 空值的产生
![1778328975612](image/lec3SQL/1778328975612.png)
将Student表中学生号为“201215200”的学生所属的系
改为空值。
```sql
UPDATE Student
SET Sdept = NULL
WHERE Sno='201215200';
```

### 3.7.2 空值的判断
判断一个属性的值是否为空值，用IS NULL或IS NOT NULL来表示
例：从Student表中找出漏填了数据的学生信息
```sql
SELECT  *
FROM   Student
WHERE  Sname  IS NULL  or 
Ssex  IS NULL  or 
Sage  IS NULL  or 
Sdept  IS NULL;
```

### 3.7.3 空值相关的完整性约束
属性定义（或者域定义）中
- 有 NOT NULL 约束条件的不能取空值
- 主码（primary key）属性不能取空值
- UNIQUE唯一性约束：当属性值非空时，其值在表中具有唯一性
可以通过 `UNIQUE + NOT NULL` 约束来确保表中元组的唯一性

### 3.7.4 空值的运算
运算规则
- 空值与另一个值（包括另一个空值）的算术运算的结果为空值
- 空值与另一个值（包括另一个空值）的比较运算的结果为UNKNOWN
- 有UNKNOWN后，传统二值（TRUE，FALSE）逻辑就扩展成了三值逻辑

在数据查询语句的WHERE子句和HAVING子句中，只有条件表达式的判断结果为TRUE时，当前元组（或组GROUP）才被作为结果输出
在关系数据库管理系统中，一般不支持三值逻辑，统一规定：空值参与比较运算的结果为逻辑**假FALSE**
![1778329105021](image/lec3SQL/1778329105021.png)
![1778329121506](image/lec3SQL/1778329121506.png)
NOT (Grade >= 60) 在 Grade 为 NULL 时结果是 UNKNOWN，WHERE 只保留 TRUE，所以缺考（NULL）不会被选出来

## 3.8 视图
查询定义的“虚拟表”，只保存定义，不保存数据（查询时再从基本表取）
可直接增删改；视图是否可更新取决于定义（复杂视图常不可更新）。
用途：表用于存储；视图用于简化查询、封装逻辑、限制访问范围（安全性）
### 3.8.1 定义视图
```sql
CREATE  VIEW  <视图名>  [(<列名>  [,<列名>]…)]
AS  <子查询>
[WITH  CHECK  OPTION];
```
WITH CHECK OPTION：对视图进行UPDATE，INSERT和DELETE操作时要保证更新、插入或删除的行满足视图定义中的谓词条件（即子查询中的条件表达式）
子查询可以是任意的SELECT语句，是否可以含有ORDER BY子句和DISTINCT短语，取决于具体系统的实现。

组成视图的属性列名：**全部省略** 或 **全部指定**
- 全部省略
由子查询中SELECT目标列中的诸字段的名字作为视图中对应列的
列名
- 明确指定视图的所有列名
具体到每一列，可以继续沿用子查询中对应目标列的列名，也可以启用一个新的更合适的列名，但必须保证视图中列名的唯一性

当出现下列情况时，必须明确指定视图的所有列名
- 某个目标列是聚集函数或列表达式
- 多表连接时选出了几个同名列作为视图的字段
- 需要在视图中为某个列启用新的更合适的名字

关系数据库管理系统执行CREATE VIEW语句时只是把视图定义存入数据字典，并不执行其中的SELECT语句。
在对视图查询时，按视图的定义从基本表中将数据查出。

例：建立信息系学生的视图。
```sql
CREATE  VIEW  IS_Student
AS 
SELECT  Sno, Sname, Sage
FROM     Student
WHERE  Sdept = 'IS';
```
![1778329500815](image/lec3SQL/1778329500815.png)
![1778329537685](image/lec3SQL/1778329537685.png)
若一个视图是从单个基本表导出的，并且只是去掉了基本表的某些行
和某些列，但保留了主码，我们称这类视图为**行列子集视图**。

上面的IS_Student视图就是一个行列子集视图。
![1778330059080](image/lec3SQL/1778330059080.png)
![1778330149160](image/lec3SQL/1778330149160.png)
![1778330267646](image/lec3SQL/1778330267646.png)

### 3.8.2 删除视图
```sql
DROP VIEW <视图名> [CASCADE];
```
如果该视图上还导出了其他视图，使用CASCADE级联删除语句，把该视图和由它导出的所有视图一起删除。否则，系统将拒绝当前的视图删除操作。
在删除基本表时如果使用了CASCADE选项，由该基本表导出的所有视图定义都会被连带一起删除；否则，必须先显式地使用DROP VIEW语句删除所有相关的视图，然后才能再删除基本表。

例子：
```sql
删除视图BT_S和IS_S1
DROP VIEW BT_S;       /*成功执行*/
DROP VIEW IS_S1;      /*拒绝执行*/    
/* 原因：在IS_S1上建有视图IS_S2 */
```
要删除IS_S1，需使用级联删除：
```sql
DROP VIEW IS_S1 CASCADE;            
/* 成功执行，并且连带删除掉视图 IS_S1 和 IS_S2 */
```

### 3.8.3 操作视图
对视图可以作**查询**操作
视图上的查询操作将首先被改写为**基表上的查询操作**，然后才能得到执行
一般不允许执行视图上的更新操作，只有满足以下条件才可以： 
- 视图的每一行必须对应基表的唯一一行
- 视图的每一列必须对应基表的唯一一列
同时满足上述两个条件的视图被称为**可更新视图**
视图上的数据更新操作需要被转化为基表上的数据更新操作，只有在可更新视图上才允许执行数据更新操作(delete、insert、update)，否则系统将拒绝执行。

对用户而言，查询视图和查询基本表没有任何区别，视图就像一个虚拟的基本表一样，可以直接查询它。
关系数据库管理系统实现视图查询的方法是**视图消解法**
![1778331309482](image/lec3SQL/1778331309482.png)
![1778331359274](image/lec3SQL/1778331359274.png)

### 3.8.4 视图的作用
当视图中数据不是直接来自基本表时，定义视图能够简化用户的操作
- 基于多张表连接形成的视图
- 基于复杂嵌套查询的视图
- 含导出属性的视图
![1778331441304](image/lec3SQL/1778331441304.png)
![1778331476151](image/lec3SQL/1778331476151.png)
