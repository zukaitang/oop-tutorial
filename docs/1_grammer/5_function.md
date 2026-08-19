# 函数的扩展

C++在C语言函数机制的基础上进行了多项重要扩展，包括**函数重载**、**默认参数**和**内联函数**。这些特性让函数的使用更加灵活、安全，也为后续的面向对象编程奠定了基础。

## C语言函数的主要局限

C语言的函数机制虽然功能完备，但在实际使用中存在一些不便之处：

| 局限             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| **函数名唯一**   | 功能相似的函数必须使用不同名称（如 `abs_int`、`abs_double`） |
| **参数固定**     | 函数参数的默认值必须由调用者提供                             |
| **宏的不安全**   | 用 `#define` 宏实现"内联"缺乏类型检查                        |
| **强制转换繁琐** | `void*` 需要显式转换为具体类型指针                           |

C++针对这些问题引入了相应的改进机制。

## 函数重载

### 什么是函数重载

**函数重载（Function Overloading）** 是指在**同一作用域**内，多个函数可以拥有**相同的函数名**，但**参数列表不同**（参数个数、类型或顺序不同）。

!!! abstract "函数重载的核心价值"

    函数重载让程序员可以用同一个函数名表达**一组功能相似的操作**，编译器根据实参的个数和类型自动选择合适的版本。这提高了代码的可读性和一致性。

!!! example "函数重载的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 重载：打印 int 类型
    void print(int value) {
        cout << "int: " << value << endl;
    }

    // 重载：打印 double 类型
    void print(double value) {
        cout << "double: " << value << endl;
    }

    // 重载：打印字符串（C风格）
    void print(const char* value) {
        cout << "const char*: " << value << endl;
    }

    // 重载：打印两个值
    void print(int a, int b) {
        cout << "int, int: " << a << ", " << b << endl;
    }

    int main() {
        print(10);              // 调用 print(int)
        print(3.14);            // 调用 print(double)
        print("hello");         // 调用 print(const char*)
        print(10, 20);          // 调用 print(int, int)

        return 0;
    }
    ```

### 重载的规则

函数重载依据**参数列表**进行区分，具体包括：

1. **参数个数不同**
2. **参数类型不同**
3. **参数顺序不同**（当类型不同时）

注意：**返回值类型不同**不能作为重载的依据。

!!! example "合法的重载示例"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 1. 参数个数不同
    void func(int a) {
        cout << "func(int)" << endl;
    }
    void func(int a, int b) {
        cout << "func(int, int)" << endl;
    }

    // 2. 参数类型不同
    void display(int x) {
        cout << "display(int)" << endl;
    }
    void display(double x) {
        cout << "display(double)" << endl;
    }

    // 3. 参数顺序不同（类型不同时）
    void process(int a, double b) {
        cout << "process(int, double)" << endl;
    }
    void process(double a, int b) {
        cout << "process(double, int)" << endl;
    }

    int main() {
        func(10);                 // func(int)
        func(10, 20);             // func(int, int)

        display(3);               // display(int)
        display(3.14);            // display(double)

        process(10, 3.14);        // process(int, double)
        process(3.14, 10);        // process(double, int)

        return 0;
    }
    ```

!!! danger "不能仅凭返回值类型重载"

    ``` cpp
    // ❌ 错误！仅返回值类型不同，不能构成重载
    // int getValue() { return 10; }
    // double getValue() { return 3.14; }

    // 编译器无法区分调用哪个版本
    // getValue();   // 二义性错误！
    ```

### 重载与类型转换

当实参类型与所有重载版本都不完全匹配时，编译器会尝试进行**隐式类型转换**来匹配。

