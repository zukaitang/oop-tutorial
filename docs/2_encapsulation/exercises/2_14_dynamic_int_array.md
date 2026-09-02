# 练习14：使用动态内存管理整数数组

---

## 任务描述

设计动态整数数组类 `IntArray`。对象创建时根据给定长度在堆区申请整数数组；对象销毁时释放该数组，并提供设置元素、读取元素、求和和输出功能。

## 相关知识

### new[] 与 delete[]

运行时才能确定数组长度时，可使用 `new[]` 在动态存储区申请数组：

``` cpp
int* data = new int[size];
```

用 `new[]` 申请的数组必须使用 `delete[]` 释放：

``` cpp
delete[] data;
data = nullptr;
```

`new[]` 与 `delete[]` 必须配对使用，不能将数组内存用普通 `delete` 释放。

### 类中管理动态资源

将动态数组指针作为类的私有成员时，构造函数负责申请资源，析构函数负责释放资源：

``` cpp
class IntArray {
private:
    int* data;
    std::size_t size;
};
```

析构函数会在对象离开作用域时自动执行，因此能确保对象拥有的动态数组被释放。

### 下标检查

访问数组元素前应确认下标在合法范围内。本题使用 `assert(index < size)` 进行检查。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `IntArray.h` 中声明类 `IntArray`，私有成员为动态数组指针 `data` 和数组长度 `size`。
3. 声明并实现以下成员函数：

   ```cpp
   explicit IntArray(std::size_t size);
   ~IntArray();
   void Set(std::size_t index, int value);
   int Get(std::size_t index) const;
   int Sum() const;
   void Print() const;
   ```

1. 构造函数使用 `new int[size]` 申请数组，并将每个元素初始化为 `0`；析构函数使用 `delete[]` 释放数组并将指针置为 `nullptr`。
2. `Set`、`Get` 在访问前使用断言检查下标；`Sum` 返回全部元素的和；`Print` 按空格分隔输出全部元素，末尾输出换行。
3. 在 `test` 中设置 4 个元素，使用 `assert` 验证读取值和总和，并捕获 `Print` 输出进行断言。

## 待完成代码

### IntArray.h

``` cpp
#ifndef INT_ARRAY_H
#define INT_ARRAY_H

#include <cstddef>

// TODO：声明 IntArray 类

#endif
```

### IntArray.cpp

``` cpp
#include "IntArray.h"

// TODO：定义 IntArray 的成员函数
```

### main.cpp

``` cpp
#include "IntArray.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    IntArray values(4);
    values.Set(0, 10);
    values.Set(1, -3);
    values.Set(2, 8);
    values.Set(3, 5);

    assert(values.Get(0) == 10);
    assert(values.Get(1) == -3);
    assert(values.Sum() == 20);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    values.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "10 -3 8 5\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

数组元素依次为 `10`、`-3`、`8`、`5`，总和为 `20`。预期输出：

``` text
10 -3 8 5
```

``` bash
g++ -std=c++11 IntArray.cpp main.cpp -o dynamic_int_array
```

---

开始你的任务吧，祝你成功！
