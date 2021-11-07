#### SQL

DB(数据库)，存储数据的“仓库”，是一系列有组织的数据

DBMS:数据库管理系统，MySQL，Oracle等；

SQL：结构化查询语言，用来访问和操作数据库系统



##### 为什么要用数据库

数据庞大时难以管理，重复代码冗杂。

而关系数据库的关系模型是基于数学理论建立的。我们把域（Domain）定义为一组具有相同数据类型的值的集合，给定一组D1,D2,...,Dn，它们的笛卡尔集定义为D1×D2×……×Dn={(d1,d2,...,dn)|di∈Di,i=1,2,...,n}， 而D1×D2×……×Dn的子集叫作在域D1,D2,...,Dn上的关系，表示为R(D1,D2,...,Dn)，这里的R表示$#%&^@!&$#;!~％¥%:(……算了，根本讲不明白，大家也不用理解。

基于数学理论的关系模型虽然讲起来挺复杂，但是，基于日常生活的关系模型却十分容易理解.比如一个班级的学生信息，根据班级id找到对应班级下的信息。



##### 主键

对于关系表，有个很重要的约束，就是任意两条记录不能重复。不能重复不是指两条记录不完全相同，而是指能够通过某个字段唯一区分出不同的记录，这个字段被称为**主键**。

对主键的要求，最关键的一点是：记录一旦插入到表中，主键最好不要再修改，因为主键是用来唯一定位记录的，修改了主键，会造成一系列的影响。

由于主键的作用十分重要，如何选取主键会对业务开发产生重要影响。如果我们以学生的身份证号作为主键，似乎能唯一定位记录。然而，身份证号也是一种业务场景，如果身份证号升位了，或者需要变更，作为主键，不得不修改的时候，就会对业务产生严重影响。所以，选取主键的一个基本原则是：不使用任何业务相关的字段作为主键。

因此，身份证号、手机号、邮箱地址这些看上去可以唯一的字段，均*不可*用作主键。

作为主键最好是完全业务无关的字段，我们一般把这个字段命名为`id`。常见的可作为`id`字段的类型有：

1. 自增整数类型：数据库会在插入数据时自动为每一条记录分配一个自增整数，这样我们就完全不用担心主键重复，也不用自己预先生成主键；
2. 全局唯一GUID类型：使用一种全局唯一的字符串作为主键，类似`8f55d96b-8acc-4636-8cb8-76bf8abc2f57`。GUID算法通过网卡MAC地址、时间戳和随机数保证任意计算机在任意时间生成的字符串都是不同的，大部分编程语言都内置了GUID算法，可以自己预算出主键。

对于大部分应用来说，通常自增类型的主键就能满足需求。我们在`students`表中定义的主键也是`BIGINT NOT NULL AUTO_INCREMENT`类型。 **如果使用INT自增类型，那么当一张表的记录数超过2147483647（约21亿）时，会达到上限而出错。使用BIGINT自增类型则可以最多约922亿亿条记录。**

###### 联合主键

关系数据库实际上还允许通过多个字段唯一标识记录，即两个或更多的字段都设置为主键，这种主键被称为联合主键。

对于联合主键，允许一列有重复，只要不是所有主键列都重复即可：

| id_num | id_type | other columns... |
| :----: | :-----: | :--------------: |
|   1    |    A    |       ...        |
|   2    |    A    |       ...        |
|   3    |    B    |       ...        |

如果我们把上述表的`id_num`和`id_type`这两列作为联合主键，那么上面的3条记录都是允许的，因为没有两列主键组合起来是相同的。

没有必要的情况下，我们尽量不使用联合主键，因为它给关系表带来了复杂度的上升。



###### 外键

在`students`表中，通过`class_id`的字段，可以把数据与另一张表关联起来，这种列称为`外键`

外键并不是通过列名实现的，而是通过定义外键约束实现的：

```sql
ALTER TABLE students
ADD CONSTRAINT fk_class_id
FOREIGN KEY (class_id)
REFERENCES classes (id);
```

