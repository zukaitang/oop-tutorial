# 枚举类型

枚举是C++中用于定义一组命名常量的机制。C++在继承C语言传统枚举的基础上，引入了更安全、更可控的**作用域枚举（enum class）**，
有效解决了传统枚举存在的问题。

## 枚举的基本概念

**枚举（Enumeration）** 是一种用户自定义的数据类型，用于定义一组相关的**命名常量**。枚举使代码更可读、更易维护。

!!! example "枚举的基本用途"

    ``` cpp linenums="1" hl_lines="5-7 10-12"
    #include <iostream>
    using namespace std;

    // 定义枚举类型：表示星期
    enum Weekday {
        Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday
    };

    // 定义枚举类型：表示颜色
    enum Color {
        Red, Green, Blue
    };

    int main() {
        Weekday today = Wednesday;
        Color bg = Blue;

        cout << "Today is: " << today << endl;   // 输出对应的整数值
        cout << "Background color: " << bg << endl;

        return 0;
    }
    ```

!!! abstract "枚举的核心作用"

    | 作用             | 说明                                    |
    | :--------------- | :-------------------------------------- |
    | **提高可读性**   | `Color c = Red` 比 `int c = 0` 更易理解 |
    | **限制取值范围** | 枚举变量只能取枚举中定义的值            |
    | **减少魔法数字** | 避免使用 0、1、2 等含义不明的数值       |
    | **便于维护**     | 修改枚举值只需在一处修改                |

## C语言风格的传统枚举

### 传统枚举的使用与赋值规则

C语言风格的枚举（在C++中仍然支持）使用 `enum` 关键字定义：

!!! example "传统枚举的定义"

    ``` cpp linenums="1" hl_lines="6-11 14-19 22-27"

    # include <iostream>

    using namespace std;

    // 定义枚举：默认从 0 开始
    enum Season {
        Spring,   // 0
        Summer,   // 1
        Autumn,   // 2
        Winter    // 3
    };

    // 手动指定枚举值
    enum HttpCode {
        OK = 200,
        NotFound = 404,
        ServerError = 500,
        Custom = 600
    };

    // 部分赋值 + 自动递增
    enum Permission {
        Read = 1,
        Write = 2,
        Execute = 4,
        All = Read | Write | Execute   // 7
    };

    int main() {
        Season s = Summer;
        HttpCode code = OK;
        Permission p = All;

        cout << "Season: " << s << endl;          // 1
        cout << "HttpCode: " << code << endl;     // 200
        cout << "Permission: " << p << endl;      // 7

        return 0;
    }
    ```

!!! info "枚举值的默认赋值规则"

    1. **默认从 0 开始**：第一个枚举值默认赋值为 0。
    2. **依次递增**：后续枚举值在前一个基础上加 1。
    3. **可手动赋值**：可以为任意枚举值指定整数值。
    4. **未指定的值自动递增**：手动赋值后的值自动递增。

### 传统枚举的问题

C语言风格的传统枚举虽然方便，但存在几个严重的问题，C++引入了作用域枚举来解决这些问题。

#### 问题一：枚举值暴露在外部作用域

传统枚举的枚举值会**泄漏到外部作用域**，可能导致命名冲突。

!!! example "枚举值命名冲突"

    ``` cpp linenums="1" hl_lines="6 10 12"

    # include <iostream>

    using namespace std;

    enum Color {
        Red, Green, Blue
    };

    enum TrafficLight {
        Red,    // 错误！Red 已经存在
        Yellow,
        Green   // 错误！Green 已经存在
    };
    ```

!!! failure "传统枚举的问题示例"

    ``` cpp
    // 无法编译：Red 和 Green 被重复定义
    enum Color { Red, Green, Blue };
    enum TrafficLight { Red, Yellow, Green };   // 编译错误！
    ```

#### 问题二：隐式转换为整数

传统枚举的值可以**隐式转换为 `int`**，这破坏了类型安全性。

