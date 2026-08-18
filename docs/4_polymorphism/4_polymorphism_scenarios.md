# 多态的运用

多态是面向对象程序设计中最核心、最强大的特性之一。它让同一段代码能够根据对象的实际类型表现出不同的行为，极大地提升了程序的灵活性、可扩展性和可维护性。

## 什么是多态

### 多态的基本概念

**多态（Polymorphism）** 源于希腊语，意为"多种形态"。在面向对象程序设计中，多态指的是**同一个接口，多种实现**，其核心是用同一段代码可以操作
继承层次中的**一组类型**（即Class Hierarchy），并自动调用适合该类型的方法。这使得代码更加通用、灵活和可扩展。

!!! abstract "多态的本质"

    - **接口统一**：所有对象通过相同的接口（基类指针/引用）被访问。
    - **行为差异**：不同对象对同一操作表现出不同的行为。
    - **运行时决定**：具体调用哪个版本的函数，在运行时根据对象的实际类型决定。
    - **关注"做什么"**：调用者只关心"做什么"，不关心"怎么做"。

### 多态的分类

!!! info "编译时多态 vs 运行时多态"

    | 类型           | 实现方式                   | 绑定时机 | 示例                              |
    | :------------- | :------------------------- | :------- | :-------------------------------- |
    | **编译时多态** | 函数重载、运算符重载、模板 | 编译时   | `cout << 10` vs `cout << "hello"` |
    | **运行时多态** | 虚函数、继承               | 运行时   | 通过 `Shape*` 调用 `draw()`       |

    通常所说的"多态"主要指**运行时多态**，即通过虚函数实现的动态绑定机制。

## 多态的应用场景

### 场景一：统一处理异构对象集合

最经典的多态应用场景是将不同类型的对象放入同一个容器中，通过统一的接口进行批量操作。

!!! example "统一处理异构集合"

    ``` cpp linenums="1" hl_lines="7 14 27 40 71-73 77-79 84"
    #include <iostream>
    #include <vector>
    #include <memory>
    using namespace std;

    // 抽象基类
    class Shape {
    public:
        virtual double area() const = 0;
        virtual void draw() const = 0;
        virtual ~Shape() {}
    };

    class Circle : public Shape {
    private:
        double radius;
    public:
        Circle(double r) : radius(r) {}
        virtual double area() const override {
            return 3.14159 * radius * radius;
        }
        virtual void draw() const override {
            cout << "○ Circle (r=" << radius << ", area=" << area() << ")" << endl;
        }
    };

    class Rectangle : public Shape {
    private:
        double w, h;
    public:
        Rectangle(double w, double h) : w(w), h(h) {}
        virtual double area() const override {
            return w * h;
        }
        virtual void draw() const override {
            cout << "▭ Rectangle (" << w << "×" << h << ", area=" << area() << ")" << endl;
        }
    };

    class Triangle : public Shape {
    private:
        double a, b, c;
    public:
        Triangle(double a, double b, double c) : a(a), b(b), c(c) {}
        virtual double area() const override {
            double s = (a + b + c) / 2;
            return sqrt(s * (s - a) * (s - b) * (s - c));
        }
        virtual void draw() const override {
            cout << "△ Triangle (sides=" << a << "," << b << "," << c
                 << ", area=" << area() << ")" << endl;
        }
    };

    int main() {
        // 创建各种形状
        Circle c1(5.0);
        Circle c2(3.0);
        Rectangle r1(4.0, 6.0);
        Triangle t1(3.0, 4.0, 5.0);

        // 使用动态数组管理异构对象集合
        vector<Shape*> shapes;
        shapes.push_back(&c1);
        shapes.push_back(&c2);
        shapes.push_back(&r1);
        shapes.push_back(&t1);

        // 统一处理：遍历并绘制所有形状
        cout << "=== Drawing All Shapes ===" << endl;
        for (const auto& shape : shapes) {
            shape->draw();   // 运行时多态！
        }

        // 统一计算总面积
        double totalArea = 0;
        for (const auto& shape : shapes) {
            totalArea += shape->area();   // 运行时多态！
        }
        cout << "\nTotal Area: " << totalArea << endl;

        // 验证类型兼容
        Shape* p = new Circle(2.0, "purple");
        p->draw();   // 调用 Circle::draw()
        delete p;

        return 0;
    }
    ```

