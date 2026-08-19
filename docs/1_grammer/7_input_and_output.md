# 输入与输出

输入输出是程序与用户交互的基本方式。C语言使用 `printf`/`scanf` 函数族进行格式化I/O，
而C++提供了基于流的 `cin`/`cout` 对象。两者在类型安全、可扩展性和使用便利性上存在显著差异。


## 从C语言的I/O说起

### C语言的输入输出

C语言通过 `<stdio.h>` 中的 `printf` 和 `scanf` 函数进行格式化输入输出。

!!! example "C风格的I/O"

    ``` c linenums="1"
    #include <stdio.h>

    int main() {
        int age;
        double height;
        char name[50];

        // 输出
        printf("Enter your name: ");

        // 输入
        scanf("%s", name);

        printf("Enter your age: ");
        scanf("%d", &age);

        printf("Enter your height: ");
        scanf("%lf", &height);

        // 格式化输出
        printf("Name: %s, Age: %d, Height: %.2f\n", name, age, height);

        return 0;
    }
    ```

### C语言I/O的问题

!!! warning "C风格I/O的主要问题"

    | 问题 | 说明 | 示例 |
    | :--- | :--- | :--- |
    | **类型不安全** | 格式字符串与参数类型不匹配时，编译器不报错 | `printf("%d", 3.14)` 输出错误 |
    | **缓冲区溢出** | `scanf` 不检查数组边界 | `scanf("%s", name)` 可能越界 |
    | **可扩展性差** | 自定义类型无法直接输出 | 需要单独编写打印函数 |
    | **格式字符串复杂** | 需要记忆大量格式说明符 | `%d`、`%f`、`%s`、`%p` 等 |
    | **参数数量不匹配** | 格式字符串与参数个数不一致时，行为未定义 | 多参数或少参数 |

!!! example "类型不安全的后果"

    ``` c
    // 格式字符串与参数类型不匹配
    int n = 10;
    double d = 3.14;
    printf("%d\n", d);   // 输出错误（未定义行为）
    printf("%f\n", n);   // 输出错误（未定义行为）

    // 缓冲区溢出风险
    char buffer[10];
    scanf("%s", buffer); // 输入超过 10 个字符时溢出
    ```


## C++的流式I/O

### 基本概念

C++使用**流（Stream）** 的概念进行输入输出。流是字节序列的抽象，数据从源流向目标。

!!! info "C++标准I/O流"

    | 对象 | 用途 | 对应C |
    | :--- | :--- | :--- |
    | `cin` | 标准输入（键盘） | `scanf` / `stdin` |
    | `cout` | 标准输出（屏幕） | `printf` / `stdout` |
    | `cerr` | 标准错误（无缓冲） | `fprintf(stderr, ...)` |
    | `clog` | 标准错误（有缓冲） | `fprintf(stderr, ...)` |

!!! example "基本的cin/cout使用"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int age;
        double height;
        string name;

        // 输出
        cout << "Enter your name: ";

        // 输入
        cin >> name;

        cout << "Enter your age: ";
        cin >> age;

        cout << "Enter your height: ";
        cin >> height;

        // 链式输出
        cout << "Name: " << name
             << ", Age: " << age
             << ", Height: " << height << endl;

        return 0;
    }
    ```

### 类型安全

C++的 `cin`/`cout` 是**类型安全的**，编译器会根据变量类型自动选择合适的输入输出方式。

!!! success "类型安全的优势"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int n = 10;
        double d = 3.14;
        const char* s = "hello";

        // 编译器自动识别类型，无需格式字符串
        cout << "int: " << n << endl;      // 10
        cout << "double: " << d << endl;   // 3.14
        cout << "string: " << s << endl;   // hello

        // 类型不匹配时，编译器会报错或进行类型转换
        // cout << n; 和 cout << d; 调用不同的重载版本

        return 0;
    }
    ```


## cout 的详细使用

### 基本输出操作

`cout` 使用 `<<` 操作符将数据输出到标准输出。

