# 第一章练习参考答案

本文档汇总“C/C++ 基础语法差异”练习的参考答案，按练习序号排列。

## 1_1_constexpr

``` cpp
constexpr bool isValidScore(int score) {
    return score >= 0 && score <= 100;
}

void test() {
    static_assert(isValidScore(0));
    static_assert(isValidScore(100));
    static_assert(!isValidScore(-1));
    static_assert(!isValidScore(101));

    assert(isValidScore(85));
    assert(!isValidScore(120));
}
```

## 1_2_auto_decltype

``` cpp
auto add(int a, double b) -> decltype(a + b) {
    return a + b;
}

void test() {
    auto result1 = add(3, 2.5);
    auto result2 = add(-10, 0.75);

    // 补充运行结果断言
    assert(result1 == 5.5);
    assert(result2 == 9.25);

    // 验证 auto 推导出的变量类型
    static_assert(std::is_same<decltype(result1), double>::value);
}
```

## 1_3_nullptr

``` cpp
int sumArray(const int* data, std::size_t size) {
    if (data == nullptr || size == 0) {
        return 0;
    }

    int sum = 0;
    for (std::size_t i = 0; i < size; ++i) {
        sum += data[i];
    }

    return sum;
}
```

## 1_4_swap

``` cpp
void swapValues(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

void test() {
    int x = 10;
    int y = 20;
    swapValues(x, y);
    assert(x == 20);
    assert(y == 10);

    int a = -5;
    int b = 0;
    swapValues(a, b);
    assert(a == 0);
    assert(b == -5);

    int same1 = 42;
    int same2 = 42;
    swapValues(same1, same2);
    assert(same1 == 42);
    assert(same2 == 42);
}
```

## 1_5_reference_for_array_element

``` cpp
char& getCharacter(char str[], std::size_t index) {
    assert(str != nullptr);
    assert(index < std::strlen(str));

    return str[index];
}

void test() {
    char text[] = "hello";

    assert(getCharacter(text, 0) == 'h');
    assert(getCharacter(text, 1) == 'e');
    assert(getCharacter(text, 4) == 'o');

    getCharacter(text, 0) = 'H';
    getCharacter(text, 4) = '!';

    assert(text[0] == 'H');
    assert(text[4] == '!');
    assert(std::string(text) == "Hell!");

    char& ch = getCharacter(text, 1);
    ch = 'A';

    assert(text[1] == 'A');
    assert(std::string(text) == "HAll!");
}
```

## 1_6_sum_positive

``` cpp
int sumPositive(const std::vector<int>& values) {
    int sum = 0;

    for (int value : values) {
        if (value > 0) {
            sum += value;
        }
    }

    return sum;
}
```

## 1_7_increase_scores

``` cpp
void increaseScores(std::vector<int>& scores, int increment) {
    for (int& score : scores) {
        score += increment;
    }
}
```

## 1_8_count_long_names

``` cpp
std::size_t countLongNames(
    const std::vector<std::string>& names,
    std::size_t minLength
) {
    std::size_t count = 0;

    for (const auto& name : names) {
        if (name.length() >= minLength) {
            ++count;
        }
    }

    return count;
}
```

## 1_9_traffic_light

``` cpp
bool canPass(TrafficLight light) {
    switch (light) {
    case TrafficLight::Green:
        return true;
    case TrafficLight::Red:
    case TrafficLight::Yellow:
        return false;
    }

    return false;
}
```

## 1_10_course_level

``` cpp
const char* toString(CourseLevel level) {
    switch (level) {
    case CourseLevel::Beginner:
        return "Beginner";
    case CourseLevel::Intermediate:
        return "Intermediate";
    case CourseLevel::Advanced:
        return "Advanced";
    default:
        return "Unknown";
    }
}
```

## 1_11_print_sum

``` cpp
void printSum(std::istream& in, std::ostream& out) {
    int a, b, c;
    in >> a >> b >> c;

    out << "Sum: " << a + b + c << '\n';
}
```

## 1_12_print_greeting

``` cpp
void printGreeting(std::istream& in, std::ostream& out) {
    std::string name;
    std::getline(in, name);

    out << "Hello, " << name << "!\n";
}
```

## 1_13_read_score

``` cpp
bool readScore(std::istream& in, int& score) {
    if (in >> score) {
        return true;
    }

    in.clear();
    in.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    return false;
}
```

## 1_14_namespace_calculate

``` cpp
namespace Geometry {
    double calculate(double radius) {
        return 3.14 * radius * radius;
    }
}

namespace Physics {
    double calculate(double mass) {
        return 9.8 * mass;
    }
}
```

## 1_15_using_is_even

``` cpp
namespace NumberUtils {
    bool isEven(int value) {
        return value % 2 == 0;
    }
}

void test() {
    // 仅引入 NumberUtils 中的 isEven
    using NumberUtils::isEven;

    assert(isEven(0));
    assert(isEven(8));
    assert(isEven(-12));

    assert(!isEven(1));
    assert(!isEven(99));
    assert(!isEven(-7));
}
```
