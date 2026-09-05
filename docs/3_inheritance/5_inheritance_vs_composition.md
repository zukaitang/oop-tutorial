# 组合与继承的对比

继承与组合是面向对象程序设计中实现代码复用的两种核心机制。
两者都能实现代码复用，但各自的适用场景、耦合程度和设计意图截然不同。
理解两者的差异并能在实际设计中做出正确选择，是衡量面向对象设计能力的重要标准。

## 概念回顾

### 继承（Inheritance）

**继承**表达的是 **"is-a"（是一种）** 关系——派生类是基类的一种特殊类型。

- 派生类自动获得基类的全部成员（代码复用）。
- 派生类可以覆盖基类的虚函数（多态支持）。
- 继承关系在编译时确定（静态关系）。

!!! example "继承的典型场景"

    ``` cpp
    class Animal { /* ... */ };
    class Dog : public Animal { /* ... */ };   // Dog is-a Animal

    class Shape { /* ... */ };
    class Circle : public Shape { /* ... */ }; // Circle is-a Shape
    ```

### 组合（Composition）

**组合**表达的是 **"has-a"（有一个）** 关系——一个对象由其他对象组成。

- 整体对象通过成员对象实现功能（功能复用）。
- 成员对象的内部细节对整体不可见（封装性好）。
- 组合关系可以在运行时动态改变（动态关系）。

!!! example "组合的典型场景"

    ``` cpp
    class Engine { /* ... */ };
    class Wheel { /* ... */ };

    class Car {
    private:
        Engine engine;      // Car has-a Engine
        Wheel wheels[4];    // Car has-a Wheels
    };
    ```

## 深入对比

### 耦合度对比

!!! warning "继承是紧耦合"

    ``` cpp
    class Base {
    public:
        virtual void process(int data) {
            // 核心逻辑
        }
    };

    class Derived : public Base {
    public:
        virtual void process(int data) override {
            // 派生类依赖基类的具体实现
            // 基类的任何改动都可能影响派生类
        }
    };
    ```

    - 派生类直接依赖基类的实现细节。
    - 基类的修改可能无意中破坏派生类的行为（**脆弱基类问题**）。
    - 基类的接口变化会传导到所有派生类。

!!! success "组合是松耦合"

    ``` cpp
    class Processor {
    public:
        virtual void process(int data) { /* 核心逻辑 */ }
    };

    class Manager {
    private:
        Processor& processor;   // 只依赖接口，不依赖实现
    public:
        Manager(Processor& p) : processor(p) {}
        void work(int data) {
            processor.process(data);   // 委托
        }
    };
    ```

    - 整体只依赖部分的接口，不依赖其实现。
    - 部分类的内部变化不影响整体。
    - 可以通过接口替换不同的实现。

### 封装性对比

!!! warning "继承可能破坏封装"

    ``` cpp
    class Base {
    protected:
        int data;   // 暴露给派生类
        void helper() { /* ... */ }
    };

    class Derived : public Base {
    public:
        void doSomething() {
            data = 100;       // 直接操作基类内部数据
            helper();         // 直接调用基类内部方法
            // 派生类依赖基类的内部实现细节
        }
    };
    ```

    - `protected` 成员暴露给派生类，形成"白盒复用"。
    - 派生类可能依赖基类的内部实现细节。
    - 基类的内部重构可能破坏派生类。

!!! success "组合保护封装"

    ``` cpp
    class Helper {
    private:
        int data;
    public:
        void setData(int d) { data = d; }
        void process() { /* ... */ }
    };

    class Manager {
    private:
        Helper helper;   // 完全封装，细节不可见
    public:
        void doSomething() {
            helper.setData(100);
            helper.process();
            // 只能通过公开接口访问部分对象
        }
    };
    ```

    - 部分对象的内部细节完全隐藏。
    - 整体只能通过公开接口访问部分对象，形成"黑盒复用"。
    - 部分对象的内部重构不影响整体。

### 灵活性对比

!!! warning "继承是静态的"

    - 继承关系在编译时确定，运行时无法改变。
    - 派生类类型一旦确定，无法替换其基类行为。

    ``` cpp
    // 继承：编译时确定，无法动态改变
    class Car : public Engine {
        // Car 的发动机类型在编译时固定
    };
    ```

!!! success "组合是动态的"

    - 组合关系可以在运行时动态调整。
    - 可以通过依赖注入等方式在运行时替换组成部分。

    ``` cpp
    // 组合：运行时可以替换
    class Car {
    private:
        Engine* engine;   // 可以动态指向不同引擎对象
    public:
        Car(Engine* e) : engine(e) {}
        void replaceEngine(Engine* e) { engine = e; }   // 运行时替换
    };

    int main() {
        ElectricEngine ee;
        GasolineEngine ge;

        Car tesla(&ee);
        Car bmw(&ge);
        // 甚至在运行时替换发动机
        // 继承做不到！
    }
    ```

### 代码复杂度对比

!!! warning "继承层次可能非常复杂"

    ```
           Vehicle
              |
           Car
          /    \
       Sedan   SUV
       /         \
    TeslaSedan  TeslaSUV
    ```

    - 深层继承层次难以理解和维护。
    - 一个类的修改可能影响整个继承树。
    - 多重继承引入二义性。

!!! success "组合层次更扁平"

    ```
    Car
      ├── Engine
      ├── Chassis
      ├── Wheels
      └── Electronics
    ```

    - 层次扁平，容易理解。
    - 各组件独立变化，互不干扰。
    - 可以通过不同组件组合出不同的行为。

