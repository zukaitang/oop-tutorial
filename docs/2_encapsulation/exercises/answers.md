# 第 2 章练习参考答案

## 2_1_student_info

### StInfo.h

``` cpp
#ifndef ST_INFO_H
#define ST_INFO_H

class StInfo {
private:
    int SID;
    char* Name;
    char* Class;
    char* Phone;

public:
    void SetInfo(int sid, char* name, char* cla, char* phone);
    void PrintInfo();
};

#endif
```

### StInfo.cpp

``` cpp
#include "StInfo.h"
#include <iostream>

void StInfo::SetInfo(int sid, char* name, char* cla, char* phone) {
    SID = sid;
    Name = name;
    Class = cla;
    Phone = phone;
}

void StInfo::PrintInfo() {
    std::cout << "学号：" << SID << '\n';
    std::cout << "姓名：" << Name << '\n';
    std::cout << "班级：" << Class << '\n';
    std::cout << "手机号：" << Phone << '\n';
}
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
    return 0;
}
```

## 2_2_rectangle

### Rectangle.h

``` cpp
#ifndef RECTANGLE_H
#define RECTANGLE_H

class Rectangle {
private:
    int height;
    int width;

public:
    void Set(int h, int w);
    int GetArea();
};

Rectangle GetRect(int h, int w);
int GetRectArea(Rectangle rect);

#endif
```

### Rectangle.cpp

``` cpp
#include "Rectangle.h"

void Rectangle::Set(int h, int w) {
    height = h;
    width = w;
}

int Rectangle::GetArea() {
    return height * width;
}

Rectangle GetRect(int h, int w) {
    Rectangle rect;
    rect.Set(h, w);
    return rect;
}

int GetRectArea(Rectangle rect) {
    return rect.GetArea();
}
```

### main.cpp

``` cpp
#include "Rectangle.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Rectangle rect1 = GetRect(10, 15);
    assert(GetRectArea(rect1) == 150);

    std::ostringstream output1;
    std::streambuf* oldBuffer = std::cout.rdbuf(output1.rdbuf());
    std::cout << "长方形的面积为：" << GetRectArea(rect1) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output1.str() == "长方形的面积为：150\n");

    Rectangle rect2 = GetRect(100, 100);
    assert(GetRectArea(rect2) == 10000);

    std::ostringstream output2;
    oldBuffer = std::cout.rdbuf(output2.rdbuf());
    std::cout << "长方形的面积为：" << GetRectArea(rect2) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output2.str() == "长方形的面积为：10000\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_3_car

### Car.h

``` cpp
#ifndef CAR_H
#define CAR_H

class Car {
private:
    bool doorOn;
    bool lightOn;
    int speed;

public:
    void init();
    void OpenDoor();
    void CloseDoor();
    void OpenLight();
    void CloseLight();
    void Accelerate();
    void Decelerate();
    void ExecuteCommand(char command);
    void PrintStatus();
};

#endif
```

### Car.cpp

``` cpp
#include "Car.h"
#include <iostream>

void Car::init() {
    doorOn = false;
    lightOn = false;
    speed = 0;
}

void Car::OpenDoor() {
    doorOn = true;
}

void Car::CloseDoor() {
    doorOn = false;
}

void Car::OpenLight() {
    lightOn = true;
}

void Car::CloseLight() {
    lightOn = false;
}

void Car::Accelerate() {
    speed += 10;
}

void Car::Decelerate() {
    speed -= 10;
}

void Car::ExecuteCommand(char command) {
    switch (command) {
    case '1':
        OpenDoor();
        break;
    case '2':
        CloseDoor();
        break;
    case '3':
        OpenLight();
        break;
    case '4':
        CloseLight();
        break;
    case '5':
        Accelerate();
        break;
    case '6':
        Decelerate();
        break;
    }
}

void Car::PrintStatus() {
    std::cout << "车门 " << (doorOn ? "ON" : "OFF") << '\n';
    std::cout << "车灯 " << (lightOn ? "ON" : "OFF") << '\n';
    std::cout << "速度 " << speed << '\n';
}
```

### main.cpp

``` cpp
#include "Car.h"

#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

void executeCommands(Car& car, const std::string& commands) {
    for (char command : commands) {
        car.ExecuteCommand(command);
    }
}

