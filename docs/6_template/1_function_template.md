# 函数模板

函数模板是C++泛型编程的核心工具之一。它允许程序员编写与类型无关的通用函数，由编译器在编译时根据实际使用情况生成特定类型的函数版本。函数模板极大地提高了代码的复用性和可维护性。

## 问题场景：重复的代码逻辑

在程序设计中，经常会遇到逻辑完全相同、仅数据类型不同的函数。

!!! example "求绝对值函数的重载"

    ``` cpp linenums="1"
    // 整数绝对值
    int absInt(int x) {
        return x < 0 ? -x : x;
    }

    // 双精度浮点数绝对值
    double absDouble(double x) {
        return x < 0 ? -x : x;
    }

    // 长整型绝对值
    long absLong(long x) {
        return x < 0 ? -x : x;
    }
    ```

    这三个函数的**逻辑完全相同**，只是数据类型不同。如果为每个需要的类型都编写重载版本，会导致：

    - **代码冗余**：大量重复的函数体。
    - **维护困难**：修改逻辑时需要同步修改所有版本，容易遗漏。
    - **可扩展性差**：每增加一个类型就需要增加一个重载版本。
    - **容易出错**：不同版本之间可能出现不一致。

!!! question "能否**只编写一份通用代码**，让编译器根据不同数据类型自动生成对应的函数？"

    **函数模板（Function Template）** 正是为此而生的解决方案。

## 函数模板的定义

### 基本语法

函数模板是一个**通用函数的蓝图**，它定义了一个函数族，其中的类型参数可以在使用时被具体类型替换。

!!! abstract "函数模板的语法格式"

    ``` cpp
    template <typename 类型参数1, typename 类型参数2, ...>
    返回类型 函数名(参数列表) {
        // 函数体
    }
    ```

    - `template` 关键字声明这是一个模板。
    - 尖括号 `<>` 中包含类型参数列表。
    - `typename` 关键字声明类型参数（也可以用 `class`，二者含义相同）。
    - 类型参数可以像普通类型一样在函数体中使用。

### 第一个函数模板

!!! example "绝对值函数模板"

    ``` cpp linenums="1" hl_lines="5-8"
    #include <iostream>
    using namespace std;

    // 定义函数模板
    template <typename T>
    T absValue(T x) {
        return x < 0 ? -x : x;
    }

    int main() {
        int n = -5;
        double d = -5.5;
        long l = -100L;

        // 编译器自动推导类型，生成对应版本的函数
        cout << absValue(n) << endl;   // 5（调用 int 版本）
        cout << absValue(d) << endl;   // 5.5（调用 double 版本）
        cout << absValue(l) << endl;   // 100（调用 long 版本）

        return 0;
    }
    ```

### typename 与 class

在模板参数声明中，`typename` 和 `class` 可以互换使用。`typename` 是 C++ 标准化后引入的关键字，语义更清晰。

!!! tip "两种写法"

    ``` cpp
    // 使用 typename（推荐，语义更清晰）
    template <typename T>
    T max(T a, T b) {
        return a > b ? a : b;
    }

    // 使用 class（传统写法，与 typename 完全等价）
    template <class T>
    T max(T a, T b) {
        return a > b ? a : b;
    }
    ```

    !!! note "区别"

        在模板参数声明中，`typename` 和 `class` **完全等价**。但 `typename` 更能准确表达"这是一个类型参数"的含义，建议优先使用。

## 函数模板的实例化

### 什么是实例化

函数模板本身不是函数，它是生成函数的"蓝图"。当编译器遇到对函数模板的调用时，会根据实参类型生成一个具体的函数，这个过程称为**模板实例化（Template Instantiation）**。

!!! info "实例化过程"

    当编译器看到 `absValue(n)`（其中 `n` 是 `int` 类型）时：

    1. 根据实参 `n` 的类型推断模板参数 `T` 为 `int`。
    2. 生成一个具体的函数：`int absValue(int x) { return x < 0 ? -x : x; }`。
    3. 编译并链接这个生成的函数。

    生成的函数称为**模板函数（Template Function）**。

### 隐式实例化（类型推导）

当编译器能够从函数实参中推导出模板参数类型时，不需要显式指定。

