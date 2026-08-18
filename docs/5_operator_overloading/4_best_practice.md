# 常用运算符重载与最佳实践

运算符重载不仅要掌握语法，更重要的是理解各类运算符的**语义约定**和**设计准则**。本节将系统介绍常用运算符的重载方式，并总结核心的实践原则。

> **说明**：运算符重载的基本语法（成员函数 vs 非成员函数）已在前两节详细介绍，本节聚焦于"各类运算符如何重载"以及"如何正确地重载"。

## 常用运算符重载

### 赋值运算符 `=`

赋值运算符是必须重点关注的运算符，它涉及深拷贝、自赋值检查和资源管理。

!!! info "赋值运算符的语义约定"

    - **返回值**：返回 `*this` 的引用（支持链式赋值 `a = b = c`）。
    - **自赋值检查**：防止 `obj = obj` 时错误释放资源。
    - **资源管理**：释放旧资源，复制新资源（深拷贝）。

!!! example "赋值运算符的典型实现"

    ``` cpp linenums="1"
    class String {
    private:
        char* data;
        size_t len;
    public:
        // 赋值运算符
        String& operator=(const String& other) {
            // 1. 自赋值检查
            if (this == &other) {
                return *this;
            }

            // 2. 释放旧资源
            delete[] data;

            // 3. 分配新资源并复制内容
            len = other.len;
            data = new char[len + 1];
            strcpy(data, other.data);

            // 4. 返回自身引用
            return *this;
        }

        // 移动赋值（C++11，提升性能）
        String& operator=(String&& other) noexcept {
            if (this == &other) return *this;
            delete[] data;
            data = other.data;
            len = other.len;
            other.data = nullptr;
            other.len = 0;
            return *this;
        }
    };
    ```

!!! tip "现代 C++ 的改进"

    使用 **拷贝交换（Copy-and-Swap）** 惯用法可以简化赋值运算符的实现：

    ``` cpp
    String& operator=(String other) {   // 传值即拷贝
        swap(data, other.data);
        swap(len, other.len);
        return *this;
    }
    ```

    这种方式同时支持拷贝赋值和移动赋值，代码更简洁、异常安全。

### 关系运算符 `==`、`!=`、`<`、`>`、`<=`、`>=`

关系运算符通常成对实现，保持逻辑一致性。

!!! info "语义约定"

    - `==` 表示"相等"，`!=` 通常通过 `!(*this == other)` 实现。
    - `<` 表示"小于"，其他比较运算符可以通过 `<` 组合实现。
    - 关系运算符通常实现为**非成员函数**，以支持左右操作数的对称性。

!!! example "关系运算符的实现"

    ``` cpp linenums="1"
    class Complex {
    private:
        double real, imag;
    public:
        // 友元声明（或通过 getter 实现）
        friend bool operator==(const Complex& a, const Complex& b);
        friend bool operator!=(const Complex& a, const Complex& b);
        friend bool operator<(const Complex& a, const Complex& b);
    };

    // 相等：实部和虚部都相等
    bool operator==(const Complex& a, const Complex& b) {
        return a.real == b.real && a.imag == b.imag;
    }

    // 不等：通过 == 实现
    bool operator!=(const Complex& a, const Complex& b) {
        return !(a == b);
    }

    // 小于：按模长比较（示例）
    bool operator<(const Complex& a, const Complex& b) {
        return (a.real * a.real + a.imag * a.imag) <
               (b.real * b.real + b.imag * b.imag);
    }
    ```

!!! tip "C++20 的改进"

    C++20 引入了**三路比较运算符（Spaceship Operator）** `<=>`，编译器可以自动生成所有关系运算符：

    ``` cpp
    class Point {
        int x, y;
    public:
        auto operator<=>(const Point&) const = default;  // 自动生成全部比较运算符
    };
    ```

### 算术运算符与复合赋值运算符

