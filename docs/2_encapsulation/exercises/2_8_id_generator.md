# 练习：使用静态成员生成编号

---

## 任务描述

设计编号生成类 `IdGenerator`。每次创建对象时，为对象分配一个从 `1001` 开始递增的唯一编号；编号序列由类的静态成员统一管理。

## 相关知识

### 共享编号序列

若每个对象各自保存编号起点，将无法保证编号不重复。将下一个可分配编号设置为静态成员后，所有对象共享同一序列：

```cpp
class IdGenerator {
private:
    static int nextId;
    int id;
};
```

### 静态函数重置共享状态

静态成员函数可用于管理类的共享状态，例如重置编号起点：

```cpp
static void Reset(int startId);
```

调用时使用 `IdGenerator::Reset(2001)`，而不是通过某个对象调用。

## 编程要求

1. 在 `IdGenerator.h` 中声明类 `IdGenerator`，包含私有成员 `id` 和静态成员 `nextId`。
2. 声明并实现：

   ```cpp
   IdGenerator();
   int GetId() const;
   static void Reset(int startId);
   ```

3. `nextId` 的初始值为 `1001`。构造函数将当前 `nextId` 赋给 `id`，然后使 `nextId` 加一。
4. `Reset` 将 `nextId` 设置为给定起始值。
5. 在 `test` 中创建对象并使用 `assert` 验证编号序列和重置后的结果。

## 待完成代码

### IdGenerator.h

```cpp
#ifndef ID_GENERATOR_H
#define ID_GENERATOR_H

// TODO：声明 IdGenerator 类

#endif
```

### IdGenerator.cpp

```cpp
#include "IdGenerator.h"

// TODO：定义静态数据成员和成员函数
```

### main.cpp

```cpp
#include "IdGenerator.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    IdGenerator first;
    IdGenerator second;
    assert(first.GetId() == 1001);
    assert(second.GetId() == 1002);

    IdGenerator::Reset(2001);
    IdGenerator third;
    assert(third.GetId() == 2001);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "新编号 " << third.GetId() << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "新编号 2001\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

前两个对象的编号应为 `1001`、`1002`；重置起点为 `2001` 后，新对象编号应为 `2001`。

```bash
g++ -std=c++11 IdGenerator.cpp main.cpp -o id_generator
```

---

开始你的任务吧，祝你成功！
