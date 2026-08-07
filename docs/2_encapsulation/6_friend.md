# 类的友元

封装提供了清晰的模块边界，但不是绝对封闭；友元提供一条受控、明确的例外通道。

## 什么是友元

在面向对象程序设计中，类的私有成员（`private`）对外部代码是不可见的。这是**封装**的核心原则——对象的状态只能通过公开接口访问和修改。

然而，在某些特殊场景下，这种严格的封装可能带来不便：

- 两个类需要**紧密协作**，却不愿通过层层接口函数绕路。
- 某个**外部函数**（如调试函数）需要深入访问对象的内部状态。
- 某个**运算符重载**需要同时操作两个对象的私有成员。

!!! abstract "友元的核心思想"

    友元（Friend）是一种**例外机制**：允许一个类**显式授权**特定的外部函数或类访问其私有成员。这就像在私人住宅上开一扇有权限控制的侧门——只有持有钥匙（友元授权）的人才能进入。
    友元是封装机制的**补充**，而不是替代。它让类能够精确控制“谁能破例访问内部”，而不是放弃所有保护。

## 友元函数

### 基本概念

**友元函数（Friend Function）** 是一个普通的外部函数（不属于任何类），但被某个类授权为友元。友元函数可以访问该类的所有成员（包括 `private` 和 `protected`）。

!!! info "友元函数的特点"

    - **不是成员函数**：友元函数是普通函数，不是类的成员。
    - **可访问私有成员**：可以访问授权类的所有成员。
    - **授权在类内声明**：友元声明放在类定义内部，用 `friend` 关键字。
    - **双向性**：友元关系是单向的，A 授权 B 为友元，不代表 B 授权 A。
    - **非传递性**：友元关系不会传递。

### 典型应用：计算两点距离

计算两点之间的距离是一个经典的友元函数应用场景。距离是两个 `Point` 对象的共同操作，
写成普通函数来操作两个相关对象比写成成员函数更符合传统编程习惯。

!!! example "友元函数：计算两点距离"

    ``` cpp linenums="1" hl_lines="9-10 20-26"
    #include <iostream>
    #include <cmath>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 友元函数声明：dist 被授权访问 Point 的私有成员
        friend double dist(const Point& a, const Point& b);

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    // 友元函数定义：可以直接访问 Point 的私有成员 x 和 y
    double dist(const Point& a, const Point& b) {
        // 虽然是普通函数，但可以访问私有成员！
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return sqrt(dx * dx + dy * dy);
    }

    int main() {
        Point p1(3, 4);
        Point p2(6, 8);

        cout << "Distance: " << dist(p1, p2) << endl;   // 5

        // dist 是普通函数，不是成员函数，不能通过对象调用
        // p1.dist(p2);   // 错误！dist 不是 Point 的成员函数

        return 0;
    }
    ```

!!! note "为什么 dist 适合作为友元函数？"

    - **语义自然**：计算两点距离是两点之间的**关系操作**，不属于任何一个点单独的行为。
    - **对称性**：`dist(p1, p2)` 和 `dist(p2, p1)` 是对称的，作为独立函数表达更自然。

### 友元函数的声明与定义

友元函数的声明在类定义内部，但**不是类的一部分**。

!!! tip "友元函数的语法结构"

    ``` cpp linenums="1"
    class MyClass {
    public:
        // ... 其他成员

        // 友元声明：friend + 函数原型
        friend ReturnType functionName(ParameterList);

    private:
        // ... 私有成员
    };

    // 友元函数的定义（普通函数，不属于类）
    ReturnType functionName(ParameterList) {
        // 可以访问 MyClass 的私有成员
    }
    ```

!!! info "注意事项"

    - 友元声明可以出现在类定义的 `public`、`private` 或 `protected` 段中，位置不影响其效果。
    - 友元函数可以在类内声明，但**必须在类外定义**（除非是内联函数）。
    - 友元函数的权限是**授予**的，而不是**索取**的。类主动授权，外部无法强迫。

## 友元类

### 基本概念

**友元类（Friend Class）** 允许一个类将另一个类授权为友元。当友元类的成员函数都需要访问授权类的私有成员时，这种方式比逐个声明友元函数更简洁。

