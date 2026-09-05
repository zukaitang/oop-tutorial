# 练习9：使用虚基类解决菱形继承

---

## 任务描述

设计 `People`、`Student`、`Employee` 和 `GraduateAssistant` 类，使用虚基类解决 `People → Student / Employee → GraduateAssistant` 的菱形继承问题。

## 相关知识

### 虚基类

若 `Student` 和 `Employee` 都继承 `People`，再由 `GraduateAssistant` 同时继承两者，普通继承会使最终对象包含两份 `People` 子对象。使用虚继承可确保只保留一份：

``` cpp
class Student : virtual public People {};
class Employee : virtual public People {};
```

最派生类 `GraduateAssistant` 负责初始化虚基类 `People`。

## 编程要求

1. `People` 私有保存姓名，提供带参构造函数和 `getName()`。
2. `Student`、`Employee` 均使用 `virtual public People` 继承，分别保存学号和工号。
3. `GraduateAssistant` 同时继承 `Student` 与 `Employee`，构造函数必须在初始化列表中显式初始化 `People(name)`。
4. 实现 `printProfile()` 输出姓名、学号、工号；在 `test` 中验证输出。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

class People { /* TODO */ };
class Student : virtual public People { /* TODO */ };
class Employee : virtual public People { /* TODO */ };

class GraduateAssistant : public Student, public Employee {
public:
    // TODO：初始化 People、Student、Employee
    GraduateAssistant(const std::string& name, int studentId, int employeeId) {
        /* TODO */
    }
    void printProfile() const {
        /* TODO */
    }
};

void test() {
    GraduateAssistant assistant("李红霞", 1, 9001);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    assistant.printProfile();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "姓名：李红霞\n学号：1\n工号：9001\n");
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
姓名：李红霞
学号：1
工号：9001
```

---

开始你的任务吧，祝你成功！
