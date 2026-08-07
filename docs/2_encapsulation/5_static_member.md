# 类的静态成员

在面向对象程序设计中，有些信息虽然与类相关，但并不隶属于某个具体的对象实例。
当一份数据属于整个类，而不属于某一个对象时，静态成员就登场了。

## 为什么需要静态成员

!!! question "核心问题：如何统计当前系统中共有多少个Point对象？"

这个问题看似简单，但用不同的方式解决，会带来截然不同的后果。我们来看看几种逐步演进的设计方案。

### 方案一：使用局部变量

最直接的想法是：在需要计数的地方（比如`main`函数中）使用一个局部变量来记录对象数量。

!!! failure "方案一：局部变量"

    ``` cpp linenums="1" hl_lines="18 21 23 29 32"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}
        ~Point() {}

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        int pointCount = 0;   // 局部变量：由调用者手动维护

        Point p1(3, 4);
        ++pointCount;         // 手动增加
        Point p2(5, 6);
        ++pointCount;         // 手动增加

        cout << "Count: " << pointCount << endl;   // 2

        {
            Point p3(7, 8);
            ++pointCount;     // 手动增加
            cout << "Count: " << pointCount << endl;   // 3
        }
        --pointCount;         // 离开作用域，需要手动减少（但容易遗漏）
        
        cout << "Count: " << pointCount << endl;   // 2

        return 0;
    }
    ```

!!! danger "局部变量方案的问题"

    - **计数器与对象生命周期脱节**：计数器的增减交由调用者手动管理。调用者必须在每个对象创建和销毁的地方都记得更新计数器。
    - **极易出错**：如果创建对象时忘了 `++pointCount`，或者异常退出时忘了 `--pointCount`，计数就会错误。这在大型程序中几乎是必然发生的。
    - **无法封装**：`pointCount` 暴露给所有使用 `Point` 的代码，每个调用者都需要知道“创建`Point`时要更新这个变量”的约定。
    - **作用域限制**：如果要跨函数维护计数，需要将 `pointCount` 作为参数传递，或者放到更高层级的作用域，实际上又回到了其他方案的问题。

!!! quote "局部变量的困境"

    局部变量就像把计数器交给每个使用者自己保管——有人记在纸上，有人记在手机里，总有人会忘记带，也总有人会记错。

### 方案二：使用全局变量

既然局部变量在每个作用域都需要单独维护太麻烦，那么把计数器放到全局区域让所有代码都能访问，并且通过构造/析构函数自动维护，能不能解决问题呢？

!!! failure "方案二：全局变量"

    ``` cpp linenums="1" hl_lines="4 9 13"
    #include <iostream>
    using namespace std;

    int pointCount = 0;   // 全局变量：记录Point对象的数量

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            ++pointCount;   // 构造时计数加1
        }

        ~Point() {
            --pointCount;   // 析构时计数减1
        }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        cout << "Count: " << pointCount << endl;   // 0

        Point p1(3, 4);
        Point p2(5, 6);

        cout << "Count: " << pointCount << endl;   // 2

        {
            Point p3(7, 8);
            cout << "Count: " << pointCount << endl;   // 3
        }   // p3 离开作用域，析构自动减1

        cout << "Count: " << pointCount << endl;   // 2

        return 0;
    }
    ```

!!! danger "全局变量方案的问题"

    - **数据与类型脱钩**：`pointCount` 并不属于 `Point` 类型，它只是恰好和 `Point` 一起使用。如果新的程序员不熟悉设计意图，可能不知道这个变量与 `Point` 的关联。
    - **访问不受限制**：全局变量是公开的，程序的**任何地方**都能修改它。可能在某个不相关的函数中意外写 `pointCount = 0`，导致计数错误。
    - **命名冲突风险**：如果程序中还需要统计其他类型（如 `Circle`、`Line`）的对象数量，就需要 `circleCount`、`lineCount` 等变量，全局命名空间变得拥挤。
    - **可维护性差**：当项目变大时，难以追踪 `pointCount` 在哪些地方被修改，bug 更难定位。

!!! quote "全局变量的困境"

    全局变量就像把家里的钥匙放在门口地毯下——方便了所有人，但也让所有人都能进来，包括那些不该进来的人。

