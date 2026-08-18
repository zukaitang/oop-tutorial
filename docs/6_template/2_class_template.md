# 类模板

类模板是C++泛型编程的另一个核心工具。它允许程序员定义与类型无关的通用类蓝图，在实例化时指定具体的数据类型。
类模板广泛应用于标准库容器（如 `vector`、`list`、`stack` 等），是构建可复用数据结构的基础。

## 问题场景：重复的数据结构定义

在程序设计中，经常遇到**数据结构相同、元素类型不同**的情况。

!!! question "不同数据类型的数组类"

    ``` cpp
    // 需要为每种数据类型编写一个数组类？
    class IntArray {
        int* data;
        int size;
        // ... 相同的逻辑
    };

    class DoubleArray {
        double* data;
        int size;
        // ... 相同的逻辑
    };

    class StudentArray {
        Student* data;
        int size;
        // ... 相同的逻辑
    };
    ```

    这种为每种类型重复编写相同逻辑的做法，与函数重载面临的冗余问题如出一辙：

    - **代码冗余**：大量重复的类定义。
    - **维护困难**：修改数据结构或算法时需要同步修改所有版本。
    - **可扩展性差**：每增加一种新类型就需要定义一个新的类。

!!! abstract "解决方案"

    **类模板（Class Template）** 允许我们定义一个通用的类蓝图，在实例化时指定元素类型，编译器自动生成对应类型的类代码。

## 类模板的定义

### 基本语法

类模板定义了一个类族，其中的类型参数在实例化时被具体类型替换。

!!! info "类模板的语法格式"

    ``` cpp
    template <typename 类型参数1, typename 类型参数2, ...>
    class 类名 {
        // 类成员声明
        // 可以使用类型参数作为：
        // - 数据成员的类型
        // - 成员函数的参数类型
        // - 成员函数的返回类型
    };
    ```

!!! example "简单的类模板示例：Point类"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 类模板：Point
    template <typename T>
    class Point {
    public:
        // 构造函数参数使用类型参数
        Point(T x = 0, T y = 0) : x(x), y(y) {}

        // 成员函数参数和返回值使用类型参数
        void setX(T value) { x = value; }
        void setY(T value) { y = value; }
        T getX() const { return x; }
        T getY() const { return y; }

        void display() const {
            cout << "(" << x << ", " << y << ")" << endl;
        }
    private:
        T x, y;    // 类型参数作为数据成员类型
    };

    int main() {
        // 实例化 int 类型的 Point
        Point<int> p1(3, 4);
        p1.display();   // (3, 4)

        // 实例化 double 类型的 Point
        Point<double> p2(3.14, 2.71);
        p2.display();   // (3.14, 2.71)

        // 实例化 string 类型的 Point
        Point<string> p3("Hello", "World");
        p3.display();   // (Hello, World)

        return 0;
    }
    ```

### 成员函数的定义

类模板的成员函数可以在**类内定义**（隐式内联），也可以在**类外定义**。类外定义时需要特殊的模板语法。

#### 方式一：类内定义

- 类内定义的成员函数**默认是内联函数**。
- 适合函数体较短、逻辑简单的情况。

!!! example "成员函数在类内定义"

    ``` cpp linenums="1"
    template <typename T>
    class Box {
    public:
        // 构造函数在类内定义
        Box(const T& v) : value(v) {}

        // 成员函数在类内定义
        void setValue(const T& v) {
            value = v;
        }

        T getValue() const {
            return value;
        }

        void display() const {
            cout << "Box(" << value << ")" << endl;
        }
    private:
        T value;
    };
    ```

#### 方式二：类外定义

!!! example "成员函数在类外定义"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cassert>
    using namespace std;

    // 类模板声明
    template <typename T>
    class Array {
    public:
        // 构造函数
        Array(int sz = 50);

        // 复制构造函数（深拷贝）
        Array(const Array<T>& a);

        // 析构函数
        ~Array();

        // 下标运算符
        T& operator[](int n);
        const T& operator[](int n) const;

        // 获取数组大小
        int getSize() const;

        // 改变数组大小
        void resize(int sz);
    private:
        T* list;     // 动态数组
        int size;    // 数组大小
    };

    // ===== 类外定义成员函数 =====

    // 1. 构造函数
    template <typename T>
    Array<T>::Array(int sz) : size(sz) {
        assert(sz >= 0);
        list = new T[size];
    }

    // 2. 复制构造函数（深拷贝）
    template <typename T>
    Array<T>::Array(const Array<T>& a) {
        size = a.size;
        list = new T[size];
        for (int i = 0; i < size; i++) {
            list[i] = a.list[i];
        }
    }

    // 3. 析构函数
    template <typename T>
    Array<T>::~Array() {
        delete[] list;
    }

    // 4. 下标运算符（非 const 版本）
    template <typename T>
    T& Array<T>::operator[](int n) {
        assert(n >= 0 && n < size);
        return list[n];
    }

    // 5. 下标运算符（const 版本）
    template <typename T>
    const T& Array<T>::operator[](int n) const {
        assert(n >= 0 && n < size);
        return list[n];
    }

    // 6. 获取数组大小
    template <typename T>
    int Array<T>::getSize() const {
        return size;
    }

    // 7. 改变数组大小
    template <typename T>
    void Array<T>::resize(int sz) {
        assert(sz >= 0);
        if (sz == size) return;

        T* newList = new T[sz];
        int n = (sz < size) ? sz : size;
        for (int i = 0; i < n; i++) {
            newList[i] = list[i];
        }
        delete[] list;
        list = newList;
        size = sz;
    }

    int main() {
        Array<int> a(5);

        for (int i = 0; i < a.getSize(); i++) {
            a[i] = i * 10;
        }

        for (int i = 0; i < a.getSize(); i++) {
            cout << a[i] << " ";
        }
        cout << endl;   // 0 10 20 30 40

        return 0;
    }
    ```

