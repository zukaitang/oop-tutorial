# 继承的基本概念

继承是面向对象程序设计的三大核心特征之一（封装、继承、多态）。它既是实现代码复用的强大工具，也是构建可扩展软件体系结构的基石。

## 从现实世界理解继承

在现实世界中，“继承”无处不在：

!!! example "现实世界的继承关系"

    - **生物分类**：动物是基类，哺乳动物继承自动物，人类继承自哺乳动物。子类拥有父类的全部特征，并发展出自己的独特性。
    - **交通工具**：车辆是基类，汽车和卡车继承自车辆。所有车辆都有车轮和重量，但汽车有载人数，卡车有载重量。
    - **几何图形**：形状是基类，圆形、矩形、三角形继承自形状。所有形状都有面积和周长，但计算方式各不相同。

这些例子揭示了一个核心规律：**子类继承了父类的全部特征，同时拥有自己独特的属性**。这正是面向对象继承思想的现实来源。

!!! quote "继承的哲学本质"

    继承表达的是 **"is-a"（是一种）** 关系。卡车**是一种**车辆，矩形**是一种**形状。这种关系是永久性的、传递性的——如果卡车是车辆，车辆是交通工具，那么卡车也是交通工具。

## 继承的定义：同一过程的两种视角

在面向对象程序设计中，**继承**和**派生**描述的是同一过程，只是观察的角度不同：

!!! info "继承与派生的关系"

    | 视角                    | 定义                                           | 方向                       |
    | :---------------------- | :--------------------------------------------- | :------------------------- |
    | **继承（Inheritance）** | 保持已有类的特性而构造新类的过程               | 自上而下（从基类到派生类） |
    | **派生（Derivation）**  | 在已有类的基础上新增自己的特性而产生新类的过程 | 自下而上（从派生类到基类） |

    - **基类（Base Class）**：被继承的已有类，也称为**父类（Parent Class）**。
    - **派生类（Derived Class）**：派生出的新类，也称为**子类（Child Class）**。
    - **直接基类（Direct Base Class）**：直接参与派生出某类的基类。
    - **间接基类（Indirect Base Class）**：基类的基类，乃至更高层的基类。

!!! example "直接基类与间接基类"

    ```
    Vehicle（车辆）          ← 顶层基类
        ↑
    Car（汽车）              ← Vehicle 的直接派生类，也是 SUV 的直接基类
        ↑
    SUV（运动型多功能车）     ← Car 的直接派生类，Vehicle 的间接派生类
    ```

    在上述继承层次中：
    - `Car` 的直接基类是 `Vehicle`，间接基类不存在（或为 `Object`，取决于语言）。
    - `SUV` 的直接基类是 `Car`，间接基类是 `Vehicle`。

## 继承的目的

### 1. 实现设计与代码的重用

继承最直接的价值是**代码复用**。派生类无需重新编写基类中已有的代码，只需专注于新增或修改的部分。

!!! example "代码复用：Vehicle → Car"

    ``` cpp linenums="1"
    class Vehicle {
    protected:
        int wheels;      // 所有车辆都有车轮
        float weight;    // 所有车辆都有重量
    public:
        Vehicle(int w, float wt) : wheels(w), weight(wt) {}
        void printVehicle() const {
            cout << "Wheels: " << wheels << ", Weight: " << weight << endl;
        }
    };

    // Car 继承 Vehicle，自动拥有 wheels、weight 和 printVehicle()
    class Car : public Vehicle {
    private:
        int passenger;   // 只需新增 Car 特有的属性
    public:
        Car(int w, float wt, int p) : Vehicle(w, wt), passenger(p) {}
        // 无需重写 wheels、weight 的存储逻辑
    };
    ```

!!! success "代码复用的价值"

    - **减少重复代码**：共性的属性和行为只需在基类中定义一次。
    - **降低维护成本**：修改基类中的共性逻辑，所有派生类自动受益。
    - **提高开发效率**：开发者可以站在前人的肩膀上，专注于新功能的开发。