!!! example "隐式实例化"

    ``` cpp linenums="1" hl_lines="11 12 15"
    template <typename T>
    T add(T a, T b) {
        return a + b;
    }

    int main() {
        int i1 = 3, i2 = 5;
        double d1 = 3.14, d2 = 2.71;

        // 隐式实例化：编译器根据实参类型推导 T
        int result1 = add(i1, i2);     // T 推导为 int
        double result2 = add(d1, d2);  // T 推导为 double

        // 不同参数类型时无法推导
        // auto r3 = add(i1, d1);       // 错误！T 不能同时是 int 和 double

        return 0;
    }
    ```

!!! warning "类型推导的限制"

    当模板参数无法从实参中唯一推导时，编译会失败：

    ``` cpp hl_lines="11 14"
    template <typename T>
    T add(T a, T b) {
        return a + b;
    }

    int main() {
        int i = 3;
        double d = 3.14;

        // 错误：T 可以推导为 int 或 double，不唯一
        // auto r = add(i, d);

        // 解决方案：显式指定类型
        auto r = add<double>(i, d);   // 将 i 转换为 double

        return 0;
    }
    ```

### 显式实例化（指定类型）

在某些情况下，需要显式指定模板参数类型。

!!! example "显式实例化"

    ``` cpp linenums="1" hl_lines="11 12"
    template <typename T>
    T add(T a, T b) {
        return a + b;
    }

    int main() {
        int i = 3;
        double d = 3.14;

        // 显式实例化：在 <> 中指定 T 的类型
        double result1 = add<double>(i, d);  // 将 i 隐式转换为 double
        int result2 = add<int>(i, d);        // 将 d 隐式转换为 int（截断）

        cout << result1 << endl;   // 6.14
        cout << result2 << endl;   // 6

        return 0;
    }
    ```

### 显式实例化声明（C++11）

C++11 允许在函数模板定义处进行显式实例化声明，强制编译器生成特定类型的函数。

!!! example "显式实例化声明"

    ``` cpp linenums="1" hl_lines="8 9"
    // 函数模板定义
    template <typename T>
    T max(T a, T b) {
        return a > b ? a : b;
    }

    // 显式实例化声明：强制生成 int 和 double 版本
    template int max<int>(int, int);
    template double max<double>(double, double);

    int main() {
        // 可以直接使用，编译器已经生成了对应版本
        cout << max(3, 5) << endl;        // 5
        cout << max(3.14, 2.71) << endl;  // 3.14

        return 0;
    }
    ```

## 多类型参数的函数模板

函数模板可以有多个类型参数，适用于不同类型的操作数。

!!! example "多个类型参数"

    ``` cpp linenums="1" hl_lines="5-8 11-14 17-20"
    #include <iostream>
    using namespace std;

    // 两个类型参数
    template <typename T1, typename T2>
    void display(T1 a, T2 b) {
        cout << "T1: " << a << ", T2: " << b << endl;
    }

    // 返回类型为 double，参数类型为模板参数
    template <typename T>
    double toDouble(T value) {
        return static_cast<double>(value);
    }

    // 返回类型与参数类型不同的模板（使用 auto 推导）
    template <typename T>
    auto square(T x) -> decltype(x * x) {
        return x * x;
    }

    int main() {
        display(10, 3.14);          // T1=int, T2=double
        display("hello", 100);      // T1=const char*, T2=int

        cout << toDouble(5) << endl;        // 5.0
        cout << toDouble(3.14f) << endl;    // 3.14

        cout << square(5) << endl;          // 25（int）
        cout << square(3.14) << endl;       // 9.8596（double）

        return 0;
    }
    ```

!!! tip "返回类型推导"

    当返回类型依赖于模板参数且无法简单确定时，可以使用：

    - `auto` + `decltype`（C++11）：`auto func(T x) -> decltype(x * x)`
    - `auto` 返回类型推导（C++14）：`auto func(T x) { return x * x; }`

## 函数模板与函数重载

函数模板可以和普通函数一起构成重载关系。当同时存在模板和普通函数时，编译器会优先选择最匹配的版本。