### 场景二：依赖抽象而非具体实现（依赖倒置）

多态是实现**依赖倒置原则（Dependency Inversion Principle）** 的关键工具——高层模块不应依赖低层模块，两者都应依赖抽象。

!!! example "依赖抽象编程"

    ``` cpp linenums="1" hl_lines="6 12 29 39 49 67-69 73-74"
    #include <iostream>
    #include <vector>
    using namespace std;

    // === 抽象接口 ===
    class DataSource {
    public:
        virtual string getData() const = 0;
        virtual ~DataSource() {}
    };

    class DataProcessor {
    public:
        // 依赖抽象：DataProcessor 不关心具体的数据源类型
        void process(const DataSource& source) {
            string data = source.getData();   // 多态调用
            cout << "Processing: " << data << endl;
            // ... 处理数据
        }

        void processAll(const vector<DataSource*>& sources) {
            for (auto src : sources) {
                process(*src);
            }
        }
    };

    // === 具体实现：各种数据源 ===
    class FileSource : public DataSource {
    private:
        string filename;
    public:
        FileSource(const string& f) : filename(f) {}
        virtual string getData() const override {
            return "Data from file: " + filename;
        }
    };

    class DatabaseSource : public DataSource {
    private:
        string query;
    public:
        DatabaseSource(const string& q) : query(q) {}
        virtual string getData() const override {
            return "Data from database: " + query;
        }
    };

    class NetworkSource : public DataSource {
    private:
        string url;
    public:
        NetworkSource(const string& u) : url(u) {}
        virtual string getData() const override {
            return "Data from network: " + url;
        }
    };

    int main() {
        FileSource file("data.txt");
        DatabaseSource db("SELECT * FROM users");
        NetworkSource net("https://api.example.com");

        DataProcessor processor;

        // 统一处理不同类型的数据源
        processor.process(file);
        processor.process(db);
        processor.process(net);

        // 批量处理
        cout << "\n=== Batch Processing ===" << endl;
        vector<DataSource*> sources = {&file, &db, &net};
        processor.processAll(sources);

        return 0;
    }
    ```

!!! success "依赖抽象的优势"

    - **扩展性好**：新增数据源只需实现 `DataSource` 接口，`DataProcessor` 无需修改。
    - **可测试性强**：可以用 `MockDataSource` 替代真实数据源进行单元测试。
    - **模块解耦**：处理器与具体数据源实现解耦，可独立开发和维护。

### 场景三：回调机制与钩子函数

多态常用来实现**回调（Callback）** 和**钩子函数（Hook Function）** ，让框架/库在特定时机调用用户自定义的行为。

!!! example "框架中的钩子函数"

    ``` cpp linenums="1" hl_lines="7 23-32 38 57 78 82"
    #include <iostream>
    #include <chrono>
    #include <thread>
    using namespace std;

    // === 应用框架：定义生命周期接口 ===
    class Application {
    public:
        // 框架定义的生命周期钩子（默认空实现）
        virtual void onStart() {
            // 可选覆盖
        }

        virtual void onTick(int elapsed) {
            // 可选覆盖
        }

        virtual void onStop() {
            // 可选覆盖
        }

        // 框架的核心逻辑（模板方法）
        void run(int seconds) {
            onStart();   // 钩子：启动时调用

            for (int i = 0; i < seconds; i++) {
                onTick(i);   // 钩子：每个周期调用
                this_thread::sleep_for(chrono::seconds(1));
            }

            onStop();    // 钩子：结束时调用
        }

        virtual ~Application() {}
    };

    // === 用户自定义应用 ===
    class TimerApp : public Application {
    private:
        int count = 0;
    public:
        virtual void onStart() override {
            cout << "⏱️ Timer started!" << endl;
            count = 0;
        }

        virtual void onTick(int elapsed) override {
            count++;
            cout << "  Tick " << elapsed + 1 << ": " << count << " seconds elapsed" << endl;
        }

        virtual void onStop() override {
            cout << "⏱️ Timer stopped! Total: " << count << " seconds" << endl;
        }
    };

    class CounterApp : public Application {
    public:
        virtual void onStart() override {
            cout << "🔢 Counter started!" << endl;
        }

        virtual void onTick(int elapsed) override {
            for (int i = 0; i <= elapsed; i++) {
                cout << ".";
            }
            cout << " " << elapsed + 1 << endl;
        }

        virtual void onStop() override {
            cout << "🔢 Counter finished!" << endl;
        }
    };

    int main() {
        TimerApp timer;
        cout << "--- Running TimerApp ---" << endl;
        timer.run(5);

        cout << "\n--- Running CounterApp ---" << endl;
        CounterApp counter;
        counter.run(8);

        return 0;
    }
    ```

