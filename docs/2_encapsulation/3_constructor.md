# 对象初始化与生命周期

本节围绕对象的构造、复制、移动和析构，深入探讨C++如何管理对象的完整生命周期。

## 对象的初始化

对象不仅要“出生”，还要“健康地出生”。在程序设计中，“初始化”指的是为对象赋予初始状态的过程。不同的语言和编程范式提供了不同的初始化方式。

### C语言的初始化方式：程序员的责任

在C语言中，结构体变量的初始化完全依赖程序员。

!!! example "C语言中的结构体初始化"

    ``` c linenums="1"
    typedef struct {
        int hour;
        int minute;
        int second;
    } Clock;

    // 方式一：声明时初始化（需要知道字段顺序）
    Clock c1 = {8, 30, 0};

    // 方式二：声明后逐个赋值
    Clock c2;
    c2.hour = 14;
    c2.minute = 5;
    c2.second = 30;

    // 方式三：部分初始化（未指定的字段为零值）
    Clock c3 = {10};   // hour=10, minute=0, second=0
    ```

!!! warning "C风格初始化的隐患"

    - **遗忘风险**：定义变量后忘记初始化，成员包含垃圾值，后续使用导致不可预测的行为。
    - **规则分散**：初始化逻辑散落在各处，修改业务规则（如时间范围）时需要追踪所有初始化点。
    - **维护困难**：当结构体新增字段时，所有已有的初始化代码都可能需要修改。

    ``` c linenums="1"
    Clock c;        // 未初始化，hour/minute/second 的值是随机的
    showClock(&c);  // 可能显示 16:73568:-129，行为未定义！
    ```

### 对象的自动初始化：编译器提供的默认值

C++在C的基础上迈出了第一步：允许在声明变量或对象时由编译器自动赋予默认值，但这一机制仅适用于**全局变量**和**静态变量**。

!!! example "自动初始化的局限性"

    ``` cpp linenums="1"
    class Clock {
    public:
        int hour;
        int minute;
        int second;
    };

    Clock globalClock;        // 全局对象：hour=0, minute=0, second=0（自动初始化）

    int main() {
        Clock localClock;     // 局部对象：hour, minute, second 未初始化（垃圾值）
        static Clock staticClock;  // 静态对象：hour=0, minute=0, second=0（自动初始化）
        return 0;
    }
    ```

!!! warning "自动初始化的不足"

    - **不统一**：自动初始化只适用于全局和静态对象，局部对象依然处于未初始化状态。
    - **不可定制**：自动初始化只提供零值（0、nullptr、false），无法指定业务上有意义的默认状态（例如时间默认为 `00:00:00`）。
    - **保护性差**：即使公共成员被初始化为零值，外部代码仍可随意修改为非法值。

### 类内初始值：C++11的统一默认值

C++11引入了**类内初始值（In-class Initializer）**，允许在声明数据成员时直接指定默认值。这为所有对象提供了一致的默认状态。

!!! example "使用类内初始值"

    ``` cpp linenums="1"
    class Clock {
    public:
        void show() const;
    private:
        int hour {0};     // 默认值为 0
        int minute {0};   // 默认值为 0
        int second {0};   // 默认值为 0
    };

    int main() {
        Clock c1;         // hour=0, minute=0, second=0
        Clock c2;         // hour=0, minute=0, second=0
        // 每个对象都有一致的默认状态
        return 0;
    }
    ```

!!! info "类内初始值的优势"

    - **统一默认状态**：所有对象的默认状态由类本身定义，不再依赖外部代码。
    - **声明即文档**：在成员声明处就能看到默认值，代码更易读。
    - **减少遗漏**：即使没有显式初始化，成员也有合理初值。
    - **消除“未初始化”状态**：任何对象（无论全局还是局部）都从合法默认值开始。

!!! tip "类内初始值的适用场景"

    类内初始值最适合表达“业务上的默认状态”，例如：

    - `Clock` 的默认时间是 `00:00:00`
    - `Student` 的默认成绩列表为空
    - `Counter` 的默认计数为 `0`

