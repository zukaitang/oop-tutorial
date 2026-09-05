# 练习2：使用保护继承设计学生信息类

---

## 任务描述

设计人员类 `People` 和学生类 `Student`。`Student` 使用**保护继承**继承 `People`，并通过自身的公有成员函数对外提供姓名和学号的设置、读取与输出功能。

## 相关知识

### 保护继承

保护继承的语法如下：

``` cpp
class Student : protected People {
    // ...
};
```

在保护继承中，`People` 的 `public` 和 `protected` 成员在 `Student` 中均以 `protected` 身份存在。它们可被 `Student` 的成员函数访问，但不能由类外代码通过 `Student` 对象直接访问。

``` cpp
Student student;
// student.setName("张大民");  // 错误：People 的公有接口因保护继承而不可在类外访问
```

### 通过派生类包装基类接口

若希望对外开放部分基类功能，可在派生类中提供公有包装函数：

``` cpp
class Student : protected People {
public:
    void setStudentName(const std::string& name) {
        People::setName(name);
    }
};
```

这样，`Student` 控制了哪些基类功能可被外部使用。

### 多文件组织

本题将 `People` 和 `Student` 分别声明在 `People.h`、`Student.h` 中；相应成员函数实现在各自的 `.cpp` 文件中。`Student.h` 需要包含 `People.h`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `People.h` 中声明 `People` 类。姓名为私有成员 `std::string name`，并提供：

   ```cpp
   void setName(const std::string& name);
   const std::string& getName() const;
   void printName() const;
   ```

1. 在 `Student.h` 中使用 `protected` 方式继承 `People`。学号为私有成员 `int sid`。
2. `Student` 应提供下列公有成员函数：

   ```cpp
   void setSid(int sid);
   int getSid() const;
   void printSid() const;
   void setStudentName(const std::string& name);
   const std::string& getStudentName() const;
   void printStudentName() const;
   ```

1. 姓名相关的三个 `Student` 成员函数应分别调用保护继承得到的 `People::setName`、`People::getName`、`People::printName`。
2. 在 `test` 中通过 `Student` 自己的设置器赋值，并使用 `assert` 验证访问器与输出内容。不得直接通过 `Student` 对象调用 `People` 的成员函数。

## 待完成代码

### People.h

``` cpp
#ifndef PEOPLE_H
#define PEOPLE_H

#include <string>

// TODO：声明 People 类

#endif
```

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include "People.h"

// TODO：使用 protected 继承声明 Student 类

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
    student.setStudentName("张大民");

    assert(student.getSid() == 1);
    assert(student.getStudentName() == "张大民");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student.printSid();
    student.printStudentName();
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

测试通过 `Student` 的包装函数设置姓名，预期输出为：

``` text
学号：1
姓名：张大民
```

可使用以下命令编译：

``` bash
g++ -std=c++11 People.cpp Student.cpp main.cpp -o protected_inheritance
```

---

开始你的任务吧，祝你成功！
