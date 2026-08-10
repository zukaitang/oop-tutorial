# 动态数组与vector

动态数组是C++编程中最常用的数据结构之一。理解如何手动管理动态数组，以及如何使用标准库提供的 `vector` 容器，
是掌握C++内存管理和标准库使用的重要环节。

## 从 C 风格数组说起

### C风格数组的局限性

在C++中，传统数组（C风格数组）存在诸多限制：

!!! warning "C风格数组的问题"

    - **大小固定**：数组大小必须在编译时确定，无法根据运行时需求调整。
    - **无越界检查**：访问超出数组范围时编译不会报错，可能导致内存错误。
    - **不能直接赋值**：数组之间不能直接复制或赋值。
    - **退化指针**：数组名在表达式中会自动退化为指针，丢失大小信息。

    ``` cpp linenums="1"
    // C风格数组的问题示例
    int arr[5];           // 大小固定为5，无法扩展
    int arr2[10];

    arr[10] = 0;          // 越界访问，编译通过但运行时可能崩溃

    // arr2 = arr;        // 错误！数组不能直接赋值

    int* p = arr;
    // p 丢失了大小信息，无法知道 arr 有多少个元素

    // 数组作为参数时需要单独传递数组长度信息
    double average1(int arr[], int n) {
        double sum = 0;
        for (int i = 0; i < n; i++) {
            sum += arr[i];
        }
        return sum / n;
    }
    ```

### 对象数组的封装需求

当需要管理对象数组，尤其是大小需要在运行时确定或调整的动态对象数组时，手动封装是解决上述问题的一种方式。

!!! note "动态对象数组的封装目标"

    - 在运行时确定数组大小。
    - 自动管理内存的分配和释放。
    - 提供安全的元素访问（带越界检查）。
    - 提供简洁自然的接口。
    - **支持动态调整大小**。

## 动态数组的手动封装

### 元素类的设计

`Point` 是动态数组中将要存放的元素类型，其结构与之前的例子基本一致.
增加的复制构造函数和`=`运算符重载主要是为了实现该类对象的复制拷贝的功能。

!!! example "数组元素类：Point"

    ``` cpp linenums="1"

    # include <iostream>
    # include <cassert>
    # include <algorithm>

    using namespace std;

    class Point {
    public:
    Point(int x = 0, int y = 0) : x(x), y(y) {
    cout << "Constructor called." << endl;
    }
    ~Point() {
    cout << "Destructor called." << endl;
    }

        Point(const Point& other) : x(other.x), y(other.y) {
            cout << "Copy Constructor called." << endl;
        }

        Point& operator=(const Point& other) {
            cout << "Assignment operator called." << endl;
            if (this != &other) {
                x = other.x;
                y = other.y;
            }
            return *this;
        }

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        int getX() const { return x; }
        int getY() const { return y; }
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
    int x, y;
    };
    ```

### 封装类的设计

下面是 `ArrayOfPoints` 类的完整实现，展示了如何封装动态对象数组，并增加了改变数组大小的功能。

