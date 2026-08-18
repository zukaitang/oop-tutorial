# 运算符重载为成员函数

将运算符重载为类的成员函数是最直观、最常用的重载方式之一。通过成员函数重载，运算符可以直接访问类的私有成员，实现自然且封装良好的运算符行为。

## 成员函数重载的基本语法

### 定义形式

将运算符重载为类的成员函数，需要在类定义中声明一个特殊的成员函数，函数名由 `operator` 关键字和运算符符号组成。

!!! tip "成员函数重载的语法格式"

    ``` cpp
    class 类名 {
    public:
        返回类型 operator运算符(形参表) {
            // 运算符的实现
        }
    };
    ```

### 参数个数规则

成员函数形式的运算符重载，其**参数个数 = 原操作数个数 - 1**，因为第一个操作数隐含为 `this` 指针（即调用该成员函数的对象本身）。

!!! info "参数个数对照表"

    | 运算符类型                | 原操作数个数 | 成员函数参数个数 | 示例                      |
    | :------------------------ | :----------: | :--------------: | :------------------------ |
    | 一元运算符（如 `!`、`~`） |      1       |        0         | `obj.operator!()`         |
    | 前置 `++` / `--`          |      1       |        0         | `obj.operator++()`        |
    | 二元运算符（如 `+`、`-`） |      2       |        1         | `obj.operator+(other)`    |
    | 后置 `++` / `--`          |      1       |    1（`int`）    | `obj.operator++(0)`       |
    | 下标运算符 `[]`           |      2       |        1         | `obj.operator[](index)`   |
    | 函数调用 `()`             |     任意     |       任意       | `obj.operator()(args...)` |

### 复数类的完整示例

在数学中，复数天然支持加减法运算。通过成员函数重载 `+` 和 `-`，可以让复数类像内置类型一样使用运算符。

!!! example "复数类的成员函数运算符重载"

    ``` cpp linenums="1" hl_lines="17 22 27 32 37"
    #include <iostream>
    using namespace std;

    class Complex {
    private:
        double real;   // 实部
        double imag;   // 虚部

    public:
        // 构造函数
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        // === 运算符重载为成员函数 ===

        // 1. 加法运算符：c1 + c2
        // 参数个数：2 - 1 = 1（另一个操作数作为参数）
        Complex operator+(const Complex& other) const {
            return Complex(real + other.real, imag + other.imag);
        }

        // 2. 减法运算符：c1 - c2
        Complex operator-(const Complex& other) const {
            return Complex(real - other.real, imag - other.imag);
        }

        // 3. 相等比较：c1 == c2
        bool operator==(const Complex& other) const {
            return real == other.real && imag == other.imag;
        }

        // 4. 不等比较：c1 != c2（通过 == 实现）
        bool operator!=(const Complex& other) const {
            return !(*this == other);   // 复用 operator==
        }

        // 5. 一元负号：-c1
        Complex operator-() const {
            return Complex(-real, -imag);
        }

        // 6. 显示函数
        void display() const {
            cout << "(" << real << ", " << imag << ")" << endl;
        }
    };

    int main() {
        Complex c1(5, 4);
        Complex c2(2, 10);

        // 使用重载的运算符
        Complex c3 = c1 + c2;   // 等价于 c1.operator+(c2)
        Complex c4 = c1 - c2;   // 等价于 c1.operator-(c2)
        Complex c5 = -c1;       // 等价于 c1.operator-()

        cout << "c1 = ";
        c1.display();
        cout << "c2 = ";
        c2.display();
        cout << "c1 + c2 = ";
        c3.display();
        cout << "c1 - c2 = ";
        c4.display();
        cout << "-c1 = ";
        c5.display();

        // 关系运算
        cout << "c1 == c2? " << (c1 == c2 ? "Yes" : "No") << endl;
        cout << "c1 != c2? " << (c1 != c2 ? "Yes" : "No") << endl;
        cout << "c1 == c1? " << (c1 == c1 ? "Yes" : "No") << endl;

        return 0;
    }
    ```

    运行结果

    ```
    c1 = (5, 4)
    c2 = (2, 10)
    c1 + c2 = (7, 14)
    c1 - c2 = (3, -6)
    -c1 = (-5, -4)
    c1 == c2? No
    c1 != c2? Yes
    c1 == c1? Yes

    ```

!!! info "成员函数运算符的调用方式解析"

    当编译器遇到 `c1 + c2` 时，会将其转换为成员函数调用：

    ``` cpp
    c1 + c2      // 原始表达式
    ↓
    c1.operator+(c2)   // 编译器转换
    ```

    其中：
    - `c1` 作为隐含的 `this` 指针传递给成员函数。
    - `c2` 作为显式参数传递给成员函数。

## 自增/自减运算符的重载

### 前置与后置的区别

自增运算符 `++` 和自减运算符 `--` 有前置和后置两种形式，它们的重载方式不同。