其中，外键约束的名称`fk_class_id`可以任意，`FOREIGN KEY (class_id)`指定了`class_id`作为外键，`REFERENCES classes (id)`指定了这个外键将关联到`classes`表的`id`列（即`classes`表的主键）。

通过定义外键约束，关系数据库可以保证无法插入无效的数据。即如果`classes`表不存在`id=99`的记录，`students`表就无法插入`class_id=99`的记录。

由于外键约束会降低数据库的性能，大部分互联网应用程序为了追求速度，并不设置外键约束，而是仅靠应用程序自身来保证逻辑的正确性。这种情况下，`class_id`仅仅是一个普通的列，只是它起到了外键的作用而已。

要删除一个外键约束，也是通过`ALTER TABLE`实现的：

```sql
ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```



关系数据库通过外键可以实现一对多、多对多和一对一的关系。外键既可以通过数据库来约束，也可以不设置约束，仅依靠应用程序的逻辑来保证。



##### 索引

在关系数据库中，如果有上万甚至上亿条记录，在查找记录的时候，想要获得非常快的速度，就需要使用索引。

索引是关系数据库中**对某一列或多个列的值进行预排序的数据结构**。通过使用索引，可以让数据库系统不必扫描整个表，而是直接定位到符合条件的记录，这样就大大加快了查询速度。

举例：对于`students`表

| id   | class_id | name | gender | score |
| :--- | :------- | :--- | :----- | :---- |
| 1    | 1        | 小明 | M      | 90    |
| 2    | 1        | 小红 | F      | 95    |
| 3    | 1        | 小军 | M      | 88    |

如果要经常根据`score`列进行查询，就可以对`score`列创建索引：

```sql
ALTER TABLE students
ADD INDEX idx_score (score);
```

使用`ADD INDEX idx_score (score)`就创建了一个名称为`idx_score`，使用列`score`的索引。索引名称是任意的，索引如果有多列，可以在括号里依次写上，例如：

```SQL
ALTER TABLE students
ADD INDEX idx_name_score (name, score);
```

索引的效率取决于索引列的值是否散列，即该列的值如果越互不相同，那么索引效率越高。反过来，如果记录的列存在大量相同的值，例如`gender`列，大约一半的记录值是`M`，另一半是`F`，因此，对该列创建索引就没有意义。

可以对一张表创建多个索引。索引的优点是**提高了查询效率**，缺点是**在插入、更新和删除记录时，需要同时修改索引**，因此，索引越多，插入、更新和删除记录的速度就越慢。

**对于主键，关系数据库会自动对其创建主键索引。使用主键索引的效率是最高的，因为主键会保证绝对唯一。**



###### 唯一索引

在设计关系数据表的时候，看上去唯一的列，例如身份证号、邮箱地址等，因为他们具有业务含义，因此不宜作为主键。

但是，这些列根据业务要求，又具有唯一性约束：**即不能出现两条记录存储了同一个身份证号**。这个时候，就可以给该列添加一个唯一索引。例如，我们假设`students`表的`name`不能重复：

```sql
ALTER TABLE students
ADD UNIQUE INDEX uni_name (name);
```

通过`UNIQUE`关键字我们就添加了一个唯一索引。

也可以只对某一列添加一个唯一约束而不创建唯一索引：

```sql
ALTER TABLE students
ADD CONSTRAINT uni_name UNIQUE (name);
```

这种情况下，`name`列没有索引，但仍然具有唯一性保证。

无论是否创建索引，对于用户和应用程序来说，使用关系数据库不会有任何区别。这里的意思是说，当我们在数据库中查询时，如果有相应的索引可用，数据库系统就会自动使用索引来提高查询效率，如果没有索引，查询也能正常执行，只是速度会变慢。因此，索引可以在使用数据库的过程中逐步优化。



#### 查询数据

```sql
-- 查询 `students` 表的所有数据
SELECT * FROM students; 
```