!!! info "友元类的特点"

    - **批量授权**：友元类的**所有成员函数**都获得访问权限。
    - **单向关系**：A 授权 B 为友元，B 可以访问 A 的私有成员，但 A 不能访问 B 的私有成员（除非 B 也授权 A）。
    - **非传递性**：C 是 B 的友元，不代表 C 是 A 的友元。
    - **非继承性**：派生类不会自动成为基类友元的友元。

### 典型应用：Window 与 WindowManager

!!! example "友元类：窗口管理器"

    ``` cpp linenums="1" hl_lines="19 32 33"
    #include <iostream>
    using namespace std;

    // 前向声明：告诉编译器 Window 是一个类
    class Window;

    // WindowManager 需要管理 Window 的私有尺寸信息
    class WindowManager {
    public:
        void resize(Window& w, int newWidth, int newHeight);
        void showInfo(const Window& w) const;
    };

    class Window {
    public:
        Window(int w = 800, int h = 600) : width(w), height(h) {}

        // 授权 WindowManager 访问私有成员
        friend class WindowManager;

        void show() const {
            cout << "Window(" << width << " x " << height << ")" << endl;
        }

    private:
        int width;
        int height;
    };

    // WindowManager 的成员函数可以访问 Window 的私有成员
    void WindowManager::resize(Window& w, int newWidth, int newHeight) {
        w.width = newWidth;     // 可以访问 Window 的私有成员
        w.height = newHeight;
    }

    void WindowManager::showInfo(const Window& w) const {
        cout << "Window size: " << w.width << " x " << w.height << endl;
    }

    int main() {
        Window win(1024, 768);
        WindowManager mgr;

        mgr.showInfo(win);      // Window size: 1024 x 768

        mgr.resize(win, 1920, 1080);
        win.show();             // Window(1920 x 1080)

        return 0;
    }
    ```

### 友元类的设计场景

!!! tip "适合使用友元类的场景"

    - **紧密协作的类对**：两个类设计为一起工作，如 `Database` 和 `Connection`。
    - **管理器模式**：如 `WindowManager` 管理 `Window` 对象，需要修改窗口内部状态。
    - **工厂模式**：工厂类需要访问产品类的私有构造函数。
    - **迭代器模式**：迭代器需要访问容器类的内部数据结构。

    关于上述设计模式的详细内容，可以参考[《设计模式》](https://item.jd.com/14270079.html)一书。

## 友元函数的替代方案

友元虽然方便，但过度使用会破坏封装性，在其它面向对象语言中也可能没有提供友元功能。
实际开发中，我们可以考虑几种替代方案。

### 方案一：使用公共接口（getter/setter）

这是最基本的封装方式——通过公共的 `get` 和 `set` 函数来访问私有成员。

!!! example "替代方案一：公共接口"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 公共接口：通过 getter 访问私有成员
        int getX() const { return x; }
        int getY() const { return y; }

    private:
        int x, y;
    };

    // 通过公共接口计算距离
    double dist(const Point& a, const Point& b) {
        double dx = a.getX() - b.getX();
        double dy = a.getY() - b.getY();
        return sqrt(dx * dx + dy * dy);
    }
    ```

!!! info "优点"

    - **保持封装**：私有成员仍然受到保护，没有破坏类的边界。
    - **接口稳定**：即使内部实现变化，`getX()`/`getY()` 的签名可以保持不变。

!!! warning "缺点"

    - **可能暴露过多**：为每个私有成员添加 getter 可能不必要地暴露内部数据结构。
    - **性能开销**：函数调用可能有微小开销（虽然内联可以优化）。
    - **冗余代码**：可能需要写大量简单的 getter/setter 函数。

### 方案二：将操作定义为成员函数

如果某个操作确实需要访问对象的内部状态，可以考虑将其作为该类的成员函数。

!!! example "替代方案二：成员函数"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 作为成员函数：直接访问私有成员
        double distFrom(const Point& other) const {
            double dx = x - other.x;
            double dy = y - other.y;
            return sqrt(dx * dx + dy * dy);
        }

    private:
        int x, y;
    };

    int main() {
        Point p1(3, 4);
        Point p2(6, 8);
        cout << p1.distFrom(p2) << endl;   // p1 到 p2 的距离
        cout << p2.distFrom(p1) << endl;   // p2 到 p1 的距离（对称）
        return 0;
    }
    ```

