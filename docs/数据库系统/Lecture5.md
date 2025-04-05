# Lecture 5：Advanced SQL

## 1. 从编程语言访问SQL  

| 方式          | 特点                                                                 | 适用场景                  |  
|---------------|----------------------------------------------------------------------|---------------------------|  
| **动态 SQL**  | 运行时构造 SQL 语句（如 JDBC、ODBC）                                 | 灵活查询，用户输入处理    |  
| **嵌入式 SQL**| 编译时预编译 SQL 语句（如 SQLJ）                                     | 高性能、静态查询          |  

## 2. JDBC（Java 数据库连接）  

### 2.1 JDBC 核心流程

```java  
public static void JDBCexample() {  
    try (  
        // 1. 建立连接  
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db", "user", "pass");  
        // 2. 创建 Statement 对象  
        Statement stmt = conn.createStatement();  
    ) {  
        // 3. 执行查询  
        ResultSet rs = stmt.executeQuery("SELECT * FROM instructor");  
        // 4. 处理结果  
        while (rs.next()) {  
            System.out.println(rs.getString("name"));  
        }  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
}  
```  

### 2.2 预编译语句（PreparedStatement）

`?`是占位符，用于表示将要插入的数据值。

```java  
// 预编译插入语句  
PreparedStatement pStmt = conn.prepareStatement("INSERT INTO instructor VALUES (?, ?, ?, ?)");  
pStmt.setString(1, "10101");  // 第一个参数  
pStmt.setString(2, "Alice");  // 第二个参数  
pStmt.setString(3, "Physics");// 第三个参数  
pStmt.setInt(4, 90000);       // 第四个参数  
pStmt.executeUpdate();  
```  

!!! warning "SQL 注入风险"  
    要避免通过字符串拼接构造 SQL！  
    ```java  
    // 危险写法，容易引起注入
    String query = "SELECT * FROM users WHERE name = '" + userInput + "'";  
    ```  
    最好用 `PreparedStatement` 的 `setXxx()` 方法传递参数。 

### 2.3 讲解说明

- `Connection`：数据库连接（需手动关闭或使用 `try-with-resources`）。  
- `Statement`：执行静态 SQL 语句（易引发 SQL 注入）。  
- `PreparedStatement`：预编译 SQL（防注入，可重用）。  
- `ResultSet`：存储查询结果（通过 `next()` 遍历）。  

## 3. ODBC（开放数据库互连）  

### 3.1 ODBC 特点

- 跨语言支持：C/C++、C#、Python 等。  
- 统一接口：通过驱动程序连接不同数据库（如 MySQL、Oracle）。  

### 3.2 基本流程

1. 配置数据源（DSN）。  
2. 使用 API 函数，例如：  
    - `SQLConnect()`：建立连接。
    - `SQLExecDirect()`：执行查询。
    - `SQLFetch()`：获取结果。

## 4. 嵌入式 SQL  

### 4.1 嵌入式 SQL 语法

使用 `EXEC SQL` 标记 SQL 语句。    

```c
// EXEC SQL BEGIN DECLARE SECTION和EXEC SQL END DECLARE SECTION之间的所有变量都被视为宿主变量
// 宿主变量也就是主机语言（这里是C语言）中定义的变量
EXEC SQL BEGIN DECLARE SECTION;  
int credit_amount;  
EXEC SQL END DECLARE SECTION;  

// 连接数据库my_db，用用户名e5trusw3nt和密码my_password连接
EXEC SQL CONNECT TO my_db USER "e5trusw3nt" USING "my_password";  

// 为了执行，这里声明了一个名为c的游标（Cursor）。
// 游标是一个指向查询结果集的指针，可以用来遍历查询结果中的每一行数据。
EXEC SQL DECLARE c CURSOR FOR   
    SELECT ID, name FROM student WHERE tot_cred > :credit_amount;
// 宿主变量（Host Variables）通过 `:` 前缀引用。
// 这里的 credit_amount 就是宿主变量。

// 打开游标c，也就是说执行前面声明的SQL查询
EXEC SQL OPEN c;
// 循环遍历查询结果集，并打印出ID和name
while (SQLSTATE != '02000') {
    // 从游标c中获取一行数据，并将其存储到宿主变量sid和sname中
    EXEC SQL FETCH c INTO :sid, :sname;
    printf("%s %d\n", sid, sname);
}
EXEC SQL CLOSE c;// 关闭游标c
```  

## 5. 存储函数与过程

### 5.1 SQL 函数

大致模板如下：

```sql
CREATE FUNCTION function_name(param1 datatype1, param2 datatype2, ...)
RETURNS return_datatype -- 声明函数返回值类型
BEGIN
    -- 声明局部变量
    DECLARE local_var1 datatype1;
    DECLARE local_var2 datatype2;
    ...

    -- 执行SQL查询或其他操作

    -- 返回结果
    RETURN local_var1;
END;
```

以输出某单位的教师数量为例。

```sql  
-- 定义函数：统计部门教师数量  
CREATE FUNCTION dept_count(dept_name VARCHAR(20))  
RETURNS INTEGER  
BEGIN  
    DECLARE d_count INTEGER;  
    SELECT COUNT(*) INTO d_count  
    FROM instructor
    -- 指定筛选条件，即instructor表中的dept_name列必须等于传入的dept_name参数。
    WHERE instructor.dept_name = dept_count.dept_name;
    RETURN d_count;  
END;  

-- 使用函数  
SELECT dept_name, budget  
FROM department  
WHERE dept_count(dept_name) > 5;  
```  

