# 练习7：使用多继承组合学生与员工信息

---

## 任务描述

设计 `Student`、`Employee` 和 `TeachingAssistant` 类。`TeachingAssistant` 公有继承前两个基类，同时拥有学号和工号。

## 相关知识

### 多继承

C++ 类可以从多个基类派生：

``` cpp
class TeachingAssistant : public Student, public Employee {
};
```

派生类对象包含每个基类的子对象，因此可复用两个基类各自提供的公有接口。

## 编程要求

1. 使用 C++11 标准编写单文件程序。
2. `Student` 私有保存 `studentId`，提供 `setStudentId`、`getStudentId`。
3. `Employee` 私有保存 `employeeId`，提供 `setEmployeeId`、`getEmployeeId`。
4. `TeachingAssistant` 公有继承 `Student` 和 `Employee`，并实现 `printProfile()`，按“学号、工号”两行输出。
5. 在 `test` 中通过继承的设置器赋值，并使用 `assert` 验证访问器与输出。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>

class Student {
    // TODO
};

class Employee {
    // TODO
};

class TeachingAssistant : public Student, public Employee {
public:
    void printProfile() const;
};

void test() {
    TeachingAssistant assistant;
    assistant.setStudentId(2026001);
    assistant.setEmployeeId(9001);

    assert(assistant.getStudentId() == 2026001);
    assert(assistant.getEmployeeId() == 9001);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    assistant.printProfile();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "学号：2026001\n工号：9001\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

预期输出：

``` text
学号：2026001
工号：9001
```

---

开始你的任务吧，祝你成功！
