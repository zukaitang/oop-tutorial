# 练习6：使用组合类模拟汽车发动机

---

## 任务描述

设计发动机类 `Engine` 和汽车类 `Car`。`Car` 对象包含一个 `Engine` 对象，通过汽车的启动、加速和熄火操作协调发动机状态，并输出汽车最终状态。

本题练习外层对象如何调用成员对象的函数，使多个类共同完成一项功能。

## 相关知识

### 组合关系中的职责划分

组合关系中，成员对象负责自身状态，外层对象负责组织整体行为。对于本题：

- `Engine` 负责记录发动机是否启动、当前转速；
- `Car` 持有一个 `Engine` 对象，并将启动、加速、熄火等请求委托给发动机。

``` cpp
class Car {
private:
    Engine engine;
};
```

### 调用成员对象的函数

在 `Car` 的成员函数中，可以直接使用成员对象名调用其公有函数：

``` cpp
void Car::Start() {
    engine.Start();
}
```

这里的 `engine` 是一个对象，不是指针，因此使用 `.` 而不是 `->`。

### 状态变化

本题规定：发动机启动后转速为 `800`；每次加速使转速增加 `500`；熄火后转速变为 `0`。发动机未启动时调用加速不应改变转速。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Car.h` 中声明 `Engine` 类。它应提供以下公有成员函数：

   ```cpp
   Engine();
   void Start();
   void Stop();
   void Accelerate();
   bool IsRunning() const;
   int GetRpm() const;
   ```

1. 在同一头文件中声明 `Car` 类，并将 `Engine engine` 声明为私有成员。`Car` 应提供：

   ```cpp
   void Start();
   void Accelerate();
   void Stop();
   void PrintStatus() const;
   ```

1. 在 `Car.cpp` 中定义所有成员函数。`Car` 的 `Start`、`Accelerate`、`Stop` 分别调用内部 `engine` 对象的对应函数。
2. `PrintStatus` 按如下格式输出：

   ```text
   发动机 ON
   转速 1800
   ```

1. 在 `main.cpp` 中创建 `Car` 对象，测试启动和两次加速后的状态，以及熄火后的状态；使用 `assert` 对输出进行验证。

## 待完成代码

### Car.h

``` cpp
#ifndef CAR_H
#define CAR_H

// TODO：声明 Engine 类和 Car 类

#endif
```

### Car.cpp

``` cpp
#include "Car.h"

// TODO：定义 Engine 和 Car 的成员函数
```

### main.cpp

``` cpp
#include "Car.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Car car;
    car.Start();
    car.Accelerate();
    car.Accelerate();

    std::ostringstream runningOutput;
    std::streambuf* oldBuffer = std::cout.rdbuf(runningOutput.rdbuf());
    car.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(runningOutput.str() == "发动机 ON\n转速 1800\n");

    car.Stop();

    std::ostringstream stoppedOutput;
    oldBuffer = std::cout.rdbuf(stoppedOutput.rdbuf());
    car.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(stoppedOutput.str() == "发动机 OFF\n转速 0\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

发动机启动后转速为 `800`，两次加速后转速为 `1800`。熄火后，发动机状态应为 `OFF`，转速应归零。

可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Car.cpp main.cpp -o car_engine
```

---

开始你的任务吧，祝你成功！
