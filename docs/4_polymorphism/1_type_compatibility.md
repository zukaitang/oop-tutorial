# 类型兼容性

类型兼容性是面向对象程序设计中的一个重要概念，它描述了基类与派生类之间可以相互使用的规则。理解这些规则是掌握多态的基础。

## 从 C 语言说起：类型兼容性的直觉

在C语言中，不同类型之间存在一定的兼容关系。这种兼容性主要基于**内存布局的匹配**：

!!! example "C语言中的类型兼容"

    ``` c linenums="1"
    #include <stdio.h>

    // 定义一个结构体
    typedef struct {
        int x;
        int y;
    } Point;

    // 定义一个"更大"的结构体（包含 Point 的所有成员）
    typedef struct {
        int x;
        int y;
        int z;
    } Point3D;

    void printPoint(const Point* p) {
        printf("(%d, %d)\n", p->x, p->y);
    }

    int main() {
        Point p = {3, 4};
        printPoint(&p);     // ✓ 参数类型完全匹配

        Point3D p3 = {5, 6, 7};
        // C语言中：可以传递 Point3D* 给 Point* 类型的参数吗？
        // printPoint(&p3); // ⚠️ 编译警告，但可以运行

        // 更危险的是：可以这样访问
        Point* pp = (Point*)&p3;
        printf("p3 as Point: (%d, %d)\n", pp->x, pp->y);  // (5, 6)

        return 0;
    }
    ```

!!! info "C语言兼容性的特点"

    - **基于内存布局**：如果两个结构体的前几个成员相同，在内存布局上存在重叠，可以相互转换。
    - **类型不安全**：编译器通常只给出警告，不强制阻止不兼容的转换。
    - **手动转换**：需要使用显式类型转换（如 `(Point*)&p3`）。
    - **没有语义保证**：`Point3D` 并不是 `Point` 的一种类型，它们只是内存布局有重叠。

## C++ 的类型兼容规则

C++ 在继承体系中建立了更严格、更安全的类型兼容规则。**公有继承**传达了 **"is-a"（是一种）** 关系，这意味着派生类对象可以被当作基类对象使用。

!!! abstract "核心规则"

    如果 `Derived` 公有继承自 `Base`，则：

    - `Derived` 的对象可以当作 `Base` 的对象使用。
    - 反之，`Base` 的对象**不能**当作 `Derived` 的对象使用。

    这是因为派生类包含了基类的所有成员（"是一种"关系），但基类不一定包含派生类新增的成员。

!!! tip "直观理解"

    - 汽车是车辆 → 可以用汽车的地方，车辆也能用吗？❌ 不是所有车辆都是汽车。
    - 所以：派生类 → 基类（向上转换）是安全的；基类 → 派生类（向下转换）是不安全的。

### 1. 派生类对象可以隐含转换为基类对象

当派生类对象赋值给基类对象时，只有派生类中从基类继承的部分被复制。

!!! example "派生类到基类的对象转换"

    ``` cpp linenums="1" hl_lines="30"
    #include <iostream>
    using namespace std;

    class Vehicle {
    public:
        int wheels;
        float weight;

        Vehicle(int w = 0, float wt = 0) : wheels(w), weight(wt) {}
        void show() const {
            cout << "Vehicle: wheels=" << wheels << ", weight=" << weight << endl;
        }
    };

    class Car : public Vehicle {
    public:
        int passenger;

        Car(int w, float wt, int p) : Vehicle(w, wt), passenger(p) {}
        void showCar() const {
            cout << "Car: wheels=" << wheels << ", weight=" << weight
                 << ", passenger=" << passenger << endl;
        }
    };

    int main() {
        Car car(4, 1.5, 5);

        // 派生类对象隐含转换为基类对象
        Vehicle v = car;   // 切片：只复制 Vehicle 部分

        v.show();          // Vehicle: wheels=4, weight=1.5
        // v.showCar();    // ❌ 错误！v 是 Vehicle 类型，没有 showCar()

        return 0;
    }
    ```

!!! warning "切片问题"

    当派生类对象赋值给基类对象时，会发生**切片（Slicing）**：派生类新增的成员被"切掉"，只保留基类部分。
    这是危险的，因为派生类特有的信息永久丢失。

    ``` cpp
    Car car(4, 1.5, 5);
    Vehicle v = car;   // car 的 passenger 信息丢失！
    ```

### 2. 派生类对象可以初始化基类的引用

基类引用可以绑定到派生类对象，这不会发生切片，因为引用只是一个别名。
通过基类引用访问对象时，只能访问基类中的成员。

