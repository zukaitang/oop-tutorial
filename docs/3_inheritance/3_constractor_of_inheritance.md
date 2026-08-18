# 派生类的构造和析构函数

构造函数和析构函数是类管理对象生命周期的重要机制。当引入继承后，派生类的构造函数和析构函数需要协调基类与派生类成员的初始化与清理顺序。理解这些规则，是编写正确继承关系代码的关键。

## 从问题出发：谁负责初始化基类成员？

!!! question "核心问题"

    派生类对象包含了继承自基类的成员。当创建派生类对象时，这些基类成员由谁来初始化？

    ``` cpp linenums="1"
    class Vehicle {
    public:
        Vehicle(int wh, float wt) : wheels(wh), weight(wt) {}
    private:
        int wheels;
        float weight;
    };

    class Car : public Vehicle {
    public:
        // 问题：wheels 和 weight 如何初始化？
        Car(int wh, float wt, int pa) : passenger(pa) {
            // wheels = wh;      // ✗ 不能直接访问基类的 private 成员
            // weight = wt;      // ✗ 不能直接访问基类的 private 成员
        }
    private:
        int passenger;
    };
    ```

!!! success "答案：派生类构造函数调用基类构造函数"

    派生类的构造函数**不直接初始化**基类成员，而是通过调用基类的构造函数来完成基类部分的初始化。这体现了**职责分离**的设计原则——每个类只负责自己成员的初始化。

## 派生类构造函数

### 构造函数的继承规则

!!! info "核心规则"

    - **基类构造函数不被继承**：派生类需要定义自己的构造函数。
    - **初始化基类成员**：派生类构造函数通过**初始化列表**调用基类构造函数。
    - **调用时机**：基类构造函数先执行，然后才执行派生类构造函数体。

!!! example "基本语法"

    ``` cpp linenums="1"
    // 单继承
    派生类名::派生类名(基类所需形参, 本类成员所需形参)
        : 基类名(基类参数列表), 本类成员初始化列表 {
        // 其他初始化
    }

    // 示例
    Car::Car(int wh, float wt, int pa)
        : Vehicle(wh, wt), passenger(pa) {
        // 基类 Vehicle 的构造函数先被调用
        // 然后 passenger 被初始化为 pa
        // 最后执行函数体
    }
    ```

!!! warning "关键注意事项"

    - 如果基类没有默认构造函数（无参构造函数），派生类**必须**在初始化列表中显式调用基类的带参构造函数。
    - 如果基类有默认构造函数，派生类可以不在初始化列表中调用基类构造函数（此时自动调用基类的默认构造函数）。

### 基类没有默认构造函数的情况

当基类没有默认构造函数时，派生类必须在初始化列表中为基类构造函数提供参数。

!!! example "强制调用基类构造函数"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        // 只有带参构造函数，没有默认构造函数
        Base(int value) {
            cout << "Base constructor: " << value << endl;
        }
    };

    class Derived : public Base {
    public:
        // 必须显式调用基类的带参构造函数
        Derived(int v) : Base(v) {
            cout << "Derived constructor" << endl;
        }

        // 错误！编译失败
        // Derived() { }   // 无法找到 Base 的默认构造函数
    };

    int main() {
        Derived d(10);   // Base constructor: 10
                         // Derived constructor
        return 0;
    }
    ```

!!! danger "常见编译错误"

    ```
    error: no matching function for call to 'Base::Base()'
    note: candidates are: Base::Base(int)
    ```

    当派生类没有在初始化列表中调用基类构造函数，而基类又没有默认构造函数时，编译器会报错。

### 基类存在默认构造函数的情况

如果基类有默认构造函数，派生类可以省略对基类构造函数的调用。

!!! example "自动调用默认构造函数"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        // 有默认构造函数
        Base() {
            cout << "Base default constructor" << endl;
        }
        Base(int value) {
            cout << "Base parameterized constructor: " << value << endl;
        }
    };

    class Derived : public Base {
    public:
        // 方式一：不显式调用基类构造函数 → 自动调用 Base::Base()
        Derived() {
            cout << "Derived default constructor" << endl;
        }

        // 方式二：显式调用基类的带参构造函数
        Derived(int v) : Base(v) {
            cout << "Derived parameterized constructor" << endl;
        }
    };

    int main() {
        cout << "--- d1: Derived() ---" << endl;
        Derived d1;          // Base default constructor
                             // Derived default constructor

        cout << "--- d2: Derived(10) ---" << endl;
        Derived d2(10);      // Base parameterized constructor: 10
                             // Derived parameterized constructor

        return 0;
    }
    ```