## 典型反例与正确设计

### 反例：错误使用继承

!!! danger "反例：用继承表达 has-a 关系"

    ``` cpp
    // ❌ 错误！Car 不是 Engine
    class Car : public Engine {
        // ...
    };

    // ❌ 错误！Person 不是 Address
    class Person : public Address {
        // ...
    };
    ```

### 正确设计对比

!!! example "反例 vs 正确设计"

    === "错误设计"

        ``` cpp
        class Car : public Engine {
        public:
            void start() { /* ... */ }
        };
        ```

        问题：
        - Car 是 Engine？逻辑错误
        - 紧耦合，无法更换发动机
        - 违反了 is-a 关系

    === "正确设计（组合）"

        ``` cpp
        class Car {
        private:
            Engine* engine;   // 组合，可动态更换
        public:
            Car(Engine* e) : engine(e) {}
            void start() {
                engine->start();
            }
            void replaceEngine(Engine* e) {
                engine = e;
            }
        };
        ```

        优点：
        - Car has-a Engine ✓
        - 松耦合
        - 可动态更换发动机

### 经典模式：策略模式

策略模式是组合优于继承的典型范例。

!!! example "策略模式（组合替代继承）"

    ``` cpp
    #include <iostream>
    using namespace std;

    // 策略接口
    class SortStrategy {
    public:
        virtual void sort(int arr[], int n) const = 0;
        virtual ~SortStrategy() {}
    };

    // 具体策略：冒泡排序
    class BubbleSort : public SortStrategy {
    public:
        virtual void sort(int arr[], int n) const override {
            cout << "Using Bubble Sort" << endl;
            // 冒泡排序实现
        }
    };

    // 具体策略：快速排序
    class QuickSort : public SortStrategy {
    public:
        virtual void sort(int arr[], int n) const override {
            cout << "Using Quick Sort" << endl;
            // 快速排序实现
        }
    };

    // 上下文：使用组合持有策略
    class Sorter {
    private:
        const SortStrategy* strategy;   // 组合
    public:
        Sorter(const SortStrategy* s) : strategy(s) {}
        void setStrategy(const SortStrategy* s) { strategy = s; }   // 运行时切换
        void sortData(int arr[], int n) const {
            strategy->sort(arr, n);
        }
    };

    int main() {
        int data[5] = {5, 3, 1, 4, 2};

        BubbleSort bubble;
        QuickSort quick;

        // 使用组合动态切换算法
        Sorter sorter(&bubble);
        sorter.sortData(data, 5);   // Using Bubble Sort

        sorter.setStrategy(&quick);
        sorter.sortData(data, 5);   // Using Quick Sort

        return 0;
    }
    ```

## 组合与继承的对比总结

!!! summary "快速参考表"

    | 对比维度         | 继承                 | 组合                   |
    | :--------------- | :------------------- | :--------------------- |
    | **关系**         | "is-a"（是一种）     | "has-a"（有一个）      |
    | **耦合度**       | 高（紧耦合）         | 低（松耦合）           |
    | **封装性**       | 白盒复用（实现可见） | 黑盒复用（仅接口可见） |
    | **运行时灵活性** | 低（静态确定）       | 高（可动态替换）       |
    | **继承层次**     | 可能很深             | 通常扁平               |
    | **多态支持**     | ✓ 天然支持           | 需额外设计             |
    | **代码复用**     | 自动继承             | 显式委托               |
    | **脆弱基类问题** | 存在                 | 不存在                 |
    | **选择原则**     | 仅当是 is-a 关系时   | 优先选择               |

## 组合与继承的选择

### 选择决策流程

!!! tip "选择继承还是组合的决策树"

    ```
    1. 是否存在 is-a 关系？
       ├── 否 → 使用组合
       └── 是 → 继续

    2. 是否满足 LSP（子类能否替换基类）？
       ├── 否 → 使用组合
       └── 是 → 继续

    3. 基类的接口是否稳定（不易变化）？
       ├── 否 → 使用组合（或谨慎使用继承）
       └── 是 → 继续

    4. 是否需要多态支持？
       ├── 是 → 使用继承
       └── 否 → 组合和继承都可以考虑，优先组合
    ```

### 使用场景对比

在设计类之间的关系时，需要根据语义选择合适的方式。

!!! tip "选择指南"

    | 场景                              | 推荐方式 | 原因                       |
    | :-------------------------------- | :------- | :------------------------- |
    | 类型层次关系（`Dog` 是 `Animal`） | **继承** | 表达 is-a 关系，支持多态   |
    | 包含关系（`Car` 有 `Engine`）     | **组合** | 表达 has-a 关系，松耦合    |
    | 需要运行时替换行为                | **组合** | 继承关系静态确定，无法替换 |
    | 需要代码复用但非 is-a             | **组合** | 使用委托而非继承           |
    | 需要多态处理一组类型              | **继承** | 通过虚函数实现运行时多态   |
    | 不确定该用哪个                    | **组合** | 组合是更安全的选择         |

!!! tip "经典设计原则"

    在软件工程中，有一条重要的设计原则：

    **“优先使用对象组合，而不是类继承”（Favor object composition over class inheritance.）**
    —— Erich Gamma 等，《设计模式》

    组合关系更灵活、耦合度更低，是构建复杂系统的首选方式。