!!! info "前置与后置的区分"

    | 形式     | 写法    | 成员函数签名      | 说明                            |
    | :------- | :------ | :---------------- | :------------------------------ |
    | **前置** | `++obj` | `operator++()`    | 无参数，返回 `*this` 的引用     |
    | **后置** | `obj++` | `operator++(int)` | 有一个 `int` 参数（仅用于区分） |

    - 后置版本中的 `int` 参数是**占位匿名参数**，用于与前置版本区分，函数体中不需要使用它。
    - 通常**调用时传递值为 0**（编译器自动处理）。

!!! example "时钟类的前置/后置自增"

    ``` cpp linenums="1" hl_lines="26 33"
    #include <iostream>
    using namespace std;

    class Clock {
    private:
        int hour, minute, second;

        // 时间增加1秒的内部函数
        void tick() {
            second++;
            if (second >= 60) {
                second = 0;
                minute++;
                if (minute >= 60) {
                    minute = 0;
                    hour = (hour + 1) % 24;
                }
            }
        }

    public:
        Clock(int h = 0, int m = 0, int s = 0) : hour(h), minute(m), second(s) {}

        // 前置自增：++obj
        // 参数个数：0，返回引用
        Clock& operator++() {
            tick();                 // 先增加
            return *this;           // 返回自身引用（新值）
        }

        // 后置自增：obj++
        // 参数个数：1（int占位），返回值（旧值）
        Clock operator++(int) {
            Clock old = *this;      // 保存旧值
            tick();                 // 增加
            return old;             // 返回旧值
        }

        void show() const {
            cout << hour << ":" << minute << ":" << second << endl;
        }
    };

    int main() {
        Clock c(23, 59, 59);

        cout << "初始值: ";
        c.show();   // 23:59:59

        // 后置自增：先使用，后增加
        (c++).show();   // 显示旧值 23:59:59，然后 c 变为 0:0:0

        cout << "后置自增后: ";
        c.show();   // 0:0:0

        // 前置自增：先增加，后使用
        (++c).show();   // c 变为 0:0:1，然后显示

        return 0;
    }
    ```

    运行结果

    ```
    初始值: 23:59:59
    23:59:59
    后置自增后: 0:0:0
    0:0:1
    ```

### 前置与后置的对比

!!! summary "前置 vs 后置"

    | 对比项           | 前置 `++obj`             | 后置 `obj++`                 |
    | :--------------- | :----------------------- | :--------------------------- |
    | **成员函数签名** | `T& operator++()`        | `T operator++(int)`          |
    | **参数**         | 无                       | 一个 `int` 参数（占位）      |
    | **返回值**       | `T&`（返回自身引用）     | `T`（返回旧值的副本）        |
    | **行为**         | 先修改，后返回           | 先保存旧值，后修改，返回旧值 |
    | **性能**         | 效率高（无临时对象）     | 效率略低（需要复制旧值）     |
    | **建议**         | 优先使用前置（性能更好） | 仅在需要旧值时使用后置       |

## 复合赋值运算符的重载

复合赋值运算符（`+=`、`-=`、`*=` 等）通常重载为成员函数，并返回 `*this` 的引用以支持链式操作。

!!! info "复合赋值运算符的特点"

    - 返回值：通常返回 `*this` 的引用（`类名&`），支持 `a += b += c`。
    - 语义：修改当前对象的状态，而不是创建新对象。

!!! example "复合赋值运算符的实现"

    ``` cpp linenums="1" hl_lines="10 16 23 41 47"
    class Complex {
    private:
        double real, imag;

    public:
        Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

        // 复合赋值运算符：c1 += c2
        // 返回引用，支持链式操作
        Complex& operator+=(const Complex& other) {
            real += other.real;
            imag += other.imag;
            return *this;   // 返回当前对象的引用
        }

        Complex& operator-=(const Complex& other) {
            real -= other.real;
            imag -= other.imag;
            return *this;
        }

        // 普通加法：通过复合赋值实现
        Complex operator+(const Complex& other) const {
            Complex result = *this;   // 复制当前对象
            result += other;          // 复用 operator+=
            return result;
        }

        void display() const {
            cout << "(" << real << ", " << imag << ")" << endl;
        }
    };

    int main() {
        Complex c1(5, 4);
        Complex c2(2, 10);

        cout << "c1 = ";
        c1.display();

        c1 += c2;
        cout << "c1 += c2 → ";
        c1.display();   // (7, 14)

        // 链式操作
        Complex c3(1, 1);
        c1 += c2 += c3;   // 先执行 c2 += c3，再执行 c1 += 结果

        cout << "c1 after chain = ";
        c1.display();

        return 0;
    }
    ```

## 下标运算符的重载

下标运算符 `[]` 用于实现类似数组的索引访问，通常提供 const 和非 const 两个版本。

!!! example "动态数组类的下标运算符"

    ``` cpp linenums="1"  hl_lines="20 26 38-40 46"
    #include <iostream>
    #include <cassert>
    using namespace std;

    class IntArray {
    private:
        int* data;
        size_t size;

    public:
        IntArray(size_t n) : size(n) {
            data = new int[n];
        }

        ~IntArray() {
            delete[] data;
        }

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

        size_t getSize() const { return size; }
    };

    int main() {
        IntArray arr(5);

        // 调用非 const 版本：可修改
        arr[0] = 10;
        arr[1] = 20;
        arr[2] = 30;

        cout << "arr[0] = " << arr[0] << endl;   // 10

        // 通过 const 引用访问：调用 const 版本，只读
        const IntArray& constRef = arr;
        cout << "constRef[1] = " << constRef[1] << endl;   // 20
        // constRef[2] = 100;   // 错误！const 版本返回 const int&，不可修改

        return 0;
    }
    ```

