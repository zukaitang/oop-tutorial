# 引用

C++在C语言的基础上引入了**引用类型**和**范围 `for` 循环**两个重要的语法特性。
引用提供了更安全、更简洁的别名机制，而范围 `for` 循环则大大简化了遍历容器的代码。
这两个特性共同体现了C++"让代码更简洁、更安全"的设计理念。

## 引用类型

### 为什么需要引用

在C语言中，函数参数传递主要有两种方式：**传值**和**传指针**。传值会复制数据，对大型对象效率低；
传指针虽然高效，但语法繁琐且需要处理空指针。

!!! question "C语言中的问题"

    ``` c linenums="1"
    // C语言：使用指针交换两个变量的值
    void swap(int* a, int* b) {
        int temp = *a;
        *a = *b;
        *b = temp;
    }

    int main() {
        int x = 10, y = 20;
        swap(&x, &y);   // 需要取地址，语法繁琐
        return 0;
    }
    ```

### 引用的定义与基本用法

C++引入了**引用（Reference）** 类型，它是变量的**别名**，不占用独立的内存空间，与被引用的变量共享同一块内存。
引用提供了一种更简洁、更安全的方式来操作数据。

!!! abstract "引用的基本语法"

    ``` cpp
    类型& 引用名 = 变量名;   // 定义引用，必须初始化
    ```

!!! example "引用的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int x = 10;
        int& ref = x;   // ref 是 x 的引用（别名）

        cout << "x = " << x << endl;     // 10
        cout << "ref = " << ref << endl; // 10

        ref = 20;       // 修改 ref 即修改 x
        cout << "x = " << x << endl;     // 20

        x = 30;         // 修改 x 即修改 ref
        cout << "ref = " << ref << endl; // 30

        // 输出地址：x 和 ref 指向同一块内存
        cout << "&x = " << &x << endl;   // 相同地址
        cout << "&ref = " << &ref << endl; // 相同地址

        return 0;
    }
    ```

!!! info "引用的重要规则"

    1. **必须初始化**：引用定义时必须指定被引用的变量，不能像指针那样先定义后赋值。
    2. **不能重新绑定**：引用一旦绑定到某个变量，就不能再改变指向其他变量。
    3. **不是独立变量**：引用不占用独立的内存空间，本质上是变量的别名。

    ``` cpp
    int x = 10, y = 20;
    int& ref = x;   // ref 绑定到 x
    
    // ref = y;     // 这不是重新绑定，而是把 y 的值赋给 x（即 ref 指向的变量）
    // 此时 x 变为 20，但 ref 仍然绑定到 x
    ```

### 引用作为函数参数

引用最重要的应用场景是作为函数参数，实现**按引用传递（Pass by Reference）**。

!!! example "引用参数实现数据交换"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // C++风格：使用引用参数
    void swap(int& a, int& b) {
        int temp = a;
        a = b;
        b = temp;
    }

    int main() {
        int x = 10, y = 20;

        cout << "Before swap: x = " << x << ", y = " << y << endl;
        swap(x, y);   // 直接传递变量名，简洁自然
        cout << "After swap: x = " << x << ", y = " << y << endl;

        return 0;
    }
    ```

!!! success "引用参数 vs 指针参数"

    | 对比项           | 指针参数            | 引用参数               |
    | :--------------- | :------------------ | :--------------------- |
    | **语法**         | `void func(int* p)` | `void func(int& r)`    |
    | **调用方式**     | `func(&x)`          | `func(x)`              |
    | **是否可能为空** | 是（`nullptr`）     | 否（必须绑定有效变量） |
    | **可读性**       | 较繁琐              | 更简洁                 |
    | **安全性**       | 需检查空指针        | 自动保证非空           |
    | **能否重新绑定** | 能                  | 不能                   |

!!! tip "何时使用引用参数"

    - **需要修改实参**：使用引用参数，语法更简洁安全。
    - **传递大型对象**：使用 `const` 引用参数，避免拷贝且不修改数据。
    - **需要空值语义**：使用指针参数（因为引用不能为空）。

### 引用作为函数返回值

函数可以返回引用，常用于容器元素的访问、链式操作等场景。但需要注意**不能返回局部变量的引用**。

