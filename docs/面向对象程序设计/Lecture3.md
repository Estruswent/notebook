# Lecture 3：Object Interactive

## 1. 构造函数与析构函数

### 1.1 构造函数（Constructor）

定义：构造函数是一种特殊的成员函数，用于在创建对象时**自动初始化对象**。其名称与类名相同，无返回类型。

```cpp
class Point {
public:
    Point(int x, int y); // 构造函数声明
private:
    int x;
    int y;
};
```

特点：

- 构造函数在对象创建时**自动调用**，确保对象初始化。
- 可以重载，支持不同初始化方式。
- 如果没有显式定义构造函数，编译器会生成一个**默认构造函数**（无参数）。

示例：
```cpp
Point a(1, 2); // 调用构造函数初始化
```

### 1.2 默认构造函数

定义：可以无参数调用的构造函数。

如果类没有定义任何构造函数，编译器会**自动生成默认构造函数**。比如下面这个代码块中没有构造`X()`的函数声明，但如果你执意要写`X object;`，它仍然合法，因为编译器自动生成了一个默认构造函数。

```cpp
class X {
public:
    X();  // 默认构造函数
};

X object; // 此时就会调用X()创建对象object
```

如果定义了带参数的构造函数，编译器**不会**自动生成默认构造函数。那么如果此时你试图调用`X()`，就会出现错误。

```cpp
class X {
public:
    X(int value); // 带参数的构造函数
};

X obj;      // 错误：没有默认构造函数可用
X obj2(10); // 正确：这里应当调用 X(int)
```

### 1.3 析构函数（Destructor）

定义：析构函数用于在对象生命周期结束时**自动清理资源**。其名称是类名前加`~`，无参数和返回类型。

```cpp
class Y {
public:
    ~Y(); // 析构函数
};
```

特点：

- 对象超出作用域时自动调用。
- 不能重载，一个类只能有一个析构函数。

比如执行以下这个例子：

```cpp
#include <iostream>
using namespace std;

class Y {
public:
    Y() {
        cout << "构造函数调用" << endl;
    }

    ~Y() {
        cout << "析构函数调用" << endl;
    }
};

int main() {
    {
        Y obj; // 创建对象
    } // 这个时候捏，因为 obj 离开了作用域，析构函数在这里自动调用

    cout << "Y被清理了（悲" << endl;
    return 0;
}
```

预期输出结果是：

```
构造函数调用
析构函数调用
Y被清理了（悲
```

## 2. 初始化与赋值

定义：成员初始化列表，顾名思义，就是用于在对象创建时直接对成员变量进行初始化。它的作用就是在**构造函数中高效初始化成员变量**。

比如用以下代码作为例子：

```cpp
class Point {
public:
    Point(float xa, float ya) : x(xa), y(ya) {}
private:
    float x;
    float y;
};
```

对于`Point(float xa, float ya)`，我们已经知道，它是构造函数；而`: x(xa), y(ya)`就是成员初始化列表。它表示：在构造`Point`对象时，直接用`xa`初始化`x`，用`ya`初始化`y`。

!!! example "成员初始化列表"

    我在这里再举个具体而略有不同的例子吧。

    ```cpp
    #include <iostream>
    using namespace std;

    class Simple {
        int a;
        int b;
    public:
        Simple(int x, int y) : a(x), b(y) {
            cout << "构造函数中：" << endl;
            cout << "a = " << a << ", b = " << b << endl;
        }
    };

    int main() {
        Simple obj(10, 20); // 调用构造函数，初始化列表设置
        // 预期结果：a = 10, b = 20
        return 0;
    }
    ```

它的特点也很鲜明：

- 直接初始化，避免先默认构造再赋值。

    什么意思呢？也就是说，当你用这种方法的时候，它**只进行一次构造而没有多余的赋值**；
    相反，如果你在构造函数体内赋值，就会出现**一次构造+一次赋值**，效率较低。

    下面这个代码很容易看出二者的区别。

    !!! example "初始化 vs 赋值"
        ```cpp
        // 初始化（更高效）
        Student::Student(string s) : name(s) {}

        // 赋值（需要默认构造函数）
        Student::Student(string s) { name = s; }
        ```
    