### 方案三：使用普通成员变量

根据封装原则，既然计数器是针对特定类型对象的，那能不能把它作为 `Point` 的一个普通成员变量？

!!! failure "方案三：普通成员变量"

    ``` cpp linenums="1" hl_lines="22"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            ++count;
        }

        ~Point() {
            --count;   // 析构时计数减1
        }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

        int getCount() const { return count; }

    private:
        int x, y;
        int count;   // 普通成员变量：每个对象各有一份
    };

    int main() {
        Point p1(3, 4);
        Point p2(5, 6);

        cout << p1.getCount() << endl;   // 1（每个对象独立计数，都是1）
        cout << p2.getCount() << endl;   // 1

        // 问题：每个对象都有自己的 count，相互间独立计数
        // 无法得到"当前共有几个Point对象"这个全局信息

        return 0;
    }
    ```

!!! danger "普通成员变量方案的问题"

    - **每个对象各有一份**：`count` 是每个 `Point` 对象自己的成员，`p1.count` 和 `p2.count` 是独立的。当创建 `p1` 时，它不知道 `p2` 的存在。
    - **无法表达类级别的信息**：“有多少个对象”是一个**属于整个类型**的信息，而不是属于某个具体对象的信息。普通成员变量无法承载这种跨对象的共享状态。
    - **内存浪费**：每个 `Point` 对象都要额外存储一份 `count`，如果创建100个对象，就有100个计数器，且它们应该相同但实际上无法同步。
    - **无法实现跨对象计数**：由于每个对象的 `count` 是独立的，无法通过 `p1.count` 知道 `p2` 是否存在，计算总数变成不可能的任务。

!!! quote "普通成员变量的困境"

    普通成员变量就像给每个学生发一份全班名单——名单上只写着自己的名字，永远不知道班上还有谁。

### 三种方案对比总结

| 方案             | 实现方式                    | 问题                                       |
| :--------------- | :-------------------------- | :----------------------------------------- |
| **局部变量**     | 在作用域内由调用者手动增减  | 极易遗漏更新，分散在各个调用点，无法封装   |
| **全局变量**     | 构造/析构自动增减，不易遗漏 | 数据与类型脱钩，访问不受限制，容易意外修改 |
| **普通成员变量** | 作为成员由构造/析构自动增减 | 每个对象各有一份，无法表达类级别的共享信息 |

!!! abstract "结论：三种方案都不能完全解决问题"

    从以上三种方案的逐步分析可以看出，我们需要一种机制：

    1. **属于类型，而非对象**：数据与 `Point` 类型绑定，而不是与某个具体的点绑定。
    2. **所有对象共享**：无论创建多少个对象，数据只有一份，所有对象访问的是同一份副本。
    3. **自动维护**：计数器的更新由构造/析构函数自动完成，不需要调用者手动干预。
    4. **受控访问**：数据受类的访问控制保护，外部不能随意修改。

C++ 的**静态成员（Static Member）**正是为此而设计的。

## 静态数据成员

### 基本概念

静态数据成员（Static Data Member）由类的**所有对象共享**，无论创建了多少个对象，静态数据成员只有**一份副本**。

!!! info "静态数据成员的特点"

    - **共享性**：所有对象访问的是同一份数据。
    - **类级别**：它属于类，而不是属于某个对象。
    - **独立存储**：不占用单个对象的内存空间。
    - **生命周期**：在程序启动时或首次使用前创建，程序结束时销毁（静态生存期）。

### 声明与初始化

在 C++ 中静态数据成员的声明和初始化需要分开完成：

!!! example "Point类的静态数据成员"

    ``` cpp linenums="1" hl_lines="13 17"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            ++count;   // 构造时计数加1
        }

        ~Point() {
            --count;   // 析构时计数减1
        }

    private:
        int x, y;
        static int count;   // 类内声明：告诉编译器有这个成员
    };

    // 类外定义：分配存储空间，初始化为0
    int Point::count = 0;
    ```

