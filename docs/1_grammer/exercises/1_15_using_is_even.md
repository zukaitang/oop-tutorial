# 练习15：使用 using 声明引入偶数判断函数

---

## 任务描述

在 `NumberUtils` 命名空间中完成函数 `isEven`，用于判断一个整数是否为偶数。

在 `test` 函数中，使用 `using NumberUtils::isEven;` 仅引入 `isEven` 这一个名称，然后直接调用 `isEven` 完成测试。通过本题理解 `using` 声明与 `using namespace` 指令的区别。

## 相关知识

### 命名空间中的函数

函数可以定义在命名空间中，用于组织同类工具：

``` cpp
namespace NumberUtils {
    bool isEven(int value) {
        return value % 2 == 0;
    }
}
```

通常可使用完全限定名调用：

``` cpp
bool result = NumberUtils::isEven(8);
```

### using 声明

`using` 声明可以只引入命名空间中的一个名称：

``` cpp
using NumberUtils::isEven;

bool result = isEven(8);
```

这比 `using namespace NumberUtils;` 更安全，因为它不会将 `NumberUtils` 的全部名称引入当前作用域。

### 取模运算符

整数除以 `2` 的余数可用于判断奇偶性：

``` cpp
value % 2 == 0  // 偶数
value % 2 != 0  // 奇数
```

`0` 和负偶数同样满足 `value % 2 == 0`。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 保留题目给出的 `namespace NumberUtils`。
3. 完成函数 `isEven` 的函数体。
4. 函数必须采用以下声明形式：

``` cpp
bool isEven(int value);
```

1. 使用 `%` 运算符判断奇偶性。
2. 偶数返回 `true`，奇数返回 `false`。
3. 在 `test` 函数中使用 `using NumberUtils::isEven;`。
4. 不得使用 `using namespace NumberUtils;`。
5. 在 `test` 函数中使用 `assert` 验证结果。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>

namespace NumberUtils {
    // TODO：偶数返回 true，奇数返回 false
    bool isEven(int value) {
        // TODO
    }
}

void test() {
    // TODO：仅引入 NumberUtils 中的 isEven
    // TODO

    assert(isEven(0));
    assert(isEven(8));
    assert(isEven(-12));

    assert(!isEven(1));
    assert(!isEven(99));
    assert(!isEven(-7));
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

| 测试项 | 函数调用      | 预期结果 |
| ------ | ------------- | :------: |
| 零     | `isEven(0)`   |  `true`  |
| 正偶数 | `isEven(8)`   |  `true`  |
| 负偶数 | `isEven(-12)` |  `true`  |
| 正奇数 | `isEven(99)`  | `false`  |
| 负奇数 | `isEven(-7)`  | `false`  |

注意：`using NumberUtils::isEven;` 只在 `test` 函数的作用域内生效，只引入一个名称，不会造成整个命名空间的名称污染。

---

开始你的任务吧，祝你成功！
