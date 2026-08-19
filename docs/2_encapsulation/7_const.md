# 用const保护数据

`const` 把“不修改”的约定变成可由编译器检查的规则。

## 为什么需要保护数据

在大型程序中，数据经常需要在多个函数、多个对象甚至多个模块之间传递和共享。共享带来了协作的便利，也带来了风险：

!!! danger "共享数据的风险"

    - **意外修改**：一个函数在读取数据时，可能无意中修改了数据，导致其他依赖该数据的函数产生错误结果。
    - **难以追踪**：当数据被意外修改时，需要排查所有可能访问该数据的代码，调试成本极高。
    - **接口模糊**：函数参数没有明确表达“只读”还是“可写”，调用者无法从函数签名中得知数据是否会改变。

!!! abstract "const 的设计目标"

    `const` 把“这个数据不应该被修改”的**约定**，变成了编译器可以强制执行的**规则**。

## 常对象

### 基本概念

**常对象（const Object）** 是指在声明时使用 `const` 关键字修饰的对象。常对象一旦创建并初始化后，其状态（数据成员的值）就**不能再被修改**。
常对象表达了“这个对象当前处于只读状态”的语义。

!!! example "常对象的定义"

    ``` cpp linenums="1" hl_lines="24"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

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
        // 普通对象：可读可写
        Point p1(3, 4);
        p1.move(1, 2);      // OK：可以修改

        // 常对象：只读，不可修改
        const Point p2(5, 6);
        p2.show();          // OK：show() 是 const 成员函数，承诺不修改对象

        // p2.move(1, 1);   // 错误！常对象不能调用可能修改对象的函数

        return 0;
    }
    ```

!!! info "常对象的核心规则"

    - **创建后不可修改**：常对象的数据成员在初始化后不能再被赋值。
    - **只能调用常成员函数**：常对象只能调用被 `const` 修饰的成员函数。
    - **构造函数例外**：构造函数可以修改常对象的数据成员（因为对象还在初始化阶段）。

!!! tip "常对象的适用场景"

    - **保护重要数据**：不希望被修改的配置对象、全局状态等。
    - **传递只读参数**：在函数参数中声明为 `const`，明确表达只读意图。
    - **表示不变状态**：如坐标原点 `const Point ORIGIN(0, 0);`。

## 常成员函数

### 基本概念

**常成员函数（const Member Function）** 是指在函数声明的参数列表之后、函数体之前添加 `const` 关键字的成员函数。它承诺：**不会修改当前对象的数据成员**。

!!! example "常成员函数的声明"

    ``` cpp linenums="1" hl_lines="10-12 15-16"
    class Point {
    public:
        // 普通成员函数：可以修改对象
        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        // 常成员函数：承诺不修改对象
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

        // 常成员函数：getter 通常标记为 const
        int getX() const { return x; }
        int getY() const { return y; }

    private:
        int x, y;
    };
    ```

!!! info "常成员函数的规则"

    - **不能修改数据成员**：常成员函数体中不能修改任何非静态数据成员。
    - **不能调用非 const 成员函数**：常成员函数只能调用其他常成员函数。
    - **可以被常对象和普通对象调用**：常对象只能调用常成员函数；普通对象两者都可调用。
    - **语法位置**：`const` 写在函数参数列表之后，函数体之前。

!!! warning "常成员函数 vs 普通成员函数"

    ``` cpp linenums="1"
    class Point {
    public:
        // 常成员函数：可以同时被 const 和非 const 对象调用
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

        // 普通成员函数：只能被非 const 对象调用
        void set(int newX, int newY) {
            x = newX;
            y = newY;
        }
    };

    int main() {
        Point p1(3, 4);
        p1.show();    // OK：普通对象可以调用常成员函数
        p1.set(5, 6); // OK：普通对象可以调用普通成员函数

        const Point p2(7, 8);
        p2.show();    // OK：常对象只能调用常成员函数
        // p2.set(9, 10); // 错误！常对象不能调用普通成员函数

        return 0;
    }
    ```

### const 成员函数的内部机制

理解 const 成员函数的内部机制，有助于更好地理解其行为。

!!! info "const 成员函数的本质"

    常成员函数中的 `this` 指针是 **指向常对象的指针**（`const T* const this`），而不是普通的 `T* const this`。

    ``` cpp
    // 普通成员函数的 this 类型：T* const this
    void move(int dx, int dy) {
        this->x += dx;   // 可以修改 this 指向的对象
    }

    // 常成员函数的 this 类型：const T* const this
    void show() const {
        cout << this->x; // 只能读取 this 指向的对象
        // this->x = 5;  // 错误：this 指向的是 const 对象
    }
    ```

    因此，常成员函数通过 `this` 指针看到的是一个**只读视图**，自然无法修改对象状态。

