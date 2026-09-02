# 练习14：使用命名空间区分同名计算函数

---

## 任务描述

在 `Geometry` 与 `Physics` 两个命名空间中分别完成同名函数 `calculate`。

- `Geometry::calculate(radius)`：计算半径为 `radius` 的圆面积，取 π 为 `3.14`；
- `Physics::calculate(mass)`：计算质量为 `mass` 的物体重力，取重力加速度为 `9.8`。

两个函数的名称相同，但属于不同命名空间，因此不会产生名称冲突。调用时必须通过完全限定名明确指定函数来源。

## 相关知识

### 命名空间与名称冲突

命名空间用于组织相关代码，并隔离不同模块中的名称：

``` cpp
namespace Geometry {
    double calculate(double radius);
}

namespace Physics {
    double calculate(double mass);
}
```

虽然两个函数都叫作 `calculate`，但它们的完整名称不同，因此可以同时存在。

### 完全限定名

使用作用域运算符 `::` 可以明确访问某个命名空间中的成员：

``` cpp
double area = Geometry::calculate(2.0);
double weight = Physics::calculate(2.0);
```

完全限定名能清楚表达名称来源，避免歧义。在本题中，不应使用 `using namespace` 将整个命名空间引入当前作用域。

### 浮点数比较

浮点数计算可能存在微小误差。测试时使用 `std::fabs` 比较计算结果与预期值之间的差值：

``` cpp
assert(std::fabs(result - expected) < 1e-9);
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 保留题目给出的 `Geometry` 与 `Physics` 命名空间。
3. 分别完成两个 `calculate` 函数的函数体。
4. `Geometry::calculate` 应返回 `3.14 * radius * radius`。
5. `Physics::calculate` 应返回 `9.8 * mass`。
6. 调用函数时必须使用完全限定名。
7. 不得使用 `using namespace Geometry;` 或 `using namespace Physics;`。
8. 在 `test` 函数中使用 `assert` 验证结果。

## 待完成代码

``` cpp
#include <cassert>
#include <cmath>
#include <iostream>

namespace Geometry {
    // TODO：计算圆面积
    double calculate(double radius) {
        // TODO
    }
}

namespace Physics {
    // TODO：计算物体重力
    double calculate(double mass) {
        // TODO
    }
}

void test() {
    assert(std::fabs(Geometry::calculate(2.0) - 12.56) < 1e-9);
    assert(std::fabs(Geometry::calculate(5.0) - 78.5) < 1e-9);

    assert(std::fabs(Physics::calculate(2.0) - 19.6) < 1e-9);
    assert(std::fabs(Physics::calculate(10.0) - 98.0) < 1e-9);
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项               | 函数调用                   | 预期结果 |
| -------------------- | -------------------------- | -------: |
| 半径为 2 的圆面积    | `Geometry::calculate(2.0)` |  `12.56` |
| 半径为 5 的圆面积    | `Geometry::calculate(5.0)` |   `78.5` |
| 质量为 2 的物体重力  | `Physics::calculate(2.0)`  |   `19.6` |
| 质量为 10 的物体重力 | `Physics::calculate(10.0)` |   `98.0` |

如果省略命名空间直接调用 `calculate(2.0)`，编译器无法确定应调用哪个同名函数。使用 `Geometry::calculate` 或 `Physics::calculate` 可以消除歧义。

---

开始你的任务吧，祝你成功！