!!! example "动态数组封装类：ArrayOfPoints"

    ``` cpp linenums="43"
    class ArrayOfPoints {
    public:
        // 构造函数：动态分配数组
        ArrayOfPoints(int size = 0) : size(size), capacity(size) {
            points = (size > 0) ? new Point[size] : nullptr;
            cout << "ArrayOfPoints Constructor called. size=" << size << endl;
        }

        // 复制构造函数：深拷贝
        ArrayOfPoints(const ArrayOfPoints& other)
            : size(other.size), capacity(other.capacity) {
            points = new Point[size];
            for (int i = 0; i < size; i++) {
                points[i] = other.points[i];
            }
            cout << "ArrayOfPoints Copy Constructor called." << endl;
        }

        // 赋值运算符：深拷贝
        ArrayOfPoints& operator=(const ArrayOfPoints& other) {
            if (this == &other) {
                return *this;
            }
            delete[] points;
            size = other.size;
            capacity = other.capacity;
            points = new Point[size];
            for (int i = 0; i < size; i++) {
                points[i] = other.points[i];
            }
            cout << "ArrayOfPoints Assignment operator called." << endl;
            return *this;
        }

        // 析构函数：释放动态数组
        ~ArrayOfPoints() {
            cout << "ArrayOfPoints Destructor called." << endl;
            delete[] points;
        }

        // 获取数组大小和容量
        int getSize() const { return size; }
        int getCapacity() const { return capacity; }

        // === 访问数组元素 ===
        // 元素访问：返回引用，支持读写；带越界检查
        Point& element(int index) {
            assert(index >= 0 && index < size);
            return points[index];
        }

        const Point& element(int index) const {
            assert(index >= 0 && index < size);
            return points[index];
        }

        // 重载下标运算符
        Point& operator[](int index) {
            return element(index);
        }

        const Point& operator[](int index) const {
            return element(index);
        }

        // === 改变数组大小 ===
        // 方式一：resize - 改变大小，新元素使用默认构造
        void resize(int newSize) {
            if (newSize == size) {
                return;
            }

            Point* newPoints = new Point[newSize];
            int copyCount = (newSize < size) ? newSize : size;

            // 复制原有元素
            for (int i = 0; i < copyCount; i++) {
                newPoints[i] = points[i];
            }

            // 如果新大小大于旧大小，剩余元素已由 Point 默认构造函数初始化

            delete[] points;
            points = newPoints;
            size = newSize;
            capacity = newSize;
            cout << "Array resized to " << newSize << endl;
        }

        // 方式二：push_back - 在末尾添加元素（自动扩展）
        void push_back(const Point& p) {
            if (size >= capacity) {
                // 需要扩容：通常按 2 倍增长
                int newCapacity = (capacity == 0) ? 1 : capacity * 2;
                reserve(newCapacity);
            }
            points[size++] = p;
            cout << "push_back: size=" << size << ", capacity=" << capacity << endl;
        }

        // 方式三：pop_back - 删除末尾元素
        void pop_back() {
            assert(size > 0);
            size--;
            cout << "pop_back: size=" << size << ", capacity=" << capacity << endl;
        }

        // 方式四：reserve - 预留容量
        void reserve(int newCapacity) {
            if (newCapacity <= capacity) {
                return;
            }

            Point* newPoints = new Point[newCapacity];
            for (int i = 0; i < size; i++) {
                newPoints[i] = points[i];
            }

            delete[] points;
            points = newPoints;
            capacity = newCapacity;
            cout << "Reserved capacity to " << newCapacity << endl;
        }

    private:
        Point* points;    // 指向动态数组首地址
        int size;         // 实际元素个数
        int capacity;     // 当前容量
    };
    ```

#### ArrayOfPoints 改变大小操作总结

!!! note "ArrayOfPoints 的动态调整操作"

    | 操作         | 函数           | 说明                         | 对容量影响                  |
    | :----------- | :------------- | :--------------------------- | :-------------------------- |
    | **改变大小** | `resize(n)`    | 将大小改为 n，新元素默认构造 | capacity = n                |
    | **添加元素** | `push_back(p)` | 在末尾添加元素，自动扩容     | capacity 不足时翻倍         |
    | **删除末尾** | `pop_back()`   | 删除末尾元素                 | capacity 不变               |
    | **预留容量** | `reserve(n)`   | 预分配容量，避免多次扩容     | capacity = max(capacity, n) |

### 封装类的使用

