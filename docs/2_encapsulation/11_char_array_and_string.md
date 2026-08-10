# 字符数组与string

字符串是程序设计中最常用的数据类型之一。C语言使用字符数组处理字符串，而C++提供了更安全、更便利的 `string` 类。理解两者的差异，是掌握C++标准库的重要一步。

## 从 C 语言出发：字符数组与字符串

### 什么是 C 风格字符串

在C语言中，字符串被定义为**以空字符 `'\0'` 结尾的字符数组**。这种字符串也被称为 **C 风格字符串（C-style String）**。

!!! example "C风格字符串的表示"

    ``` c linenums="1"
    // 方式一：字符数组初始化
    char str1[8] = {'p', 'r', 'o', 'g', 'r', 'a', 'm', '\0'};
    
    // 方式二：字符串常量初始化（自动添加 '\0'）
    char str2[8] = "program";
    
    // 方式三：省略数组大小（编译器自动计算）
    char str3[] = "program";   // 大小为 8（7个字符 + '\0'）
    
    // 方式四：指向字符串常量的指针
    const char* str4 = "program";
    ```

!!! info "C风格字符串的核心规则"

    - **以 `'\0'` 结尾**：空字符标记字符串的结束，是字符串处理的依据。
    - **字符数组存储**：每个字符连续存储，占据一个字节。
    - **字符串常量**：`"program"` 在表达式中表示字符数组的首地址。
    - **首地址可赋值**：可以赋给 `const char*` 指针。

### C 风格字符串的操作

C语言通过标准库 `<string.h>`（在C++中改为 `<cstring>`）提供了一系列字符串操作函数。

!!! example "C风格字符串的常用操作"

    ``` c linenums="1" hl_lines="9 15 21 26"
    #include <iostream>
    #include <cstring>   // C风格字符串操作函数
    using namespace std;

    int main() {
        // 1. 字符串复制
        char src[] = "Hello";
        char dest[20];
        strcpy(dest, src);          // 复制字符串
        cout << "strcpy: " << dest << endl;   // Hello

        // 2. 字符串连接
        char str1[20] = "Hello";
        char str2[] = " World";
        strcat(str1, str2);         // 连接字符串
        cout << "strcat: " << str1 << endl;   // Hello World

        // 3. 字符串比较
        char a[] = "Apple";
        char b[] = "Banana";
        int cmp = strcmp(a, b);     // 比较字符串（按字典序）
        cout << "strcmp: " << cmp << endl;   // 负数（Apple < Banana）

        // 4. 字符串长度
        char text[] = "program";
        size_t len = strlen(text);  // 获取长度（不含 '\0'）
        cout << "strlen: " << len << endl;   // 7

        return 0;
    }
    ```

### C 风格字符串的问题

!!! danger "C风格字符串的局限性"

    - **数组越界风险**：没有自动边界检查，字符串操作可能写入越界。
    - **手动管理内存**：需要预先分配足够的空间，无法动态扩展。
    - **函数不安全**：`strcpy`、`strcat` 等函数不检查目标缓冲区大小，容易导致缓冲区溢出。
    - **操作繁琐**：复制、连接、比较等操作都需要显式调用库函数，需要记住众多函数名（`strcpy`、`strcat`、`strcmp`、`strlen`...）。
    - **动态字符串困难**：当字符串长度不确定时，需要用 `new` 动态分配，最后用 `delete` 释放，容易出错。

    ``` c linenums="1"
    // 缓冲区溢出示例
    char buffer[5];
    strcpy(buffer, "Hello, World!");   // 危险！buffer 只有 5 个字节
    // 程序可能崩溃或产生未定义行为
    ```

    这就像每次搬家都要自己造车、修路、找仓库——能做，但不轻松。

## C++ 的解决方案：string 类

### 标准库的概念

在深入 `string` 之前，先了解一个重要的概念——**C++ 标准库**。

