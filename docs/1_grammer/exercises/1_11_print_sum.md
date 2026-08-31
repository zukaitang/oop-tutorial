# 编程：使用链式输入计算三个整数之和

---

## 任务描述

编写函数 `printSum`，从输入流中读取三个整数，计算它们的总和，并将结果输出到输出流。

本题使用 `std::istream` 与 `std::ostream` 作为函数参数，使输入与输出可以在 `test` 函数中自动验证。实际使用时，可以传入 `std::cin` 与 `std::cout`。

## 相关知识

### 链式输入

输入流支持使用 `>>` 连续读取多个数据：

```cpp
int a, b, c;
std::cin >> a >> b >> c;
```

上述代码会按顺序读取三个整数，并分别存入 `a`、`b` 与 `c`。

### 流式输出

输出流使用 `<<` 按顺序输出文本和变量：

```cpp
std::cout << "Sum: " << a + b + c << '\n';
```

### 字符串流测试

`std::istringstream` 可以模拟输入流，`std::ostringstream` 可以收集输出内容：

```cpp
std::istringstream input("10 20 30");
std::ostringstream output;
```

因此，测试代码可以比较 `output.str()` 与期望字符串，无需人工输入。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `printSum` 的函数体。
3. 函数必须采用以下声明形式：

```cpp
void printSum(std::istream& in, std::ostream& out);
```

4. 必须使用链式输入 `in >> a >> b >> c` 读取三个整数。
5. 必须使用 `out <<` 输出计算结果。
6. 输出格式必须严格为 `Sum: 数值`，并以换行结束。
7. 在 `test` 函数中使用 `assert` 验证输出内容。

## 待完成代码

```cpp
#include <cassert>
#include <iostream>
#include <sstream>

// TODO：使用链式输入读取 a、b、c，并输出总和
void printSum(std::istream& in, std::ostream& out) {
    // TODO
}

void test() {
    std::istringstream input1("10 20 30");
    std::ostringstream output1;

    printSum(input1, output1);
    assert(output1.str() == "Sum: 60\n");

    std::istringstream input2("-5 0 12");
    std::ostringstream output2;

    printSum(input2, output2);
    assert(output2.str() == "Sum: 7\n");
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 输入内容 | 预期输出 |
|---|---|---|
| 三个正整数 | `10 20 30` | `Sum: 60` |
| 负数、零和正数 | `-5 0 12` | `Sum: 7` |

注意：输出中的空格、冒号和换行均会被测试。若输出为 `Sum=60` 或缺少换行，断言将失败。

---

开始你的任务吧，祝你成功！