void test() {
    Car car1;
    car1.init();
    executeCommands(car1, "135");

    std::ostringstream output1;
    std::streambuf* oldBuffer = std::cout.rdbuf(output1.rdbuf());
    car1.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(output1.str() == "车门 ON\n车灯 ON\n速度 10\n");

    Car car2;
    car2.init();
    executeCommands(car2, "135562");

    std::ostringstream output2;
    oldBuffer = std::cout.rdbuf(output2.rdbuf());
    car2.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(output2.str() == "车门 OFF\n车灯 ON\n速度 10\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_4_student_constructor_destructor

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

class Student {
public:
    int SID;
    std::string Name;

    Student();
    Student(int sid, const std::string& name);
    ~Student();
};

#endif
```

### Student.cpp

``` cpp
#include "Student.h"
#include <iostream>

Student::Student() : SID(0), Name("王小明") {}

Student::Student(int sid, const std::string& name) : SID(sid), Name(name) {}

Student::~Student() {
    std::cout << SID << ' ' << Name << " 退出程序\n";
}
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

        assert(first.SID == 1);
        assert(first.Name == "李红霞");
        assert(second.SID == 2);
        assert(second.Name == "张如雪");
        assert(third.SID == 3);
        assert(third.Name == "刘俊民");
        assert(defaultStudent.SID == 0);
        assert(defaultStudent.Name == "王小明");
    }

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

## 2_5_student_birthday

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

class Date {
private:
    int year;
    int month;
    int day;

public:
    Date(int year, int month, int day);
    void Print() const;
};

class Student {
private:
    std::string name;
    Date birthday;

public:
    Student(const std::string& name, int year, int month, int day);
    void Print() const;
};

#endif
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iostream>

Date::Date(int year, int month, int day)
    : year(year), month(month), day(day) {}

void Date::Print() const {
    std::cout << year << '-' << month << '-' << day;
}

Student::Student(const std::string& name, int year, int month, int day)
    : name(name), birthday(year, month, day) {}

void Student::Print() const {
    std::cout << "姓名 " << name << " 生日 ";
    birthday.Print();
    std::cout << '\n';
}
```

### main.cpp

``` cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Student student1("李红霞", 2004, 5, 12);
    Student student2("张如雪", 2003, 11, 8);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    student1.Print();
    student2.Print();
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "姓名 李红霞 生日 2004-5-12\n"
           "姓名 张如雪 生日 2003-11-8\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_6_car_engine

### Car.h

``` cpp
#ifndef CAR_H
#define CAR_H

class Engine {
private:
    bool running;
    int rpm;

public:
    Engine();
    void Start();
    void Stop();
    void Accelerate();
    bool IsRunning() const;
    int GetRpm() const;
};

class Car {
private:
    Engine engine;

public:
    void Start();
    void Accelerate();
    void Stop();
    void PrintStatus() const;
};

#endif
```

### Car.cpp

``` cpp
#include "Car.h"

#include <iostream>

Engine::Engine() : running(false), rpm(0) {}

void Engine::Start() {
    running = true;
    rpm = 800;
}

void Engine::Stop() {
    running = false;
    rpm = 0;
}

void Engine::Accelerate() {
    if (running) {
        rpm += 500;
    }
}

bool Engine::IsRunning() const {
    return running;
}

int Engine::GetRpm() const {
    return rpm;
}

void Car::Start() {
    engine.Start();
}

void Car::Accelerate() {
    engine.Accelerate();
}

void Car::Stop() {
    engine.Stop();
}

void Car::PrintStatus() const {
    std::cout << "发动机 " << (engine.IsRunning() ? "ON" : "OFF") << '\n';
    std::cout << "转速 " << engine.GetRpm() << '\n';
}
```

### main.cpp

``` cpp
#include "Car.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Car car;
    car.Start();
    car.Accelerate();
    car.Accelerate();

    std::ostringstream runningOutput;
    std::streambuf* oldBuffer = std::cout.rdbuf(runningOutput.rdbuf());
    car.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(runningOutput.str() == "发动机 ON\n转速 1800\n");

    car.Stop();

    std::ostringstream stoppedOutput;
    oldBuffer = std::cout.rdbuf(stoppedOutput.rdbuf());
    car.PrintStatus();
    std::cout.rdbuf(oldBuffer);
    assert(stoppedOutput.str() == "发动机 OFF\n转速 0\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_7_student_count

### Student.h

```cpp
#ifndef STUDENT_H
#define STUDENT_H

class Student {
private:
    static int count;

public:
    Student();
    ~Student();
    static int GetCount();
};

#endif
```

### Student.cpp

```cpp
#include "Student.h"

int Student::count = 0;

Student::Student() {
    ++count;
}

Student::~Student() {
    --count;
}

int Student::GetCount() {
    return count;
}
```

### main.cpp

```cpp
#include "Student.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    assert(Student::GetCount() == 0);
    Student first;
    Student second;
    assert(Student::GetCount() == 2);
    {
        Student third;
        assert(Student::GetCount() == 3);
    }
    assert(Student::GetCount() == 2);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "当前学生数 " << Student::GetCount() << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "当前学生数 2\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_8_id_generator

### IdGenerator.h

```cpp
#ifndef ID_GENERATOR_H
#define ID_GENERATOR_H

class IdGenerator {
private:
    int id;
    static int nextId;

public:
    IdGenerator();
    int GetId() const;
    static void Reset(int startId);
};

#endif
```

### IdGenerator.cpp

```cpp
#include "IdGenerator.h"

int IdGenerator::nextId = 1001;

IdGenerator::IdGenerator() : id(nextId++) {}

int IdGenerator::GetId() const {
    return id;
}

void IdGenerator::Reset(int startId) {
    nextId = startId;
}
```

### main.cpp

```cpp
#include "IdGenerator.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    IdGenerator first;
    IdGenerator second;
    assert(first.GetId() == 1001);
    assert(second.GetId() == 1002);
    IdGenerator::Reset(2001);
    IdGenerator third;
    assert(third.GetId() == 2001);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "新编号 " << third.GetId() << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "新编号 2001\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_9_point_friend

### Point.h

```cpp
#ifndef POINT_H
#define POINT_H

class Point {
private:
    double x;
    double y;

public:
    Point(double x, double y);
    friend double Distance(const Point& first, const Point& second);
};

double Distance(const Point& first, const Point& second);

#endif
```

### Point.cpp

```cpp
#include "Point.h"

#include <cmath>

Point::Point(double x, double y) : x(x), y(y) {}

double Distance(const Point& first, const Point& second) {
    const double deltaX = first.x - second.x;
    const double deltaY = first.y - second.y;
    return std::sqrt(deltaX * deltaX + deltaY * deltaY);
}
```

### main.cpp

```cpp
#include "Point.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Point origin(0.0, 0.0);
    Point point(3.0, 4.0);
    assert(Distance(origin, point) == 5.0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "两点距离 " << Distance(origin, point) << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "两点距离 5\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_10_const_book

### Book.h

```cpp
#ifndef BOOK_H
#define BOOK_H

#include <string>

class Book {
private:
    std::string title;
    double price;

public:
    Book(const std::string& title, double price);
    const std::string& GetTitle() const;
    double GetPrice() const;
    void Print() const;
};

void PrintBook(const Book& book);

#endif
```

### Book.cpp

```cpp
#include "Book.h"

#include <iomanip>
#include <iostream>

Book::Book(const std::string& title, double price)
    : title(title), price(price) {}

const std::string& Book::GetTitle() const {
    return title;
}

double Book::GetPrice() const {
    return price;
}

void Book::Print() const {
    std::cout << "书名 " << title << '\n';
    std::cout << "价格 " << std::fixed << std::setprecision(2) << price << '\n';
}

void PrintBook(const Book& book) {
    book.Print();
}
```

### main.cpp

```cpp
#include "Book.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    const Book book("C++ 程序设计", 68.50);
    assert(book.GetTitle() == "C++ 程序设计");
    assert(book.GetPrice() == 68.50);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    PrintBook(book);
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "书名 C++ 程序设计\n价格 68.50\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_11_student_array

### Student.h

``` cpp
#ifndef STUDENT_H
#define STUDENT_H

#include <string>

class Student {
public:
    int SID;
    std::string Name;
    float Score;

    Student(int sid, const std::string& name, float sco);
};

void Add(int sid, const std::string& name, float sco);
void PrintAll();
void Average();

#endif
```

### Student.cpp

``` cpp
#include "Student.h"

#include <iomanip>
#include <iostream>

Student students[5] = {
    Student(0, "", 0.0f),
    Student(0, "", 0.0f),
    Student(0, "", 0.0f),
    Student(0, "", 0.0f),
    Student(0, "", 0.0f)
};
int studentCount = 0;

Student::Student(int sid, const std::string& name, float sco)
    : SID(sid), Name(name), Score(sco) {}

void Add(int sid, const std::string& name, float sco) {
    if (studentCount < 5) {
        students[studentCount] = Student(sid, name, sco);
        ++studentCount;
    }
}

void PrintAll() {
    for (int i = 0; i < studentCount; ++i) {
        std::cout << students[i].SID << ' '
                  << students[i].Name << ' '
                  << students[i].Score << '\n';
    }
}

void Average() {
    float sum = 0.0f;
    for (int i = 0; i < studentCount; ++i) {
        sum += students[i].Score;
    }

    float average = studentCount == 0 ? 0.0f : sum / studentCount;
    std::cout << "平均成绩 " << std::fixed << std::setprecision(4)
              << average << '\n';
}
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

## 2_12_product_array

### Product.h

``` cpp
#ifndef PRODUCT_H
#define PRODUCT_H

#include <cstddef>
#include <string>

class Product {
private:
    std::string name;
    double price;

public:
    Product(const std::string& name, double price);
    const std::string& GetName() const;
    double GetPrice() const;
};

void PrintProducts(const Product products[], std::size_t count);
double CalculateTotal(const Product products[], std::size_t count);

#endif
```

### Product.cpp

``` cpp
#include "Product.h"

#include <iomanip>
#include <iostream>

Product::Product(const std::string& name, double price)
    : name(name), price(price) {}

const std::string& Product::GetName() const {
    return name;
}

double Product::GetPrice() const {
    return price;
}

void PrintProducts(const Product products[], std::size_t count) {
    for (std::size_t i = 0; i < count; ++i) {
        std::cout << products[i].GetName() << ' '
                  << std::fixed << std::setprecision(2)
                  << products[i].GetPrice() << '\n';
    }
}

double CalculateTotal(const Product products[], std::size_t count) {
    double total = 0.0;
    for (std::size_t i = 0; i < count; ++i) {
        total += products[i].GetPrice();
    }
    return total;
}
```

### main.cpp

``` cpp
#include "Product.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Product products[] = {
        Product("笔记本", 12.50),
        Product("水杯", 35.00),
        Product("书包", 89.50)
    };
    const std::size_t count = sizeof(products) / sizeof(products[0]);

    assert(CalculateTotal(products, count) == 137.00);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    PrintProducts(products, count);
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "笔记本 12.50\n"
           "水杯 35.00\n"
           "书包 89.50\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_13_account_pointer

### Account.h

``` cpp
#ifndef ACCOUNT_H
#define ACCOUNT_H

#include <string>

class Account {
private:
    std::string id;
    double balance;

public:
    Account(const std::string& id, double balance);
    void Deposit(double amount);
    bool Withdraw(double amount);
    const std::string& GetId() const;
    double GetBalance() const;
    void Print() const;
};

#endif
```

### Account.cpp

``` cpp
#include "Account.h"

#include <iomanip>
#include <iostream>

Account::Account(const std::string& id, double balance)
    : id(id), balance(balance) {}

void Account::Deposit(double amount) {
    if (amount > 0) {
        balance += amount;
    }
}

bool Account::Withdraw(double amount) {
    if (amount > 0 && amount <= balance) {
        balance -= amount;
        return true;
    }
    return false;
}

const std::string& Account::GetId() const {
    return id;
}

double Account::GetBalance() const {
    return balance;
}

void Account::Print() const {
    std::cout << "账号 " << id << '\n';
    std::cout << "余额 " << std::fixed << std::setprecision(2)
              << balance << '\n';
}
```

### main.cpp

``` cpp
#include "Account.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Account* account = new Account("A001", 100.00);

    account->Deposit(50.00);
    assert(account->GetBalance() == 150.00);

    assert(account->Withdraw(20.00));
    assert(account->GetBalance() == 130.00);

    assert(!account->Withdraw(200.00));
    assert(account->GetBalance() == 130.00);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    account->Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "账号 A001\n余额 130.00\n");

    delete account;
    account = nullptr;
    assert(account == nullptr);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_14_dynamic_int_array

### IntArray.h

```cpp
#ifndef INT_ARRAY_H
#define INT_ARRAY_H

#include <cstddef>

class IntArray {
private:
    int* data;
    std::size_t size;

public:
    explicit IntArray(std::size_t size);
    ~IntArray();
    void Set(std::size_t index, int value);
    int Get(std::size_t index) const;
    int Sum() const;
    void Print() const;
};

#endif
```

### IntArray.cpp

```cpp
#include "IntArray.h"

#include <cassert>
#include <iostream>

IntArray::IntArray(std::size_t size) : data(new int[size]), size(size) {
    for (std::size_t i = 0; i < size; ++i) {
        data[i] = 0;
    }
}

IntArray::~IntArray() {
    delete[] data;
    data = nullptr;
}

void IntArray::Set(std::size_t index, int value) {
    assert(index < size);
    data[index] = value;
}

int IntArray::Get(std::size_t index) const {
    assert(index < size);
    return data[index];
}

int IntArray::Sum() const {
    int sum = 0;
    for (std::size_t i = 0; i < size; ++i) {
        sum += data[i];
    }
    return sum;
}

void IntArray::Print() const {
    for (std::size_t i = 0; i < size; ++i) {
        if (i != 0) {
            std::cout << ' ';
        }
        std::cout << data[i];
    }
    std::cout << '\n';
}
```

### main.cpp

```cpp
#include "IntArray.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    IntArray values(4);
    values.Set(0, 10);
    values.Set(1, -3);
    values.Set(2, 8);
    values.Set(3, 5);

    assert(values.Get(0) == 10);
    assert(values.Get(1) == -3);
    assert(values.Sum() == 20);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    values.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "10 -3 8 5\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_15_dynamic_string

### Text.h

```cpp
#ifndef TEXT_H
#define TEXT_H

#include <cstddef>

class Text {
private:
    char* content;

public:
    explicit Text(const char* text);
    ~Text();
    std::size_t Length() const;
    char Get(std::size_t index) const;
    void Print() const;
};

#endif
```

### Text.cpp

```cpp
#include "Text.h"

#include <cassert>
#include <cstring>
#include <iostream>

Text::Text(const char* text) {
    assert(text != nullptr);
    content = new char[std::strlen(text) + 1];
    std::strcpy(content, text);
}

Text::~Text() {
    delete[] content;
    content = nullptr;
}

std::size_t Text::Length() const {
    return std::strlen(content);
}

char Text::Get(std::size_t index) const {
    assert(index < Length());
    return content[index];
}

void Text::Print() const {
    std::cout << content << '\n';
}
```

### main.cpp

```cpp
#include "Text.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    char source[] = "hello";
    Text text(source);
    source[0] = 'H';

    assert(text.Length() == 5);
    assert(text.Get(0) == 'h');
    assert(text.Get(4) == 'o');

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    text.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "hello\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_16_deep_copy_text

### Text.h

```cpp
#ifndef TEXT_H
#define TEXT_H

#include <cstddef>

class Text {
private:
    char* content;

public:
    explicit Text(const char* text);
    Text(const Text& other);
    ~Text();
    void Set(std::size_t index, char value);
    const char* CStr() const;
    void Print() const;
};

#endif
```

### Text.cpp

```cpp
#include "Text.h"

#include <cassert>
#include <cstring>
#include <iostream>

Text::Text(const char* text) {
    assert(text != nullptr);
    content = new char[std::strlen(text) + 1];
    std::strcpy(content, text);
}

Text::Text(const Text& other) {
    content = new char[std::strlen(other.content) + 1];
    std::strcpy(content, other.content);
}

Text::~Text() {
    delete[] content;
    content = nullptr;
}

void Text::Set(std::size_t index, char value) {
    assert(index < std::strlen(content));
    content[index] = value;
}

const char* Text::CStr() const {
    return content;
}

void Text::Print() const {
    std::cout << content << '\n';
}
```

### main.cpp

```cpp
#include "Text.h"

#include <cassert>
#include <cstring>
#include <iostream>
#include <sstream>

void test() {
    Text original("hello");
    Text copy = original;
    copy.Set(0, 'H');

    assert(std::strcmp(original.CStr(), "hello") == 0);
    assert(std::strcmp(copy.CStr(), "Hello") == 0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    original.Print();
    copy.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "hello\nHello\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_17_deep_copy_assignment

### IntArray.h

```cpp
#ifndef INT_ARRAY_H
#define INT_ARRAY_H

#include <cstddef>

class IntArray {
private:
    int* data;
    std::size_t size;

public:
    explicit IntArray(std::size_t size);
    IntArray(const IntArray& other);
    IntArray& operator=(const IntArray& other);
    ~IntArray();
    void Set(std::size_t index, int value);
    int Get(std::size_t index) const;
};

#endif
```

### IntArray.cpp

```cpp
#include "IntArray.h"

#include <cassert>

IntArray::IntArray(std::size_t size) : data(new int[size]), size(size) {
    for (std::size_t i = 0; i < size; ++i) {
        data[i] = 0;
    }
}

IntArray::IntArray(const IntArray& other)
    : data(new int[other.size]), size(other.size) {
    for (std::size_t i = 0; i < size; ++i) {
        data[i] = other.data[i];
    }
}

IntArray& IntArray::operator=(const IntArray& other) {
    if (this != &other) {
        int* newData = new int[other.size];
        for (std::size_t i = 0; i < other.size; ++i) {
            newData[i] = other.data[i];
        }

        delete[] data;
        data = newData;
        size = other.size;
    }
    return *this;
}

IntArray::~IntArray() {
    delete[] data;
    data = nullptr;
}

void IntArray::Set(std::size_t index, int value) {
    assert(index < size);
    data[index] = value;
}

int IntArray::Get(std::size_t index) const {
    assert(index < size);
    return data[index];
}
```

### main.cpp

```cpp
#include "IntArray.h"

#include <cassert>
#include <iostream>

void test() {
    IntArray source(3);
    source.Set(0, 10);
    source.Set(1, 20);
    source.Set(2, 30);

    IntArray copied(source);
    copied.Set(0, 99);
    assert(source.Get(0) == 10);
    assert(copied.Get(0) == 99);

    IntArray assigned(1);
    assigned = source;
    assigned.Set(1, 88);
    assert(source.Get(1) == 20);
    assert(assigned.Get(1) == 88);

    assigned = assigned;
    assert(assigned.Get(0) == 10);
    assert(assigned.Get(1) == 88);
    assert(assigned.Get(2) == 30);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 2_18_cstring_copy

```cpp
void copyName(char destination[], std::size_t capacity, const char source[]) {
    assert(destination != nullptr);
    assert(source != nullptr);
    assert(capacity >= std::strlen(source) + 1);

    std::strcpy(destination, source);
}

void test() {
    char source[] = "张如雪";
    char destination[32] = {};
    copyName(destination, sizeof(destination), source);

    assert(std::strcmp(destination, "张如雪") == 0);
    assert(std::strlen(destination) == std::strlen(source));

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << "姓名 " << destination << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "姓名 张如雪\n");
}
```

## 2_19_string_greeting

```cpp
std::string makeGreeting(const std::string& name) {
    return std::string("你好，") + name + "！欢迎学习 C++。";
}

void test() {
    assert(makeGreeting("李红霞") == "你好，李红霞！欢迎学习 C++。");
    assert(makeGreeting("张如雪") == "你好，张如雪！欢迎学习 C++。");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << makeGreeting("李红霞") << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "你好，李红霞！欢迎学习 C++。\n");
}
```

## 2_20_vector_scores

### ScoreList.h

```cpp
#ifndef SCORE_LIST_H
#define SCORE_LIST_H

#include <cstddef>
#include <vector>

class ScoreList {
private:
    std::vector<int> scores;

public:
    void Add(int score);
    std::size_t Size() const;
    double Average() const;
    void Print() const;
};

#endif
```

### ScoreList.cpp

```cpp
#include "ScoreList.h"

#include <iostream>

void ScoreList::Add(int score) {
    scores.push_back(score);
}

std::size_t ScoreList::Size() const {
    return scores.size();
}

double ScoreList::Average() const {
    if (scores.empty()) {
        return 0.0;
    }

    int sum = 0;
    for (int score : scores) {
        sum += score;
    }
    return static_cast<double>(sum) / scores.size();
}

void ScoreList::Print() const {
    std::cout << "分数列表：";
    for (std::size_t i = 0; i < scores.size(); ++i) {
        if (i != 0) {
            std::cout << ' ';
        }
        std::cout << scores[i];
    }
    std::cout << '\n';
}
```

### main.cpp

```cpp
#include "ScoreList.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    ScoreList scores;
    scores.Add(90);
    scores.Add(85);
    scores.Add(76);

    assert(scores.Size() == 3);
    assert(scores.Average() == (90.0 + 85.0 + 76.0) / 3.0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    scores.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "分数列表：90 85 76\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```