!!! example "输出不同类型"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        int n = 42;
        double pi = 3.1415926;
        string msg = "Hello";

        // 基本输出
        cout << "n = " << n << endl;
        cout << "pi = " << pi << endl;
        cout << "msg = " << msg << endl;

        // 链式输出
        cout << "n = " << n << ", pi = " << pi << ", msg = " << msg << endl;

        return 0;
    }
    ```

### 格式化输出

使用 `<iomanip>` 头文件中的操纵符可以控制输出格式。

!!! example "常用格式化操纵符"

    ``` cpp linenums="1"
    #include <iostream>
    #include <iomanip>
    using namespace std;

    int main() {
        double pi = 3.14159265358979;

        cout << "=== 控制小数位数 ===" << endl;
        cout << "默认: " << pi << endl;
        cout << fixed << "fixed: " << pi << endl;
        cout << setprecision(2) << "setprecision(2): " << pi << endl;
        cout << setprecision(4) << "setprecision(4): " << pi << endl;

        cout << "\n=== 控制宽度和对齐 ===" << endl;
        cout << setw(10) << "ID" << setw(15) << "Name" << endl;
        cout << setw(10) << 1 << setw(15) << "Alice" << endl;
        cout << setw(10) << 2 << setw(15) << "Bob" << endl;

        cout << "\n=== 控制填充字符 ===" << endl;
        cout << setfill('*') << setw(10) << 42 << endl;
        cout << setfill(' ') << setw(10) << 42 << endl;

        cout << "\n=== 控制进制 ===" << endl;
        int n = 255;
        cout << "dec: " << dec << n << endl;   // 255
        cout << "hex: " << hex << n << endl;   // ff
        cout << "oct: " << oct << n << endl;   // 377
        cout << dec;  // 恢复十进制

        cout << "\n=== 控制布尔输出 ===" << endl;
        bool flag = true;
        cout << "boolalpha: " << boolalpha << flag << endl;   // true
        cout << "noboolalpha: " << noboolalpha << flag << endl; // 1

        return 0;
    }
    ```

    运行结果：
    ```
    === 控制小数位数 ===
    默认: 3.14159
    fixed: 3.141593
    setprecision(2): 3.14
    setprecision(4): 3.1416

    === 控制宽度和对齐 ===
            ID           Name
             1          Alice
             2            Bob

    === 控制填充字符 ===
    *******42
          42

    === 控制进制 ===
    dec: 255
    hex: ff
    oct: 377

    === 控制布尔输出 ===
    boolalpha: true
    noboolalpha: 1
    ```


## cin 的详细使用

### 基本输入操作

`cin` 使用 `>>` 操作符从标准输入读取数据，自动根据变量类型进行解析。

!!! example "读取不同类型的数据"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        int i;
        double d;
        string s;
        char c;

        // 读取不同类型
        cout << "Enter an int: ";
        cin >> i;

        cout << "Enter a double: ";
        cin >> d;

        cout << "Enter a string: ";
        cin >> s;

        cout << "Enter a char: ";
        cin >> c;

        cout << "You entered: " << i << ", " << d
             << ", " << s << ", " << c << endl;

        return 0;
    }
    ```

### 链式输入

`cin` 支持链式操作，可以一次读取多个值。

!!! example "链式输入"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int a, b, c;

        // 链式输入：输入用空格或换行分隔
        cout << "Enter three integers: ";
        cin >> a >> b >> c;

        cout << "Sum: " << a + b + c << endl;

        return 0;
    }
    ```

### 输入整行字符串

`cin >>` 以空白符（空格、制表符、换行）作为分隔符。要读取包含空格的整行，需使用 `getline()`。

!!! example "getline 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        string fullName;
        string city;

        cout << "Enter your full name: ";
        getline(cin, fullName);   // 读取整行

        cout << "Enter your city: ";
        getline(cin, city);

        cout << "Name: " << fullName << endl;
        cout << "City: " << city << endl;

        return 0;
    }
    ```

### 指定分隔符

`getline` 可以指定第三个参数作为分隔符。

!!! example "自定义分隔符"

    ``` cpp linenums="1"
    #include <iostream>
    #include <string>
    using namespace std;

    int main() {
        string part1, part2, part3;

        cout << "Enter data (format: part1,part2,part3): ";
        getline(cin, part1, ',');
        getline(cin, part2, ',');
        getline(cin, part3);

        cout << "Part1: " << part1 << endl;
        cout << "Part2: " << part2 << endl;
        cout << "Part3: " << part3 << endl;

        return 0;
    }
    ```

    !!! warning "cin 与 getline 的混合使用"

        使用 `cin >>` 后调用 `getline` 时，需要处理残留的换行符：

        ``` cpp
        int age;
        string name;

        cout << "Enter age: ";
        cin >> age;
        cin.ignore();   // 忽略换行符

        cout << "Enter name: ";
        getline(cin, name);
        ```

