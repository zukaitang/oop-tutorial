# 动态内存管理

C语言使用 `malloc/free` 进行堆内存管理，而C++提供了更安全、更简洁的 `new/delete` 操作符，
能够在运行时按需创建和销毁对象，是C++动态内存管理的核心机制。
理解两者的区别，是编写健壮C++程序的基础。

## 为什么需要动态内存分配

### 栈内存与堆内存

在C/C++程序中，内存主要分为**栈（Stack）** 和**堆（Heap）** 两个区域：

!!! info "栈内存与堆内存的对比"

    | 对比项       | 栈（Stack）      | 堆（Heap）             |
    | :----------- | :--------------- | :--------------------- |
    | **分配方式** | 编译器自动分配   | 程序员手动分配         |
    | **分配速度** | 快               | 较慢                   |
    | **生命周期** | 随作用域自动销毁 | 程序员控制             |
    | **大小限制** | 有限（通常几MB） | 较大（取决于系统内存） |
    | **内存碎片** | 无               | 可能产生               |

!!! example "栈内存的自动管理"

    ``` cpp linenums="1"
    void function() {
        int x = 10;        // 栈上分配
        double arr[100];   // 栈上分配
        // x 和 arr 在函数返回时自动释放
    }
    ```

### 用堆内存进行动态分配

在程序设计中，很多时候无法预先知道需要创建多少个对象，也无法确定数组的大小。静态分配（编译时确定大小）无法满足这些需求。

!!! example "场景一：根据用户输入决定数组大小"

    ``` cpp linenums="1"
    int n;
    cout << "请输入学生人数: ";
    cin >> n;
    // int scores[n];   // 错误！C++中数组大小必须是编译时常量
    ```

!!! example "场景二：对象的生存期需要超出作用域"

    ``` cpp linenums="1"
    // 需要在函数返回后对象仍然存在
    Point* createPoint() {
        Point p(3, 4);   // 局部对象，函数返回后被销毁
        return &p;       // 危险！返回的是已销毁对象的地址
    }
    ```

!!! info "动态内存分配的解决方案"

    动态内存分配允许程序在**运行时**根据需要申请和释放内存，具有以下优势：

    - **按需分配**：根据程序运行时的实际需要申请内存。
    - **生存期可控**：动态对象的生存期由程序员控制，可以超越创建它的作用域。
    - **灵活调整**：可以根据程序状态动态调整内存使用。

!!! tip "动态内存的特点"

    - **分配时机**：程序运行时，而非编译时。
    - **分配位置**：堆（Heap）内存，而非栈（Stack）。
    - **管理方式**：需要程序员显式地申请和释放。
    - **生存期**：从分配到释放，由程序员控制。

## C语言的内存管理

### C语言的内存管理函数

C语言通过 `<stdlib.h>` 中的 `malloc`、`calloc`、`realloc` 和 `free` 函数进行动态内存管理。

!!! example "C风格的内存管理"

    ``` c linenums="1"
    #include <stdio.h>
    #include <stdlib.h>

    int main() {
        // 1. malloc：分配未初始化的内存
        int* p1 = (int*)malloc(sizeof(int) * 10);
        if (p1 == NULL) {
            printf("Memory allocation failed!\n");
            return 1;
        }

        // 2. calloc：分配并初始化为 0
        int* p2 = (int*)calloc(10, sizeof(int));

        // 3. realloc：重新调整内存大小
        int* p3 = (int*)realloc(p1, sizeof(int) * 20);
        if (p3 != NULL) {
            p1 = p3;
        }

        // 使用内存...
        for (int i = 0; i < 10; i++) {
            p1[i] = i;
        }

        // 4. free：释放内存
        free(p1);
        free(p2);

        return 0;
    }
    ```

### C语言内存管理的痛点

!!! danger "C风格内存管理的主要问题"

    | 问题                | 说明                                | 示例                       |
    | :------------------ | :---------------------------------- | :------------------------- |
    | **类型不安全**      | `malloc` 返回 `void*`，需要强制转换 | `(int*)malloc(...)`        |
    | **手动计算大小**    | 需要 `sizeof` 手动计算字节数        | `malloc(sizeof(int) * 10)` |
    | **忘记释放**        | 导致内存泄漏                        | 没有对应的 `free`          |
    | **重复释放**        | 导致程序崩溃                        | 对同一指针多次 `free`      |
    | **释放后使用**      | 访问已释放的内存                    | `free(p); *p = 10;`        |
    | **不调用构造/析构** | 对自定义类型无效                    | 不能用于C++对象            |