### 常函数的重载：区分读写语境

同一个函数名可以通过 `const` 重载，让**非 const 对象**和 **const 对象**调用不同的版本。

!!! example "通过 const 重载实现读写分离"

    ``` cpp linenums="1" hl_lines="9 15 29 32"
    #include <iostream>
    using namespace std;

    class Buffer {
    public:
        Buffer(int size) : data(new char[size]), sz(size) {}

        // 非 const 版本：返回可修改的引用（可写）
        char& at(int index) {
            cout << "Non-const at() called" << endl;
            return data[index];
        }

        // const 版本：返回只读引用（只读）
        const char& at(int index) const {
            cout << "Const at() called" << endl;
            return data[index];
        }

        ~Buffer() { delete[] data; }

    private:
        char* data;
        int sz;
    };

    int main() {
        Buffer b(10);
        b.at(0) = 'A';           // 调用非 const 版本，返回可修改引用

        const Buffer cb(10);
        cout << cb.at(0) << endl; // 调用 const 版本，返回只读引用

        return 0;
    }
    ```

!!! info "常见用法"

    - **非 const 版本**：返回引用或指针，允许修改。
    - **const 版本**：返回 const 引用或值，只允许读取。
    - 标准库容器（如 `vector`、`string`）的 `operator[]` 和 `at()` 函数都采用了这种重载模式。

### const 成员函数的最佳实践

!!! tip "何时标记为 const？"

    - **getter 函数**：所有读取数据成员而不修改的函数，都应该标记为 `const`。
    - **查询函数**：只返回计算结果而不修改对象的函数。
    - **显示/打印函数**：如 `show()`、`print()` 等。
    - **比较函数**：如 `equals()`、`compare()` 等。

!!! warning "何时不能标记为 const？"

    - 修改数据成员的函数（如 `set()`、`move()`）。
    - 返回非 const 引用的函数（允许外部修改内部状态）。
    - 调用其他非 const 成员函数的函数。

!!! tip "关键原则"

    **默认将不修改对象的成员函数标记为 const**。这既是良好的编码规范，也是让常对象在更多场景下可用的前提（例如常对象只能调用 const 函数）。

## 常数据成员

### 基本概念

**常数据成员（const Data Member）** 是指在类中声明为 `const` 的数据成员。常数据成员在对象创建后**不能再被修改**。

!!! example "常数据成员的声明与初始化"

    ``` cpp linenums="1" hl_lines="12"
    class Student {
    public:
        // 常数据成员必须在初始化列表中初始化
        Student(int id, string name)
            : studentId(id), name(name) {}

        void show() const {
            cout << "ID: " << studentId << ", Name: " << name << endl;
        }

    private:
        const int studentId;   // 常数据成员：学号，创建后不可修改
        string name;           // 普通成员：姓名，可以修改
    };

    int main() {
        Student s1(2024001, "张三");
        s1.show();

        // s1.studentId = 2024002;   // 错误！const 成员不可修改
        // s1.name = "李四";         // OK：普通成员可以修改

        return 0;
    }
    ```

!!! info "常数据成员的初始化规则"

    - **必须在初始化列表中初始化**：常数据成员不能通过赋值初始化，必须在构造函数的初始化列表中完成初始化。

    ``` cpp
    class Example {
    public:
        // 正确：在初始化列表中初始化
        Example(int val) : constMember(val) {}

        // 错误：不能在构造函数体中赋值
        // Example(int val) { constMember = val; }

    private:
        const int constMember;
    };
    ```

    - **不能使用类内初始值赋值**：`const int id = 0;` 虽然语法允许，但初始化列表仍然可以覆盖，且更推荐在初始化列表中表达依赖参数的初始化。
    - **一旦初始化，不可修改**：常数据成员的值在对象整个生命周期中保持不变。

!!! tip "常数据成员的适用场景"

    - **对象的固定标识**：如学号、身份证号、订单编号等。
    - **配置参数**：对象创建后不应变更的配置值。
    - **类型/版本标识**：用于标识对象类型或版本号的常量。

### const 成员与 static const 成员

常数据成员和静态常数据成员有不同用途：