!!! example "重载解析中的类型转换"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    void show(char c) {
        cout << "char: " << c << endl;
    }

    void show(int n) {
        cout << "int: " << n << endl;
    }

    void show(double d) {
        cout << "double: " << d << endl;
    }

    int main() {
        show('A');     // 完全匹配 char → show(char)
        show(10);      // 完全匹配 int → show(int)
        show(3.14);    // 完全匹配 double → show(double)

        show(10L);     // long → 转换为 int（或 double），存在二义性
        // show('A');  // char 可以转换为 int，但 show(char) 更优先

        return 0;
    }
    ```

!!! info "重载解析的优先级"

    编译器在选择最佳匹配版本时，按以下优先级进行：

    1. **精确匹配**：类型完全一致。
    2. **提升匹配**：`char` → `int`，`float` → `double` 等。
    3. **标准转换**：`int` → `long`，`int` → `double` 等。
    4. **用户定义转换**：通过构造函数或类型转换运算符。
    5. **可变参数**：`...`。

## 默认参数

### 默认参数的语法

**默认参数（Default Arguments）** 允许在函数声明时为参数指定默认值。调用时如果不提供该参数，则使用默认值。

!!! info "默认参数的语法"

    ``` cpp
    返回类型 函数名(参数类型 参数名 = 默认值, ...) {
        // 函数体
    }
    ```

!!! example "默认参数的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 带默认参数的函数
    void greet(const string& name = "Guest", const string& prefix = "Hello") {
        cout << prefix << ", " << name << "!" << endl;
    }

    int main() {
        greet();                      // Hello, Guest!
        greet("Alice");               // Hello, Alice!
        greet("Bob", "Welcome");      // Welcome, Bob!

        return 0;
    }
    ```

### 默认参数的规则

!!! warning "默认参数必须从右向左依次设定，位于参数列表的尾部"

    ``` cpp
    // ✓ 正确：从右向左依次设定默认值
    void func1(int a, int b = 10, int c = 20);   // 正确
    void func2(int a = 5, int b = 10, int c = 20); // 正确

    // ✗ 错误：中间缺少默认值
    // void func3(int a, int b = 10, int c);   // 错误！c 没有默认值

    // ✗ 错误：左边有默认值而右边没有
    // void func4(int a = 5, int b, int c);    // 错误！
    ```

!!! example "默认参数的传递规则"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 默认参数必须从右向左设定
    void show(int a, int b = 10, int c = 20) {
        cout << "a=" << a << ", b=" << b << ", c=" << c << endl;
    }

    int main() {
        show(1);       // a=1, b=10, c=20
        show(1, 2);    // a=1, b=2, c=20
        show(1, 2, 3); // a=1, b=2, c=3

        // show(1, , 3); // 错误！不能跳过 b 给 c 传值

        return 0;
    }
    ```

### 默认参数与函数重载

默认参数和函数重载可以互相替代，但各有适用场景。

!!! example "默认参数 vs 函数重载"

    ``` cpp
    // 方式一：使用默认参数
    void print(const char* msg, int times = 1) {
        for (int i = 0; i < times; i++) {
            cout << msg << endl;
        }
    }

    // 方式二：使用函数重载（等效）
    void print(const char* msg) {
        print(msg, 1);   // 委托给另一个版本
    }
    void print(const char* msg, int times) {
        for (int i = 0; i < times; i++) {
            cout << msg << endl;
        }
    }
    ```

!!! tip "选择建议"

    - **使用默认参数**：当需要减少参数个数，且参数之间的逻辑关系明确时。
    - **使用函数重载**：当需要不同的函数行为（不仅仅是参数不同），或者默认值不满足需求时。

### 默认参数的声明位置

!!! info "默认参数只能在声明中指定"

    默认参数应该在**函数声明（通常在头文件中）** 中指定，而不应在定义中重复指定。

    ``` cpp
    // 头文件：声明中指定默认参数
    void func(int a, int b = 10);

    // 实现文件：定义中不重复默认参数
    void func(int a, int b) {
        // 函数体
    }
    ```

    如果在多个声明中重复指定默认参数，且值不同，会导致编译错误。

## 内联函数

### 为什么需要内联函数

在C语言中，频繁调用短小函数会带来函数调用开销（压栈、跳转、返回等）。为了解决这个问题，C程序员常使用**函数宏（Function-like Macro）**：

!!! example "C语言中的函数宏"

    ``` c
    #define SQUARE(x) ((x) * (x))

    int main() {
        int a = 5;
        int b = SQUARE(a);  // 展开为 ((a) * (a))
        return 0;
    }
    ```

但宏存在严重问题：没有类型检查、容易产生副作用、调试困难。C++的**内联函数（Inline Function）** 在保持宏的性能优势的同时，提供了类型安全和函数的所有特性。

!!! info "内联函数的设计目标"

    内联函数向编译器建议：将函数体在调用点**展开**，避免函数调用的开销。这结合了宏的性能和函数的类型安全性。

### 内联函数的定义

!!! abstract "内联函数的语法"

    ``` cpp
    inline 返回类型 函数名(参数列表) {
        // 函数体
    }
    ```

!!! example "内联函数的使用"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 内联函数：求平方
    inline int square(int x) {
        return x * x;
    }

    // 内联函数：求绝对值
    inline int absValue(int x) {
        return x < 0 ? -x : x;
    }

    // 内联函数：交换两个值
    inline void swapInt(int& a, int& b) {
        int temp = a;
        a = b;
        b = temp;
    }

    int main() {
        int x = 5;
        int y = square(x);     // 编译时可能展开为：int y = x * x;
        cout << "square(5) = " << y << endl;

        int a = -10;
        cout << "abs(-10) = " << absValue(a) << endl;

        int p = 10, q = 20;
        swapInt(p, q);         // 展开为函数体
        cout << "p=" << p << ", q=" << q << endl;

        return 0;
    }
    ```

