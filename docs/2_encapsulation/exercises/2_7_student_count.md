# 练习：使用静态成员统计学生对象数量

---

## 任务描述

设计学生类 `Student`，使用静态数据成员统计当前存活的学生对象数量。每创建一个对象，数量加一；每销毁一个对象，数量减一。

## 相关知识

### 静态数据成员

静态数据成员属于类本身，而不属于某一个对象。同一个类的所有对象共享同一份静态数据成员。

```cpp
class Student {
private:
    static int count;
};

int Student::count = 0;  // 在类外定义并初始化
```

### 静态成员函数

静态成员函数可使用类名直接调用，不需要先创建对象。静态成员函数只能直接访问静态成员。

```cpp
int Student::GetCount() {
    return count;
}

int current = Student::GetCount();
```

## 编程要求

1. 在 `Student.h` 中声明 `Student` 类，包含私有静态数据成员 `count`。
2. 声明公有构造函数、析构函数和静态成员函数：

   ```cpp
   Student();
   ~Student();
   static int GetCount();
   ```

3. 在 `Student.cpp` 中将 `Student::count` 初始化为 `0`；构造函数使其加一，析构函数使其减一。
4. 在 `test` 中用 `assert` 验证不同作用域内对象数量的变化，并验证输出。

## 待完成代码

### Student.h

```cpp
#ifndef STUDENT_H
#define STUDENT_H

// TODO：声明 Student 类

#endif
```

### Student.cpp

```cpp
#include "Student.h"

// TODO：定义静态数据成员和成员函数
```

### main.cpp

```cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    assert(Student::GetCount() == 0);

    Student first;
    Student second;
    assert(Student::GetCount() == 2);

    {
        Student third;
        assert(Student::GetCount() == 3);
    }
    assert(Student::GetCount() == 2);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "当前学生数 " << Student::GetCount() << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "当前学生数 2\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`third` 离开内部作用域后会自动析构，数量从 `3` 恢复为 `2`。最终预期输出：

```text
当前学生数 2
```

```bash
g++ -std=c++11 Student.cpp main.cpp -o student_count
```

---

开始你的任务吧，祝你成功！
