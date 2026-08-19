# 关键字与类型检查

C++在C语言的基础上引入了多个新的关键字，并增强了类型检查机制。这些改进使得代码更安全、更易于维护，
也体现了C++"类型安全"的设计理念。

## 新增关键字概览

C++新增了许多C语言中没有的关键字，其中与面向过程编程密切相关的包括：

!!! abstract "C++新增的部分关键字"

    | 关键字          | 用途                          | 对应C语言写法    |
    | :-------------- | :---------------------------- | :--------------- |
    | `const`         | 常量修饰符（C++中语义更严格） | `#define`宏      |
    | `constexpr`     | 编译期常量表达式（C++11）     | `#define`宏      |
    | `auto`          | 自动类型推导（C++11）         | 必须显式声明类型 |
    | `decltype`      | 表达式类型推导（C++11）       | 无对应           |
    | `nullptr`       | 类型安全的空指针（C++11）     | `NULL`宏         |
    | `inline`        | 内联函数                      | 函数宏           |
    | `namespace`     | 命名空间                      | 无对应           |
    | `using`         | 类型别名/引入命名空间         | `typedef`        |
    | `static_assert` | 编译期断言（C++11）           | 无对应           |

本章将重点介绍 `const`、`constexpr`、`auto`、`decltype`、`nullptr`，以及C++中更严格的类型检查机制。

## const 关键字的增强

### C语言中的 const

在C语言中，`const` 修饰的变量表示**只读变量**，但本质上仍然是变量。它有以下局限性：

- 不是真正的编译期常量（不能用于数组长度）
- 可以通过指针间接修改（有警告，但能编译）

!!! example "C语言中的 const"

    ``` c linenums="1"
    #include <stdio.h>

    int main() {
        const int size = 10;
        
        // 在C中，const变量不能用于定义数组长度
        // int arr[size];   // 错误！C语言中const不是编译期常量

        // 在C中，const变量可以被指针修改（警告）
        int* p = (int*)&size;
        *p = 20;            // 编译通过，但行为未定义

        printf("%d\n", size);   // 可能输出20（取决于编译器）

        return 0;
    }
    ```

### C++中的 const

C++对 `const` 进行了增强，使其行为更符合直觉：

!!! info "C++中 const 的改进"

    1. **编译期常量**：`const` 变量可用于数组长度等需要编译期常量的场景。
    2. **必须有初值**：`const` 变量必须在声明时初始化。
    3. **默认内部链接**：C++中 `const` 变量默认具有内部链接（相当于 `static`），避免重复定义。
    4. **更严格**：通过指针修改 `const` 变量是**编译错误**。

!!! example "C++中的 const"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        // const变量必须初始化
        const int size = 10;

        // C++中const是真正的编译期常量，可用于数组长度
        int arr[size];      // ✓ 正确！

        // 通过指针修改const变量 → 编译错误
        // int* p = &size;          // 错误！不能将 const int* 转为 int*
        // const_cast<int*>(&size); // 可以强制转换，但修改是未定义行为

        cout << "size = " << size << endl;

        return 0;
    }
    ```

### C++中 const 的推荐用法

在C++中，更常用 `const` 替代 `#define`：

- `const` 有类型检查，`#define` 是文本替换。
- `const` 有作用域控制，`#define` 全局有效。
- `const` 在调试器中可见，`#define` 不可见。

!!! example "用 const 替代 #define"

    ``` cpp
    // C语言风格：使用宏
    #define MAX_SIZE 100
    #define PI 3.14159

    // C++推荐风格：使用 const
    const int MAX_SIZE = 100;
    const double PI = 3.14159;
    ```

## constexpr：编译期常量表达式（C++11）

`constexpr` 是C++11引入的关键字，用于在**编译期**计算表达式的值，比 `const` 更严格。

