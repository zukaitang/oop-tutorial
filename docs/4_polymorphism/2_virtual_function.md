# 动态绑定与虚函数

虚函数是C++实现运行时多态的核心机制。它允许通过基类指针或引用调用派生类中覆盖的函数，使得程序能够在运行时根据对象的实际类型决定调用哪个函数版本。

## 从问题出发：为什么需要虚函数

### 静态绑定的局限性

回顾上一节的类型兼容性示例，我们发现在没有虚函数的情况下，通过基类指针调用的函数版本由**指针类型**决定，而非**对象实际类型**。

!!! example "问题场景"

    ``` cpp linenums="1" hl_lines="18-20 26 27"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        void display() const {
            cout << "Base::display()" << endl;
        }
    };

    class Derived : public Base {
    public:
        void display() const {
            cout << "Derived::display()" << endl;
        }
    };

    void show(Base* p) {
        p->display();   // 调用哪个版本？
    }

    int main() {
        Base b;
        Derived d;

        show(&b);   // Base::display()
        show(&d);   // Base::display()  ← 不是期望的 Derived::display()！

        return 0;
    }
    ```

!!! danger "问题分析"

    在上述代码中，`show(&d)` 调用的是 `Base::display()` 而不是 `Derived::display()`。这是因为：

    - 编译器在**编译时**就确定了 `p->display()` 应调用哪个函数。
    - `p` 的类型是 `Base*`，因此调用的是 `Base::display()`。
    - 这种在编译时确定函数调用的方式称为**静态绑定（Static Binding）**。

    但面向对象程序设计期望的是：**根据对象的实际类型调用相应的函数**。这就是 **动态绑定（Dynamic Binding）** 的需求。

!!! info "核心需求"

    我们需要一种机制，使得：

    - 通过 `Base*` 或 `Base&` 操作对象。
    - 如果对象实际是 `Derived` 类型，调用 `Derived` 版本的函数。
    - 如果对象实际是 `Base` 类型，调用 `Base` 版本的函数。

    **虚函数正是为实现这一需求而设计的。**

## 虚函数的定义与基本使用

### 什么是虚函数

**虚函数（Virtual Function）** 是在基类中用 `virtual` 关键字声明的非静态成员函数。它告诉编译器：在调用这个函数时，应该根据对象的实际类型来决定调用哪个版本。

!!! info "虚函数的声明语法"

    ``` cpp
    class 基类名 {
    public:
        virtual 返回类型 函数名(参数列表);
        // 可以有函数体（在类外实现），也可以为纯虚函数
    };
    ```

    - 虚函数在**基类**中声明，使用 `virtual` 关键字。
    - 派生类中可以**覆盖（override）** 基类的虚函数。
    - 派生类覆盖时，`virtual` 关键字可以省略（但建议保留以增加可读性）。

### 虚函数的简单示例

!!! example "虚函数实现运行时多态"

    ``` cpp linenums="1" hl_lines="7 17 23 30 31"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        // 声明为虚函数
        virtual void display() const {
            cout << "Base::display()" << endl;
        }

        virtual ~Base() {}   // 虚析构函数
    };

    class Derived : public Base {
    public:
        // 覆盖基类的虚函数
        virtual void display() const override {
            cout << "Derived::display()" << endl;
        }
    };

    void show(Base* p) {
        p->display();   // 运行时多态：根据对象实际类型调用
    }

    int main() {
        Base b;
        Derived d;

        show(&b);   // Base::display()
        show(&d);   // Derived::display()  ✓ 正确！

        return 0;
    }
    ```

    运行结果：

    ```
    Base::display()
    Derived::display()
    ```

## 静态绑定与动态绑定

### 静态绑定（Static Binding）

**静态绑定**（也称为**早期绑定（Early Binding）**）是在**编译时**确定函数调用的目标。

!!! info "静态绑定的特点"

    - 编译器根据**指针或引用的静态类型**决定调用哪个函数。
    - 调用目标在编译时就已经确定。
    - 效率高，没有运行时开销。
    - 适用于非虚函数、重载函数、普通函数。