### 构造函数的执行顺序

!!! info "构造顺序规则"

    1. **基类构造函数**：按照继承声明顺序调用基类的构造函数。
    2. **成员对象构造函数**：按照在类中声明的顺序，初始化成员对象（如有）。
    3. **派生类构造函数体**：执行派生类构造函数体中的代码。

!!! example "单继承构造顺序样例"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        Base(int i) {
            cout << "1. Base constructor: " << i << endl;
        }
    };

    class Member {
    public:
        Member(int j) {
            cout << "2. Member constructor: " << j << endl;
        }
    };

    class Derived : public Base {
    private:
        Member m;
    public:
        Derived(int a, int b) : Base(a), m(b) {
            cout << "3. Derived constructor body" << endl;
        }
    };

    int main() {
        Derived d(10, 20);
        return 0;
    }
    ```

    运行结果：

    ```
    1. Base constructor: 10
    2. Member constructor: 20
    3. Derived constructor body
    ```

!!! note "执行顺序与初始化列表书写顺序无关"

    构造函数的执行顺序由**继承声明顺序**和**成员声明顺序**决定，与初始化列表中的书写顺序无关：

    ``` cpp
    class Derived : public Base1, public Base2 {
        // Base1 先于 Base2 初始化（由继承声明顺序决定）
    };
    ```

在多继承情况下，基类构造函数的调用顺序按照**派生类定义时基类出现的顺序**（从左到右）。

!!! example "多继承构造顺序样例"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base1 {
    public:
        Base1(int i) { cout << "Base1: " << i << endl; }
    };

    class Base2 {
    public:
        Base2(int j) { cout << "Base2: " << j << endl; }
    };

    class Base3 {
    public:
        Base3() { cout << "Base3: default" << endl; }
    };

    // 继承顺序：Base2 → Base1 → Base3
    class Derived : public Base2, public Base1, public Base3 {
    private:
        Base1 m1;
        Base2 m2;
        Base3 m3;
    public:
        Derived(int a, int b, int c, int d)
            : Base1(a), m2(d), m1(c), Base2(b) {
            cout << "Derived body" << endl;
        }
    };

    int main() {
        Derived d(1, 2, 3, 4);
        return 0;
    }
    ```

    运行结果：

    ```
    Base2: 2      ← 按继承声明顺序：Base2 先于 Base1
    Base1: 1      ← Base1 先于 Base3
    Base3: default
    Base1: 3      ← 按成员声明顺序：m1 先于 m2
    Base2: 4      ← m2 先于 m3（m3 是 Base3 类型）
    Base3: default
    Derived body
    ```

!!! warning "关键观察"

    - **基类初始化顺序** = **继承声明顺序**（从左到右）。
    - **成员初始化顺序** = **成员声明顺序**（从上到下）。
    - **初始化列表书写顺序不影响实际执行顺序**。

## C++11 的构造函数继承

### using 声明继承构造函数

C++11 引入了 `using Base::Base;` 语法，允许派生类直接继承基类的构造函数。