!!! info "C++ 标准库（C++ Standard Library）"

    C++ 标准库是C++语言的核心组成部分，提供了一系列**预定义的类型、函数和类模板**，用于处理常见的编程任务。

    标准库的核心理念是：
    - **复用性**：提供通用的数据结构和算法，避免重复造轮子。
    - **安全性**：封装了底层细节，提供更安全的接口。
    - **效率**：经过精心优化，性能可以媲美手写代码。
    - **可移植性**：所有C++编译器都支持标准库，代码可以在不同平台间移植。

    C++标准库的详细内容可以参见 <https://www.cppreference.com>

!!! success "C++ 标准库的主要组成部分"

    | 组成部分                         | 说明                 | 示例                              |
    | :------------------------------- | :------------------- | :-------------------------------- |
    | **容器（Containers）**           | 存储数据的数据结构   | `string`、`vector`、`list`、`map` |
    | **算法（Algorithms）**           | 通用的数据处理操作   | `sort`、`find`、`copy`            |
    | **迭代器（Iterators）**          | 容器与算法之间的桥梁 | `begin()`、`end()`                |
    | **函数对象（Function Objects）** | 行为类似函数的对象   | `greater`、`less`                 |
    | **智能指针（Smart Pointers）**   | 自动管理内存         | `unique_ptr`、`shared_ptr`        |

    `string` 类是**标准库容器**中的一员，专门用于处理字符串。

### string 类的设计目标

C++ 的 `string` 类将字符串封装为一种**对象**，让字符串像基本类型一样易于使用。

!!! note "string 类的设计目标"

    - **封装字符串数据**：将字符数组和长度信息封装在对象内部。
    - **自动内存管理**：根据字符串长度自动分配和释放内存。
    - **提供丰富接口**：支持赋值、连接、比较、查找等常用操作。
    - **操作符重载**：使用 `=`、`+`、`==`、`[]` 等运算符，使用更自然。
    - **类型安全**：避免 C 风格字符串的缓冲区溢出问题。

### string 的基本使用

