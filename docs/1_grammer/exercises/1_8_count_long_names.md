# 编程：使用 const auto& 统计长姓名数量

---

## 任务描述

编写函数 `countLongNames`，统计姓名列表中长度不小于给定值 `minLength` 的姓名数量。

函数只读取姓名列表，不修改其中的任何字符串。由于 `std::string` 对象可能较大，遍历时应使用 `const auto&`，避免复制字符串并保护原数据。

## 相关知识

### 范围 for 与 const 引用

遍历只读容器时，推荐使用 `const auto&`：

```cpp
for (const auto& name : names) {
    // 可以读取 name，但不能修改 name
}
```

`auto` 自动推导元素类型，`&` 避免复制字符串，`const` 防止误修改容器元素。

### 字符串长度

`std::string` 的 `size()` 与 `length()` 都可以返回字符串长度：

```cpp
std::string name = "Alice";
name.length();  // 5
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `countLongNames` 的函数体。
3. 函数必须采用以下声明形式：

```cpp
std::size_t countLongNames(
    const std::vector<std::string>& names,
    std::size_t minLength
);
```

4. 必须使用范围 `for` 循环遍历姓名列表。
5. 循环变量必须采用 `const auto& name`。
6. 当 `name.length() >= minLength` 时，计数加一。
7. 不得修改 `names` 中的字符串。
8. 在 `test` 函数中使用 `assert` 验证统计结果。

## 待完成代码

```cpp
#include <cassert>
#include <cstddef>
#include <iostream>
#include <string>
#include <vector>

// TODO：使用范围 for + const auto& 统计符合条件的姓名数量
std::size_t countLongNames(
    const std::vector<std::string>& names,
    std::size_t minLength
) {
    // TODO
}

void test() {
    std::vector<std::string> names1 = {
        "Ada", "Alice", "Bob", "Charlie", "Lin"
    };

    assert(countLongNames(names1, 5) == 2);
    assert(countLongNames(names1, 3) == 5);
    assert(countLongNames(names1, 10) == 0);

    std::vector<std::string> names2 = {};
    assert(countLongNames(names2, 1) == 0);
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 输入 | `minLength` | 预期结果 |
|---|---|---:|---:|
| 统计长度不少于 5 的姓名 | `Ada, Alice, Bob, Charlie, Lin` | `5` | `2` |
| 统计长度不少于 3 的姓名 | 同上 | `3` | `5` |
| 阈值大于所有姓名长度 | 同上 | `10` | `0` |
| 空列表 | `{}` | `1` | `0` |

在本题中，若使用 `auto name` 也能得到正确结果，但每次循环都会复制一个字符串。使用 `const auto& name` 更高效，也能防止在循环中意外修改姓名。

---

开始你的任务吧，祝你成功！
