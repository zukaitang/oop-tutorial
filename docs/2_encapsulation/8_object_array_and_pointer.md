# 对象数组与对象指针

对象数组让程序能够批量管理同类型的对象，对象指针让程序能够间接操作对象。两者结合使用，为C++程序提供了灵活而强大的对象管理能力。

## 对象数组

### 对象数组的定义

对象数组中的每个元素都是同类型的对象，每个对象都有自己的数据成员。

!!! example "对象数组的定义"

    ``` cpp linenums="1" hl_lines="16-17"
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

    // 定义对象数组
    Point arr1[5];              // 包含5个Point对象，使用默认构造函数
    Point arr2[3] = {Point(1, 2), Point(3, 4), Point(5, 6)};  // 显式初始化
    ```

!!! info "对象数组的定义语法"

    - **基本形式**：`类名 数组名[元素个数];`
    - **访问元素**：通过下标访问，如 `arr[0]`、`arr[1]`。
    - **访问成员**：`数组名[下标].成员名`，如 `arr[0].show()`。

### 对象数组的初始化

对象数组的初始化依赖于类的构造函数。数组中的每个元素在创建时都会调用构造函数。

!!! example "对象数组的初始化方式"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        // 默认构造函数
        Point() : x(0), y(0) {
            cout << "Default Constructor called." << endl;
        }

        // 带参数构造函数
        Point(int x, int y) : x(x), y(y) {
            cout << "Constructor called: Point(" << x << ", " << y << ")" << endl;
        }

        // 析构函数
        ~Point() {
            cout << "Destructor called." << endl;
        }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        cout << "--- 方式一：使用默认构造函数 ---" << endl;
        Point a[2];              // 每个元素调用默认构造函数

        cout << "--- 方式二：显式初始化 ---" << endl;
        Point b[3] = {Point(1, 2), Point(3, 4), Point(5, 6)};

        cout << "--- 方式三：部分初始化 ---" << endl;
        Point c[3] = {Point(1, 2), Point(3, 4)};  // 第三个元素调用默认构造函数

        return 0;
    }
    ```

!!! info "对象数组初始化的规则"

    | 初始化方式                               | 构造函数调用                             | 说明                         |
    | :--------------------------------------- | :--------------------------------------- | :--------------------------- |
    | `Point a[2];`                            | 每个元素调用默认构造函数                 | 要求类有可访问的默认构造函数 |
    | `Point a[3] = {p1, p2, p3};`             | 用已有对象复制构造每个元素               | 调用复制构造函数             |
    | `Point a[3] = {Point(1,2), Point(3,4)};` | 第1、2个用带参构造，第3个用默认构造      | 类需有对应的构造函数         |
    | `Point a[3] = {1,2,3,4,5,6};`            | 用参数构造每个元素（需要对应的构造函数） | 较少使用，可读性差           |

!!! warning "注意"

    - 如果类没有默认构造函数，定义对象数组时必须为每个元素提供显式初始化。
    - 对象数组的元素在销毁时，每个元素都会调用析构函数，析构顺序与构造顺序相反。

### 对象数组的访问

通过下标访问对象数组的各个元素，然后使用 `.` 运算符调用成员函数或访问数据成员。

!!! example "对象数组的访问"

    ``` cpp linenums="1" hl_lines="29 30 34"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        int getX() const { return x; }
        int getY() const { return y; }

        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {
        Point points[3] = {Point(1, 2), Point(3, 4), Point(5, 6)};

        // 遍历数组，访问每个元素的成员
        for (int i = 0; i < 3; i++) {
            points[i].move(i * 2, i * 3);
            points[i].show();
        }

        // 通过下标读取数据
        cout << "Point 0 x = " << points[0].getX() << endl;

        return 0;
    }
    ```

### 对象数组在内存中的布局

对象数组在内存中是**连续存放**的，每个元素占据其类对象所需的内存空间。

!!! info "内存布局特点"

    - 数组元素按顺序连续存储。
    - 数组名表示数组首元素的地址。
    - 元素之间的地址间隔等于类对象的大小（`sizeof(类名)`）。

## 对象指针

### 指针的基本概念

指针是C/C++中重要的概念，它存储的是内存地址，通过地址可以间接访问内存单元。指针变量就是用于存放地址的变量。

!!! info "内存访问的两种方式"

    - **通过变量名访问**：编译器将变量名映射到内存地址，程序通过名字直接操作内存。
    - **通过地址访问**：程序先获取变量在内存中的起始地址，再通过该地址间接操作变量。

    ``` cpp linenums="1"
    int var = 10;
    int* ptr = &var;   // ptr 存储了 var 的地址
    *ptr = 20;         // 通过地址间接修改 var 的值
    ```

### 对象指针的定义与使用

对象指针是指向对象的指针变量，它存储的是对象在内存中的起始地址。

!!! example "对象指针的定义与使用"

    ``` cpp linenums="1" hl_lines="25 28-30 36-37"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        int getX() const { return x; }
        int getY() const { return y; }
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
        Point a(4, 5);           // 普通对象
        Point* ptr = &a;         // 对象指针：指向 a

        // 通过指针访问对象成员（使用 -> 运算符）
        cout << ptr->getX() << endl;   // 4
        ptr->move(1, 2);
        ptr->show();             // Point(5, 7)

        // 通过对象名访问成员（使用 . 运算符）
        cout << a.getX() << endl;      // 5

        // 使用 (*ptr).member 等价于 ptr->member
        (*ptr).move(1, 1);       // 解引用后使用点运算符
        ptr->show();             // Point(6, 8)

        return 0;
    }
    ```

!!! info "对象指针的语法要点"

    - **定义**：`类名* 指针变量名;`，如 `Point* ptr;`
    - **赋值**：`ptr = &对象;`，取对象的地址赋给指针。
    - **访问成员**：通过 `->` 运算符（箭头运算符）访问指针所指对象的成员。
    - **等价关系**：`ptr->member` 等价于 `(*ptr).member`。

!!! warning "`.` 与 `->` 的区别"

    | 访问方式        | 适用对象           | 示例                     |
    | :-------------- | :----------------- | :----------------------- |
    | `.` 点运算符    | 对象本身或对象引用 | `a.getX()`、`ref.getX()` |
    | `->` 箭头运算符 | 指向对象的指针     | `ptr->getX()`            |

### 通过对象指针遍历对象数组

对象指针常用于遍历对象数组，通过指针算术运算依次指向数组的每个元素。

!!! example "使用指针遍历对象数组"

    ``` cpp linenums="1" hl_lines="31-36 39-42"
    #include <iostream>
    using namespace std;

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
        Point points[3] = {Point(1, 2), Point(3, 4), Point(5, 6)};

        // 方式一：使用下标
        cout << "--- 使用下标 ---" << endl;
        for (int i = 0; i < 3; i++) {
            points[i].show();
        }

        // 方式二：使用指针遍历（数组名即首地址）
        cout << "--- 使用指针遍历 ---" << endl;
        Point* p = points;          // 等价于 &points[0]
        for (int i = 0; i < 3; i++) {
            p->show();              // p 指向当前元素
            p++;                    // 移动到下一个元素
        }

        // 方式三：使用指针和偏移量
        cout << "--- 使用指针和偏移量 ---" << endl;
        for (int i = 0; i < 3; i++) {
            (points + i)->show();   // 等价于 p[i].show()
        }

        return 0;
    }
    ```

!!! info "指针与数组的关系"

    - **数组名是常量指针**：数组名 `points` 表示数组首元素的地址，相当于 `&points[0]`。
    - **指针算术**：`p + i` 指向第 `i` 个元素的地址。
    - **等价关系**：`points[i]` 等价于 `*(points + i)`，也等价于 `p[i]`（当 `p` 指向数组首地址时）。

### 指向对象成员的指针

除了指向整个对象，指针也可以指向对象的成员（数据成员或成员函数）。

!!! example "指向数据成员的指针"

    ``` cpp linenums="1"
    class Point {
    public:
        int x, y;   // 公开数据成员（仅用于演示成员指针）
    };

    int main() {
        Point p;
        // 定义指向 Point 类 int 型数据成员的指针
        int Point::*ptr = &Point::x;

        p.*ptr = 10;                 // 通过成员指针赋值
        cout << p.x << endl;         // 10

        Point* pp = &p;
        pp->*ptr = 20;               // 通过对象指针 + 成员指针赋值
        cout << pp->x << endl;       // 20

        return 0;
    }
    ```

!!! example "指向成员函数的指针"

    ``` cpp linenums="1"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}
        int getX() const { return x; }
        int getY() const { return y; }
    private:
        int x, y;
    };

    int main() {
        // 定义指向 Point 类 const 成员函数（无参，返回 int）的指针
        int (Point::*funcPtr)() const = &Point::getX;

        Point p(3, 4);
        cout << (p.*funcPtr)() << endl;   // 3（通过对象调用）

        Point* pp = &p;
        cout << (pp->*funcPtr)() << endl;  // 3（通过对象指针调用）

        return 0;
    }
    ```

!!! info "成员指针的注意事项"

    - 成员指针不能指向普通函数，只能指向类的成员。
    - 调用成员指针时需要绑定到具体的对象（`p.*ptr` 或 `pp->*ptr`）。
    - 成员指针的语法较为复杂，在实际开发中使用频率较低。

## this 指针

### this 指针的概念

`this` 指针是面向对象程序设计中的重要概念，是 C++ 类中的一个**隐含指针**，
它存在于每个非静态成员函数中，指向**当前正在操作的对象**。

!!! info "this 指针的本质"

    - `this` 指针的类型是 `类名* const`（指向本类对象的常量指针）。
    - 它由编译器自动传递给成员函数，程序员无需显式定义。
    - `this` 指针的值就是当前对象的地址。
    - 它是成员函数知道“自己在为哪个对象工作”的关键机制。

    ``` cpp
    class Point {
    public:
        void setX(int value) {
            // 编译器实际处理为：
            // this->x = value;
            x = value;
        }
    private:
        int x;
    };
    ```

!!! abstract "this 指针的意义"

    当多个对象调用同一个成员函数时，系统会通过 `this` 指针区分出“当前操作的是哪个对象”。这就是为什么 `p1.setX(5)` 修改的是 `p1` 的 `x`，而 `p2.setX(10)` 修改的是 `p2` 的 `x`。

### this 指针的隐式使用

在成员函数中，直接访问的数据成员或调用的成员函数，实际上都隐式使用了 `this` 指针。

!!! example "this 指针的隐式使用"

    ``` cpp linenums="1" hl_lines="10-11 16"
    #include <iostream>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        void move(int dx, int dy) {
            // 以下两种写法等价：
            x += dx;           // 隐式使用 this
            this->y += dy;     // 显式使用 this
        }

        void show() const {
            // 这里也隐式使用了 this
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

        // 返回当前对象的引用（便于链式调用）
        Point& self() {
            return *this;
        }

    private:
        int x, y;
    };

    int main() {
        Point p1(3, 4);
        Point p2(5, 6);

        p1.move(1, 2);    // this 指向 p1
        p2.move(3, 4);    // this 指向 p2

        p1.show();        // Point(4, 6)
        p2.show();        // Point(8, 10)

        // 链式调用：self() 返回当前对象的引用
        p1.self().move(1, 1);
        p1.show();        // Point(5, 7)

        return 0;
    }
    ```

### this 指针的显式使用场景

虽然 `this` 通常被隐式使用，但在某些场景下需要显式使用：

!!! example "场景一：解决成员变量与参数同名问题"

    ``` cpp linenums="1" hl_lines="6-7"
    class Point {
    public:
        // 构造函数参数与数据成员同名
        Point(int x, int y) {
            // 必须使用 this 来区分参数和数据成员
            this->x = x;
            this->y = y;
        }
    private:
        int x, y;
    };
    ```

!!! example "场景二：返回当前对象的引用（链式调用）"

    ``` cpp linenums="1" hl_lines="5 10"
    class Calculator {
    public:
        Calculator& add(int n) {
            value += n;
            return *this;     // 返回当前对象的引用
        }

        Calculator& subtract(int n) {
            value -= n;
            return *this;
        }

        int getValue() const { return value; }

    private:
        int value = 0;
    };

    int main() {
        Calculator calc;
        // 链式调用
        int result = calc.add(5).subtract(2).add(10).getValue();
        cout << result << endl;   // 13
        return 0;
    }
    ```

!!! example "场景三：判断当前对象与另一个对象的关系"

    ``` cpp linenums="1" hl_lines="7 12"
    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        bool isSame(const Point& other) const {
            // 通过 this 和 &other 比较地址，判断是否是同一个对象
            return this == &other;
        }

        bool equals(const Point& other) const {
            // 比较两个对象的内容是否相同
            return x == other.x && y == other.y;
        }

    private:
        int x, y;
    };

    int main() {
        Point p1(3, 4);
        Point p2(3, 4);
        Point p3(5, 6);

        cout << p1.isSame(p2) << endl;    // 0（不同对象，地址不同）
        cout << p1.isSame(p1) << endl;    // 1（同一个对象）
        cout << p1.equals(p2) << endl;    // 1（内容相同）
        cout << p1.equals(p3) << endl;    // 0（内容不同）

        return 0;
    }
    ```

### 常成员函数与 this 指针

在常成员函数中，`this` 指针的类型有所不同——它是一个**指向常对象的常量指针**（`const 类名* const`），因此不能修改对象的数据成员。

!!! info "常成员函数中 this 指针的类型"

    | 成员函数类型      | this 指针类型            | 能否修改数据成员 |
    | :---------------- | :----------------------- | :--------------: |
    | 非 const 成员函数 | `类名* const this`       |     可以修改     |
    | const 成员函数    | `const 类名* const this` |     不能修改     |

    ``` cpp linenums="1"
    class Point {
    public:
        void move(int dx, int dy) {
            // this 类型：Point* const this
            x += dx;          // 可以修改
        }

        void show() const {
            // this 类型：const Point* const this
            cout << x << endl;   // 可以读取
            // x = 5;            // 错误！不能修改
        }
    private:
        int x, y;
    };
    ```

### this 指针的使用注意事项

!!! warning "常见错误"

    1. **静态成员函数没有 this 指针**：静态成员函数属于类而不是对象，因此不存在 `this` 指针。
    2. **不能取 this 的地址**：`this` 本身是一个右值，不能对其取地址。
    3. **this 指针不能被赋值**：`this = &other;` 是错误的。

    ``` cpp linenums="1"
    class Point {
    public:
        static void staticFunc() {
            // cout << this->x;   // 错误！静态成员函数没有 this 指针
        }

        void badOperation() {
            // this = &other;     // 错误！this 不能被赋值
        }
    };
    ```

## 对象指针与对象数组的应用场景

!!! tip "典型应用场景"

    | 场景               | 说明                                                   |
    | :----------------- | :----------------------------------------------------- |
    | **批量管理对象**   | 使用对象数组存储同类型对象的集合                       |
    | **函数间传递对象** | 使用对象指针在函数间传递对象，避免复制开销             |
    | **多态操作**       | 基类指针指向派生类对象，实现运行时多态（后续章节详述） |
    | **数据结构实现**   | 链表、树等数据结构中使用指针连接对象                   |
    | **链式调用**       | 通过返回 `*this` 实现连续的函数调用                    |
    | **对象标识**       | 通过 `this` 指针判断是否为同一对象                     |

## 小结

1. **对象数组**：批量管理同类型对象的容器，每个元素独立存储对象的数据成员。数组名即首元素地址。

2. **对象指针**：指向对象的指针变量，通过 `->` 运算符访问对象成员。指针可以独立于对象存在，也可以用于遍历数组。

3. **指针与数组的关系**：数组名是常量指针，指针算术运算可以遍历数组元素。

4. **this 指针**：
5. 隐含于每个非静态成员函数中，指向当前操作的对象。
6. 用于解决成员名与参数名冲突、返回当前对象、链式调用等。
7. 在常成员函数中，`this` 的类型为 `const 类名* const`，禁止修改数据成员。
8. 静态成员函数中没有 `this` 指针。