### 类内初始值的局限性

尽管类内初始值解决了“默认状态”的问题，但它仍然存在不足：

!!! warning "类内初始值无法解决的问题"

    - **只能提供一种默认状态**：无法在创建对象时指定不同的初始值。
    - **无法处理复杂初始化逻辑**：无法根据参数计算初始值，也无法进行合法性检查。
    - **无法表达“必须由外部提供”的成员**：有些成员状态必须在对象创建时由外部传入，类内初始值无法强制要求。

    ``` cpp linenums="1"
    class Student {
    private:
        string name {""};     // 默认姓名为空
        string id {""};       // 默认学号为空
        // 问题：空姓名和空学号在业务上可能是非法的，
        // 但类内初始值无法表达“姓名和学号必须在创建Student类型的对象时提供”这个规则
    };
    ```

!!! question "思考：如何让对象在创建时强制指定姓名和学号？"

    类内初始值提供了默认状态，但无法强制要求外部提供初始化信息。这就需要**构造函数**登场了。

## 构造函数

构造函数（Constructor）是面向对象程序设计语言为解决对象初始化问题而设计的核心机制。
它让初始化过程成为对象创建的一部分，实现了“对象出生即有效”的目标。

### 为什么需要构造函数

类内初始值虽然好，但它只能提供一种默认状态，无法满足更丰富的初始化需求：

1. **需要强制提供初始化参数**：有些对象必须在创建时传入特定信息（如学生的姓名和学号）。
2. **需要执行初始化逻辑**：根据参数计算初始值，或进行合法性检查。
3. **需要多种初始化方式**：同一个类可能需要支持多种创建方式（如默认时间、指定时间、从字符串解析时间）。

构造函数正是为解决这些问题而设计的：

- 构造函数在对象创建时**自动执行**，无法遗忘。
- 构造函数可以接收参数，支持灵活的初始化方式。
- 构造函数可以包含检查逻辑，**阻止非法对象产生**。

!!! note "核心理念"

    设计良好的类应该做到“**对象出生即有效**”。对象的创建者不应再额外记住调用某个 `init()` 函数来执行对象初始化操作。

### 构造函数的基本形式

构造函数是一种特殊的成员函数，其名称与类名相同，且没有返回类型。

!!! example "构造函数的声明"

    ``` cpp linenums="1"
    class Clock {
    public:
        Clock();                      // 默认构造函数
        Clock(int h, int m, int s);   // 带参数的构造函数
    private:
        int hour, minute, second;
    };
    ```

!!! info "构造函数的语法要点"

    - **名称与类名相同**：这是编译器识别构造函数的依据。
    - **没有返回类型**：`void` 也不写，构造函数不返回值。
    - **可以重载**：一个类可以声明多个构造函数，通过参数列表的不同来区分。
    - **自动调用**：在创建对象时，编译器会根据参数自动匹配并调用相应的构造函数。

### 默认构造函数

默认构造函数（Default Constructor）也是无参构造函数，不需要传入任何参数就可被调用。它负责创建对象的默认状态。

!!! example "默认构造函数的几种写法"

    === "方式一：编译器自动生成"

        ``` cpp linenums="1"

        class Clock {

        // 如果用户未声明任何构造函数，编译器会自动生成默认构造函数
        // 为避免生成不确定状态的对象，可通过类内初始值为对象指定初始状态

        private:
            int hour {0};
            int minute {0};
            int second {0};
        };

        Clock c;   // 编译器自动生成默认构造函数，对象初始化为 00:00:00

        ```

    === "方式二：使用 `= default`"

        ``` cpp linenums="1"

        class Clock {
        public:
            Clock() = default;   // 明确要求编译器生成默认版本
        private:
            int hour {0};
            int minute {0};
            int second {0};
        };

        Clock c;   // 调用默认构造函数，时间为 00:00:00

        ```

    === "方式三：用户自定义无参构造函数"

        ``` cpp linenums="1"
        class Clock {
        public:
            Clock() {
                hour = 0;
                minute = 0;
                second = 0;
            }
        private:
            int hour, minute, second;
        };
        ```

