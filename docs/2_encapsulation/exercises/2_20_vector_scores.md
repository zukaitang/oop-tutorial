# 练习20：使用 vector 管理学生分数

---

## 任务描述

设计分数表类 `ScoreList`，使用 `std::vector<int>` 动态保存任意数量的分数，并实现添加分数、计算平均分和输出全部分数的功能。

## 相关知识

### vector 与动态数组

`std::vector` 是可自动扩容的动态数组容器。与手动使用 `new[]`/`delete[]` 相比，它会自动管理内存，通常更安全方便。

``` cpp
std::vector<int> scores;
scores.push_back(90);
scores.push_back(85);
```

### vector 常用操作

- `push_back(value)`：在末尾添加元素；
- `size()`：获取元素数量；
- `operator[]`：通过下标访问元素。

遍历时可使用范围 `for` 循环：

``` cpp
for (int score : scores) {
    sum += score;
}
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `ScoreList.h` 中声明 `ScoreList` 类，私有成员为 `std::vector<int> scores`。
3. 声明并实现：

   ```cpp
   void Add(int score);
   std::size_t Size() const;
   double Average() const;
   void Print() const;
   ```

1. `Add` 使用 `push_back` 添加分数；`Average` 返回平均分。若列表为空，返回 `0.0`。
2. `Print` 按“分数列表：90 85 76”的格式输出，元素间一个空格，末尾换行。
3. 在 `test` 中添加三项分数，使用 `assert` 验证长度、平均分和输出。

## 待完成代码

### ScoreList.h

``` cpp
#ifndef SCORE_LIST_H
#define SCORE_LIST_H

#include <cstddef>
#include <vector>

// TODO：声明 ScoreList 类

#endif
```

### ScoreList.cpp

``` cpp
#include "ScoreList.h"

// TODO：定义 ScoreList 的成员函数
```

### main.cpp

``` cpp
#include "ScoreList.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    ScoreList scores;
    scores.Add(90);
    scores.Add(85);
    scores.Add(76);

    assert(scores.Size() == 3);
    assert(scores.Average() == (90.0 + 85.0 + 76.0) / 3.0);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    scores.Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "分数列表：90 85 76\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

三项分数为 `90`、`85`、`76`，平均分为 `251 / 3`。预期输出：

``` text
分数列表：90 85 76
```

``` bash
g++ -std=c++11 ScoreList.cpp main.cpp -o vector_scores
```

---

开始你的任务吧，祝你成功！
