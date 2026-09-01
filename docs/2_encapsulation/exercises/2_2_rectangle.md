# 练习：创建对象并计算长方形面积

---

## 任务描述

设计一个长方形类 `Rectangle`。通过普通函数创建 `Rectangle` 对象，并通过对象调用成员函数计算长方形面积。

本题将练习对象的创建、对象作为函数返回值与参数，以及通过对象访问公有成员函数。

## 相关知识

### 对象

类是对一组具有共同特征的对象的抽象描述；类的实例称为对象。创建对象的语法与声明普通变量类似：

```cpp
Rectangle rect;  // 创建一个 Rectangle 对象
```

每个对象分别拥有自己的数据成员存储空间，而同一个类的对象共享该类定义的成员函数。

### 通过对象访问成员

使用成员访问运算符 `.` 可以访问对象的公有成员。对于成员函数，使用对象名、`.` 和函数名调用：

```cpp
rect.Set(10, 15);
int area = rect.GetArea();
```

私有数据成员不能在类外直接访问，因此下面的写法是错误的：

```cpp
// rect.height = 10;  // 错误：height 是私有成员
```

应当通过公有成员函数 `Set` 设置长方形的高和宽。

### 对象作为函数的返回值和参数

对象可以作为普通函数的返回值或参数：

```cpp
Rectangle GetRect(int h, int w);  // 返回 Rectangle 对象
int GetRectArea(Rectangle rect);  // 接收 Rectangle 对象并返回面积
```

在 `GetRect` 中创建对象、设置其数据后返回；在 `GetRectArea` 中通过 `rect.GetArea()` 调用对象的公有成员函数。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在文件 `Rectangle.h` 中声明类 `Rectangle`，包含以下成员：

   ```cpp
   private:
       int height;
       int width;

   public:
       void Set(int h, int w);
       int GetArea();
   ```

3. 在 `Rectangle.h` 中同时声明以下普通函数：

   ```cpp
   Rectangle GetRect(int h, int w);
   int GetRectArea(Rectangle rect);
   ```

4. 在文件 `Rectangle.cpp` 中包含 `Rectangle.h`，并完成两个成员函数和两个普通函数的定义。
5. `Set` 将参数 `h`、`w` 分别赋给 `height`、`width`；`GetArea` 返回 `height * width`。
6. `GetRect` 创建一个 `Rectangle` 对象，调用 `Set(h, w)` 设置长和宽后返回该对象。
7. `GetRectArea` 调用参数对象的 `GetArea()`，并返回得到的面积。
8. 在 `main.cpp` 的 `test` 函数中使用 `assert` 验证两组面积计算结果，并断言输出字符串与预期完全一致；在 `main` 函数中调用 `test`。

## 待完成代码

### Rectangle.h

```cpp
#ifndef RECTANGLE_H
#define RECTANGLE_H

// TODO：声明 Rectangle 类和两个普通函数

#endif
```

### Rectangle.cpp

```cpp
#include "Rectangle.h"

// TODO：定义 Rectangle 的成员函数和两个普通函数
```

### main.cpp

```cpp
#include "Rectangle.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Rectangle rect1 = GetRect(10, 15);
    assert(GetRectArea(rect1) == 150);

    std::ostringstream output1;
    std::streambuf* oldBuffer = std::cout.rdbuf(output1.rdbuf());
    std::cout << "长方形的面积为：" << GetRectArea(rect1) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output1.str() == "长方形的面积为：150\n");

    Rectangle rect2 = GetRect(100, 100);
    assert(GetRectArea(rect2) == 10000);

    std::ostringstream output2;
    oldBuffer = std::cout.rdbuf(output2.rdbuf());
    std::cout << "长方形的面积为：" << GetRectArea(rect2) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output2.str() == "长方形的面积为：10000\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

测试包含两组长方形尺寸。`assert` 既验证 `GetRectArea` 的返回值，也验证输出格式：

| 高度 | 宽度 | 预期面积 | 预期输出 |
|---:|---:|---:|---|
| `10` | `15` | `150` | `长方形的面积为：150` |
| `100` | `100` | `10000` | `长方形的面积为：10000` |

可使用以下命令编译三个文件：

```bash
g++ -std=c++11 Rectangle.cpp main.cpp -o rectangle
```

---

开始你的任务吧，祝你成功！