!!! example "ArrayOfPoints 的使用样例"

    ``` cpp linenums="1"
    int main() {
        cout << "=== 创建初始数组 size=2 ===" << endl;
        ArrayOfPoints arr(2);

        cout << "\n=== 使用 element 函数访问数组内容 ===" << endl;
        arr.element(0).move(5, 10);
        arr.element(1).move(15, 20);

        cout << "\n=== 使用下标运算符访问数组内容 ===" << endl;
        for (int i = 0; i < arr.getSize(); i++) {
            cout << "arr[" << i << "]: ";
            arr[i].show();
        }

        // 越界访问会被 assert 拦截（调试模式下）
        // arr[2].show();  // 断言失败

        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;
        

        // 1. 使用 resize 扩大数组
        cout << "\n=== resize(5) 扩大数组 ===" << endl;
        arr.resize(5);
        arr[2].move(25, 30);
        arr[3].move(35, 40);
        arr[4].move(45, 50);

        cout << "\n=== 扩大后的数组内容 ===" << endl;
        for (int i = 0; i < arr.getSize(); i++) {
            cout << "arr[" << i << "]: ";
            arr[i].show();
        }
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        // 2. 使用 resize 缩小数组
        cout << "\n=== resize(3) 缩小数组 ===" << endl;
        arr.resize(3);

        cout << "\n=== 缩小后的数组内容 ===" << endl;
        for (int i = 0; i < arr.getSize(); i++) {
            cout << "arr[" << i << "]: ";
            arr[i].show();
        }
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        // 3. 使用 push_back 添加元素（自动扩容）
        cout << "\n=== push_back 添加元素 ===" << endl;
        arr.push_back(Point(55, 60));
        arr.push_back(Point(65, 70));
        arr.push_back(Point(75, 80));

        cout << "\n=== push_back 后的数组内容 ===" << endl;
        for (int i = 0; i < arr.getSize(); i++) {
            cout << "arr[" << i << "]: ";
            arr[i].show();
        }
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        // 4. 使用 pop_back 删除末尾元素
        cout << "\n=== pop_back 删除末尾元素 ===" << endl;
        arr.pop_back();
        arr.pop_back();

        cout << "\n=== pop_back 后的数组内容 ===" << endl;
        for (int i = 0; i < arr.getSize(); i++) {
            cout << "arr[" << i << "]: ";
            arr[i].show();
        }
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        // 5. 使用 reserve 预留容量
        cout << "\n=== reserve(10) 预留容量 ===" << endl;
        arr.reserve(10);
        cout << "Size: " << arr.getSize() << ", Capacity: " << arr.getCapacity() << endl;

        // arr 离开作用域，析构函数自动释放内存
        cout << "\n=== 程序结束，开始析构 ===" << endl;
        return 0;
    }
    ```

    运行结果

    ``` linenums="1"
    === 创建初始数组 size=2 ===
    Constructor called.
    Constructor called.
    ArrayOfPoints Constructor called. size=2

    === 使用下标运算符访问数组内容 ===
    arr[0]: Point(5, 10)
    arr[1]: Point(15, 20)
    Size: 2, Capacity: 2

    === resize(5) 扩大数组 ===
    Constructor called.    // 新分配的 5 个元素
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Assignment operator called.  // 复制原有元素
    Assignment operator called.
    Array resized to 5
    Destructor called.    // 释放旧数组
    Destructor called.

    === 扩大后的数组内容 ===
    arr[0]: Point(5, 10)
    arr[1]: Point(15, 20)
    arr[2]: Point(25, 30)
    arr[3]: Point(35, 40)
    arr[4]: Point(45, 50)
    Size: 5, Capacity: 5

    === resize(3) 缩小数组 ===
    Constructor called.    // 新分配的 3 个元素
    Constructor called.
    Constructor called.
    Assignment operator called.  // 复制前 3 个元素
    Assignment operator called.
    Assignment operator called.
    Array resized to 3
    Destructor called.    // 释放旧数组（5个元素，其中2个被销毁）
    Destructor called.
    Destructor called.
    Destructor called.
    Destructor called.

    === 缩小后的数组内容 ===
    arr[0]: Point(5, 10)
    arr[1]: Point(15, 20)
    arr[2]: Point(25, 30)
    Size: 3, Capacity: 3

    === push_back 添加元素 ===
    Copy Constructor called.    // 复制 Point(55,60) 作为参数
    push_back: size=4, capacity=6
    Copy Constructor called.
    push_back: size=5, capacity=6
    Copy Constructor called.
    push_back: size=6, capacity=6
    Destructor called.   // 销毁临时对象
    Destructor called.
    Destructor called.

    === push_back 后的数组内容 ===
    arr[0]: Point(5, 10)
    arr[1]: Point(15, 20)
    arr[2]: Point(25, 30)
    arr[3]: Point(55, 60)
    arr[4]: Point(65, 70)
    arr[5]: Point(75, 80)
    Size: 6, Capacity: 6

    === pop_back 删除末尾元素 ===
    pop_back: size=5, capacity=6
    pop_back: size=4, capacity=6

    === pop_back 后的数组内容 ===
    arr[0]: Point(5, 10)
    arr[1]: Point(15, 20)
    arr[2]: Point(25, 30)
    arr[3]: Point(55, 60)
    Size: 4, Capacity: 6

    === reserve(10) 预留容量 ===
    Constructor called.    // 新分配的 10 个元素
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Constructor called.
    Assignment operator called.  // 复制原有 4 个元素
    Assignment operator called.
    Assignment operator called.
    Assignment operator called.
    Reserved capacity to 10
    Destructor called.    // 释放旧数组（6个元素）
    Destructor called.
    Destructor called.
    Destructor called.
    Destructor called.
    Destructor called.
    Destructor called.
    Size: 4, Capacity: 10

    === 程序结束，开始析构 ===
    ArrayOfPoints Destructor called.
    Destructor called.    // 释放 4 个元素
    Destructor called.
    Destructor called.
    Destructor called.
    ```

