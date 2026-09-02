# 练习17：为动态数组类实现深拷贝赋值

---

## 任务描述

设计动态整数数组类 `IntArray`，实现拷贝构造函数和拷贝赋值运算符，使对象在复制和赋值后都拥有独立的动态数组。

## 相关知识

### 拷贝赋值运算符

对象已经存在时，使用赋值语句会调用拷贝赋值运算符：

``` cpp
IntArray target(1);
target = source;
```

其函数原型通常为：

``` cpp
IntArray& operator=(const IntArray& other);
```

返回 `IntArray&` 可支持连续赋值，例如 `a = b = c`。

### 安全的深拷贝赋值

赋值时应避免内存泄漏和自赋值错误。一个基本步骤是：

1. 判断 `this != &other`；
2. 为新数据申请内存并复制源数据；
3. 释放旧内存；
4. 更新成员并返回 `*this`。

``` cpp
if (this != &other) {
    // 复制 other 的数据，并替换当前对象的数据
}
return *this;
```

### 三法则

类若自行管理动态资源，通常需要同时考虑析构函数、拷贝构造函数和拷贝赋值运算符，避免资源被重复释放或泄漏。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `IntArray.h` 中声明类 `IntArray`，私有成员为 `int* data` 和 `std::size_t size`。
3. 声明并实现：

   ```cpp
   explicit IntArray(std::size_t size);
   IntArray(const IntArray& other);
   IntArray& operator=(const IntArray& other);
   ~IntArray();
   void Set(std::size_t index, int value);
   int Get(std::size_t index) const;
   ```

1. 拷贝构造函数和赋值运算符都必须深拷贝数组元素。赋值运算符需要处理 `array = array` 的自赋值情况。
2. 在 `test` 中验证拷贝构造、普通赋值和自赋值；修改副本后，源对象的元素不能改变。

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

// TODO：定义 IntArray 的全部成员函数
```

### main.cpp

``` cpp
#include "IntArray.h"

#include <cassert>
#include <iostream>

void test() {
    IntArray source(3);
    source.Set(0, 10);
    source.Set(1, 20);
    source.Set(2, 30);

    IntArray copied(source);
    copied.Set(0, 99);
    assert(source.Get(0) == 10);
    assert(copied.Get(0) == 99);

    IntArray assigned(1);
    assigned = source;
    assigned.Set(1, 88);
    assert(source.Get(1) == 20);
    assert(assigned.Get(1) == 88);

    assigned = assigned;
    assert(assigned.Get(0) == 10);
    assert(assigned.Get(1) == 88);
    assert(assigned.Get(2) == 30);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

测试覆盖以下情形：拷贝构造后修改副本、赋值后修改目标对象、对象自赋值。每种情况下，动态数组都应安全管理，且不应让不同对象共享同一块数组内存。

``` bash
g++ -std=c++11 IntArray.cpp main.cpp -o deep_copy_assignment
```

---

开始你的任务吧，祝你成功！
