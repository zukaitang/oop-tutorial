# 动态内存分配

动态内存分配让程序能够在运行时按需创建和销毁对象，是C++灵活管理内存的核心机制。

## 为什么需要动态内存分配

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
    - **管理方式**：需要程序员显式地申请（`new`）和释放（`delete`）。
    - **生存期**：从 `new` 分配到 `delete` 释放，由程序员控制。

## new 和 delete 操作符

C++ 使用 `new` 操作符动态申请内存，其返回值是对应类型的指针。
使用 `delete` 操作符释放内存，其操作数也必须是一个指针类型。

### 基础数据的动态创建与释放

!!! example "new 和 delete 的基本用法"

    ``` cpp linenums="1"
    // 申请和释放单个变量
    int* p1 = new int;          // 申请一个 int 空间（未初始化，内容不确定）
    int* p2 = new int(10);      // 申请一个 int 空间，初始化为 10
    int* p3 = new int();        // 申请一个 int 空间，初始化为 0

    delete p1;                  // 释放 p1 指向的内存
    delete p2;
    delete p3;

    // 申请和释放数组
    int* arr = new int[100];    // 申请 100 个 int 的连续空间（未初始化）
    int* arr2 = new int[100](); // 申请 100 个 int，全部初始化为 0

    delete[] arr;               // 释放数组空间（注意 []）
    delete[] arr2;
    ```

!!! info "new 的几种形式"

    | 语法           | 说明                                                |
    | :------------- | :-------------------------------------------------- |
    | `new T`        | 分配一个 T 类型对象，不初始化                       |
    | `new T()`      | 分配一个 T 类型对象，值初始化（内置类型初始化为 0） |
    | `new T(value)` | 分配一个 T 类型对象，用 value 初始化                |
    | `new T[n]`     | 分配 n 个 T 类型对象的数组，不初始化                |
    | `new T[n]()`   | 分配 n 个 T 类型对象的数组，值初始化                |

!!! warning "new 和 delete 的关键规则"

    - **配对使用**：用 `new` 分配的内存，必须用 `delete` 释放。
    - **数组释放**：用 `new[]` 分配的数组，必须用 `delete[]` 释放。
    - **不可重复释放**：同一块内存不能释放两次（会导致未定义行为）。
    - **空指针释放安全**：`delete nullptr;` 是安全的（不执行任何操作）。
    - **不要混用**：`new` 配 `delete`，`new[]` 配 `delete[]`，不能混用。

### 对象的动态创建与释放

在面向对象语言中，动态内存分配最常见的用途是动态创建对象。
`new` 在分配内存的同时会调用对象的构造函数，`delete` 在释放内存之前会调用对象的析构函数。

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

### 内存分配失败的处理

当内存不足时，`new` 操作符会抛出 `bad_alloc` 异常（而不是返回空指针）。