!!! example "模板与重载的配合"

    ``` cpp linenums="1" hl_lines="5-8 11-13 16-18"
    #include <iostream>
    using namespace std;

    // 函数模板：适用于任意类型
    template <typename T>
    void print(T value) {
        cout << "Template: " << value << endl;
    }

    // 普通函数：专门用于 int 类型
    void print(int value) {
        cout << "Specialized int: " << value << endl;
    }

    // 普通函数：专门用于 const char*
    void print(const char* value) {
        cout << "Specialized string: " << value << endl;
    }

    int main() {
        print(10);        // 调用普通函数（int 版本更匹配）
        print(3.14);      // 调用模板（double 没有普通函数）
        print("hello");   // 调用普通函数（const char* 版本更匹配）
        print('A');       // 调用模板（char 没有普通函数）

        // 可以显式指定使用模板版本
        print<int>(20);   // 强制调用模板的 int 版本

        return 0;
    }
    ```

    运行结果：

    ```
    Specialized int: 10
    Template: 3.14
    Specialized string: hello
    Template: A
    Template: 20
    ```

!!! info "重载解析规则"

    1. **优先匹配普通函数**：如果普通函数与实参类型完全匹配，优先调用普通函数。
    2. **其次匹配模板**：如果没有完全匹配的普通函数，尝试从模板实例化。
    3. **显式指定模板参数**：可以强制使用模板版本，忽略普通函数。

## 函数模板的特化

### 全特化

当模板的通用实现不适用于某些特定类型时，可以对这些类型提供**特化（Specialization）**版本。

!!! example "函数模板的全特化"

    ``` cpp linenums="1" hl_lines="6-9 12-15" 
    #include <iostream>
    #include <cstring>
    using namespace std;

    // 通用模板：适用于任意类型
    template <typename T>
    bool isEqual(T a, T b) {
        return a == b;
    }

    // 特化版本：针对 const char*（C风格字符串）
    template <>
    bool isEqual<const char*>(const char* a, const char* b) {
        return strcmp(a, b) == 0;
    }

    int main() {
        int x = 5, y = 5;
        cout << isEqual(x, y) << endl;       // 1（使用通用模板）

        double d1 = 3.14, d2 = 3.14;
        cout << isEqual(d1, d2) << endl;     // 1（使用通用模板）

        const char* s1 = "hello";
        const char* s2 = "hello";
        const char* s3 = "world";
        cout << isEqual(s1, s2) << endl;     // 1（使用特化版本，比较内容）
        cout << isEqual(s1, s3) << endl;     // 0（使用特化版本）

        return 0;
    }
    ```

!!! abstract "特化语法"

    - `template <>` 表示这是一个特化版本。
    - 函数名后跟 `<类型>` 指定特化的类型。
    - 特化版本的参数类型必须与通用模板一致。

### 特化 vs 重载

对于函数模板，特化和重载都能实现"针对特定类型的定制行为"，但二者有所区别。

!!! example "优先选择重载"

    对于函数模板，**重载通常比特化更优先**：

    ``` cpp
    // 通用模板
    template <typename T>
    void func(T x) { cout << "Template" << endl; }

    // 特化版本（不推荐，重载更好）
    template <>
    void func<int>(int x) { cout << "Specialization" << endl; }

    // 重载版本（推荐）
    void func(int x) { cout << "Overload" << endl; }

    int main() {
        func(10);   // 调用重载版本（优先于特化）
        return 0;
    }
    ```

!!! tip "建议"

    对于函数模板，优先使用**普通函数重载**来处理特定类型，而不是模板特化。类模板则必须使用特化。

## 函数模板的约束

### 隐式约束

函数模板并非可以处理所有类型。模板函数体中使用的操作，必须对实际类型参数是合法的。

!!! warning "模板的隐式约束"

    ``` cpp linenums="1"
    template <typename T>
    T max(T a, T b) {
        return a > b ? a : b;  // 要求 T 类型支持 > 运算符
    }

    template <typename T>
    T sum(T a, T b) {
        return a + b;          // 要求 T 类型支持 + 运算符
    }

    template <typename T>
    void print(T value) {
        cout << value;         // 要求 T 类型支持 << 运算符
    }
    ```

如果需要自定义类型作为模板参数，必须重载模板中使用到的运算符。

!!! example "自定义类型作为模板参数"

    ``` cpp linenums="1" hl_lines="12-14 23-26"
    #include <iostream>
    using namespace std;

    class Point {
    private:
        int x, y;

    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 重载 > 运算符（支持 max 模板）
        bool operator>(const Point& other) const {
            return (x * x + y * y) > (other.x * other.x + other.y * other.y);
        }

        // 重载 << 运算符（支持 print 模板）
        friend ostream& operator<<(ostream& os, const Point& p) {
            os << "(" << p.x << ", " << p.y << ")";
            return os;
        }
    };

    template <typename T>
    T max(T a, T b) {
        return a > b ? a : b;
    }

    int main() {
        Point p1(3, 4);
        Point p2(5, 0);

        Point p3 = max(p1, p2);
        cout << "Max point: " << p3 << endl;   // 距离更大的是 (5, 0)

        return 0;
    }
    ```

