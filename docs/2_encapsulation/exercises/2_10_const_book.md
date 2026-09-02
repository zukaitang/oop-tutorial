# 练习10：使用 const 保护图书成员

---

## 任务描述

设计图书类 `Book`。将不修改对象状态的成员函数声明为 `const`，并通过常量对象与常量引用安全地读取图书信息。

## 相关知识

### const 成员函数

在成员函数参数列表后添加 `const`，表示该函数不会修改对象的数据成员：

``` cpp
double Book::GetPrice() const {
    return price;
}
```

常量对象只能调用 `const` 成员函数。因此，用于读取信息的 `GetTitle`、`GetPrice` 和 `Print` 都应声明为 `const`。

### const 对象与 const 引用

``` cpp
const Book book("C++ 程序设计", 68.50);
std::cout << book.GetPrice();
```

将对象作为 `const Book&` 参数传递，既避免复制对象，也保证函数不会修改原对象。

## 编程要求

1. 在 `Book.h` 中声明 `Book` 类，私有成员为图书标题 `title` 和价格 `price`。
2. 声明构造函数以及以下 `const` 成员函数：

   ```cpp
   Book(const std::string& title, double price);
   const std::string& GetTitle() const;
   double GetPrice() const;
   void Print() const;
   ```

1. 在 `Book.h` 中声明普通函数 `void PrintBook(const Book& book);`。
2. 在 `Book.cpp` 中完成所有函数定义。`Print` 按“书名 标题\n价格 数值”的格式输出，价格保留两位小数；`PrintBook` 调用 `book.Print()`。
3. 在 `test` 中创建 `const Book` 对象，并使用 `assert` 验证 getter 和输出结果。

## 待完成代码

### Book.h

``` cpp
#ifndef BOOK_H
#define BOOK_H

#include <string>

// TODO：声明 Book 类和 PrintBook 函数

#endif
```

### Book.cpp

``` cpp
#include "Book.h"

// TODO：定义所有函数
```

### main.cpp

``` cpp
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

## 测试说明

测试使用常量 `Book` 对象，因此若读取成员函数缺少 `const`，程序将无法编译。预期输出为：

``` text
书名 C++ 程序设计
价格 68.50
```

``` bash
g++ -std=c++11 Book.cpp main.cpp -o const_book
```

---

开始你的任务吧，祝你成功！