!!! info "声明与初始化的区别"

    |                | **类内声明**               | **类外初始化**                 |
    | :------------- | :------------------------- | :----------------------------- |
    | **位置**       | 在类定义内部               | 在类定义外部（通常在.cpp文件） |
    | **语法**       | `static int count;`        | `int Point::count = 0;`        |
    | **作用**       | 告诉编译器成员的存在和类型 | 分配实际的存储空间             |
    | **是否可缺省** | 必须声明                   | 必须定义一次                   |

    !!! danger "常见错误：忘记对静态数据成员进行类外初始化"

        ``` cpp
        class Point {
            static int count;   // 只有声明
        };
        // 忘记写：int Point::count = 0;
        // 链接时会出现 "undefined reference" 错误
        ```

!!! note "类内初始化的特殊情况（C++17）"

    在C++17之前，只有`const`整型的静态数据成员可以在类内初始化。C++17引入了`inline static`，允许在类内直接初始化：

    ``` cpp
    class Point {
    public:
        static inline int count = 0;   // C++17: 可以直接在类内定义并初始化
    };
    ```

    教学中仍应理解传统的“声明与初始化分离”方式，因为它更通用，也更清楚地体现了编译/链接的边界。

### 静态数据成员的访问

#### 直接访问静态数据成员

静态数据成员可以通过**类名**或**对象**来访问：

!!! example "访问静态数据成员"

    ``` cpp linenums="1" hl_lines="18 23 24"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) { ++count; }
        ~Point() { --count; }

        static int count;   // 声明

    private:
        int x, y;
    };

    int Point::count = 0;   // 初始化

    int main() {
        cout << Point::count << endl;   // 0（通过类名访问）

        Point p1(3, 4);
        Point p2(5, 6);

        cout << Point::count << endl;   // 2（通过类名访问）
        cout << p1.count << endl;       // 2（通过对象访问，语法允许但不推荐）

        return 0;
    }
    ```

!!! tip "推荐的访问方式"

    静态成员优先使用**类名加作用域运算符**访问（`Point::count`），这样清晰表达“这是属于类的信息”，而不是属于某个对象的信息。

#### 通过普通成员函数访问静态数据成员

我们先来看看普通（非静态）成员函数能否胜任访问静态数据成员的任务。

!!! question "思考：普通成员函数能否访问静态数据成员？"

    答案是**可以**。普通成员函数可以访问静态数据成员，因为静态数据成员属于类，可以被该类的所有对象所共享，而普通成员函数也是类的一部分。

    但这会带来一个**关键问题**：要调用普通成员函数，必须先创建一个对象。如果还没有任何对象存在，我们就无法通过普通成员函数来访问或显示静态数据成员的值。

!!! example "普通成员函数访问静态数据成员"

    ``` cpp linenums="1" hl_lines="9-12 26-27 30 33"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) { ++count; }
        ~Point() { --count; }

        // 普通成员函数：可以访问静态数据成员
        void showCount() const {
            cout << "Current count: " << count << endl;
        }

        void showPoint() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
        static int count;
    };

    int Point::count = 0;

    int main() {
        // 还没有创建任何 Point 对象
        // Point::showCount();   // 错误！showCount() 是普通成员函数，必需通过具体的对象才能调用

        Point p1(3, 4);
        p1.showCount();   // 必须通过对象调用：Current count: 1

        Point p2(5, 6);
        p2.showCount();   // Current count: 2

        return 0;
    }
    ```

!!! danger "普通成员函数访问静态数据成员的问题"

    - **需要先有对象**：普通成员函数必须通过对象（或对象指针、引用）来调用。在没有任何对象存在时，无法访问静态数据成员的值。
      虽然而静态数据成员在没有任何对象时就已经存在。想要在对象创建之前访问类级别的信息，普通成员函数无能为力。
    - **语义不匹配**：`showCount()` 表达的是类级别的信息，却需要通过某个具体对象来调用，这给阅读者带来困惑——`count` 到底属于对象还是属于类？
      **访问类级别信息的操作，本身也应该属于类级别**，而不应该依赖于某个具体对象。

    ``` cpp linenums="1"
    int main() {
        // 场景：程序刚启动，还没有创建任何 Point 对象
        // 但我们想显示 "当前有 0 个 Point 对象"

        // 做不到！因为必须有一个对象才能调用 showCount()
        // Point p;            // 为了调用 showCount() 而创建一个无意义的对象
        // p.showCount();      // 显示 1，而不是 0！（因为创建了一个对象）

        // 矛盾：为了显示初始计数，反而改变了计数本身

        return 0;
    }
    ```