### 封装设计的优点

!!! success "ArrayOfPoints 的优点"

    - **自动内存管理**：构造函数分配内存，析构函数释放内存，无需用户手动 `delete`。
    - **运行时确定大小**：数组大小通过构造函数参数传入，支持动态确定。
    - **越界检查**：`element()` 函数使用 `assert` 检查下标合法性，防止内存越界。
    - **自然接口**：重载 `operator[]` 使得访问方式与原生数组一致。
    - **常版本支持**：提供 `const` 版本的访问函数，支持常对象的使用。
    - **支持动态扩展**：数组大小可以根据需要动态增长。

### 封装的局限性

!!! warning "手动封装的不足"

    - **只能管理一种类型**：`ArrayOfPoints` 只适用于 `Point` 类型，无法通用。为适配不同类型可能需要编写大量相似代码。
    - **复制问题**：必须实现深拷贝（见前一节“浅拷贝与深拷贝”）。
    - **不支持迭代器**：无法使用标准库算法。
    - **功能有限**：缺少插入、删除、排序等常用操作。
    - **实现较为复杂**：动态控制逻辑的编写需要一定经验。

## 标准库容器：vector

### vector 简介

`vector` 是 C++ 标准模板库（STL）中最常用的容器之一，它封装了动态数组，提供了安全、高效、通用的数组管理功能。

!!! info "vector 的特点"

    - **通用性**：可以存储任何类型的元素（通过模板实现）。
    - **动态扩展**：可以自动增长容量，无需手动管理内存。
    - **安全访问**：提供 `at()` 方法进行越界检查。
    - **丰富的接口**：支持迭代器、插入、删除、排序等操作。
    - **内存管理**：自动管理内存分配和释放。

### vector 的基本使用

