# 练习：由最派生类初始化虚基类

---

## 任务描述

设计 `Person`、`Student`、`Employee` 和 `TeachingAssistant` 四个类。`Student` 与 `Employee` 虚继承 `Person`，`TeachingAssistant` 同时继承两者，并负责初始化唯一的 `Person` 虚基类子对象。

## 相关知识

### 最派生类初始化虚基类

虚基类由最派生类负责构造。即使中间类的初始化列表中写有 `Person(...)`，创建 `TeachingAssistant` 对象时，实际生效的是 `TeachingAssistant` 对虚基类的初始化：

```cpp
class Student : virtual public Person { /* ... */ };
class Employee : virtual public Person { /* ... */ };

class TeachingAssistant : public Student, public Employee {
public:
    TeachingAssistant(const std::string& name, int sid, int eid)
        : Person(name), Student(sid), Employee(eid) {
    }
};
```

### 虚基类只构造一次

虚继承确保 `TeachingAssistant` 内只有一个 `Person` 子对象，因此 `Person` 构造函数只调用一次。

## 编程要求

1. 使用 C++11 标准编写单文件程序。
2. `Person` 构造函数接收姓名和日志引用，保存姓名并记录 `"构造 Person：姓名"`。
3. `Student`、`Employee` 均使用 `virtual public Person` 继承，分别记录 `"构造 Student"`、`"构造 Employee"`。
4. `TeachingAssistant` 同时继承 `Student`、`Employee`。其构造函数必须在初始化列表中显式调用 `Person(name)`，并记录 `"构造 TeachingAssistant"`。
5. 在 `test` 中创建 `TeachingAssistant("李红霞", 1, 9001)`，验证日志中 `Person` 只出现一次，且构造顺序正确。

## 待完成代码

```cpp
#include <cassert>
#include <string>
#include <vector>

class Person {
public:
    // TODO
};

class Student : virtual public Person {
public:
    // TODO
};

class Employee : virtual public Person {
public:
    // TODO
};

class TeachingAssistant : public Student, public Employee {
public:
    // TODO：由最派生类初始化 Person(name)
    TeachingAssistant(const std::string& name, int studentId, int employeeId,
                     std::vector<std::string>& log);
};

void test() {
    std::vector<std::string> log;
    TeachingAssistant assistant("李红霞", 1, 9001, log);

    assert(log.size() == 4);
    assert(log[0] == "构造 Person：李红霞");
    assert(log[1] == "构造 Student");
    assert(log[2] == "构造 Employee");
    assert(log[3] == "构造 TeachingAssistant");
}

int main() {
    test();
    return 0;
}
```

## 测试说明

日志中只能出现一次 `构造 Person：李红霞`。预期构造顺序为：

```text
构造 Person：李红霞
构造 Student
构造 Employee
构造 TeachingAssistant
```

---

开始你的任务吧，祝你成功！
