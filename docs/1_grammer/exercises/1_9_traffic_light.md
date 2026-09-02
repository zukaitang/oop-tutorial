# 练习9：使用 enum class 判断交通信号

---

## 任务描述

定义作用域枚举 `TrafficLight`，表示红灯、黄灯和绿灯三种交通信号。

编写函数 `canPass`，根据传入的交通信号判断车辆是否可以通行：只有绿灯 `TrafficLight::Green` 时返回 `true`，红灯与黄灯时均返回 `false`。

## 相关知识

### 作用域枚举 `enum class`

C++11 的 `enum class` 用于定义作用域枚举。枚举值必须通过“枚举类型名 + `::` + 枚举值”访问：

``` cpp
enum class TrafficLight {
    Red,
    Yellow,
    Green
};

TrafficLight light = TrafficLight::Red;
```

与传统 `enum` 相比，`enum class` 具有更好的类型安全性，不会隐式转换为整数。

### 使用 switch 判断枚举值

`switch` 语句适合根据枚举值执行不同分支：

``` cpp
switch (light) {
case TrafficLight::Green:
    return true;
case TrafficLight::Red:
case TrafficLight::Yellow:
    return false;
}
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 保留题目给出的 `enum class TrafficLight` 定义。
3. 完成函数 `canPass` 的函数体。
4. 函数必须采用以下声明形式：

``` cpp
bool canPass(TrafficLight light);
```

1. 必须使用 `switch` 语句完成判断。
2. 只有 `TrafficLight::Green` 返回 `true`。
3. `TrafficLight::Red` 和 `TrafficLight::Yellow` 均返回 `false`。
4. 不得将枚举值转换为整数后再进行比较。
5. 在 `test` 函数中使用 `assert` 验证三种信号的结果。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>

enum class TrafficLight {
    Red,
    Yellow,
    Green
};

// TODO：只有 Green 返回 true，其他情况返回 false
bool canPass(TrafficLight light) {
    // TODO
}

void test() {
    assert(!canPass(TrafficLight::Red));
    assert(!canPass(TrafficLight::Yellow));
    assert(canPass(TrafficLight::Green));
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 函数调用                        | 预期结果 |
| ------ | ------------------------------- | :------: |
| 红灯   | `canPass(TrafficLight::Red)`    | `false`  |
| 黄灯   | `canPass(TrafficLight::Yellow)` | `false`  |
| 绿灯   | `canPass(TrafficLight::Green)`  |  `true`  |

注意：不能写成 `canPass(Red)`。`enum class` 的枚举值位于自己的作用域中，必须写为 `TrafficLight::Red`。

---

开始你的任务吧，祝你成功！