## C++的 new 和 delete 操作符

### 基本语法

C++ 使用 `new` 操作符动态申请内存，其返回值是对应类型的指针。
使用 `delete` 操作符释放内存，其操作数也必须是一个指针类型。
它们比 `malloc/free` 更简洁、更安全。

!!! abstract "new 和 delete 的基本语法"

    ``` cpp
    // 分配单个对象
    类型* 指针名 = new 类型(初始化值);
    delete 指针名;

    // 分配数组
    类型* 指针名 = new 类型[元素个数];
    delete[] 指针名;
    ```

!!! example "基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        // 1. 分配单个 int
        int* p1 = new int;          // 分配未初始化的 int
        int* p2 = new int(10);      // 分配并初始化为 10
        int* p3 = new int();        // 分配并初始化为 0

        cout << "*p1 = " << *p1 << endl;   // 未初始化，值不确定
        cout << "*p2 = " << *p2 << endl;   // 10
        cout << "*p3 = " << *p3 << endl;   // 0

        // 释放单个对象
        delete p1;
        delete p2;
        delete p3;

        // 2. 分配数组
        int* arr = new int[5];      // 分配 5 个 int（未初始化）
        int* arr2 = new int[5]();   // 分配 5 个 int（全部初始化为 0）

        for (int i = 0; i < 5; i++) {
            arr[i] = i * 10;
        }

        // 释放数组（注意 delete[]）
        delete[] arr;
        delete[] arr2;

        return 0;
    }
    ```

### new 的不同形式

!!! info "new 的几种形式"

    | 形式           | 含义                               | 示例                      |
    | :------------- | :--------------------------------- | :------------------------ |
    | `new T`        | 分配 T 类型对象，不初始化          | `new int`                 |
    | `new T()`      | 分配 T 类型对象，值初始化          | `new int()`（初始化为 0） |
    | `new T(value)` | 分配 T 类型对象，用 value 初始化   | `new int(10)`             |
    | `new T[n]`     | 分配 n 个 T 类型对象的数组         | `new int[10]`             |
    | `new T[n]()`   | 分配 n 个 T 类型对象数组，值初始化 | `new int[10]()`           |

!!! warning "new 和 delete 的关键规则"

    - **配对使用**：用 `new` 分配的内存，必须用 `delete` 释放。
    - **数组释放**：用 `new[]` 分配的数组，必须用 `delete[]` 释放。
    - **不可重复释放**：同一块内存不能释放两次（会导致未定义行为）。
    - **空指针释放安全**：`delete nullptr;` 是安全的（不执行任何操作）。
    - **不要混用**：`new` 配 `delete`，`new[]` 配 `delete[]`，不能混用。

### 对象的动态创建与释放

在面向对象语言中，动态内存分配最常见的用途是动态创建对象。
`new` 在分配内存的同时会**调用对象的构造函数**，`delete` 会**调用析构函数**。这是 `new/delete` 与 `malloc/free` 最核心的区别。

!!! example "动态创建和释放对象"

    ``` cpp linenums="1" hl_lines="24 29"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            cout << "Constructor called: Point(" << x << ", " << y << ")" << endl;
        }

        ~Point() {
            cout << "Destructor called" << endl;
        }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        cout << "--- 动态创建单个对象 ---" << endl;
        Point* p1 = new Point(3, 4);    // 分配内存 + 调用构造函数
        p1->show();
        delete p1;                       // 调用析构函数 + 释放内存

        cout << "--- 动态创建对象数组 ---" << endl;
        Point* arr = new Point[3];       // 每个元素调用默认构造函数
        arr[0].show();
        arr[1].show();
        arr[2].show();
        delete[] arr;                    // 每个元素调用析构函数

        return 0;
    }
    ```

    运行结果：

    ```
    --- 动态创建单个对象 ---
    Constructor called: Point(3, 4)
    Point(3, 4)
    Destructor called
    --- 动态创建对象数组 ---
    Constructor called: Point(0, 0)
    Constructor called: Point(0, 0)
    Constructor called: Point(0, 0)
    Point(0, 0)
    Point(0, 0)
    Point(0, 0)
    Destructor called
    Destructor called
    Destructor called
    ```

!!! info "new 和 delete 与构造函数/析构函数的对应关系"

    | 操作          | 行为                                  |
    | :------------ | :------------------------------------ |
    | `new T`       | 分配内存 → 调用默认构造函数 `T()`     |
    | `new T(args)` | 分配内存 → 调用构造函数 `T(args)`     |
    | `delete p`    | 调用析构函数 `~T()` → 释放内存        |
    | `new T[n]`    | 分配内存 → 对每个元素调用默认构造函数 |
    | `delete[] p`  | 对每个元素调用析构函数 → 释放内存     |

## new/delete vs malloc/free

!!! summary "全面对比"

    | 对比项         | `malloc` / `free`      | `new` / `delete`           |
    | :------------- | :--------------------- | :------------------------- |
    | **来源**       | C标准库（`<cstdlib>`） | C++语言操作符              |
    | **内存大小**   | 需手动计算（`sizeof`） | 自动计算                   |
    | **返回类型**   | `void*`（需强制转换）  | `T*`（正确类型）           |
    | **初始化**     | 只分配，不初始化       | 可初始化（调用构造函数）   |
    | **构造/析构**  | 不调用                 | **调用**                   |
    | **失败处理**   | 返回 `NULL`            | 抛出 `std::bad_alloc` 异常 |
    | **重新分配**   | 支持 `realloc`         | 不支持（需手动实现）       |
    | **是否操作符** | 函数                   | 操作符（可重载）           |
    | **使用场景**   | C兼容、纯内存分配      | C++对象分配                |

!!! example "对比示例"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cstdlib>
    using namespace std;

    class Student {
    private:
        string name;
        int age;
    public:
        Student(const string& n, int a) : name(n), age(a) {
            cout << "Student constructed: " << name << endl;
        }
        ~Student() {
            cout << "Student destroyed: " << name << endl;
        }
    };

    int main() {
        // C风格：不会调用构造/析构
        cout << "=== C style (malloc/free) ===" << endl;
        Student* s1 = (Student*)malloc(sizeof(Student));
        // 内存未初始化，不能使用
        free(s1);   // 不会调用析构函数

        // C++风格：调用构造/析构
        cout << "\n=== C++ style (new/delete) ===" << endl;
        Student* s2 = new Student("Alice", 20);
        delete s2;   // 调用析构函数

        // 数组分配对比
        cout << "\n=== Array allocation ===" << endl;
        // C风格：分配并释放
        int* arr1 = (int*)malloc(sizeof(int) * 10);
        free(arr1);

        // C++风格
        int* arr2 = new int[10];
        delete[] arr2;

        return 0;
    }
    ```

    运行结果：

    ```
    === C style (malloc/free) ===

    === C++ style (new/delete) ===
    Student constructed: Alice
    Student destroyed: Alice

    === Array allocation ===
    ```