!!! example "处理内存分配失败"

    ``` cpp linenums="1"
    #include <iostream>
    #include <new>      // 包含 bad_alloc 异常
    using namespace std;

    int main() {
        try {
            // 尝试分配大量内存
            int* p = new int[1000000000];
            // 使用 p...
            delete[] p;
        } catch (const bad_alloc& e) {
            cout << "内存分配失败: " << e.what() << endl;
        }

        // 或者使用 nothrow 版本（返回空指针）
        int* p2 = new (nothrow) int[1000000000];
        if (p2 == nullptr) {
            cout << "内存分配失败" << endl;
        } else {
            delete[] p2;
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

## 动态数组的封装 （TODO)

手动管理动态数组比较繁琐，而且容易出错。一种常见的做法是将动态数组封装成类，提供更安全、更便利的接口。

!!! example "动态数组类的封装"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cassert>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            cout << "Constructor called." << endl;
        }
        ~Point() {
            cout << "Destructor called." << endl;
        }

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        int getX() const { return x; }
        int getY() const { return y; }
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    // 动态数组封装类
    class ArrayOfPoints {
    public:
        // 构造函数：动态分配数组
        ArrayOfPoints(int size) : size(size) {
            points = new Point[size];
        }

        // 析构函数：释放动态数组
        ~ArrayOfPoints() {
            cout << "Deleting array..." << endl;
            delete[] points;
        }

        // 元素访问：返回引用，支持读写；带越界检查
        Point& element(int index) {
            assert(index >= 0 && index < size);
            return points[index];
        }

        // 常版本：用于只读访问
        const Point& element(int index) const {
            assert(index >= 0 && index < size);
            return points[index];
        }

        // 获取数组大小
        int getSize() const { return size; }

        // 重载下标运算符（更自然的访问方式）
        Point& operator[](int index) {
            return element(index);
        }

        const Point& operator[](int index) const {
            return element(index);
        }

    private:
        Point* points;    // 指向动态数组首地址
        int size;         // 数组大小
    };

    int main() {
        int count;
        cout << "Please enter the count of points: ";
        cin >> count;

        ArrayOfPoints arr(count);

        // 使用 element 函数访问
        arr.element(0).move(5, 10);
        arr.element(1).move(15, 20);

        // 使用重载的下标运算符访问
        arr[0].show();    // Point(5, 10)
        arr[1].show();    // Point(15, 20)

        // 越界访问会被 assert 拦截（调试模式下）
        // arr[2].show();  // 断言失败

        cout << "Array size: " << arr.getSize() << endl;

        return 0;
    }
    ```

!!! success "封装的好处"

    - **自动管理内存**：构造函数中分配，析构函数中释放，避免忘记 `delete`。
    - **越界检查**：`element()` 函数使用 `assert` 检查下标是否合法，防止内存越界。
    - **简化使用**：用户不需要关心底层的内存分配细节。
    - **可复用**：封装后的类可以在多个地方安全使用。
    - **更自然的接口**：可以重载 `operator[]` 让访问更自然。

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

## 动态内存分配的常见错误

!!! danger "常见错误及防范"

    | 错误类型               | 错误示例                | 后果       | 正确做法                                 |
    | :--------------------- | :---------------------- | :--------- | :--------------------------------------- |
    | **忘记释放内存**       | `new` 后没有 `delete`   | 内存泄漏   | 使用智能指针或确保配对 `delete`          |
    | **释放顺序错误**       | `delete[]` 用于单个对象 | 未定义行为 | `new` 配 `delete`，`new[]` 配 `delete[]` |
    | **重复释放**           | 对同一指针多次 `delete` | 程序崩溃   | 释放后置空指针，使用智能指针             |
    | **使用已释放的内存**   | 释放后继续使用指针      | 未定义行为 | 释放后置空指针，使用智能指针             |
    | **内存分配失败未处理** | 未检查 `new` 是否成功   | 程序崩溃   | 捕获 `bad_alloc` 异常                    |

## 小结

1. **动态内存分配**使用 `new` 和 `delete` 在运行时管理内存，灵活但需要谨慎。

2. **new 操作符**：
3. 在堆上分配内存。
4. 自动调用对象的构造函数。
5. 返回指向分配内存的指针。

6. **delete 操作符**：
7. 调用对象的析构函数。
8. 释放堆上的内存。
9. 必须与 `new` 配对使用。

10. **动态数组**：
11. 使用 `new[]` 和 `delete[]`。
12. 元素必须具有默认构造函数。
13. 构造顺序与析构顺序相反。

14. **封装动态数组**：将动态数组封装为类，可以简化内存管理，增强安全性。

15. **智能指针**（C++11）：
16. `unique_ptr`：独占所有权。
17. `shared_ptr`：共享所有权，引用计数。
18. `weak_ptr`：弱引用，解决循环引用。

19. **最佳实践**：优先使用智能指针，避免手动管理动态内存，减少内存管理错误。