!!! example "派生类对象初始化基类引用"

    ``` cpp linenums="1" hl_lines="28"
    #include <iostream>
    using namespace std;

    class Vehicle {
    public:
        int wheels;
        float weight;
        Vehicle(int w = 0, float wt = 0) : wheels(w), weight(wt) {}
        void show() const {
            cout << "Vehicle: wheels=" << wheels << ", weight=" << weight << endl;
        }
    };

    class Car : public Vehicle {
    public:
        int passenger;
        Car(int w, float wt, int p) : Vehicle(w, wt), passenger(p) {}
        void showCar() const {
            cout << "Car: wheels=" << wheels << ", weight=" << weight
                 << ", passenger=" << passenger << endl;
        }
    };

    int main() {
        Car car(4, 1.5, 5);

        // 基类引用绑定到派生类对象
        Vehicle& vRef = car;   // 不切片，只是别名

        vRef.show();           // Vehicle: wheels=4, weight=1.5
        // vRef.showCar();    // ❌ 错误！Vehicle& 没有 showCar()

        // 但通过引用操作的对象仍然是 car
        vRef.weight = 2.0;
        car.showCar();         // Car: wheels=4, weight=2, passenger=5

        return 0;
    }
    ```

### 3. 派生类指针可以隐含转换为基类指针

派生类指针可以隐含转换为基类指针，这是实现**多态**的基础。通过基类指针访问对象时，也只能访问基类中的成员。

!!! example "派生类指针到基类指针的转换"

    ``` cpp linenums="1"  hl_lines="38 39"
    #include <iostream>
    using namespace std;

    class Vehicle {
    public:
        virtual void show() const {
            cout << "Vehicle::show()" << endl;
        }
        virtual ~Vehicle() {}
    };

    class Car : public Vehicle {
    public:
        void show() const override {
            cout << "Car::show()" << endl;
        }
        void special() const {
            cout << "Car special method" << endl;
        }
    };

    class Truck : public Vehicle {
    public:
        void show() const override {
            cout << "Truck::show()" << endl;
        }
    };

    void display(const Vehicle& v) {
        v.show();   // 运行时多态：调用实际类型的 show()
    }

    int main() {
        Car car;
        Truck truck;

        // 派生类指针隐含转换为基类指针
        Vehicle* p1 = &car;
        Vehicle* p2 = &truck;

        display(*p1);   // Car::show()
        display(*p2);   // Truck::show()

        // 通过基类指针只能访问基类成员
        // p1->special(); // ❌ 错误！Vehicle* 没有 special()

        return 0;
    }
    ```

!!! note "基类指针只能访问基类成员"

    通过基类指针操作派生类对象时，只能访问从基类继承的成员。派生类新增的成员是不可见的。这就是**类型兼容规则的"单向性"**——向上转换可行，向下转换受限。

## 类型兼容的"单向性"

类型兼容规则是**单向**的：

!!! success "向上转换（Upcasting）——安全"

    ``` cpp
    Derived d;
    Base* p = &d;      // ✓ 隐含转换，安全
    Base& r = d;       // ✓ 隐含转换，安全
    Base b = d;        // ✓ 隐含转换，安全（但切片）
    ```

    向上转换是安全的，因为派生类包含基类的全部成员。编译器允许隐式转换。

!!! danger "向下转换（Downcasting）——不安全"

    ``` cpp
    Base b;
    Derived* p = &b;   // ❌ 不能隐式转换！
    // 需要显式转换，但运行时可能出错
    ```

    向下转换是不安全的，因为基类对象不包含派生类新增的成员。编译器禁止隐式向下转换。

## C++ 中的类型转换方式

在需要显式类型转换时，C++ 提供了多种转换方式，每种有不同的安全级别和适用场景。

### 1. C风格强制转换（不推荐）

``` cpp
Base* pBase = new Derived();
Derived* pDerived = (Derived*)pBase;   // C风格强制转换
```

!!! warning "C风格转换的问题"

    - 几乎可以做任何转换，危险性高。
    - 编译器不会检查转换的合法性。
    - 在大型程序中难以追踪和调试。

### 2. static_cast（编译时检查）

`static_cast` 用于编译时已知的类型转换，包括安全的向上转换和需要确认的向下转换。