!!! info "解决方案：静态成员函数"

    正是为了解决这个问题，C++提供了**静态成员函数（Static Member Function）**：

    - 静态成员函数**不依赖于任何对象**，可以直接通过类名调用。
    - 即使还没有创建任何对象，也可以通过 `Point::showCount()` 来访问静态数据成员。
    - 静态成员函数清晰地表达了“这个操作属于类，而不是属于某个对象”的设计意图。

## 静态成员函数

### 基本概念

静态成员函数（Static Member Function）是**属于类**的成员函数，它不与任何具体对象绑定。

!!! info "静态成员函数的特点"

    - **只能访问静态成员**：可以直接访问静态数据成员和其他静态成员函数。
    - **访问非静态成员**：需要显式传入对象或对象引用。
    - **调用方式**：可以通过类名调用（推荐），也可以通过对象调用（语法允许）。

### 定义与使用

!!! example "为Point类添加静态成员函数"

    ``` cpp linenums="1" hl_lines="6-9 11-14 24 29 32"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) { ++count; }
        ~Point() { --count; }

        // 静态成员函数：返回当前对象数量
        static int getCount() {
            return count;   // 可以访问静态数据成员
        }

        // 静态成员函数：显示统计信息
        static void showCount() {
            cout << "Current Point objects: " << count << endl;
        }

    private:
        int x, y;
        static int count;
    };

    int Point::count = 0;

    int main() {
        Point::showCount();   // Current Point objects: 0

        Point p1(3, 4);
        Point p2(5, 6);

        cout << Point::getCount() << endl;   // 2

        Point p3(7, 8);
        Point::showCount();   // Current Point objects: 3

        return 0;
    }
    ```

### 静态成员函数的限制

静态成员函数**不能直接访问非静态成员**，因为它不属于任何一个对象（没有`this`指针），因而也不知道要操作哪个对象的数据。

!!! warning "为什么不能访问非静态成员？"

    ``` cpp linenums="1"
    class Point {
    public:
        static void badFunction() {
            // x = 10;        // 错误！不能直接访问非静态成员 x
            // move(1, 1);    // 错误！不能直接调用非静态成员函数 move()
        }

        static void goodFunction(const Point& p) {
            cout << p.x << endl;   // 可以：显式传入对象后访问其私有成员
        }

    private:
        int x, y;
        static int count;
    };
    ```

!!! info "理解本质"

    - 静态成员函数属于**类**，而不是属于某个对象。
    - 非静态成员变量属于**对象**，每个对象有自己的值。
    - 在不知道是哪个对象，或还没有创建任何对象的情况下，无法访问属于对象的信息。

### 对象调用 vs 类名调用

静态成员函数既可以通过**类名**调用，也可以通过**对象**调用，但语义不同。

!!! example "两种调用方式的对比"

    ``` cpp linenums="1"
    int main() {
        Point p1(3, 4);
        Point p2(5, 6);

        // 方式一：通过类名调用（推荐）
        cout << Point::getCount() << endl;   // 清晰表达：这是 Point 类的信息

        // 方式二：通过对象调用（语法允许，但不推荐）
        cout << p1.getCount() << endl;       // 容易误以为 count 属于对象 p1

        return 0;
    }
    ```

!!! tip "最佳实践"

    优先使用**类名加作用域运算符**（`Point::getCount()`）调用静态成员函数。虽然通过对象调用语法上也能通过，但容易让阅读者产生误解。

## Point计数案例：完整演示