!!! warning "重要规则：构造函数的自动生成"

    - 任何一个类都至少具有一个构造函数。
    - 如果程序员**没有在类定义中声明任何构造函数**，编译器会**自动生成**一个默认构造函数。
    - 如果程序员**显式声明了任一构造函数**（无论是否有参数），编译器就**不再自动生成**默认构造函数，除非通过 `= default` 明确要求。

    !!! danger "常见错误：默认构造函数不存在"

        ``` cpp linenums="1"
        class Clock {
        public:
            Clock(int h, int m, int s);  // 显示声明了带参构造函数
        private:
            int hour, minute, second;
        };

        Clock c;   // 错误！由于编译器不再生成默认构造函数，匹配不到合适的构造函数
        ```

    !!! tip "最佳实践"

        如果一个类需要有多种构造方式，建议**显式声明默认构造函数**（使用 `= default`），确保默认构造方式始终可用。

### 带参数的构造函数

带参数的构造函数允许在创建对象时直接指定初始状态，使对象的初始化更加灵活。

!!! example "带参数的构造函数"

    ``` cpp linenums="1"

    class Clock {
    public:
        Clock(int h, int m, int s);   // 声明构造函数
    private:
        int hour, minute, second;
    };
    // 实现：使用初始化列表
    Clock::Clock(int h, int m, int s) 
    : hour(h), minute(m), second(s) {
        // 初始化列表先于构造函数体执行
    }
    // 使用
    Clock c1(8, 30, 0);     // 08:30:00
    Clock c2(23, 59, 59);   // 23:59:59

    ```

!!! info "初始化列表（Initializer List）"

    - 初始化列表位于函数参数列表之后，函数体之前，以冒号 `:` 开头。
    - 多个成员的初始化用逗号 `,` 分隔。
    - **初始化列表先于构造函数体执行**，在成员变量被创建时直接赋予初值。
    - 在C++的编程实践中，推荐优先使用初始化列表而不是在函数体内赋值，因为初始化更高效，且是某些场景（如 `const` 成员、引用成员）的唯一方式。

#### 初始化列表与函数体赋值的区别

!!! example "对比示例"

    === "方式一：初始化列表（推荐）"

        ``` cpp linenums="1"
        Clock::Clock(int h, int m, int s)
            : hour(h), minute(m), second(s) {
            // 成员直接用 h/m/s 构造，一步完成
        }
        ```

    === "方式二：函数体赋值（不推荐）"

        ``` cpp linenums="1"
        Clock::Clock(int h, int m, int s) {
            hour = h;     // hour 先被默认构造（或未初始化），再被赋值
            minute = m;
            second = s;
        }
        ```

理解这两者的区别，是写出高效且正确C++代码的关键。

|              | **初始化列表**                                   | **函数体赋值**                   |
| :----------- | :----------------------------------------------- | :------------------------------- |
| **执行时机** | 成员变量创建时（更早）                           | 成员变量创建之后（更晚）         |
| **执行过程** | 直接用初值构造成员                               | 先默认构造（或未初始化），再赋值 |
| **效率**     | 更高（一步到位）                                 | 稍低（两步操作）                 |
| **适用场景** | `const` 成员、引用成员、无默认构造函数的成员对象 | 普通成员变量                     |

!!! tip "最佳实践"

    除非有特殊原因，否则应该**优先使用初始化列表**初始化所有成员变量。

#### 相互冲突的构造函数

如果在类定义中，同时出现**参数列表为空**的构造函数和**全部参数都有默认值**的构造函数，则会产生编译错误。

!!! example "冲突的构造函数"

    ``` cpp linenums="1" hl_lines="13"

    class Clock {
    public:
        Clock() = default;
        Clock(int h = 0, int m = 0, int s = 0);
    private:
    int hour, minute, second;
    };

    Clock::Clock(int h, int m, int s)
        : hour(0), minute(0), second(0) {
    }

    Clock c;     // 编译器无法区分具体要调用哪个构造函数
    ```