## 内存分配失败的处理

### C语言风格：返回 NULL

``` c
int* p = (int*)malloc(sizeof(int) * 1000);
if (p == NULL) {
    // 处理分配失败
}
```

### C++风格：抛出异常

C++中，当 `new` 分配失败时，会抛出 `std::bad_alloc` 异常。

!!! example "处理 bad_alloc 异常"

    ``` cpp linenums="1"
    #include <iostream>
    #include <new>
    using namespace std;

    int main() {
        try {
            // 尝试分配大量内存
            int* p = new int[1000000000];
            // 使用 p...
            delete[] p;
        } catch (const bad_alloc& e) {
            cout << "Memory allocation failed: " << e.what() << endl;
        }

        return 0;
    }
    ```

### nothrow 版本

如果希望 `new` 失败时返回 `NULL` 而不是抛出异常，可以使用 `nothrow` 版本。

!!! example "nothrow 版本的 new"

    ``` cpp linenums="1"
    #include <iostream>
    #include <new>
    using namespace std;

    int main() {
        // nothrow 版本：失败返回 nullptr
        int* p = new (nothrow) int[1000000000];

        if (p == nullptr) {
            cout << "Memory allocation failed" << endl;
        } else {
            delete[] p;
        }

        return 0;
    }
    ```

## 动态创建对象数组

当需要在运行时确定数组大小时，动态创建对象数组是非常有用的方式。

