# 命名空间

命名空间是C++中用于组织代码、避免名称冲突的重要机制。
随着程序规模的增长，全局命名空间中的名称越来越多，命名冲突的风险也随之增加。
命名空间为解决这一问题提供了系统化的方案。

## C语言的全局命名空间的问题

在C语言中，所有函数和全局变量都位于**同一个全局作用域**中。当程序规模增大时，不同模块之间的名称很容易发生冲突。
尤其在：

- **多个开发者协作**：各自定义的函数可能同名。
- **使用第三方库**：库中的名称可能与项目中的名称冲突。
- **版本演进**：不同版本的代码可能引入同名标识符。

!!! example "C语言的名称冲突问题"

    ``` c
    // module_a.c
    int count = 0;
    void process() { /* ... */ }

    // module_b.c
    int count = 100;   // 错误！count 重复定义
    void process() {   // 错误！process 重复定义
        // ...
    }
    ```

!!! failure "传统解决方案的局限性"

    | 方案                         | 问题                                |
    | :--------------------------- | :---------------------------------- |
    | **使用不同前缀**             | 名称冗长（如 `mylib_process_data`） |
    | **使用 `static` 限制作用域** | 只能限制在文件内，无法跨文件共享    |
    | **依赖于开发者记忆**         | 容易遗忘，不可靠                    |

C++的**命名空间（Namespace）** 提供了更优雅、更系统的解决方案。

## 命名空间的基本概念

**命名空间（Namespace）** 是一种**作用域机制**，用于将相关的函数、类、变量等组织在一起，形成一个独立的名称区域。
不同命名空间中的同名标识符互不冲突。

!!! abstract "命名空间的设计目标"

    - **避免名称冲突**：不同命名空间中的名称相互隔离。
    - **组织代码**：将相关的功能模块分组。
    - **控制可见性**：通过 `using` 选择性导入名称，精确控制哪些名称可见。

### 声明命名空间

!!! info "命名空间的语法"

    ``` cpp
    namespace 命名空间名 {
        // 可以包含：变量、函数、类、结构体、枚举、其他命名空间等
    }
    ```

!!! example "定义和使用命名空间"

    ``` cpp linenums="1" hl_lines="5 18 32 33 37"
    #include <iostream>
    using namespace std;

    // 定义命名空间：数学工具
    namespace MathUtils {
        const double PI = 3.1415926;

        double square(double x) {
            return x * x;
        }

        double cube(double x) {
            return x * x * x;
        }
    }

    // 定义命名空间：字符串工具
    namespace StringUtils {
        int length(const char* str) {
            int len = 0;
            while (str[len]) len++;
            return len;
        }

        bool isEmpty(const char* str) {
            return str[0] == '\0';
        }
    }

    int main() {
        // 使用作用域运算符访问命名空间中的成员
        cout << "PI = " << MathUtils::PI << endl;
        cout << "square(5) = " << MathUtils::square(5) << endl;

        const char* text = "hello";
        cout << "Length of '" << text << "' = "
             << StringUtils::length(text) << endl;

        return 0;
    }
    ```

## 命名空间的使用方式

### 方式一：完全限定名（推荐）

使用**作用域运算符 `::`** 显式指定命名空间，这是最清晰、最安全的方式。

!!! example "完全限定名的使用"

    ``` cpp linenums="1" hl_lines="21 22"
    #include <iostream>
    using namespace std;

    namespace Geometry {
        const double PI = 3.14159;
        double circleArea(double radius) {
            return PI * radius * radius;
        }
    }

    namespace Physics {
        const double G = 9.8;
        double kineticEnergy(double mass, double velocity) {
            return 0.5 * mass * velocity * velocity;
        }
    }

    int main() {
        // 使用完全限定名访问
        double radius = 5.0;
        double area = Geometry::circleArea(radius);
        double energy = Physics::kineticEnergy(10.0, 3.0);

        cout << "Circle area: " << area << endl;
        cout << "Kinetic energy: " << energy << endl;

        return 0;
    }
    ```

!!! success "完全限定名的优势"

    - **无歧义**：明确指定来源，避免冲突。
    - **可读性好**：读者能立即知道名称来自哪个模块。
    - **适合头文件**：不会污染其他模块的命名空间。

### 方式二：using 声明

**`using` 声明** 将命名空间中的**单个名称**引入当前作用域。

