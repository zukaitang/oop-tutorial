# 练习：复制 C 风格字符串

---

## 任务描述

编写函数 `copyName`，将源 C 风格字符串复制到目标字符数组中，并使用断言保证目标数组容量足够。

## 相关知识

### C 风格字符串

C 风格字符串是以空字符 `\0` 结束的字符数组：

```cpp
char name[] = "Alice";
```

数组中除了五个可见字符外，还保存一个结尾的 `\0`，因此字符串长度与数组容量并不相同。

### <cstring> 工具函数

`std::strlen` 返回字符串中可见字符的数量；`std::strcpy` 会连同结尾的 `\0` 一起复制字符串。

```cpp
std::size_t length = std::strlen(source);
std::strcpy(destination, source);
```

复制前，目标数组容量必须至少为 `length + 1`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数：

   ```cpp
   void copyName(char destination[], std::size_t capacity, const char source[]);
   ```

3. 使用断言检查 `destination` 和 `source` 不为空，并检查 `capacity >= std::strlen(source) + 1`。
4. 使用 `<cstring>` 中的函数复制字符串。
5. 在 `test` 中验证复制结果、字符串长度和输出内容。

## 待完成代码

```cpp
#include <cassert>
#include <cstddef>
#include <cstring>
#include <iostream>
#include <sstream>

// TODO：完成字符串复制
void copyName(char destination[], std::size_t capacity, const char source[]) {
    // TODO
}

void test() {
    char source[] = "张如雪";
    char destination[32] = {};
    copyName(destination, sizeof(destination), source);

    assert(std::strcmp(destination, "张如雪") == 0);
    assert(std::strlen(destination) == std::strlen(source));

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "姓名 " << destination << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "姓名 张如雪\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

复制后，目标数组应独立保存 `张如雪`，预期输出为：

```text
姓名 张如雪
```

---

开始你的任务吧，祝你成功！