算术运算符（`+`、`-`、`*`、`/`）通常配合复合赋值运算符（`+=`、`-=` 等）一起实现，以减少重复代码。

!!! info "实现策略"

    1. 先实现复合赋值运算符（`+=`、`-=` 等）。
    2. 再通过复合赋值运算符实现对应的算术运算符。
    3. 算术运算符通常实现为**非成员函数**，以支持混合类型运算（如 `3 + obj`）。

!!! example "算术运算符的实现"

    ``` cpp linenums="1"
    class Complex {
    private:
        double real, imag;
    public:
        // 复合赋值运算符（成员函数）
        Complex& operator+=(const Complex& other) {
            real += other.real;
            imag += other.imag;
            return *this;
        }

        Complex& operator-=(const Complex& other) {
            real -= other.real;
            imag -= other.imag;
            return *this;
        }

        // 友元声明（用于非成员算术运算符）
        friend Complex operator+(Complex a, const Complex& b);
        friend Complex operator-(Complex a, const Complex& b);
    };

    // 算术运算符（非成员函数）：通过复合赋值实现
    Complex operator+(Complex a, const Complex& b) {
        a += b;   // 复用 operator+=
        return a;
    }

    Complex operator-(Complex a, const Complex& b) {
        a -= b;   // 复用 operator-=
        return a;
    }
    ```

!!! note "参数传递技巧"

    算术运算符的第一个参数**按值传递**，这样可以直接修改副本并返回，无需额外的临时对象。这种写法简洁且高效（结合了移动语义）。

### 下标运算符 `[]`

下标运算符用于实现类似数组的访问行为，通常需要提供两个版本：非 `const` 版本（可修改）和 `const` 版本（只读）。

!!! example "下标运算符的实现"

    ``` cpp linenums="1"
    class Array {
    private:
        int* data;
        size_t size;
    public:
        // 非 const 版本：返回引用，允许修改
        int& operator[](size_t index) {
            assert(index < size);
            return data[index];
        }

        // const 版本：返回 const 引用，只读
        const int& operator[](size_t index) const {
            assert(index < size);
            return data[index];
        }
    };

    int main() {
        Array arr(5);

        arr[0] = 10;      // 调用非 const 版本，可修改

        const Array& ref = arr;
        int x = ref[0];   // 调用 const 版本，只读

        return 0;
    }
    ```

!!! tip "返回值类型的选择"

    | 场景         | 返回类型                        | 说明                         |
    | :----------- | :------------------------------ | :--------------------------- |
    | 普通数组     | `T&` / `const T&`               | 最基本的实现                 |
    | 容器类       | `reference` / `const_reference` | 使用类型别名增强可读性       |
    | 需要代理行为 | `proxy` 类型                    | 如 `vector<bool>` 的特殊实现 |

### 函数调用运算符 `()`

重载 `()` 可以将对象变为**函数对象（Functor）**，使对象可以像函数一样被调用。

!!! example "函数对象的实现"

    ``` cpp linenums="1"
    #include <algorithm>
    #include <vector>

    // 一元函数对象：判断是否为偶数
    class IsEven {
    public:
        bool operator()(int x) const {
            return x % 2 == 0;
        }
    };

    // 带状态的函数对象：按阈值比较
    class GreaterThan {
    private:
        int threshold;
    public:
        GreaterThan(int t) : threshold(t) {}

        bool operator()(int x) const {
            return x > threshold;
        }
    };

    int main() {
        std::vector<int> v = {1, 2, 3, 4, 5, 6};

        // 使用函数对象
        IsEven isEven;
        int count = std::count_if(v.begin(), v.end(), isEven);   // 3

        // 使用带状态的函数对象
        GreaterThan gt(3);
        int count2 = std::count_if(v.begin(), v.end(), gt);      // 3（4、5、6）

        return 0;
    }
    ```