- 初始化顺序与成员声明顺序一致（与初始化列表顺序无关）。

    这个也有点抽象，举个例子很好懂。

    !!! example "初始化的顺序"

        如你所见，在下面这块代码中，我们先声明了`b`，接着声明了`a`。

        那么虽然你在初始化列表里写的是`a(x), b(y)`，但实际上类成员变量的声明顺序由于`b`在前，`a`在后，所以它们的初始化顺序是`b(y)`，然后才是`a(x)`

        ```cpp
        class MyClass {
            int b;
            int a;
        public:
            MyClass(int x, int y) : a(x), b(y) {}
        };
        ```

## 3. 函数重载与默认参数

### 3.1 函数重载

定义：在**同一作用域**内，函数名相同但参数列表不同。也就是说，函数重载允许你在同一个作用域中使用相同的函数名，只要它们的参数列表不同。而编译器会根据你调用时传入的参数自动选择正确的函数版本。（而我们之前学的C语言一般不支持函数重载这个东西）

以下面的函数为例：

```cpp
void print(char str, int width);
void print(char *str, int width);
void print(double d, int width);
void print(int i, int width);
```

| 调用方式 | 匹配哪个函数 |
|----------|---------------|
| `print("1.00", 10);` | `void print(char *str, int width)`|
| `print('1', 10);`     | `void print(char str, int width)`|
| `print(1, 10);`      | `void print(int i, int width)`|
|  `orint(1.00, 10)`      | `void print(double d, int width)`|



### 3.2 默认参数

定义：为函数参数提供默认值，调用时可省略。

说起来不那么具象化，具体是什么意思呢？举个例子：

```cpp
// 芝士一个函数
int Wen(int n, int m = 4, int j = 5);
```

在上述代码中，我们将第二位参数`m`默认值设置为4，第三位参数`j`的默认值设置为5；

而当你调用函数函数时，如果对第二位和第三位的赋值直接忽略，它就会用默认值来隐式赋值。

当然，在这个例子里，第一位`n`必须赋值，不然会报错，毕竟没有设置默认值嘛。

```cpp
Wen(10);         // 使用默认值：m=4, j=5
Wen(10, 20);     // 使用 j=5
Wen(10, 20, 30); // 全部显式传值
```

规则：

- 默认参数必须从右向左连续定义。
    即：不能在某个参数**有默认值之前，跳过它后面的参数**。
    ```cpp
    int Wen(int a = 10, int b, int c); // 错误：a是默认参数，但后面b和c却是非默认
    ```
- 不能与非默认参数交叉。
    即：默认参数之间**不能夹着“没有默认值”的参数**。
    ```cpp
    // 错误：b和d是默认参数，但c却是非默认参数
    int Wen(int a, int b = 20, int c, int d = 40);
    ```

## 4. const关键字

### 4.1 const对象

定义：一个对象创建后，其状态不可修改。

你可以声明一个对象（Object）为`const`，表示这个对象一旦创建后，它的状态（即成员变量）就不能再被修改，这个和我们的C是如出一辙的。

```cpp
const Currency the_raise(42, 38);
```

限制：

- 只能调用`const`成员函数。

    关于`const`成员函数，请见下文。

- 不能修改任何成员变量。

    不能直接或间接地修改`const`的任何成员变量，这个很好理解。

### 4.2 const成员函数

- **定义**：在 C++ 中，你可以将一个成员函数声明为`const`，此时这个声明代表了**这个函数不会修改类的成员变量**（即对象的状态）。
- **使用方法**：把`const`关键字写在函数参数列表之后，函数体之前。（如下例代码所示）

```cpp
class Date {
public:
    int get_day() const; // 不修改成员变量
};
```