### 动态绑定（Dynamic Binding）

**动态绑定**（也称为**晚期绑定（Late Binding）**）是在**运行时**根据对象的实际类型确定函数调用的目标。

!!! info "动态绑定的特点"

    - 编译器在编译时不决定调用的具体函数版本。
    - 程序运行时，根据**对象的实际类型**决定调用哪个函数。
    - 需要虚函数的支持。
    - 有轻微的运行时开销（查虚表）。
    - 是实现多态的基础。

!!! info "动态绑定的条件"

    要实现动态绑定，需要同时满足三个条件：

    1. **虚函数**：被调用的函数必须是虚函数。
    2. **指针或引用**：必须通过基类指针或引用来调用。
    3. **覆盖**：派生类必须覆盖基类的虚函数。

!!! example "静态与动态绑定示例"

    ``` cpp linenums="1"
    class Base {
    public:
        void func() { cout << "Base::func()" << endl; }
        virtual void vfunc() { cout << "Base::vfunc()" endl; }
    };

    class Derived : public Base {
    public:
        void func() { cout << "Derived::func()" << endl; }
        virtual void vfunc() override { cout << "Derived::vfunc()" endl; }
    };

    int main() {
        Derived d;
        Base* p = &d;

        p->func();     // 静态绑定 → Base::func()（p 的类型是 Base*）
        p->vfunc();    // 动态绑定 → Derived::vfunc()（对象实际是 Derived）

        return 0;
    }
    ```

## Override与函数覆盖

### 覆盖的条件

`override`关键字写在函数签名的结尾，表示派生类对基类中同名函数的重写覆盖。
派生类中的虚函数要**覆盖（override）** 基类的虚函数，必须满足：

!!! info "覆盖规则"

    | 条件               | 说明                             |
    | :----------------- | :------------------------------- |
    | **函数名相同**     | 派生类和基类的函数名必须一致     |
    | **参数列表相同**   | 参数个数、类型、顺序必须完全一致 |
    | **返回类型相同**   | 普通情况下返回类型必须相同       |
    | **cv限定符相同**   | `const`、`volatile` 限定必须相同 |
    | **引用限定符相同** | `&` 或 `&&`（C++11）必须相同     |

!!! note "关于返回类型的特殊规则：协变返回类型"

    如果基类虚函数返回**基类指针或引用**，派生类覆盖时返回**派生类指针或引用**是允许的。这称为**协变返回类型（Covariant Return Type）**。

    ``` cpp
    class Base {
    public:
        virtual Base* clone() const { return new Base(*this); }
    };

    class Derived : public Base {
    public:
        // 返回 Derived* 是允许的（协变返回类型）
        virtual Derived* clone() const override {
            return new Derived(*this);
        }
    };
    ```

### 派生类虚函数的声明风格

!!! warning "三种风格"

    === "风格一：不写 virtual（不推荐）"

        ``` cpp
        class Derived : public Base {
        public:
            void display() const {   // 虽然也是虚函数，但不直观
                cout << "Derived::display()" << endl;
            }
        };
        ```

        如果不写 `virtual`，只要函数签名与基类的虚函数匹配，它仍然是虚函数。但这种方式**可读性差**，读者无法一眼看出这是一个虚函数覆盖。

    === "风格二：写 virtual（推荐）"

        ``` cpp
        class Derived : public Base {
        public:
            virtual void display() const {
                cout << "Derived::display()" << endl;
            }
        };
        ```

        明确标出 `virtual`，让读者知道这是一个虚函数。

    === "风格三：virtual + override（C++11，最佳）"

        ``` cpp
        class Derived : public Base {
        public:
            virtual void display() const override {
                cout << "Derived::display()" << endl;
            }
        };
        ```

        `override` 关键字让编译器检查是否正确覆盖了基类的虚函数。

### override 关键字（C++11）

`override` 关键字用于显式声明一个函数要覆盖基类的虚函数。如果声明的函数实际上并未覆盖基类虚函数，编译器会报错。