!!! example "引用返回值"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    // 返回数组元素的引用（可用于修改）
    int& getElement(vector<int>& arr, int index) {
        return arr[index];
    }

    // 返回常量引用（只读）
    const int& getConstElement(const vector<int>& arr, int index) {
        return arr[index];
    }

    // ❌ 错误示例：返回局部变量的引用
    // int& badFunc() {
    //     int local = 10;
    //     return local;   // 危险！local 在函数结束后被销毁
    // }


    int main() {
        // 通过引用修改数组元素
        vector<int> v = {10, 20, 30, 40, 50};
        getElement(v, 0) = 100;
        cout << "v[0] = " << v[0] << endl;   // 100

        // 常量引用只读
        cout << "v[1] = " << getConstElement(v, 1) << endl;   // 20

        return 0;
    }
    ```

!!! danger "返回引用的危险场景"

    ``` cpp
    // 返回局部变量的引用 → 悬垂引用（Dangling Reference）
    int& getLocal() {
        int x = 10;
        return x;   // 危险！x 在函数返回后被销毁
    }

    // 返回临时对象的引用 → 悬垂引用
    const string& getString() {
        return "hello";   // 危险！临时字符串在函数返回后被销毁
    }
    ```

### 常引用（const 引用）

`const` 引用（常引用）是C++中最常用的参数传递方式之一，它既保证了高效（不拷贝），又保证了安全（不修改）。

!!! info "常引用的特点"

    - **只读访问**：不能通过常引用修改所绑定的变量。
    - **延长临时对象生命周期**：常引用可以绑定到临时对象，并延长其生命周期。
    - **函数重载**：常引用与非常量引用可以构成重载。

!!! example "常引用的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    // 使用 const 引用：只读，不拷贝
    void printString(const string& s) {
        cout << s << endl;
        // s += "!";   // 错误！不能修改 const 引用
    }

    // 常引用 vs 非常量引用
    void modifyRef(int& r) {
        r = 100;   // 可以修改
    }

    void readRef(const int& r) {
        cout << r << endl;
        // r = 100;   // 错误！不能修改
    }

    int main() {
        // 1. 常引用绑定到普通变量
        int x = 10;
        const int& ref = x;
        cout << ref << endl;   // 可以读取
        // ref = 20;           // 错误！不能通过常引用修改

        // 2. 常引用绑定到临时对象（延长生命周期）
        const string& temp = string("temporary");
        cout << temp << endl;  // 合法！临时对象生命周期被延长

        // 3. 常引用作为函数参数
        string text = "Hello";
        printString(text);     // 不拷贝

        // 4. 非常量引用不能绑定到临时对象
        // int& bad = 10;      // 错误！不能绑定到右值
        const int& good = 10;  // 正确！常引用可以绑定到右值

        return 0;
    }
    ```

### 引用 vs 指针

!!! summary "引用与指针的全面对比"

    | 对比项             | 引用（`&`）                  | 指针（`*`）                  |
    | :----------------- | :--------------------------- | :--------------------------- |
    | **是否必须初始化** | 必须初始化                   | 可以初始化，也可以先声明     |
    | **能否重新绑定**   | 不能                         | 能                           |
    | **能否为空**       | 不能（始终指向有效对象）     | 能（`nullptr`）              |
    | **访问成员语法**   | `.`（与普通变量相同）        | `->` 或 `*`                  |
    | **作为函数参数**   | 自动传递地址，无需取地址     | 需显式取地址                 |
    | **内存占用**       | 不占用独立空间（编译器实现） | 占用指针变量空间             |
    | **使用场景**       | 函数参数传递、返回值、别名   | 动态内存、空值语义、遍历数组 |

## 参数传递方式对比

在C++中，引用常用作函数的参数。理解不同参数传递方式的区别，是写出高效、安全代码的关键。

!!! example "四种参数传递方式的对比"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    struct Student {
        String id;
        String name;
        String department;    
    };

    // 方式一：传值（复制一次）
    void byValue(Student s) {
        cout << s.name << endl;
    }

    // 方式二：传指针（不复制，但可能为空）
    void byPtr(const Student* s) {
        if (s) cout << s.name << endl;;
    }

    // 方式三：传引用（不复制，不能为空）
    void byRef(const Student& s) {
        cout << s.name << endl;
    }

    // 方式四：传可修改引用（允许修改）
    void byMutableRef(Student& s) {
        s.name = "Tom";   // 可以修改
    }

    int main() {
        Student s = {"001", "Jack", "Computer"};

        cout << "--- byValue ---" << endl;
        byValue(s);      // 会复制

        cout << "--- byPtr ---" << endl;
        byPtr(&s);       // 不复制

        cout << "--- byRef ---" << endl;
        byRef(s);        // 不复制

        return 0;
    }
    ```

!!! summary "参数传递方式选择指南"

    | 方式           | 语法                   |   复制成本   |   是否可修改   | 可否为 NULL | 适用场景                     |
    | :------------- | :--------------------- | :----------: | :------------: | :---------: | :--------------------------- |
    | **传值**       | `void f(T param)`      |  高（复制）  | 否（局部副本） |     N/A     | 简单数据，需要副本           |
    | **传指针**     | `void f(const T* p)`   | 低（不复制） |  否（const）   |     是      | 可能为空，C风格              |
    | **常引用**     | `void f(const T& ref)` | 低（不复制） |       否       |     否      | **传递复杂结构数据（推荐）** |
    | **可修改引用** | `void f(T& ref)`       | 低（不复制） |       是       |     否      | 需要修改复杂结构数据         |

!!! tip "最佳实践"

    - **优先使用常引用**（`const T&`）传递只读的复杂结构数据：既高效又安全。
    - **使用引用而非指针**：当参数不应为空时，引用比指针更清晰。
    - **小对象可以传值**：`int`、`double`、`char` 等基本类型，传值通常更简单高效。

## 小结

1.  **引用（Reference）** ：
    1. 是变量的别名，必须初始化，不能重新绑定。
    2. 作为函数参数比指针更简洁安全。
    3. 作为返回值支持链式调用和容器元素修改。
    4. **常引用** `const T&`：只读、不拷贝、可绑定临时对象。

2.  **引用 vs 指针**：
    1. 引用更安全（不能为空）、语法更简洁。
    2. 指针更灵活（可重新绑定、可为空）。