!!! example "vector 的基本操作"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>   // 必须包含 vector 头文件
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {
            cout << "Constructor called." << endl;
        }
        Point(const Point& other) : x(other.x), y(other.y) {
            cout << "Copy Constructor called." << endl;
        }
        ~Point() {
            cout << "Destructor called." << endl;
        }

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        int getX() const { return x; }
        int getY() const { return y; }
        void show() const {
            cout << "Point(" << x << ", " << y << ")" << endl;
        }

    private:
        int x, y;
    };

    int main() {

        // 1. 多种初始化方式

        // 创建空的 vector
        vector<Point> points1;

        // 创建指定大小的 vector（元素使用默认构造）
        vector<Point> points2(3);     // 3 个 Point，初始为 (0,0)

        // 创建并初始化
        vector<Point> points3 = {Point(1, 2), Point(3, 4), Point(5, 6)};

        // 预留空间不直接初始化
        cout << "=== 1. 创建 vector 并初始化 ===" << endl;
        vector<Point> v;
        v.reserve(2);   // 预留容量
        v.push_back(Point(5, 10));
        v.push_back(Point(15, 20));

        cout << "\n=== 显示 vector 内容 ===" << endl;
        for (size_t i = 0; i < v.size(); i++) {
            cout << "v[" << i << "]: ";
            v[i].show();
        }
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 2. resize：改变大小
        cout << "\n=== 2. resize(5) 扩大 vector ===" << endl;
        v.resize(5);
        v[2].move(25, 30);
        v[3].move(35, 40);
        v[4].move(45, 50);

        cout << "\n=== 扩大后的 vector 内容 ===" << endl;
        for (size_t i = 0; i < v.size(); i++) {
            cout << "v[" << i << "]: ";
            v[i].show();
        }
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 3. resize 缩小
        cout << "\n=== 3. resize(3) 缩小 vector ===" << endl;
        v.resize(3);

        cout << "\n=== 缩小后的 vector 内容 ===" << endl;
        for (size_t i = 0; i < v.size(); i++) {
            cout << "v[" << i << "]: ";
            v[i].show();
        }
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 4. push_back 自动扩容
        cout << "\n=== 4. push_back 添加元素 ===" << endl;
        v.push_back(Point(55, 60));
        v.push_back(Point(65, 70));
        v.push_back(Point(75, 80));

        cout << "\n=== push_back 后的 vector 内容 ===" << endl;
        for (size_t i = 0; i < v.size(); i++) {
            cout << "v[" << i << "]: ";
            v[i].show();
        }
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 5. pop_back 删除末尾
        cout << "\n=== 5. pop_back 删除末尾元素 ===" << endl;
        v.pop_back();
        v.pop_back();

        cout << "\n=== pop_back 后的 vector 内容 ===" << endl;
        for (size_t i = 0; i < v.size(); i++) {
            cout << "v[" << i << "]: ";
            v[i].show();
        }
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 6. reserve 预留容量
        cout << "\n=== 6. reserve(10) 预留容量 ===" << endl;
        v.reserve(10);
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        // 7. shrink_to_fit 释放多余容量（C++11）
        cout << "\n=== 7. shrink_to_fit 收缩容量 ===" << endl;
        v.shrink_to_fit();
        cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;

        cout << "\n=== 程序结束，vector 自动释放内存 ===" << endl;
        return 0;


        return 0;
    }   // vector 自动释放内存
    ```

### vector 的常用操作

!!! info "vector 常用成员函数"

    | 操作         | 函数                  | 说明                   |
    | :----------- | :-------------------- | :--------------------- |
    | **元素访问** | `at(idx)`             | 带越界检查的访问       |
    |              | `operator[](idx)`     | 下标访问（无越界检查） |
    |              | `front()` / `back()`  | 访问首/尾元素          |
    |              | `data()`              | 获取底层数组指针       |
    | **大小**     | `size()`              | 当前元素个数           |
    |              | `capacity()`          | 当前容量               |
    |              | `empty()`             | 是否为空               |
    |              | `resize(n)`           | 调整大小               |
    | **修改**     | `push_back(val)`      | 在末尾添加元素         |
    |              | `pop_back()`          | 删除末尾元素           |
    |              | `insert(pos, val)`    | 在指定位置插入         |
    |              | `erase(pos)`          | 删除指定位置元素       |
    |              | `clear()`             | 清空所有元素           |
    |              | `reserve(n)`          | 预留容量               |
    | **迭代器**   | `begin()` / `end()`   | 获取迭代器             |
    |              | `rbegin()` / `rend()` | 获取反向迭代器         |

### 元素访问方式对比

!!! example "三种访问方式的比较"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<int> v = {10, 20, 30};

        // 方式一：下标运算符 operator[]（无越界检查，速度快）
        cout << "operator[]: " << v[1] << endl;   // 20
        // v[10] = 100;   // 未定义行为，可能崩溃

        // 方式二：at() 成员函数（有越界检查，安全）
        try {
            cout << "at(): " << v.at(1) << endl;   // 20
            // cout << v.at(10) << endl;   // 抛出 out_of_range 异常
        } catch (const out_of_range& e) {
            cout << "Exception: " << e.what() << endl;
        }

        // 方式三：迭代器（通用方式，支持各种算法）
        for (auto it = v.begin(); it != v.end(); ++it) {
            cout << *it << " ";
        }
        cout << endl;

        return 0;
    }
    ```

### vector 的内存管理

`vector` 自动管理内存，当元素数量超过容量时，会自动重新分配更大的内存空间，并将原有元素移动到新空间。