!!! example "using 声明"

    ``` cpp linenums="1" hl_lines="12 14 17"
    #include <iostream>
    using namespace std;

    namespace MathUtils {
        double square(double x) { return x * x; }
        double cube(double x) { return x * x * x; }
        bool isPositive(double x) { return x > 0; }
    }

    int main() {
        // 只引入 square，其他仍需限定名
        using MathUtils::square;

        cout << "square(5) = " << square(5) << endl;

        // cube 仍需限定名
        cout << "cube(5) = " << MathUtils::cube(5) << endl;

        return 0;
    }
    ```

!!! tip "using 声明 vs 完全限定名"

    | 对比项         | 完全限定名         | using 声明                 |
    | :------------- | :----------------- | :------------------------- |
    | **明确来源**   | ✓ 非常清晰         | ✓ 声明处清晰，使用处不明确 |
    | **冲突风险**   | ✓ 无风险           | ⚠️ 可能与本地名称冲突      |
    | **代码简洁性** | 稍繁琐             | 更简洁                     |
    | **推荐场景**   | 头文件、全局作用域 | 局部函数、少量使用         |

### 方式三：using namespace 指令

**`using namespace` 指令** 将命名空间中的**所有名称**引入当前作用域。

!!! example "using namespace 指令"

    ``` cpp linenums="1" hl_lines="12 15 16"
    #include <iostream>
    using namespace std;   // 将 std 中所有名称引入

    namespace MathUtils {
        double square(double x) { return x * x; }
        double cube(double x) { return x * x * x; }
        int max(int a, int b) { return a > b ? a : b; }
    }

    int main() {
        // 引入 MathUtils 中所有名称
        using namespace MathUtils;

        // 直接使用，无需限定名
        cout << "square(5) = " << square(5) << endl;
        cout << "cube(5) = " << cube(5) << endl;

        return 0;
    }
    ```