### 构造函数中的合法性检查

构造函数的另一个重要职责是**阻止非法对象产生**。在构造函数中，我们可以通过调用其他成员函数或编写显式检查逻辑来确保对象状态合法。

!!! example "在构造函数中进行合法性检查"

    ``` cpp linenums="1"
    class Clock {
    public:
        Clock(int h, int m, int s)
            : hour(0), minute(0), second(0) {   // 先设置为安全的默认值
            setTime(h, m, s);   // 复用 setTime 的检查逻辑
        }

        bool setTime(int h, int m, int s) {
            if (h < 0 || h >= 24) return false;
            if (m < 0 || m >= 60) return false;
            if (s < 0 || s >= 60) return false;
            hour = h;
            minute = m;
            second = s;
            return true;
        }
    private:
        int hour, minute, second;
    };
    ```

!!! note "设计策略"

    - **先给安全默认值**：即使传入非法参数，对象也不会处于未定义状态。
    - **复用业务规则**：构造函数调用 `setTime()` 复用检查逻辑，避免代码重复。
    - **返回结果处理**：如果构造函数无法构造合法对象，也可以考虑抛出异常（后续章节会涉及）。

### 委托构造函数（C++11）

委托构造函数（Delegating Constructor）允许一个构造函数调用同类的另一个构造函数，从而减少重复代码。

!!! example "委托构造函数"

    ``` cpp linenums="1"
    class Clock {
    public:
        // 无参构造函数委托给带参构造函数
        Clock() : Clock(0, 0, 0) {}

        // 带参构造函数包含完整的初始化逻辑
        Clock(int h, int m, int s) : hour(h), minute(m), second(s) {}

    private:
        int hour, minute, second;
    };
    ```

!!! info "委托构造函数的优点"

    - **减少重复**：所有初始化逻辑集中在一个构造函数中。
    - **规则集中**：修改初始化规则时只需修改被委托的构造函数。
    - **提高可维护性**：代码更简洁，逻辑更清晰。

### 构造函数的调用时机

构造函数在对象创建时被**自动隐式调用**，通常无需程序员手动调用。

!!! example "构造函数的调用方式"

    ``` cpp linenums="1"
    int main() {
        Clock c1;                // 调用默认构造函数（无参）
        Clock c2(8, 30, 0);      // 调用带参构造函数

        // Clock c3();           // 错误！这不是对象定义，而是函数声明

        Clock c4 = Clock(14, 5, 30);  // 先构造临时对象，再拷贝给 c4（不推荐，实际可能被优化）

        return 0;
    }
    ```

!!! warning "容易犯错的语法"

    - `Clock c1;` ✓ 调用默认构造函数。
    - `Clock c1()` ✗ 这被编译器解析为**函数声明**，而不是对象定义。

## 复制构造函数

复制构造函数（Copy Constructor）用已有对象创建一个新对象，新对象与原对象具有相同或等价的状态。

### 复制构造函数的定义

复制构造函数的参数是**当前类类型的 const 引用**。

!!! example "Point 类的复制构造函数"

    ``` cpp linenums="1" hl_lines="6-7 18"
    class Point {
    public:
        // 普通构造函数
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 复制构造函数
        Point(const Point& other) : x(other.x), y(other.y) {}

        void show() const {
            cout << "(" << x << ", " << y << ")";
        }
    private:
        int x, y;
    };

    // 使用复制构造函数
    Point p1(3, 4);
    Point p2(p1);   // 调用复制构造函数，p2 成为 p1 的副本
    p1.show();      // (3, 4)
    p2.show();      // (3, 4)
    ```

!!! info "复制构造函数的特征"

    - **参数**：必须是 `const 类名&`，使用引用 `&` 避免无限递归，使用 `const` 表示不修改源对象。
    - **访问权限**：同一个类的成员函数可以访问同类其他对象的私有成员（`other.x` 是合法访问）。
    - **默认行为**：如果程序员未定义复制构造函数，编译器会自动生成一个“浅拷贝”版本。

