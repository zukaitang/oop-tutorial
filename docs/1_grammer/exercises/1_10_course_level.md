# 练习10：将课程难度枚举转换为文本

---

## 任务描述

定义作用域枚举 `CourseLevel`，表示课程的三种难度：入门、中级和高级。

编写函数 `toString`，将 `CourseLevel` 枚举值转换为对应英文文本。对于未定义的枚举值，函数应返回 `"Unknown"`。

## 相关知识

### enum class 的类型安全

作用域枚举不会隐式转换为整数：

``` cpp
enum class CourseLevel {
    Beginner,
    Intermediate,
    Advanced
};

CourseLevel level = CourseLevel::Beginner;
// int value = level;  // 错误：不能隐式转换为 int
```

如需从整数构造一个枚举值，必须使用显式转换：

``` cpp
auto unknownLevel = static_cast<CourseLevel>(99);
```

这类值不属于枚举中定义的三个值，函数应通过 `default` 分支安全处理。

### switch 与字符串字面量

函数可以使用 `switch` 根据枚举值返回不同的字符串字面量：

``` cpp
const char* toString(CourseLevel level) {
    switch (level) {
    case CourseLevel::Beginner:
        return "Beginner";
    default:
        return "Unknown";
    }
}
```

字符串字面量不能被修改，因此函数返回类型应为 `const char*`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 保留题目给出的 `enum class CourseLevel` 定义。
3. 完成函数 `toString` 的函数体。
4. 函数必须采用以下声明形式：

``` cpp
const char* toString(CourseLevel level);
```

1. 必须使用 `switch` 语句完成转换。
2. 分别返回 `"Beginner"`、`"Intermediate"`、`"Advanced"`。
3. 使用 `default` 分支处理未定义枚举值，并返回 `"Unknown"`。
4. 在 `test` 函数中使用 `std::strcmp` 与 `assert` 验证返回结果。

## 待完成代码

``` cpp
#include <cassert>
#include <cstring>
#include <iostream>

enum class CourseLevel {
    Beginner,
    Intermediate,
    Advanced
};

// TODO：将不同枚举值转换为对应字符串
const char* toString(CourseLevel level) {
    // TODO
}

void test() {
    assert(std::strcmp(toString(CourseLevel::Beginner), "Beginner") == 0);
    assert(std::strcmp(toString(CourseLevel::Intermediate), "Intermediate") == 0);
    assert(std::strcmp(toString(CourseLevel::Advanced), "Advanced") == 0);

    // 测试未定义的枚举值
    auto unknownLevel = static_cast<CourseLevel>(99);
    assert(std::strcmp(toString(unknownLevel), "Unknown") == 0);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项       | 函数调用                                 | 预期结果         |
| ------------ | ---------------------------------------- | ---------------- |
| 入门课程     | `toString(CourseLevel::Beginner)`        | `"Beginner"`     |
| 中级课程     | `toString(CourseLevel::Intermediate)`    | `"Intermediate"` |
| 高级课程     | `toString(CourseLevel::Advanced)`        | `"Advanced"`     |
| 未定义枚举值 | `toString(static_cast<CourseLevel>(99))` | `"Unknown"`      |

`std::strcmp` 比较两个 C 字符串的内容：返回值为 `0` 表示两个字符串内容相同。

如果遗漏 `default` 分支，传入未定义枚举值时，函数可能无法返回有效结果。

---

开始你的任务吧，祝你成功！