!!! example "动态创建对象数组的完整示例"

    ``` cpp linenums="1" hl_lines="35 48"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point() : x(0), y(0) {
            cout << "Default Constructor called." << endl;
        }
        Point(int x, int y) : x(x), y(y) {
            cout << "Constructor called: Point(" << x << ", " << y << ")" << endl;
        }
        ~Point() {
            cout << "Destructor called." << endl;
        }

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        int count;
        cout << "Please enter the count of points: ";
        cin >> count;

        // 动态创建对象数组
        Point* points = new Point[count];

        // 通过指针访问数组元素
        for (int i = 0; i < count; i++) {
            points[i].move(i * 5, i * 10);
        }

        for (int i = 0; i < count; i++) {
            cout << "Point " << i << ": ";
            points[i].show();
        }

        // 释放对象数组
        delete[] points;

        return 0;
    }
    ```

!!! info "动态创建对象数组的注意事项"

    - 动态创建对象数组时，对象必须具有**可访问的默认构造函数**（无参构造函数）。
    - 数组元素按**从低地址到高地址**的顺序构造。
    - 数组元素按**从高地址到低地址**的逆序析构。
    - 释放数组时必须使用 `delete[]`，而不是 `delete`（后者只会释放第一个元素，导致内存泄漏）。

### 动态创建多维数组

`new` 也可以用于创建多维动态数组，但语法相对复杂，涉及到指针数组与数组指针等概念。

!!! example "动态创建二维数组"

    ``` cpp linenums="1" hl_lines="10 13 27 29 31"
    #include <iostream>
    using namespace std;

    int main() {
        int rows, cols;
        cout << "Enter rows and cols: ";
        cin >> rows >> cols;

        // 方法一：使用 new 创建二维数组（只有第一维可以是变量）
        int (*matrix)[4] = new int[rows][4];   // 第二维必须是常量

        // 方法二：使用指针数组（两维都可以是变量）
        int** matrix2 = new int*[rows];
        for (int i = 0; i < rows; i++) {
            matrix2[i] = new int[cols];
        }

        // 使用矩阵
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                matrix2[i][j] = i * cols + j;
            }
        }

        // 释放矩阵（二维释放需要循环）
        for (int i = 0; i < rows; i++) {
            delete[] matrix2[i];
        }
        delete[] matrix2;

        delete[] matrix;

        return 0;
    }
    ```

## 常见错误与最佳实践

### 常见错误

!!! danger "动态内存管理的常见错误"

    | 错误类型               | 错误示例                              | 后果                     |
    | :--------------------- | :------------------------------------ | :----------------------- |
    | **内存泄漏**           | `new` 后没有 `delete`                 | 内存无法回收             |
    | **重复释放**           | 对同一指针多次 `delete`               | 程序崩溃                 |
    | **释放后使用**         | `delete p; *p = 10;`                  | 未定义行为               |
    | **混用分配方式**       | `new` 配 `free`，`malloc` 配 `delete` | 未定义行为               |
    | **数组释放错误**       | `new[]` 配 `delete`（少 `[]`）        | 仅释放首元素，未定义行为 |
    | **忘记置空**           | 释放后未置 `nullptr`                  | 可能被重复使用           |
    | **内存分配失败未处理** | 未检查 `new` 是否成功                 | 程序崩溃                 |

!!! example "错误的后果"

    ``` cpp
    // 1. 内存泄漏
    void leakyFunction() {
        int* p = new int(10);
        // 忘记 delete p; → p 指向的内存无法释放
    }

    // 2. 重复释放
    void doubleFree() {
        int* p = new int(10);
        delete p;
        delete p;   // 危险！重复释放
    }

    // 3. 释放后使用
    void useAfterFree() {
        int* p = new int(10);
        delete p;
        *p = 20;    // 危险！访问已释放的内存
    }

    // 4. 混用 new/free（错误）
    void mixedAllocation() {
        int* p = new int(10);
        free(p);    // 危险！行为未定义
    }

    // 5. 数组释放错误
    void wrongArrayDelete() {
        int* p = new int[10];
        delete p;   // 错误！应该用 delete[]
    }
    ```

### 最佳实践

!!! success "动态内存管理的最佳实践"

    1. **使用 `new`/`delete` 替代 `malloc`/`free`**：类型安全、自动调用构造/析构。

    2. **配对使用**：`new` 配 `delete`，`new[]` 配 `delete[]`。

    3. **释放后置空**：`delete p; p = nullptr;` 防止重复释放。

    4. **使用智能指针**（`unique_ptr`、`shared_ptr`）：自动管理内存，避免手动 `delete`。

    5. **优先使用容器**：`vector`、`string` 等容器自动管理内存。

    6. **检查分配结果**：处理 `bad_alloc` 异常或使用 `nothrow` 版本。

