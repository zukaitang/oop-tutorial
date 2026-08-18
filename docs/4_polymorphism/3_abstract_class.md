# 纯虚函数与抽象类

纯虚函数和抽象类是C++中实现**接口设计**和**规范继承行为**的重要机制。它们允许基类为派生类定义共同的接口规范，而将具体实现留给派生类完成。

## 从问题出发：为什么需要纯虚函数

在某些情况下，基类中虽然存在某个函数，但由于基类过于抽象，**无法给出有意义的实现**。

!!! example "问题场景"

    ``` cpp linenums="1" hl_lines="9-11 22-24"
    #include <iostream>
    #include <cmath>
    using namespace std;

    class Shape {
    public:
        // 基类的 area() 应该返回什么？
        // 形状没有具体形状，面积没有意义！
        virtual double area() const {
            return 0.0;   // 返回 0 只是临时方案，并不合理
        }

        virtual void draw() const {
            cout << "Drawing a shape" << endl;   // 也不知道该画什么
        }
    };

    class Circle : public Shape {
    public:
        Circle(double r) : radius(r) {}
        // 派生类必须记住要去覆盖这些函数
        virtual double area() const override {
            return 3.14159 * radius * radius;
        }
        virtual void draw() const override {
            cout << "Drawing a circle" << endl;
        }
    private:
        double radius;
    };

    int main() {
        Shape s;   // Shape 的对象有意义吗？它不是任何具体形状
        cout << s.area() << endl;   // 输出 0，意义不明

        return 0;
    }
    ```

!!! danger "上述设计的问题"

    1. **基类对象无意义**：`Shape` 作为一个抽象概念，不应该存在实例。
    2. **基类实现不合理**：`Shape::area()` 返回 0 只是一个"占位符"，没有实际意义。
    3. **无法强制派生类实现**：派生类可能忘记覆盖 `area()`，导致使用基类的默认实现（返回 0），产生逻辑错误。
    4. **缺乏设计约束**：无法在语法层面要求派生类必须提供 `area()` 的具体实现。

!!! abstract "解决方案：纯虚函数"

    将 `area()` 声明为**纯虚函数**，使其**没有实现**，并强制派生类**必须提供实现**。同时，含有纯虚函数的类成为**抽象类**，**不能实例化**。

## 纯虚函数的定义

### 基本语法

纯虚函数是在基类中声明的虚函数，在声明时使用 `= 0` 初始化器，表示该函数在基类中没有实现。

!!! info "纯虚函数的声明语法"

    ``` cpp
    class 类名 {
    public:
        virtual 返回类型 函数名(参数列表) = 0;
    };
    ```

    - `= 0` 是纯虚函数的标志，告诉编译器"这个函数在基类中没有实现"。
    - 纯虚函数可以有函数体（在类外实现），但通常不提供。
    - 含有纯虚函数的类称为**抽象类（Abstract Class）**。

!!! example "声明纯虚函数"

    ``` cpp linenums="1" hl_lines="4 7"
    class Shape {
    public:
        // 纯虚函数：没有实现，要求派生类必须覆盖
        virtual double area() const = 0;

        // 纯虚函数：要求派生类必须覆盖
        virtual void draw() const = 0;

        // 普通虚函数：提供默认实现，派生类可以选择覆盖
        virtual void move(double dx, double dy) {
            // 移动形状的默认实现
        }

        // 虚析构函数：抽象类也需要
        virtual ~Shape() {}
    };
    ```

### 纯虚函数可以有实现

虽然纯虚函数通常不提供实现，但C++允许为纯虚函数提供函数体（在类外定义）。