### 内联函数 vs 宏

!!! summary "内联函数与宏的对比"

    | 对比项         | 内联函数           | 宏                         |
    | :------------- | :----------------- | :------------------------- |
    | **类型检查**   | ✓ 有类型检查       | ✗ 无类型检查（文本替换）   |
    | **参数求值**   | 只求值一次         | 多次求值（可能产生副作用） |
    | **作用域控制** | ✓ 有作用域         | ✗ 全局有效                 |
    | **调试支持**   | ✓ 可调试           | ✗ 调试困难                 |
    | **代码膨胀**   | 可能（函数体展开） | 必然（每次展开）           |
    | **C++推荐**    | ✓ 推荐使用         | ✗ 尽量少用                 |

!!! danger "宏的陷阱：副作用"

    ``` cpp
    // 宏定义（危险）
    #define SQUARE(x) ((x) * (x))

    int main() {
        int a = 5;
        int b = SQUARE(++a);   // 展开为 ((++a) * (++a))，a 被修改两次！
        // a 变为 7，b 变为 49，行为不符合预期

        // 内联函数（安全）
        int c = 5;
        int d = square(++c);   // c 被修改一次，行为可预测
        // c 变为 6，d 变为 36

        return 0;
    }
    ```

### 内联函数的限制

!!! warning "`inline` 只是建议，而非强制"

    `inline` 关键字只是向编译器提出**建议**，编译器有权决定是否真正内联展开。以下情况编译器通常会忽略 `inline`：

    - 函数体过大（包含循环、复杂控制结构）。
    - 递归函数（无法内联）。
    - 虚函数（需要动态绑定）。
    - 通过函数指针调用的函数。

    ``` cpp
    // 编译器可能忽略 inline
    inline int complexFunc(int n) {
        int sum = 0;
        for (int i = 0; i < n; i++) {   // 包含循环
            sum += i;
        }
        return sum;
    }

    inline int recursiveFunc(int n) {   // 递归函数
        return n <= 1 ? 1 : n * recursiveFunc(n - 1);
    }
    ```

### 内联函数的定义位置

!!! tip "内联函数的定义通常放在头文件中"

    内联函数的定义需要在使用时可见（编译器需要在调用点展开函数体），因此通常定义在**头文件**中，并在开头加上 `inline` 关键字。

    ``` cpp
    // math_utils.h
    #ifndef MATH_UTILS_H
    #define MATH_UTILS_H

    // 内联函数定义在头文件中
    inline int square(int x) {
        return x * x;
    }

    inline int max(int a, int b) {
        return a > b ? a : b;
    }

    #endif
    ```

    与普通函数不同，内联函数在头文件中定义不会引起"重复定义"错误，因为每个编译单元都会生成自己的内联版本。

