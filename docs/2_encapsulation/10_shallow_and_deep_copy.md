# 浅拷贝与深拷贝

通过 **对象初始化与生命周期** 一节中关于复制构造函数的介绍能够知道，对象的复制是C++中常见且重要的操作。
在对象复制过程中，理解浅拷贝和深拷贝的区别，是编写健壮C++程序的关键。

## 对象的复制

如果在类定义中没有显式定义复制构造函数和赋值运算符，编译器会**自动生成**缺省复制构造函数，
执行**浅拷贝**（Shallow Copy）**——逐字节复制成员变量的值。

!!! example "缺省复制构造函数"

    ``` cpp
    // 编译器自动生成的缺省复制构造函数（等价于）
    ClassName(const ClassName& other)
        : member1(other.member1),
          member2(other.member2),
          // ... 逐个成员复制
    {}
    ```

!!! abstract "浅拷贝的含义"

    **浅拷贝（Shallow Copy）** 是指简单地复制对象的数据成员值。对于指针类型成员，
    浅拷贝只复制指针本身（即地址值），而不复制指针所指向的内容。

对于简单类（只包含基本类型成员）而言，基于浅拷贝的复制构造函数和赋值运算符已经够用。
但对于管理动态资源的类（如包含指针成员），问题就出现了。浅拷贝会导致多个对象指向同
一块内存，并引发**重复释放（Double Free）**问题。

### 浅拷贝的问题

