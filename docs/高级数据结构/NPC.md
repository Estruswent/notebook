# 常见 NP-Complete（NPC）问题速查表

## 一、经典起源问题

| 问题        | 说明                           |
| --------- | ---------------------------- |
| **SAT**   | 第一个被证明为 NP-Complete（Cook 定理） |
| **3-SAT** | 每个子句 3 个文字，最常用归约源            |


## 二、图论类

### 路径 / 环

| 问题                                | NPC |
| --------------------------------- | --- |
| **Hamiltonian Path**              | ✔   |
| **Hamiltonian Cycle**             | ✔   |
| **Longest Path**                  | ✔   |
| **Traveling Salesman (Decision)** | ✔   |


### 覆盖 / 独立集 / 团

| 问题                  | NPC |
| ------------------- | --- |
| **Vertex Cover**    | ✔   |
| **Independent Set** | ✔   |
| **Clique**          | ✔   |

> 三者 **互相可多项式归约**

### 着色

| 问题                     | NPC      |
| ---------------------- | -------- |
| **k-Coloring (k ≥ 3)** | ✔        |
| 2-Coloring             | ❌（P，二分图） |

### 子图与割

| 问题                                | NPC |
| --------------------------------- | --- |
| **Steiner Tree**                  | ✔   |
| **Minimum Cut (with constraint)** | ✔   |


## 三、集合 / 数值类

| 问题              | NPC |
| --------------- | --- |
| **Subset Sum**  | ✔   |
| **Partition**   | ✔   |
| **Set Cover**   | ✔   |
| **Exact Cover** | ✔   |

## 四、背包与调度

### 背包

| 问题                    | NPC  |
| --------------------- | ---- |
| **0/1 Knapsack（判定版）** | ✔    |
| Fractional Knapsack   | ❌（P） |

??? note "什么叫判定版？"

    举个例子说明吧

    | 版本                | 是否 NPC        |
    | ----------------- | ------------- |
    | 最小顶点覆盖是多少？        | （优化问题）不是       |
    | 是否存在大小不大于 k 的顶点覆盖？ | 是    |


### 调度

| 问题                           | NPC |
| ---------------------------- | --- |
| Job Scheduling (≥2 machines) | ✔   |
| Single Machine Scheduling    | ❌   |


## 五、数据结构类

| 问题                                          | NPC |
| ------------------------------------------- | --- |
| **Minimum Degree Spanning Tree (Δ ≤ 2 判定)** | ✔   |
| **Treewidth (decision)**                    | ✔   |
| **Feedback Vertex Set**                     | ✔   |


## 六、其它

| 问题                              | NPC |
| ------------------------------- | --- |
| **Exact Cover by 3-Sets (X3C)** | ✔   |
| **Hitting Set**                 | ✔   |

# 七、常见「不是 NPC」但容易误判的

| 问题                  | 复杂度 |
| --------------------- | --- |
| Shortest Path         | P   |
| Minimum Spanning Tree | P   |
| Max Flow              | P   |
| Bipartite Matching    | P   |
| 2-SAT                 | P   |
| Fractional Knapsack   | P   |


??? note "常见等价关系"

    Vertex Cover ⇄ Independent Set
    
    Clique ⇄ Independent Set（补图）
    
    Hamiltonian Path ⇄ Hamiltonian Cycle
    
    Subset Sum ⇄ Partition