- **特点**：
    - 可以被`const`对象调用。

        !!! example "`const`对象调用`const`函数"
            `const`对象只能调用`const`成员函数在下面的代码中，因为`today`是一个`const Date`类型的对象，它不允许被修改，所以只能调用那些标记为`const`的成员函数（如`get_day()`）。而`set_day()`没有标记为`const`，意味着它可能修改对象状态，因此不能被`const`对象调用。
            ```cpp
            class Date {
            public:
                int get_day() const;
                void set_day(int d);
            };

            int main() {
                const Date today; // 常量对象
                today.get_day();  // 正确：因为 get_day 是 const 函数
                today.set_day(5); // 错误：set_day 不是 const 函数
            }
            ```

    - 不能调用非`const`成员函数。

        这句话是指：一个`const`成员函数内部不能调用其他非`const`成员函数。

        !!! example "不能调用非`const`成员函数"
            比如在下面这串代码中，`func2`就不能在内部调用到`func1`。
            ```cpp
            class MyClass {
            public:
                void func1();
                void func2() const {
                    func1(); // 这里错误：不能从 const 函数中调用非 const 函数
                }
            };
            ```

## 5. 类与对象设计

### 5.1 类的作用域

#### 1. 成员变量（字段 / Member Variables）

**定义**：

- 成员变量是定义在**类内部**（但在任何函数外部）的变量。
- 它们属于类的**状态（State）**，描述对象的属性。

**特点**：

1. **作用域**：
      - 在类的**所有成员函数**（方法）中都可以访问。
      - 在类外部，如果成员是 `public`，可以通过对象访问；如果是 `private`，则不能直接访问。

2. **生命周期**：
    - 成员变量的生命周期**与对象相同**：
        - 对象创建时，成员变量被初始化（构造函数调用）。
        - 对象销毁时，成员变量被释放（析构函数调用）。

3. **初始化方式**：
    - 可以在**构造函数**中初始化（推荐使用**初始化列表**）。
    - 如果没有显式初始化，基本类型（如 `int`、`float`）会初始化为**未定义值**（除非使用 `= default` 或类内初始化）。

**示例**：

```cpp
class Student {
private:
    string name;  // 成员变量（字段）
    int age;
public:
    Student(string n, int a) : name(n), age(a) {}  // 构造函数初始化成员变量
    void printInfo() {
        cout << "Name: " << name << ", Age: " << age << endl;  // 成员变量在类内可直接访问
    }
};

int main() {
    Student s("Alice", 20);
    s.printInfo();
    // cout << s.name;  // 错误：name是private，不能在类外直接访问
    return 0;
}
```

#### 2. 局部变量（Local Variables）

**定义**：

- 局部变量是在**函数或代码块内部**定义的变量。
- 它们的作用域仅限于**定义它们的函数或代码块**。

**特点**：

1. **作用域**：

    - 仅在**定义它们的函数或代码块**内有效。
    - 不能在函数外部访问。

2. **生命周期**：

    - 在**函数调用时创建**，在**函数返回时销毁**（自动释放）。
    - 如果是 `static` 局部变量，生命周期会延长到程序结束（但作用域仍然限于函数）。

3. **初始化方式**：

    - 必须**显式初始化**，否则值是未定义的（可能导致未定义行为）。

**示例**：

```cpp
void exampleFunction() {
    int localVar = 10;  // 局部变量
    cout << "Local variable: " << localVar << endl;
}  // localVar 在这里被销毁

int main() {
    exampleFunction();
    // cout << localVar;  // 错误：localVar 的作用域仅限于 exampleFunction
    return 0;
}
```

#### 3. 参数（Parameters）
**定义**：

- 参数是**函数或方法的输入变量**，在函数调用时由调用者传递。
- 它们的作用域和生命周期类似于**局部变量**。（本质上就是特殊点的局部变量）

**特点**：

1. **作用域**：

    - 仅在**函数内部**有效。
    - 不能在函数外部访问。

2. **生命周期**：

    - 在**函数调用时创建**，在**函数返回时销毁**。
    - 如果是**引用或指针参数**，它们可能指向外部变量（但参数本身仍然是局部的）。

3. **传递方式**：

    - **值传递（Pass by Value）**：函数内部修改不影响外部变量。
    - **引用传递（Pass by Reference）**：函数内部修改会影响外部变量。
    - **指针传递（Pass by Pointer）**：类似引用传递，但可以传递 `nullptr`。

**示例**：

```cpp
void modifyValue(int x) {  // 值传递（参数x是局部变量）
    x = 100;  // 修改的是局部副本，不影响外部变量
}

void modifyReference(int &x) {  // 引用传递（x是外部变量的别名）
    x = 100;  // 修改会影响外部变量
}

int main() {
    int a = 10;
    modifyValue(a);
    cout << "After modifyValue: " << a << endl;  // 仍然是10

    modifyReference(a);
    cout << "After modifyReference: " << a << endl;  // 现在是100
    return 0;
}
```