!!! success "函数对象的优势"

    - **带状态**：函数对象可以持有成员变量，保存状态信息（如 `GreaterThan` 中的 `threshold`）。
    - **内联优化**：编译器更容易内联函数对象的调用，性能优于函数指针。
    - **类型安全**：每个函数对象都有独立的类型，支持模板特化。

### 流插入/提取运算符 `<<` 和 `>>`

流运算符必须实现为**非成员函数**，因为左操作数是 `std::ostream` 或 `std::istream`，不是自定义类的对象。

!!! example "流运算符的实现"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Point {
    private:
        int x, y;
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}

        // 友元声明
        friend ostream& operator<<(ostream& os, const Point& p);
        friend istream& operator>>(istream& is, Point& p);
    };

    // 输出运算符
    ostream& operator<<(ostream& os, const Point& p) {
        os << "(" << p.x << ", " << p.y << ")";
        return os;   // 返回流引用，支持链式输出
    }

    // 输入运算符
    istream& operator>>(istream& is, Point& p) {
        char ch;
        is >> ch;   // 读取 '('
        if (ch != '(') {
            is.setstate(ios_base::failbit);
            return is;
        }
        is >> p.x >> ch;   // 读取 x 和 ','
        is >> p.y >> ch;   // 读取 y 和 ')'
        return is;   // 返回流引用，支持链式输入
    }

    int main() {
        Point p1(3, 4), p2;

        cout << "p1 = " << p1 << endl;          // p1 = (3, 4)

        cout << "Enter point (format: (x, y)): ";
        cin >> p2;                               // 输入 "(5, 6)"
        cout << "p2 = " << p2 << endl;           // p2 = (5, 6)

        cout << "Multiple: " << p1 << " and " << p2 << endl;
        return 0;
    }
    ```

!!! warning "流输入运算符的错误处理"

    应该检查输入状态，在格式错误时设置 `failbit`，以便调用者可以通过 `cin.fail()` 检测错误。

## 常见陷阱与注意事项

!!! danger "需要特别注意的问题"

    **1. 自赋值问题（赋值运算符）**

    ``` cpp
    // 没有自赋值检查的赋值运算符
    String& operator=(const String& other) {
        delete[] data;              // 如果 this == &other，data 已被删除！
        data = new char[other.len];
        // ...
    }

    // 正确：先检查自赋值，或使用 Copy-and-Swap 惯用法
    ```

    **2. 短路求值失效（`&&` 和 `||`）**

    ``` cpp
    // 重载 && 后，不再有短路求值
    if (p != nullptr && p->value() > 10) {
        // 重载 && 后，两个操作数都会被求值！
        // 如果 p == nullptr，p->value() 仍会被调用 → 崩溃
    }
    ```

    **3. 临时对象的生命周期**

    ``` cpp
    Complex operator+(const Complex& a, const Complex& b) {
        return Complex(a.real + b.real, a.imag + b.imag);
        // 返回的是临时对象，值语义，安全
    }

    const Complex& operator+(const Complex& a, const Complex& b) {
        Complex result(...);
        return result;   // 危险！返回局部对象的引用
    }
    ```

    **4. 成员函数 vs 非成员函数的选择**

    | 场景                     | 推荐方式      | 原因                       |
    | :----------------------- | :------------ | :------------------------- |
    | 左操作数是 `this` 类型   | 成员函数      | `obj + 3` 可工作           |
    | 左操作数不是 `this` 类型 | 非成员函数    | `3 + obj` 才能工作         |
    | 需要访问私有成员         | 非成员 + 友元 | 保持封装的同时支持混合运算 |

## 综合示例

!!! example "完善 Vector3D 类"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cmath>
    using namespace std;

    class Vector3D {
    private:
        double x, y, z;

    public:
        // === 构造函数 ===
        Vector3D(double x = 0, double y = 0, double z = 0) : x(x), y(y), z(z) {}

        // === 算术运算符（成员函数） ===
        Vector3D& operator+=(const Vector3D& other) {
            x += other.x; y += other.y; z += other.z;
            return *this;
        }

        Vector3D& operator-=(const Vector3D& other) {
            x -= other.x; y -= other.y; z -= other.z;
            return *this;
        }

        Vector3D& operator*=(double scalar) {
            x *= scalar; y *= scalar; z *= scalar;
            return *this;
        }

        // === 算术运算符（非成员，通过复合赋值实现） ===
        friend Vector3D operator+(Vector3D a, const Vector3D& b) {
            a += b;
            return a;
        }

        friend Vector3D operator-(Vector3D a, const Vector3D& b) {
            a -= b;
            return a;
        }

        friend Vector3D operator*(Vector3D a, double s) {
            a *= s;
            return a;
        }

        friend Vector3D operator*(double s, Vector3D a) {
            a *= s;
            return a;
        }

        // === 关系运算符 ===
        friend bool operator==(const Vector3D& a, const Vector3D& b) {
            return a.x == b.x && a.y == b.y && a.z == b.z;
        }

        friend bool operator!=(const Vector3D& a, const Vector3D& b) {
            return !(a == b);
        }

        // === 下标运算符 ===
        double& operator[](int index) {
            assert(index >= 0 && index < 3);
            return index == 0 ? x : (index == 1 ? y : z);
        }

        const double& operator[](int index) const {
            assert(index >= 0 && index < 3);
            return index == 0 ? x : (index == 1 ? y : z);
        }

        // === 流运算符 ===
        friend ostream& operator<<(ostream& os, const Vector3D& v) {
            os << "(" << v.x << ", " << v.y << ", " << v.z << ")";
            return os;
        }

        friend istream& operator>>(istream& is, Vector3D& v) {
            char ch;
            is >> ch;
            if (ch != '(') { is.setstate(ios_base::failbit); return is; }
            is >> v.x >> ch >> v.y >> ch >> v.z >> ch;
            return is;
        }

        // === 额外功能 ===
        double length() const {
            return sqrt(x*x + y*y + z*z);
        }
    };

    int main() {
        Vector3D v1(1, 2, 3), v2(4, 5, 6);

        // 算术运算
        Vector3D v3 = v1 + v2;
        Vector3D v4 = v1 * 2.0;
        Vector3D v5 = 3.0 * v1;

        cout << "v1 = " << v1 << endl;
        cout << "v2 = " << v2 << endl;
        cout << "v1 + v2 = " << v3 << endl;
        cout << "v1 * 2 = " << v4 << endl;
        cout << "3 * v1 = " << v5 << endl;

        // 关系运算
        cout << "v1 == v2? " << (v1 == v2 ? "Yes" : "No") << endl;
        cout << "v1 == v1? " << (v1 == v1 ? "Yes" : "No") << endl;

        // 下标访问
        cout << "v1[0] = " << v1[0] << ", v1[1] = " << v1[1] << ", v1[2] = " << v1[2] << endl;

        return 0;
    }
    ```

## 小结

1. **赋值运算符 `=`**：注意深拷贝和自赋值检查，使用 Copy-and-Swap 惯用法简化实现。

2. **关系运算符**：保持逻辑一致性，`!=` 通过 `==` 实现。

3. **算术/复合赋值运算符**：先实现复合赋值，算术运算符通过复合赋值实现，避免代码重复。

4. **下标运算符 `[]`**：提供 const 和非 const 两个版本，支持读写分离。

5. **函数调用运算符 `()`**：将对象变为函数对象，支持状态保存和内联优化。

6. **流运算符 `<<` / `>>`**：必须是非成员函数，注意输入错误处理。

7. **设计准则**：保持自然语义、一致性和直觉。避免重载 `&&`、`||` 和 `,` 运算符。

8. **常见陷阱**：自赋值、短路求值失效、返回临时对象的引用、成员/非成员的选择。