!!! tip "类外定义的关键语法要点"

    | 要点               | 说明                                             | 示例                    |
    | :----------------- | :----------------------------------------------- | :---------------------- |
    | **重复模板声明**   | 每个成员函数定义前都需要 `template <typename T>` | `template <typename T>` |
    | **类名带模板参数** | 类名要写成 `Array<T>` 而不是 `Array`             | `Array<T>::Array`       |
    | **作用域运算符**   | `::` 前的类型是完整的 `Array<T>`                 | `Array<T>::operator[]`  |

## 类模板的实例化

与函数模板不同，类模板的实例化必须**显式指定模板参数**（编译器无法推导）。

!!! example "类模板的实例化方式"

    ``` cpp linenums="1" hl_lines="31 35 40 46"
    #include <iostream>
    #include <string>
    using namespace std;

    template <typename T>
    class Store {
    private:
        T item;
        bool hasValue;

    public:
        Store() : hasValue(false) {}

        void put(const T& value) {
            item = value;
            hasValue = true;
        }

        T get() const {
            if (!hasValue) {
                throw runtime_error("No value stored");
            }
            return item;
        }

        bool isEmpty() const { return !hasValue; }
    };

    int main() {
        // 方式一：使用 <类型> 显式指定
        Store<int> intStore;
        intStore.put(100);
        cout << intStore.get() << endl;   // 100

        Store<string> stringStore;
        stringStore.put("Hello");
        cout << stringStore.get() << endl;   // Hello

        // 方式二：使用类型别名简化（推荐）
        using IntStore = Store<int>;
        IntStore store2;
        store2.put(200);
        cout << store2.get() << endl;   // 200

        // 方式三：typedef（C++98/03 风格）
        typedef Store<double> DoubleStore;
        DoubleStore store3;
        store3.put(3.14);
        cout << store3.get() << endl;   // 3.14

        return 0;
    }
    ```

## 类模板的默认模板参数

类模板可以为类型参数指定默认值，方便使用。

