# 编程：使用 constexpr 实现编译期合法性检查

---

## 任务描述

编写函数 `isValidScore`，判断一个整数成绩是否有效。

成绩的合法范围为 **0 到 100**，包含边界值 `0` 和 `100`：成绩在该范围内时返回 `true`，否则返回 `false`。

本任务要求将该函数声明为 `constexpr`，使编译器能够在编译期调用它，并通过 `static_assert` 对结果进行检查。

## 相关知识

### `constexpr`：编译期常量表达式

`constexpr` 是 C++11 引入的关键字，用于表示可以在**编译期**计算的常量表达式，比 `const` 更严格。

| 对比项       | `const`                       | `constexpr`                |
| :----------- | :---------------------------- | :------------------------- |
| **求值时机** | 可以是运行时，也可以是编译期  | **必须是编译期**           |
| **初始化**   | 可以在运行时初始化            | 必须在编译期初始化         |
| **用途**     | 表示"只读"                    | 表示"编译期常量"           |
| **函数支持** | 不能修饰函数（函数没有const） | 可以修饰函数（编译期求值） |

``` cpp
constexpr int square(int x) {
    return x * x;
}

constexpr int result = square(5);  // 编译期计算
```

`constexpr` 函数既可以在编译期调用，也可以在运行期调用。

``` cpp
constexpr int cube(int x) {
    return x * x * x;
}

constexpr bool compileTimeResult = cube(85);  // 编译期调用
bool runtimeResult = cube(90);                // 运行期调用
```

使用建议

- 需要**编译期常量**时优先使用 `constexpr`（如数组长度、模板参数）。
- 表示**运行时只读**时使用 `const`。
- `constexpr` 函数可以在运行时调用，也可以在编译期调用。

### `static_assert`：编译期断言

`static_assert` 用于在编译期验证一个条件。条件为 `false` 时，程序无法通过编译。

``` cpp
static_assert(1 + 1 == 2);  // 条件成立，可以编译
```

本任务中，`static_assert` 用于验证 `isValidScore` 能够在编译期正确判断边界成绩和非法成绩。

如果函数未使用 `constexpr` 声明，则下列代码无法通过编译：

``` cpp
bool isValidScore(int score) {
    return score >= 0 && score <= 100;
}

static_assert(isValidScore(85));  // 错误：条件不是编译期常量表达式
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `isValidScore` 的函数体。
3. 函数必须采用以下声明形式：

``` cpp
constexpr bool isValidScore(int score);
```

1. 当 `score` 在 `[0, 100]` 范围内时返回 `true`。
2. 当 `score` 小于 `0` 或大于 `100` 时返回 `false`。
3. 在 `test` 函数中使用 `static_assert` 进行编译期测试。
4. 在 `test` 函数中使用 `assert` 进行运行期测试。
5. 在 `main` 函数中调用 `test` 函数，并保留测试通过信息与返回语句。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>

// TODO：完成函数体，函数必须能够在编译期求值
constexpr bool isValidScore(int score) {
    // TODO
}

void test() {
    // 编译期测试
    static_assert(isValidScore(0));
    static_assert(isValidScore(100));
    static_assert(!isValidScore(-1));
    static_assert(!isValidScore(101));

    // 运行期测试
    assert(isValidScore(85));
    assert(!isValidScore(120));
}

int main() {
    test();

    std::cout << "第1关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`test` 函数应覆盖以下成绩边界与典型情况：

| 测试项       | 函数调用            | 预期结果 | 测试阶段 |
| ------------ | ------------------- | :------: | -------- |
| 最小合法成绩 | `isValidScore(0)`   |  `true`  | 编译期   |
| 最大合法成绩 | `isValidScore(100)` |  `true`  | 编译期   |
| 小于最小值   | `isValidScore(-1)`  | `false`  | 编译期   |
| 大于最大值   | `isValidScore(101)` | `false`  | 编译期   |
| 正常合法成绩 | `isValidScore(85)`  |  `true`  | 运行期   |
| 正常非法成绩 | `isValidScore(120)` | `false`  | 运行期   |

若将判断条件错误写为：

``` cpp
return score > 0 && score < 100;
```

则 `static_assert(isValidScore(0))` 和 `static_assert(isValidScore(100))` 会导致编译失败。这说明边界值 `0` 与 `100` 也必须被正确处理。

---

开始你的任务吧，祝你成功！