### 复制构造函数被调用的三种场景

复制构造函数更容易被隐式调用。理解复制构造函数的调用时机，比死记复制构造函数的定义语法更重要。

!!! example "场景一：用已有对象初始化新对象"

    ``` cpp linenums="1"
    Point p1(3, 4);
    Point p2(p1);    // 调用复制构造函数
    Point p3 = p1;   // 也调用复制构造函数（注意：这是初始化，不是赋值）
    ```

!!! example "场景二：函数参数传值"

    ``` cpp linenums="1"
    void printPoint(Point p) {   // 传值参数，实参拷贝给形参时调用复制构造函数
        p.show();
    }

    Point p1(3, 4);
    printPoint(p1);   // 调用复制构造函数（实参 p1 拷贝到形参 p）
    ```

!!! example "场景三：函数返回对象"

    ``` cpp linenums="1"
    Point createPoint() {
        Point p(5, 6);
        return p;     // 返回时可能调用复制构造函数（实际可能被返回值优化 RVO 优化掉）
    }

    Point p2 = createPoint();   // 可能调用复制构造函数
    ```

### 缺省复制构造函数

每个类可以有一个复制构造函数。如果没有为类显式定义复制构造函数，编译器会自动生成一个**缺省复制构造函数（Default Copy Constructor）**。
它的行为是**逐个成员复制（Memberwise Copy）**——依次复制每个数据成员的值。对于大多数**只包含简单数据类型成员**的类，缺省复制构造函数就足够了：

!!! example "样例一"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 如果未明确定义复制构造函数，编译器自动生成缺省复制构造函数，等价于：
        // Point(const Point& other) : x(other.x), y(other.y) {}

    private:
        int x, y;
    };

    Point p1(3, 4);
    Point p2(p1);   // 调用编译器生成的缺省复制构造函数
                    // p2.x = p1.x = 3, p2.y = p1.y = 4
    ```

!!! example "样例二"

    ``` cpp linenums="1"
    class Clock {
    public:
        Clock(int h, int m, int s) : hour(h), minute(m), second(s) {}

        // 缺省复制构造函数足够，逐字节复制三个 int 即可

    private:
        int hour, minute, second;
    };

    Clock c1(8, 30, 0);
    Clock c2(c1);   // c2 成为 c1 的独立副本
    ```

!!! warning "缺省复制构造函数的局限性"

    当类包含**指针成员**或**管理动态资源**时，缺省复制构造函数执行的是**浅拷贝（Shallow Copy）**——只复制指针值（地址），而不复制指针所指向的内容。

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}
        // 使用编译器生成的缺省复制构造函数
        // ~Buffer() { delete[] data; }
    private:
        int* data;
        int sz;
    };

    Buffer b1(10);
    Buffer b2(b1);   // 缺省复制：b2.data = b1.data（指向同一块内存）
                     // 析构时：b1 和 b2 都尝试 delete[] 同一块内存 → 重复释放（Double Free）！
    ```

    !!! danger "浅拷贝导致的经典问题：重复释放"

        当两个对象共享同一块动态内存时，对象销毁时会重复释放同一块内存，导致程序崩溃或未定义行为。

        ``` cpp

        // 错误示意
        b1 构造: data → [0x1000] (分配内存)
        b2 复制: data → [0x1000] (复制指针，指向同一块内存)
        b1 析构: delete[] 0x1000 (释放内存)
        b2 析构: delete[] 0x1000 (再次释放同一内存 → 错误！)

        ```

    !!! info "关于浅拷贝与深拷贝的问题，在后续章节中还将进一步讨论"

!!! note "设计决策：何时需要自定义复制构造函数？"

    - **不需要自定义**：类只包含基本类型成员（`int`、`double`、`char`等）或标准库容器（`string`、`vector`等），缺省复制构造函数行为正确。
    - **需要自定义**：类直接管理动态内存（`new`/`delete`）、文件句柄、网络连接等资源，需要执行**深拷贝（Deep Copy）**来复制资源内容。