### C++20 的概念（Concepts）

C++20 引入了**概念（Concepts）**，可以显式约束模板参数，提供更清晰的错误信息。

!!! example "使用概念约束（C++20）"

    ``` cpp linenums="1"
    #include <concepts>

    // 要求 T 类型支持 < 运算符
    template <typename T>
    requires std::totally_ordered<T>
    T max(T a, T b) {
        return a > b ? a : b;
    }

    // 或者使用更简洁的语法
    template <std::totally_ordered T>
    T max(T a, T b) {
        return a > b ? a : b;
    }
    ```

## 综合示例

!!! example "函数模板示例"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    // === 1. 简单函数模板：交换两个值 ===
    template <typename T>
    void mySwap(T& a, T& b) {
        T temp = a;
        a = b;
        b = temp;
    }

    // === 2. 函数模板：查找数组最大值 ===
    template <typename T>
    T maxInArray(const T arr[], int size) {
        T maxVal = arr[0];
        for (int i = 1; i < size; i++) {
            if (arr[i] > maxVal) {
                maxVal = arr[i];
            }
        }
        return maxVal;
    }

    // === 3. 函数模板：输出容器内容（支持 vector） ===
    template <typename T>
    void printContainer(const vector<T>& container) {
        cout << "[";
        for (size_t i = 0; i < container.size(); i++) {
            cout << container[i];
            if (i < container.size() - 1) cout << ", ";
        }
        cout << "]" << endl;
    }

    // === 4. 多类型参数：将容器转换为字符串 ===
    template <typename T>
    string toString(const T& value) {
        return to_string(value);
    }

    // string 特化
    template <>
    string toString(const string& value) {
        return value;
    }

    // === 5. 函数模板与重载配合 ===
    template <typename T>
    void show(T value) {
        cout << "Template: " << value << endl;
    }

    void show(int value) {
        cout << "Overloaded int: " << value << endl;
    }

    int main() {
        // 测试 1：交换
        int a = 5, b = 10;
        cout << "Before swap: a=" << a << ", b=" << b << endl;
        mySwap(a, b);
        cout << "After swap:  a=" << a << ", b=" << b << endl;

        double c = 3.14, d = 2.71;
        mySwap(c, d);
        cout << "After swap: c=" << c << ", d=" << d << endl;

        // 测试 2：数组最大值
        int intArr[] = {3, 7, 2, 9, 5};
        double doubleArr[] = {3.14, 2.71, 1.41, 6.28};

        cout << "Max int: " << maxInArray(intArr, 5) << endl;     // 9
        cout << "Max double: " << maxInArray(doubleArr, 4) << endl; // 6.28

        // 测试 3：容器输出
        vector<int> vi = {1, 2, 3, 4, 5};
        vector<string> vs = {"hello", "world", "C++"};

        printContainer(vi);
        printContainer(vs);

        // 测试 4：多类型与特化
        cout << "toString(100) = " << toString(100) << endl;
        cout << "toString(3.14) = " << toString(3.14) << endl;
        cout << "toString(\"hello\") = " << toString(string("hello")) << endl;

        // 测试 5：模板与重载
        show(10);        // Overloaded int: 10
        show(3.14);      // Template: 3.14
        show("hello");   // Template: hello
        show<int>(20);   // Template: 20（强制使用模板）

        return 0;
    }
    ```

## 小结

1.  **函数模板的定义**：`template <typename T>` + 普通函数定义，类型参数 `T` 代表通用类型。

2.  **模板实例化**：
    - **隐式实例化**：编译器根据实参类型自动推导模板参数。
    - **显式实例化**：在 `<>` 中指定模板参数类型。

3.  **多类型参数**：`template <typename T1, typename T2>` 支持多个类型参数。

4.  **模板与重载**：
    - 普通函数优先于模板函数匹配。
    - 可以用 `show<int>(value)` 强制使用模板版本。

5.  **模板特化**：为特定类型提供专门的实现，使用 `template <>` 语法。

6.  **模板的约束**：模板函数体中使用的操作必须对实际类型参数合法。