# 练习7：使用范围 for 修改全部成绩

---

## 任务描述

编写函数 `increaseScores`，为成绩容器中的每个元素增加指定分数。

函数接收一个可修改的整数 `vector` 与增量 `increment`。函数调用后，容器中的所有成绩都应被直接修改；当容器为空时，函数不执行任何操作。

## 相关知识

### 范围 for 中的引用传递

范围 `for` 循环可通过引用直接访问容器元素：

``` cpp
for (int& score : scores) {
    score += 5;
}
```

此处的 `score` 是容器中当前元素的别名。修改 `score` 会直接修改原容器中的元素。

若写成 `int score`，循环变量只是元素副本，修改后不会影响原容器：

``` cpp
for (int score : scores) {
    score += 5;  // 只修改副本
}
```

### 遍历时不修改容器大小

范围 `for` 循环中不应调用 `push_back`、`erase` 等会改变容器大小的操作，因为这可能导致迭代器失效。本题只修改已有元素的值。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数 `increaseScores` 的函数体。
3. 函数必须采用以下声明形式：

``` cpp
void increaseScores(std::vector<int>& scores, int increment);
```

1. 必须使用范围 `for` 循环遍历容器。
2. 循环变量必须使用非常量引用，例如 `int& score`。
3. 每个元素都应增加 `increment`。
4. 不得在循环中改变容器大小。
5. 在 `test` 函数中使用 `assert` 验证修改结果。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <vector>

// TODO：使用范围 for + 引用修改每个元素
void increaseScores(std::vector<int>& scores, int increment) {
    // TODO
}

void test() {
    std::vector<int> scores1 = {60, 75, 88};
    increaseScores(scores1, 5);

    assert(scores1[0] == 65);
    assert(scores1[1] == 80);
    assert(scores1[2] == 93);

    std::vector<int> scores2 = {};
    increaseScores(scores2, 10);
    assert(scores2.empty());

    std::vector<int> scores3 = {100};
    increaseScores(scores3, -10);
    assert(scores3[0] == 90);
}

int main() {
    test();

    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项           | 操作                    | 预期结果       |
| ---------------- | ----------------------- | -------------- |
| 多个元素增加分数 | `{60, 75, 88}` 增加 `5` | `{65, 80, 93}` |
| 空容器           | `{}` 增加 `10`          | 仍为空容器     |
| 负增量           | `{100}` 增加 `-10`      | `{90}`         |

如果循环变量错误地使用值传递 `int score`，断言 `assert(scores1[0] == 65)` 将失败，因为原容器中的元素并未改变。

---

开始你的任务吧，祝你成功！