### = delete：禁止复制

如果有些对象不允许被复制（例如管理文件句柄、网络连接等独占资源），可以显式禁止复制操作。

!!! example "禁止复制的类"

    ``` cpp linenums="1" hl_lines="5"
    class UniqueFile {
    public:
        UniqueFile(const string& filename);
        // 禁止复制构造
        UniqueFile(const UniqueFile&) = delete;
        // 禁止复制赋值
        UniqueFile& operator=(const UniqueFile&) = delete;
        ~UniqueFile();
    private:
        FILE* handle;
    };

    UniqueFile f1("data.txt");
    // UniqueFile f2(f1);   // 错误！复制构造函数被 delete
    // UniqueFile f2 = f1;  // 错误！赋值操作符也被 delete
    ```

!!! info "设计意图"

    使用 `= delete` 可以阻止编译器自动生成缺省复制构造函数，从而在编译期阻止非法复制操作，
    清晰地表达类型的设计意图——“这个类型不允许复制”。

## 移动构造函数

### 左值、右值与移动语义

在理解移动构造函数之前，需要先建立对左值和右值的直觉认识。

!!! info "左值与右值（初步概念）"

    - **左值（Lvalue）**：有持久身份、有明确内存地址的表达式。可以出现在赋值号左边。
    - **右值（Rvalue）**：临时对象或即将销毁的值。通常出现在赋值号右边。

    ``` cpp linenums="1"
    int a = 10;      // a 是左值，10 是右值
    int b = a;       // b 是左值，a 是左值（但可以读）
    int c = a + b;   // a + b 是右值（临时结果）
    ```

!!! note "移动语义的直觉"

    移动语义（Move Semantics）利用右值“即将消亡”的特点，将其资源直接“转移”给新对象，而不是重新复制一份。

    想象一下搬家：**复制**是把所有家具重新买一份；**移动**是把现有家具直接搬到新家，旧房子里的家具清空。

### 移动构造函数的定义

移动构造函数（Move Constructor）用于从即将消亡的对象中“窃取”资源，避免昂贵的复制开销。

!!! example "移动构造函数的定义"

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}

        // 移动构造函数
        Buffer(Buffer&& other) noexcept
            : data(other.data), sz(other.sz) {
            other.data = nullptr;   // 源对象不再拥有资源
            other.sz = 0;
        }

        ~Buffer() {
            delete[] data;   // data 为 nullptr 时 delete 是安全的
        }

    private:
        int* data;
        int sz;
    };
    ```

!!! info "移动构造函数的关键特征"

    - **参数**：`类名&&`——**右值引用**，表示参数是一个即将消亡的临时对象。
    - **操作**：直接“接管”源对象的资源，而不是复制。
    - **源对象处理**：将源对象的指针置空，避免析构时重复释放。
    - **`noexcept`**：标记为不抛出异常，使标准容器在扩容时优先使用移动而非复制，提高性能。

## 析构函数

析构函数（Destructor）在对象生命周期结束时**自动调用**，负责清理对象占用的资源。

!!! example "析构函数的定义"

    ``` cpp linenums="1"
    class FileGuard {
    public:
        FileGuard(const string& filename) {
            file = fopen(filename.c_str(), "r");
        }

        ~FileGuard() {   // 析构函数
            if (file) {
                fclose(file);
                cout << "File closed automatically" << endl;
            }
        }

    private:
        FILE* file;
    };

    // 使用
    void processData() {
        FileGuard fg("data.txt");
        // ... 处理文件数据
    }   // fg 离开作用域，析构函数自动调用，文件被关闭
    ```

!!! info "析构函数的特征"

    - **名称**：类名前加波浪号 `~`，无参数，无返回类型。
    - **唯一性**：一个类只有一个析构函数，不能重载。
    - **自动调用**：局部对象离开作用域、动态对象被 `delete` 时自动调用。
    - **资源释放**：常用于释放动态内存、关闭文件、断开网络连接等清理工作。

### 缺省析构函数

如果没有为类显式定义析构函数，编译器会自动生成一个**缺省析构函数（Default Destructor）**。
它的行为是**按成员依次析构（Memberwise Destruct）**——按照成员声明顺序的逆序，依次调用每个数据成员的析构函数。
对于大多数**只包含基本类型成员**或**成员对象会自动管理资源**的类，缺省析构函数就足够了

!!! example "样例一"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 编译器自动生成缺省析构函数，等价于：
        // ~Point() {}

    private:
        int x, y;
    };

    void func() {
        Point p(3, 4);
        // p 离开作用域时，调用编译器生成的缺省析构函数
        // 对于基本类型 int，析构函数体为空，不需要做任何清理
    }
    ```

