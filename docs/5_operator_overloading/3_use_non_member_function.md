# 运算符重载为非成员函数

将运算符重载为非成员函数是成员函数方式的重要补充。当运算符的左操作数不是本类对象时，非成员函数是唯一的选择。典型场景包括流运算符、混合类型运算等。

## 非成员函数重载的基本概念

### 什么时候需要非成员函数

!!! abstract "成员函数的局限性"

    回顾上一部分，成员函数运算符的**左操作数必须是本类对象**（即 `this` 指向的对象）：

    ``` cpp
    class Complex {
    public:
        Complex operator+(const Complex& other) const;  // 左操作数必须是 Complex
    };

    Complex c(3, 4);
    Complex c2 = c + 5.0;   // ✓ 左操作数是 Complex
    // Complex c3 = 5.0 + c;   // ✗ 错误！double 不是 Complex，无法调用成员函数
    ```

    但在实际编程中，我们期望支持 `5.0 + c` 这样的对称运算，也期望能够像 `cout << c` 这样使用流运算符。**非成员函数**正是为这些场景设计的。

!!! info "非成员函数重载的语法格式"

    ``` cpp
    返回类型 operator运算符(操作数1, 操作数2) {
        // 运算符的实现
    }
    ```

    与成员函数不同，非成员函数的**参数个数 = 原操作数个数**（后置 `++`/`--` 除外）。

### 何时必须使用非成员函数

!!! summary "三种必须使用非成员函数的场景"

    | 场景                         | 示例                       | 说明                                  |
    | :--------------------------- | :------------------------- | :------------------------------------ |
    | **左操作数不是类类型**       | `5.0 + c`                  | 左操作数是 `double`，无法调用成员函数 |
    | **左操作数是标准库类型**     | `cout << c`                | 不能修改 `std::ostream` 的源代码      |
    | **左操作数指向的类不可修改** | 无法为第三方类添加成员函数 | 需要在外部定义运算符                  |

    **强制要求**：非成员函数重载的运算符，**至少有一个操作数是自定义类型**。

## 非成员函数与友元

### 访问私有成员的问题

非成员函数不属于类，默认情况下无法访问类的私有成员。当运算符需要访问对象的内部状态时，有两种解决方案：

!!! tip "两种解决方案"

    | 方案             | 实现方式                       | 适用场景                           |
    | :--------------- | :----------------------------- | :--------------------------------- |
    | **使用公共接口** | 通过 `getX()`、`getY()` 等访问 | 类已经提供了足够的访问接口         |
    | **声明为友元**   | 在类中用 `friend` 声明         | 需要直接访问私有成员，避免接口膨胀 |

### 友元函数的声明

!!! example "通过友元访问私有成员"

    ``` cpp linenums="1" hl_lines="6 7 10"
    class Complex {
    public:
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        // 声明非成员函数为友元：允许访问私有成员
        friend Complex operator+(const Complex& a, const Complex& b);
        friend Complex operator-(const Complex& a, const Complex& b);

        // 声明流运算符为友元
        friend ostream& operator<<(ostream& os, const Complex& c);
    private:
        double real, imag;
    };

    // 非成员函数实现：可以直接访问 real 和 imag
    Complex operator+(const Complex& a, const Complex& b) {
        return Complex(a.real + b.real, a.imag + b.imag);
    }

    Complex operator-(const Complex& a, const Complex& b) {
        return Complex(a.real - b.real, a.imag - b.imag);
    }

    ostream& operator<<(ostream& os, const Complex& c) {
        os << "(" << c.real << ", " << c.imag << ")";
        return os;
    }
    ```

!!! note "友元 vs 公共接口"

    ``` cpp
    // 方式一：友元（推荐用于运算符重载）
    class Point {
        friend Point operator+(const Point& a, const Point& b);
    private:
        int x, y;
    };

    // 方式二：公共接口（仅当类已有 getter 时使用）
    class Point {
    public:
        int getX() const { return x; }
        int getY() const { return y; }
    private:
        int x, y;
    };

    Point operator+(const Point& a, const Point& b) {
        return Point(a.getX() + b.getX(), a.getY() + b.getY());
    }
    ```

    对于运算符重载，友元是更自然的选择——它让运算符能够直接访问内部状态，同时保持了封装（只有被声明的函数能访问）。

## 输出/输入运算符的重载

流运算符 `<<` 和 `>>` 是最典型的非成员函数重载场景，因为左操作数是 `std::ostream` 或 `std::istream`，无法修改这些标准库类型。

### 输出运算符 `<<`