## 智能指针

虽然手动管理内存可以精确控制资源的分配和释放，但也容易出错（内存泄漏、重复释放等）。C++11 提供了**智能指针**来自动管理动态内存，实现一定程度的内存自动管理。

!!! warning "手动内存管理的常见问题"

    - **内存泄漏**：忘记调用 `delete`，导致内存无法被回收。
    - **重复释放**：对同一块内存调用多次 `delete`，导致程序崩溃。
    - **野指针**：释放后未将指针置空，继续使用已释放的内存。
    - **异常安全问题**：发生异常时，`delete` 可能不会被执行。

!!! info "C++11 中的智能指针"

    | 智能指针         | 头文件     | 特点                   | 适用场景                         |
    | :--------------- | :--------- | :--------------------- | :------------------------------- |
    | **`unique_ptr`** | `<memory>` | 独占所有权，不允许复制 | 确保资源只有一个所有者           |
    | **`shared_ptr`** | `<memory>` | 共享所有权，引用计数   | 多个对象共享同一资源             |
    | **`weak_ptr`**   | `<memory>` | 弱引用，不增加引用计数 | 解决 `shared_ptr` 的循环引用问题 |

### unique_ptr

`unique_ptr` 独占所管理的对象，不允许复制，但可以转移所有权。

!!! example "unique_ptr 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <memory>   // 智能指针头文件
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            cout << "Point constructed" << endl;
        }
        ~Point() {
            cout << "Point destroyed" << endl;
        }
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }
        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }
    private:
        int x, y;
    };

    int main() {
        // 创建 unique_ptr
        unique_ptr<Point> p1(new Point(3, 4));
        p1->show();

        // 也可以使用 make_unique（C++14）
        auto p2 = make_unique<Point>(5, 6);
        p2->show();

        // p1 不能复制，但可以转移所有权
        // unique_ptr<Point> p3 = p1;   // 错误！不能复制

        unique_ptr<Point> p3 = move(p1);   // 所有权转移
        if (p1 == nullptr) {
            cout << "p1 is now empty" << endl;
        }
        p3->show();

        // 离开作用域时自动释放
        return 0;
    }
    ```

### shared_ptr

`shared_ptr` 使用引用计数来共享所有权。当最后一个 `shared_ptr` 被销毁时，所管理的对象才会被释放。

!!! example "shared_ptr 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <memory>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            cout << "Point constructed" << endl;
        }
        ~Point() {
            cout << "Point destroyed" << endl;
        }
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }
    private:
        int x, y;
    };

    int main() {
        // 创建 shared_ptr
        shared_ptr<Point> sp1 = make_shared<Point>(3, 4);
        cout << "Reference count: " << sp1.use_count() << endl;   // 1

        {
            shared_ptr<Point> sp2 = sp1;   // 共享所有权，引用计数增加
            cout << "Reference count: " << sp1.use_count() << endl;   // 2
            sp2->show();
        }   // sp2 离开作用域，引用计数减少

        cout << "Reference count: " << sp1.use_count() << endl;   // 1

        // sp1 离开作用域，引用计数变为 0，对象被销毁
        return 0;
    }
    ```

### weak_ptr

`weak_ptr` 是一种弱引用，它不增加引用计数，主要用于解决 `shared_ptr` 的循环引用问题。

