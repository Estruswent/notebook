# Lecture 4：Intermediate SQL

## 1. 连接表达式（Join Expressions）  

### 1.1 自然连接（Natural Join）  

定义：基于两个关系的**共有属性**自动匹配，并保留一份共有属性。  

```sql  
-- 传统写法  
SELECT name, course_id  
FROM student, takes  
WHERE student.ID = takes.ID;  

-- 自然连接写法  
SELECT name, course_id  
FROM student NATURAL JOIN takes;  
```  

!!! warning 
    若存在无关同名属性，可能导致错误匹配。例如：  

    ```sql  
     -- 错误写法（可能因部门名称同名错误匹配）  
     SELECT name, title  
     FROM student NATURAL JOIN takes NATURAL JOIN course;  

     -- 正确写法（显式指定连接属性）  
     SELECT name, title  
     FROM student NATURAL JOIN takes, course  
     WHERE takes.course_id = course.course_id;  -- 保证只要课程名称相同
    ```  

---

### 1.2 内连接（Inner Join）  

定义：通过`ON`子句指定连接条件，保留满足条件的元组。  
  
```sql  
SELECT *  
FROM student JOIN takes ON student.ID = takes.ID;  
```

等价于：

```sql  
SELECT *  
FROM student, takes  
WHERE student.ID = takes.ID;  
```  

### 1.3 外连接（Outer Join）  

- 定义：外连接（Outer Join）是用于合并多个表数据的操作，与内连接（Inner Join）不同，它不仅返回两个表中满足条件的匹配行，还会包含**至少一个表中不满足匹配条件的行**，这些不匹配的列会用 `NULL` 值填充。  
- 类型：  
    - 左外连接（保留左表所有元组）  
    - 右外连接（保留右表所有元组）  
    - 全外连接（保留左右表所有元组）  
- 特点：
    - 包含不匹配的行：外连接会保留一个表（或两个表）中所有行（尽管可能这些行在另一个表中没有匹配的记录！）。
    - 用 `NULL` 填充缺失值：当某一行在另一张表中没有匹配时，对应列的值会显示为 `NULL`。
    - 与内连接的区别：内连接仅返回两个表中**完全匹配的行**，而外连接会保留更多数据。

#### 左外连接（Left Outer Join / LEFT JOIN）

定义：返回左表（写在 `LEFT JOIN` 前的表）**所有行**，以及右表中满足连接条件的匹配行。如果右表中没有匹配行，则结果中右表的列显示为 `NULL`。语法如下：

  ```sql
  SELECT 列名
  FROM 左表 LEFT JOIN 右表 -- 左连
  ON 连接条件;
  ```

!!! example
    **表1（员工表 `emp`）**：

    |   ID   | name   | dept_ID |
    |--------|--------|--------|
    | 1      | 张三   | 101    |
    | 2      | 李四   | 102    |
    | 3      | 王五   | NULL   |

    **表2（部门表 `dept`）**：
    
    | dept_ID | dept_name|
    |--------|----------|
    | 101    | 技术部   |
    | 103    | 设计部   |

左外连接查询：

```sql
    SELECT emp.name, dept.dept_name
    FROM emp LEFT JOIN dept
    ON emp.dept_ID = dept.dept_ID;
```

左连接实际上就是保留`LEFT JOIN`左侧表的所有行，所以结果如下：（其他列略）

| 姓名   | 部门名称 |
|--------|----------|
| 张三   | 技术部   |
| 李四   | NULL     |  
| 王五   | NULL     |  

#### 右外连接（Right Outer Join / RIGHT JOIN）
定义：返回右表（写在 `RIGHT JOIN` 后的表）**所有行**，以及左表中满足连接条件的匹配行。如果左表中没有匹配行，则结果中左表的列显示为 `NULL`。语法如下：

```sql
  SELECT 列名
  FROM 左表 RIGHT JOIN 右表 -- 右连
  ON 连接条件;
```

右外连接查询：（使用上述 `emp` 和 `dept` 表）

```sql
  SELECT emp.name, dept.dept_name
  FROM emp RIGHT JOIN dept
  ON emp.dept_ID = dept.dept_ID;
```

结果如下：（其他列略）

| name   | dept_name |
|--------|----------|
| 张三   | 技术部   |
| NULL   | 设计部   |

右外连接只保留右表（部门表）的全部行，因此 `设计部`会被保留，而其它不存在，所以不会出现在结果中。

#### 全外连接（Full Outer Join / FULL JOIN）

定义：返回左表和右表**所有行**，无论是否匹配。如果某一行在另一表中没有匹配，则对应列显示为 `NULL`。语法如下：

```sql
SELECT 列名
FROM 左表 FULL [OUTER] JOIN 右表
ON 连接条件;
```

全外连接查询：

