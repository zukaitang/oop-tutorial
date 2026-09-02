# 练习12：使用对象数组管理商品

---

## 任务描述

设计商品类 `Product`，使用对象数组保存多件商品的信息，并实现输出商品清单和计算商品总价的功能。

本题重点练习对象数组的列表初始化、通过下标访问数组元素，以及将对象数组传递给普通函数。

## 相关知识

### 对象数组的创建

对象数组的每个元素都是一个独立对象。若类提供带参构造函数，可以在列表初始化时为每个元素提供构造参数：

``` cpp
Product products[3] = {
    Product("笔记本", 12.50),
    Product("水杯", 35.00),
    Product("书包", 89.50)
};
```

数组创建时，会依次调用每个元素对应的构造函数。

### 遍历对象数组

使用下标访问对象数组中的元素，再通过 `.` 调用对象的公有成员函数：

``` cpp
for (std::size_t i = 0; i < count; ++i) {
    total += products[i].GetPrice();
}
```

### 将数组传递给函数

数组作为函数参数时通常会退化为指针，因此需要额外传入元素个数。若函数不修改数组元素，应使用 `const` 修饰参数：

``` cpp
double CalculateTotal(const Product products[], std::size_t count);
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Product.h` 中声明 `Product` 类，私有数据成员包括商品名称和单价：

   ```cpp
   std::string name;
   double price;
   ```

1. 声明并实现以下公有成员函数：

   ```cpp
   Product(const std::string& name, double price);
   const std::string& GetName() const;
   double GetPrice() const;
   ```

1. 在 `Product.h` 中声明普通函数：

   ```cpp
   void PrintProducts(const Product products[], std::size_t count);
   double CalculateTotal(const Product products[], std::size_t count);
   ```

1. `PrintProducts` 按“商品名 单价”的格式逐行输出对象数组中的全部商品，单价固定保留两位小数。
2. `CalculateTotal` 返回数组中所有商品单价之和。
3. 在 `main.cpp` 中创建包含 3 个 `Product` 对象的数组，使用 `assert` 验证总价，并捕获 `PrintProducts` 输出后断言其格式。

## 待完成代码

### Product.h

``` cpp
#ifndef PRODUCT_H
#define PRODUCT_H

#include <cstddef>
#include <string>

// TODO：声明 Product 类和两个普通函数

#endif
```

### Product.cpp

``` cpp
#include "Product.h"

// TODO：定义 Product 的成员函数和两个普通函数
```

### main.cpp

``` cpp
#include "Product.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Product products[] = {
        Product("笔记本", 12.50),
        Product("水杯", 35.00),
        Product("书包", 89.50)
    };
    const std::size_t count = sizeof(products) / sizeof(products[0]);

    assert(CalculateTotal(products, count) == 137.00);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    PrintProducts(products, count);
    std::cout.rdbuf(oldBuffer);

    assert(output.str() ==
           "笔记本 12.50\n"
           "水杯 35.00\n"
           "书包 89.50\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

对象数组中共有三件商品，预期总价为 `12.50 + 35.00 + 89.50 = 137.00`。输出应为：

``` text
笔记本 12.50
水杯 35.00
书包 89.50
```

可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Product.cpp main.cpp -o product_array
```

---

开始你的任务吧，祝你成功！