| 对比项       | `const`                       | `constexpr`                |
| :----------- | :---------------------------- | :------------------------- |
| **求值时机** | 可以是运行时，也可以是编译期  | **必须是编译期**           |
| **初始化**   | 可以在运行时初始化            | 必须在编译期初始化         |
| **用途**     | 表示"只读"                    | 表示"编译期常量"           |
| **函数支持** | 不能修饰函数（函数没有const） | 可以修饰函数（编译期求值） |

!!! example "constexpr 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // constexpr函数：在编译期求值
    constexpr int square(int x) {
        return x * x;
    }

    int main() {
        // constexpr声明的数据必须是编译期常量
        constexpr int SIZE = 10;              // ✓ 编译期常量
        constexpr int ARR_SIZE = SIZE * 2;    // ✓ 编译期常量
        constexpr int SQUARE = square(5);     // ✓ 编译期求值

        int arr[ARR_SIZE];                    // ✓ 可用于数组长度

        cout << "SIZE = " << SIZE << endl;
        cout << "ARR_SIZE = " << ARR_SIZE << endl;
        cout << "SQUARE = " << SQUARE << endl;

        return 0;
    }
    ```

!!! note "使用建议"

    - 需要**编译期常量**时优先使用 `constexpr`（如数组长度、模板参数）。
    - 表示**运行时只读**时使用 `const`。
    - `constexpr` 函数可以在运行时调用，也可以在编译期调用。

## auto：自动类型推导（C++11）

在C语言中，变量的类型必须显式声明。C++11引入的 `auto` 关键字可以让编译器根据初始化表达式自动推导变量类型。

!!! example "auto 的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        // auto自动推导类型
        auto i = 10;           // int
        auto d = 3.14;         // double
        auto c = 'A';          // char
        auto s = "hello";      // const char*

        // 复杂类型用auto更简洁
        vector<int> v = {1, 2, 3, 4, 5};

        // C++98/03风格：迭代器类型很长
        for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
            cout << *it << " ";
        }
        cout << endl;

        // C++11风格：使用auto
        for (auto it = v.begin(); it != v.end(); ++it) {
            cout << *it << " ";
        }
        cout << endl;

        return 0;
    }
    ```

!!! info "auto 的使用规则"

    1. `auto` 变量**必须初始化**（编译器需要从初始化表达式推导类型）。
    2. `auto` 会忽略引用和 `const`（除非显式添加）。
    3. `auto` 可以推导复杂类型，简化代码。
    4. 多条声明语句中，`auto` 推导的类型必须一致。

!!! example "auto 的类型推导规则"

    ``` cpp linenums="1"
    int x = 10;
    const int cx = 20;
    int& rx = x;

    auto a1 = x;      // int（忽略const和引用）
    auto a2 = cx;     // int（忽略const）
    auto a3 = rx;     // int（忽略引用）

    const auto a4 = x;  // const int（显式添加const）
    auto& a5 = rx;      // int&（显式添加引用）
    ```

!!! tip "auto 的推荐使用场景"

    - 类型名较长的复杂类型（如迭代器、lambda表达式）
    - 模板编程中类型不确定的场合
    - 范围 `for` 循环中遍历容器元素

## decltype：表达式类型推导（C++11）

`decltype` 用于**推导表达式的类型**，而不实际执行表达式。与 `auto` 不同，`decltype` 不会忽略引用和 `const`。

