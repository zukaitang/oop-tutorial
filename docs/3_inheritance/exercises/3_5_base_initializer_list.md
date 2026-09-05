# 练习5：在初始化列表中调用基类构造函数

---

## 任务描述

设计人员类 `People` 和学生类 `Student`。`People` 只提供带姓名参数的构造函数，`Student` 必须在初始化列表中显式调用该基类构造函数。

## 相关知识

### 基类没有默认构造函数

若基类没有无参构造函数，派生类不能省略基类初始化。下面的写法会导致编译错误：

``` cpp
class Student : public People {
public:
    Student(const std::string& name, int sid) {
        // 错误：编译器会尝试调用 People()
    }
};
```

### 派生类初始化列表

应在派生类构造函数初始化列表中调用匹配的基类构造函数：

``` cpp
Student::Student(const std::string& name, int sid)
    : People(name), sid(sid) {
}
```

## 编程要求

1. 使用 C++11 标准编写单文件程序。
2. `People` 的姓名成员为私有 `std::string name`；只提供构造函数 `People(const std::string& name)` 和访问器 `getName()`。
3. `Student` 公有继承 `People`，私有成员为 `int sid`，构造函数为：

   ```cpp
   Student(const std::string& name, int sid);
   ```

1. 在 `Student` 的初始化列表中调用 `People(name)` 并初始化 `sid`。
2. 提供 `getSid()` 和 `print()`，在测试中断言成员值和输出。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

class People {
private:
    std::string name;

public:
    explicit People(const std::string& name);
    const std::string& getName() const;
};

class Student : public People {
private:
    int sid;

public:
    // TODO：通过初始化列表调用 People(name)
    Student(const std::string& name, int sid);
    int getSid() const;
    void print() const;
};

void test() {
    Student student("李红霞", 1);
    assert(student.getName() == "李红霞");
    assert(student.getSid() == 1);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student.print();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "姓名：李红霞\n学号：1\n");
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
```

---

开始你的任务吧，祝你成功！