!!! example "override 防止错误"

    ``` cpp linenums="1"
    class Base {
    public:
        virtual void func(int x) const { }
        virtual ~Base() { }
    };

    class Derived : public Base {
    public:
        // 错误！基类没有 virtual void func(int)（参数类型不同）
        // virtual void func(double x) const override { }

        // 错误！基类没有 virtual void func(int)（缺少 const）
        // virtual void func(int x) override { }

        // 正确：完全匹配基类的虚函数
        virtual void func(int x) const override { }
    };
    ```

!!! success "override 的价值"

    - **编译时检查**：确保派生类确实覆盖了基类的虚函数。
    - **防止错误**：避免因签名不匹配而导致"意外"创建了新函数而非覆盖。
    - **代码可读性**：明确表达"这是一个覆盖"的意图。

## 虚函数表（vtable）与动态绑定的实现

### 虚表的基本概念

C++ 编译器通过**虚函数表（Virtual Table，简称 vtable 或虚表）** 来实现动态绑定。

!!! info "虚表的核心机制"

    1. **每个多态类有一个虚表**：包含该类所有虚函数的入口地址。
    2. **每个对象有一个虚指针（vptr）**：指向所属类的虚表。
    3. **构造对象时设置 vptr**：构造函数中为对象的 vptr 赋值。
    4. **调用时查表**：通过虚指针找到虚表，再根据索引找到函数地址并调用。

### 虚表示意图

```
类 Base：
┌─────────────────────────────────────┐
│              Base 对象              │
│  ┌─────────┐    ┌─────────────────┐ │
│  │  vptr   │───→│  Base 虚表      │ │
│  ├─────────┤    ├─────────────────┤ │
│  │  data   │    │ &Base::func1()  │ │
│  └─────────┘    │ &Base::func2()  │ │
│                 └─────────────────┘ │
└─────────────────────────────────────┘

类 Derived（继承自 Base）：
┌─────────────────────────────────────┐
│            Derived 对象             │
│  ┌─────────┐    ┌─────────────────┐ │
│  │  vptr   │───→│ Derived 虚表    │ │
│  ├─────────┤    ├─────────────────┤ │
│  │ Base数据 │    │ &Derived::func1│ │
│  ├─────────┤    │ &Base::func2()  │ │
│  │Derived数│    └─────────────────┘ │
│  └─────────┘                        │
└─────────────────────────────────────┘
```

!!! note "动态绑定的执行过程"

    当通过 `Base*` 调用 `func()` 时：

    1. 从对象中获取 `vptr`（虚指针）。
    2. 通过 `vptr` 找到对应类的虚表。
    3. 在虚表中查找 `func` 的入口地址。
    4. 通过该地址调用实际的函数。

    这个过程在运行时完成，因此称为"动态绑定"。

## 虚函数的"遗传性"

一旦一个函数在基类中被声明为虚函数，它在派生类的继承体系中**始终保持虚函数特性**，无论派生类是否显式使用 `virtual` 关键字。

!!! example "虚函数的遗传性"

    ``` cpp linenums="1" hl_lines="6 14 22"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        virtual void show() {
            cout << "Base::show()" << endl;
        }
    };

    // Derived 中未写 virtual，但 show 仍是虚函数
    class Derived : public Base {
    public:
        void show() {   // 仍然是虚函数（从 Base 继承来的特性）
            cout << "Derived::show()" << endl;
        }
    };

    // GrandDerived 中未写 virtual，但 show 仍是虚函数
    class GrandDerived : public Derived {
    public:
        void show() {   // 仍然是虚函数
            cout << "GrandDerived::show()" << endl;
        }
    };

    int main() {
        Base* p = new GrandDerived();
        p->show();   // GrandDerived::show()（动态绑定）

        delete p;
        return 0;
    }
    ```

!!! tip "最佳实践"

    虽然派生类中不写 `virtual` 也能保持虚函数特性，但**建议在派生类中也使用 `virtual` 关键字**（配合 `override`），以增强代码的可读性和可维护性。

## 哪些成员函数可以是虚函数

