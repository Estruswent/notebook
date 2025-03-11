# 树、二叉树与线索二叉树

## 一、术语

- 树：节点的集合，包含一个根节点和若干子树。
- 根节点 (Root)：树的唯一起始节点。
- 叶子节点 (Leaf)：度为0的节点（无子节点）。
- 父节点 (Parent)：拥有子树的节点。
- 子节点 (Children)：父节点的直接下级节点。
- 兄弟节点 (Siblings)：同一父节点的子节点。
- 深度 (Depth)：从根到该节点的唯一路径长度（根深度为0）。
- 高度 (Height)：从节点到最深叶子的最长路径长度（叶子高度为0）。
- 路径：节点序列，每个节点是下一节点的父节点。
- 度(degree)：节点的度表示拥有的子树数目；树的度是所有节点的度的最大值。

##  二、树的表达方式

- 列表表示：用嵌套括号表示层次结构（如 `A(B(C), D)`）。
- FirstChild-Sibling表示法：每个节点记录第一个子节点和下一个兄弟节点，通过旋转45°可转换为二叉树结构。

![alt text](image.png)

![alt text](image-1.png)

## 三、二叉树

### 1. 定义

每个节点最多有两个子节点（称为左子节点和右子节点）的树。

### 2. 遍历方式

前序遍历：根 → 左子树 → 右子树  

```cpp
/* 递归实现 */
void preorderTraversal(TreeNode* root) {
    if (root == nullptr) return;// 空树
    cout << root->val << " ";
    preorderTraversal(root->left);
    preorderTraversal(root->right);
}
```

```cpp
/* 迭代实现 */
void preorderTraversal(TreeNode* root) {
    if (root == nullptr) return;
    stack<TreeNode*> s;// 辅助栈
    s.push(root);
    while (!s.empty()) {
        TreeNode* node = s.top();
        s.pop();
        cout << node->val << " ";
        if (node->right) s.push(node->right);
        if (node->left) s.push(node->left);
    }
}
```

中序遍历：左子树 → 根 → 右子树

```cpp
/* 递归实现 */
void inorderTraversal(TreeNode* root) {
    if (root == nullptr) return;// 空树
    inorderTraversal(root->left);
    cout << root->val << " ";
    inorderTraversal(root->right);
}
```

```cpp
/* 迭代实现 */
void inorderTraversal(TreeNode* root) {
    stack<TreeNode*> s;// 辅助栈
    TreeNode* current = root;
    while (current != nullptr || !s.empty()) {
        while (current != nullptr) {
            s.push(current);
            current = current->left;// 一直向左遍历，直到空
        }
        current = s.top();
        s.pop();
        cout << current->val << " ";
        current = current->right;// 最后再转回右节点
    }
}
```

后序遍历：左子树 → 右子树 → 根

```cpp
/* 递归实现 */
void postorderTraversal(TreeNode* root) {
    if (root == nullptr) return;
    postorderTraversal(root->left);
    postorderTraversal(root->right);
    cout << root->val << " ";
}
```

```cpp
/* 迭代实现 */
void postorderTraversal(TreeNode* root) {
    if (root == nullptr) return;
    stack<TreeNode*> s;
    TreeNode* lastVisited = nullptr;
    while (!s.empty() || root != nullptr) {
        if (root != nullptr) {
            s.push(root);
            root = root->left;
        } else {
            TreeNode* peekNode = s.top();
            if (peekNode->right != nullptr && lastVisited != peekNode->right) {
                root = peekNode->right;
            } else {
                cout << peekNode->val << " ";
                lastVisited = s.pop();
            }
        }
    }
}
```

层序遍历：按一层一层的顺序遍历树的节点。

```cpp
void levelorderTraversal(TreeNode* root) {
    if (root == nullptr) return;
    queue<TreeNode*> q;
    q.push(root);// 根入队
    while (!q.empty()) {
        TreeNode* node = q.front();// 获得队首元素
        q.pop();// 队首元素出队
        cout << node->val << " ";
        if (node->left) q.push(node->left);// 左子树入队
        if (node->right) q.push(node->right);// 右子树入队
    }
}
```

3. 表达式转化为树

| 表达式类型 | 遍历方式       |树构建方法               |
|------------|----------------|--------------------------|
| **中缀**   | 中序遍历       |根节点可以通过左右个数判断，左边为根节点的左子树，右边为根节点的右子树 |
| **后缀**   | 后序遍历       |根节点为最后一个，从左到右构建子树，一步一步拼接        |
| **前缀**   | 前序遍历       |根节点为第一个，从左到右构建子树，一步一步拼接        |

**(FDS HW4)** 例题：Given the shape of a binary tree shown by the figure below. If its **inorder** traversal sequence is { E, A, D, B, F, H, C, G }, then the node on the same level of C must be:

![alt text](19185355-2b08-4b74-9bb7-8262720437bd.jpg)

显而易见，F为根节点，左侧为{E, A, D, B}，右侧为{H, C, G}，所以E为C的兄弟节点。

**(FDS HW4 Derivative)** 例题：Given the shape of a binary tree shown by the figure which has been mentioned in the previous question. If its **preorder** traversal sequence is { E, A, D, B, F, H, C, G }, then the node on the same level of C must be:

显而易见，E为根节点，左侧为{A, D, B, F}，右侧为{H, C, G}，所以D和G为C的兄弟节点。

4. 树转化为表达式

按照对应的遍历读树，写成表达式即可。

例题：某表达式的前序遍历树如下，求前序表达式。

```
  +
 / \
A   *
   / \
  B   C
```

前序表达式为 `+A*BC`。

## 四、线索二叉树

### 1. 作用

普通二叉树有 `n+1` 个空指针，浪费空间，而线索化利用空指针指向节点的前驱或后继，加速遍历。

### 2. 规则

1. 若左子为空，指向中序前驱。
2. 若右子为空，指向中序后继。
3. 必须包含（**没有就自己添加**）头节点，左指针指向树的根节点。

### 3. 示例

表达式树 `A + B * C / D` 的线索化。

![alt text](image-2.png)

比如对于`B`而言，其中序表达式（或者说中序遍历）的两侧正好是+号和*号。