!!! danger "using namespace 的风险"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    namespace LibA {
        void print() { cout << "LibA::print()" << endl; }
        int value = 10;
    }

    namespace LibB {
        void print() { cout << "LibB::print()" << endl; }
        int value = 20;
    }

    int main() {
        // 两个命名空间都被引入，产生冲突
        using namespace LibA;
        using namespace LibB;

        // print();   // 错误！二义性：LibA::print 还是 LibB::print？
        // cout << value; // 错误！二义性：LibA::value 还是 LibB::value？

        // 只能使用限定名避免冲突
        LibA::print();
        LibB::print();

        return 0;
    }
    ```

!!! warning "using namespace 的使用建议"

    - **不要**在头文件中使用 `using namespace`（会污染所有包含该头文件的源文件）。
    - **谨慎**在全局作用域使用（可能导致名称冲突）。
    - **可以**在局部作用域（函数内部）使用，影响范围有限。
    - **优先**使用 `using` 声明（导入单个名称）或完全限定名。

## using 的不同用法

### using 用于类型别名

`using` 可以替代 `typedef` 创建类型别名，语法更直观。

!!! example "using 类型别名"

    ``` cpp linenums="1"  hl_lines="7 8 11-13 16 17"
    #include <iostream>
    #include <vector>
    #include <string>
    using namespace std;

    // 传统方式：typedef
    typedef vector<int> IntVector;
    typedef void(*FuncPtr)(int);   // 函数指针类型

    // C++11 方式：using（更直观）
    using IntList = vector<int>;
    using StringList = vector<string>;
    using PrintFunc = void(*)(int);

    int main() {
        IntList numbers = {1, 2, 3, 4, 5};
        StringList names = {"Alice", "Bob", "Charlie"};

        for (int n : numbers) {
            cout << n << " ";
        }
        cout << endl;

        for (const auto& name : names) {
            cout << name << " ";
        }
        cout << endl;

        return 0;
    }
    ```

!!! info "using vs typedef"

    | 对比项       | `typedef`               | `using`                   |
    | :----------- | :---------------------- | :------------------------ |
    | **基本语法** | `typedef 原类型 新名称` | `using 新名称 = 原类型`   |
    | **可读性**   | 较难（尤其是函数指针）  | 更直观                    |
    | **模板支持** | 不支持模板别名          | **支持模板别名**（C++11） |
    | **C++推荐**  | 传统方式                | 推荐使用                  |

## 命名空间的高级特性

### 命名空间的嵌套

命名空间可以嵌套定义，形成层次结构。

!!! example "嵌套命名空间"

    ``` cpp linenums="1"  hl_lines="31 34 35 37"
    #include <iostream>
    using namespace std;

    namespace Company {
        namespace Finance {
            double calculateTax(double income) {
                return income * 0.2;
            }
        }

        namespace HR {
            struct Employee {
                string name;
                int id;
            };
            void printEmployee(const Employee& e) {
                cout << e.name << " (ID: " << e.id << ")" << endl;
            }
        }
    }

    // C++17 简化嵌套语法
    namespace Company::IT {
        void printVersion() {
            cout << "Version 1.0" << endl;
        }
    }

    int main() {
        // 访问嵌套命名空间
        double tax = Company::Finance::calculateTax(50000);
        cout << "Tax: " << tax << endl;

        Company::HR::Employee e{"Alice", 1001};
        Company::HR::printEmployee(e);

        Company::IT::printVersion();

        return 0;
    }
    ```

### 命名空间的别名

可以为命名空间定义**别名**，简化长名称的使用。

!!! example "命名空间别名"

    ``` cpp linenums="1"  hl_lines="12 14"
    #include <iostream>
    using namespace std;

    namespace VeryLongNamespaceName_For_Mathematics {
        const double PI = 3.14159;
        double sin(double x) { /* ... */ return x; }
        double cos(double x) { /* ... */ return x; }
    }

    int main() {
        // 使用别名简化
        namespace Math = VeryLongNamespaceName_For_Mathematics;

        cout << "PI = " << Math::PI << endl;

        return 0;
    }
    ```

### 匿名命名空间

**匿名命名空间（Unnamed Namespace）** 中的名称仅在当前编译单元内可见，相当于C语言中的 `static` 修饰。

!!! example "匿名命名空间"

    ``` cpp linenums="1" hl_lines="5"
    #include <iostream>
    using namespace std;

    // 匿名命名空间：仅在当前文件可见
    namespace {
        int internalCounter = 0;

        void internalHelper() {
            cout << "Internal helper called" << endl;
        }
    }

    // 等价于 C 语言的：
    // static int internalCounter = 0;
    // static void internalHelper() { ... }

    int main() {
        // 可以直接访问（在同一编译单元内）
        internalCounter++;
        internalHelper();

        cout << "Counter: " << internalCounter << endl;

        return 0;
    }
    ```

!!! tip "匿名命名空间的优势"

    与C语言的 `static` 相比，匿名命名空间可以包含更复杂的类型（如类、结构体），且语法更一致。

### 命名空间与头文件

!!! danger "头文件中的 using namespace"

    **不要**在头文件中使用 `using namespace`，这会污染所有包含该头文件的源文件。

    ``` cpp
    // bad_header.h
    using namespace std;   // ❌ 危险！会污染所有包含此头文件的文件

    class MyClass {
        // ...
    };
    ```

!!! success "头文件中的正确做法"

    ``` cpp
    // good_header.h
    #include <string>

    class MyClass {
    public:
        void setName(const std::string& name);  // 使用完全限定名
    private:
        std::string m_name;                     // 使用完全限定名
    };
    ```

## std 命名空间

### 什么是 std

C++标准库中的所有功能都定义在 `std` 命名空间中，包括 `cout`、`cin`、`string`、`vector` 等。

!!! example "std 命名空间的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    #include <vector>

    // 方式一：完全限定名（推荐）
    std::cout << "Hello" << std::endl;
    std::string name = "Alice";
    std::vector<int> numbers;

    // 方式二：using 声明
    using std::cout;
    using std::endl;
    cout << "Hello" << endl;

    // 方式三：using namespace（谨慎使用）
    using namespace std;   // 简单程序中常见，大项目中应避免
    cout << "Hello" << endl;
    ```

### 使用建议

!!! tip "std 命名空间的使用建议"

    | 场景                | 建议                                       |
    | :------------------ | :----------------------------------------- |
    | **头文件**          | 使用完全限定名（`std::string`）            |
    | **实现文件（cpp）** | 可以小范围使用 `using` 声明                |
    | **简单程序**        | 可以使用 `using namespace std`             |
    | **大型项目**        | 避免 `using namespace std`，使用完全限定名 |
    | **函数内部**        | 可以使用 `using` 声明，影响范围有限        |

## 综合示例

