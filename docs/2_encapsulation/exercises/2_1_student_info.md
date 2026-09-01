# 练习：声明并定义学生信息类

---

## 任务描述

声明并定义一个学生信息类 `StInfo`，用于保存并打印一名学生的基本信息。

类中包含学号、姓名、班级和手机号四项数据；通过 `SetInfo` 为这些成员赋值，并通过 `PrintInfo` 按指定格式输出学生信息。

## 相关知识

### 类的声明

类将数据成员和操作这些数据的成员函数组织在一起，是一种用户自定义的数据类型。类的基本声明形式如下：

``` cpp
class 类名 {
public:
    数据类型 成员变量;
    返回类型 成员函数(参数列表);
private:
    数据类型 私有成员变量;
};  // 类定义末尾必须有分号
```

`public` 表示其后的成员可以在类外访问；`private` 表示其后的成员只能在类的成员函数内部访问。本题为了练习类的基本声明，要求所有成员均为 `public`。

### 类成员函数的定义

成员函数可以在类外定义。定义时，需要在函数名前加上“`类名::`”，其中 `::` 是作用域运算符，用于说明该函数属于哪个类。

``` cpp
class Counter {
public:
    int value;
    void setValue(int number);
};

void Counter::setValue(int number) {
    value = number;
}
```

### 头文件与源文件

通常将类的声明放在头文件（`.h`）中，将成员函数的实现放在源文件（`.cpp`）中。需要使用该类的源文件通过 `#include` 引入头文件；实现文件同样需要包含头文件，以保证声明和定义一致。

``` cpp
// Counter.h：类的声明
class Counter {
public:
    void setValue(int number);
};

// Counter.cpp：成员函数的实现
#include "Counter.h"

void Counter::setValue(int number) {
    // ...
}
```

### 字符串指针成员

本题使用 `char*` 保存姓名、班级和手机号。`SetInfo` 中只需保存传入的字符数组地址，不需要复制字符串内容；因此调用者提供的字符数组在 `StInfo` 对象使用期间必须保持有效。

``` cpp
char name[] = "小郭";
char* namePtr = name;
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在文件 `StInfo.h`中声明类 `StInfo`，其包含以下四个私有数据成员和两个公有成员函数：

   ```cpp
   int SID;
   char* Name;
   char* Class;
   char* Phone;

   void SetInfo(int sid, char* name, char* cla, char* phone);
   void PrintInfo();
   ```

1. 在文件 `StInfo.cpp`中包含 `StInfo.h`，并在该文件中使用类名前缀 `StInfo::` 分别定义 `SetInfo` 和 `PrintInfo`。

2. `SetInfo` 应将四个参数分别赋给对应的数据成员。
3. `PrintInfo` 必须按以下格式输出，每项占一行，标签后的中文冒号为全角字符 `：`：

   ```text
   学号：1
   姓名：小郭
   班级：计科1班
   手机号：12312340000
   ```

1. 在 `main.cpp` 的 `test` 函数中创建两个 `StInfo` 对象。分别调用 `PrintInfo` 时，使用 `std::ostringstream` 捕获 `std::cout` 的输出，并使用 `assert` 断言输出字符串与预期结果完全一致。
2. 在 `main` 函数中调用 `test`。

## 待完成代码

### StInfo.h

``` cpp
#ifndef ST_INFO_H
#define ST_INFO_H

// TODO：声明 StInfo 类

#endif
```

### StInfo.cpp

``` cpp
#include "StInfo.h"
#include <iostream>

// TODO：在类外定义 StInfo 的成员函数
```

### main.cpp

``` cpp
#include "StInfo.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    char name1[] = "小郭";
    char class1[] = "计科1班";
    char phone1[] = "12312340000";

    StInfo student1;
    student1.SetInfo(1, name1, class1, phone1);

    std::ostringstream output1;
    std::streambuf* oldBuffer = std::cout.rdbuf(output1.rdbuf());
    student1.PrintInfo();
    std::cout.rdbuf(oldBuffer);

    assert(output1.str() ==
           "学号：1\n姓名：小郭\n班级：计科1班\n手机号：12312340000\n");

    char name2[] = "小王";
    char class2[] = "计科2班";
    char phone2[] = "12312340001";

    StInfo student2;
    student2.SetInfo(2, name2, class2, phone2);

    std::ostringstream output2;
    oldBuffer = std::cout.rdbuf(output2.rdbuf());
    student2.PrintInfo();
    std::cout.rdbuf(oldBuffer);

    assert(output2.str() ==
           "学号：2\n姓名：小王\n班级：计科2班\n手机号：12312340001\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

使用 `std::ostringstream` 可以将本应写入标准输出的内容保存为字符串，从而使用 `assert` 对 `PrintInfo` 的输出进行精确验证。测试中的预期字符串如下：

``` text
学号：1
姓名：小郭
班级：计科1班
手机号：12312340000
学号：2
姓名：小王
班级：计科2班
手机号：12312340001
```

测试时重点检查以下内容：

| 测试项       | 预期结果                                                                |
| ------------ | ----------------------------------------------------------------------- |
| 成员赋值     | `SID`、`Name`、`Class`、`Phone` 分别保存传入的四项信息                  |
| 文件组织     | 类声明位于 `StInfo.h`，成员函数实现在 `StInfo.cpp`                      |
| 成员函数定义 | 类外定义时正确使用 `StInfo::` 前缀                                      |
| 输出格式     | 每次调用 `PrintInfo` 后，捕获的输出均通过 `assert` 与预期字符串完全一致 |

可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 StInfo.cpp main.cpp -o student_info
```

---

开始你的任务吧，祝你成功！