!!! example "完整的Point类计数演示"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        // 构造函数：计数加1
        Point(int x = 0, int y = 0) : x(x), y(y) {
            ++count;
            cout << "Point created. Count: " << count << endl;
        }

        // 拷贝构造函数：计数加1
        Point(const Point& other) : x(other.x), y(other.y) {
            ++count;
            cout << "Point copied. Count: " << count << endl;
        }

        // 析构函数：计数减1
        ~Point() {
            --count;
            cout << "Point destroyed. Count: " << count << endl;
        }

        // 静态成员函数：获取当前对象数量
        static int getCount() {
            return count;
        }

        // 静态成员函数：显示统计信息
        static void showCount() {
            cout << "=== Current Point objects: " << count << " ===" << endl;
        }

        // 普通成员函数：移动点
        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        // 普通成员函数：显示点信息
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
        static int count;   // 静态数据成员声明
    };

    // 静态数据成员初始化
    int Point::count = 0;

    // 测试函数：创建临时对象
    void testFunction() {
        cout << "Entering testFunction..." << endl;
        Point temp(10, 20);
        Point::showCount();
        cout << "Exiting testFunction..." << endl;
    }

    int main() {
        Point::showCount();   // 0

        Point p1(3, 4);       // 创建 p1，count = 1
        Point p2(5, 6);       // 创建 p2，count = 2

        p1.show();
        p2.show();

        Point p3 = p2;        // 拷贝构造，count = 3
        p3.show();

        cout << "Total points: " << Point::getCount() << endl;   // 3

        testFunction();       // 进入函数创建temp，离开函数temp被销毁

        Point::showCount();   // 应回到 3

        return 0;
    }
    ```

    运行结果

    ```
    === Current Point objects: 0 ===
    Point created. Count: 1
    Point created. Count: 2
    Point(3, 4)
    Point(5, 6)
    Point copied. Count: 3
    Point(5, 6)
    Total points: 3
    Entering testFunction...
    Point created. Count: 4
    === Current Point objects: 4 ===
    Exiting testFunction...
    Point destroyed. Count: 3
    === Current Point objects: 3 ===
    Point destroyed. Count: 2
    Point destroyed. Count: 1
    Point destroyed. Count: 0
    ```

!!! note "执行过程分析"

    | 步骤 | 操作                       | count值 |
    | :--- | :------------------------- | :-----: |
    | 1    | 程序开始，初始化静态成员   |    0    |
    | 2    | 构造 p1                    |    1    |
    | 3    | 构造 p2                    |    2    |
    | 4    | 拷贝构造 p3（来自p2）      |    3    |
    | 5    | 进入testFunction，构造temp |    4    |
    | 6    | 离开testFunction，析构temp |    3    |
    | 7    | main结束，析构 p3          |    2    |
    | 8    | 析构 p2                    |    1    |
    | 9    | 析构 p1                    |    0    |

## 静态成员的设计意图

!!! summary "静态成员解决了什么问题？"

    - **共享类级别信息**：如对象计数、类型ID、全局配置等。
    - **避免全局变量污染**：把原本可能放在全局区域的数据，放入类的作用域内。
    - **表达设计意图**：`Point::count` 比全局的 `int pointCount` 更清晰地表达了“这个计数器属于Point类型”。
    - **封装性**：可以把计数器的访问控制放在类内部，防止外部随意修改。

!!! tip "何时使用静态成员？"

    - 数据或操作**不属于任何一个具体对象**，而是属于**整个类型**。
    - 需要在类的**所有对象之间共享**数据。
    - 需要提供**类级别的辅助功能**（如工厂方法、工具函数等）。

!!! question "思考：静态成员与全局变量有什么区别？"

    - **作用域**：静态成员在类作用域内，受访问控制约束；全局变量在全局作用域，任何地方都能访问。
    - **语义**：静态成员表达了“属于这个类”的设计意图；全局变量没有归属关系。
    - **可维护性**：静态成员与类的其他成员集中管理；全局变量散落在各处，难以追踪。

## 小结

1. **静态数据成员**由类的所有对象共享，只有一份副本，需要在类外定义。
2. **静态成员函数**属于类而非对象，没有`this`指针，只能直接访问静态成员。
3. 静态成员通过**类名加作用域运算符**访问（`Point::count`）最清晰。
4. 静态成员是类级别的共享机制，比全局变量更安全、更有组织、更易维护。

将数据和行为归类管理，是面向对象程序设计的基本思路。静态成员让类不仅能描述对象的行为，也能承载类型自身的状态和职责。