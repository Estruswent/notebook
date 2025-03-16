# Lecture3: introduction of SQL

## 1. SQL概述

### 1.1 历史背景  

略过就好，感觉不考，只是翻译复制粘贴一下。

SQL起源于IBM的Sequel语言，作为System R项目的一部分，后更名为结构化查询语言（SQL）。经过多次标准化，包括SQL-86、SQL-89、SQL-92、SQL:1999和SQL:2003。主流数据库系统通常支持SQL-92标准及后续版本的部分功能。  

### 1.2 SQL组成部分  

SQL包含多个功能模块：**数据定义语言（DDL）**用于定义、修改和删除关系模式；**数据操作语言（DML）**用于查询、插入、删除和修改数据；**完整性约束**确保数据一致性，如主键、外键和非空约束；**事务控制**管理事务的开始与结束；嵌入式SQL允许在编程语言中嵌入SQL语句；**授权机制**管理对关系和视图的访问权限。  

## 2. 数据定义语言（DDL）  

### 2.1 创建表

使用`CREATE TABLE`定义表结构（第一个实验就用过了）例如：

```sql
CREATE TABLE instructor (
    ID      CHAR(5),
    name    VARCHAR(20) NOT NULL,
    dept_name VARCHAR(20),
    salary  NUMERIC(8,2),
    PRIMARY KEY (ID),
    FOREIGN KEY (dept_name) REFERENCES department
);
```  

- 主键（PRIMARY KEY）：唯一标识记录并隐含非空约束；
- 外键（FOREIGN KEY）：确保引用时的完整性。  

### 2.2 数据类型  

常见数据类型包括：固定长度字符串（`CHAR(n)`）、可变长度字符串（`VARCHAR(n)`）、定点数（`NUMERIC(p,d)`）、整数（`INT`）和浮点数（`FLOAT`）。 

1.固定长度字符串 `CHAR(n)`  
示例：`ID  CHAR(5)`  
适用场景：存储长度固定的数据（**不足的自动补空格**）。如国家代码（如CN）、固定长度的ID（如A001）。  

2.可变长度字符串 `VARCHAR(n)` 
 
示例：`name  VARCHAR(50)`   
适用场景：存储**长度不固定**的文本.如用户姓名、地址、文章标题等。  
与CHAR的区别：VARCHAR按实际长度存储，节省空间；CHAR固定长度，查询更快但可能浪费空间。  

3.定点数 `NUMERIC(p,d)` 

示例：`salary  NUMERIC(5,2)`    
适用场景：需要**精确小数**的场景(其中总位数`p`，小数位数`d`)。如金融金额（如99.99）、科学测量值（如3.14）。  

4.整数 `INT`  

示例：`QQ_ID  INT`    
适用场景：存储无小数的数值（存小数的话，小数会被**直接截断小数点后的数字**）。如年龄、商品数量、用户积分等。  

5.浮点数 `FLOAT` 

示例：`Temperature FLOAT`    
适用场景：科学计算或需要大范围数值的场景（**但浮点数为近似值，不适用于精确计算**，比如某些金融问题就不行） 。如温度值（如36.6°C）、地理坐标等。  
 

### 2.3 修改表  

通过`ALTER TABLE`添加或删除列：  

```sql
-- 添加列
ALTER TABLE student ADD resume VARCHAR(256);

-- 删除列
ALTER TABLE student DROP resume;
```  

---

## 3. 基本查询结构  

### 3.1 SELECT子句  

`SELECT`用于**指定查询结果中的属性**。例如，查询所有教师姓名： 

```sql
SELECT name FROM instructor;
```  

使用`DISTINCT`可以去重：（也就是重复的属性只展示一次）  

```sql
SELECT DISTINCT dept_name FROM instructor;
```  

支持算术表达式和别名：  

```sql
SELECT ID, name, salary/12 AS monthly_salary FROM instructor;
```  

### 3.2 WHERE子句  

`WHERE`用于**过滤记录**。例如，查询计算机系工资超过70000的教师：  

```sql
SELECT name 
FROM instructor 
WHERE dept_name = 'Comp. Sci.' AND salary > 70000;
```  

使用`BETWEEN`选择范围查询：  

```sql
SELECT name 
FROM instructor 
WHERE salary BETWEEN 90000 AND 100000; -- 90000 <= salary <= 100000
```  

### 3.3 FROM子句与连接  

`FROM`指定查询涉及的关系。例如，通过笛卡尔积和条件过滤实现自然连接：  

```sql
SELECT name, course_id 
FROM instructor, teaches 
WHERE instructor.ID = teaches.ID;
```  

### 3.4 `LIKE`子句与字符串操作  

`LIKE`支持模式匹配。例如，查询包含“wen”的姓名：  

```sql
SELECT name 
FROM instructor 
WHERE name LIKE '%wen%';
```  

匹配长度为3的字符串：  

```sql
SELECT course_id 
FROM course 
WHERE course_id LIKE '___';
```  

### 3.5 `ORDER BY`子句与排序  

`ORDER BY`指定结果排序方式。例如，按姓名升序排列：  

```sql
SELECT name FROM instructor ORDER BY name;

```  

多级排序：  

```sql
SELECT name, dept_name 
FROM instructor 
ORDER BY dept_name ASC, name DESC; -- 其中ASC表示升序，DESC表示降序
```  

## 4. 高级操作

### 4.1 集合操作  

`UNION`（并集）、`INTERSECT`（交集）、`EXCEPT`（差集）用于合并查询结果。例如： 

```sql
-- 并集（去重）
(SELECT course_id FROM section WHERE year=2017)
UNION
(SELECT course_id FROM section WHERE year=2018);
```  