!!! example "默认模板参数"

    ``` cpp linenums="1" hl_lines="5 24 29 34"
    #include <iostream>
    using namespace std;

    // 默认模板参数：T 默认为 int
    template <typename T = int>
    class Counter {
    public:
        Counter(T initial = 0) : count(initial) {}

        void increment() { count++; }
        void decrement() { count--; }

        T getCount() const { return count; }

        void display() const {
            cout << "Count: " << count << endl;
        }
    private:
        T count;
    };

    int main() {
        // 使用默认的 int 类型
        Counter<> c1;        // 注意：需要写 <>
        c1.increment();
        c1.display();        // Count: 1

        // 指定为 double 类型
        Counter<double> c2(3.14);
        c2.increment();
        c2.display();        // Count: 4.14

        // C++17 起可以省略 <>
        Counter c3(100);     // 自动推导为 int
        c3.display();        // Count: 100

        return 0;
    }
    ```

!!! warning "注意事项"

    - 使用默认模板参数时，空模板参数列表需要写成 `<>`（除非使用 C++17 的类模板参数推导）。
    - 如果有多个模板参数，默认值可以从右向左指定（即可以为部分参数指定默认值）。

## 类模板的友元

类模板中的友元声明需要特别注意，因为友元函数或类本身也可能是模板。

### 友元函数在类内定义

!!! example "友元函数在类内定义"

    ``` cpp linenums="1" hl_lines="11-14 17-19"
    #include <iostream>
    using namespace std;

    template <typename T>
    class Box {
    public:
        Box(const T& v) : value(v) {}

        // 友元函数：在类内定义（隐式内联）
        // 注意：这里的 Box<T> 与类模板参数一致
        friend ostream& operator<<(ostream& os, const Box<T>& b) {
            os << "Box(" << b.value << ")";
            return os;
        }

        // 友元函数：也可以与类模板参数无关
        friend void showBox() {
            cout << "This is a Box" << endl;
        }
    private:
        T value;
    };

    int main() {
        Box<int> b1(10);
        Box<double> b2(3.14);

        cout << b1 << endl;   // Box(10)
        cout << b2 << endl;   // Box(3.14)

        showBox();            // This is a Box

        return 0;
    }
    ```

### 友元函数在类外定义

!!! example "友元函数在类外定义"

    ``` cpp linenums="1" hl_lines="18 26-29"
    #include <iostream>
    using namespace std;

    // 前向声明
    template <typename T>
    class Box;

    // 友元函数声明（需要先声明模板）
    template <typename T>
    bool operator==(const Box<T>& a, const Box<T>& b);

    template <typename T>
    class Box {
    public:
        Box(const T& v) : value(v) {}

        // 友元声明：指向类外定义的函数模板
        friend bool operator== <T>(const Box<T>& a, const Box<T>& b);

        T getValue() const { return value; }
    private:
        T value;
    };

    // 友元函数在类外定义
    template <typename T>
    bool operator==(const Box<T>& a, const Box<T>& b) {
        return a.value == b.value;
    }

    int main() {
        Box<int> b1(10);
        Box<int> b2(20);
        Box<int> b3(10);

        cout << (b1 == b2 ? "相等" : "不相等") << endl;   // 不相等
        cout << (b1 == b3 ? "相等" : "不相等") << endl;   // 相等

        return 0;
    }
    ```

### 友元类

!!! example "友元类"

    ``` cpp linenums="1" hl_lines="10-11"
    #include <iostream>
    using namespace std;

    template <typename T>
    class Container {
    public:
        Container(const T& d) : data(d) {}

        // 将整个 Manager 类声明为友元
        template <typename U>
        friend class Manager;
    private:
        T data;
    };

    template <typename T>
    class Manager {
    public:
        void show(const Container<T>& c) {
            cout << "Container data: " << c.data << endl;
        }

        void setData(Container<T>& c, const T& value) {
            c.data = value;
        }
    };

    int main() {
        Container<int> c(100);
        Manager<int> mgr;

        mgr.show(c);        // Container data: 100
        mgr.setData(c, 200);
        mgr.show(c);        // Container data: 200

        return 0;
    }
    ```

## 类模板的静态成员

类模板也可以有静态成员。每个**模板实例化版本**拥有自己独立的静态成员副本。