!!! example "继承构造函数"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        Base() { cout << "Base()" << endl; }
        Base(int i) { cout << "Base(int): " << i << endl; }
        Base(double d) { cout << "Base(double): " << d << endl; }
    };

    class Derived : public Base {
    public:
        // 继承基类的所有构造函数
        using Base::Base;

        // 仍然可以定义自己的构造函数
        Derived(int i, int j) : Base(i) {
            cout << "Derived(int, int): " << i << ", " << j << endl;
        }
    };

    int main() {
        Derived d1;           // Base()
        Derived d2(10);       // Base(int): 10
        Derived d3(3.14);     // Base(double): 3.14
        Derived d4(10, 20);   // Base(int): 10
                              // Derived(int, int): 10, 20
        return 0;
    }
    ```

!!! warning "注意事项"

    - 继承的构造函数只能初始化基类成员，不能初始化派生类的新增成员。
    - 如果派生类新增的成员需要初始化，仍需要定义自己的构造函数。
    - 如果基类构造函数被标记为 `delete` 或 `private`，派生类无法继承。

## 派生类的复制构造函数

### 隐含的复制构造函数

如果派生类没有显式定义复制构造函数，编译器会生成一个隐含的复制构造函数，它会：

1. 调用基类的复制构造函数。
2. 为派生类新增的成员执行复制（对对象成员调用其复制构造函数，对基本类型逐字节复制）。

!!! example "隐含复制构造函数的行为"

    ``` cpp linenums="1" hl_lines="15-17"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        Base() { cout << "Base default" << endl; }
        Base(const Base& other) {
            cout << "Base copy constructor" << endl;
        }
    };

    class Derived : public Base {
    public:
        Derived() { cout << "Derived default" << endl; }
        // 未显式定义复制构造函数，编译器生成隐含版本
        // 等价于：
        // Derived(const Derived& other) : Base(other) { }
    };

    int main() {
        Derived d1;
        Derived d2(d1);   // 调用隐含的复制构造函数
        return 0;
    }
    ```

    运行结果：

    ```
    Base default
    Derived default
    Base copy constructor
    ```

### 显式定义复制构造函数

当派生类需要自定义复制行为时，应显式定义复制构造函数，并确保基类的复制构造函数被正确调用。

!!! example "显式定义复制构造函数"

    ``` cpp linenums="1" hl_lines="28-33"
    #include <iostream>
    #include <string>
    using namespace std;

    class Person {
    private:
        string name;
        int age;
    public:
        Person(const string& n, int a) : name(n), age(a) {}
        Person(const Person& other) : name(other.name), age(other.age) {
            cout << "Person copy constructor" << endl;
        }
        void show() const {
            cout << name << ", " << age << endl;
        }
    };

    class Student : public Person {
    private:
        string school;
        int grade;
    public:
        Student(const string& n, int a, const string& s, int g)
            : Person(n, a), school(s), grade(g) {}

        // 显式定义复制构造函数
        Student(const Student& other)
            : Person(other),      // 基类复制构造函数
              school(other.school),
              grade(other.grade) {
            cout << "Student copy constructor" << endl;
        }

        void show() const {
            Person::show();
            cout << school << ", " << grade << endl;
        }
    };

    int main() {
        Student s1("张三", 20, "清华大学", 3);
        Student s2(s1);   // 调用复制构造函数
        s2.show();
        return 0;
    }
    ```

    运行结果：

    ```
    Person copy constructor
    Student copy constructor
    张三, 20
    清华大学, 3
    ```

!!! warning "关键注意事项"

    - 派生类的复制构造函数必须**显式调用基类的复制构造函数**，否则基类部分会被默认构造。
    - 如果基类没有复制构造函数（例如被 `delete` 禁用），派生类也无法进行复制。

## 派生类的移动构造函数（C++11）

C++11 引入了移动语义，派生类也可以定义移动构造函数。

!!! example "移动构造函数"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    class Base {
    private:
        vector<int> data;
    public:
        Base(int size) : data(size) {
            cout << "Base constructed" << endl;
        }
        Base(Base&& other) noexcept : data(move(other.data)) {
            cout << "Base move constructor" << endl;
        }
        Base(const Base& other) : data(other.data) {
            cout << "Base copy constructor" << endl;
        }
    };

    class Derived : public Base {
    private:
        vector<double> extra;
    public:
        Derived(int size, int extraSize) : Base(size), extra(extraSize) {}
        Derived(Derived&& other) noexcept
            : Base(move(other)),      // 移动基类
              extra(move(other.extra)) {
            cout << "Derived move constructor" << endl;
        }
    };

    int main() {
        Derived d1(10, 5);
        Derived d2(move(d1));   // 调用移动构造函数
        return 0;
    }
    ```