### 输入状态检查

`cin` 提供了状态检查功能，可以检测输入是否成功。

!!! example "检查输入状态"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    int main() {
        int value;

        cout << "Enter an integer: ";
        cin >> value;

        if (cin.fail()) {
            cout << "Invalid input!" << endl;
            cin.clear();           // 清除错误状态
            cin.ignore(100, '\n'); // 忽略错误输入
        } else {
            cout << "You entered: " << value << endl;
        }

        return 0;
    }
    ```



## C vs C++ I/O 对比

!!! summary "全面对比"

    | 对比项 | C语言（printf/scanf） | C++（cin/cout） |
    | :--- | :--- | :--- |
    | **类型安全** | ✗ 格式字符串与参数需匹配 | ✓ 编译器自动识别类型 |
    | **缓冲区溢出** | ✗ 容易发生 | ✓ 安全（使用 string） |
    | **可扩展性** | ✗ 自定义类型需单独处理 | ✓ 可重载 `<<` 和 `>>` |
    | **语法简洁性** | 较复杂（格式字符串） | 较简洁（链式操作） |
    | **格式化控制** | 格式字符串（`%d`、`%f` 等） | 操纵符（`setw`、`setprecision` 等） |
    | **性能** | 通常较快 | 略慢（可优化） |
    | **C++推荐** | 不推荐（C风格） | ✓ 推荐 |

!!! info "性能说明"

    在默认情况下，C++的 `cin`/`cout` 为了与C的 `stdio` 同步，性能比 `printf`/`scanf` 略慢。如果不需要同步，可以取消：

    ``` cpp
    ios::sync_with_stdio(false);   // 取消与C标准I/O的同步
    cin.tie(nullptr);               // 取消 cin 与 cout 的绑定
    ```

    这样可以显著提升 C++ I/O 的性能。


## 字符串流

C++提供了**字符串流（stringstream）**，可以在内存中对字符串进行类似流的操作，非常实用。

!!! example "stringstream 的使用"

    ``` cpp linenums="1"
    #include <iostream>
    #include <sstream>
    #include <string>
    using namespace std;

    int main() {
        // 1. 字符串 → 数据（解析）
        string data = "42 3.14 Hello";
        stringstream ss(data);

        int n;
        double d;
        string s;

        ss >> n >> d >> s;
        cout << "Parsed: n=" << n << ", d=" << d << ", s=" << s << endl;

        // 2. 数据 → 字符串（格式化）
        stringstream out;
        out << "Value: " << n << ", PI: " << d;
        string result = out.str();
        cout << "Formatted: " << result << endl;

        // 3. 数值与字符串互转
        // 字符串 → 数值
        string numStr = "123";
        int num;
        stringstream(numStr) >> num;
        cout << "num + 10 = " << num + 10 << endl;   // 133

        // 数值 → 字符串
        int value = 456;
        stringstream ss2;
        ss2 << value;
        string str = ss2.str();
        cout << "str = \"" << str << "\"" << endl;

        return 0;
    }
    ```


## 小结

1. **C++流式I/O的特点**：
   - 类型安全：编译器自动识别类型。
   - 语法简洁：链式操作，无需格式字符串。
   - 可扩展：通过运算符重载支持自定义类型。
2. **cin/cout 的使用**：
   - `cin >> var` 读取数据，`cout << var` 输出数据。
   - `getline(cin, str)` 读取整行（包含空格）。
   - 链式操作：`cin >> a >> b >> c`。
3. **格式化输出**：
   - `<iomanip>` 头文件中的操纵符：
   - `setw(n)`：设置宽度。
   - `setprecision(n)`：设置精度。
   - `fixed`、`scientific`：设置浮点格式。
   - `boolalpha`：输出 `true`/`false`。
   - `dec`、`hex`、`oct`：设置进制。
4. **输入状态检查**：
   - `cin.fail()`：检查输入是否失败。
   - `cin.clear()`：清除错误状态。
   - `cin.ignore()`：忽略缓冲区中的字符。
5. **字符串流**：`stringstream` 实现了字符串与数据之间的转换。