!!! example "weak_ptr 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <memory>
    using namespace std;

    int main() {
        shared_ptr<int> sp = make_shared<int>(10);
        weak_ptr<int> wp = sp;   // 弱引用，不增加引用计数

        cout << "Reference count: " << sp.use_count() << endl;   // 1

        // 使用 weak_ptr 前需要 lock() 获取 shared_ptr
        if (auto temp = wp.lock()) {
            cout << "Value: " << *temp << endl;   // 10
        }

        sp.reset();   // 释放 shared_ptr

        if (wp.expired()) {
            cout << "Weak pointer expired" << endl;   // 对象已被释放
        }

        return 0;
    }
    ```

!!! tip "使用建议"

    - **优先使用智能指针**：在大多数场景下，智能指针比原始指针更安全、更方便。
    - **`unique_ptr` 是首选**：除非确实需要共享所有权，否则优先使用 `unique_ptr`。
    - **使用 `make_shared` 和 `make_unique`**：比直接使用 `new` 更安全、更高效。
    - **避免手动 `delete`**：使用智能指针后，通常不需要再手动调用 `delete`。

## 综合示例

!!! example "动态内存分配"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    class Person {
    private:
        string name;
        int age;
    public:
        Person(const string& n = "", int a = 0) : name(n), age(a) {
            cout << "Person constructed: " << name << endl;
        }
        ~Person() {
            cout << "Person destroyed: " << name << endl;
        }

        void introduce() const {
            cout << "I'm " << name << ", " << age << " years old." << endl;
        }
    };

    int main() {
        cout << "=== 1. 单个对象的分配和释放 ===" << endl;
        Person* p1 = new Person("Alice", 25);
        p1->introduce();
        delete p1;
        p1 = nullptr;

        cout << "\n=== 2. 对象数组的分配和释放 ===" << endl;
        Person* arr = new Person[3] {
            Person("Bob", 20),
            Person("Charlie", 22),
            Person("David", 24)
        };

        for (int i = 0; i < 3; i++) {
            arr[i].introduce();
        }
        delete[] arr;

        cout << "\n=== 3. 基本类型的分配 ===" << endl;
        // 基本类型初始化
        int* n1 = new int;        // 未初始化
        int* n2 = new int(100);   // 初始化为 100
        int* n3 = new int();      // 初始化为 0

        cout << "*n1 = " << *n1 << " (未初始化)" << endl;
        cout << "*n2 = " << *n2 << endl;
        cout << "*n3 = " << *n3 << endl;

        delete n1;
        delete n2;
        delete n3;

        cout << "\n=== 4. 内存分配失败处理 ===" << endl;
        try {
            // 尝试分配大量内存
            int* huge = new int[1000000000];
            cout << "Memory allocated successfully" << endl;
            delete[] huge;
        } catch (const bad_alloc& e) {
            cout << "Allocation failed: " << e.what() << endl;
        }

        cout << "\n=== 5. 容器自动管理内存（推荐） ===" << endl;
        // 使用 vector 自动管理内存，无需手动 delete
        vector<Person> people;
        people.push_back(Person("Eve", 30));
        people.push_back(Person("Frank", 28));
        for (const auto& p : people) {
            p.introduce();
        }
        // vector 在析构时自动释放内存

        return 0;
    }
    ```

    运行结果

    ```
    === 1. 单个对象的分配和释放 ===
    Person constructed: Alice
    I'm Alice, 25 years old.
    Person destroyed: Alice

    === 2. 对象数组的分配和释放 ===
    Person constructed: Bob
    Person constructed: Charlie
    Person constructed: David
    I'm Bob, 20 years old.
    I'm Charlie, 22 years old.
    I'm David, 24 years old.
    Person destroyed: David
    Person destroyed: Charlie
    Person destroyed: Bob

    === 3. 基本类型的分配 ===
    *n1 = 0 (未初始化)
    *n2 = 100
    *n3 = 0

    === 4. 内存分配失败处理 ===
    Allocation failed: std::bad_alloc

    === 5. 容器自动管理内存（推荐） ===
    Person constructed: Eve
    Person constructed: Frank
    I'm Eve, 30 years old.
    I'm Frank, 28 years old.
    Person destroyed: Frank
    Person destroyed: Eve
    ```

## 小结

1.  **动态内存分配**使用 `new` 和 `delete` 在运行时管理内存，灵活但需要谨慎。

2.  **new 操作符**：
    - 在堆上分配内存。
    - 自动调用对象的构造函数。
    - 返回指向分配内存的指针。

3.  **delete 操作符**：
    - 调用对象的析构函数。
    - 释放堆上的内存。
    - 必须与 `new` 配对使用。

4.  **动态数组**：
    - 使用 `new[]` 和 `delete[]`。
    - 元素必须具有默认构造函数。
    - 构造顺序与析构顺序相反。

5.  **错误处理**：
    - C++ 中 `new` 失败抛出 `std::bad_alloc`。
    - 可使用 `new (nothrow)` 返回 `nullptr`。

6.  **智能指针**（C++11）：
    - `unique_ptr`：独占所有权。
    - `shared_ptr`：共享所有权，引用计数。
    - `weak_ptr`：弱引用，解决循环引用。

7.  **最佳实践**：
    - 优先使用 `new`/`delete` 而非 `malloc`/`free`。
    - 优先使用智能指针，避免手动管理动态内存，减少内存管理错误。