!!! example "类模板的静态成员"

    ``` cpp linenums="1" hl_lines="15-17 19 24-25"
    #include <iostream>
    using namespace std;

    template <typename T>
    class Counter {
    public:
        Counter() {
            count++;
        }

        ~Counter() {
            count--;
        }

        static int getCount() {
            return count;
        }
    private:
        static int count;   // 静态成员声明

    };

    // 静态成员定义（每个实例化版本各有一份）
    template <typename T>
    int Counter<T>::count = 0;

    int main() {
        cout << "Counter<int> count: " << Counter<int>::getCount() << endl;   // 0

        Counter<int> c1, c2, c3;
        cout << "Counter<int> count: " << Counter<int>::getCount() << endl;   // 3

        {
            Counter<double> d1, d2;
            cout << "Counter<double> count: " << Counter<double>::getCount() << endl; // 2
            cout << "Counter<int> count: " << Counter<int>::getCount() << endl;       // 3（独立）
        }

        cout << "Counter<double> count: " << Counter<double>::getCount() << endl; // 0

        return 0;
    }
    ```

!!! info "重要概念"

    每个模板实例化版本（如 `Counter<int>` 和 `Counter<double>`）拥有**独立的静态成员副本**。`Counter<int>::count` 和 `Counter<double>::count` 是两个不同的变量。

## 类模板的特化

当类模板的通用实现不适用于某些特定类型时，可以对这些类型提供**特化（Specialization）**版本。

### 全特化