### 2. 实现系统功能的扩展与演化

当新的需求出现，原有程序无法满足时，可以通过派生在现有类型的基础上进行扩展。

!!! example "功能扩展：应对新需求"

    ``` cpp linenums="1"
    // 原有系统：只支持普通汽车
    class Car : public Vehicle {
        // 汽车相关功能
    };

    // 新需求：需要支持自动驾驶汽车
    // 无需修改原有的 Car 类，通过派生扩展即可
    class SelfDrivingCar : public Car {
    private:
        SensorSystem sensors;   // 新增传感器系统
        AIProcessor ai;         // 新增AI处理器
    public:
        void autoDrive() {      // 新增自动驾驶功能
            // 实现自动驾驶逻辑
        }
    };
    ```

!!! note "开闭原则（Open-Closed Principle）"

    继承是实现面向对象设计原则中**开闭原则**的重要方式：
    - **对扩展开放**：通过派生类添加新功能。
    - **对修改关闭**：不修改已有基类的稳定代码。

    这使得软件系统既能适应需求变化，又能保持核心代码的稳定性。

### 3. 建立类型层次结构

继承能够建立类的层次化结构，让不同类型之间形成逻辑关联，便于理解和组织代码。

!!! example "类型层次结构示例"

    ```
                        Shape（形状）
                      /      |      \
               Circle   Rectangle  Triangle
                /          |           \
          Ellipse     Square      RightTriangle
    ```

    这种层次结构：
    - **表达类型关系**：清晰地展示了 "Square is-a Rectangle" 的语义。
    - **支持多态**：可以通过基类指针统一操作所有派生类对象。
    - **便于扩展**：新增形状类只需继承 `Shape`，不影响已有代码。

## 继承对面向对象程序设计的贡献

!!! summary "继承的三大贡献"

    | 贡献         | 说明                                           |
    | :----------- | :--------------------------------------------- |
    | **代码复用** | 避免重复编写相同的代码，提高开发效率和可维护性 |
    | **功能扩展** | 在不修改已有代码的前提下，通过派生类添加新功能 |
    | **多态基础** | 继承是多态的前提，没有继承就无法实现运行时多态 |

!!! info "继承与封装的关系"

    继承不会破坏封装，而是对其的补充：
    - 基类的 `private` 成员在派生类中**不可直接访问**，保护了基类的实现细节。
    - 基类的 `protected` 成员在派生类中**可直接访问**，方便派生类扩展。
    - 这种设计平衡了**复用需求**和**封装保护**之间的矛盾。

!!! tip "何时应该使用继承？"

    只有在满足 **"is-a"（是一种）** 关系时才使用继承：
    - ✅ `Car` 是 `Vehicle` 的一种 → 使用继承 ✓
    - ✅ `Circle` 是 `Shape` 的一种 → 使用继承 ✓
    - ❌ `Car` 有 `Engine` → 应使用组合（has-a），而非继承 ✗

    错误使用继承会导致设计僵化、难以维护。优先考虑组合，只在确实存在 "is-a" 关系时才使用继承。

## 一个完整的继承示例：Vehicle → Car → Truck

以下示例展示了继承如何实现代码复用和功能扩展：

### 基类 Vehicle

``` cpp linenums="1"
#include <iostream>
using namespace std;

class Vehicle {
protected:
    int wheels;      // 车轮数
    float weight;    // 车重
public:
    // 构造函数
    Vehicle(int wh, float wt) : wheels(wh), weight(wt) {
        cout << "新建了一个 Vehicle 对象" << endl;
    }

    // 析构函数
    ~Vehicle() {
        cout << "回收了一个 Vehicle 对象" << endl;
    }

    // 打印车辆基本信息
    void printVehicle() const {
        cout << "车轮个数: " << wheels << endl;
        cout << "车重: " << weight << endl;
    }
};
```

