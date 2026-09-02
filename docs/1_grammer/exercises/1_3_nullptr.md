# 练习3：使用 `nullptr` 实现安全的数组求和

---

## 任务描述

编写函数 `sumArray`，计算整型数组中所有元素的和。

函数接收数组首地址 `data` 与元素个数 `size`。调用者可能传入空指针，因此函数必须先判断 `data` 是否为 `nullptr`，避免解引用空指针导致程序异常。

当数组指针为空或数组长度为 `0` 时，函数应返回 `0`。

## 相关知识

### `nullptr`：类型安全的空指针

C++11 使用 `nullptr` 表示空指针。与传统的 `NULL` 或整数 `0` 相比，`nullptr` 具有明确的空指针类型，能够避免类型歧义。

``` cpp
int* p = nullptr;
```

使用指针前，应先判断其是否为空：

``` cpp
if (p == nullptr) {
    // 指针为空，不能通过 *p 访问数据
}
```

### `const int*` 参数

函数参数使用 `const int*`：

``` cpp
const int* data
```

表示函数可以读取数组元素，但不能修改原数组内容。例如，以下代码不合法：

``` cpp
data[0] = 100;  // 错误：不能通过 const int* 修改数据
```

这体现了 C++ 的类型安全与只读约束。

### 数组与指针访问

数组首地址可通过指针传递给函数，使用下标访问数组元素：

``` cpp
data[i]
```

在访问前，必须确保：

1. `data` 不为 `nullptr`；
2. 下标 `i` 小于数组长度 `size`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `sumArray`。
3. 函数参数必须使用以下形式：

``` cpp
int sumArray(const int* data, std::size_t size);
```

1. 当 `data == nullptr` 时，返回 `0`。
2. 当 `size == 0` 时，返回 `0`。
3. 当数组有效时，遍历数组并返回所有元素之和。
4. 编写 `test` 函数，对 `sumArray` 进行断言测试。
5. 在 `main` 函数中调用 `test` 函数，并保留测试通过信息与返回语句。

## 待完成代码

``` cpp
#include <cassert>
#include <cstddef>
#include <iostream>

// TODO：完成数组求和函数
int sumArray(const int* data, std::size_t size) {
    // TODO
}

void test() {
    int scores[] = {78, 85, 92, 88};

    // 完成下列测试
    assert(sumArray(scores, 4) == 343);
    assert(sumArray(scores, 0) == 0);
    assert(sumArray(nullptr, 0) == 0);
    assert(sumArray(nullptr, 10) == 0);
}

int main() {
    test();

    std::cout << "第3关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`test` 函数应至少覆盖以下四类场景：

| 测试项             | 调用方式                | 预期结果 |
| ------------------ | ----------------------- | -------: |
| 正常数组求和       | `sumArray(scores, 4)`   |    `343` |
| 有效数组但长度为 0 | `sumArray(scores, 0)`   |      `0` |
| 空指针且长度为 0   | `sumArray(nullptr, 0)`  |      `0` |
| 空指针但长度非 0   | `sumArray(nullptr, 10)` |      `0` |

其中，最后一项用于验证函数是否真正进行了空指针检查。若未判断 `data == nullptr` 而直接访问 `data[i]`，程序可能发生运行时错误。

---

开始你的任务吧，祝你成功！