!!! example "样例二"

    ``` cpp linenums="1"
    class Student {
    public:
        Student(string name, string id) : name(name), id(id) {}
        // 缺省析构函数足够，string 的析构函数会自动释放内部内存
    private:
        string name;
        string id;
    };

    void func() {
        Student s("张三", "2024001");
        // s 离开作用域时，缺省析构函数自动调用 name 和 id 的析构函数
        // string 的析构函数会自动释放其管理的字符数组内存
    }
    ```

!!! info "缺省析构函数的工作流程"

    编译器生成的缺省析构函数执行以下操作：

    1. **执行函数体**：缺省析构函数的函数体为空（不做任何显式操作）。
    2. **析构数据成员**：按照**声明顺序的逆序**，依次调用每个非静态数据成员的析构函数。
    3. **析构基类**：如果类有基类，在成员析构之后，再逆序调用基类的析构函数。

    对于基本类型（`int`、`double`、指针等），析构操作不做任何事情。

!!! warning "缺省析构函数的局限性"

    当类**直接管理动态资源**（通过 `new` 分配的内存、`fopen` 打开的文件、网络连接等）时，缺省析构函数无法自动释放这些资源，因为它不知道如何释放自定义资源。

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}
        // 使用编译器生成的缺省析构函数
        // 没有自定义析构函数释放 data
    private:
        int* data;
        int sz;
    };

    void func() {
        Buffer b(10);
        // b 离开作用域时，缺省析构函数只销毁指针变量 data 本身（4/8字节），
        // 但不释放 data 所指向的动态内存 → 内存泄漏！
    }
    ```

    !!! danger "缺省析构函数导致的内存泄漏"

        ```
        // 错误示意
        b 构造: data → [0x1000] (new 分配了 10 个 int 的内存)
        b 析构: 缺省析构函数体为空，仅销毁 data 指针变量本身
        [0x1000] 的内存未被释放 → 内存泄漏！

    ```

!!! note "设计决策：何时需要自定义析构函数？"

    这个决策与"**三/五法则（Rule of Three/Five）**"密切相关：

    - **不需要自定义**：类只包含基本类型成员或标准库容器（`string`、`vector`等），缺省析构函数行为正确。
    - **需要自定义**：类直接管理动态内存（`new`/`delete`）、文件句柄、网络连接等资源时，需要自定义析构函数来释放资源。

    !!! tip "三/五法则（Rule of Three/Five）"

        如果一个类需要自定义析构函数，那么它通常也需要自定义复制构造函数和复制赋值运算符（以及C++11后的移动构造函数和移动赋值运算符）。反之亦然。

        这是因为需要自定义析构函数通常意味着类管理着某种资源，而资源的正确管理需要复制/移动操作也遵循相同的语义。

        ``` cpp linenums="1"
        class Buffer {
        public:
            Buffer(int size) : data(new int[size]), sz(size) {}
            ~Buffer() { delete[] data; }  // 自定义析构函数

            // 三/五法则：需要同时自定义复制控制成员
            Buffer(const Buffer& other);           // 复制构造
            Buffer& operator=(const Buffer& other); // 复制赋值
            Buffer(Buffer&& other) noexcept;        // 移动构造（C++11）
            Buffer& operator=(Buffer&& other) noexcept; // 移动赋值（C++11）
        private:
            int* data;
            int sz;
        };
        ```

!!! info "关于缺省析构函数的访问权限"

    缺省析构函数与缺省构造函数类似，具有 `public` 访问权限。但以下情况会导致缺省析构函数被隐式删除：

    - 如果类的任何数据成员或基类具有 `private` 或 `deleted` 的析构函数，编译器会隐式删除缺省析构函数。
    - 这体现了C++的设计原则：**类的可析构性由其成员共同决定**。

    ``` cpp linenums="1"
    class NonDestructible {
    private:
        ~NonDestructible() {}   // 私有析构函数
    };

    class MyClass {
    private:
        NonDestructible nd;     // 成员无法被外部析构
        // MyClass 的缺省析构函数被隐式删除！
    };

    // MyClass obj;   // 错误！无法析构 obj
    ```

### 自定义析构函数

当类管理资源时，需要自定义析构函数来正确释放资源。

!!! example "自定义析构函数释放动态内存"

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}

        ~Buffer() {   // 自定义析构函数
            delete[] data;
            cout << "Buffer memory released" << endl;
        }

    private:
        int* data;
        int sz;
    };

    void func() {
        Buffer b(10);
        // ... 使用 b
    }   // b 离开作用域，自定义析构函数自动调用，释放 data 指向的内存
    ```

