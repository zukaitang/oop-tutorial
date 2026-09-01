# 编程：检查整数成绩输入是否合法

---

## 任务描述

编写函数 `readScore`，从输入流中读取一个整数成绩。

读取成功时，将结果写入引用参数 `score` 并返回 `true`；读取失败时，函数应清除输入流的错误状态、忽略当前错误行，并返回 `false`。完成错误处理后，输入流应能继续读取下一行数据。

## 相关知识

### 输入状态检查

当输入内容与目标变量类型不匹配时，输入流会进入失败状态：

```cpp
int score;
std::cin >> score;

if (std::cin.fail()) {
    // 输入失败
}
```

### `clear` 与 `ignore`

输入失败后，需要先清除错误状态，再忽略当前行中的错误内容：

```cpp
in.clear();
in.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
```

`clear()` 使输入流恢复可读状态；`ignore()` 丢弃当前行剩余内容，避免下一次读取重复遇到同一份错误输入。

### 引用输出参数

函数参数 `int& score` 是引用参数。读取成功后，函数可以直接修改调用者的变量。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `readScore` 的函数体。
3. 函数必须采用以下声明形式：

```cpp
bool readScore(std::istream& in, int& score);
```

4. 使用 `in >> score` 读取整数。
5. 输入成功时返回 `true`。
6. 输入失败时返回 `false`，并依次调用 `in.clear()` 与 `in.ignore(...)`。
7. `ignore` 必须忽略至换行符 `\n`。
8. 在 `test` 函数中使用 `assert` 验证成功输入、失败输入及失败后的继续读取。

## 待完成代码

```cpp
#include <cassert>
#include <iostream>
#include <limits>
#include <sstream>

// TODO：读取一个整数；输入失败时恢复输入流状态并返回 false
bool readScore(std::istream& in, int& score) {
    // TODO
}

void test() {
    int score = 0;

    std::istringstream input1("85\n");
    assert(readScore(input1, score));
    assert(score == 85);

    std::istringstream input2("abc\n90\n");
    assert(!readScore(input2, score));  // 第一次读取失败
    assert(readScore(input2, score));   // 错误行被忽略后，可继续读取下一行
    assert(score == 90);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 输入内容 | 预期结果 |
|---|---|---|
| 正常整数输入 | `85` | 返回 `true`，`score == 85` |
| 非法输入 | `abc` | 返回 `false` |
| 失败后继续读取 | `abc` 后接 `90` | 第二次返回 `true`，`score == 90` |

若输入失败后只调用 `clear()` 而未调用 `ignore()`，下一次读取仍会遇到 `abc`，导致第二次测试失败。

---

开始你的任务吧，祝你成功！
