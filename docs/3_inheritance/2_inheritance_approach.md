# 继承方式

继承方式是派生类定义中的核心要素，它决定了基类成员在派生类中的访问属性，进而影响了派生类与外部代码对基类成员的可见性。
理解三种继承方式的区别，是正确设计继承关系的基础。

## 访问控制回顾：三种访问属性

在理解继承方式之前，需要先回顾类成员的三种访问属性：

!!! note "类成员的访问属性"

    | 访问属性 | 关键字      | 类内访问 |    派生类访问     | 外部访问 |
    | :------- | :---------- | :------: | :---------------: | :------: |
    | **公有** | `public`    |    ✓     |  取决于继承方式   |    ✓     |
    | **保护** | `protected` |    ✓     |  取决于继承方式   |    ✗     |
    | **私有** | `private`   |    ✓     | ✗（始终不可访问） |    ✗     |

    无论采用何种继承方式，基类的 `private` 成员在派生类中**始终不可直接访问**。这是封装原则的体现——基类的实现细节对派生类也是隐藏的。

## 三种继承方式

C++ 提供了三种继承方式：**公有继承（public）**、**私有继承（private）** 和 **保护继承（protected）**。
它们共同决定了基类成员在派生类中的访问状态。不同的继承方式主要影响两个方面：

1. **派生类成员函数**对基类成员的访问权限。
2. **派生类对象（外部代码）** 对从基类继承的成员的访问权限。

!!! warning "继承方式与访问控制"

    继承方式与访问控制都使用关键字 `public`、`private` 和 `protected`，但其所表达的含义并不相同。
    继承方式与访问控制会共同影响派生类成员的访问权限。

### 1. 公有继承（public inheritance）

公有继承是最常用的继承方式，它在派生类中保持了从基类中继承的成员的访问属性，最能体现 "is-a" 关系。

!!! info "公有继承的访问控制规则"

    | 基类成员属性 | 在派生类中的访问属性      |
    | :----------- | :------------------------ |
    | `public`     | **public**（保持不变）    |
    | `protected`  | **protected**（保持不变） |
    | `private`    | **不可直接访问**          |

    - **派生类成员函数**：可以直接访问基类的 `public` 和 `protected` 成员。
    - **外部代码（派生类对象）**：只能访问从基类继承的 `public` 成员。

!!! example "公有继承示例"

    ``` cpp linenums="1" hl_lines="22"
    #include <iostream>
    using namespace std;

    // 基类 Point
    class Point {
    public:
        void initPoint(float x = 0, float y = 0) {
            this->x = x;
            this->y = y;
        }
        void move(float offX, float offY) {
            x += offX;
            y += offY;
        }
        float getX() const { return x; }
        float getY() const { return y; }
    private:
        float x, y;      // 私有成员：派生类不可直接访问
    };

    // 派生类 Rectangle：公有继承 Point
    class Rectangle : public Point {
    public:
        void initRectangle(float x, float y, float w, float h) {
            initPoint(x, y);     // ✓ 可以访问基类的 public 成员函数
            this->w = w;
            this->h = h;
            // this->x = x;      // ✗ 错误！不能直接访问基类的 private 成员
        }

        float getW() const { return w; }
        float getH() const { return h; }

    private:
        float w, h;      // 新增成员
    };

    int main() {
        Rectangle rect;

        rect.initRectangle(2, 3, 20, 10);  // ✓ 调用派生类成员函数
        rect.move(3, 2);                   // ✓ 调用继承自基类的 public 成员函数
        cout << rect.getX() << endl;       // ✓ 调用继承自基类的 public 成员函数
        // cout << rect.x << endl;         // ✗ 错误！不能直接访问基类的 private 成员

        return 0;
    }
    ```

!!! tip "公有继承的语义"

    公有继承传达的是 **"is-a"（是一种）** 关系。如果 `Rectangle` 公有继承自 `Point`，
    意味着 **Rectangle 是一种 Point**。这种关系对外部代码是可见的：任何需要 `Point` 的地方
    都可以使用 `Rectangle` 对象（类型兼容原则）。

