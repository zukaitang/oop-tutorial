# 范围 for 循环（C++11）

## 传统 for 循环 vs 范围 for 循环

C语言中遍历数组需要使用下标或指针，语法相对繁琐。C++11引入了**范围 for 循环（Range-based for loop）**，可以更简洁地遍历容器或数组的所有元素。

!!! info "范围 for 循环的语法"

    ``` cpp
    for (声明 : 表达式) {
        // 循环体
    }
    ```

    - **声明**：定义一个变量，用于依次接收集合中的每个元素。
    - **表达式**：一个集合或数组，提供要遍历的元素。
    - 循环自动遍历表达式中的所有元素，直到全部处理完毕。

!!! example "范围 for 循环的基本用法"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        // 1. 遍历数组
        int arr[] = {1, 2, 3, 4, 5};
        
        cout << "Array: ";
        for (int x : arr) {
            cout << x << " ";
        }
        cout << endl;   // 1 2 3 4 5

        // 2. 遍历 vector
        vector<string> names = {"Alice", "Bob", "Charlie"};
        
        cout << "Names: ";
        for (const string& name : names) {   // 使用 const 引用避免拷贝
            cout << name << " ";
        }
        cout << endl;   // Alice Bob Charlie

        // 3. 使用 auto 自动推导类型
        double scores[] = {85.5, 90.0, 78.5};
        
        cout << "Scores: ";
        for (auto score : scores) {
            cout << score << " ";
        }
        cout << endl;   // 85.5 90 78.5

        return 0;
    }
    ```

## 范围 for 循环与 auto 的配合

范围 `for` 循环与 `auto` 配合使用，可以进一步简化代码，尤其在处理复杂类型时：

!!! example "auto 简化类型声明"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<vector<int>> matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        // 传统写法：类型很长
        for (vector<vector<int>>::iterator it = matrix.begin(); 
             it != matrix.end(); 
             ++it) {
            for (vector<int>::iterator jt = it->begin(); 
                 jt != it->end(); ++jt) {
                cout << *jt << " ";
            }
            cout << endl;
        }

        // 范围 for + auto：简洁清晰
        cout << "Matrix: " << endl;
        for (const auto& row : matrix) {
            for (auto elem : row) {
                cout << elem << " ";
            }
            cout << endl;
        }

        return 0;
    }
    ```

## 值传递与引用传递

范围 `for` 循环中，元素的传递方式会影响性能和能否修改原数据：

!!! info "三种传递方式对比"

    | 写法                           | 含义           | 能否修改原数据 | 性能       |
    | :----------------------------- | :------------- | :------------: | :--------- |
    | `for (T x : container)`        | 值传递（拷贝） |      不能      | 有拷贝开销 |
    | `for (T& x : container)`       | 引用传递       |     **能**     | 无拷贝开销 |
    | `for (const T& x : container)` | 常引用传递     |      不能      | 无拷贝开销 |

!!! example "三种方式的对比"

    ``` cpp linenums="1" hl_lines="10 24 38"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<int> v = {1, 2, 3, 4, 5};

        // 方式一：值传递（拷贝）
        cout << "Value copy: ";
        for (int x : v) {
            x *= 2;              // 修改的是副本，不影响原数据
            cout << x << " ";
        }
        cout << endl;   // 2 4 6 8 10

        cout << "Original: ";
        for (int x : v) {
            cout << x << " ";    // 1 2 3 4 5（未被修改）
        }
        cout << endl;

        // 方式二：引用传递（可修改）
        cout << "Reference (modify): ";
        for (int& x : v) {
            x *= 2;              // 修改的是原数据
            cout << x << " ";
        }
        cout << endl;   // 2 4 6 8 10

        cout << "After modification: ";
        for (int x : v) {
            cout << x << " ";    // 2 4 6 8 10（已被修改）
        }
        cout << endl;

        // 方式三：常引用传递（只读）
        cout << "Const reference: ";
        for (const int& x : v) {
            // x *= 2;           // 错误！不能修改 const 引用
            cout << x << " ";
        }
        cout << endl;

        return 0;
    }
    ```

!!! tip "使用建议"

    - **只读遍历**：使用 `const T&`（避免拷贝，保护原数据）。
    - **需要修改元素**：使用 `T&`（引用传递）。
    - **基本类型**（`int`、`double` 等）：可以使用值传递（拷贝开销小）。

## 范围 for 的底层原理与适用条件

范围 `for` 循环本质上是对迭代器的封装，编译器会将其展开为传统的迭代器循环。

!!! info "范围 for 的展开"

    ``` cpp
    // 范围 for 写法
    for (auto x : container) {
        // 循环体
    }

    // 编译器展开后的等价代码（简化）
    for (auto it = container.begin(); it != container.end(); ++it) {
        auto x = *it;
        // 循环体
    }
    ```

因此，范围 `for` 循环适用于所有支持 `begin()` 和 `end()` 的类型：

- 数组（内置数组）
- STL 容器（`vector`、`list`、`map` 等）
- 自定义类型（只要实现了 `begin()` 和 `end()` 成员函数）

## 范围 for 循环的限制

!!! warning "不应在范围 for 循环中修改容器大小"

    ``` cpp
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<int> v = {1, 2, 3, 4, 5};

        // 危险！在循环中修改容器大小会导致迭代器失效
        for (auto x : v) {
            if (x == 3) {
                v.push_back(6);   // 危险！迭代器失效，未定义行为
            }
        }

        return 0;
    }
    ```

!!! note "范围 for 与普通 for 的选择"

    | 场景                         | 推荐方式                |
    | :--------------------------- | :---------------------- |
    | 遍历整个容器，不需要索引     | 范围 `for`              |
    | 需要反向遍历                 | 普通 `for` 或反向迭代器 |
    | 需要跳跃遍历（隔一个取一个） | 普通 `for`              |
    | 需要索引值                   | 普通 `for`              |
    | 需要在循环中修改容器大小     | 普通 `for`（谨慎操作）  |

# 小结

1. 语法：`for (声明 : 表达式) { 循环体 }`
2. 与 `auto` 配合使用效果更佳。
3. 注意值传递、引用传递和常引用传递的区别。
4. 不应在循环中修改容器大小。