!!! info "优点"

    - **完全封装**：所有访问都在类内部完成。
    - **符合面向对象原则**：操作与数据在同一个类中。

### 方案三：使用静态成员函数

当操作与类强相关但不需要访问特定对象的非静态成员时，可以将其设计为静态成员函数。

!!! example "替代方案三：静态成员函数"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 静态成员函数：可以访问私有成员
        static double dist(const Point& a, const Point& b) {
            double dx = a.x - b.x;
            double dy = a.y - b.y;
            return sqrt(dx * dx + dy * dy);
        }

    private:
        int x, y;
    };

    int main() {
        Point p1(3, 4);
        Point p2(6, 8);
        cout << Point::dist(p1, p2) << endl;   // 通过类名调用
        return 0;
    }
    ```

!!! info "优点"

    - **语义清晰**：`Point::dist(p1, p2)` 明确表示这是 Point 类相关的操作。
    - **无需对象**：可以直接通过类名调用。
    - **访问私有成员**：可以访问任何 Point 对象的私有成员。

### 方案对比总结

| 方案                   | 访问权限       | 调用方式            | 封装性               | 适用场景                     |
| :--------------------- | :------------- | :------------------ | :------------------- | :--------------------------- |
| **友元函数**           | 可访问私有成员 | `func(a, b)`        | 破坏封装（可控例外） | 操作对称性强、不属于单个对象 |
| **公共接口（getter）** | 仅通过接口     | `func(a.getX())`    | 完全封装             | 暴露少量简单属性             |
| **成员函数**           | 可访问私有成员 | `a.func(b)`         | 完全封装             | 操作与单个对象强绑定         |
| **静态成员函数**       | 可访问私有成员 | `Class::func(a, b)` | 完全封装             | 操作属于类，不属于对象       |

## 友元的设计原则

!!! abstract "使用友元前的三个问题"

    - **必要吗？** 公共接口能否自然表达该操作？
    - **有限吗？** 能否只授权一个函数而非整个类？
    - **稳定吗？** 这两个抽象是否确实需要紧密协作？

!!! tip "最佳实践"

    - 优先使用 **公共接口**，只有确实需要访问私有成员时才考虑友元。
    - 如果友元确实必要，**优先授权函数**而非整个类（更精细的控制）。
    - **文档化友元关系**：在类的注释中说明哪些函数/类是友元，以及为什么要授权。
    - 如果发现友元使用频繁，考虑重新设计类的接口，避免频繁的“例外”。

!!! danger "避免滥用友元"

    ``` cpp
    // 反模式：滥用友元
    class Data {
    public:
        friend void func1();
        friend void func2(); 
        friend void func3();
        // ...
    };
    ```

    太多的友元声明意味着类的封装边界已经模糊，需要考虑重新设计。

### 友元的限制

!!! info "友元关系的三条性质"

    | 性质         | 说明                                              | 示例                                                                  |
    | :----------- | :------------------------------------------------ | :-------------------------------------------------------------------- |
    | **单向性**   | A 授权 B，B 不自动授权 A                          | `WindowManager` 可访问 `Window`，但 `Window` 不能访问 `WindowManager` |
    | **非传递性** | B 是 A 的友元，C 是 B 的友元，C 不自动是 A 的友元 | 需要显式授权                                                          |
    | **非继承性** | 子类不继承父类的友元关系                          | 父类的友元不能访问子类的私有成员                                      |

!!! example "友元关系的非传递性"

    ``` cpp linenums="1"
    class A {
        friend class B;   // B 可以访问 A
    };

    class B {
        friend class C;   // C 可以访问 B
    };

    // C 不能访问 A，除非 A 显式声明 friend class C
    ```

## 小结

1. **友元函数**是普通函数，被类授权后可访问私有成员，适合处理对称性强的操作。
2. **友元类**的所有成员函数都获得访问权限，适合管理器、工厂等设计模式。
3. **替代方案**：优先考虑公共接口、成员函数或静态成员函数，只在必要时使用友元。
4. **设计原则**：友元是封装机制的例外，应当谨慎使用，明确授权范围。
5. **性质限制**：友元关系是单向、非传递、非继承的。

友元是 C++ 提供的一种**精确授权机制**。把它当作“破例的通行证”，而不是“自由的通行证”，才能写出既安全又优雅的代码。