!!! example "decltype 的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int x = 10;
        const int cx = 20;
        int& rx = x;

        decltype(x) d1 = 5;      // int
        decltype(cx) d2 = 30;    // const int（保留const）
        decltype(rx) d3 = x;     // int&（保留引用）

        // 表达式推导
        decltype(x + 3.14) d4 = x + 3.14;  // double
        decltype(&x) d5 = &x;              // int*

        cout << "d1 = " << d1 << endl;
        cout << "d2 = " << d2 << endl;
        cout << "d3 = " << d3 << endl;
        cout << "d4 = " << d4 << endl;

        return 0;
    }
    ```

!!! info "auto vs decltype"

    | 对比项        | `auto`                     | `decltype`             |
    | :------------ | :------------------------- | :--------------------- |
    | **推导内容**  | 从初始化表达式推导变量类型 | 推导任意表达式的类型   |
    | **保留const** | 不保留（可显式添加）       | 保留                   |
    | **保留引用**  | 不保留（可显式添加）       | 保留                   |
    | **使用场景**  | 声明变量时简化类型         | 模板编程、返回类型推导 |

!!! tip "decltype 与 auto 的配合"

    C++14支持 `decltype(auto)`，结合了 `auto` 和 `decltype` 的特点，保留完整的类型信息：

    ``` cpp
    int x = 10;
    int& rx = x;

    auto a = rx;           // int（忽略引用）
    decltype(auto) b = rx; // int&（保留引用）

    const int cx = 20;
    auto c = cx;           // int（忽略const）
    decltype(auto) d = cx; // const int（保留const）
    ```

## nullptr：类型安全的空指针（C++11）

### C语言中的 NULL

在C语言中，空指针使用 `NULL` 宏表示，通常定义为 `((void*)0)` 或 `0`。

!!! warning "NULL 的问题"

    ``` c
    // C语言中
    #define NULL ((void*)0)    // 通常定义
    int* p = NULL;             // 可以

    // 问题：NULL 的类型不明确，可以是 int 或 void*
    // 在C++中会导致重载解析错误
    ```

### C++中的 NULL 问题

在C++中，`NULL` 通常定义为整数 `0`，这会导致类型混淆：

!!! example "NULL 导致的函数重载问题"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    void func(int x) {
        cout << "func(int) called" << endl;
    }

    void func(char* p) {
        cout << "func(char*) called" << endl;
    }

    int main() {
        func(0);       // 调用 func(int)
        func(NULL);    // 调用 func(int)！而不是 char* 版本
        // func(nullptr); // C++11：调用 func(char*)

        return 0;
    }
    ```

### C++11 的 nullptr

`nullptr` 是C++11引入的**类型安全的空指针**，它是 `std::nullptr_t` 类型的常量，可以隐式转换为任何指针类型，但不能转换为整数类型。