### 派生类 Car

``` cpp linenums="1"
// Car 是 Vehicle 的派生类，继承了 wheels 和 weight
class Car : public Vehicle {
private:
    int passenger;   // 新增属性：载人数
public:
    // 构造函数：调用基类构造函数初始化继承的成员
    Car(int wh, float wt, int pa = 4) : Vehicle(wh, wt), passenger(pa) {
        cout << "新建了一个 Car 对象" << endl;
    }

    ~Car() {
        cout << "回收了一个 Car 对象" << endl;
    }

    // 扩展功能：打印完整信息（包含新增属性）
    void printCar() const {
        cout << "车轮个数: " << wheels << '\t';
        cout << "重量: " << weight << '\t';
        cout << "载人数: " << passenger << endl;
    }
};
```

### 派生类 Truck

``` cpp linenums="1"
// Truck 也是 Vehicle 的派生类
class Truck : public Vehicle {
private:
    int passenger;   // 新增属性：载人数
    float payload;   // 新增属性：载重量
public:
    Truck(int wh, float wt, int pa = 2, float maxload = 10.0)
        : Vehicle(wh, wt), passenger(pa), payload(maxload) {
        cout << "新建了一个 Truck 对象" << endl;
    }

    ~Truck() {
        cout << "回收了一个 Truck 对象" << endl;
    }

    void printTruck() const {
        cout << "车轮个数: " << wheels << '\t';
        cout << "重量: " << weight << '\t';
        cout << "载人数: " << passenger << '\t';
        cout << "载重量: " << payload << endl;
    }
};
```

### 主函数

``` cpp linenums="1"
int main() {
    // 创建基类对象
    Vehicle v(4, 2.0);
    v.printVehicle();

    // 创建派生类 Car 对象
    Car c(4, 1.5);
    c.printCar();

    // 创建派生类 Truck 对象
    Truck t(4, 4.0, 2, 8.0);
    t.printTruck();

    return 0;
}
```

### 运行结果分析

```
新建了一个 Vehicle 对象
车轮个数: 4
车重: 2
新建了一个 Vehicle 对象
新建了一个 Car 对象
车轮个数: 4  重量: 1.5  载人数: 4
新建了一个 Vehicle 对象
新建了一个 Truck 对象
车轮个数: 4  重量: 4  载人数: 2  载重量: 8
回收了一个 Truck 对象
回收了一个 Vehicle 对象
回收了一个 Car 对象
回收了一个 Vehicle 对象
回收了一个 Vehicle 对象
```

!!! note "关键观察"

    - **构造顺序**：先基类（Vehicle），后派生类（Car/Truck）。
    - **析构顺序**：先派生类（Car/Truck），后基类（Vehicle）。
    - **代码复用**：Car 和 Truck 都无需重新定义 `wheels` 和 `weight`。
    - **功能扩展**：Car 增加了 `passenger`，Truck 增加了 `passenger` 和 `payload`。

## 小结

!!! summary "核心要点"

    1. **继承的本质**：保持已有类的特性并构造新类的过程，表达 **"is-a"** 关系。

    2. **继承与派生**：同一过程的两种视角——继承强调"复用已有"，派生强调"新增特性"。

    3. **继承的目的**：
       4. 实现设计与代码的重用。
       5. 实现系统功能的扩展与演化。
       6. 建立类型层次结构，支持多态。

    7. **继承的价值**：
       8. 减少重复代码，降低维护成本。
       9. 支持开闭原则（对扩展开放，对修改关闭）。
       10. 为运行时多态提供基础。

    11. **使用准则**：只有在确实存在 **"is-a"** 关系时才使用继承；否则优先考虑组合。

继承是面向对象程序设计的纽带，它让类与类之间形成有意义的层次关系，使代码既能复用又能扩展。下一部分将深入探讨不同继承方式对成员访问控制的影响。