!!! example "命名空间"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    #include <vector>
    using namespace std;

    // === 定义多个命名空间 ===

    // 数学工具
    namespace Math {
        const double PI = 3.1415926535;

        double square(double x) { return x * x; }

        double circleArea(double radius) {
            return PI * radius * radius;
        }

        // 嵌套命名空间
        namespace Advanced {
            double factorial(int n) {
                double result = 1;
                for (int i = 2; i <= n; i++) {
                    result *= i;
                }
                return result;
            }
        }
    }

    // 字符串工具
    namespace String {
        bool isEmpty(const string& s) {
            return s.empty();
        }

        string toUpper(const string& s) {
            string result = s;
            for (char& c : result) {
                if (c >= 'a' && c <= 'z') {
                    c = c - 'a' + 'A';
                }
            }
            return result;
        }
    }

    // 容器工具
    namespace Container {
        template<typename T>
        void print(const vector<T>& v, const string& name = "vector") {
            cout << name << ": [";
            for (size_t i = 0; i < v.size(); i++) {
                cout << v[i];
                if (i < v.size() - 1) cout << ", ";
            }
            cout << "]" << endl;
        }
    }

    // === 匿名命名空间 ===
    namespace {
        int callCount = 0;   // 仅当前文件可见

        void incrementCallCount() {
            callCount++;
        }
    }

    int main() {
        // 1. 使用完全限定名
        cout << "=== 完全限定名 ===" << endl;
        cout << "PI = " << Math::PI << endl;
        cout << "Circle area (r=5): " << Math::circleArea(5) << endl;
        cout << "5! = " << Math::Advanced::factorial(5) << endl;

        // 2. 使用 using 声明
        cout << "\n=== using 声明 ===" << endl;
        using String::toUpper;
        string text = "hello world";
        cout << "Original: " << text << endl;
        cout << "Uppercase: " << toUpper(text) << endl;

        // 3. 使用局部 using namespace
        cout << "\n=== 局部 using namespace ===" << endl;
        {
            using namespace Container;
            vector<int> v = {1, 2, 3, 4, 5};
            print(v, "numbers");
            print(vector<string>{"Alice", "Bob", "Charlie"}, "names");
        }

        // 4. 命名空间别名
        cout << "\n=== 命名空间别名 ===" << endl;
        namespace AdvMath = Math::Advanced;
        cout << "6! = " << AdvMath::factorial(6) << endl;

        // 5. 匿名命名空间
        cout << "\n=== 匿名命名空间 ===" << endl;
        incrementCallCount();
        incrementCallCount();
        cout << "Call count: " << callCount << endl;

        // 6. 命名空间中的名称冲突
        cout << "\n=== 处理名称冲突 ===" << endl;
        // 假设有两个命名空间都有 sqrt 函数
        // 使用完全限定名明确指定
        // double r = Math::sqrt(16);   // 如存在
        // double r2 = AnotherMath::sqrt(16);

        cout << "Math::PI = " << Math::PI << endl;

        return 0;
    }
    ```

    运行结果

    ```
    === 完全限定名 ===
    PI = 3.14159
    Circle area (r=5): 78.5398
    5! = 120

    === using 声明 ===
    Original: hello world
    Uppercase: HELLO WORLD

    === 局部 using namespace ===
    numbers: [1, 2, 3, 4, 5]
    names: [Alice, Bob, Charlie]

    === 命名空间别名 ===
    6! = 720

    === 匿名命名空间 ===
    Call count: 2

    === 处理名称冲突 ===
    Math::PI = 3.14159
    ```

## 小结

1.  **命名空间的作用**：
    - 避免名称冲突。
    - 将相关功能组织在一起。
    - 控制名称的可见范围。
2.  **命名空间的定义**：`namespace 名称 { ... }`
3.  **三种使用方式**：
    - **完全限定名**：`命名空间::名称`（最安全，推荐）。
    - **using 声明**：`using 命名空间::名称`（导入单个名称）。
    - **using 指令**：`using namespace 命名空间`（导入所有名称，谨慎使用）。
4.  **高级特性**：
    - 嵌套命名空间。
    - 命名空间别名（简化长名称）。
    - 匿名命名空间（当前文件内可见）。
5.  **头文件中的规则**：**不要**在头文件中使用 `using namespace`。
6.  **std 命名空间**：C++标准库所有功能都在 `std` 中。