### 2. 私有继承（private inheritance）

私有继承是最严格的继承方式，它将基类的所有可访问成员都变为派生类的 `private` 成员。

!!! info "私有继承的访问控制规则"

    | 基类成员属性 | 在派生类中的访问属性 |
    | :----------- | :------------------- |
    | `public`     | **private**          |
    | `protected`  | **private**          |
    | `private`    | **不可直接访问**     |

    - **派生类成员函数**：可以直接访问基类的 `public` 和 `protected` 成员（在派生类中它们变为 `private`）。
    - **外部代码（派生类对象）**：**不能访问**任何从基类继承的成员（所有继承成员在派生类中都成了 `private`）。

!!! example "私有继承示例"

    ``` cpp linenums="1" hl_lines="21"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        void initPoint(float x = 0, float y = 0) {
            this->x = x;
            this->y = y;
        }
        void move(float offX, float offY) {
            x += offX;
            y += offY;
        }
        float getX() const { return x; }
        float getY() const { return y; }
    private:
        float x, y;
    };

    // Rectangle：私有继承 Point
    class Rectangle : private Point {
    public:
        void initRectangle(float x, float y, float w, float h) {
            initPoint(x, y);     // ✓ 可以访问（但在派生类中为 private）
            this->w = w;
            this->h = h;
        }

        // 派生类为从 Point 继承的成员提供公共访问接口，因为从 Point 继承的成员在派生类中为 private
        void move(float offX, float offY) {
            Point::move(offX, offY);   // 通过基类名限定调用
        }

        float getX() const { return Point::getX(); }
        float getY() const { return Point::getY(); }

        float getW() const { return w; }
        float getH() const { return h; }

    private:
        float w, h;
    };

    int main() {
        Rectangle rect;
        rect.initRectangle(2, 3, 20, 10);

        rect.move(3, 2);             // ✓ move() 在派生类中为 private
        rect.Rectangle::move(3, 2);  // ✓ 也可通过派生类名限定（但很少这样做）

        // 必须通过派生类提供的公有接口访问
        cout << rect.getX() << endl;   // ✓

        // cout << rect.Point::getX(); // 因为 Point::getX 在派生类中为 private，外部不能访问

        return 0;
    }
    ```

!!! tip "私有继承的语义"

    私有继承表达的是 **"is-implemented-in-terms-of"（用...来实现）** 关系。它主要用于代码实现层面的复用，而不是接口层面的继承。外部代码不应知道派生类与基类之间的关系。

### 3. 保护继承（protected inheritance）

保护继承介于公有继承和私有继承之间，它将基类的可访问成员变为派生类的 `protected` 成员。

!!! info "保护继承的访问控制规则"

    | 基类成员属性 | 在派生类中的访问属性      |
    | :----------- | :------------------------ |
    | `public`     | **protected**             |
    | `protected`  | **protected**（保持不变） |
    | `private`    | **不可直接访问**          |

    - **派生类成员函数**：可以直接访问基类的 `public` 和 `protected` 成员（在派生类中它们为 `protected`）。
    - **外部代码（派生类对象）**：**不能访问**任何从基类继承的成员。

!!! example "保护继承示例"

    ``` cpp linenums="1" hl_lines="11 21"
    class Base {
    public:
        int pub = 1;
    protected:
        int prot = 2;
    private:
        int priv = 3;     // 派生类不可访问
    };

    // 保护继承 Base
    class Derived : protected Base {
    public:
        void show() {
            cout << pub << endl;   // ✓ pub 在派生类中为 protected
            cout << prot << endl;  // ✓ prot 在派生类中仍为 protected
            // cout << priv;       // ✗ 不能访问基类的 private 成员
        }
    };

    // 进一步的派生类
    class GrandDerived : public Derived {
    public:
        void showMore() {
            cout << pub << endl;   // ✓ 仍然可以访问（GrandDerived 中仍为 protected）
            cout << prot << endl;  // ✓ 仍然可以访问
        }
    };

    int main() {
        Derived d;
        // cout << d.pub;          // ✗ 外部不能访问 protected 成员

        GrandDerived gd;
        // cout << gd.pub;         // ✗ 外部也不能访问

        return 0;
    }
    ```