### 4.2 空值处理  

空值（NULL）表示未知或不存在的数据。使用`IS NULL`检测空值：

```sql
SELECT name FROM instructor WHERE salary IS NULL;
```  

注意：涉及空值的算术或逻辑运算结果为NULL或UNKNOWN，以下为详细解释。

> SQL使用三值逻辑处理包含 NULL 的表达式，结果可能是：
> 
> - TRUE（真）
> - FALSE（假）
> - UNKNOWN（未知）
> 
> 数据库里的`NULL`表示数据的**缺失**或**未知**，既不是空字符串也不是0。
>
> 因此，`NULL`实际上在逻辑运算里对应的就是`UNKNOWN`。

#### 4.2.1 布尔运算  

<div style="display: flex; justify-content: center; gap: 20px; margin: 20px 0;">
  <table border="1" style="text-align: center;">
    <caption><strong>AND 运算（A AND B）</strong></caption>
    <thead>
      <tr>
        <th>A</th>
        <th>B</th>
        <th>结果</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>TRUE</td>
        <td>UNKNOWN</td>
        <td><strong>UNKNOWN</strong></td>
      </tr>
      <tr>
        <td>FALSE</td>
        <td>UNKNOWN</td>
        <td>FALSE</td>
      </tr>
      <tr>
        <td>UNKNOWN</td>
        <td>UNKNOWN</td>
        <td><strong>UNKNOWN</strong></td>
      </tr>
    </tbody>
  </table>

  <table border="1" style="text-align: center;">
    <caption><strong>OR 运算（A OR B）</strong></caption>
    <thead>
      <tr>
        <th>A</th>
        <th>B</th>
        <th>结果</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>TRUE</td>
        <td>UNKNOWN</td>
        <td>TRUE</td>
      </tr>
      <tr>
        <td>FALSE</td>
        <td>UNKNOWN</td>
        <td><strong>UNKNOWN</strong></td>
      </tr>
      <tr>
        <td>UNKNOWN</td>
        <td>UNKNOWN</td>
        <td><strong>UNKNOWN</strong></td>
      </tr>
    </tbody>
  </table>
</div>

AND 运算： 

- 若任意一操作数为 `FALSE`，结果为 `FALSE`。  
- 若存在 `UNKNOWN` 且无 `FALSE`，结果为 `UNKNOWN`。

OR 运算： 

- 若任意一操作数为 `TRUE`，结果为 `TRUE`。  
- 若存在 `UNKNOWN` 且无 `TRUE`，结果为 `UNKNOWN`。

NOT 运算：  

- 对 `UNKNOWN` 取反，结果仍为 `UNKNOWN`。

#### 4.2.2 比较运算  

任何与NULL的比较结果均为UNKNOWN：
```sql
  NULL = 5       -- → UNKNOWN  
  NULL > 10      -- → UNKNOWN  
  NULL = NULL    -- → UNKNOWN  
  -- 注意：NULL不等于NULL！
```

只有使用 `IS NULL` 或 `IS NOT NULL` 才能明确判断：

```sql
  salary IS NULL      -- → TRUE （若工资未录入）
  salary IS NOT NULL  -- → TRUE （若工资已录入）
```

### 4.3 聚合函数与分组  

聚合函数（如`AVG`、`SUM`）对数据汇总。例如，按系计算平均工资：

```sql
SELECT dept_name, AVG(salary) AS avg_salary
FROM instructor 
GROUP BY dept_name;
```  

`HAVING`过滤分组后的结果：  

```sql
SELECT dept_name, AVG(salary) AS avg_salary
FROM instructor 
GROUP BY dept_name 
HAVING AVG(salary) > 42000;
```  

## 5. 嵌套子查询

### 5.1 子查询位置  

子查询可嵌入`FROM`或`WHERE`子句。例如，在`FROM`中使用子查询：  

```sql
SELECT dept_name, avg_salary 
FROM (
    SELECT dept_name, AVG(salary) AS avg_salary 
    FROM instructor 
    GROUP BY dept_name
) 
WHERE avg_salary > 42000;
```  

### 5.2 存在性测试  

`EXISTS`检查子查询是否返回结果。例如，查询2024年秋季和2025年春季均开课的课程：  

```sql
SELECT course_id 
FROM section S 
WHERE semester='Fall' AND year=2024 
  AND EXISTS (
    SELECT * 
    FROM section T 
    WHERE semester='Spring' AND year=2025 
      AND S.course_id = T.course_id
);
```  

## 6. 数据库的修改  
### 6.1 插入数据  
插入完整元组或查询结果：  
```sql
-- 插入单条记录
INSERT INTO course 
VALUES ('CS-437', 'Database Systems', 'Comp. Sci.', 4);

-- 插入查询结果
INSERT INTO instructor 
SELECT ID, name, dept_name, 18000 
FROM student 
WHERE dept_name = 'Music' AND total_cred > 144;
```  

### 6.2 删除数据  
按条件删除记录：  
```sql
DELETE FROM instructor 
WHERE salary < (SELECT AVG(salary) FROM instructor);
```  

### 6.3 更新数据  

使用`CASE`实现条件更新。详细点来说使用`CASE`在执行 UPDATE 语句时，**根据不同的条件设置**字段的新值。（类比那个switch...case语句） 

```sql
UPDATE instructor 
SET salary = CASE 
    WHEN salary < 100000 THEN salary * 1.05 
    WHEN salary BETWEEN 100000 AND 150000 THEN salary * 1.04 
    ELSE salary * 1.03 
END;
``` 