!!! summary "虚函数的使用限制"

    | 成员函数类型       | 能否为虚函数 | 说明                                       |
    | :----------------- | :----------: | :----------------------------------------- |
    | **非静态成员函数** |    ✓ 可以    | 最常见的虚函数                             |
    | **静态成员函数**   |   ✗ 不可以   | 属于类而非对象，无 `this` 指针             |
    | **构造函数**       |   ✗ 不可以   | 对象构造时虚表尚未建立                     |
    | **析构函数**       |    ✓ 可以    | 虚析构函数是最重要的虚函数之一             |
    | **内联函数**       |    不推荐    | 内联是静态的，虚函数是动态的，两者语义冲突 |
    | **友元函数**       |   ✗ 不可以   | 友元不是类的成员函数                       |

### 为什么构造函数不能是虚函数

!!! info "原因分析"

    1. **虚表尚未建立**：调用构造函数时，对象的虚指针（vptr）尚未初始化，无法完成动态绑定。
    2. **对象类型明确**：构造对象时，对象的类型在编译时就是确定的，不需要动态绑定。
    3. **语义冲突**：构造函数的作用是初始化对象，虚函数的目的是实现多态，两者目的不同。

## 虚析构函数

将析构函数声明为虚函数是 C++ 编程的**重要准则**：如果类可能被用作基类，析构函数应声明为 `virtual`。
当通过基类指针删除派生类对象时，如果基类的析构函数不是虚函数，派生类的析构函数将不会被调用，导致资源泄漏。

!!! danger "非虚析构函数的问题"

    当通过基类指针删除派生类对象时，如果析构函数不是虚函数，只会调用基类的析构函数，派生类的析构函数不会被调用，导致**资源泄漏**。

    ``` cpp linenums="1" hl_lines="24"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        ~Base() {   // 非虚析构函数
            cout << "Base destructor" << endl;
        }
    };

    class Derived : public Base {
    private:
        int* data;
    public:
        Derived() : data(new int[100]) {}
        ~Derived() {
            cout << "Derived destructor" << endl;
            delete[] data;   // 不会被调用！
        }
    };

    int main() {
        Base* p = new Derived();
        delete p;   // 只调用 Base::~Base()，Derived 的资源泄漏！
        return 0;
    }
    ```

!!! success "正确的做法：虚析构函数"

    ``` cpp linenums="1"
    class Base {
    public:
        virtual ~Base() {   // 虚析构函数
            cout << "Base destructor" << endl;
        }
    };

    // 当 delete 基类指针时：
    // 1. 先调用 Derived::~Derived()（释放资源）
    // 2. 再调用 Base::~Base()
    ```

!!! tip "最佳实践"

    **如果类有可能被用作基类（即存在派生类），应将析构函数声明为 `virtual`。** 这是 C++ 编程的黄金规则之一。

    即使类没有派生类，将析构函数声明为虚函数也通常无害（但会增加虚函数表开销），建议在基类中做此声明。

## 虚函数与默认参数

!!! warning "重要陷阱：虚函数与默认参数"

    虚函数的默认参数是在**编译时**根据**静态类型**确定的，而非运行时根据动态类型确定。

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        virtual void func(int x = 1) {
            cout << "Base: " << x << endl;
        }
    };

    class Derived : public Base {
    public:
        virtual void func(int x = 2) override {
            cout << "Derived: " << x << endl;
        }
    };

    int main() {
        Derived d;
        Base* p = &d;

        p->func();   // Derived: 1  （不是 Derived: 2！）

        return 0;
    }
    ```

!!! tip "最佳实践"

    **避免在虚函数中使用默认参数。** 如果必须使用，确保基类和派生类中的默认参数值一致。

## final 关键字（C++11）

`final` 关键字有两种用法：

### 1. final 类：禁止被继承

``` cpp linenums="1"
class Base final {
    // 这个类不能被继承
};

// class Derived : public Base { };  // 编译错误！
```

### 2. final 虚函数：禁止被覆盖

``` cpp linenums="1"
class Base {
public:
    virtual void func() final {
        // 这个函数在派生类中不能被覆盖
    }
};