```sql
-- 计算 100 + 200
SELECT 100+200;
```

上述查询会直接计算出表达式的结果。虽然SELECT可以用作计算，但它并不是SQL的强项。但是，不带FROM子句的SELECT语句有一个有用的用途，就是用来判断当前到数据库的连接是否有效

###### 条件查询

```sql
SELECT * FROM students WHERE score >= 80;
```



如果不加括号，条件运算按照`NOT`、`AND`、`OR`的优先级进行，即`NOT`优先级最高，其次是`AND`，最后是`OR`。加上括号可以改变优先级。

**常用的条件表达式：**

| 条件                 | 表达式举例1     | 表达式举例2      | 说明                                              |
| :------------------- | :-------------- | :--------------- | :------------------------------------------------ |
| 使用=判断相等        | score = 80      | name = 'abc'     | 字符串需要用单引号括起来                          |
| 使用>判断大于        | score > 80      | name > 'abc'     | 字符串比较根据ASCII码，中文字符比较根据数据库设置 |
| 使用>=判断大于或相等 | score >= 80     | name >= 'abc'    |                                                   |
| 使用<判断小于        | score < 80      | name <= 'abc'    |                                                   |
| 使用<=判断小于或相等 | score <= 80     | name <= 'abc'    |                                                   |
| 使用<>判断不相等     | score <> 80     | name <> 'abc'    |                                                   |
| 使用LIKE判断相似     | name LIKE 'ab%' | name LIKE '%bc%' | %表示任意字符，例如'ab%'将匹配'ab'，'abc'，'abcd' |



###### 投影查询

使用`SELECT * FROM <表名> WHERE <条件>`可以选出表中的若干条记录。我们注意到返回的二维表结构和原表是相同的，即结果集的所有列与原表的所有列都一一对应。

如果我们只希望返回某些列的数据，而不是所有列的数据，我们可以用`SELECT 列1, 列2, 列3 FROM ...`，让结果集仅包含**指定列**。这种操作称为**投影查询**。

```sql
SELECT id, score, name FROM students;
```

`SELECT`语句可以对结果集的列进行重命名

```sql
SELECT id, score points, name FROM students;

SELECT id, score points, name FROM students WHERE gender = 'M';
```



###### 排序

我们使用SELECT查询时，细心的读者可能注意到，查询结果集通常是按照`id`排序的，也就是根据主键排序。这也是大部分数据库的做法。如果我们要根据其他条件排序怎么办？可以加上`ORDER BY`子句。例如按照成绩从低到高进行排序：

```sql
SELECT * FROM students ORDER BY score	
-- 倒序,加上 DESC
SELECT * FROM students ORDER BY score DESC

-- 如果有WHERE子句，那么ORDER BY子句要放到WHERE子句后面
SELECT id, name, gender, score
FROM students
WHERE class_id = 1
ORDER BY score DESC;

```



###### 分页

分页实际上就是从结果集中“截取”出第M~N条记录。这个查询可以通过`LIMIT <M> OFFSET <N>`子句实现

```sql
-- 1 将成绩从低到高排列
SELECT id, name, gender, score FROM students ORDER BY score DESC;

-- 2 查询LIMIT 3 OFFSET 0表示，对结果集从0号记录开始，最多取3条。注意SQL记录集的索引从0开始。
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 0;

-- 3 查询第2页,从索引3开始，查询3条；
SELECT id , name ,gender ,score FEOM students ORDER BY score DESC LIMIT 3 OFFSET 3
-- 4 查询第3页,从索引6开始，查询3条；
SELECT id , name ,gender ,score FEOM students ORDER BY score DESC LIMIT 3 OFFSET 6

```

- `LIMIT`总是设定为`pageSize`；查询条数
- `OFFSET`计算公式为`pageSize * (pageIndex - 1)`。pageIndex 是指你当前查询的第几页
- `OFFSET`超过了查询的最大数量并不会报错，而是得到一个空的结果集。共10条，从15条开始,LIMIT 3 OFFSET 15;