!!! info "保护成员的特点"

    `protected` 成员在不同上下文中有不同的表现：

    | 上下文               | 对 protected 成员的访问权限 |
    | :------------------- | :-------------------------: |
    | 类本身（成员函数）   |          ✓ 可访问           |
    | 派生类（成员函数）   |          ✓ 可访问           |
    | 外部代码（通过对象） |         ✗ 不可访问          |

    这种设计既实现了数据隐藏，又方便了派生类的扩展。

## 三种继承方式的对比

### 对比总览

| 基类成员    | 继承方式 | 在派生类中的访问属性 | 派生类成员函数访问 | 派生类对象（外部）访问 |
| :---------- | :------- | :------------------- | :----------------: | :--------------------: |
| `public`    | 公有继承 | `public`             |         ✓          |           ✓            |
| `protected` | 公有继承 | `protected`          |         ✓          |           ✗            |
| `private`   | 公有继承 | 不可访问             |         ✗          |           ✗            |
| `public`    | 私有继承 | `private`            |         ✓          |           ✗            |
| `protected` | 私有继承 | `private`            |         ✓          |           ✗            |
| `private`   | 私有继承 | 不可访问             |         ✗          |           ✗            |
| `public`    | 保护继承 | `protected`          |         ✓          |           ✗            |
| `protected` | 保护继承 | `protected`          |         ✓          |           ✗            |
| `private`   | 保护继承 | 不可访问             |         ✗          |           ✗            |

### 继承方式选择指南

!!! tip "如何选择合适的继承方式"

    | 继承方式     | 适用场景                             | 语义                                        |
    | :----------- | :----------------------------------- | :------------------------------------------ |
    | **公有继承** | 默认选择，绝大数情况使用             | "is-a"（是一种）                            |
    | **私有继承** | 实现层面的代码复用，不想暴露基类接口 | "is-implemented-in-terms-of"（用...来实现） |
    | **保护继承** | 希望只在继承体系内部传递基类功能     | 介于公有和私有之间                          |

!!! abstract "设计原则"

    - **优先使用公有继承**：除非有明确理由，否则应使用公有继承。它最能表达 "is-a" 关系，也是最易于理解的继承方式。
    - **私有继承优先考虑组合**：在很多情况下，私有继承可以用"包含一个对象"（组合）来替代，而组合更清晰、更灵活。
    - **保护继承较少使用**：保护继承主要用于构建"继承树内部的共享"。

## 访问基类被隐藏的成员

当派生类中新增的成员与基类成员同名时，基类的同名成员会被"隐藏"。在派生类中可以通过**基类名限定**来访问被隐藏的成员。

!!! example "访问被隐藏的基类成员"

    ``` cpp linenums="1" hl_lines="29-30"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        int value = 10;
        void show() {
            cout << "Base::show()" << endl;
        }
    };

    class Derived : public Base {
    public:
        int value = 20;      // 隐藏了基类的 value
        void show() {        // 隐藏了基类的 show()
            cout << "Derived::show()" << endl;
        }

        void accessBase() {
            cout << "Derived value: " << value << endl;            // 20
            cout << "Base value: " << Base::value << endl;         // 10（通过基类名限定）
            Base::show();     // 调用基类的 show()
            show();           // 调用派生类的 show()
        }
    };

    int main() {
        Derived d;
        d.show();               // Derived::show()
        d.Base::show();         // Base::show()（通过基类名限定访问）

        cout << d.value << endl;        // 20
        cout << d.Base::value << endl;  // 10

        return 0;
    }
    ```

!!! info "同名隐藏与函数重载的区别"

    - **同名隐藏**：派生类中的同名成员会隐藏基类中所有同名的成员（即使参数列表不同）。
    - **函数重载**：在**同一个作用域**内，同名但参数列表不同的函数构成重载。

    因此，派生类中的 `fun(int)` 会隐藏基类中的 `fun(double)`，即使在基类中它们是不同的重载版本。