!!! note "设计模式中的应用"

    这种"框架定义钩子，用户实现钩子"的模式，是**模板方法模式（Template Method Pattern）** 的典型应用。框架（Application）定义了算法的骨架（`run` 函数），将具体步骤留给子类实现。

### 场景四：策略模式中的多态

多态是实现**策略模式（Strategy Pattern）** 的基础——将算法封装在独立的类中，运行时动态选择不同的算法策略。

!!! example "策略模式：不同排序算法"

    ``` cpp linenums="1" hl_lines="7 15 32 59 70"
    #include <iostream>
    #include <vector>
    #include <algorithm>
    using namespace std;

    // === 策略接口 ===
    class SortingStrategy {
    public:
        virtual void sort(vector<int>& data) const = 0;
        virtual string getName() const = 0;
        virtual ~SortingStrategy() {}
    };

    // === 具体策略：冒泡排序 ===
    class BubbleSort : public SortingStrategy {
    public:
        virtual void sort(vector<int>& data) const override {
            for (size_t i = 0; i < data.size() - 1; i++) {
                for (size_t j = 0; j < data.size() - i - 1; j++) {
                    if (data[j] > data[j + 1]) {
                        swap(data[j], data[j + 1]);
                    }
                }
            }
        }
        virtual string getName() const override {
            return "Bubble Sort";
        }
    };

    // === 具体策略：快速排序 ===
    class QuickSort : public SortingStrategy {
    public:
        virtual void sort(vector<int>& data) const override {
            quicksort(data, 0, data.size() - 1);
        }
        virtual string getName() const override {
            return "Quick Sort";
        }
    private:
        void quicksort(vector<int>& data, int left, int right) const {
            if (left >= right) return;
            int pivot = data[(left + right) / 2];
            int i = left, j = right;
            while (i <= j) {
                while (data[i] < pivot) i++;
                while (data[j] > pivot) j--;
                if (i <= j) {
                    swap(data[i], data[j]);
                    i++; j--;
                }
            }
            quicksort(data, left, j);
            quicksort(data, i, right);
        }
    };

    // === 具体策略：标准库排序 ===
    class StdSort : public SortingStrategy {
    public:
        virtual void sort(vector<int>& data) const override {
            std::sort(data.begin(), data.end());
        }
        virtual string getName() const override {
            return "Standard Sort";
        }
    };

    // === 使用策略的上下文 ===
    class Sorter {
    public:
        void setStrategy(const SortingStrategy& s) {
            strategy = &s;
        }

        void sortAndPrint(vector<int> data) const {
            if (!strategy) {
                cout << "No strategy set!" << endl;
                return;
            }
            cout << "Using: " << strategy->getName() << endl;
            cout << "  Before: ";
            for (int x : data) cout << x << " ";
            cout << endl;

            strategy->sort(data);

            cout << "  After:  ";
            for (int x : data) cout << x << " ";
            cout << endl;
        }
    private:
        const SortingStrategy* strategy = nullptr;
    };

    int main() {
        vector<int> data = {64, 34, 25, 12, 22, 11, 90};

        BubbleSort bubble;
        QuickSort quick;
        StdSort standard;

        Sorter sorter;

        sorter.setStrategy(bubble);
        sorter.sortAndPrint(data);

        sorter.setStrategy(quick);
        sorter.sortAndPrint(data);

        sorter.setStrategy(standard);
        sorter.sortAndPrint(data);

        return 0;
    }
    ```

