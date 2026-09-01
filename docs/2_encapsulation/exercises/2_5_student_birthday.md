# 练习：使用组合类表示学生生日

---

## 任务描述

设计日期类 `Date` 和学生类 `Student`。将 `Date` 对象作为 `Student` 的成员，用于保存学生生日，并输出完整的学生信息。

本题练习“类的组合”：一个类的对象作为另一个类的数据成员。`Student` 对象创建时，其内部的 `Date` 成员对象也会随之构造。

## 相关知识

### 类的组合

当一个类拥有另一个类的对象作为数据成员时，称为类的组合。例如，学生拥有一个生日日期：

```cpp
class Student {
private:
    std::string name;
    Date birthday;
};
```

`birthday` 是 `Student` 的组成部分，它的生命周期与所属的 `Student` 对象一致。

### 成员对象的初始化

如果成员对象没有无参构造函数，外层类必须在构造函数初始化列表中调用其构造函数：

```cpp
Student::Student(const std::string& name, int year, int month, int day)
    : name(name), birthday(year, month, day) {
}
```

成员对象的构造顺序由其在类中的声明顺序决定；析构时顺序相反。

### 委托成员对象完成工作

外层类可调用成员对象的公有成员函数。例如，`Student::Print` 可调用 `birthday.Print()` 输出生日。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Student.h` 中声明 `Date` 类，私有成员为 `year`、`month`、`day`，并提供：

   ```cpp
   Date(int year, int month, int day);
   void Print() const;
   ```

3. 在同一头文件中声明 `Student` 类，私有成员包括姓名和一个 `Date` 类型的生日成员：

   ```cpp
   std::string name;
   Date birthday;
   ```

4. `Student` 应提供构造函数和 `Print` 函数：

   ```cpp
   Student(const std::string& name, int year, int month, int day);
   void Print() const;
   ```

5. 在 `Student.cpp` 中使用初始化列表构造 `Student` 的 `birthday` 成员。`Date::Print` 输出 `年-月-日`；`Student::Print` 输出姓名和生日，格式见测试说明。
6. 在 `main.cpp` 中创建两个 `Student` 对象，捕获 `Print` 输出并用 `assert` 验证。

## 待完成代码

### Student.h

```cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

// TODO：声明 Date 类和 Student 类

#endif
```

### Student.cpp

```cpp
#include "Student.h"

// TODO：定义 Date 和 Student 的成员函数
```

### main.cpp

```cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Student student1("李红霞", 2004, 5, 12);
    Student student2("张如雪", 2003, 11, 8);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student1.Print();
    student2.Print();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "姓名 李红霞 生日 2004-5-12\n"
           "姓名 张如雪 生日 2003-11-8\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`Student::Print` 应调用内部 `Date` 对象的输出功能。两名学生的预期输出为：

```text
姓名 李红霞 生日 2004-5-12
姓名 张如雪 生日 2003-11-8
```

可使用以下命令编译三个文件：

```bash
g++ -std=c++11 Student.cpp main.cpp -o student_birthday
```

---

开始你的任务吧，祝你成功！