!!! example "重载 `<<` 支持自定义输出"

    ``` cpp linenums="1" hl_lines="9 15 24"
    #include <iostream>
    using namespace std;

    class Complex {
    public:
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        // 友元声明
        friend ostream& operator<<(ostream& os, const Complex& c);
    private:
        double real, imag;
    };

    // 输出运算符实现
    ostream& operator<<(ostream& os, const Complex& c) {
        os << "(" << c.real << ", " << c.imag << ")";
        return os;   // 返回流引用，支持链式输出
    }

    int main() {
        Complex c1(3, 4), c2(5, 6);

        // 链式输出
        cout << "c1 = " << c1 << ", c2 = " << c2 << endl;
        // 输出：c1 = (3, 4), c2 = (5, 6)

        return 0;
    }
    ```

!!! info "输出运算符的规范"

    | 规范           | 说明                                           |
    | :------------- | :--------------------------------------------- |
    | **返回类型**   | `ostream&`（流引用）                           |
    | **第一个参数** | `ostream&`（流对象引用，非常量，需要修改状态） |
    | **第二个参数** | `const 类名&`（要输出的对象，只读）            |
    | **返回值**     | `os`（流引用，支持链式调用）                   |
    | **格式**       | 不应输出多余的换行（由调用者决定）             |

### 输入运算符 `>>`

!!! example "重载 `>>` 支持自定义输入"

    ``` cpp linenums="1" hl_lines="10 22 45"
    #include <iostream>
    #include <string>
    using namespace std;

    class Complex {
    public:
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        friend ostream& operator<<(ostream& os, const Complex& c);
        friend istream& operator>>(istream& is, Complex& c);

    private:
        double real, imag;
    };

    ostream& operator<<(ostream& os, const Complex& c) {
        os << "(" << c.real << ", " << c.imag << ")";
        return os;
    }

    // 输入运算符实现：格式为 (real, imag)
    istream& operator>>(istream& is, Complex& c) {
        char ch;
        is >> ch;   // 读取 '('
        if (ch != '(') {
            is.setstate(ios_base::failbit);   // 设置错误标志
            return is;
        }
        is >> c.real >> ch;   // 读取 real 和 ','
        if (ch != ',') {
            is.setstate(ios_base::failbit);
            return is;
        }
        is >> c.imag >> ch;   // 读取 imag 和 ')'
        if (ch != ')') {
            is.setstate(ios_base::failbit);
        }
        return is;
    }

    int main() {
        Complex c1, c2;

        cout << "请输入两个复数（格式：(3, 4)）：" << endl;
        cin >> c1 >> c2;   // 链式输入

        cout << "c1 = " << c1 << endl;
        cout << "c2 = " << c2 << endl;

        return 0;
    }
    ```

!!! info "输入运算符的规范"

    | 规范           | 说明                            |
    | :------------- | :------------------------------ |
    | **返回类型**   | `istream&`（流引用）            |
    | **第一个参数** | `istream&`（流对象引用）        |
    | **第二个参数** | `类名&`（要读取的对象，可修改） |
    | **返回值**     | `is`（流引用，支持链式调用）    |
    | **错误处理**   | 格式错误时设置 `failbit`        |

## 支持混合类型运算

非成员函数的一个重要应用是支持**混合类型运算**，让自定义类型能够与内置类型或其他类型进行运算。

### 对称的混合运算