### 5.2 封装与抽象

封装和抽象是 OOP 的两大核心概念，这二者可以共同作用，让代码更模块化、更安全、更易维护。

#### 1. 封装（Encapsulation）

**定义**：封装是指**将数据（成员变量）和操作数据的方法（成员函数）捆绑在一起**，并对外隐藏内部实现细节，仅暴露必要的接口。

**核心思想**：

- **隐藏实现细节**：外部代码不需要知道类内部如何工作，只需知道如何使用它。
- **控制访问权限**：通过 `public`、`private`、`protected` 关键字限制对成员的访问。
- **数据保护**：防止外部代码直接修改对象内部状态，避免数据不一致。

**如何实现封装？**

1. **使用 `private` 保护数据**

    - 成员变量通常设为 `private`，防止外部直接修改。
    - 提供 `public` 方法（getter/setter）来间接访问或修改数据。

2. **暴露必要的接口**：只提供必要的 `public` 方法，隐藏复杂的内部逻辑。

!!! example "示例：银行账户"
    下面的代码演示了如何封装一个银行账户，在实现存取款和查询余额的情况下尽可能多地隐藏细节。
    ```cpp
    class BankAccount {
    private:
        double balance;  // 私有成员变量，外部无法直接访问

    public:
        // 公有方法，提供受控的访问方式
        void deposit(double amount) {
            if (amount > 0) balance += amount;
        }

        bool withdraw(double amount) {
            if (amount <= balance) {
                balance -= amount;
                return true;  // 取款成功
            }
            return false;  // 余额不足
        }

        double getBalance() const {  // const 表示不修改对象状态
            return balance;
        }
    };

    int main() {
        BankAccount account;
        account.deposit(1000);  // 通过公有方法修改 balance
        account.withdraw(500);
        cout << "Current balance: " << account.getBalance() << endl;  // 输出 500

        // account.balance = 1000000;  // 错误：balance 是 private，不能直接访问
        return 0;
    }
    ```

**封装的好处**：

- **安全性**：防止非法修改（如直接设置 `balance = -1000`）。
- **灵活性**：可以随时修改内部实现（如改用 `long` 存储余额），而不影响外部代码。
- **易维护**：外部代码只依赖公有接口，不依赖具体实现。

#### 2. 抽象（Abstraction）

**定义**：抽象是指**仅向用户展示核心功能，隐藏不必要的细节**。它强调的是“做什么”（What），而不是“怎么做”（How）。

**核心思想**：

- **简化**复杂性：用户不需要知道类的内部实现，只需知道如何使用它。
- **定义清晰的接口**：类通过公有方法提供明确的操作方式。

**如何实现抽象？**：

1. **定义清晰的公有方法**：比如，`Car` 类提供 `start()`、`accelerate()`、`brake()` 等方法，但隐藏引擎如何工作的细节。

2. **隐藏底层**实现：例如，`ClockDisplay` 类内部可能用两个 `NumberDisplay` 对象存储小时和分钟，但用户只需要知道 `setTime()` 和 `display()`。

!!! example "示例：时钟显示"

    下面的代码就是表现一个内部复杂逻辑而外在接口简单易懂的时钟。

    ```cpp
    class NumberDisplay {
    private:
        int value;
        int limit;  // 最大值（如分钟限制为60）

    public:
        NumberDisplay(int lim) : limit(lim), value(0) {}
        void increment() {
            value = (value + 1) % limit;  // 超过限制时归零
        }
        int getValue() const { return value; }
    };

    class ClockDisplay {
    private:
        NumberDisplay hours;    // 内部使用 NumberDisplay 管理小时
        NumberDisplay minutes;  // 内部使用 NumberDisplay 管理分钟

    public:
        ClockDisplay() : hours(24), minutes(60) {}  // 初始化小时（0-23）、分钟（0-59）

        // 对外提供简单的设置时间接口
        void setTime(int h, int m) {
            // 内部涉及复杂的逻辑（如校验时间合法性）
            hours.increment();   // 仅示例，实际可能更复杂
            minutes.increment(); // 这里简化了
        }

        // 显示时间
        void display() const {
            cout << "Time: " << hours.getValue() << ":" << minutes.getValue() << endl;
        }
    };

    int main() {
        ClockDisplay clock;
        clock.setTime(10, 30);  // 用户不需要知道内部如何存储时间
        clock.display();        // 输出 "Time: 11:31"（仅示例）
        return 0;
    }
    /* 到这里之后，你会发现类里的实现（哪怕简化后）对初学者来说很复杂，但是在main里却简单易懂 */
    ```

