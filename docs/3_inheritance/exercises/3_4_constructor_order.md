# 练习4：观察基类与派生类的构造顺序

---

## 任务描述

设计基类 `People` 和派生类 `Student`，记录对象构造过程中基类构造函数和派生类构造函数的执行顺序。

## 相关知识

### 派生类对象的构造顺序

创建派生类对象时，必须先构造其基类子对象，再构造派生类自身部分。因此，构造顺序始终是：

``` text
基类构造函数 → 派生类构造函数
```

派生类构造函数体执行之前，基类部分已经完成初始化。

### 使用引用记录过程

本题将 `std::vector<std::string>&` 传给两个构造函数，用于记录调用顺序。引用使基类和派生类能操作同一个日志容器。

## 编程要求

1. 使用 C++11 标准编写单文件程序。
2. 声明 `People` 类，其构造函数接收日志引用，并向日志追加 `"构造 People"`。
3. 声明 `Student : public People`，其构造函数也接收日志引用；必须在初始化列表中调用 `People(log)`，并在自身构造函数体中追加 `"构造 Student"`。
4. 在 `test` 中创建一个 `Student` 对象，并使用 `assert` 验证日志顺序。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

class People {
public:
    People(std::vector<std::string>& log);
};

// TODO：实现构造函数，记录“构造 People”

class Student : public People {
public:
    Student(std::vector<std::string>& log);
};

// TODO：先构造 People，再记录“构造 Student”

void test() {
    std::vector<std::string> log;
    Student student(log);

    assert(log.size() == 2);
    assert(log[0] == "构造 People");
    assert(log[1] == "构造 Student");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    for (const std::string& message : log) {
        std::cout << message << '\n';
    }
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "构造 People\n构造 Student\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

预期输出说明基类先于派生类构造：

``` text
构造 People
构造 Student
```

---

开始你的任务吧，祝你成功！
