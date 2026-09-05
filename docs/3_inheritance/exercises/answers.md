# 第 3 章练习参考答案

## 3_1_public_inheritance

### People.h

``` cpp
#ifndef PEOPLE_H
#define PEOPLE_H

#include <string>

class People {
private:
    std::string name;

public:
    void setName(const std::string& name);
    const std::string& getName() const;
    void printName() const;
};

#endif
```

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include "People.h"

class Student : public People {
private:
    int sid;

public:
    void setSid(int sid);
    int getSid() const;
    void printSid() const;
};

#endif
```

### People.cpp

``` cpp
#include "People.h"

#include <iostream>

void People::setName(const std::string& name) {
    this->name = name;
}

const std::string& People::getName() const {
    return name;
}

void People::printName() const {
    std::cout << "姓名：" << name << '\n';
}
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iostream>

void Student::setSid(int sid) {
    this->sid = sid;
}

int Student::getSid() const {
    return sid;
}

void Student::printSid() const {
    std::cout << "学号：" << sid << '\n';
}
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Student student;
    student.setSid(1);
    student.setName("张大民");

    assert(student.getSid() == 1);
    assert(student.getName() == "张大民");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student.printSid();
    student.printName();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "学号：1\n姓名：张大民\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 3_2_protected_inheritance

### People.h

``` cpp
#ifndef PEOPLE_H
#define PEOPLE_H

#include <string>

class People {
private:
    std::string name;

public:
    void setName(const std::string& name);
    const std::string& getName() const;
    void printName() const;
};

#endif
```

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include "People.h"

class Student : protected People {
private:
    int sid;

public:
    void setSid(int sid);
    int getSid() const;
    void printSid() const;
    void setStudentName(const std::string& name);
    const std::string& getStudentName() const;
    void printStudentName() const;
};

#endif
```

### People.cpp

``` cpp
#include "People.h"

#include <iostream>

void People::setName(const std::string& name) {
    this->name = name;
}

const std::string& People::getName() const {
    return name;
}

void People::printName() const {
    std::cout << "姓名：" << name << '\n';
}
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iostream>

void Student::setSid(int sid) {
    this->sid = sid;
}

int Student::getSid() const {
    return sid;
}

void Student::printSid() const {
    std::cout << "学号：" << sid << '\n';
}

void Student::setStudentName(const std::string& name) {
    People::setName(name);
}

const std::string& Student::getStudentName() const {
    return People::getName();
}

void Student::printStudentName() const {
    People::printName();
}
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Student student;
    student.setSid(1);
    student.setStudentName("张大民");

    assert(student.getSid() == 1);
    assert(student.getStudentName() == "张大民");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student.printSid();
    student.printStudentName();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() == "学号：1\n姓名：张大民\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 3_4_constructor_order

```cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

class People {
public:
    People(std::vector<std::string>& log) {
        log.push_back("构造 People");
    }
};

class Student : public People {
public:
    Student(std::vector<std::string>& log) : People(log) {
        log.push_back("构造 Student");
    }
};

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

## 3_5_base_initializer_list

```cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

class People {
private:
    std::string name;

public:
    explicit People(const std::string& name) : name(name) {}

    const std::string& getName() const {
        return name;
    }
};

class Student : public People {
private:
    int sid;

public:
    Student(const std::string& name, int sid) : People(name), sid(sid) {}

    int getSid() const {
        return sid;
    }

