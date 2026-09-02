# 练习2：使用 auto 与 decltype 实现异类型数值求和

---

## 任务描述

编写函数 `add`，完成一个 `int` 整数与一个 `double` 浮点数的求和，并返回计算结果。

要求函数的返回类型不直接写为 `double`，而是通过 `decltype(a + b)` 根据表达式 `a + b` 自动推导；在主函数中，使用 `auto` 自动推导接收返回值的变量类型，并验证结果和值类型是否正确。

本任务不使用函数模板，仅处理 `int` 与 `double` 这一组固定参数类型。

## 相关知识

### `auto`：自动类型推导

`auto` 可以根据初始化表达式自动推导变量类型，变量必须在声明时初始化。

``` cpp
auto result = 3 + 2.5;  // result 被推导为 double
```

在本任务中，使用 `auto` 声明变量 `result`，由编译器自动推导 `add` 函数返回值的类型。

### `decltype`：表达式类型推导

`decltype(表达式)` 用于推导表达式的类型，但不会执行该表达式。

``` cpp
int a = 3;
double b = 2.5;

decltype(a + b) result = a + b;  // result 的类型为 double
```

由于 `int` 和 `double` 相加的结果为 `double`，因此：

``` cpp
decltype(a + b)
```

可作为函数 `add` 的返回类型。

### 尾置返回类型

当返回类型依赖函数参数时，可使用 C++11 的尾置返回类型语法：

``` cpp
auto 函数名(参数列表) -> 返回类型
```

例如：

``` cpp
auto add(int a, double b) -> decltype(a + b)
```

其中，`-> decltype(a + b)` 表示函数返回类型由表达式 `a + b` 的类型决定。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 不使用函数模板。
3. 完成函数 `add` 的函数体。
4. 函数必须采用以下声明形式：

``` cpp
auto add(int a, double b) -> decltype(a + b);
```

1. 函数应返回 `a` 与 `b` 的和。
2. 在 `main` 函数中使用 `auto` 接收函数返回值。
3. 使用 `static_assert` 和 `std::is_same` 验证结果变量的类型为 `double`。
4. 使用 `assert` 验证计算结果正确。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <type_traits>

// TODO：完成函数体
auto add(int a, double b) -> decltype(a + b) {
    // TODO
}

int test() {
    auto result1 = add(3, 2.5);
    auto result2 = add(-10, 0.75);

    // TODO：补充运行结果断言
    assert(/* result1 的结果判断 */);
    assert(/* result2 的结果判断 */);

    // 验证 auto 推导出的变量类型
    static_assert(
        std::is_same<decltype(result1), double>::value);
}

int main() {
    test();

    std::cout << "第2关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

程序至少应完成以下两组测试：

| 测试项     | 函数调用         | 预期结果 |
| ---------- | ---------------- | -------: |
| 正数相加   | `add(3, 2.5)`    |    `5.5` |
| 含负数相加 | `add(-10, 0.75)` |  `-9.25` |

同时，以下编译期检查必须通过：

``` cpp
static_assert(
    std::is_same<decltype(result1), double>::value
);
```

这说明：

- `decltype(a + b)` 正确推导出 `add` 的返回类型；
- `auto result1` 正确推导出变量类型；
- `int + double` 的计算结果类型为 `double`。

---

开始你的任务吧，祝你成功！