```sql
SELECT emp.name, dept.dept_name
FROM emp FULL OUTER JOIN dept
ON emp.dept_ID = dept.dept_ID;
```

左右都要保留，因此结果如下：

| name   | dept_name|
|--------|----------|
| 张三   | 技术部   |
| 李四   | NULL     |
| 王五   | NULL     |
| NULL   | 设计部   |

注意：并非所有的数据库系统都支持 `FULL OUTER JOIN` 语法。例如，在 MySQL 中需要通过组合 `LEFT JOIN` 和 `RIGHT JOIN` 来实现类似的效果。以下是 MySQL 实现全外连接的一个示例：

```sql
SELECT emp.name, dept.dept_name
FROM emp
LEFT JOIN dept ON emp.dept_ID = dept.dept_ID

UNION ALL

SELECT emp.name, dept.dept_name
FROM emp
RIGHT JOIN dept ON emp.dept_ID = dept.dept_ID
WHERE emp.ID IS NULL;
```

用 `LEFT JOIN` 获取左表的所有行，然后用 `RIGHT JOIN` 获取右表的所有行，并通过 `UNION ALL` 将两部分结果合并。最后的 `WHERE` 子句确保只添加那些在左表中没有匹配的右表行。

## 2. 视图（Views）  

### 2.1 视图定义  
作用：隐藏数据复杂性或敏感信息，提供虚拟表。（个人理解就是取一部分出来做成一个单独的表来给人看）
语法：（以`instructor(ID, name, dept_name)`为例）

```sql  
CREATE VIEW faculty AS  -- 创建faculty视图  
SELECT ID, name, dept_name  
FROM instructor;  
```  

### 2.2 视图查询  

```sql  
-- 查询生物学系的教师姓名  
SELECT name  
FROM faculty  
WHERE dept_name = 'Biology';
```  

### 2.3 视图更新限制 

!!! warning
    视图的更新限制：

      - 视图的`FROM`子句仅包含一个基表。  
      - `SELECT`子句仅包含属性名，无表达式或聚合函数。  
      - 无`GROUP BY`或`HAVING`子句。 

```sql  
  CREATE VIEW instructor_info AS  
  SELECT ID, name, building  
  FROM instructor, department  
  WHERE instructor.dept_name = department.dept_name;  
  
  INSERT INTO instructor_info VALUES ('67656', 'MDM', 'ZJG');  
  -- 问题：无法确定部门（若存在多个同名 building）  
```  

## 3. 索引（Index）  

**加速查询**，避免全表扫描。比如说假设我们需要频繁执行某个SQL查询，而该查询涉及的表中有大量数据，那么查询效率就会受到影响。

语法如下：

```sql  
CREATE INDEX studentID_index ON student(ID);  
```  

这样之后索引会生成一个类似以下的“目录”：

```plaintext
ID → 物理存储位置
1 → ID为1的地址
2 → ID为2的地址
3 → ID为3的地址
```

当执行`SELECT * FROM student WHERE ID = X;`（X为某个值）时，数据库首先在索引中快速查找 ID=3 的位置。然后直接跳转到该位置读取数据，避免全表扫描。

## 4. 事务（Transactions）  

### 4.1 事务特性（ACID）

这个不多赘述。

- **原子性**（Atomicity）：事务完全执行或完全不执行。  
- **隔离性**（Isolation）：并发事务互不干扰。  
- **持久性**（Durability）：提交后修改永久保存。  
- **一致性**（Consistency）：事务前后数据库状态一致。  

### 4.2 事务控制  

正常来说的事务控制流程如下：

```sql  
BEGIN TRANSACTION;  
-- 这里进行执行操作（如插入、更新）  
COMMIT; -- 提交  
```

但是，有的时候因为插入错误了，这时我们就要回滚：

```sql
BEGIN TRANSACTION;  
-- 这里进行执行操作（如插入、更新） 
-- 然后某一步出现错误了
ROLLBACK; -- 回滚
```  

不过，回滚只能撤销`COMMIT`之前的操作，并不会影响到已经提交的部分。

## 5. 完整性约束  

### 5.1 单关系约束  

在某个关系（表）中，对它的某一属性设置约束。

| 约束类型   |                           说明                  | 示例                          |  
|------------|-------------------------------------------------|-------------------------------|  
| `NOT NULL` | **禁止空值**                                         | `name VARCHAR(20) NOT NULL`   |  
| `UNIQUE`   | 属性**组合唯一**（可空）                             | `UNIQUE (dept_name, building)`|  
| `CHECK`    | 自定义条件，不满足条件的的写入等操作会被拒绝            | `CHECK (salary > 10000)`      |  

### 5.2 外键与级联操作

- `CASCADE`：主表删除/更新时，从表同步操作。  
- `SET NULL`：主表操作后，从表外键设为`NULL`。  

比如以以下这个创建 `course` 表为例：