!!! example "string 类的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>   // 必须包含 string 头文件
    using namespace std;

    int main() {
        // 1. 创建字符串
        string s1;                    // 空字符串
        string s2 = "Hello";          // 从字符串常量初始化
        string s3("World");           // 构造函数初始化
        string s4 = s2;               // 复制构造

        // 2. 字符串赋值
        s1 = "C++";
        s2 = s3;                      // 字符串之间可直接赋值

        // 3. 字符串连接
        string s5 = s2 + " " + s3;    // 使用 + 运算符
        s1 += " Programming";         // 使用 += 追加

        cout << "s1: " << s1 << endl;   // C++ Programming
        cout << "s5: " << s5 << endl;   // World World

        // 4. 字符串比较（使用关系运算符）
        string a = "Apple";
        string b = "Banana";
        if (a < b) {
            cout << a << " comes before " << b << endl;
        }

        // 5. 访问字符
        char ch = s1[0];              // 下标访问
        cout << "First char: " << ch << endl;   // C
        s1[0] = 'c';                  // 可以修改
        cout << "After modify: " << s1 << endl; // c++ Programming

        // 6. 获取长度
        cout << "Length: " << s1.length() << endl;   // 15
        cout << "Size: " << s1.size() << endl;       // 15（同 length）

        return 0;
    }
    ```

### string 的常用操作

!!! info "string 类常用成员函数"

    | 操作类别 | 函数                     | 说明                                          |
    | :------- | :----------------------- | :-------------------------------------------- |
    | **大小** | `size()` / `length()`    | 返回字符串长度                                |
    |          | `empty()`                | 判断字符串是否为空                            |
    | **访问** | `operator[](idx)`        | 下标访问（无越界检查）                        |
    |          | `at(idx)`                | 下标访问（有越界检查）                        |
    |          | `front()` / `back()`     | 访问首/尾字符                                 |
    | **查找** | `find(str)`              | 查找子串，返回位置                            |
    |          | `rfind(str)`             | 从右向左查找                                  |
    |          | `find_first_of(str)`     | 查找任意匹配字符                              |
    | **修改** | `append(str)`            | 追加字符串                                    |
    |          | `push_back(ch)`          | 追加一个字符                                  |
    |          | `insert(pos, str)`       | 在指定位置插入                                |
    |          | `erase(pos, len)`        | 删除指定部分                                  |
    |          | `replace(pos, len, str)` | 替换指定部分                                  |
    |          | `clear()`                | 清空字符串                                    |
    | **子串** | `substr(pos, len)`       | 提取子串                                      |
    | **转换** | `c_str()`                | 返回 C 风格字符串（`const char*`）            |
    |          | `data()`                 | 返回字符数组指针（C++11 起与 `c_str()` 相同） |

## string vs C风格字符串：全面对比

!!! summary "对比总结"

    | 对比项       | **C风格字符串（char[] / char*）** | **C++ string 类**                  |
    | :----------- | :-------------------------------- | :--------------------------------- |
    | **数据类型** | 字符数组（基本类型）              | 类对象（标准库类型）               |
    | **内存管理** | 手动（需要预分配空间）            | 自动（根据内容动态调整）           |
    | **赋值操作** | 需用 `strcpy()`                   | 直接用 `=` 运算符                  |
    | **连接操作** | 需用 `strcat()`                   | 直接用 `+` 或 `+=` 运算符          |
    | **比较操作** | 需用 `strcmp()`                   | 直接用 `==`、`<`、`>` 等运算符     |
    | **长度获取** | 需用 `strlen()`                   | 用 `length()` 或 `size()` 成员函数 |
    | **越界检查** | 无（缓冲区溢出风险）              | `at()` 方法带检查                  |
    | **动态扩展** | 需要 `new`/`delete` 手动管理      | 自动扩展                           |
    | **安全性**   | 较低（需谨慎处理边界）            | 较高（封装了边界管理）             |
    | **可读性**   | 较差（函数调用多）                | 较好（运算符直观）                 |
    | **头文件**   | `<cstring>` 或 `<string.h>`       | `<string>`                         |

### 对比示例

!!! example "C风格 vs C++风格"

    === "C风格字符串"

        ``` c linenums="1"
        #include <cstring>
        #include <iostream>
        using namespace std;

        int main() {
            // 需要预先分配足够空间
            char str1[20] = "Hello";
            char str2[] = " World";
            
            // 连接：使用 strcat（需确保目标空间足够）
            strcat(str1, str2);
            
            // 复制
            char str3[20];
            strcpy(str3, str1);
            
            // 比较
            if (strcmp(str1, str2) != 0) {
                cout << "Strings are different" << endl;
            }
            
            // 长度
            size_t len = strlen(str1);
            
            return 0;
        }
        ```

    === "C++ string风格"

        ``` cpp linenums="1"
        #include <string>
        #include <iostream>
        using namespace std;

        int main() {
            // 无需预先分配空间
            string str1 = "Hello";
            string str2 = " World";
            
            // 连接：直接用 +
            string str3 = str1 + str2;
            
            // 复制：直接赋值
            string str4 = str3;
            
            // 比较：直接用关系运算符
            if (str1 != str2) {
                cout << "Strings are different" << endl;
            }
            
            // 长度：调用成员函数
            size_t len = str1.length();
            
            return 0;
        }
        ```

## string 的进阶用法

### 输入整行字符串

使用 `getline` 可以读取包含空格的整行字符串。

!!! example "getline 的用法"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        string city, state;

        // 方式一：默认以换行符结束
        cout << "Enter city: ";
        getline(cin, city);
        cout << "City: " << city << endl;

        // 方式二：指定分隔符
        cout << "Enter city,state: ";
        getline(cin, city, ',');
        getline(cin, state);
        cout << "City: " << city << ", State: " << state << endl;

        return 0;
    }
    ```