## 析构函数

### 析构函数的基本规则

!!! info "核心规则"

    - **析构函数不被继承**：派生类需要自行声明析构函数（如果需要）。
    - **自动调用基类析构函数**：派生类析构函数执行完毕后，系统会**自动调用**基类的析构函数。
    - **调用顺序与构造函数相反**：先派生类，后基类。

!!! example "析构顺序演示"

    ``` cpp linenums="1" hl_lines="7 13"
    #include <iostream>
    using namespace std;

    class Base {
    public:
        Base() { cout << "Base constructor" << endl; }
        ~Base() { cout << "Base destructor" << endl; }
    };

    class Derived : public Base {
    public:
        Derived() { cout << "Derived constructor" << endl; }
        ~Derived() { cout << "Derived destructor" << endl; }
    };

    int main() {
        Derived d;
        cout << "--- 对象离开作用域 ---" << endl;
        return 0;
    }
    ```

    运行结果：

    ```
    Base constructor
    Derived constructor
    --- 对象离开作用域 ---
    Derived destructor
    Base destructor
    ```

!!! note "构造与析构的对称性"

    构造顺序：基类 → 派生类
    析构顺序：派生类 → 基类

    这种对称性保证了资源按照正确的顺序分配和释放。

### 多继承的析构顺序

在多继承情况下，析构顺序与构造函数正好相反：先执行派生类析构函数体，然后按照继承声明顺序的**逆序**调用基类析构函数。

!!! example "多继承析构顺序"

    ``` cpp linenums="1" hl_lines="6 10 14 20"
    #include <iostream>
    using namespace std;

    class Base1 {
    public:
        ~Base1() { cout << "Base1 destructor" << endl; }
    };
    class Base2 {
    public:
        ~Base2() { cout << "Base2 destructor" << endl; }
    };
    class Base3 {
    public:
        ~Base3() { cout << "Base3 destructor" << endl; }
    };

    // 继承声明顺序：Base2 → Base1 → Base3
    class Derived : public Base2, public Base1, public Base3 {
    public:
        ~Derived() { cout << "Derived destructor" << endl; }
    };

    int main() {
        Derived d;
        return 0;
    }
    ```

    运行结果：

    ```
    Derived destructor
    Base3 destructor    ← 逆序：最后一个继承的先析构
    Base1 destructor
    Base2 destructor
    ```

## 构造函数与析构函数完整示例