!!! example "static_cast 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Vehicle {
    public:
        virtual ~Vehicle() {}
    };
    class Car : public Vehicle {};

    class Truck : public Vehicle {};

    int main() {
        // 向上转换：安全，通常不需要显式转换
        Car car;
        Vehicle* pV = static_cast<Vehicle*>(&car);   // 相当于隐式转换

        // 向下转换：不安全，但 static_cast 不检查
        Vehicle* pV2 = new Car();
        Car* pC = static_cast<Car*>(pV2);    // 程序员的承诺
        // 如果 pV2 实际指向 Truck，static_cast 不会报错，但访问会出问题

        // 基本类型转换
        int a = 10;
        double d = static_cast<double>(a);

        return 0;
    }
    ```

### 3. dynamic_cast（运行时检查）

`dynamic_cast` 用于运行时安全的向下转换和跨类型转换。它会检查转换是否合法，如果转换失败：

- 对指针返回 `nullptr`。
- 对引用抛出 `bad_cast` 异常。

!!! example "dynamic_cast 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Vehicle {
    public:
        virtual ~Vehicle() {}    // 需要虚函数表，dynamic_cast 才能工作
    };
    class Car : public Vehicle {
    public:
        void drive() { cout << "Driving car" << endl; }
    };
    class Truck : public Vehicle {
    public:
        void load() { cout << "Loading truck" << endl; }
    };

    void operate(Vehicle* p) {
        // 尝试转换为 Car*
        Car* pCar = dynamic_cast<Car*>(p);
        if (pCar) {
            pCar->drive();
            return;
        }

        // 尝试转换为 Truck*
        Truck* pTruck = dynamic_cast<Truck*>(p);
        if (pTruck) {
            pTruck->load();
            return;
        }

        cout << "Unknown vehicle type" << endl;
    }

    int main() {
        Vehicle* p1 = new Car();
        Vehicle* p2 = new Truck();

        operate(p1);   // Driving car
        operate(p2);   // Loading truck

        delete p1;
        delete p2;

        return 0;
    }
    ```

!!! info "dynamic_cast 的使用条件"

    - 基类必须有至少一个虚函数（通常为虚析构函数）。
    - 只能用于多态类型（有虚函数表）。
    - 运行时有一定开销（检查类型信息）。

### 4. reinterpret_cast（底层转换，危险）

`reinterpret_cast` 进行最低级别的转换，直接将一个指针转换为另一种类型指针。几乎不做任何检查，非常危险。

!!! warning "reinterpret_cast 的使用"

    ``` cpp
    int* pInt = new int(10);
    char* pChar = reinterpret_cast<char*>(pInt);   // 将 int* 转为 char*
    // 危险！尝试通过 pChar 访问会得到原始字节，而非字符
    ```

    **除非有特殊底层需求，否则应避免使用 `reinterpret_cast`。**

### 5. const_cast（添加/移除 const）

`const_cast` 用于添加或移除变量的 `const` 属性。通常用于调用需要非 `const` 参数的旧代码。

!!! example "const_cast 的使用"

    ``` cpp
    const int a = 10;
    // int* p = &a;        // ❌ 错误：不能将 const int* 转为 int*
    int* p = const_cast<int*>(&a);   // ✓ 移除 const
    *p = 20;              // 但修改 const 对象是未定义行为！

    // 正确的使用场景：传递参数给旧 API
    void legacyFunc(char* str);   // 旧 API 需要 char*
    const char* data = "hello";
    legacyFunc(const_cast<char*>(data));   // 如果函数不修改数据，这是安全的
    ```

## 类型转换方式对比

!!! summary "C++ 类型转换方式总结"

    | 转换方式             | 安全级别 | 运行时开销 | 适用场景                         |
    | :------------------- | :------: | :--------: | :------------------------------- |
    | **隐式转换**         |    高    |     无     | 向上转换、数值提升               |
    | **static_cast**      |    中    |     无     | 显式向上/向下转换、基本类型转换  |
    | **dynamic_cast**     |    高    |     有     | 运行时安全的向下转换、跨类型转换 |
    | **reinterpret_cast** |   极低   |     无     | 底层位操作，几乎不推荐           |
    | **const_cast**       |    低    |     无     | 添加/移除 const（谨慎使用）      |
    | **C风格转换**        |   极低   |     无     | 不推荐（现代C++应避免）          |

!!! tip "最佳实践"

    1. **优先使用隐式转换**：向上转换是安全的，尽量让编译器自动处理。
    2. **使用 `static_cast` 替代 C 风格转换**：让编译器能进行一定程度的检查。
    3. **多态场景使用 `dynamic_cast`**：当不确定对象实际类型时，用 `dynamic_cast` 安全转换。
    4. **避免 `reinterpret_cast` 和 `const_cast`**：除非确实需要底层操作或兼容旧代码。

## 综合示例

