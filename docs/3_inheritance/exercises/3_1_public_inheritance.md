# 练习1：使用公有继承设计学生信息类

---

## 任务描述

设计人员类 `People` 和学生类 `Student`。`Student` 使用**公有继承**复用 `People` 的姓名功能，并在不同头文件中分别声明两个类。

姓名和学号均应为私有数据成员；测试代码通过设置器为对象赋值，再调用输出函数验证结果。

## 相关知识

### 公有继承

公有继承表达“是一种”（is-a）关系。学生是一种人员，因此可以使用如下语法：

``` cpp
class Student : public People {
    // Student 保留 People 的公有成员接口
};
```

基类的 `public` 成员在派生类中仍保持 `public` 访问性。派生类的成员函数和派生类对象都能调用基类的公有成员函数。

### 私有数据与访问器、设置器

数据成员设为 `private` 可防止类外代码直接修改对象状态。类通过公有设置器（setter）和访问器（getter）提供受控访问：

``` cpp
class People {
private:
    std::string name;

public:
    void setName(const std::string& name);
    const std::string& getName() const;
};
```

`const` 访问器只读取对象状态，不会修改数据成员。

### 多文件组织

本题将两个类分别声明在不同头文件中：

``` text
People.h     // People 类声明
Student.h    // 包含 People.h，并声明 Student 类
People.cpp   // People 成员函数实现
Student.cpp  // Student 成员函数实现
main.cpp     // 测试代码
```

`Student.h` 需要包含 `People.h`，因为 `Student` 的定义依赖其基类定义。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `People.h` 中声明 `People` 类。姓名成员必须为私有的 `std::string name`，并提供：

   ```cpp
       void setName(const std::string& name);
       const std::string& getName() const;
       void printName() const;
   ```

1. 在 `Student.h` 中包含 `People.h`，并声明 `class Student : public People`。学号成员必须为私有的 `int sid`，并提供：

   ```cpp
   void setSid(int sid);
   int getSid() const;
   void printSid() const;
   ```

1. 在 `People.cpp` 和 `Student.cpp` 中分别实现对应成员函数。
2. `printSid` 输出 `学号：学号值`；`printName` 输出 `姓名：姓名值`。
3. 在 `test` 函数中，仅使用 `setSid` 和继承得到的 `setName` 为对象赋值；使用 `assert` 验证访问器返回值和输出内容。

## 待完成代码

### People.h

``` cpp
#ifndef PEOPLE_H
#define PEOPLE_H

#include <string>

// TODO：声明 People 类，姓名成员必须为 private

#endif
```

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include "People.h"

// TODO：使用 public 继承声明 Student 类，学号成员必须为 private

#endif
```

### People.cpp

``` cpp
#include "People.h"

#include <iostream>

// TODO：定义 People 的成员函数
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iostream>

// TODO：定义 Student 的成员函数
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Student student;
    student.setSid(1);
    student.setName("张大民");

    assert(student.getSid() == 1);
    assert(student.getName() == "张大民");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student.printSid();
    student.printName();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "学号：1\n姓名：张大民\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

测试通过设置器将学生学号设置为 `1`、姓名设置为 `张大民`。预期输出为：

``` text
学号：1
姓名：张大民
```

可使用以下命令编译源文件：

``` bash
g++ -std=c++11 People.cpp Student.cpp main.cpp -o public_inheritance
```

---

开始你的任务吧，祝你成功！