!!! example "复数与实数的混合运算"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Complex {
    private:
        double real, imag;

    public:
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        // 支持隐式转换的构造函数（允许 double → Complex）
        // Complex(double r) : real(r), imag(0) {}

        friend Complex operator+(const Complex& a, const Complex& b);
        friend Complex operator+(const Complex& a, double d);
        friend Complex operator+(double d, const Complex& a);

        friend ostream& operator<<(ostream& os, const Complex& c);
    };

    // 复数 + 复数
    Complex operator+(const Complex& a, const Complex& b) {
        return Complex(a.real + b.real, a.imag + b.imag);
    }

    // 复数 + 实数
    Complex operator+(const Complex& a, double d) {
        return Complex(a.real + d, a.imag);
    }

    // 实数 + 复数（保证对称性）
    Complex operator+(double d, const Complex& a) {
        return Complex(a.real + d, a.imag);
    }

    ostream& operator<<(ostream& os, const Complex& c) {
        os << "(" << c.real << ", " << c.imag << ")";
        return os;
    }

    int main() {
        Complex c1(3, 4);
        Complex c2(5, 6);

        Complex c3 = c1 + 5.0;   // 复数 + 实数
        Complex c4 = 7.0 + c1;   // 实数 + 复数
        Complex c5 = c1 + c2;    // 复数 + 复数

        cout << "c1 = " << c1 << endl;
        cout << "c1 + 5 = " << c3 << endl;
        cout << "7 + c1 = " << c4 << endl;
        cout << "c1 + c2 = " << c5 << endl;

        return 0;
    }
    ```

### 与转换构造函数的互动

!!! warning "注意：避免二义性"

    当同时提供非成员混合运算符和转换构造函数时，可能产生二义性：

    ``` cpp
    class Complex {
    public:
        Complex(double r = 0.0, double i = 0.0);   // 转换构造函数
        friend Complex operator+(const Complex& a, const Complex& b);
    };

    // 问题：c + 5.0 可以匹配：
    // 1. operator+(c, Complex(5.0))  → 通过转换构造函数
    // 2. 如果存在 operator+(c, double)，也有匹配

    // 解决方案：提供一组精确匹配的运算符，让编译器正确选择
    ```

## 非成员函数 vs 成员函数：如何选择

!!! summary "选择指南"

    | 场景                    | 推荐方式       | 理由                           |
    | :---------------------- | :------------- | :----------------------------- |
    | 左操作数是本类对象      | 成员函数       | 更简洁，直接访问私有成员       |
    | 左操作数不是本类对象    | **非成员函数** | 必须使用                       |
    | 需要支持混合类型运算    | 非成员函数     | 支持左右操作数的对称性         |
    | 流运算符 `<<` / `>>`    | **非成员函数** | 左操作数是 `ostream`/`istream` |
    | 一元运算符（`!`、`~`）  | 成员函数       | 左操作数就是本类对象           |
    | 算术运算符（`+`、`-`）  | 非成员函数     | 支持混合类型和对称性           |
    | 复合赋值运算符（`+=`）  | 成员函数       | 修改 `this`，返回 `*this` 引用 |
    | 关系运算符（`==`、`<`） | 非成员函数     | 支持左右操作数的对称性         |
    | 下标运算符 `[]`         | 成员函数       | 左操作数必须是本类对象         |

!!! tip "最佳实践：优先考虑非成员函数"

    著名 C++ 专家 Scott Meyers 在其著作《Effective C++》中建议：

    > **优先将运算符重载为非成员函数，除非必须为成员函数。**（尽量保证操作数的对称性）

    非成员函数能够保证 `a + b` 和 `b + a` 的行为一致，而成员函数可能因类型转换而产生不对称。

### 典型案例：重载关系运算符

关系运算符通常重载为非成员函数，保证左右操作数的对称性。

!!! example "关系运算符的完整实现"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cmath>
    using namespace std;

    class Point {
    private:
        double x, y;

    public:
        Point(double x = 0, double y = 0) : x(x), y(y) {}

        // 友元声明：关系运算符需要访问私有成员
        friend bool operator==(const Point& a, const Point& b);
        friend bool operator!=(const Point& a, const Point& b);
        friend bool operator<(const Point& a, const Point& b);
        friend bool operator>(const Point& a, const Point& b);
        friend bool operator<=(const Point& a, const Point& b);
        friend bool operator>=(const Point& a, const Point& b);

        friend ostream& operator<<(ostream& os, const Point& p);
    };

    // 相等：坐标完全相同
    bool operator==(const Point& a, const Point& b) {
        return a.x == b.x && a.y == b.y;
    }

    // 不等：通过 == 实现
    bool operator!=(const Point& a, const Point& b) {
        return !(a == b);
    }

    // 小于：按到原点的距离比较
    bool operator<(const Point& a, const Point& b) {
        double distA = sqrt(a.x * a.x + a.y * a.y);
        double distB = sqrt(b.x * b.x + b.y * b.y);
        return distA < distB;
    }

    // 大于：通过 < 实现（交换操作数）
    bool operator>(const Point& a, const Point& b) {
        return b < a;
    }

    // 小于等于：通过 < 和 == 实现
    bool operator<=(const Point& a, const Point& b) {
        return (a < b) || (a == b);
    }

    // 大于等于：通过 < 和 == 实现
    bool operator>=(const Point& a, const Point& b) {
        return (a > b) || (a == b);
    }

    ostream& operator<<(ostream& os, const Point& p) {
        os << "(" << p.x << ", " << p.y << ")";
        return os;
    }

    int main() {
        Point p1(3, 4), p2(3, 4), p3(5, 0);

        cout << "p1 = " << p1 << endl;
        cout << "p2 = " << p2 << endl;
        cout << "p3 = " << p3 << endl;

        cout << "p1 == p2? " << (p1 == p2 ? "Yes" : "No") << endl;   // Yes
        cout << "p1 != p3? " << (p1 != p3 ? "Yes" : "No") << endl;   // Yes
        cout << "p1 < p3? " << (p1 < p3 ? "Yes" : "No") << endl;     // No (距离相同)
        cout << "p1 > p3? " << (p1 > p3 ? "Yes" : "No") << endl;     // No (距离相同)
        cout << "p3 < p1? " << (p3 < p1 ? "Yes" : "No") << endl;     // No (距离相同)

        Point p4(0, 6);
        cout << "p1 < p4? " << (p1 < p4 ? "Yes" : "No") << endl;     // Yes (距离更近)

        return 0;
    }
    ```