!!! example "static const 成员"

    ``` cpp linenums="1"
    class MathConstants {
    public:
        // 静态常数据成员：属于类，所有对象共享
        static const double PI;
        static constexpr double E = 2.71828;   // C++17: 可以在类内定义

    private:
        // 普通常数据成员：属于对象，每个对象各自有一份
        const int objectId;
    };

    // 静态常数据成员在类外定义（C++17 之前）
    const double MathConstants::PI = 3.14159;
    ```

|            | **常数据成员**           | **静态常数据成员**                 |
| :--------- | :----------------------- | :--------------------------------- |
| **归属**   | 属于每个对象             | 属于整个类                         |
| **存储**   | 每个对象各有一份         | 所有对象共享一份                   |
| **初始化** | 在构造函数的初始化列表中 | 在类外定义（C++17前）或类内 inline |
| **用途**   | 对象的固定属性           | 类级别的常量                       |

## 常引用

### 基本概念

**常引用（const Reference）** 是指用 `const` 修饰的引用，即 `const T&`。常引用指向的对象**不能被修改**，但常引用本身不会复制对象。

!!! example "常引用的使用"

    ``` cpp linenums="1" hl_lines="1"
    void process(const string& str) {
        // 只能读取 str，不能修改
        cout << str.length() << endl;
        // str += "!";   // 错误！不能修改 const 引用指向的对象
    }

    int main() {
        string s = "Hello";
        process(s);   // 传递引用，不复制
        return 0;
    }
    ```

!!! info "常引用的核心优势"

    - **只读访问**：保证不会修改被引用的对象。
    - **避免复制**：对于大型对象，传递引用比传递值效率高得多。
    - **可接受临时对象**：常引用可以绑定到临时对象（右值），延长临时对象的生命周期。

### 常引用与临时对象

常引用的一个重要特性是：**可以绑定到临时对象（右值）**，并延长其生命周期。

!!! example "常引用延长临时对象生命周期"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x, int y) : x(x), y(y) {
            cout << "Point constructed" << endl;
        }
        ~Point() {
            cout << "Point destroyed" << endl;
        }

        void show() const { cout << "Point(" << x << ", " << y << ")" << endl; }

    private:
        int x, y;
    };

    // 常引用参数可以接受临时对象
    void display(const Point& p) {
        p.show();
    }

    int main() {
        display(Point(3, 4));   // 临时对象绑定到常引用，生命周期延长到函数结束

        // 常引用绑定临时对象
        const Point& ref = Point(5, 6);   // 临时对象生命周期延长到 ref 生命周期结束
        ref.show();

        // Point& badRef = Point(7, 8);   // 错误！普通引用不能绑定临时对象

        return 0;
    }
    ```

!!! warning "关键区别"

    - **常引用**（`const T&`）：可以绑定临时对象，延长其生命周期。
    - **普通引用**（`T&`）：不能绑定临时对象（除非是 `T&&` 右值引用）。

## const 设计指南

!!! summary "const 在不同上下文中的含义"

    | 使用场景       | 示例                  | 含义                       |
    | :------------- | :-------------------- | :------------------------- |
    | **常对象**     | `const Point p;`      | 该对象状态不可修改         |
    | **常成员函数** | `void show() const;`  | 该函数不修改对象状态       |
    | **常数据成员** | `const int id;`       | 该成员在对象创建后不可修改 |
    | **常引用参数** | `void f(const T& p);` | 函数借用对象，只读不复制   |
    | **常指针**     | `const T* p;`         | 指针指向的内容不可修改     |
    | **指针常量**   | `T* const p;`         | 指针本身不可修改           |

!!! tip "设计原则"

    1. **默认使用 const**：尽量将不修改对象的成员函数标记为 `const`。
    2. **传递大对象使用常引用**：`const T&` 是传递只读对象的首选方式。
    3. **固定身份使用 const 成员**：对象的唯一标识符应声明为 `const`。
    4. **const 是接口的一部分**：const 修饰表达了函数的意图，调用者可以据此判断函数的行为。

## 小结

1. **常对象**创建后不可修改，只能调用常成员函数，适合表达只读状态。
2. **常成员函数**承诺不修改对象状态，是常对象可用的函数。getter 和查询函数应标记为 `const`。
3. **const 重载**允许同名函数根据对象是否为 const 提供不同版本（读/写分离）。
4. **常数据成员**在对象创建后不可修改，必须在初始化列表中初始化，适合表达固定标识。
5. **常引用**传递对象时既避免复制，又保证只读，是传递大型对象的推荐方式。

`const` 是 C++ 中表达“只读”语义的核心工具。它不仅让编译器帮助检查错误，更让代码的意图更清晰。在设计类时，合理使用 `const` 是写出高质量 C++ 代码的基本功。