!!! example "浅拷贝的问题"

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}
        ~Buffer() { delete[] data; }   // 析构时释放资源
    private:
        int* data;
        int sz;
    };

    Buffer b1(10);
    Buffer b2(b1);   // 浅拷贝：b2.data 和 b1.data 指向同一块内存
    // 程序结束时，b1 和 b2 的析构函数都会 delete[] data，导致重复释放！
    ```

!!! danger "浅拷贝导致的三大问题"

    1. **数据共享**：两个对象的 `data` 指针指向同一块内存，修改一个对象会影响另一个对象。
    2. **重复释放（Double Free）**：两个对象析构时都会调用 `delete[] data`，同一块内存被释放两次，导致程序崩溃。
    3. **资源所有权混乱**：无法确定哪个对象拥有这块内存，资源管理变得不可控。

    !!! info "内存状态"

        ```
        浅拷贝前：
        b1.data → [int0][int1]  (堆内存)

        浅拷贝后（缺省复制构造函数）：
        b1.data → [int0][int1]  (堆内存)
        b2.data ──┘  (指向同一块内存)

        析构时：
        b1 析构 → delete[] data → 释放内存
        b2 析构 → delete[] data → 重复释放！(崩溃)
        ```

## 深拷贝

### 深拷贝的定义

**深拷贝（Deep Copy）** 不仅复制指针本身，还会**复制指针所指向的内容**。深拷贝确保每个对象拥有自己独立的资源副本。

!!! abstract "深拷贝的核心思想"

    深拷贝为每个对象**独立分配**资源，并**复制内容**，使得各对象的资源互不影响。

### 深拷贝的实现

对于管理动态资源的类，需要自定义复制构造函数和赋值运算符，实现深拷贝——为新对象独立分配资源并复制内容。。

!!! example "解决方案：自定义深拷贝"

    ``` cpp linenums="1"
    class Buffer {
    public:
        Buffer(int size) : data(new int[size]), sz(size) {}

        // 深拷贝构造函数
        Buffer(const Buffer& other) : data(new int[other.sz]), sz(other.sz) {
            for (int i = 0; i < sz; ++i) {
                data[i] = other.data[i];   // 逐元素复制
            }
        }

        ~Buffer() { delete[] data; }
    private:
        int* data;
        int sz;
    };
    ```

!!! info "深拷贝的效果"

    - **数据独立**：两个对象拥有各自的动态内存，互不影响。
    - **安全析构**：每块内存只被释放一次，不会重复释放。
    - **清晰的所有权**：每个对象独立管理自己的资源。

    !!! info "内存状态"

        ```
        深拷贝后（自定义复制构造函数）：
        b1.points → [int0][int1]  (堆内存1)
        b2.points → [int0][int1]  (堆内存2，独立分配)

        析构时：
        b1 析构 → delete[] data → 释放堆内存1
        b2 析构 → delete[] data → 释放堆内存2
        (安全！每块内存只释放一次)
        ```

### 完整的深拷贝类实现 (TODO)

!!! example "遵循五法则的完整实现"

    ``` cpp linenums="1"
    #include <iostream>
    #include <cassert>
    #include <algorithm>
    using namespace std;

    class Point {
    public:
        Point(int x = 0, int y = 0) : x(x), y(y) {}
        Point(const Point& other) = default;
        ~Point() = default;

        void move(int dx, int dy) {
            x += dx;
            y += dy;
        }

        int getX() const { return x; }
        int getY() const { return y; }

        // 赋值运算符（使用缺省版本）
        Point& operator=(const Point& other) = default;

    private:
        int x, y;
    };

    class ArrayOfPoints {
    public:
        // 构造函数
        ArrayOfPoints(int size) : size(size) {
            points = new Point[size];
        }

        // 深拷贝构造函数
        ArrayOfPoints(const ArrayOfPoints& other)
            : size(other.size) {
            points = new Point[size];
            for (int i = 0; i < size; i++) {
                points[i] = other.points[i];
            }
        }

        // 深拷贝赋值运算符
        ArrayOfPoints& operator=(const ArrayOfPoints& other) {
            // 1. 检查自赋值
            if (this == &other) {
                return *this;
            }

            // 2. 释放原有资源
            delete[] points;

            // 3. 分配新资源并复制内容
            size = other.size;
            points = new Point[size];
            for (int i = 0; i < size; i++) {
                points[i] = other.points[i];
            }

            return *this;
        }

        // 析构函数
        ~ArrayOfPoints() {
            delete[] points;
        }

        // 移动构造函数（C++11）
        ArrayOfPoints(ArrayOfPoints&& other) noexcept
            : points(other.points), size(other.size) {
            other.points = nullptr;
            other.size = 0;
        }

        // 移动赋值运算符（C++11）
        ArrayOfPoints& operator=(ArrayOfPoints&& other) noexcept {
            if (this == &other) {
                return *this;
            }

            delete[] points;

            points = other.points;
            size = other.size;

            other.points = nullptr;
            other.size = 0;

            return *this;
        }

        Point& element(int index) {
            assert(index >= 0 && index < size);
            return points[index];
        }

        const Point& element(int index) const {
            assert(index >= 0 && index < size);
            return points[index];
        }

        int getSize() const { return size; }

    private:
        Point* points;
        int size;
    };

    int main() {
        ArrayOfPoints arr1(2);
        arr1.element(0).move(10, 20);

        ArrayOfPoints arr2(3);
        arr2 = arr1;   // 调用深拷贝赋值运算符

        cout << "arr1[0]: (" << arr1.element(0).getX() << ", "
             << arr1.element(0).getY() << ")" << endl;
        cout << "arr2[0]: (" << arr2.element(0).getX() << ", "
             << arr2.element(0).getY() << ")" << endl;

        return 0;
    }
    ```

## 对比：浅拷贝 vs 深拷贝

| 对比项       | **浅拷贝**                       | **深拷贝**                     |
| :----------- | :------------------------------- | :----------------------------- |
| **指针处理** | 只复制指针值（地址）             | 复制指针所指向的内容           |
| **资源分配** | 不分配新资源                     | 独立分配新资源                 |
| **数据共享** | 多个对象共享同一份数据           | 每个对象拥有独立的数据副本     |
| **相互影响** | 修改一个对象会影响另一个         | 各对象独立，互不影响           |
| **析构安全** | 可能导致重复释放                 | 安全，每块内存只释放一次       |
| **实现方式** | 编译器缺省生成（或手动简单复制） | 需要自定义复制控制成员         |
| **适用场景** | 类不管理动态资源                 | 类管理动态资源（指针、句柄等） |

!!! tip "选择指南"

    - **不需要自定义析构函数的类**：通常也不需要自定义复制控制成员，使用缺省版本即可。
    - **需要自定义析构函数的类**：通常也需要自定义复制构造函数和赋值运算符（深拷贝），遵循三/五法则。

!!! abstract "三/五法则（Rule of Three/Five）"

    - **三法则（C++98/03）**：如果类需要自定义析构函数，则通常也需要自定义复制构造函数和复制赋值运算符。
    - **五法则（C++11）**：在三法则的基础上，增加移动构造函数和移动赋值运算符。

    这是因为需要自定义析构函数通常意味着类管理着某种资源，而资源的正确管理需要复制/移动操作也遵循相同的语义。

## 小结

1. **浅拷贝**是编译器生成的缺省行为，逐成员复制数据。对于指针成员，只复制地址值。

2. **浅拷贝的问题**：
3. 多个对象共享同一块动态内存。
4. 修改一个对象会影响其他对象。
5. 析构时重复释放内存，导致程序崩溃。

6. **深拷贝**为每个对象独立分配资源，复制内容而非地址，确保各对象互不影响。

7. **实现深拷贝**需要自定义复制构造函数和赋值运算符：
8. 分配新内存。
9. 复制源对象的内容。
10. 处理自赋值情况。

11. **三/五法则**：需要自定义析构函数的类，通常也需要自定义复制控制成员。

12. **使用智能指针**（如 `shared_ptr`、`unique_ptr`）可以避免手动管理深拷贝的复杂性，由标准库自动管理资源。