## 综合示例

!!! example "非成员函数重载"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cmath>
    using namespace std;

    class Vector3D {
    private:
        double x, y, z;

    public:
        Vector3D(double x = 0, double y = 0, double z = 0) : x(x), y(y), z(z) {}

        // ===== 友元声明 =====

        // 算术运算符
        friend Vector3D operator+(const Vector3D& a, const Vector3D& b);
        friend Vector3D operator-(const Vector3D& a, const Vector3D& b);
        friend Vector3D operator*(const Vector3D& v, double s);
        friend Vector3D operator*(double s, const Vector3D& v);

        // 关系运算符
        friend bool operator==(const Vector3D& a, const Vector3D& b);

        // 流运算符
        friend ostream& operator<<(ostream& os, const Vector3D& v);
        friend istream& operator>>(istream& is, Vector3D& v);

        // 复合赋值（仍为成员函数）
        Vector3D& operator+=(const Vector3D& other);
    };

    // ===== 算术运算符 =====
    Vector3D operator+(const Vector3D& a, const Vector3D& b) {
        return Vector3D(a.x + b.x, a.y + b.y, a.z + b.z);
    }

    Vector3D operator-(const Vector3D& a, const Vector3D& b) {
        return Vector3D(a.x - b.x, a.y - b.y, a.z - b.z);
    }

    Vector3D operator*(const Vector3D& v, double s) {
        return Vector3D(v.x * s, v.y * s, v.z * s);
    }

    Vector3D operator*(double s, const Vector3D& v) {
        return v * s;   // 复用 Vector3D * double
    }

    // ===== 复合赋值（成员函数） =====
    Vector3D& Vector3D::operator+=(const Vector3D& other) {
        x += other.x;
        y += other.y;
        z += other.z;
        return *this;
    }

    // ===== 关系运算符 =====
    bool operator==(const Vector3D& a, const Vector3D& b) {
        return a.x == b.x && a.y == b.y && a.z == b.z;
    }

    // ===== 流运算符 =====
    ostream& operator<<(ostream& os, const Vector3D& v) {
        os << "(" << v.x << ", " << v.y << ", " << v.z << ")";
        return os;
    }

    istream& operator>>(istream& is, Vector3D& v) {
        char ch;
        is >> ch;   // '('
        if (ch != '(') { is.setstate(ios_base::failbit); return is; }
        is >> v.x >> ch;   // x 和 ','
        if (ch != ',') { is.setstate(ios_base::failbit); return is; }
        is >> v.y >> ch;   // y 和 ','
        if (ch != ',') { is.setstate(ios_base::failbit); return is; }
        is >> v.z >> ch;   // z 和 ')'
        if (ch != ')') { is.setstate(ios_base::failbit); }
        return is;
    }

    // ===== 主函数 =====
    int main() {
        Vector3D v1(1, 2, 3);
        Vector3D v2(4, 5, 6);

        cout << "v1 = " << v1 << endl;
        cout << "v2 = " << v2 << endl;

        // 混合运算
        Vector3D v3 = v1 + v2;
        Vector3D v4 = v1 * 2.0;
        Vector3D v5 = 3.0 * v1;

        cout << "v1 + v2 = " << v3 << endl;
        cout << "v1 * 2 = " << v4 << endl;
        cout << "3 * v1 = " << v5 << endl;

        // 关系运算
        cout << "v1 == v2? " << (v1 == v2 ? "Yes" : "No") << endl;

        // 链式输入输出
        Vector3D v6;
        cout << "请输入向量（格式：(1, 2, 3)）：";
        cin >> v6;
        cout << "您输入了：" << v6 << endl;

        return 0;
    }
    ```

## 小结

1.  **非成员函数重载的语法**：`返回类型 operator运算符(参数列表) { ... }`
2.  **必须使用非成员函数的场景**：
    - 左操作数不是类类型（如 `5.0 + c`）。
    - 左操作数是标准库类型（如 `cout << c`）。
    - 需要保证左右操作数的对称性。
3.  **访问私有成员的两种方式**：
    - **友元声明**：推荐，直接访问私有成员。
    - **公共接口**（getter）：适用于类已有公开访问接口。
4.  **流运算符重载**：
    - `<<`：返回 `ostream&`，格式为 `os << obj`。
    - `>>`：返回 `istream&`，注意错误处理。
5.  **成员函数 vs 非成员函数的选择**：
    - 左操作数是本类对象 → 成员函数。
    - 左操作数不是本类对象 → 非成员函数。
    - 算术运算符 → 优先非成员函数（保证对称性）。
    - 复合赋值运算符 → 成员函数（返回 `*this`）。
    - 下标运算符 → 成员函数（左操作数必须是本类对象）。