### 5.2 存储过程
以输出部门平均工资为例：

```sql  
-- 定义过程：输出部门平均工资
-- 定义一个名为dept_avg_salary的存储过程。该过程接受两个参数：
-- IN dept_name VARCHAR(20)：输入参数，表示部门名称，类型为VARCHAR(20)。
-- OUT avg_sal NUMERIC(8,2)：输出参数，用于存储计算得到的平均工资，类型为NUMERIC(8,2)。
CREATE PROCEDURE dept_avg_salary(IN dept_name VARCHAR(20), OUT avg_sal NUMERIC(8,2))  
BEGIN  
    SELECT AVG(salary) INTO avg_sal  
    FROM instructor  
    WHERE instructor.dept_name = dept_avg_salary.dept_name;  
END;  

-- 调用过程
CALL dept_avg_salary('Comp. Sci.', @avg_sal);  
SELECT @avg_sal;
```

### 5.3 流程控制

布什戈门，这不就是...所以这个就不用多说了。

**条件分支：**

```sql  
  IF salary > 100000 THEN  
    SET bonus = salary * 0.1;  
  ELSEIF salary > 50000 THEN  
    SET bonus = salary * 0.05;  
  ELSE  
    SET bonus = 0;  
  END IF;
```  

**循环：**

```sql  
  WHILE i < 10 DO  
    SET total = total + i;  
    SET i = i + 1;  
  END WHILE;  
```

## 6. 元数据操作  

### 6.1 获取结果集元数据

```java 
ResultSetMetaData rsmd = rs.getMetaData();
// 列是从1开始计的
// rsmd.getColumnCount()：获取列数。
for (int i = 1; i<=rsmd.getColumnCount(); i ++) {  
    // 分别用getColumnName()和getColumnTypeName()获取列名和类型
    System.out.println("列名: " + rsmd.getColumnName(i));  
    System.out.println("类型: " + rsmd.getColumnTypeName(i));  
}  
```

### 6.2 获取数据库元数据

```java
//dbmd.getTables(catalogName, schemaPattern, tableNamePattern, types)返回一个ResultSet对象，其中包含匹配指定模式的所有表的信息。
//catalogName：目录名称（通常是数据库名称），如果为null，则表示不指定目录。
//schemaPattern：模式名称（通常是用户名或架构名称），如果为null，则表示不指定模式。
//tableNamePattern：表名模式，支持通配符（例如%表示任意字符序列）。这里使用"%"表示获取所有表。
//types：这里用new String[]{"TABLE"}表示只检索表。
DatabaseMetaData dbmd = conn.getMetaData();  
ResultSet rs = dbmd.getTables(null, null, "%", new String[]{"TABLE"});  
while (rs.next()) {  
    System.out.println("表名: " + rs.getString("TABLE_NAME"));  
}
```

!!! example "简单的示例"
    以下是一个简单的java获取数据库元数据的示例。
    ```java
    import java.sql.Connection;
    import java.sql.DatabaseMetaData;
    import java.sql.DriverManager;
    import java.sql.ResultSet;

    public class DatabaseMetaDataExample {
        public static void main(String[] args) {
            // 声明Connection对象，用于管理与数据库的连接
            Connection conn = null;
            // 声明ResultSet对象，用于存储查询结果
            ResultSet rs = null;

            try {
                // 加载MySQL JDBC驱动程序
                Class.forName("com.mysql.cj.jdbc.Driver");

                // 建立与数据库的连接
                // URL格式为 jdbc:mysql://主机:端口/数据库名
                // 用户名和密码分别为 "user" 和 "password"
                conn = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/univdb", 
                    "user", 
                    "password"
                );

                // 获取DatabaseMetaData对象，该对象包含关于数据库的各种元数据信息
                DatabaseMetaData dbmd = conn.getMetaData();

                // 获取数据库中所有表的信息
                // 参数说明：
                // - catalogName: 目录名称（通常是数据库名称），如果为null，则表示不指定目录
                // - schemaPattern: 模式名称（通常是用户名或架构名称），如果为null，则表示不指定模式
                // - tableNamePattern: 表名模式，支持通配符（例如 "%" 表示任意字符序列）
                // - types: 一个字符串数组，指定要检索的对象类型（例如 new String[]{"TABLE"} 表示只检索表）
                rs = dbmd.getTables(null, null, "%", new String[]{"TABLE"});

                // 遍历结果集中的每一行，并打印出表名
                while (rs.next()) {
                    // 获取当前行中 TABLE_NAME 列的值，并打印出来
                    System.out.println("表名: " + rs.getString("TABLE_NAME"));
                }

            } catch (Exception e) {
                // 捕获并打印异常堆栈信息
                e.printStackTrace();
            } finally {
                // 确保在完成操作后关闭所有打开的资源，以避免资源泄漏
                try {
                    // 关闭ResultSet对象
                    if (rs != null) rs.close();
                    // 关闭Connection对象
                    if (conn != null) conn.close();
                } catch (Exception e) {
                    // 捕获并打印异常堆栈信息
                    e.printStackTrace();
                }
            }
        }
    }
    ```

## 7. 总结：高级特性对比

| 特性               |     JDBC/ODBC            | 嵌入式 SQL                |  
|--------------------|---------------------------|---------------------------|  
| **灵活性**         | 高（动态构造查询）        | 低（需预编译）            |  
| **性能**           | 一般                      | 高（编译优化）            |  
| **安全性**         | 依赖预处理（防注入）      | 高（静态检查）            |  
| **适用场景**       | 动态查询、Web 应用        | 高性能事务、复杂逻辑      |