!!! example "C++类型兼容样例"

    ``` cpp linenums="1"
    #include <iostream>
    #include <typeinfo>
    using namespace std;

    // 基类 Shape
    class Shape {
    public:
        virtual void draw() const {
            cout << "Shape::draw()" << endl;
        }
        virtual ~Shape() {}
    };

    // 派生类 Circle
    class Circle : public Shape {
    private:
        double radius;
    public:
        Circle(double r = 1.0) : radius(r) {}
        void draw() const override {
            cout << "Circle::draw() with radius " << radius << endl;
        }
        double getRadius() const { return radius; }
    };

    // 派生类 Rectangle
    class Rectangle : public Shape {
    private:
        double width, height;
    public:
        Rectangle(double w = 1.0, double h = 1.0) : width(w), height(h) {}
        void draw() const override {
            cout << "Rectangle::draw() with " << width << "x" << height << endl;
        }
        double getArea() const { return width * height; }
    };

    // 全局绘制函数：接受基类引用
    void render(const Shape& shape) {
        shape.draw();   // 运行时多态
    }

    // 处理特定类型
    void processShape(Shape* p) {
        // 使用 dynamic_cast 检查实际类型
        Circle* pCircle = dynamic_cast<Circle*>(p);
        if (pCircle) {
            cout << "Processing Circle, radius = " << pCircle->getRadius() << endl;
            pCircle->draw();
            return;
        }

        Rectangle* pRect = dynamic_cast<Rectangle*>(p);
        if (pRect) {
            cout << "Processing Rectangle, area = " << pRect->getArea() << endl;
            pRect->draw();
            return;
        }

        cout << "Unknown shape" << endl;
    }

    int main() {
        cout << "=== 类型兼容的三种形式 ===" << endl;

        // 1. 派生类对象 → 基类对象（切片）
        Circle c(5.0);
        Shape s = c;   // 切片：Circle 的 radius 丢失
        s.draw();      // Shape::draw()（而不是 Circle::draw()）

        // 2. 派生类对象 → 基类引用（不切片）
        Circle c2(3.0);
        Shape& ref = c2;
        ref.draw();    // Circle::draw() with radius 3（多态生效）

        // 3. 派生类指针 → 基类指针（多态的基础）
        Shape* p1 = new Circle(4.0);
        Shape* p2 = new Rectangle(5.0, 3.0);

        render(*p1);   // Circle::draw() with radius 4
        render(*p2);   // Rectangle::draw() with 5x3

        cout << "\n=== 使用 dynamic_cast 安全转换 ===" << endl;
        processShape(p1);   // Processing Circle, radius = 4
        processShape(p2);   // Processing Rectangle, area = 15

        cout << "\n=== 错误的向下转换（不安全） ===" << endl;
        // 假设我们错误地认为 p1 是 Rectangle
        // Rectangle* pBad = static_cast<Rectangle*>(p1);  // 编译通过，但运行错误
        // pBad->getArea();   // 访问不存在的成员 → 未定义行为

        // 使用 dynamic_cast 安全地避免
        Rectangle* pSafe = dynamic_cast<Rectangle*>(p1);
        if (pSafe) {
            cout << "Area = " << pSafe->getArea() << endl;
        } else {
            cout << "p1 is not a Rectangle" << endl;   // 输出此消息
        }

        delete p1;
        delete p2;

        return 0;
    }
    ```

    运行结果

    ```
    === 类型兼容的三种形式 ===
    Shape::draw()
    Circle::draw() with radius 3
    Circle::draw() with radius 4
    Rectangle::draw() with 5x3

    === 使用 dynamic_cast 安全转换 ===
    Processing Circle, radius = 4
    Circle::draw() with radius 4
    Processing Rectangle, area = 15
    Rectangle::draw() with 5x3

    === 错误的向下转换（不安全） ===
    p1 is not a Rectangle
    ```

## 小结

1.  **类型兼容规则**（公有继承下）：
    - 派生类对象可以当作基类对象使用（向上转换）。
    - 基类对象不能当作派生类对象使用（向下转换不安全）。

2.  **三种表现形式**：
    - 派生类对象 → 基类对象（切片，派生类信息丢失）。
    - 派生类对象 → 基类引用（不切片，保留多态能力）。
    - 派生类指针 → 基类指针（多态的基础）。

3.  **C++ 类型转换方式**：
    - `static_cast`：编译时检查，用于安全转换和显式向下转换。
    - `dynamic_cast`：运行时检查，用于多态类型的向下转换。
    - `reinterpret_cast`：底层转换，危险。
    - `const_cast`：添加/移除 const。

4.  **选择建议**：
    - 向上转换：让编译器隐式转换。
    - 向下转换：优先使用 `dynamic_cast`（多态类型）或 `static_cast`（非多态，需程序员保证安全）。
    - 避免 C 风格强制转换。

类型兼容规则是面向对象多态的基础。理解这些规则，是正确使用虚函数、实现运行时多态的前提。下一部分将深入探讨成员覆盖与多态的实现机制。