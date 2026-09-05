# 练习6：观察基类与派生类的析构顺序

---

## 任务描述

设计基类 `People` 和派生类 `Student`，记录对象创建和离开作用域时构造函数、析构函数的调用顺序。

## 相关知识

### 派生类对象的析构顺序

派生类对象销毁时，析构过程与构造过程相反：先执行派生类析构函数，再执行基类析构函数。

``` text
构造：People → Student
析构：Student → People
```

这样可确保派生类先清理自身资源，随后再清理基类资源。

### 作用域与自动析构

局部对象离开其所在的花括号作用域时会自动析构。本题使用内部作用域，确保在断言前完成析构。

## 编程要求

1. 使用 C++11 标准编写单文件程序。
2. `People` 的构造函数和析构函数分别向日志追加 `"构造 People"`、`"析构 People"`。
3. `Student : public People` 的构造函数和析构函数分别追加 `"构造 Student"`、`"析构 Student"`。
4. 两个类都通过构造函数接收同一个日志引用。
5. 在 `test` 的内部作用域中创建 `Student` 对象；作用域结束后用 `assert` 验证完整顺序。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

class People {
private:
    std::vector<std::string>& log;

public:
    explicit People(std::vector<std::string>& log);
    ~People();
};

class Student : public People {
private:
    std::vector<std::string>& log;

public:
    // TODO：构造时先初始化 People(log)
    explicit Student(std::vector<std::string>& log);
    ~Student();
};

void test() {
    std::vector<std::string> log;
    {
        Student student(log);
    }

    assert(log.size() == 4);
    assert(log[0] == "构造 People");
    assert(log[1] == "构造 Student");
    assert(log[2] == "析构 Student");
    assert(log[3] == "析构 People");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    for (const std::string& message : log) {
        std::cout << message << '\n';
    }
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "构造 People\n构造 Student\n析构 Student\n析构 People\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

预期输出为：

``` text
构造 People
构造 Student
析构 Student
析构 People
```

---

开始你的任务吧，祝你成功！
