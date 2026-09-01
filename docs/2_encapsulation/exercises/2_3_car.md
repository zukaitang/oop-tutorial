# 练习：设计汽车类并控制汽车状态

---

## 任务描述

设计一个汽车类 `Car`，模拟车门、车灯和速度的状态变化。程序根据由字符 `1`～`6` 组成的命令串依次执行对应操作，并输出汽车的最终状态。

例如，命令串 `135` 依次表示打开车门、打开车灯、加速，最终状态应为车门和车灯均为 `ON`，速度为 `10`。

## 相关知识

### 对象的状态

对象的数据成员用于保存对象的状态。本题中，车门和车灯可用 `bool` 类型表示开关状态，速度可用 `int` 类型表示。

``` cpp
bool doorOn;   // true 表示 ON，false 表示 OFF
bool lightOn;
int speed;
```

不同的 `Car` 对象拥有各自独立的状态；对一个对象调用成员函数不会影响其他对象。

### 构造函数与初始状态

构造函数在创建对象时自动执行，常用于为数据成员设置初始值。构造函数的名称与类名相同，且没有返回类型。

``` cpp
class Car {
public:
    Car();
};

Car::Car() {
    // 初始化数据成员
}
```

本题要求新创建的汽车对象满足：车门 `OFF`、车灯 `OFF`、速度为 `0`。

### 根据命令调用成员函数

可以遍历字符串中的每个命令字符，并使用 `switch` 语句调用对应的成员函数：

``` cpp
switch (command) {
case '1':
    OpenDoor();
    break;
case '2':
    CloseDoor();
    break;
// 其余命令类似处理
}
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Car.h` 中声明类 `Car`。类中应有私有数据成员表示车门、车灯和速度，并声明以下公有成员函数：

   ```cpp
   Car();
   void OpenDoor();
   void CloseDoor();
   void OpenLight();
   void CloseLight();
   void Accelerate();
   void Decelerate();
   void ExecuteCommand(char command);
   void PrintStatus();
   ```

1. 在 `Car.cpp` 中定义所有成员函数：

2. 构造函数将车门、车灯初始化为 `OFF`，速度初始化为 `0`；
3. `OpenDoor`、`CloseDoor`、`OpenLight`、`CloseLight` 分别改变对应开关状态；
4. `Accelerate` 每次使速度增加 `10`，`Decelerate` 每次使速度减少 `10`；
5. `ExecuteCommand` 将字符 `'1'`～`'6'` 分别映射为上述六种操作；
6. `PrintStatus` 按“车门、车灯、速度”的顺序输出最终状态。

7. 在 `main.cpp` 中完成普通函数 `executeCommands`：遍历命令字符串，并对 `Car` 对象逐个调用 `ExecuteCommand`。
8. 在 `test` 中使用 `assert` 验证两组命令的输出字符串与预期结果完全一致；在 `main` 中调用 `test`。

## 待完成代码

### Car.h

``` cpp
#ifndef CAR_H
#define CAR_H

// TODO：声明 Car 类

#endif
```

### Car.cpp

``` cpp
#include "Car.h"
#include <iostream>

// TODO：定义 Car 的成员函数
```

### main.cpp

``` cpp
#include "Car.h"

#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

// TODO：遍历 commands，依次调用 car.ExecuteCommand(command)
void executeCommands(Car& car, const std::string& commands) {
    // TODO
}

void test() {
    Car car1;
    car1.init();
    executeCommands(car1, "135");

    std::ostringstream output1;
    std::streambuf* oldBuffer = std::cout.rdbuf(output1.rdbuf());
    car1.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(output1.str() == "车门 ON\n车灯 ON\n速度 10\n");

    Car car2;
    car2.init()
    executeCommands(car2, "135562");

    std::ostringstream output2;
    oldBuffer = std::cout.rdbuf(output2.rdbuf());
    car2.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(output2.str() == "车门 OFF\n车灯 ON\n速度 10\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

命令串长度不超过 `20`，且只包含字符 `1`～`6`。测试通过捕获 `PrintStatus` 的输出，并用 `assert` 逐字比较。

| 命令串   | 执行的关键操作                               | 预期输出                         |
| -------- | -------------------------------------------- | -------------------------------- |
| `135`    | 打开车门、打开车灯、加速一次                 | `车门 ON`、`车灯 ON`、`速度 10`  |
| `135562` | 打开车门和车灯、加速两次、减速一次、关闭车门 | `车门 OFF`、`车灯 ON`、`速度 10` |

可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Car.cpp main.cpp -o car
```

---

开始你的任务吧，祝你成功！
