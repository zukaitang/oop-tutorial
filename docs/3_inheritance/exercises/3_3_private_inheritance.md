# 练习3：使用私有继承设计学生与研究生信息类

---

## 任务描述

设计 `People`、`Student` 和 `Graduate` 三个类，形成 `People → Student → Graduate` 的继承关系。其中 `Student` **私有继承** `People`，`Graduate` **公有继承** `Student`。

通过 `Graduate` 对象设置并输出姓名、学号和研究方向，理解私有继承如何限制基类接口在后续派生类和类外代码中的可见性。

## 相关知识

### 私有继承

私有继承的语法如下：

``` cpp
class Student : private People {
    // ...
};
```

在私有继承中，`People` 的 `public` 和 `protected` 成员在 `Student` 中都成为私有成员。`Student` 的成员函数可以访问它们，但 `Graduate` 和类外代码不能直接访问这些继承成员。

### 通过 Student 重新开放受控接口

若 `Graduate` 需要使用人员姓名功能，`Student` 必须提供自己的公有包装函数：

``` cpp
class Student : private People {
public:
    void setStudentName(const std::string& name) {
        People::setName(name);
    }
};
```

`Graduate` 虽不能直接访问 `People::setName`，但可继承并调用 `Student::setStudentName`。

### 多层继承与访问控制

本题的访问路径如下：

``` text
People（姓名）
  └─ private → Student（学号、姓名包装接口）
       └─ public → Graduate（研究方向）
```

所有数据成员保持私有；类外代码只能通过各层对外公开的设置器、访问器和输出函数访问状态。

## 编程要求

1. 使用 C++11 标准编写程序。将 `People`、`Student`、`Graduate` 分别声明在 `People.h`、`Student.h`、`Graduate.h` 中，并在对应 `.cpp` 文件中实现成员函数。
2. `People` 的私有成员为 `std::string name`，提供：

   ```cpp
   void setName(const std::string& name);
   const std::string& getName() const;
   void printName() const;
   ```

1. `Student` 使用 `private` 继承 `People`，私有成员为 `int sid`，提供：

   ```cpp
   void setSid(int sid);
   int getSid() const;
   void printSid() const;
   void setStudentName(const std::string& name);
   const std::string& getStudentName() const;
   void printStudentName() const;
   ```

1. `Graduate` 使用 `public` 继承 `Student`，私有成员为 `int researchId`，提供：

   ```cpp
   void setResearchId(int researchId);
   int getResearchId() const;
   void printResearchId() const;
   ```

1. 姓名包装函数由 `Student` 调用其私有继承得到的 `People` 接口完成。不得定义题干中的独立 `Set` 函数。
2. 在 `test` 中通过 `Graduate` 对象的设置器赋值，使用 `assert` 验证三个访问器和输出结果。

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

// TODO：使用 private 继承声明 Student 类

#endif
```

### Graduate.h

``` cpp
#ifndef GRADUATE_H
#define GRADUATE_H

#include "Student.h"

// TODO：使用 public 继承声明 Graduate 类

#endif
```

### People.cpp

``` cpp
#include "People.h"

// TODO：定义 People 的成员函数
```

### Student.cpp

``` cpp
#include "Student.h"

// TODO：定义 Student 的成员函数
```

### Graduate.cpp

``` cpp
#include "Graduate.h"

// TODO：定义 Graduate 的成员函数
```

### main.cpp

``` cpp
#include "Graduate.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Graduate graduate;
    graduate.setSid(1);
    graduate.setStudentName("厉宏富");
    graduate.setResearchId(304);

    assert(graduate.getSid() == 1);
    assert(graduate.getStudentName() == "厉宏富");
    assert(graduate.getResearchId() == 304);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    graduate.printSid();
    graduate.printStudentName();
    graduate.printResearchId();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "学号：1\n姓名：厉宏富\n研究方向：304\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

设置学号 `1`、姓名 `厉宏富`、研究方向 `304` 后，预期输出为：

``` text
学号：1
姓名：厉宏富
研究方向：304
```

可使用以下命令编译：

``` bash
g++ -std=c++11 People.cpp Student.cpp Graduate.cpp main.cpp -o private_inheritance
```

---

开始你的任务吧，祝你成功！