### 场景五：工厂模式中的多态

多态是实现**工厂模式（Factory Pattern）** 的关键——将对象的创建逻辑封装在工厂中，运行时决定创建哪种类型的对象。

!!! example "工厂模式：创建不同类型的日志记录器"

    ``` cpp linenums="1" hl_lines="7 14 22 33 44 51 58 68 79"
    #include <iostream>
    #include <string>
    #include <memory>
    using namespace std;

    // === 抽象产品 ===
    class Logger {
    public:
        virtual void log(const string& message) = 0;
        virtual ~Logger() {}
    };

    // === 具体产品：控制台日志 ===
    class ConsoleLogger : public Logger {
    public:
        virtual void log(const string& message) override {
            cout << "[CONSOLE] " << message << endl;
        }
    };

    // === 具体产品：文件日志 ===
    class FileLogger : public Logger {
    private:
        string filename;
    public:
        FileLogger(const string& f) : filename(f) {}
        virtual void log(const string& message) override {
            cout << "[FILE: " << filename << "] " << message << endl;
        }
    };

    // === 具体产品：远程日志 ===
    class RemoteLogger : public Logger {
    private:
        string endpoint;
    public:
        RemoteLogger(const string& e) : endpoint(e) {}
        virtual void log(const string& message) override {
            cout << "[REMOTE: " << endpoint << "] " << message << endl;
        }
    };

    // === 抽象工厂 ===
    class LoggerFactory {
    public:
        virtual unique_ptr<Logger> createLogger() const = 0;
        virtual ~LoggerFactory() {}
    };

    // === 具体工厂 ===
    class ConsoleLoggerFactory : public LoggerFactory {
    public:
        virtual unique_ptr<Logger> createLogger() const override {
            return make_unique<ConsoleLogger>();
        }
    };

    class FileLoggerFactory : public LoggerFactory {
    private:
        string filename;
    public:
        FileLoggerFactory(const string& f) : filename(f) {}
        virtual unique_ptr<Logger> createLogger() const override {
            return make_unique<FileLogger>(filename);
        }
    };

    class RemoteLoggerFactory : public LoggerFactory {
    private:
        string endpoint;
    public:
        RemoteLoggerFactory(const string& e) : endpoint(e) {}
        virtual unique_ptr<Logger> createLogger() const override {
            return make_unique<RemoteLogger>(endpoint);
        }
    };

    // === 使用工厂 ===
    class Application {
    public:
        void run(const LoggerFactory& factory) {
            auto logger = factory.createLogger();   // 多态创建
            logger->log("Application started");
            logger->log("Processing data...");
            logger->log("Application finished");
        }
    };

    int main() {
        Application app;

        cout << "=== Console Mode ===" << endl;
        ConsoleLoggerFactory consoleFactory;
        app.run(consoleFactory);

        cout << "\n=== File Mode ===" << endl;
        FileLoggerFactory fileFactory("app.log");
        app.run(fileFactory);

        cout << "\n=== Remote Mode ===" << endl;
        RemoteLoggerFactory remoteFactory("https://log.example.com");
        app.run(remoteFactory);

        return 0;
    }
    ```

!!! success "工厂模式的价值"

    - **解耦**：客户端代码不依赖具体产品类，只依赖抽象接口。
    - **可扩展**：新增日志类型只需新增对应的产品类和工厂类，无需修改现有代码。
    - **集中管理**：对象的创建逻辑集中，便于维护和控制。

## 多态的扩展性与可维护性

### 新增类型的对比

多态最大的优势在于**无需修改已有代码即可扩展系统**。