!!! example "类模板的全特化"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cstring>
    using namespace std;

    // 通用模板
    template <typename T>
    class Comparator {
    public:
        bool isEqual(const T& a, const T& b) const {
            return a == b;
        }
    };

    // 全特化：针对 const char* 类型
    template <>
    class Comparator<const char*> {
    public:
        bool isEqual(const char* a, const char* b) const {
            return strcmp(a, b) == 0;
        }
    };

    // 全特化：针对 char* 类型
    template <>
    class Comparator<char*> {
    public:
        bool isEqual(char* a, char* b) const {
            return strcmp(a, b) == 0;
        }
    };

    int main() {
        Comparator<int> intComp;
        cout << intComp.isEqual(5, 5) << endl;   // 1
        cout << intComp.isEqual(5, 3) << endl;   // 0

        Comparator<const char*> strComp;
        cout << strComp.isEqual("hello", "hello") << endl;   // 1
        cout << strComp.isEqual("hello", "world") << endl;   // 0

        return 0;
    }
    ```

### 偏特化（部分特化）

类模板支持偏特化，当模板有多个参数时，可以为部分参数指定特化。

!!! example "类模板的偏特化"

    ``` cpp linenums="1"
    #include <iostream>
    using namespace std;

    // 通用模板：两个类型参数
    template <typename T1, typename T2>
    class Pair {
    public:
        void print() const {
            cout << "General Pair" << endl;
        }
    };

    // 偏特化：两个类型相同
    template <typename T>
    class Pair<T, T> {
    public:
        void print() const {
            cout << "Same-type Pair" << endl;
        }
    };

    // 偏特化：第二个类型是 int
    template <typename T>
    class Pair<T, int> {
    public:
        void print() const {
            cout << "Second type is int" << endl;
        }
    };

    // 偏特化：两个都是指针类型
    template <typename T1, typename T2>
    class Pair<T1*, T2*> {
    public:
        void print() const {
            cout << "Pointer Pair" << endl;
        }
    };

    int main() {
        Pair<int, double> p1;   // 通用模板
        p1.print();             // General Pair

        Pair<int, int> p2;      // 偏特化：相同类型
        p2.print();             // Same-type Pair

        Pair<double, int> p3;   // 偏特化：第二个是 int
        p3.print();             // Second type is int

        Pair<int*, double*> p4; // 偏特化：指针类型
        p4.print();             // Pointer Pair

        return 0;
    }
    ```

## 类模板与继承

### 从普通类继承

类模板可以从普通类（非模板）继承。

!!! example "从普通类继承"

    ``` cpp linenums="1"
    class Base {
    public:
        Base(int i = 0) : id(i) {}
        void showId() const { cout << "ID: " << id << endl; }
    protected:
        int id;
    };

    template <typename T>
    class Derived : public Base {
    public:
        Derived(int id, const T& v) : Base(id), value(v) {}

        void display() const {
            showId();
            cout << "Value: " << value << endl;
        }
    private:
        T value;
    };

    int main() {
        Derived<int> d(100, 200);
        d.display();   // ID: 100, Value: 200

        Derived<string> d2(101, "Hello");
        d2.display();  // ID: 101, Value: Hello

        return 0;
    }
    ```

### 从类模板继承

基类本身也可以是类模板，或指定特定的实例化版本。

!!! example "从类模板继承"

    ``` cpp linenums="1"
    // 基类模板
    template <typename T>
    class Base {
    public:
        Base(const T& d) : data(d) {}
        T getData() const { return data; }
    protected:
        T data;
    };

    // 派生类也是模板
    template <typename T>
    class Derived : public Base<T> {
    public:
        Derived(const T& d, const T& e) : Base<T>(d), extra(e) {}

        void display() const {
            cout << "Data: " << this->getData() << ", Extra: " << extra << endl;
        }
    private:
        T extra;
    };

    // 派生类指定基类的实例化版本
    class IntDerived : public Base<int> {
    public:
        IntDerived(int d) : Base<int>(d) {}
        void show() const { cout << "Value: " << data << endl; }
    };

    int main() {
        Derived<int> d1(10, 20);
        d1.display();   // Data: 10, Extra: 20

        Derived<string> d2("Hello", "World");
        d2.display();   // Data: Hello, Extra: World

        IntDerived d3(100);
        d3.show();      // Value: 100

        return 0;
    }
    ```

!!! warning "依赖名称查找"

    在派生类模板中访问基类模板的成员时，需要使用 `this->` 或 `Base<T>::` 来限定，否则编译器可能找不到：

    ``` cpp
    template <typename T>
    class Derived : public Base<T> {
    public:
        void func() {
            // data = 10;        // 错误！编译器找不到 data
            this->data = 10;     // 正确
            Base<T>::data = 10;  // 也正确
        }
    };
    ```

## 综合示例

!!! example "动态数组类模板"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cassert>
    using namespace std;

    // 动态数组类模板
    template <typename T>
    class DynamicArray {
    private:
        T* data;
        int size;
        int capacity;

        // 内部扩容函数
        void grow() {
            int newCapacity = capacity * 2;
            T* newData = new T[newCapacity];
            for (int i = 0; i < size; i++) {
                newData[i] = data[i];
            }
            delete[] data;
            data = newData;
            capacity = newCapacity;
        }

    public:
        // 构造函数
        DynamicArray(int initialCapacity = 10)
            : size(0), capacity(initialCapacity) {
            assert(initialCapacity > 0);
            data = new T[capacity];
            cout << "DynamicArray created, capacity: " << capacity << endl;
        }

        // 复制构造函数（深拷贝）
        DynamicArray(const DynamicArray<T>& other)
            : size(other.size), capacity(other.capacity) {
            data = new T[capacity];
            for (int i = 0; i < size; i++) {
                data[i] = other.data[i];
            }
            cout << "DynamicArray copied" << endl;
        }

        // 赋值运算符
        DynamicArray<T>& operator=(const DynamicArray<T>& other) {
            if (this == &other) return *this;

            delete[] data;
            size = other.size;
            capacity = other.capacity;
            data = new T[capacity];
            for (int i = 0; i < size; i++) {
                data[i] = other.data[i];
            }

            cout << "DynamicArray assigned" << endl;
            return *this;
        }

        // 析构函数
        ~DynamicArray() {
            delete[] data;
            cout << "DynamicArray destroyed" << endl;
        }

        // 在末尾添加元素
        void pushBack(const T& value) {
            if (size >= capacity) {
                grow();
            }
            data[size++] = value;
        }

        // 删除末尾元素
        void popBack() {
            assert(size > 0);
            size--;
        }

        // 下标运算符（非 const）
        T& operator[](int index) {
            assert(index >= 0 && index < size);
            return data[index];
        }

        // 下标运算符（const）
        const T& operator[](int index) const {
            assert(index >= 0 && index < size);
            return data[index];
        }

        // 获取大小
        int getSize() const { return size; }

        // 获取容量
        int getCapacity() const { return capacity; }

        // 判断是否为空
        bool isEmpty() const { return size == 0; }

        // 清空
        void clear() {
            size = 0;
        }

        // 友元：输出运算符
        friend ostream& operator<<(ostream& os, const DynamicArray<T>& arr) {
            os << "[";
            for (int i = 0; i < arr.size; i++) {
                os << arr.data[i];
                if (i < arr.size - 1) os << ", ";
            }
            os << "]";
            return os;
        }
    };

    int main() {
        cout << "=== 创建 DynamicArray<int> ===" << endl;
        DynamicArray<int> arr;

        cout << "\n=== 添加元素 ===" << endl;
        for (int i = 1; i <= 15; i++) {
            arr.pushBack(i * 10);
        }

        cout << "Array: " << arr << endl;
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        cout << "\n=== 访问元素 ===" << endl;
        cout << "arr[0] = " << arr[0] << endl;
        cout << "arr[5] = " << arr[5] << endl;
        cout << "arr[10] = " << arr[10] << endl;

        cout << "\n=== 修改元素 ===" << endl;
        arr[0] = 999;
        arr[5] = 888;
        cout << "After modification: " << arr << endl;

        cout << "\n=== 删除元素 ===" << endl;
        arr.popBack();
        arr.popBack();
        arr.popBack();
        cout << "After popBack: " << arr << endl;
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        cout << "\n=== 使用不同数据类型 ===" << endl;
        DynamicArray<string> strArr;
        strArr.pushBack("Hello");
        strArr.pushBack("World");
        strArr.pushBack("C++");
        cout << "String array: " << strArr << endl;

        DynamicArray<double> dblArr;
        dblArr.pushBack(3.14);
        dblArr.pushBack(2.71);
        dblArr.pushBack(1.41);
        cout << "Double array: " << dblArr << endl;

        cout << "\n=== 复制构造 ===" << endl;
        DynamicArray<int> arrCopy = arr;
        cout << "Copy: " << arrCopy << endl;

        return 0;
    }
    ```

