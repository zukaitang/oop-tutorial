# 练习：使用友元函数计算两点距离

---

## 任务描述

设计平面点类 `Point`，坐标成员设为私有。编写友元函数 `Distance`，直接访问两个点的私有坐标并计算它们之间的欧氏距离。

## 相关知识

### 友元函数

友元函数不是类的成员函数，但可访问类的私有成员。需要在类中使用 `friend` 声明：

```cpp
class Point {
private:
    double x;
    double y;

public:
    friend double Distance(const Point& first, const Point& second);
};
```

友元声明只授予访问权限，函数定义时不应使用 `Point::` 前缀。

### 欧氏距离

两点 `(x1, y1)` 和 `(x2, y2)` 的距离为：

```text
sqrt((x1 - x2)² + (y1 - y2)²)
```

可通过 `<cmath>` 中的 `std::sqrt` 计算平方根。

## 编程要求

1. 在 `Point.h` 中声明 `Point` 类，私有成员为 `double x`、`double y`。
2. 声明构造函数 `Point(double x, double y)`，并将普通函数 `Distance` 声明为 `Point` 的友元。
3. 在 `Point.cpp` 中定义构造函数和友元函数 `Distance`。
4. `Distance` 必须直接使用两个对象的私有 `x`、`y` 成员计算距离。
5. 在 `test` 中验证点 `(0, 0)` 与 `(3, 4)` 的距离为 `5`，并验证输出。

## 待完成代码

### Point.h

```cpp
#ifndef POINT_H
#define POINT_H

// TODO：声明 Point 类和友元函数

#endif
```

### Point.cpp

```cpp
#include "Point.h"

// TODO：定义 Point 的构造函数和 Distance 函数
```

### main.cpp

```cpp
#include "Point.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Point origin(0.0, 0.0);
    Point point(3.0, 4.0);
    assert(Distance(origin, point) == 5.0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "两点距离 " << Distance(origin, point) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "两点距离 5\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`(0, 0)` 与 `(3, 4)` 构成直角三角形，其距离为 `5`。可使用以下命令编译：

```bash
g++ -std=c++11 Point.cpp main.cpp -o point_friend
```

---

开始你的任务吧，祝你成功！
