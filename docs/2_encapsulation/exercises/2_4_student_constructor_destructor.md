# 练习：学生类的构造函数和析构函数

---

## 任务描述

设计一个学生类 `Student`，其中包含学号和姓名两个数据成员，并实现无参构造函数、带参构造函数和析构函数。

程序创建多个学生对象。对象离开其作用域时，析构函数按对象创建的相反顺序执行，并输出“学号 姓名 退出程序”。

## 相关知识

### 构造函数

构造函数是对象创建时自动调用的特殊成员函数。它与类同名、没有返回类型，可以重载：同一个类可以定义多个参数列表不同的构造函数。

``` cpp
class Student {
public:
    Student();
    Student(int sid, const std::string& name);
};
```

创建对象时，编译器会根据实参选择匹配的构造函数：

``` cpp
Student first;                 // 调用无参构造函数
Student second(1, "李红霞");  // 调用带参构造函数
```

### 成员初始化列表

构造函数可使用成员初始化列表初始化数据成员。成员初始化列表位于参数列表之后，以冒号 `:` 开始：

``` cpp
Student::Student(int sid, const std::string& name)
    : SID(sid), Name(name) {
}
```

数据成员的实际初始化顺序由它们在类中的声明顺序决定，而不是由初始化列表中的书写顺序决定。因此，初始化列表的顺序应与成员声明顺序保持一致。

### 析构函数与销毁顺序

析构函数用于对象销毁时的清理工作。析构函数以 `~` 开头、与类同名，不能带参数或返回值：

``` cpp
Student::~Student() {
    // 清理工作
}
```

同一作用域中，局部对象会按照“后创建、先销毁”的逆序析构。例如，先后创建 `first`、`second`、`third`，离开作用域时析构顺序为 `third`、`second`、`first`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Student.h` 中声明类 `Student`。`SID` 和 `Name` 均为公有数据成员：

   ```cpp
   int SID;
   std::string Name;
   ```

1. 在 `Student.h` 中声明以下公有成员函数：

   ```cpp
   Student();
   Student(int sid, const std::string& name);
   ~Student();
   ```

1. 在 `Student.cpp` 中定义成员函数：

2. 无参构造函数将 `SID` 初始化为 `0`，将 `Name` 初始化为 `"王小明"`；
3. 带参构造函数将 `sid` 和 `name` 分别初始化给 `SID` 和 `Name`；
4. 析构函数按以下格式输出一行：`学号 姓名 退出程序`。

5. 在 `main.cpp` 的 `test` 函数中创建一个无参对象和三个带参对象，并使用 `assert` 检查各对象成员的初始化结果。
6. 将四个对象放入同一个局部作用域。使用 `std::ostringstream` 捕获该作用域结束时的析构输出，并使用 `assert` 验证销毁顺序和输出内容。
7. 在 `main` 函数中调用 `test`。

## 待完成代码

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

// TODO：声明 Student 类

#endif
```

### Student.cpp

``` cpp
#include "Student.h"
#include <iostream>

// TODO：定义两个构造函数和析构函数
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());

    {
        Student first(1, "李红霞");
        Student second(2, "张如雪");
        Student third(3, "刘俊民");
        Student defaultStudent;

        // TODO：使用 assert 检查四个对象的成员值
    }  // 四个对象在此处按逆序析构

    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "0 王小明 退出程序\n"
           "3 刘俊民 退出程序\n"
           "2 张如雪 退出程序\n"
           "1 李红霞 退出程序\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

测试在同一局部作用域中按以下顺序创建对象：`first`、`second`、`third`、`defaultStudent`。因此作用域结束时，析构函数的输出顺序必须相反：

``` text
0 王小明 退出程序
3 刘俊民 退出程序
2 张如雪 退出程序
1 李红霞 退出程序
```

测试同时验证无参构造函数和带参构造函数的初始化结果。可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Student.cpp main.cpp -o student
```

---

开始你的任务吧，祝你成功！