!!! success "nullptr 的优势"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    void func(int x) {
        cout << "func(int) called" << endl;
    }

    void func(char* p) {
        cout << "func(char*) called" << endl;
    }

    void func(void* p) {
        cout << "func(void*) called" << endl;
    }

    int main() {
        // nullptr 是类型安全的
        func(0);         // func(int) called
        func(nullptr);   // func(void*) called（更匹配 void* 或 char*）

        // nullptr 不能转换为整数
        // int n = nullptr;   // 错误！

        // nullptr 可以转换为任何指针类型
        int* p = nullptr;
        char* q = nullptr;
        double* r = nullptr;

        // 与指针比较
        if (p == nullptr) {
            cout << "p is null" << endl;
        }

        return 0;
    }
    ```

!!! tip "使用建议"

    - **在C++中始终使用 `nullptr`**，而不是 `NULL` 或 `0`。
    - `nullptr` 类型安全，避免重载解析的混淆。
    - `nullptr` 提高了代码的可读性和类型安全性。

## 更严格的类型检查

C++在多个方面加强了类型检查，使得编译器能够捕捉更多潜在错误。

### 函数原型必须声明

!!! warning "C语言允许隐式函数声明"

    ``` c
    // C语言中，即使没有声明函数原型，也可以调用
    int main() {
        int x = add(3, 5);   // C会假设 add 返回 int，参数为 int
        return 0;
    }

    // 如果函数实际定义不同，可能导致错误
    double add(double a, double b) {
        return a + b;
    }
    ```

!!! success "C++必须声明函数原型"

    ``` cpp
    // C++中，函数必须在使用前声明或定义
    double add(double a, double b);   // 必须有声明

    int main() {
        // double x = add(3, 5);   // ✓ 正确，类型匹配
        // int y = add(3, 5);      // ✓ 可以，返回值从 double 转为 int
        return 0;
    }
    ```

### void* 不能隐式转换

!!! warning "C语言允许 void* 隐式转换"

    ``` c
    // C语言中，void* 可以隐式转换为任何指针类型
    int* p = malloc(sizeof(int) * 10);   // ✓ 可以
    ```

!!! success "C++必须显式转换"

    ``` cpp
    // C++中，void* 不能隐式转换
    // int* p = malloc(sizeof(int) * 10);   // 错误！

    // 必须显式转换
    int* p = (int*)malloc(sizeof(int) * 10);   // 老式风格
    // int* p = static_cast<int*>(malloc(sizeof(int) * 10)); // C++风格

    // 更好的方式是使用 new
    int* q = new int[10];   // 无需转换，类型安全
    ```

### 条件表达式中的类型限制

!!! example "C++中更严格的条件表达式"

    ``` cpp
    // C++中，条件表达式的条件可以是 bool 类型
    int x = 10;
    if (x) { }      // ✓ 可以，int 隐式转换为 bool

    // 指针建议显式与 nullptr 比较
    int* p = getPtr();
    // if (p) { }      // ✓ 可以，但推荐 if (p != nullptr)
    if (p != nullptr) { }  // 更清晰
    ```

## 综合示例

!!! example "C++中的关键字"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    // constexpr 函数：编译期求值
    constexpr int square(int x) {
        return x * x;
    }

    // 函数重载演示
    void printValue(int x) {
        cout << "printValue(int): " << x << endl;
    }

    void printValue(double x) {
        cout << "printValue(double): " << x << endl;
    }

    void printValue(const char* s) {
        cout << "printValue(const char*): " << s << endl;
    }

    // 接受指针参数的函数
    void process(int* p) {
        if (p != nullptr) {   // 推荐使用 nullptr
            cout << "process: " << *p << endl;
        } else {
            cout << "process: null pointer" << endl;
        }
    }

    int main() {
        // 1. constexpr：编译期常量
        constexpr int ARRAY_SIZE = square(5);   // 编译期计算 25
        int arr[ARRAY_SIZE];                    // 可用于数组长度

        // 2. auto：类型推导
        auto i = 10;                    // int
        auto d = 3.14;                  // double
        auto s = "hello";               // const char*
        auto v = vector<int>{1, 2, 3};  // vector<int>

        // 3. decltype：表达式类型推导
        decltype(i) a = 20;             // int
        decltype(d) b = 2.71;           // double
        decltype(i + d) c = i + d;      // double

        // 4. nullptr：类型安全的空指针
        int* p1 = nullptr;
        int* p2 = nullptr;

        // 5. 函数调用
        printValue(10);       // printValue(int)
        printValue(3.14);     // printValue(double)
        printValue("hello");  // printValue(const char*)

        // 6. nullptr 与函数重载
        process(p1);          // process: null pointer
        process(&a);          // process: 20

        // 7. const vs constexpr
        const int runtime_const = i + 10;   // 运行时只读
        constexpr int compile_const = 100;  // 编译期常量

        // 8. 范围 for + auto（简单示例）
        cout << "Vector elements: ";
        for (auto x : v) {
            cout << x << " ";
        }
        cout << endl;

        return 0;
    }
    ```

    运行结果

    ```
    printValue(int): 10
    printValue(double): 3.14
    printValue(const char*): hello
    process: null pointer
    process: 20
    Vector elements: 1 2 3 
    ```

## 小结

1.  **`const`**：C++中语义更严格，是真正的编译期常量，可用于数组长度等场景。

2.  **`constexpr`**（C++11）：编译期常量表达式，比 `const` 更严格，可修饰函数和变量。

3.  **`auto`**（C++11）：自动类型推导，简化复杂类型的声明。

4.  **`decltype`**（C++11）：推导任意表达式的类型，保留 `const` 和引用。

5.  **`nullptr`**（C++11）：类型安全的空指针，替代 `NULL` 和 `0`。

6.  **更严格的类型检查**：
    - 函数必须声明原型后才能使用。
    - `void*` 不能隐式转换为其他指针类型。