###### 聚合查询

比如查询条数

```sql
SELECT COUNT(*) FROM students	

-- 也可使用 WHERE 条件，
SELECT COUNT(*) boys FROM students WHERE gender = 'M';
```

除了`COUNT()`函数外，SQL还提供了如下聚合函数：

| 函数 | 说明                                   |
| :--- | :------------------------------------- |
| SUM  | 计算某一列的合计值，该列必须为数值类型 |
| AVG  | 计算某一列的平均值，该列必须为数值类型 |
| MAX  | 计算某一列的最大值                     |
| MIN  | 计算某一列的最小值                     |

注意，`MAX()`和`MIN()`函数并不限于数值类型。如果是字符类型，`MAX()`和`MIN()`会返回排序最后和排序最前的字符。

```sql
SELECT AVG(score) average FROM students WHERE gender = 'M';
```

如果聚合查询的`WHERE`条件没有匹配到任何行，`COUNT()`会返回0，而`SUM()`、`AVG()`、`MAX()`和`MIN()`会返回`NULL`：





```sql
SELECT COUNT(*) num FROM students GROUP BY class_id;
```

执行这个查询，`COUNT()`的结果不再是一个，而是3个，这是因为，`GROUP BY`子句指定了按`class_id`分组，因此，执行该`SELECT`语句时，会把`class_id`相同的列先分组，再分别计算，因此，得到了3行结果。



###### 多表查询

同时从`students`表和`classes`表的“乘积”，即查询数据

```SQL
SELECT * FROM students, classes;
```

为了区分同名列的两个表，可以使用别名的投影查询

```sql
-- 表 students 的id 就会取得别名 sid，classes 的 name ，就会取名 cname 。
SELECT
    students.id sid,
    students.name,
    students.gender,
    students.score,
    classes.id cid,
    classes.name cname
FROM students, classes;

-- 简化表名 students ，classes 
SELECT
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname
FROM students s, classes c;

-- 多表查询添加 WHERE 条件
SELECT
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname
FROM students s, classes c
WHERE s.gender = 'M' AND c.id = 1;
```

###### 连接查询

多表查询的一种，最常用的一种内连接——INNER JOIN

```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
INNER JOIN classes c
ON s.class_id = c.id;

```

1. 先确定主表，仍然使用`FROM <表1>`的语法；
2. 再确定需要连接的表，使用`INNER JOIN <表2>`的语法；
3. 然后确定连接条件，使用`ON <条件...>`，这里的条件是`s.class_id = c.id`，表示`students`表的`class_id`列与`classes`表的`id`列相同的行需要连接；
4. 可选：加上`WHERE`子句、`ORDER BY`等子句。



还有RIGHT OUTER JOIN，LEFT OUTER JOIN，以及FULL OUTER JOIN。它们的区别是：

* INNER JOIN只返回同时存在于两张表的行数据，由于`students`表的`class_id`包含1，2，3，`classes`表的`id`包含1，2，3，4，所以，INNER JOIN根据条件`s.class_id = c.id`返回的结果集仅包含1，2，3。

* RIGHT OUTER JOIN返回右表都存在的行。如果某一行仅在右表存在，那么结果集就会以`NULL`填充剩下的字段。

* LEFT OUTER JOIN则返回左表都存在的行。如果我们给students表增加一行，并添加class_id=5，由于classes表并不存在id=5的行，所以，LEFT OUTER JOIN的结果会增加一行，对应的`class_name`是`NULL`：

* FULL OUTER JOIN，它会把两张表的所有记录全部选择出来，并且，自动把对方不存在的列填充为NULL：

如何选择，请看下图

![连接查询](D:\work\note\SQL\img\连接查询.png)



#### 修改数据

##### INSERT

```sql
-- INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...);

INSERT INTO students (class_id, name, gender, score) VALUES (2, '大牛', 'M', 80);
-- 查询并观察结果:
SELECT * FROM students;

-- 添加多条数据
INSERT INTO students (class_id,name,gender,score) VALUES 
(2,'xixi','M',79),
(1,'hahha','F',81);
SELECT * FROM students
```



