# 练习：使用 string 拼接问候语

---

## 任务描述

编写函数 `makeGreeting`，使用 `std::string` 拼接姓名和问候内容，生成一条完整的问候语。

## 相关知识

### std::string

`std::string` 是 C++ 标准库提供的字符串类型，会自动管理字符存储空间，使用时无需手动处理结尾空字符或动态内存释放。

```cpp
std::string name = "李红霞";
std::string message = "你好，" + name;
```

### const 引用参数

只读取字符串时，应使用 `const std::string&` 参数，避免不必要的复制并防止函数修改实参。

```cpp
std::string makeGreeting(const std::string& name);
```

### 字符串拼接

`+` 可连接 `std::string` 对象和字符串字面量。若左侧是字面量，应先构造一个 `std::string` 对象。

```cpp
return std::string("你好，") + name + "！";
```

## 编程要求

1. 使用 C++11 标准编写程序。
2. 完成函数：

   ```cpp
   std::string makeGreeting(const std::string& name);
   ```

3. 返回格式必须为 `你好，姓名！欢迎学习 C++。`，其中标点均为全角字符。
4. 在 `test` 中使用 `assert` 验证两组字符串返回值和输出结果。

## 待完成代码

```cpp
#include <cassert>
#include <iostream>
#include <sstream>
#include <string>

// TODO：使用 std::string 拼接问候语
std::string makeGreeting(const std::string& name) {
    // TODO
}

void test() {
    assert(makeGreeting("李红霞") == "你好，李红霞！欢迎学习 C++。");
    assert(makeGreeting("张如雪") == "你好，张如雪！欢迎学习 C++。");

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    std::cout << makeGreeting("李红霞") << '\n';
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "你好，李红霞！欢迎学习 C++。\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

预期输出：

```text
你好，李红霞！欢迎学习 C++。
```

---

开始你的任务吧，祝你成功！