!!! example "隐式转换导致类型不安全"

    ``` cpp linenums="1" hl_lines="13-14 17-18 20 25"
    #include <iostream>
    using namespace std;

    enum Color { Red, Green, Blue };
    enum Direction { Up, Down, Left, Right };

    void processColor(Color c) {
        cout << "Color: " << c << endl;
    }

    int main() {
        // 传统枚举可以隐式转换为整数
        int n = Red;           // ✓ 可以（枚举 → int）
        Color c = 1;           // ✓ 可以（int → 枚举）

        // 不同枚举类型之间可以混用
        Color color = Red;
        Direction dir = Up;

        if (color == dir) {    // ✓ 可以编译！但逻辑错误
            cout << "They are equal!" << endl;
        }

        // 传递错误类型
        processColor(Up);      // ✓ 可以编译！但语义错误

        return 0;
    }
    ```

#### 问题三：不能指定底层类型

传统枚举的底层整数类型由编译器决定，不可控制。

!!! example "传统枚举的大小不确定"

    ``` cpp
    // 传统枚举的大小由编译器决定
    enum Small { A, B, C };      // 通常占 4 字节
    enum CharEnum { X = 127, Y }; // 可能占 4 字节，即使值很小
    ```

## C++11 作用域枚举（enum class）

为了解决传统枚举的上述问题，C++11引入了**作用域枚举（Scoped Enumeration）** ，使用 `enum class` 或 `enum struct` 关键字定义。

!!! abstract "enum class 的设计目标"

    - **限定作用域**：枚举值在枚举类型的作用域内，不会泄漏。
    - **强类型**：不会隐式转换为整数，需要显式转换。
    - **可指定底层类型**：可以控制枚举的内存大小。

### 基本语法与使用

!!! info "enum class 的语法"

    ``` cpp
    enum class 枚举类型名 {
        枚举值1,
        枚举值2,
        // ...
    };
    ```

    访问枚举值时需要使用**作用域运算符** `::`：`枚举类型名::枚举值`

!!! example "enum class 的基本使用"

    ``` cpp linenums="1" hl_lines="5-7 9-13 17 18"
    #include <iostream>
    using namespace std;

    // 定义作用域枚举
    enum class Color {
        Red, Green, Blue
    };

    enum class TrafficLight {
        Red,    // 可以与 Color::Red 共存
        Yellow,
        Green   // 可以与 Color::Green 共存
    };

    int main() {
        // 使用作用域运算符访问枚举值
        Color c = Color::Red;
        TrafficLight t = TrafficLight::Green;

        // cout << c << endl;   // 错误！enum class 不能隐式转换为 int

        // 不同枚举类型之间不能比较
        // if (c == t) { }   // 错误！不同类型不能比较

        return 0;
    }
    ```

### 解决命名冲突

!!! example "enum class 解决命名冲突"

    ``` cpp linenums="1" hl_lines="5-7 9-11 16-18 23 24"
    #include <iostream>
    using namespace std;

    // 使用 enum class：名称不会冲突
    enum class Color {
        Red, Green, Blue
    };

    enum class TrafficLight {
        Red, Yellow, Green   // 与 Color::Red/Green 共存
    };

    // 使用 enum class：函数参数类型安全
    void printColor(Color c) {
        switch (c) {
            case Color::Red:   cout << "Red" << endl; break;
            case Color::Green: cout << "Green" << endl; break;
            case Color::Blue:  cout << "Blue" << endl; break;
        }
    }

    int main() {
        Color c = Color::Red;
        TrafficLight t = TrafficLight::Red;

        printColor(c);
        // printColor(t);   // 错误！类型不匹配

        // 可以共存
        cout << "Color::Red = " << static_cast<int>(Color::Red) << endl;
        cout << "TrafficLight::Red = " << static_cast<int>(TrafficLight::Red) << endl;

        return 0;
    }
    ```

### 指定底层类型

`enum class` 允许指定枚举的**底层类型（Underlying Type）** ，从而控制枚举的内存大小。

!!! info "指定底层类型的语法"

    ``` cpp
    enum class 枚举类型名 : 底层类型 {
        枚举值1,
        枚举值2,
        // ...
    };
    ```

    底层类型可以是：`char`、`short`、`int`、`long`、`unsigned char` 等整数类型。