##### UPDATE

```sql
-- UPDATE <表名> SET 字段1=值1, 字段2=值2, ... WHERE ...;
UPDATE students SET name='大牛', score=66 WHERE id=1;
-- 查询并观察结果:
SELECT * FROM students WHERE id=1;

-- 更新多条
UPDATE students SET name='小牛', score=77 WHERE id>=5 AND id<=7;
-- 查询并观察结果:
SELECT * FROM students;

--在UPDATE语句中，更新字段时可以使用表达式。例如，把所有80分以下的同学的成绩加10分：
UPDATE students SET score = score + 10 WHERE score < 80 ;
SELECT * FROM students;

--如果WHERE条件没有匹配到任何记录，UPDATE语句不会报错，也不会有任何记录被更新。
UPDATE students SET score=100 WHERE id=999;
-- 查询并观察结果:
SELECT * FROM students;

--要特别小心的是，UPDATE语句可以没有WHERE条件
UPDATE students SET score=60;
--在执行UPDATE语句时要非常小心，最好先用SELECT语句来测试WHERE条件是否筛选出了期望的记录集，然后再用UPDATE更新。

```



##### DELETE

```SQL
--DELETE FROM <表名> WHERE ...;

-- 删除多条，WHERE控制条件
DELETE FROM students WHERE id>=5 AND id<=7;
-- 查询并观察结果:
SELECT * FROM students;

--WHERE条件没有匹配到任何记录，DELETE语句不会报错，也不会有任何记录被删除
DELETE FROM students WHERE id = 999 ;

-- 要特别小心的是，和UPDATE类似，不带WHERE条件的DELETE语句会删除整个表的数据：
DELETE FROM students;

```





#### 事务

把多条语句作为一个整体进行操作的功能，被称为数据库*事务*。数据库事务可以确保该事务范围内的所有操作都可以全部成功或者全部失败。如果事务失败，那么效果就和没有执行这些SQL一样，不会对数据库数据有任何改动。

比如，转账：涉及两个账户变动

id = 1 的账户 减少金额，

id = 2 的账户增加金额，

这是两句SQL,第一条成功，第二条失败，判断不成功，转账失败；就必须全部撤销。



特点：

* A：Atomic，原子性，将所有SQL作为原子工作单元执行，要么全部执行，要么全部不执行；
* C：Consistent，一致性，事务完成后，所有数据的状态都是一致的，即A账户只要减去了100，B账户则必定加上了100；
* I：Isolation，隔离性，如果有多个事务并发执行，每个事务作出的修改必须与其他事务隔离；
* D：Duration，持久性，即事务完成后，对数据库数据的修改被持久化存储。

对于单条SQL语句，数据库系统自动将其作为一个事务执行，这种事务被称为*隐式事务*。

要手动把多条SQL语句作为一个事务执行，使用`BEGIN`开启一个事务，使用`COMMIT`提交一个事务，这种事务被称为*显式事务*，例如，把上述的转账操作作为一个显式事务：

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

很显然多条SQL语句要想作为一个事务执行，就必须使用显式事务。

`COMMIT`是指提交事务，即试图把事务内的所有SQL所做的修改永久保存。如果`COMMIT`语句执行失败了，整个事务也会失败。

有些时候，我们希望主动让事务失败，这时，可以用`ROLLBACK`回滚事务，整个事务会失败：

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
ROLLBACK;
```

数据库事务是由数据库系统保证的，我们只需要根据业务逻辑使用它就可以。

##### 隔离级别

SQL标准定义了4种隔离级别，分别对应可能出现的数据不一致的情况：

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------- |
| Read Uncommitted | Yes                | Yes                               | Yes                  |
| Read Committed   | -                  | Yes                               | Yes                  |
| Repeatable Read  | -                  | -                                 | Yes                  |
| Serializable     | -                  | -                                 | -                    |