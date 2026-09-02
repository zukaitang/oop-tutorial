# 练习12：使用 getline 输出姓名问候语

---

## 任务描述

编写函数 `printGreeting`，从输入流中读取一整行姓名，并输出对应的问候语。

姓名可能包含空格，例如 `Ada Lovelace`。函数必须完整读取姓名，输出格式为 `Hello, 姓名!`。

## 相关知识

### `>>` 与 `getline` 的区别

使用 `>>` 读取字符串时，遇到空白符会停止：

``` cpp
std::string name;
std::cin >> name;  // 输入 Ada Lovelace 后，只读取 Ada
```

使用 `std::getline` 可以读取整行内容，包括其中的空格：

``` cpp
std::string name;
std::getline(std::cin, name);  // 读取 Ada Lovelace
```

### 流参数与测试

函数使用 `std::istream&` 和 `std::ostream&` 参数，可在测试时传入字符串流：

``` cpp
std::istringstream input("Ada Lovelace\n");
std::ostringstream output;
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `printGreeting` 的函数体。
3. 函数必须采用以下声明形式：

``` cpp
void printGreeting(std::istream& in, std::ostream& out);
```

1. 必须使用 `std::getline(in, name)` 读取姓名。
2. 不得使用 `in >> name` 读取姓名。
3. 输出格式必须严格为 `Hello, 姓名!`，并以换行结束。
4. 在 `test` 函数中使用 `assert` 验证输出内容。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

// TODO：使用 getline 读取完整姓名，并输出问候语
void printGreeting(std::istream& in, std::ostream& out) {
    // TODO
}

void test() {
    std::istringstream input1("Ada Lovelace\n");
    std::ostringstream output1;

    printGreeting(input1, output1);
    assert(output1.str() == "Hello, Ada Lovelace!\n");

    std::istringstream input2("Zhang San\n");
    std::ostringstream output2;

    printGreeting(input2, output2);
    assert(output2.str() == "Hello, Zhang San!\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项           | 输入内容       | 预期输出               |
| ---------------- | -------------- | ---------------------- |
| 英文全名         | `Ada Lovelace` | `Hello, Ada Lovelace!` |
| 含空格的拼音姓名 | `Zhang San`    | `Hello, Zhang San!`    |

若使用 `in >> name`，第一组测试将只输出 `Hello, Ada!`，无法通过断言。

---

开始你的任务吧，祝你成功！
