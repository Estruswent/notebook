# 一、搜索树类

## AVL / RB / Splay

| operation             | **AVL Tree** | **Red-Black Tree** | **Splay Tree**                    |
| -------------- | ------------ | ------------------ | --------------------------------- |
| Search         | $O(\log N)$  | $O(\log N)$        | $O(N)$ worst / $O(\log N)$ amort. |
| Insert         | $O(\log N)$  | $O(\log N)$        | $O(N)$ worst / $O(\log N)$ amort. |
| Delete         | $O(\log N)$  | $O(\log N)$        | $O(N)$ worst / $O(\log N)$ amort. |
| **旋转（Search）** | **0**        | **0**              | **Θ(depth)**（每次 access）           |
| **旋转（Insert）** | **≤ 2**      | **≤ 2**            | **Θ(depth)**                      |
| **旋转（Delete）** | **O(log n)** | **≤ 3**            | **Θ(depth)**                      |
| 是否每次 access 调整 | ❌            | ❌                  | ✔                                 |
| 高度保证           | ✔            | ✔                  | ❌                                 |

# 二、堆类

## 堆结构旋转/交换次数

| operation          | **Leftist Heap**       | **Skew Heap**         |
| ---------- | ---------------------- | --------------------- |
| Find-min   | (O(1))                 | (O(1))                |
| Merge      | $O(\log N)$ worst-case | $O(N)$ worst / $O(\log N)$ amort. |
| Insert     | $O(\log N)$            | $O(N)$ worst / $O(\log N)$ amort. |
| Delete-min | $O(\log N)$            | $O(N)$ worst / $O(\log N)$ amort. |
| Extra info | NPL                    | ❌ 无                   |

??? warning 易错点

    The number of **light nodes** along the right path of a **skew heap** is $O(\log N)$.

    **The worst case** of the number of nodes along the right path of a **skew heap** is $O(N)$.

    **The worst case** of the number of nodes along the right path of a **leftist heap** is $O(N)$.