    void print() const {
        std::cout << "姓名：" << getName() << '\n';
        std::cout << "学号：" << sid << '\n';
    }
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

## 3_6_destructor_order

```cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

class People {
private:
    std::vector<std::string>& log;

public:
    explicit People(std::vector<std::string>& log) : log(log) {
        log.push_back("构造 People");
    }

    ~People() {
        log.push_back("析构 People");
    }
};

class Student : public People {
private:
    std::vector<std::string>& log;

public:
    explicit Student(std::vector<std::string>& log) : People(log), log(log) {
        log.push_back("构造 Student");
    }

    ~Student() {
        log.push_back("析构 Student");
    }
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

## 3_3_private_inheritance

### People.h

``` cpp
#ifndef PEOPLE_H
#define PEOPLE_H

#include <string>

class People {
private:
    std::string name;

public:
    void setName(const std::string& name);
    const std::string& getName() const;
    void printName() const;
};

#endif
```

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include "People.h"

class Student : private People {
private:
    int sid;

public:
    void setSid(int sid);
    int getSid() const;
    void printSid() const;
    void setStudentName(const std::string& name);
    const std::string& getStudentName() const;
    void printStudentName() const;
};

#endif
```

### Graduate.h

``` cpp
#ifndef GRADUATE_H
#define GRADUATE_H

#include "Student.h"

class Graduate : public Student {
private:
    int researchId;

public:
    void setResearchId(int researchId);
    int getResearchId() const;
    void printResearchId() const;
};

#endif
```

### People.cpp

``` cpp
#include "People.h"

#include <iostream>

void People::setName(const std::string& name) {
    this->name = name;
}

const std::string& People::getName() const {
    return name;
}

void People::printName() const {
    std::cout << "姓名：" << name << '\n';
}
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iostream>

void Student::setSid(int sid) {
    this->sid = sid;
}

int Student::getSid() const {
    return sid;
}

void Student::printSid() const {
    std::cout << "学号：" << sid << '\n';
}

void Student::setStudentName(const std::string& name) {
    People::setName(name);
}

const std::string& Student::getStudentName() const {
    return People::getName();
}

void Student::printStudentName() const {
    People::printName();
}
```

### Graduate.cpp

``` cpp
#include "Graduate.h"

#include <iostream>

void Graduate::setResearchId(int researchId) {
    this->researchId = researchId;
}

int Graduate::getResearchId() const {
    return researchId;
}

void Graduate::printResearchId() const {
    std::cout << "研究方向：" << researchId << '\n';
}
```

### main.cpp

```cpp
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

## 3_7_multiple_inheritance_profile

```cpp
class Student {
private:
    int studentId;
public:
    void setStudentId(int value) { studentId = value; }
    int getStudentId() const { return studentId; }
};

class Employee {
private:
    int employeeId;
public:
    void setEmployeeId(int value) { employeeId = value; }
    int getEmployeeId() const { return employeeId; }
};

class TeachingAssistant : public Student, public Employee {
public:
    void printProfile() const {
        std::cout << "学号：" << getStudentId() << '\n';
        std::cout << "工号：" << getEmployeeId() << '\n';
    }
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
```

## 3_8_ambiguity_resolution

```cpp
class Painter {
private:
    int level;
public:
    void setLevel(int value) { level = value; }
    int getLevel() const { return level; }
};

class Musician {
private:
    int level;
public:
    void setLevel(int value) { level = value; }
    int getLevel() const { return level; }
};

class Artist : public Painter, public Musician {
public:
    void setLevels(int paintingLevel, int musicLevel) {
        Painter::setLevel(paintingLevel);
        Musician::setLevel(musicLevel);
    }
    void printLevels() const {
        std::cout << "绘画等级：" << Painter::getLevel() << '\n';
        std::cout << "音乐等级：" << Musician::getLevel() << '\n';
    }
};

void test() {
    Artist artist;
    artist.setLevels(3, 5);
    assert(artist.Painter::getLevel() == 3);
    assert(artist.Musician::getLevel() == 5);
    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    artist.printLevels();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "绘画等级：3\n音乐等级：5\n");
}
```

## 3_9_virtual_base_class

```cpp
class People {
private:
    std::string name;
public:
    explicit People(const std::string& name) : name(name) {}
    const std::string& getName() const { return name; }
};

class Student : virtual public People {
private:
    int studentId;
public:
    Student(int studentId) : People(""), studentId(studentId) {}
    int getStudentId() const { return studentId; }
};

class Employee : virtual public People {
private:
    int employeeId;
public:
    Employee(int employeeId) : People(""), employeeId(employeeId) {}
    int getEmployeeId() const { return employeeId; }
};

class GraduateAssistant : public Student, public Employee {
public:
    GraduateAssistant(const std::string& name, int studentId, int employeeId)
        : People(name), Student(studentId), Employee(employeeId) {}

    void printProfile() const {
        std::cout << "姓名：" << getName() << '\n';
        std::cout << "学号：" << getStudentId() << '\n';
        std::cout << "工号：" << getEmployeeId() << '\n';
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
```

## 3_10_virtual_base_constructor

```cpp
#include <cassert>
#include <string>
#include <vector>

class Person {
protected:
    Person(const std::string& name, std::vector<std::string>& log) {
        log.push_back("构造 Person：" + name);
    }
};

class Student : virtual public Person {
public:
    Student(int studentId, std::vector<std::string>& log)
        : Person("", log) {
        (void)studentId;
        log.push_back("构造 Student");
    }
};

class Employee : virtual public Person {
public:
    Employee(int employeeId, std::vector<std::string>& log)
        : Person("", log) {
        (void)employeeId;
        log.push_back("构造 Employee");
    }
};

class TeachingAssistant : public Student, public Employee {
public:
    TeachingAssistant(const std::string& name, int studentId, int employeeId,
                     std::vector<std::string>& log)
        : Person(name, log), Student(studentId, log), Employee(employeeId, log) {
        log.push_back("构造 TeachingAssistant");
    }
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
