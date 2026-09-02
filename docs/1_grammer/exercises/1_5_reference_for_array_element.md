# 练习5：通过引用访问和修改 C 字符串字符

---

## 任务描述

编写函数 `getCharacter`，返回 C 风格字符串中指定下标字符的引用。

调用者既可以读取函数返回的字符，也可以通过该返回值直接修改字符串中的指定字符。例如，`getCharacter(text, 0) = 'H';` 应将字符串 `text` 的第一个字符修改为 `'H'`。

函数还需要使用断言检查指针是否为空以及下标是否越界，避免访问 C 字符串结尾的 `\0` 或不存在的位置。

## 相关知识

### 引用返回值

函数返回 `char&` 时，返回的是字符串中某个字符的别名，而不是该字符的副本。

``` cpp
char& getCharacter(char str[], std::size_t index) {
    return str[index];
}
```

因此，函数调用可以出现在赋值运算符左侧：

``` cpp
char text[] = "hello";
getCharacter(text, 0) = 'H';
// text 变为 "Hello"
```

### C 风格字符串与有效下标

C 风格字符串以空字符 `\0` 结尾。`std::strlen(str)` 返回有效字符的数量，但不包含末尾的 `\0`。

``` cpp
char text[] = "hello";
std::strlen(text);  // 结果为 5
```

因此，字符串 `"hello"` 的有效下标是 `0` 到 `4`；下标 `5` 指向结尾的 `\0`，不应通过本函数访问或修改。

### `assert`：运行期断言

`assert` 用于验证程序运行时必须满足的条件。条件为 `false` 时，程序会终止，并提示断言失败的位置。

``` cpp
assert(str != nullptr);
assert(index < std::strlen(str));
```

本任务中，第一条断言确保字符串指针有效；第二条断言确保下标位于有效字符范围内。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `getCharacter` 的函数体。
3. 函数必须采用以下声明形式：

``` cpp
char& getCharacter(char str[], std::size_t index);
```

1. 函数必须先使用 `assert` 检查 `str != nullptr`。
2. 函数必须使用 `assert(index < std::strlen(str))` 检查下标未越界。
3. 函数应返回 `str[index]` 的引用，而不是返回局部变量的引用。
4. 在 `test` 函数中，先验证函数的返回值，再通过返回的引用修改字符串字符。
5. 在 `main` 函数中调用 `test` 函数，并保留测试通过信息与返回语句。

## 待完成代码

``` cpp
#include <cassert>
#include <cstddef>
#include <cstring>
#include <iostream>
#include <string>

// TODO：返回 str 中下标为 index 的字符引用
char& getCharacter(char str[], std::size_t index) {
    // TODO：检查 str 是否为空
    // TODO：检查 index 是否在有效字符范围内
    // TODO：返回指定字符的引用
}

void test() {
    char text[] = "hello";

    // 先测试函数返回的字符值是否正确
    assert(getCharacter(text, 0) == 'h');
    assert(getCharacter(text, 1) == 'e');
    assert(getCharacter(text, 4) == 'o');

    // 再通过函数返回的引用修改指定字符
    getCharacter(text, 0) = 'H';
    getCharacter(text, 4) = '!';

    assert(text[0] == 'H');
    assert(text[4] == '!');
    assert(std::string(text) == "Hell!");

    char& ch = getCharacter(text, 1);
    ch = 'A';

    assert(text[1] == 'A');
    assert(std::string(text) == "HAll!");

    // 下列调用会触发函数内部断言并终止程序，因此不应在正常测试中执行：
    // getCharacter(text, 5);
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`test` 函数应至少覆盖以下三类测试：

| 测试项           | 调用方式                                      | 预期结果                    |
| ---------------- | --------------------------------------------- | --------------------------- |
| 读取字符         | `getCharacter(text, 0)`                       | 返回 `'h'`                  |
| 直接修改字符     | `getCharacter(text, 0) = 'H'`                 | 字符串变为 `"Hello"` 的开头 |
| 通过引用变量修改 | `char& ch = getCharacter(text, 1); ch = 'A';` | 字符串变为 `"HAll!"`        |

对于字符串 `"hello"`，`std::strlen(text)` 的结果为 `5`，有效下标范围是 `[0, 4]`。以下调用属于越界访问：

``` cpp
getCharacter(text, 5);
```

该调用会触发 `assert(index < std::strlen(str))`，程序将终止。这正是断言用于发现违反函数前置条件错误的作用。

---

开始你的任务吧，祝你成功！