## 综合示例：三种继承方式的完整对比

!!! example "继承方式综合示例"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 基类
    class Base {
    public:
        int pub = 1;
        void pubFunc() {
            cout << "Base::pubFunc()" << endl;
        }

    protected:
        int prot = 2;
        void protFunc() {
            cout << "Base::protFunc()" << endl;
        }

    private:
        int priv = 3;
        void privFunc() {
            cout << "Base::privFunc()" << endl;
        }
    };

    // 公有继承
    class PublicDerived : public Base {
    public:
        void test() {
            cout << "PublicDerived test:" << endl;
            cout << pub << endl;      // ✓ 可以访问 public 成员
            cout << prot << endl;     // ✓ 可以访问 protected 成员
            // cout << priv;          // ✗ 不能访问 private 成员
            pubFunc();                // ✓
            protFunc();               // ✓
            // privFunc();            // ✗
        }
    };

    // 私有继承
    class PrivateDerived : private Base {
    public:
        void test() {
            cout << "PrivateDerived test:" << endl;
            cout << pub << endl;      // ✓ 可以访问（但在派生类中为 private）
            cout << prot << endl;     // ✓ 可以访问（但在派生类中为 private）
            pubFunc();                // ✓
            protFunc();               // ✓
        }

        // 需要手动暴露接口
        void accessPub() { pubFunc(); }
    };

    // 保护继承
    class ProtectedDerived : protected Base {
    public:
        void test() {
            cout << "ProtectedDerived test:" << endl;
            cout << pub << endl;      // ✓ 可以访问（在派生类中为 protected）
            cout << prot << endl;     // ✓ 可以访问（仍为 protected）
            pubFunc();                // ✓
            protFunc();               // ✓
        }
    };

    int main() {
        // 公有继承：外部可以访问 public 成员
        PublicDerived pd;
        pd.pubFunc();         // ✓
        // pd.protFunc();     // ✗ protected 不可访问
        // pd.privFunc();     // ✗ private 不可访问

        // 私有继承：外部不能访问任何基类成员
        PrivateDerived prd;
        // prd.pubFunc();     // ✗ 不可访问
        prd.accessPub();      // ✓ 通过派生类提供的接口

        // 保护继承：外部不能访问任何基类成员
        ProtectedDerived pod;
        // pod.pubFunc();     // ✗ 不可访问

        return 0;
    }
    ```

    运行结果分析

    ```
    PublicDerived test:
    1
    2
    Base::pubFunc()
    Base::protFunc()
    PrivateDerived test:
    1
    2
    Base::pubFunc()
    Base::protFunc()
    ProtectedDerived test:
    1
    2
    Base::pubFunc()
    Base::protFunc()
    ```

## 小结

1.  **派生类定义语法**：`class 派生类名 : 继承方式 基类名 { ... };`

2.  **三种继承方式对比**：

    | 继承方式     | `public` 成员 | `protected` 成员 | `private` 成员 |
    | :----------- | :------------ | :--------------- | :------------- |
    | **公有继承** | → `public`    | → `protected`    | 不可访问       |
    | **私有继承** | → `private`   | → `private`      | 不可访问       |
    | **保护继承** | → `protected` | → `protected`    | 不可访问       |

3.  **选择指南**：
    1. **公有继承**：表达 "is-a" 关系，最常用。
    2. **私有继承**：表达 "用...来实现"，可用组合替代。
    3. **保护继承**：仅用于继承树内部共享，较少使用。

4.  **同名隐藏**：派生类中新成员会隐藏基类的同名成员，可通过 `基类名::成员名` 访问被隐藏的成员。

继承方式是派生类设计的"权限开关"，选择合适的继承方式是正确表达类型关系的关键。公有继承是最自然、最常用的继承方式，它让 "is-a" 关系对外可见，为多态打下基础。