!!! example "vector 的内存自动扩展"

    ``` cpp linenums="1"
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<int> v;
        
        cout << "Initial size: " << v.size() << ", capacity: " << v.capacity() << endl;

        for (int i = 0; i < 10; i++) {
            v.push_back(i);
            cout << "After push " << i << ": size = " << v.size() 
                 << ", capacity = " << v.capacity() << endl;
        }

        return 0;
    }
    ```

    典型的运行结果（容量增长策略因编译器而异）：

    ```
    Initial size: 0, capacity: 0
    After push 0: size = 1, capacity = 1
    After push 1: size = 2, capacity = 2
    After push 2: size = 3, capacity = 4
    After push 3: size = 4, capacity = 4
    After push 4: size = 5, capacity = 8
    After push 5: size = 6, capacity = 8
    After push 6: size = 7, capacity = 8
    After push 7: size = 8, capacity = 8
    After push 8: size = 9, capacity = 16
    After push 9: size = 10, capacity = 16
    ```

!!! tip "性能建议"

    - 如果预先知道元素数量，使用 `reserve()` 预留容量，避免多次重新分配。
    - 使用 `reserve()` 而不是 `resize()` 来预分配空间（`reserve` 只分配空间，不构造元素）。

## 手动封装 vs vector 的对比

!!! summary "对比总结"

    | 对比项          | 手动封装 (ArrayOfPoints)      | vector                                 |
    | :-------------- | :---------------------------- | :------------------------------------- |
    | **通用性**      | 只能管理特定类型              | 模板化，可管理任何类型                 |
    | **动态扩展**    | 不支持                        | 自动扩展容量                           |
    | **越界检查**    | assert（调试模式）            | at() 抛出异常                          |
    | **内存管理**    | 手动 new/delete               | 自动管理                               |
    | **遍历方式**    | 仅下标                        | 下标、迭代器、范围for                  |
    | **STL算法支持** | 不支持                        | 完全支持                               |
    | **拷贝/赋值**   | 需要自定义深拷贝              | 自动处理（值语义）                     |
    | **改变大小**    | `resize(n)`（capacity = n）   | `resize(n)`（capacity 可能不变或变化） |
    | **添加元素**    | `push_back(p)`（扩容翻倍）    | `push_back(p)`（扩容翻倍）             |
    | **删除末尾**    | `pop_back()`（capacity 不变） | `pop_back()`（capacity 不变）          |
    | **预留容量**    | `reserve(n)`                  | `reserve(n)`                           |
    | **收缩容量**    | 不支持                        | `shrink_to_fit()`（C++11）             |
    | **容量策略**    | 手动实现翻倍扩容              | 标准库实现，通常翻倍                   |
    | **编写成本**    | 高                            | 中等                                   |

!!! tip "使用建议"

    - **优先使用 `vector`**：在绝大多数场景下，应优先使用标准库的 `vector`，它更安全、更高效、功能更丰富。
    - **理解封装原理**：学习手动封装动态数组有助于理解 `vector` 的内部工作原理，是深入学习 C++ 的重要基础。
    - **特殊场景可自定义**：当有特殊需求（如特定的内存分配策略、轻量级容器等）时，才考虑手动实现。

## 小结

1. **动态数组手动封装**是理解内存管理的重要练习，但实际开发中更推荐使用标准库容器。

2. **`vector`** 是 STL 中最常用的容器，提供了动态数组的完整实现：
3. 自动内存管理（构造/析构自动分配和释放）。
4. 动态扩展（自动调整容量）。
5. 安全访问（`at()` 带越界检查）。
6. 丰富的接口（迭代器、插入、删除等）。

7. **`vector` 与数组的对比**：
8. `vector` 比 C 风格数组更安全、更灵活。
9. `vector` 比手动封装的动态数组更通用、功能更丰富。

10. **性能考虑**：
11. 使用 `reserve()` 预分配容量，减少重新分配的开销。
12. `operator[]` 比 `at()` 更快（无越界检查开销）。

13. **最佳实践**：优先使用 `vector`，理解其原理，只在特殊需求下才手动实现动态数组。