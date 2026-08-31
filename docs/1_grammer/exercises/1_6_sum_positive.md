# 编程：使用范围 for 计算正数之和

---

## 任务描述

编写函数 `sumPositive`，计算整型容器中所有正数元素的和。

函数接收一个整数 `vector`，遍历其中的每个元素：大于 `0` 的元素参与求和，负数与 `0` 不参与计算。若容器为空或不包含正数，函数应返回 `0`。

## 相关知识

### 范围 for 循环

C++11 的范围 `for` 循环可用于依次访问数组或容器中的全部元素：

```cpp
for (声明 : 表达式) {
    // 循环体
}
```

遍历 `vector<int>` 时，可以使用值传递：

```cpp
for (int value : values) {
    // value 是当前元素的副本
}
```

本题只读取 `int` 元素，且 `int` 的复制开销很小，因此可以使用值传递。

### 常引用参数

函数参数使用 `const std::vector<int>&`，既避免复制整个容器，又保证函数不会修改容器内容。

```cpp
int sumPositive(const std::vector<int>& values);
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `sumPositive` 的函数体。
3. 函数必须采用以下声明形式：

```cpp
int sumPositive(const std::vector<int>& values);
```

4. 必须使用范围 `for` 循环遍历容器。
5. 循环变量使用值传递，例如 `int value`。
6. 只有大于 `0` 的元素参与求和。
7. 不得使用下标访问或迭代器遍历容器。
8. 在 `test` 函数中使用 `assert` 验证结果。

## 待完成代码

```cpp
#include <cassert>
#include <iostream>
#include <vector>

// TODO：使用范围 for 循环完成求和
int sumPositive(const std::vector<int>& values) {
    // TODO
}

void test() {
    std::vector<int> values1 = {10, -5, 20, 0, -3};
    std::vector<int> values2 = {-1, -2, -3};
    std::vector<int> values3 = {};

    assert(sumPositive(values1) == 30);
    assert(sumPositive(values2) == 0);
    assert(sumPositive(values3) == 0);
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 输入 | 预期结果 |
|---|---|---:|
| 包含正数、负数和零 | `{10, -5, 20, 0, -3}` | `30` |
| 不包含正数 | `{-1, -2, -3}` | `0` |
| 空容器 | `{}` | `0` |

注意：条件应写为 `value > 0`。如果误写为 `value >= 0`，则元素 `0` 会被错误地纳入求和。

---

开始你的任务吧，祝你成功！
