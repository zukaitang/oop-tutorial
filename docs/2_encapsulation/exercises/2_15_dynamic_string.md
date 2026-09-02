# 练习15：使用动态内存保存 C 字符串

---

## 任务描述

设计文本类 `Text`，使用动态内存保存传入的 C 字符串。构造函数应复制字符串内容，而析构函数负责释放申请的字符数组。

## 相关知识

### 动态字符数组

C 风格字符串以空字符 `\0` 结尾。若字符串长度为 `length`，保存它需要申请 `length + 1` 个字符：

``` cpp
char* content = new char[length + 1];
```

使用 `<cstring>` 中的 `std::strlen` 可得到字符串长度，`std::strcpy` 可复制字符串内容。

### 深拷贝字符串内容

不能只保存传入指针的地址，否则外部字符数组发生变化或失效后，类中的内容也会受到影响。应申请自己的内存并复制内容：

``` cpp
content = new char[std::strlen(text) + 1];
std::strcpy(content, text);
```

### 析构函数释放资源

`content` 通过 `new[]` 申请，因此析构函数必须使用 `delete[]` 释放。释放后应将指针置为 `nullptr`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Text.h` 中声明类 `Text`，私有成员为 `char* content`。
3. 声明并实现以下成员函数：

   ```cpp
   explicit Text(const char* text);
   ~Text();
   std::size_t Length() const;
   char Get(std::size_t index) const;
   void Print() const;
   ```

1. 构造函数申请足够的字符数组空间，复制参数字符串；析构函数使用 `delete[]` 释放内存。
2. `Length` 返回字符串长度；`Get` 返回指定下标字符并使用断言检查下标；`Print` 输出字符串和换行。
3. 在 `test` 中先修改原始字符数组，再验证 `Text` 对象仍保留构造时复制的内容；同时断言长度、字符和输出。

## 待完成代码

### Text.h

``` cpp
#ifndef TEXT_H
#define TEXT_H

#include <cstddef>

// TODO：声明 Text 类

#endif
```

### Text.cpp

``` cpp
#include "Text.h"

// TODO：定义 Text 的成员函数
```

### main.cpp

``` cpp
#include "Text.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    char source[] = "hello";
    Text text(source);
    source[0] = 'H';

    assert(text.Length() == 5);
    assert(text.Get(0) == 'h');
    assert(text.Get(4) == 'o');

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    text.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "hello\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`source` 从 `"hello"` 改为 `"Hello"` 后，`Text` 中保存的副本仍应为 `"hello"`。预期输出：

``` text
hello
```

``` bash
g++ -std=c++11 Text.cpp main.cpp -o dynamic_string
```

---

开始你的任务吧，祝你成功！
