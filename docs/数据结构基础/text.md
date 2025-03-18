# 数据结构基础：搜索树（二叉搜索树）

## §2 二叉树

### 斜二叉树
- **左斜二叉树**：所有节点只有左子树的二叉树。
- **右斜二叉树**：所有节点只有右子树的二叉树。

### 完全二叉树
- 所有叶子节点位于相邻的两层。
- 除最后一层外，其他层的节点数达到最大值。

### 二叉树的性质
1. **每层最大节点数**  
   第 $i$ 层的最大节点数为 $2^{i-1}$（$i \geq 1$）。
2. **总节点数**  
   深度为 $k$ 的二叉树最多有 $2^k - 1$ 个节点（$k \geq 1$）。
3. **叶子节点与度为2节点的关系**  
   对于非空二叉树，叶子节点数 $n_0 = n_2 + 1$（$n_2$ 为度为2的节点数）。

**证明思路**  
- 总节点数 $n = n_0 + n_1 + n_2$。
- 分支数 $B = n_1 + 2n_2$（每个度为1的节点贡献1个分支，度为2的贡献2个分支）。
- 总节点数 $n = B + 1$（根节点无父分支）。
- 联立方程可得 $n_0 = n_2 + 1$。

---

## §3 二叉搜索树（BST）

### 定义
- 二叉树，可为空。
- 非空时满足以下条件：
  1. 每个节点有唯一整数键。
  2. 左子树所有节点的键小于根节点的键。
  3. 右子树所有节点的键大于根节点的键。
  4. 左、右子树也是二叉搜索树。

### ADT（抽象数据类型）
支持以下操作：
- `MakeEmpty(T)`：创建空树。
- `Find(X, T)`：查找键为 `X` 的节点。
- `FindMin(T)`/`FindMax(T)`：查找最小/最大键节点。
- `Insert(X, T)`：插入键 `X`。
- `Delete(X, T)`：删除键 `X`。
- `Retrieve(P)`：获取节点 `P` 的值。

---

### 实现细节

#### 查找操作
- **递归实现**  
  从根节点开始，根据键的大小递归查找左/右子树。
  ```c
  Position Find(ElementType X, SearchTree T) {
      if (T == NULL) return NULL;
      if (X < T->Element) return Find(X, T->Left);
      else if (X > T->Element) return Find(X, T->Right);
      else return T;
  }
  ```
- **迭代实现**  
  通过循环替代递归，减少函数调用开销。
  ```c
  Position Iter_Find(ElementType X, SearchTree T) {
      while (T) {
          if (X == T->Element) return T;
          T = (X < T->Element) ? T->Left : T->Right;
      }
      return NULL;
  }
  ```

#### 查找最小/最大值
- **最小值**：沿左子树递归至最左叶子。
  ```c
  Position FindMin(SearchTree T) {
      if (T == NULL) return NULL;
      return (T->Left) ? FindMin(T->Left) : T;
  }
  ```
- **最大值**：沿右子树迭代至最右叶子。
  ```c
  Position FindMax(SearchTree T) {
      if (T != NULL)
          while (T->Right) T = T->Right;
      return T;
  }
  ```

#### 插入操作
- 递归查找插入位置，若节点为空则创建新节点。
  ```c
  SearchTree Insert(ElementType X, SearchTree T) {
      if (T == NULL) {
          T = malloc(sizeof(TreeNode));
          T->Element = X;
          T->Left = T->Right = NULL;
      } else if (X < T->Element) {
          T->Left = Insert(X, T->Left);
      } else if (X > T->Element) {
          T->Right = Insert(X, T->Right);
      }
      return T; // 忽略重复键
  }
  ```

#### 删除操作
分三种情况处理：
1. **叶子节点**：直接删除，父节点指针置空。
2. **单子节点**：用子节点替换被删除节点。
3. **双子节点**：  
   - 用左子树的最大值或右子树的最小值替换当前节点。
   - 递归删除被替换的节点。

**代码实现**  
```c
SearchTree Delete(ElementType X, SearchTree T) {
    if (T == NULL) Error("Element not found");
    else if (X < T->Element) T->Left = Delete(X, T->Left);
    else if (X > T->Element) T->Right = Delete(X, T->Right);
    else {
        if (T->Left && T->Right) { // 双子节点
            Position Tmp = FindMin(T->Right);
            T->Element = Tmp->Element;
            T->Right = Delete(T->Element, T->Right);
        } else { // 单子或叶子节点
            Position Tmp = T;
            T = (T->Left) ? T->Left : T->Right;
            free(Tmp);
        }
    }
    return T;
}
```

---

### 延迟删除（Lazy Deletion）
- 为节点添加标记字段，标记为“已删除”而非立即释放内存。
- 优点：减少频繁的内存操作。
- 缺点：若删除节点过多，可能影响查找效率。

---

### 平均情况分析
- **树高与插入顺序相关**：
  - 最优情况：平衡插入（如中位数优先），树高为 $O(\log n)$。
  - 最差情况：顺序插入（如升序或降序），树高为 $O(n)$。
- **示例**：
  - 插入序列 `4, 2, 1, 3, 6, 5, 7` 生成平衡树（高度为2）。
  - 插入序列 `1, 2, 3, 4, 5, 6, 7` 生成右斜树（高度为6）。

---

**总结**  
二叉搜索树的效率高度依赖树的结构。理想情况下，平衡树的操作时间复杂度为 $O(\log n)$，但极端情况下可能退化为链表（$O(n)$）。实际应用中常使用平衡二叉搜索树（如AVL树、红黑树）来优化性能。