!!! example "指定底层类型"

    ``` cpp linenums="1" hl_lines="5 12 20"
    #include <iostream>
    using namespace std;

    // 指定使用 unsigned char（1字节）
    enum class Status : unsigned char {
        Pending = 0,
        Approved = 1,
        Rejected = 2
    };

    // 指定使用 short（2字节）
    enum class ErrorCode : short {
        None = 0,
        NotFound = -1,
        Timeout = -2,
        Unknown = -99
    };

    // 指定使用 long long（8字节）
    enum class Flags : unsigned long long {
        Read = 1ULL << 0,
        Write = 1ULL << 1,
        Execute = 1ULL << 2
    };

    int main() {
        cout << "sizeof(Status) = " << sizeof(Status) << endl;       // 1
        cout << "sizeof(ErrorCode) = " << sizeof(ErrorCode) << endl; // 2
        cout << "sizeof(Flags) = " << sizeof(Flags) << endl;         // 8

        return 0;
    }
    ```

### 与整数的互操作

`enum class` 可以与整数互相转换，但需要**显式转换**。

!!! example "显式转换"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    enum class Status : int {
        Pending = 0,
        Active = 1,
        Inactive = 2
    };

    int main() {
        // 枚举 → 整数
        Status s = Status::Active;
        int value = static_cast<int>(s);
        cout << "Status value: " << value << endl;   // 1

        // 整数 → 枚举
        int code = 2;
        Status s2 = static_cast<Status>(code);
        // cout << s2;   // 错误！不能直接输出

        // 用于 switch 语句（自动匹配）
        switch (s) {
            case Status::Pending:  cout << "Pending" << endl; break;
            case Status::Active:   cout << "Active" << endl; break;
            case Status::Inactive: cout << "Inactive" << endl; break;
        }

        return 0;
    }
    ```

## 传统 enum vs enum class

!!! summary "全面对比"

    | 对比项           | 传统 enum                   | enum class（C++11）            |
    | :--------------- | :-------------------------- | :----------------------------- |
    | **枚举值作用域** | 外部作用域（全局/命名空间） | 枚举类型作用域内               |
    | **命名冲突**     | 不同枚举不能有同名值        | 可以同名，通过 `类型名::` 区分 |
    | **隐式转换**     | 可隐式转换为 `int`          | 不能隐式转换                   |
    | **类型安全**     | 弱（可与整数混用）          | 强（不能与整数混用）           |
    | **指定底层类型** | 不支持（C++11之前）         | 支持                           |
    | **前向声明**     | 不支持（C++11之前）         | 支持                           |
    | **Switch 使用**  | 直接使用枚举值              | 需使用 `类型名::` 限定         |
    | **使用场景**     | 简单常量定义                | 类型安全的枚举                 |

!!! tip "使用建议"

    - **优先使用 `enum class`**：在现代C++编程中，优先使用 `enum class`，以获得更好的类型安全性和作用域控制。
    - **何时使用传统 `enum`**：
      - 需要与C代码交互。
      - 枚举值需要隐式转换为整数（如位掩码操作）。
      - 枚举值需要作为数组索引。
      - 遗留代码维护。

## 综合示例

!!! example "作用域枚举综合示例"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // === 传统枚举（保留用于特定场景） ===
    // 位掩码场景：需要隐式转换为整数
    enum FilePermission {
        PERM_READ = 1,
        PERM_WRITE = 2,
        PERM_EXECUTE = 4
    };

    // 与C代码交互时使用
    extern "C" {
        void cFunction(int mode);   // 假设是C函数
    }

    // === 作用域枚举（推荐） ===
    enum class LogLevel {
        Debug,
        Info,
        Warning,
        Error,
        Critical
    };

    enum class Priority {
        Low = 10,
        Medium = 20,
        High = 30,
        Critical = 40
    };

    enum class HttpStatus : unsigned short {
        OK = 200,
        BadRequest = 400,
        NotFound = 404,
        InternalError = 500
    };

    // === 使用 enum class 作为函数参数 ===
    void logMessage(const string& msg, LogLevel level) {
        string levelStr;
        switch (level) {
            case LogLevel::Debug:    levelStr = "DEBUG"; break;
            case LogLevel::Info:     levelStr = "INFO"; break;
            case LogLevel::Warning:  levelStr = "WARNING"; break;
            case LogLevel::Error:    levelStr = "ERROR"; break;
            case LogLevel::Critical: levelStr = "CRITICAL"; break;
        }
        cout << "[" << levelStr << "] " << msg << endl;
    }

    void handleStatus(HttpStatus status) {
        int code = static_cast<int>(status);
        string desc;
        switch (status) {
            case HttpStatus::OK:           desc = "Success"; break;
            case HttpStatus::BadRequest:   desc = "Bad Request"; break;
            case HttpStatus::NotFound:     desc = "Not Found"; break;
            case HttpStatus::InternalError:desc = "Internal Error"; break;
        }
        cout << "HTTP " << code << ": " << desc << endl;
    }

    // === 指定底层类型的枚举 ===
    enum class DeviceState : unsigned char {
        Off = 0,
        Standby = 1,
        Running = 2,
        Error = 255
    };

    int main() {
        cout << "=== enum class 基本使用 ===" << endl;
        LogLevel lvl = LogLevel::Info;
        logMessage("System started", lvl);

        Priority p = Priority::High;
        cout << "Priority value: " << static_cast<int>(p) << endl;

        cout << "\n=== 指定底层类型 ===" << endl;
        cout << "sizeof(DeviceState) = " << sizeof(DeviceState) << endl;

        DeviceState state = DeviceState::Running;
        unsigned char val = static_cast<unsigned char>(state);
        cout << "DeviceState value: " << (int)val << endl;   // 2

        cout << "\n=== 与传统枚举的对比 ===" << endl;

        // 传统枚举：可以隐式转换
        int permissions = PERM_READ | PERM_WRITE;
        cout << "Permissions: " << permissions << endl;   // 3

        // enum class：需要显式转换
        HttpStatus status = HttpStatus::NotFound;
        // int code = status;   // 错误！
        int code = static_cast<int>(status);
        handleStatus(status);

        cout << "\n=== Switch 语句中使用 ===" << endl;
        LogLevel testLevel = LogLevel::Error;
        switch (testLevel) {
            case LogLevel::Debug:    cout << "Debug" << endl; break;
            case LogLevel::Info:     cout << "Info" << endl; break;
            case LogLevel::Warning:  cout << "Warning" << endl; break;
            case LogLevel::Error:    cout << "Error" << endl; break;
            case LogLevel::Critical: cout << "Critical" << endl; break;
        }

        return 0;
    }
    ```

    运行结果

    ```
    === enum class 基本使用 ===
    [INFO] System started
    Priority value: 30

    === 指定底层类型 ===
    sizeof(DeviceState) = 1
    DeviceState value: 2

    === 与传统枚举的对比 ===
    Permissions: 3
    HTTP 404: Not Found

    === Switch 语句中使用 ===
    Error
    ```

## 小结

1.  **传统枚举（`enum`）** ：
    - 枚举值暴露在外部作用域，容易造成命名冲突。
    - 可隐式转换为 `int`，类型不安全。
    - 不能指定底层类型（C++11之前）。

2.  **作用域枚举（`enum class`）** （C++11）：
    - 枚举值在枚举类型作用域内，访问需使用 `类型名::枚举值`。
    - 不能隐式转换为 `int`，需显式 `static_cast`。
    - 可指定底层类型，控制内存大小。
    - 类型安全，推荐在现代C++中使用。

3.  **选择建议**：
    - **优先使用 `enum class`**：类型安全、无命名冲突。
    - **传统 `enum` 适用场景**：位掩码操作、与C代码交互、数组索引、遗留代码。

4.  **底层类型指定**：使用 `enum class 类型名 : 底层类型` 格式，底层类型为整数类型（`char`、`short`、`int`、`long` 等）。