## 综合示例

!!! example "C++中的函数扩展示例"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    // === 内联函数 ===
    inline int maxValue(int a, int b) {
        return a > b ? a : b;
    }

    inline double maxValue(double a, double b) {
        return a > b ? a : b;
    }

    // === 带默认参数的函数 ===
    void logMessage(const string& msg, int level = 1, bool timestamp = true) {
        if (timestamp) {
            cout << "[LOG] ";
        }
        cout << "Level " << level << ": " << msg << endl;
    }

    // === 函数重载 ===
    // 处理整数
    string format(int value) {
        return to_string(value);
    }

    // 处理浮点数
    string format(double value) {
        return to_string(value);
    }

    // 处理字符串
    string format(const string& value) {
        return "\"" + value + "\"";
    }

    // 处理布尔值
    string format(bool value) {
        return value ? "true" : "false";
    }

    // 处理两个值
    string format(const string& name, int value) {
        return name + " = " + to_string(value);
    }

    // === 默认参数与重载配合 ===
    void createWindow(int width = 800, int height = 600, const string& title = "Window") {
        cout << "Creating window: " << title
            << " (" << width << "x" << height << ")" << endl;
    }

    int main() {
        cout << "=== 内联函数 ===" << endl;
        cout << "max(10, 20) = " << maxValue(10, 20) << endl;
        cout << "max(3.14, 2.71) = " << maxValue(3.14, 2.71) << endl;

        cout << "\n=== 默认参数 ===" << endl;
        logMessage("System started");              // [LOG] Level 1: System started
        logMessage("Warning", 2);                  // [LOG] Level 2: Warning
        logMessage("Error", 3, false);             // Level 3: Error

        cout << "\n=== 函数重载 ===" << endl;
        cout << format(10) << endl;                // 10
        cout << format(3.14) << endl;              // 3.140000
        cout << format("hello") << endl;           // "hello"
        cout << format(true) << endl;              // true
        cout << format("count", 42) << endl;       // count = 42

        cout << "\n=== 默认参数与重载配合 ===" << endl;
        createWindow();                            // Creating window: Window (800x600)
        createWindow(1024);                        // Creating window: Window (1024x600)
        createWindow(1024, 768);                   // Creating window: Window (1024x768)
        createWindow(1024, 768, "MainWindow");     // Creating window: MainWindow (1024x768)

        return 0;
    }
    ```

    运行结果

    ```
    === 内联函数 ===
    max(10, 20) = 20
    max(3.14, 2.71) = 3.14

    === 默认参数 ===
    [LOG] Level 1: System started
    [LOG] Level 2: Warning
    Level 3: Error

    === 函数重载 ===
    10
    3.140000
    "hello"
    true
    count = 42

    === 默认参数与重载配合 ===
    Creating window: Window (800x600)
    Creating window: Window (1024x600)
    Creating window: Window (1024x768)
    Creating window: MainWindow (1024x768)
    ```

## 小结

1.  **函数重载**：
    1. 同一作用域内，多个函数可以使用相同名称，但参数列表必须不同。
    2. 参数列表不同指：参数个数、类型或顺序不同。
    3. 返回值类型不能作为重载的依据。
2.  **默认参数**：
    1. 为函数参数指定默认值，简化函数调用。
    2. 默认参数必须从右向左依次设定。
    3. 默认参数应在函数声明中指定，不在定义中重复。
3.  **内联函数**：
    1. 使用 `inline` 关键字建议编译器展开函数体，避免函数调用开销。
    2. `inline` 只是建议，编译器可能忽略。
    3. 内联函数定义通常放在头文件中。
    4. 内联函数比宏更安全（有类型检查，无副作用）。
4.  **三者关系**：
    1. 重载提高了函数名的一致性。
    2. 默认参数简化了函数调用。
    3. 内联函数消除了宏的缺陷，兼顾性能与类型安全。