!!! example "类继承中的构造函数与析构函数"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 基类 Vehicle
    class Vehicle {
    public:
        // 构造函数
        Vehicle(int wh, float wt) : wheels(wh), weight(wt) {
            cout << "Vehicle constructor: wheels=" << wh << ", weight=" << wt << endl;
        }

        // 复制构造函数
        Vehicle(const Vehicle& other) : wheels(other.wheels), weight(other.weight) {
            cout << "Vehicle copy constructor" << endl;
        }

        // 析构函数（虚函数）
        virtual ~Vehicle() {
            cout << "Vehicle destructor" << endl;
        }

        void print() const {
            cout << "wheels=" << wheels << ", weight=" << weight;
        }
    private:
        int wheels;
        float weight;
    };

    // 派生类 Car
    class Car : public Vehicle {
    public:
        // 构造函数
        Car(int wh, float wt, int pa, const char* br)
            : Vehicle(wh, wt), passenger(pa) {
            brand = new char[strlen(br) + 1];
            strcpy(brand, br);
            cout << "Car constructor: passenger=" << pa << ", brand=" << br << endl;
        }

        // 复制构造函数（深拷贝）
        Car(const Car& other)
            : Vehicle(other), passenger(other.passenger) {
            brand = new char[strlen(other.brand) + 1];
            strcpy(brand, other.brand);
            cout << "Car copy constructor (deep copy)" << endl;
        }

        // 析构函数
        ~Car() {
            cout << "Car destructor: deleting brand" << endl;
            delete[] brand;
        }

        void print() const {
            Vehicle::print();
            cout << ", passenger=" << passenger << ", brand=" << brand << endl;
        }
    private:
        int passenger;
        char* brand;   // 动态分配的字符串，演示资源管理
    };

    // 进一步的派生类 SUV
    class SUV : public Car {
    public:
        SUV(int wh, float wt, int pa, const char* br, bool is4WD)
            : Car(wh, wt, pa, br), isFourWheelDrive(is4WD) {
            cout << "SUV constructor: 4WD=" << (is4WD ? "yes" : "no") << endl;
        }

        SUV(const SUV& other) : Car(other), isFourWheelDrive(other.isFourWheelDrive) {
            cout << "SUV copy constructor" << endl;
        }

        ~SUV() {
            cout << "SUV destructor" << endl;
        }

        void print() const {
            Car::print();
            cout << ", 4WD=" << (isFourWheelDrive ? "yes" : "no") << endl;
        }
    private:
        bool isFourWheelDrive;
    };

    int main() {
        cout << "=== 创建 SUV 对象 ===" << endl;
        SUV suv(4, 2.0, 5, "Toyota", true);

        cout << "\n=== 复制 SUV 对象 ===" << endl;
        SUV suv2(suv);

        cout << "\n=== 显示对象信息 ===" << endl;
        cout << "suv: ";
        suv.print();
        cout << "suv2: ";
        suv2.print();

        cout << "\n=== 通过基类指针删除对象 ===" << endl;
        Vehicle* p = new SUV(4, 1.8, 4, "Honda", false);
        delete p;   // 由于析构函数为虚，会正确调用所有析构函数

        cout << "\n=== 程序结束 ===" << endl;
        return 0;
    }
    ```

    运行结果

    ```
    === 创建 SUV 对象 ===
    Vehicle constructor: wheels=4, weight=2
    Car constructor: passenger=5, brand=Toyota
    SUV constructor: 4WD=yes

    === 复制 SUV 对象 ===
    Vehicle copy constructor
    Car copy constructor (deep copy)
    SUV copy constructor

    === 显示对象信息 ===
    suv: wheels=4, weight=2, passenger=5, brand=Toyota, 4WD=yes
    suv2: wheels=4, weight=2, passenger=5, brand=Toyota, 4WD=yes

    === 通过基类指针删除对象 ===
    Vehicle constructor: wheels=4, weight=1.8
    Car constructor: passenger=4, brand=Honda
    SUV constructor: 4WD=no
    SUV destructor
    Car destructor: deleting brand
    Vehicle destructor

    === 程序结束 ===
    SUV destructor
    Car destructor: deleting brand
    Vehicle destructor
    SUV destructor
    Car destructor: deleting brand
    Vehicle destructor
    ```

## 小结

1.  **构造函数**：
    1. 基类构造函数**不被继承**，派生类需定义自己的构造函数。
    2. 派生类构造函数通过初始化列表调用基类构造函数。
    3. 若基类无默认构造函数，派生类**必须显式调用**基类的带参构造函数。
2.  **构造执行顺序**：
    1. 基类构造函数（按继承声明顺序）→ 成员对象构造函数（按声明顺序）→ 派生类构造函数体。
    2. 顺序与初始化列表书写顺序无关。
3.  **复制构造函数**：
    1. 隐含版本自动调用基类复制构造函数。
    2. 显式定义时，必须显式调用基类复制构造函数：`Derived(const Derived& d) : Base(d) { ... }`
4.  **移动构造函数**（C++11）：
    1. 使用 `Base(move(other))` 调用基类移动构造函数。
5.  **析构函数**：
    1. 派生类析构函数体执行后，自动调用基类析构函数。
    2. 析构顺序与构造顺序相反。
    3. **基类析构函数应声明为 `virtual`**，避免资源泄漏。
6.  **构造函数继承**（C++11）：
    1. 使用 `using Base::Base;` 可以继承基类的构造函数。