## 类模板与函数模板的对比

| 对比项           | 函数模板                | 类模板                             |
| :--------------- | :---------------------- | :--------------------------------- |
| **声明关键字**   | `template <typename T>` | `template <typename T>`            |
| **类型参数推导** | ✓ 支持（隐式/显式）     | ✗ 不支持（必须显式指定）           |
| **实例化时机**   | 调用函数时              | 定义对象时                         |
| **默认模板参数** | ✓ 支持                  | ✓ 支持                             |
| **特化**         | 全特化（不常用）        | 全特化 + 偏特化                    |
| **成员函数定义** | 无需特殊语法            | 类外定义需 `template <typename T>` |
| **静态成员**     | 无（函数没有静态成员）  | 每个实例化版本各有一份             |
| **使用场景**     | 通用算法、工具函数      | 通用数据结构                       |

## 小结

1.  **类模板的定义**：
    1. `template <typename T> class ClassName { ... }`
    2. 类型参数可用于数据成员、成员函数参数、返回类型等。

2.  **成员函数的类外定义**：
    1. 每个成员函数前都要写 `template <typename T>`
    2. 类名要写成 `ClassName<T>` 而非 `ClassName`

3.  **类模板的实例化**：
    1. 必须显式指定模板参数：`ClassName<int> obj`
    2. 可以使用 `using` 或 `typedef` 简化类型名

4.  **类模板的特化**：
    1. **全特化**：为特定类型提供完整实现 `template <> class ClassName<SpecialType>`
    2. **偏特化**：为满足特定条件的类型提供部分特化

5.  **类模板的静态成员**：
    1. 每个实例化版本拥有独立的静态成员副本

6.  **类模板与继承**：
    1. 可以从普通类继承。
    2. 可以从类模板继承（需要指定实例化版本或保持模板特性）。