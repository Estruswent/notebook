# 并查集（Disjoint Set ADT）  

## 一、基本概念

### 1. 等价关系与等价类

- **等价关系**：若一个关系满足以下三条性质：  
    - **自反性**(reflexive)：元素与自己相关（如 `a ~ a`）。  
    - **对称性**(symmetric)：若 `a ~ b`，则 `b ~ a`。  
    - **传递性**(transitive)：若 `a ~ b` 且 `b ~ c`，则 `a ~ c`。   
- **等价类**：所有互相等价的元素构成的集合。例如，若 `1 ~ 3` 且 `3 ~ 5`，则 `{1, 3, 5}` 是一个等价类。  

### 2. 动态等价问题  
**问题描述**：给定一组元素和若干等价关系，动态合并等价类，并快速判断两个元素是否属于同一类。（这么一说肯定很抽象，但是不急，请慢慢看下方）

**核心操作**：

- `Union(a, b)`：合并 `a` 和 `b` 所在的集合。  
- `Find(a)`：查找 `a` 所在集合的根节点（代表元素）。  

## 二、初始化：数组表示  
### 1. 数据结构  

**数组 `parent[]`**：存储每个元素的父节点。  

**初始化**：每个元素独立为一个集合，`parent[i] = -1`（负数表示根节点，绝对值表示集合大小）。  

```C  
#define MAX_SIZE 100007

int parent[MAX_SIZE];  
void Initialize(int N) {  
    for (int i = 0; i < N; i ++) {  
        parent[i] = -1; // 初始化为 -1 表示什么都还未合并
    }  
}  
```  

## 三、合并优化：按大小合并（Union-by-Size）  
### 1. 规则  

- **合并策略**：将较小的树作为子树挂到较大的树下。  
- **优势**：避免树过高，保证 `Find` 操作的时间复杂度为 `O(log N)`。
- **规律**：根节点一般为**负值**，且其**绝对值为树的大小**；子节点一般为**正值**，且**其值对应的是其父节点**。

### 2. 代码实现

#### 2.1 `Find`函数迭代版

```c
int Find(int x) {  
    // 不断向上查找父节点，直到找到根（parent[root] < 0）  
    while (parent[x] >= 0) {  
        x = parent[x];  
    }  
    return x;  
}
```

#### 2.2 `Find`函数递归版

```c
int Find(int x) {  
    if (parent[x] < 0) {  
        return x;  // 找到根节点  
    }  
    // 递归查找根，并更新当前节点的父节点为根  
    parent[x] = Find_Recursive(parent[x]);  
    return parent[x];  
}  
```

#### 2.3 `Union`函数

```C  
void Union(int a, int b) {  
    int rootA = Find(a);  
    int rootB = Find(b);// Find 函数主要实现找到根节点
    if (rootA == rootB) return;  // 已在同一集合  
    
    // 按大小合并
    // parent[rootA] 更小，说明绝对值越大，即 rootA 的集合更大 
    if (parent[rootA] < parent[rootB]) {  
        parent[rootA] += parent[rootB];  
        parent[rootB] = rootA;  
    } else {  
        parent[rootB] += parent[rootA];  
        parent[rootA] = rootB;  
    }  
}  
```  

### 3. 操作示例

初始集合 `{1, 2, 3, 4}`：

```C
parent[1] = -1  
parent[2] = -1  
parent[3] = -1  
parent[4] = -1  
```  

执行 `Union(1, 2)` 后：

```C
parent[1] = -2  // 根节点，集合大小为2  
parent[2] = 1   // 子节点指向父节点
parent[3] = -1  // 其他元素不变
parent[4] = -1  // 其他元素不变
```

此时树的结构为：

```
    1 (size=2)  
   /  
  2  
```

执行 `Union(1, 3)`后

```C
parent[1] = -3  // 根节点，集合大小为3
parent[2] = 1   // 子节点指向父节点
parent[3] = 1   // 子节点指向父节点
parent[4] = -1  // 其他元素不变
```

此时树的结构为：

```
    1 (size=2)  
   / \
  2   3
```

执行 `Find(3)` 后会得到：

```
1
```

## 四、路径压缩（Path Compression）  
### 1. 目标  
缩短查找路径，使后续 `Find` 操作更快。  

### 2. 实现方法
- **递归版**：在 `Find` 过程中将节点的父节点直接指向根。
 
```C  
int Find(int x) {  
    if (parent[x] < 0) return x;  
    return parent[x] = Find(parent[x]); // 路径压缩  
}  
```  

- **迭代版**：两次遍历，先找根，再更新路径上的所有节点。

```C  
int Find(int x) {  
    int root = x;
    // 找到根
    while (parent[root] >= 0) root = parent[root];  
    while (x != root) { // 压缩路径  
        int temp = parent[x];  
        parent[x] = root;  
        x = temp;  
    }  
    return root;  
}  
```  

### 3. 示例  
查找路径 `3 → 2 → 1 → 根`，压缩后变为：  
```
3 → 根  
2 → 根  
1 → 根  
```  

## 五、时间复杂度分析  
| 方法                | 时间复杂度（均摊） | 适用场景         |  
|---------------------|-------------------|------------------|  
| 普通合并 + 普通查找 | $O(N)$              | 小规模数据       |  
| 按大小合并          | $O(\log N)$          | 频繁合并         |  
| 路径压缩            | $O(\alpha (N))$           | 大规模数据       |  

!!! note  
    **阿克曼函数反函数 α(N)**：几乎为常数（在真实数据中不超过 5）。  

---

## 六、实例演练  
**题目**：初始集合 `{1, 2, 3, 4, 5}`，依次执行：  
`Union(1, 2)`、`Union(3, 4)`、`Union(1, 3)`、`Find(4)`。  

**步骤**：  
1. `Union(1, 2)` → 树根为 1，大小 2。  
2. `Union(3, 4)` → 树根为 3，大小 2。  
3. `Union(1, 3)` → 合并后根为 1（更大树），大小 4。  
4. `Find(4)` → 路径压缩后 `4 → 1`。  


## 七、常见问题答疑  

### 1. 为什么要按大小合并？  

避免树退化成链表，保证树的高度平衡。  

### 2. 路径压缩会破坏树的高度吗？  

会的，但“按秩合并”中的“秩”可以理解为估计高度，实际影响不大。  

### 3. 如何判断两个元素是否属于同一集合？ 

检查它们的根节点是否相同：`Find(a) == Find(b)`。  