class Derived : public Base {
public:
    // virtual void func() override { }  // 编译错误！
};
```

!!! tip "使用建议"

    - 当设计需要**完全确定性的行为**时，使用 `final` 防止意外修改。
    - 当类**不应被扩展**时，标记为 `final`。

## 综合示例

!!! example "虚函数综合示例"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    #include <memory>
    using namespace std;

    // 基类 Shape
    class Shape {
    public:
        // 虚函数：计算面积
        virtual double area() const = 0;

        // 虚函数：绘制形状
        virtual void draw() const {
            cout << "Drawing a shape" << endl;
        }

        // 虚析构函数（重要！）
        virtual ~Shape() {
            cout << "Shape destroyed" << endl;
        }
    };

    // Circle 继承 Shape
    class Circle : public Shape {
    private:
        double radius;
    public:
        Circle(double r) : radius(r) {}

        // 覆盖 area
        virtual double area() const override {
            return 3.14159 * radius * radius;
        }

        // 覆盖 draw
        virtual void draw() const override {
            cout << "Drawing a circle with radius " << radius << endl;
        }

        virtual ~Circle() {
            cout << "Circle destroyed" << endl;
        }
    };

    // Rectangle 继承 Shape
    class Rectangle : public Shape {
    private:
        double width, height;
    public:
        Rectangle(double w, double h) : width(w), height(h) {}

        virtual double area() const override {
            return width * height;
        }

        virtual void draw() const override {
            cout << "Drawing a rectangle " << width << "x" << height << endl;
        }

        virtual ~Rectangle() {
            cout << "Rectangle destroyed" << endl;
        }
    };

    // 显示所有形状的信息
    void showInfo(const Shape& shape) {
        shape.draw();
        cout << "Area: " << shape.area() << endl;
        cout << "---" << endl;
    }

    int main() {
        cout << "=== 多态演示 ===" << endl;

        // 使用基类指针指向派生类对象
        Shape* p1 = new Circle(5.0);
        Shape* p2 = new Rectangle(4.0, 3.0);

        // 动态绑定：调用实际类型的函数
        p1->draw();   // Drawing a circle with radius 5
        p2->draw();   // Drawing a rectangle 4x3

        cout << "\n=== 使用引用调用 ===" << endl;
        Circle c(3.0);
        Rectangle r(2.0, 6.0);

        showInfo(c);   // Circle 版本
        showInfo(r);   // Rectangle 版本

        cout << "\n=== 使用 vector 统一管理 ===" << endl;
        vector<Shape*> shapes;
        shapes.push_back(new Circle(2.0));
        shapes.push_back(new Rectangle(3.0, 4.0));
        shapes.push_back(new Circle(1.0));

        for (auto shape : shapes) {
            shape->draw();
            cout << "Area: " << shape->area() << endl;
        }

        cout << "\n=== 清理资源 ===" << endl;
        // 虚析构函数确保正确清理
        delete p1;
        delete p2;
        for (auto shape : shapes) {
            delete shape;
        }

        return 0;
    }
    ```

## 小结

1.  **虚函数**是用 `virtual` 关键字声明的非静态成员函数，是实现运行时多态的基础。

2.  **静态绑定 vs 动态绑定**：
    - 静态绑定：编译时确定函数调用，效率高，适用于非虚函数。
    - 动态绑定：运行时根据对象实际类型确定函数调用，需要虚函数支持。

3.  **虚函数的覆盖规则**：函数名、参数列表、返回类型、cv限定符必须完全一致（协变返回类型除外）。

4.  **虚函数表（vtable）** ：编译器通过虚表和虚指针（vptr）实现动态绑定。

5.  **虚析构函数**：如果类可能被用作基类，析构函数应声明为 `virtual`，避免资源泄漏。

6.  **C++11 关键字**：
    - `override`：显式声明覆盖，让编译器检查是否正确覆盖。
    - `final`：禁止类被继承或虚函数被覆盖。

7.  **注意事项**：
    - 构造函数不能是虚函数。
    - 静态成员函数不能是虚函数。
    - 避免在虚函数中使用默认参数。