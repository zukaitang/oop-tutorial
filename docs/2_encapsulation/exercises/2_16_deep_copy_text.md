# 练习：为动态文本类实现深拷贝构造

---

## 任务描述

设计动态文本类 `Text`。该类使用 `char*` 保存字符串内容，并实现拷贝构造函数，使复制得到的新对象拥有独立的字符数组。

## 相关知识

### 浅拷贝的问题

若类中有指针成员，而不自定义拷贝构造函数，编译器生成的默认拷贝构造通常只复制指针地址。这种浅拷贝会让两个对象指向同一段动态内存。

```cpp
// 错误示意：两个对象共享同一块 content 内存
Text second = first;
```

共享内存会导致修改一个对象影响另一个对象；两个析构函数重复释放同一地址时，还会产生未定义行为。

### 深拷贝构造

深拷贝需要为新对象重新申请内存，并复制原对象内容：

```cpp
Text::Text(const Text& other) {
    content = new char[std::strlen(other.content) + 1];
    std::strcpy(content, other.content);
}
```

拷贝构造函数的参数应使用 `const Text&`，避免递归调用拷贝构造函数，并保证不会修改源对象。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Text.h` 中声明类 `Text`，私有成员为 `char* content`。
3. 声明并实现以下成员函数：

   ```cpp
   explicit Text(const char* text);
   Text(const Text& other);
   ~Text();
   void Set(std::size_t index, char value);
   const char* CStr() const;
   void Print() const;
   ```

4. 普通构造函数和拷贝构造函数都必须申请独立的字符数组并复制内容；析构函数使用 `delete[]` 释放内存。
5. `Set` 修改指定字符，并使用断言检查下标不能超过字符串长度。
6. 在 `test` 中使用 `Text copy = original;` 调用拷贝构造函数，修改副本后验证原对象不变，并用 `assert` 验证输出。

## 待完成代码

### Text.h

```cpp
#ifndef TEXT_H
#define TEXT_H

#include <cstddef>

// TODO：声明 Text 类

#endif
```

### Text.cpp

```cpp
#include "Text.h"

// TODO：定义 Text 的构造、拷贝构造、析构和成员函数
```

### main.cpp

```cpp
#include "Text.h"

#include <cassert>
#include <cstring>
#include <iostream>
#include <sstream>

void test() {
    Text original("hello");
    Text copy = original;
    copy.Set(0, 'H');

    assert(std::strcmp(original.CStr(), "hello") == 0);
    assert(std::strcmp(copy.CStr(), "Hello") == 0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    original.Print();
    copy.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "hello\nHello\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

修改副本首字符后，原对象仍为 `hello`，副本为 `Hello`。若错误地进行浅拷贝，原对象也会被修改，或在程序结束时发生重复释放问题。

```bash
g++ -std=c++11 Text.cpp main.cpp -o deep_copy_text
```

---

开始你的任务吧，祝你成功！