```sql  
CREATE TABLE course (  
    course_id VARCHAR(8),  
    dept_name VARCHAR(20),  
    FOREIGN KEY (dept_name) REFERENCES department  
    ON DELETE CASCADE  -- 主表删除时，从表同步删除  
    ON UPDATE SET NULL -- 主表更新时，从表外键设为NULL  
);  
```

## 6. 更多的 SQL 数据类型与模式  

### 6.1 大对象类型  

| 类型  | 说明                  | 示例                  |  
|-------|-----------------------|-----------------------|  
| `BLOB`| 二进制大对象（如图片）| `image BLOB(10MB)`    |  
| `CLOB`| 字符大对象（如长文本）| `resume CLOB(1KB)`    |  

### 6.2 用户定义类型与域

```sql  
-- 自定义类型  
CREATE TYPE Dollars AS NUMERIC(12,2);  

-- 自定义域（含约束）  
CREATE DOMAIN degree_level VARCHAR(10)  
CHECK (VALUE IN ('Bachelors', 'Masters', 'Doctorate'));  
```

!!! note "域（Domain）"
    作用：创建一个（在上述代码中名为`degree_level`，其底层类型为`VARCHAR(10)`）自定义数据类型，可以在其中定义约束条件。

    本质：

    - 是带有约束的数据类型，用于限制字段的取值范围。
    - 可以复用，简化表定义并确保数据一致性。

## 7. 授权（Authorization）  

### 7.1 权限类型

| 权限      | 说明                |  
|-----------|---------------------|  
| `SELECT`  | 读取数据            |  
| `INSERT`  | 插入数据            |  
| `UPDATE`  | 更新数据            |  
| `DELETE`  | 删除数据            |  
| `REFERENCES` | 创建外键引用     |
|`ALL PRIVILEGES`| 授予所有权限（包括 `GRANT` 权限）|

### 7.2 角色管理

用`CREATE ROLE`创建角色并用`GRANT`授权：

```sql 
CREATE ROLE localhost;  
GRANT SELECT ON my_table TO localhost;  -- 授予localhost角色对my_table的SELECT权限  
GRANT localhost TO e5trusw3nt; -- 授予e5trusw3nt用户localhost的权限  
```   

`WITH GRANT OPTION`参数允许传递权限：

```sql
-- 授予所有权限  
GRANT ALL PRIVILEGES ON my_table TO admin WITH GRANT OPTION;
-- WITH GRANT OPTION参数可将权限传递给其他用户，也就是说，admin可以将权限授予其他用户。
```

使用`REVOKE`回收权限：

```sql
REVOKE SELECT ON my_table FROM e5trusw3nt CASCADE; -- CASCADE参数可将所有依赖于该角色的权限一并回收  
```

## 8. 触发器（Triggers）  

### 8.1 触发器详解：自动转换空成绩为 NULL  

#### 1. 触发器定义语法解析

```sql  
CREATE TRIGGER setnull_trigger                           -- 1. 触发器名称  
[BEFORE | AFTER] <触发事件> ON <表名>                      -- 2. 触发时机与事件  
[REFERENCING OLD ROW AS <旧行别名> NEW ROW AS <新行别名>]   -- 3. 引用新或旧行数据  
[FOR EACH ROW | FOR EACH STATEMENT]                      -- 4. 行或语句级触发器  
[WHEN (<条件>)]                                           -- 5. 触发条件  
BEGIN  
    -- 执行（SQL语句）
END;  
```  

新旧行引用语句：

- `OLD ROW`：删除或修改前的旧数据（DELETE/UPDATE时有效）。
- `NEW ROW`：插入或修改后的新数据（INSERT/UPDATE时有效）。

触发粒度：

- `FOR EACH ROW`：每影响一行触发一次（行级触发器）。
- `FOR EACH STATEMENT`：每条SQL语句触发一次（默认）。

#### 2. 触发器示例：选课时自动检查学分

场景：学生选课（`takes`表插入记录）时，若当前总学分（`tot_cred`）过多，则阻止插入并提示。

```sql
CREATE TRIGGER check_credits
BEFORE INSERT ON takes
REFERENCING NEW ROW AS nrow      -- 引用新插入的行
FOR EACH ROW
WHEN (  
    (SELECT tot_cred FROM student WHERE ID = nrow.ID) > 30  -- 学生总学分超过30  
)  
BEGIN  
    SIGNAL SQLSTATE '45000'                  -- 抛出错误（阻止插入）  
    SET MESSAGE_TEXT = '你小子，少选点课';  
END;
```

执行逻辑：

- 触发时机：在向 `takes` 表插入新记录前（`BEFORE INSERT`）。
- 条件检查：通过子查询检查该学生 `tot_cred` 是否超过30。
- 阻止操作：若条件为真，抛出错误码`45000`并终止插入。