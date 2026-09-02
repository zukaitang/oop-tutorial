# 练习11：使用对象数组管理学生信息

---

## 任务描述

设计 `Student` 类，并使用最多容纳 5 条记录的全局学生对象数组管理学生信息。实现添加学生记录、输出全部记录和计算平均成绩三个普通函数。

本题将综合练习带参构造函数的调用、对象数组的初始化，以及通过对象数组访问对象成员。

## 相关知识

### 带参构造函数与对象创建

定义对象时传入实参，会自动调用与参数匹配的构造函数：

``` cpp
Student student(1, "李红霞", 96.0f);
```

构造函数不能像普通成员函数一样由对象直接调用；它只能在对象创建时自动执行。

### 对象数组

对象数组会在创建时为每个元素调用构造函数。若类没有无参构造函数，则需要为每个数组元素提供构造参数：

``` cpp
Student students[2] = {
    Student(0, "", 0.0f),
    Student(0, "", 0.0f)
};
```

本题的 `Student` 类只要求提供带参构造函数，因此全局数组的 5 个元素都应使用上述方式初始化。`studentCount` 记录当前已添加的学生数量，添加时将新记录存入 `students[studentCount]`。

### 对象数组元素的访问

数组下标运算符 `[]` 先选中对象，再用 `.` 访问该对象的公有成员：

``` cpp
students[index].SID;
students[index].Name;
students[index].Score;
```

### 平均成绩与输出精度

计算平均值时，应使用 `float` 或 `double` 保存结果，避免整数除法丢失小数部分。使用 `<iomanip>` 中的 `std::fixed` 和 `std::setprecision(4)` 可使平均成绩固定输出 4 位小数。

``` cpp
std::cout << std::fixed << std::setprecision(4) << average;
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Student.h` 中声明 `Student` 类。学号、姓名、分数均为公有数据成员：

   ```cpp
   int SID;
   std::string Name;
   float Score;
   ```

1. 在类中声明公有带参构造函数：

   ```cpp
   Student(int sid, const std::string& name, float sco);
   ```

1. 在 `Student.h` 中声明以下普通函数：

   ```cpp
   void Add(int sid, const std::string& name, float sco);
   void PrintAll();
   void Average();
   ```

1. 在 `Student.cpp` 中定义一个最多容纳 5 个元素的全局 `Student` 对象数组，以及记录当前学生数的计数变量。
2. 完成三个普通函数：

3. `Add`：在学生表末尾添加一条记录；
4. `PrintAll`：按“学号 姓名 成绩”的格式逐行输出已添加记录；
5. `Average`：计算已添加学生的平均成绩，并按“平均成绩 计算结果”的格式输出，保留 4 位小数。

6. 本题测试数据不超过 5 条记录。`test` 函数使用 `std::ostringstream` 捕获 `PrintAll` 与 `Average` 的输出，并用 `assert` 验证结果。
7. 在 `main` 函数中调用 `test`。

## 待完成代码

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

// TODO：声明 Student 类和三个普通函数

#endif
```

### Student.cpp

``` cpp
#include "Student.h"

// TODO：定义全局对象数组、构造函数和三个普通函数
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Add(0, "李红霞", 96.0f);
    Add(1, "张如雪", 85.0f);
    Add(2, "刘俊民", 76.0f);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    PrintAll();
    Average();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "0 李红霞 96\n"
           "1 张如雪 85\n"
           "2 刘俊民 76\n"
           "平均成绩 85.6667\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

按顺序添加以下三条记录：`(0, 李红霞, 96)`、`(1, 张如雪, 85)`、`(2, 刘俊民, 76)`。预期输出为：

``` text
0 李红霞 96
1 张如雪 85
2 刘俊民 76
平均成绩 85.6667
```

平均值的计算为 `(96 + 85 + 76) / 3 = 85.6667`（保留 4 位小数）。可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Student.cpp main.cpp -o student_array
```

---

开始你的任务吧，祝你成功！