!!! example "为纯虚函数提供实现"

    ``` cpp linenums="1" hl_lines="13-15 22"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        // 纯虚函数声明
        virtual void show() const = 0;

        virtual ~Base() {}
    };

    // 为纯虚函数提供实现（虽然不常见，但语法允许）
    void Base::show() const {
        cout << "Base::show() default implementation" << endl;
    }

    class Derived : public Base {
    public:
        // 覆盖纯虚函数
        virtual void show() const override {
            // 可以选择调用基类的纯虚函数实现
            Base::show();   // 调用基类的默认实现
            cout << "Derived::show() additional behavior" << endl;
        }
    };

    int main() {
        Derived d;
        d.show();
        return 0;
    }
    ```

!!! note "使用场景"

    为纯虚函数提供实现的场景不多，但偶尔用于：

    - 为派生类提供一个"默认行为"的选项。
    - 在派生类的覆盖中复用基类的部分逻辑。

## 抽象类的概念与特征

### 什么是抽象类

**抽象类（Abstract Class）** 是指含有至少一个纯虚函数的类。抽象类不能实例化，只能作为基类使用。

!!! example "抽象类的使用"

    ``` cpp linenums="1" hl_lines="5 23 41"
    #include <iostream>
    using namespace std;

    // 抽象类
    class Shape {
    public:
        // 纯虚函数：定义接口
        virtual double area() const = 0;
        virtual void draw() const = 0;

        // 普通成员函数：提供共享行为
        void setName(const string& n) { name = n; }
        string getName() const { return name; }

        // 虚析构函数
        virtual ~Shape() {}

    protected:
        string name;   // 可以被派生类访问
    };

    // 具体类：实现了所有纯虚函数
    class Circle : public Shape {
    private:
        double radius;
    public:
        Circle(double r, const string& n) : radius(r) {
            name = n;   // 访问基类的 protected 成员
        }

        virtual double area() const override {
            return 3.14159 * radius * radius;
        }

        virtual void draw() const override {
            cout << "Drawing Circle '" << name << "' with radius " << radius << endl;
        }
    };

    // 具体类：实现了所有纯虚函数
    class Rectangle : public Shape {
    private:
        double width, height;
    public:
        Rectangle(double w, double h, const string& n) : width(w), height(h) {
            name = n;
        }

        virtual double area() const override {
            return width * height;
        }

        virtual void draw() const override {
            cout << "Drawing Rectangle '" << name << "' " << width << "x" << height << endl;
        }
    };

    int main() {
        // Shape s;   // ❌ 错误！不能实例化抽象类

        Circle c(5.0, "MyCircle");
        Rectangle r(4.0, 3.0, "MyRect");

        c.draw();   // Drawing Circle 'MyCircle' with radius 5
        r.draw();   // Drawing Rectangle 'MyRect' 4x3

        return 0;
    }
    ```

!!! info "抽象类的核心特征"

    | 特征                       | 说明                                   |
    | :------------------------- | :------------------------------------- |
    | **不能实例化**             | 不能定义抽象类的对象                   |
    | **只能作为基类**           | 用于被其他类继承                       |
    | **可以包含普通成员**       | 可以有普通成员函数、数据成员和构造函数 |
    | **可以有虚函数实现**       | 抽象类可以为部分虚函数提供默认实现     |
    | **派生类必须实现纯虚函数** | 否则派生类也成为抽象类                 |

### 抽象类与具体类的继承关系

在继承层次中，一个类是否是抽象类，取决于它是否实现了所有从基类继承来的纯虚函数。

!!! info "继承层次中的抽象与具体"

    ```
    Shape (抽象类)
        └── area() = 0
        └── draw() = 0
    
    TwoDShape (抽象类) : public Shape
        └── area() 已实现
        └── draw() = 0     ← 还有纯虚函数，仍是抽象类
    
    Circle (具体类) : public TwoDShape
        └── draw() 已实现  ← 所有纯虚函数都已实现，成为具体类
    ```