## 成员函数重载的优势与限制

### 优势

!!! success "成员函数重载的优点"

    - **直接访问私有成员**：成员函数可以访问所有私有成员，无需友元声明。
    - **封装性更好**：运算符的逻辑与类的数据封装在同一个类中。
    - **`this` 指针可用**：可以方便地使用 `*this` 返回当前对象。
    - **调用简洁**：无需额外传参，左操作数隐含为 `this`。

### 限制

!!! warning "成员函数重载的限制"

    当运算符的左操作数**不是本类对象**时，无法使用成员函数重载：

    ``` cpp
    class Complex {
    public:
        Complex operator+(const Complex& other) const;
    };

    int main() {
        Complex c(3, 4);

        // ✓ 正确：左操作数是 Complex
        Complex c2 = c + 5.0;   // 如果 Complex 有对应的构造函数

        // ✗ 错误：左操作数是 double，不是 Complex
        Complex c3 = 5.0 + c;   // 无法调用 Complex::operator+
    }
    ```

    !!! note "解决方案"

        对于左操作数不是本类对象的场景，可以使用**非成员函数**重载（详见第三部分），或者提供适当的**类型转换构造函数**。

        ``` cpp
        // 方案一：非成员函数
        Complex operator+(double d, const Complex& c);

        // 方案二：隐式类型转换（如果 Complex 有 double 构造函数）
        // 5.0 会通过 Complex(double) 构造临时 Complex 对象，然后调用成员函数
        // 但这可能产生不易察觉的转换行为，需谨慎使用
        ```

## 综合示例

!!! example "完整的学生成绩类"

    ``` 
    cpp linenums="1"

    # include <iostream>
    # include <string>
    # include <vector>

    using namespace std;

    class Student {
    private:
    string name;
    vector<double> scores;

    public:
    Student(const string& n = "") : name(n) {}

        // 添加成绩
        void addScore(double s) {
            scores.push_back(s);
        }

        // 重载 +=：追加成绩
        Student& operator+=(double score) {
            scores.push_back(score);
            return *this;
        }

        // 重载 []：访问成绩（只读）
        double operator[](size_t index) const {
            if (index >= scores.size()) {
                throw out_of_range("Index out of range");
            }
            return scores[index];
        }

        // 重载 ++：计算平均分（前置）
        double operator++() {
            if (scores.empty()) return 0;
            double sum = 0;
            for (double s : scores) sum += s;
            return sum / scores.size();
        }

        // 重载 !：判断是否及格（所有成绩 >= 60）
        bool operator!() const {
            for (double s : scores) {
                if (s < 60) return false;
            }
            return !scores.empty();
        }

        // 重载 ==：比较姓名
        bool operator==(const Student& other) const {
            return name == other.name;
        }

        void show() const {
            cout << name << ": ";
            for (double s : scores) cout << s << " ";
            cout << endl;
        }

    };

    int main() {
    Student s1("张三");

        // 使用 += 添加成绩
        s1 += 85.5;
        s1 += 92.0;
        s1 += 78.5;

        s1.show();   // 张三: 85.5 92 78.5

        // 使用 [] 访问成绩
        cout << "成绩[1] = " << s1[1] << endl;   // 92

        // 使用 ++ 计算平均分
        double avg = ++s1;
        cout << "平均分 = " << avg << endl;   // 85.333...

        // 使用 ! 判断是否及格
        cout << "全部及格? " << (!s1 ? "是" : "否") << endl;   // 是

        return 0;

    }
    ```

## 小结

成员函数是运算符重载最基本、最常用的形式。它让运算符的逻辑与类的数据紧密封装在一起，是实现自然运算符行为的重要工具。

1.  **成员函数重载的语法**：`返回类型 operator运算符(参数) { ... }`。

2.  **参数个数规则**：
    - 一元运算符：0 个参数（`this` 作为左操作数）。
    - 二元运算符：1 个参数（右操作数）。
    - 后置 `++`/`--`：1 个 `int` 参数（占位）。

3.  **返回值约定**：
    - 复合赋值运算符（`+=` 等）：返回 `*this` 的引用。
    - 前置 `++`/`--`：返回 `*this` 的引用。
    - 后置 `++`/`--`：返回旧值的副本（非引用）。
    - 算术运算符（`+` 等）：返回新创建的对象。

4.  **典型应用**：
    - 算术运算符（复数类的 `+`、`-`）。
    - 复合赋值运算符（`+=`、`-=`）。
    - 自增/自减运算符（前置和后置）。
    - 下标运算符（`[]`，提供 const 和非 const 版本）。

5.  **限制**：当左操作数不是本类对象时，成员函数重载无法处理，需使用非成员函数。