**抽象的优点**：

- **降低复杂度**：用户只需知道 `setTime()` 和 `display()`，不需要关心 `NumberDisplay` 如何工作。
- 提高**可扩展性**：可以随时优化内部实现（如改用 `string` 存储时间），而不影响调用方。

#### 3. 封装与抽象的类比

实际上，二者常常一起使用，这里只是为了让概念理解更清晰。

| **特性**       | **封装（Encapsulation）**               | **抽象（Abstraction）**               |
|----------------|----------------------------------------|---------------------------------------|
| **核心目标**   | 保护数据，隐藏实现细节                 | 简化复杂性，提供清晰接口              |
| **实现方式**   | 使用 `private`/`public` 控制访问权限   | 定义高层次的接口，隐藏底层逻辑        |
| **关注点**     | **数据隐藏**（防止非法访问）           | **接口设计**（让调用方更易使用）      |
| **典型应用**   | Getter/Setter 方法                     | 提供简单的方法（如 `startEngine()`）  |

#### 4. 其它补充

- 应用场景举例：额，挺多的。

    比如你正在修读”数据库系统“的话，Java语言（也是一门面向对象的编程语言）里面的JDBC就有，它隐藏了 SQL 的一些细节，接口名字也简单易懂，比如`executeQuery`；

    又比如，某MMO游戏为了防止作弊，把角色`Player`类封装`health`，只能通过`takeDamage()`等指令修改。

- 不要忽略**const方法**：如果方法不修改对象状态，应该标记为 `const`（如 `getBalance() const`）。

## 6. 内存管理

### 6.1 存储分配

在 C++ 中，对象的内存是在它所在的代码块被执行时分配的，构造函数在你定义对象的那一行代码处被调用。

- 对象存储空间在进入作用域时分配。

以下面代码为例子。

```cpp
void func() {
    {
        Point p1(1, 2); // 定义一个对象
        // 使用 p1 ...
    } // p1 离开作用域，内存释放，析构函数被调用

    {
        Point p2(3, 4); // 又定义一个对象
        // 使用 p2 ...
    } // p2 离开作用域
}
```

当程序执行到`{ Point p1(1, 2); ... }`这个代码块时，`p1`的存储空间就被分配了，其生命周期仅限于这个花括号内部。一旦离开这个作用域（遇到`}`），`p1`的析构函数会被自动调用，内存也会被释放。（`p2`以此类推）

- 构造函数在对象定义点调用。

这一点其实很好懂，以下面代码为例子。

```cpp
#include <iostream>
using namespace std;

class MyClass {
public:
    MyClass() {
        cout << "构造函数调用" << endl;
    }

    ~MyClass() {
        cout << "析构函数调用" << endl;
    }
};

void demo( ) {
    cout << "函数开始" << endl;

    MyClass obj; // ← 先前构造函数在这里被调用

    cout << "函数结束" << endl;
}
```

我们的预期输出结果是：

```txt
函数开始
构造函数调用
函数结束
析构函数调用
```

### 6.2 资源管理建议

- 避免在函数中分配资源并返回指针。
- 使用RAII（Resource Acquisition Is Initialization）模式管理资源。

```cpp
// 不推荐
char* foo() {
    char* p = new char[10];
    return p; // 调用者需记得释放
}

// 推荐使用智能指针或明确所有权
```

## 7. 聚合初始化

用途：初始化数组或结构体。

```cpp
struct X {
    int i;
    float f;
    char c;
};

X x1 = {1, 2.2, 'c'}; // 聚合初始化
```

特点：

- 按声明顺序初始化成员。
- 可用于数组和简单类类型。