### 字符串查找与替换

!!! example "查找与替换"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        string text = "Hello, World! Hello, C++!";

        // 查找子串
        size_t pos = text.find("World");
        if (pos != string::npos) {
            cout << "Found 'World' at position: " << pos << endl;   // 7
        }

        // 查找所有匹配
        string target = "Hello";
        pos = text.find(target);
        while (pos != string::npos) {
            cout << "Found '" << target << "' at: " << pos << endl;
            pos = text.find(target, pos + 1);
        }
        // 输出：Found 'Hello' at: 0
        // 输出：Found 'Hello' at: 14

        // 替换
        string str = "I like C++";
        str.replace(7, 3, "Java");   // 从位置7开始，替换3个字符
        cout << "After replace: " << str << endl;   // I like Java

        // 提取子串
        string sub = str.substr(2, 4);
        cout << "Substring: " << sub << endl;   // like

        return 0;
    }
    ```

### 数值转换（C++11）

!!! example "string 与数值的转换"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        // 数值 → string
        int num = 123;
        string s1 = to_string(num);
        double pi = 3.14159;
        string s2 = to_string(pi);

        cout << "s1: " << s1 << endl;   // "123"
        cout << "s2: " << s2 << endl;   // "3.141590"

        // string → 数值（C++11 stoi/stol/stoll/stof/stod）
        string numStr = "456";
        int n = stoi(numStr);
        double d = stod("3.14");

        cout << "n + 10 = " << n + 10 << endl;     // 466
        cout << "d * 2 = " << d * 2 << endl;       // 6.28

        return 0;
    }
    ```

## 从 C 到 C++：标准库的价值

!!! abstract "C++ 标准库的设计哲学"

    C++ 标准库的设计目标是让程序员能够**专注于解决问题，而不是纠结于底层细节**。

    以字符串为例：
    - **C 语言**：程序员需要管理字符数组、分配内存、调用函数，每一步都充满风险。
    - **C++ 标准库**：程序员使用 `string` 对象，像操作基本类型一样操作字符串，编译器管理内存，运行效率高。

!!! success "标准库的关键优势"

    | 优势         | 说明                                             |
    | :----------- | :----------------------------------------------- |
    | **安全性**   | 封装边界检查，减少缓冲区溢出、内存泄漏等常见错误 |
    | **生产力**   | 简洁的语法，减少代码量，提高开发效率             |
    | **可移植性** | 标准库在所有平台上有相同的行为                   |
    | **性能**     | 经过高度优化，性能接近手写代码                   |
    | **可维护性** | 代码更清晰、更易读、更易维护                     |

!!! tip "学习建议"

    在学习C++的过程中，应当：
    1. **理解底层原理**：了解C风格字符串的表示和操作，理解内存管理的挑战。
    2. **使用标准库**：在实际编程中优先使用 `string` 而不是字符数组。
    3. **持续探索**：标准库提供了丰富的工具，`string` 只是其中之一。`vector`、`map`、`algorithm` 等都是高效编程的好帮手。

## 小结

1. **C风格字符串**是字符数组，以 `'\0'` 结尾，操作繁琐且不安全。

2. **C++ string 类**将字符串封装为对象：
3. 自动管理内存，无需手动分配/释放。
4. 支持运算符重载，使用更自然。
5. 提供丰富的成员函数，功能强大。

6. **C 与 C++ 的对比**：
7. C 风格：`strcpy`、`strcat`、`strcmp`、`strlen`，手动管理内存。
8. C++ 风格：`=`、`+`、`==`、`length()`，自动管理内存。

9. **C++ 标准库**是C++编程的核心工具，提供了容器、算法、迭代器等丰富的组件，让程序员能够**安全、高效、优雅**地编写代码。