!!! example "部分实现纯虚函数的派生类"

    ``` cpp linenums="1" hl_lines="4 12 21"
    #include <iostream>
    using namespace std;

    class Shape {
    public:
        virtual double area() const = 0;
        virtual void draw() const = 0;
        virtual ~Shape() {}
    };

    // TwoDShape 只实现了 area，未实现 draw，仍是抽象类
    class TwoDShape : public Shape {
    public:
        virtual double area() const override {
            return 0.0;   // 默认实现，但可以被子类覆盖
        }
        // draw() 仍然是纯虚函数（继承自 Shape）
    };

    // Circle 实现了所有纯虚函数，成为具体类
    class Circle : public TwoDShape {
    private:
        double radius;
    public:
        Circle(double r) : radius(r) {}

        virtual double area() const override {
            return 3.14159 * radius * radius;
        }

        virtual void draw() const override {
            cout << "Drawing Circle" << endl;
        }
    };

    int main() {
        // TwoDShape s;   // ❌ 错误！仍包含纯虚函数 draw()，是抽象类

        Circle c(5.0);   // ✓ 具体类，可以实例化
        c.draw();        // Drawing Circle

        return 0;
    }
    ```

## 抽象类的作用与设计意图

### 1. 定义接口规范

抽象类定义了派生类必须遵循的接口规范，确保所有派生类都提供一致的操作集合。

!!! success "接口规范的价值"

    ``` cpp linenums="1"
    // 定义统一的接口规范
    class Drawable {
    public:
        virtual void draw() const = 0;
        virtual void resize(double factor) = 0;
        virtual ~Drawable() {}
    };

    // 所有实现 Drawable 的类都必须提供这些操作
    class Circle : public Drawable { /* ... */ };
    class Rectangle : public Drawable { /* ... */ };
    class Triangle : public Drawable { /* ... */ };

    // 可以统一处理
    void renderAll(const vector<Drawable*>& items) {
        for (auto p : items) {
            p->draw();   // 所有 Drawable 都有 draw()
        }
    }
    ```

### 2. 实现代码复用

抽象类可以包含普通成员函数和数据成员，为派生类提供共享的实现。

!!! example "代码复用"

    ``` cpp linenums="1"
    class Logger {
    public:
        virtual void log(const string& message) = 0;

        // 共享功能：带时间戳的日志
        void logWithTimestamp(const string& message) {
            time_t now = time(nullptr);
            string timestamp = ctime(&now);
            timestamp.pop_back();   // 去掉换行符
            log("[" + timestamp + "] " + message);
        }

        virtual ~Logger() {}
    };

    class FileLogger : public Logger {
    private:
        ofstream file;
    public:
        FileLogger(const string& filename) : file(filename, ios::app) {}

        virtual void log(const string& message) override {
            file << message << endl;
        }
    };

    class ConsoleLogger : public Logger {
    public:
        virtual void log(const string& message) override {
            cout << message << endl;
        }
    };
    ```

### 3. 表达设计意图

抽象类清晰地表达了一个类的设计意图——"我定义了一套接口，请派生类来实现具体细节"。

!!! tip "抽象类的设计模式角色"

    在常见的软件设计模式中，抽象类扮演着重要角色：

    | 设计模式       | 抽象类的作用                               |
    | :------------- | :----------------------------------------- |
    | **模板方法**   | 定义算法的骨架，将具体步骤留给子类实现     |
    | **工厂方法**   | 定义创建对象的接口，让子类决定实例化哪个类 |
    | **策略模式**   | 定义算法的接口，不同子类实现不同策略       |
    | **观察者模式** | 定义观察者的更新接口                       |

## 抽象类与纯虚函数的约束规则

### 派生类必须实现所有纯虚函数

如果派生类没有实现基类的所有纯虚函数，它将仍然是抽象类，不能实例化。

!!! example "未完全实现纯虚函数"

    ``` cpp linenums="1"
    class Base {
    public:
        virtual void func1() = 0;
        virtual void func2() = 0;
        virtual ~Base() {}
    };

    // Derived 只实现了 func1，没有实现 func2
    class Derived : public Base {
    public:
        virtual void func1() override {
            cout << "Derived::func1()" << endl;
        }
        // func2() 仍然是纯虚函数
    };

    int main() {
        // Derived d;   // ❌ 错误！Derived 仍然是抽象类

        // 可以定义指向抽象类的指针
        Derived* p = nullptr;   // ✓ 指针可以定义

        return 0;
    }
    ```

