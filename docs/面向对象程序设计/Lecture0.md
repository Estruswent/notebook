# Lecture0: Course Introduction

## 一、课程目标

通过 **C++** 学习面向对象编程。

## 二、OOP核心概念（Buzzwords）

- 封装（Encapsulation） 
- 继承（Inheritance）  
- 多态（Polymorphism）  
- 模板（Template）  
- 接口（Interface）  
- 耦合（Coupling）与内聚（Cohesion）  
- 迭代器（Iterators）  
- 方法重写（Overriding）  
- ...

## 三、课程资料

### 1. 教材

- 《Thinking in C++》（Vol.1 & 2）  
- 《C++ Primer》（第5版）  

### 2. 参考书

- 《The C++ Programming Language》  
- 《Effective C++》  
- 《C++ Templates》  

## 四、课程分数组成（xww班）

- 实验（50%）  ：**每双周**提交（第2、4、6...16周），截止时间为周六晚上24:00。  
- 期末考试（50%）  

## 五、关于C++

### 1. C++的简单例子

Hello World（懂的都懂）

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World! I am " << 18 << " Today!" << endl;
    return 0;
}
```

输入输出数字

```cpp
#include <iostream>
using namespace std;

int main() {
    int number;
    cout << "Enter a decimal number: ";
    cin >> number;
    cout << "The number you entered is " << number << "." << endl;
    return 0;
}
```

### 2.C vs C++

\[
\left\{
\begin{array}{l@{\quad}l}
\text{C语言局限性} & \left\{
\begin{array}{l}
\text{类型检查不足} \\
\text{不支持大规模编程} \\
\text{面向过程编程}
\end{array}
\right. \\
\text{C++改进} & \left\{
\begin{array}{l}
\text{数据抽象} \\
\text{访问控制} \\
\text{函数重载} \\
\text{模板与STL} \\
\text{异常处理}
\end{array}
\right. \\
\end{array}
\right.
\]

## 六、联系老师

邮件取名开头`[oop]`，正文注明姓名和学号，使用纯文本格式。