!!! example "多态带来的扩展性"

    ``` cpp linenums="1"
    // 假设系统已有 Shape 层次
    // 现在需要新增菱形 Diamond

    class Diamond : public Shape {
    private:
        double d1, d2;
    public:
        Diamond(double d1, double d2) : d1(d1), d2(d2) {}
        virtual double area() const override { return d1 * d2 / 2; }
        virtual void draw() const override {
            cout << "◇ Diamond (diagonals=" << d1 << "," << d2 << ")" << endl;
        }
    };

    int main() {
        // 现有代码无需任何修改即可支持新类型
        vector<unique_ptr<Shape>> shapes;
        shapes.push_back(make_unique<Diamond>(4.0, 6.0));

        for (const auto& s : shapes) {
            s->draw();   // 自动支持 Diamond！
        }
        return 0;
    }
    ```

!!! success "开闭原则的体现"

    多态天然支持**开闭原则（Open-Closed Principle）** ：

    - **对扩展开放**：可以通过添加新的派生类来扩展系统功能。
    - **对修改关闭**：已有代码（基类、调用代码）无需修改。

### 条件分支 vs 多态

多态是实现**用多态替代条件分支**这一重构手法的核心工具。

!!! danger "条件分支的局限性"

    ``` cpp
    // 不使用多态：到处是条件分支
    void draw(ShapeType type, void* shape) {
        if (type == CIRCLE) {
            drawCircle(*(Circle*)shape);
        } else if (type == RECTANGLE) {
            drawRectangle(*(Rectangle*)shape);
        } else if (type == TRIANGLE) {
            drawTriangle(*(Triangle*)shape);
        }
        // 每次新增类型都要修改这个函数！
    }
    ```

!!! success "多态的优势"

    ``` cpp
    // 使用多态：无需条件分支
    void draw(const Shape& shape) {
        shape.draw();   // 多态自动分发
    }
    // 新增类型无需修改 draw 函数！
    ```

## 多态的运行时开销

!!! warning "多态的代价"

    | 代价类型     | 说明                                                   |
    | :----------- | :----------------------------------------------------- |
    | **内存开销** | 每个对象多一个虚指针（通常 8 字节）                    |
    | **性能开销** | 虚函数调用需要查虚表，比直接调用稍慢                   |
    | **对象大小** | 包含虚函数表的类，对象大小增加一个指针                 |
    | **编译限制** | 虚函数不能内联（内联是编译时展开，虚函数是运行时决定） |

!!! tip "何时使用多态"

    - **应该使用**：需要处理多种类型、需要扩展性、需要运行时动态行为。
    - **可以不使用**：类型单一、性能敏感、简单的小型程序。

## 使用建议

!!! summary "多态使用建议"

    1. **用基类指针/引用操作派生类对象**：这是多态的基本用法。
    2. **基类析构函数声明为 virtual**：避免资源泄漏。
    3. **使用 override 关键字**：让编译器检查是否正确覆盖。
    4. **使用智能指针管理多态对象**：避免手动 delete。
    5. **抽象类定义接口**：在基类中使用纯虚函数定义规范。
    6. **避免在虚函数中使用默认参数**：默认参数是静态绑定的。

## 小结

1.  **多态的本质**：同一个接口，多种实现。通过基类指针/引用调用虚函数，运行时决定调用哪个版本。

2.  **多态的实现条件**：
    - 继承关系。
    - 虚函数机制。
    - 基类指针或引用。
    - 派生类覆盖虚函数。

3.  **主要应用场景**：
    - 统一处理异构对象集合。
    - 依赖抽象编程（依赖倒置）。
    - 框架的回调与钩子函数。
    - 策略模式（动态切换算法）。
    - 工厂模式（动态创建对象）。

4.  **核心价值**：
    - **扩展性**：新增类型无需修改已有代码（开闭原则）。
    - **可维护性**：减少条件分支，代码更清晰。
    - **可重用性**：统一接口处理多种类型。

5.  **代价与权衡**：轻微的性能和内存开销，但对于大多数应用是可接受的。在需要扩展性和灵活性的场景中，多态的收益远大于代价。