### 抽象类中可以有普通成员

抽象类不仅可以有纯虚函数，还可以包含普通成员函数、数据成员和构造函数。

!!! example "抽象类的完整结构"

    ``` cpp linenums="1"
    class AbstractClass {
    public:
        // 纯虚函数
        virtual void pureVirtualFunc() = 0;

        // 普通虚函数（提供默认实现）
        virtual void virtualFunc() {
            cout << "Default implementation" << endl;
        }

        // 普通成员函数（静态绑定）
        void normalFunc() {
            cout << "Normal function" << endl;
        }

        // 数据成员
        int data;

        // 构造函数
        AbstractClass(int d) : data(d) {}

        // 虚析构函数
        virtual ~AbstractClass() {}
    };
    ```

### 抽象类不能作为函数参数类型（传值）

抽象类不能作为**传值参数**或**返回类型**（因为值传递需要创建对象），但可以作为**指针**或**引用**参数。

!!! example "抽象类作为参数"

    ``` cpp linenums="1"
    class Abstract {
    public:
        virtual void func() = 0;
        virtual ~Abstract() {}
    };

    // ❌ 错误：不能传值（需要对象）
    // void process(Abstract obj) { }

    // ✓ 正确：使用引用
    void process(Abstract& obj) {
        obj.func();
    }

    // ✓ 正确：使用指针
    void process(Abstract* obj) {
        obj->func();
    }

    // ✓ 正确：返回指针或引用
    Abstract* createAbstract();   // 可以返回指针
    Abstract& getAbstract();      // 可以返回引用
    ```

## 抽象类 vs 具体类：对比总结

!!! summary "抽象类与具体类的对比"

    | 特性               | 抽象类                 | 具体类                       |
    | :----------------- | :--------------------- | :--------------------------- |
    | **包含纯虚函数**   | 至少一个               | 无                           |
    | **可以实例化**     | ✗ 不能                 | ✓ 可以                       |
    | **可以作为基类**   | ✓ 是主要用途           | ✓ 可以（但可能不设计为基类） |
    | **可以有构造函数** | ✓ 可以（供派生类调用） | ✓ 可以                       |
    | **可以有数据成员** | ✓ 可以                 | ✓ 可以                       |
    | **可以有普通函数** | ✓ 可以                 | ✓ 可以                       |
    | **多态性支持**     | ✓ 支持                 | ✓ 支持                       |
    | **主要设计目的**   | 定义接口规范           | 提供具体功能                 |

## 小结

纯虚函数和抽象类是C++实现接口设计和规范继承行为的重要机制。它们让基类能够定义"契约"，而派生类负责实现"契约"，是构建可扩展、可维护软件体系的关键工具。

1.  **纯虚函数**：使用 `virtual 返回类型 函数名(参数) = 0;` 声明，表示该函数在基类中没有实现，强制派生类必须覆盖。

2.  **抽象类**：含有至少一个纯虚函数的类。不能实例化，只能作为基类使用。

3.  **抽象类的特征**：
    - 不能创建对象（不能实例化）。
    - 可以包含普通成员函数、数据成员、构造函数。
    - 派生类必须实现所有纯虚函数，否则仍然是抽象类。
    - 主要用于定义接口规范和实现代码复用。

4.  **设计意图**：
    - 定义统一的接口规范（"做什么"）。
    - 将具体实现留给派生类（"怎么做"）。
    - 表达"这是抽象概念，不应有实例"的设计语义。

5.  **使用建议**：
    - 当基类无法给出有意义的函数实现时，声明为纯虚函数。
    - 抽象类应至少有一个虚析构函数，确保派生类对象正确析构。
    - 通过抽象类指针或引用实现多态操作。
