# Lecture 2: relations models and algebra

## 一、关系模型基础

### 关系结构
- **关系（Relation）**：由行（元组）和列（属性）组成的二维表。例如，`Instructor`表包含属性`ID`, `name`, `dept_name`, `salary`。
- **属性（Attribute）**：列的命名，每个属性有对应的域（Domain），规定允许的取值范围。属性值需为原子值（不可再分），允许`null`表示未知值。
- **元组（Tuple）**：表中的一行数据。

### 模式与实例
- **关系模式（Schema）**：关系的逻辑结构，定义属性集合。例如，`instructor(ID, name, dept_name, salary)`。
- **关系实例（Instance）**：某一时刻关系中具体的数据集合，即表的实际内容。

### 关系无序性
关系的元组没有固定顺序，存储和查询时顺序不影响结果。

## 二、键（Keys）

### 超键与候选键

- **超键（Superkey）**：能唯一标识元组的属性集合。例如，`{ID}`和`{ID, name}`均为超键。
- **候选键（Candidate Key）**：极小超键（无法进一步删减属性）。例如，`{ID}`是候选键。

### 主键与外键

- **主键（Primary Key）**：从候选键中选定的唯一标识符。
- **外键（Foreign Key）**：一个关系中的属性集合，需引用另一关系的主键。例如，`teaches`表中的`ID`引用`instructor`表的`ID`。

## 三、关系代数

关系代数是过程式查询语言，包含6种基本操作：

### 基本操作符

1. **选择（Select）**  
   $\sigma_p(r)$：筛选满足谓词$p$的元组。  
   示例：查询物理系教师  
   $$
   \sigma_{\text{dept_name="Physics"}}(\text{instructor})
   $$

2. **投影（Project）**  
   $\Pi_{A_1, A_2, \dots, A_k}(r)$：保留指定属性，去重后输出。  
   示例：获取教师ID和姓名  
   $$
   \Pi_{\text{ID, name}}(\text{instructor})
   $$

3. **并（Union）**  
   $r \cup s$：合并两个同构关系的元组。  
   示例：查询2017秋季或2018春季开设的课程
   $$
   \Pi_{course\_id}(\sigma_{semester=Fall \land year=2017}(section)) \cup \Pi_{course\_id}(\sigma_{semester=Spring \land year=2018}(section))
   $$


4. **差（Set Difference）**  
   $r - s$：保留在$r$中但不在$s$中的元组。  
   示例：查询2017秋季开设但2018春季未开设的课程  
$$
\Pi_{course\_id}(\sigma_{semester=Fall \land year=2017}(section)) - \Pi_{course\_id}(\sigma_{semester=Spring \land year=2018}(section))
$$


5. **笛卡尔积（Cartesian Product）**  
   $r \times s$：组合两关系的所有元组对。  
   示例：$instructor × teaches $生成所有可能的教师-课程组合。

6. **重命名（Rename）**  
   $\rho_{x(A_1, A_2, \dots, A_n)}(E)$：将表达式$E$的结果重命名为$x$，并可改属性名。

### 其它操作符

1. **连接（Join）**  
   $\Join_{\theta}$：结合选择与笛卡尔积，保留满足条件$\theta$的元组。  
   示例：查询教师及其对应的授课信息  
   $$
   \text{instructor} \Join_{\text{instructor.ID=teaches.ID}} \text{teaches}
   $$
   注：自然连接是自动匹配同名属性，省略$\theta$条件。

2. **集合交（Intersection）**  
   $r \cap s$：返回同时在$r$和$s$中的元组。  
   示例：查询同时在2017秋季和2018春季开设的课程  
   $$
   \Pi_{\text{course_id}}(\sigma_{\text{semester=Fall} \land \text{year=2017}}(\text{section})) \cap \Pi_{\text{course_id}}(\sigma_{\text{semester=Spring} \land \text{year=2018}}(\text{section}))
   $$


## 四、等价查询

同一查询可通过不同关系代数表达式实现。例如，查询物理系高薪（>90000）教师：

**表达式1**：先选择后投影  
$$
\Pi_{\text{name}}(\sigma_{\text{dept_name=Physics} \land \text{salary>90000}}(\text{instructor}))
$$

**表达式2**：分步赋值  
$$
\text{PhysicsHigh} \leftarrow \sigma_{\text{dept_name=Physics} \land \text{salary>90000}}(\text{instructor})
$$  