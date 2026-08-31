# 编程：使用引用参数交换两个整数

---

## 任务描述

编写函数 `swapValues`，交换两个 `int` 整数变量的值。

函数必须使用**非常量引用**作为参数。调用函数后，实参本身的值应被交换，而不是只交换函数内部副本的值。

例如，若调用前 `x = 10`、`y = 20`，执行 `swapValues(x, y)` 后，应得到 `x = 20`、`y = 10`。

## 相关知识

### 引用：变量的别名

引用是已存在变量的别名，定义时必须初始化。

```cpp
int x = 10;
int& ref = x;

ref = 20;  // 修改 ref，也会修改 x
```

引用一旦绑定到变量后不能重新绑定；对引用赋值是在修改它所绑定的变量。

### 引用作为函数参数

函数参数声明为 `int&` 时，函数可以直接修改调用者传入的变量。

```cpp
void increase(int& value) {
    value += 1;
}

int score = 80;
increase(score);  // score 变为 81
```

与指针参数相比，引用参数在调用时不需要使用取地址运算符：

```cpp
// 指针参数：swap(&x, &y)
// 引用参数：swapValues(x, y)
```

引用不能为空，因此本题不需要进行空指针检查。

### 临时变量

交换两个变量的值通常需要一个临时变量，避免第一个赋值操作覆盖原来的数据。

```cpp
int temp = a;
a = b;
b = temp;
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `swapValues` 的函数体。
3. 函数必须采用以下声明形式：

```cpp
void swapValues(int& a, int& b);
```

4. 不得使用指针参数或取地址运算符调用函数。
5. 不得调用标准库的 `std::swap`。
6. 函数调用后，`a` 与 `b` 所绑定变量的值必须互换。
7. 在 `test` 函数中使用 `assert` 验证交换结果。
8. 在 `main` 函数中调用 `test` 函数，并保留测试通过信息与返回语句。

## 待完成代码

```cpp
#include <cassert>
#include <iostream>

// TODO：使用引用参数完成交换
void swapValues(int& a, int& b) {
    // TODO
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

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

`test` 函数应至少覆盖以下三种情况：

| 测试项 | 调用前 | 函数调用 | 预期结果 |
|---|---|---|---|
| 两个正整数 | `x = 10, y = 20` | `swapValues(x, y)` | `x = 20, y = 10` |
| 负数与零 | `a = -5, b = 0` | `swapValues(a, b)` | `a = 0, b = -5` |
| 两个相同值 | `same1 = 42, same2 = 42` | `swapValues(same1, same2)` | 两者仍为 `42` |

如果函数错误地采用值传递：

```cpp
void swapValues(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}
```

则函数只能交换内部副本，调用者的变量不会改变，第一组 `assert` 测试将失败。

---

开始你的任务吧，祝你成功！