!!! example "自定义析构函数关闭文件句柄"

    ``` cpp linenums="1"
    class FileGuard {
    public:
        FileGuard(const string& filename) {
            file = fopen(filename.c_str(), "r");
            if (!file) {
                throw runtime_error("Cannot open file");
            }
        }

        ~FileGuard() {   // 自定义析构函数
            if (file) {
                fclose(file);
                cout << "File closed automatically" << endl;
            }
        }

    private:
        FILE* file;
    };
    ```

!!! summary "缺省析构函数与自定义析构函数要点回顾"

    | 类型               | 来源           | 行为                       | 适用场景                         |
    | :----------------- | :------------- | :------------------------- | :------------------------------- |
    | **缺省析构函数**   | 编译器自动生成 | 按成员逆序析构，函数体为空 | 类不管理资源，或成员自行管理资源 |
    | **自定义析构函数** | 用户定义       | 释放类管理的资源           | 类直接管理动态内存、文件、连接等 |

## 构造与析构：描述完整的对象生命周期

理解对象什么时候被创建、转移和销毁，是写出可靠 C++ 代码的关键。其核心问题是：

- 这个对象何时有效？（构造）
- 谁拥有资源？（复制/移动的语义）
- 离开时如何收尾？（析构）

!!! summary "对象生命周期全景图"

    | 阶段     | 操作         | 说明                       |
    | :------- | :----------- | :------------------------- |
    | **出生** | 构造函数     | 获得初始状态，对象开始有效 |
    | **复制** | 复制构造函数 | 创建等价的新对象           |
    | **转移** | 移动构造函数 | 转移资源，源对象变为空壳   |
    | **消亡** | 析构函数     | 自动收尾，释放资源         |

## 小结

构造函数和析构函数是 C++ 管理对象生命周期的核心机制。

1. **对象初始化**从 C 风格的手动初始化，发展到 C++ 的类内初始值，再到功能完备的构造函数，逐步提升了初始化的安全性和表达能力。
2. **构造函数**确保对象“出生即有效”，通过初始化列表高效初始化成员，通过合法性检查阻止非法状态。
3. **复制构造函数**定义对象的复制语义，对于管理资源的类需要自定义深拷贝。
4. **移动构造函数**通过转移资源避免不必要的复制，提升性能。
5. **析构函数**在对象销毁时自动释放资源，是 RAII 惯用法的基础。

掌握这些基础，才能写出健壮、高效、可维护的 C++ 代码。