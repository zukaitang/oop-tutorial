# 练习13：使用对象指针管理账户

---

## 任务描述

设计账户类 `Account`，通过对象指针动态创建账户对象，完成存款、取款和账户信息输出操作，最后释放动态创建的对象。

本题重点练习对象指针、`new` 与 `delete`，以及使用 `->` 访问指针所指对象的成员。

## 相关知识

### 对象指针

对象指针用于保存对象的地址。使用 `new` 可以在动态存储区创建对象，并获得指向该对象的指针：

``` cpp
Account* account = new Account("A001", 100.00);
```

通过对象指针访问成员时，使用 `->` 运算符：

``` cpp
account->Deposit(50.00);
double balance = account->GetBalance();
```

`account->Deposit(50.00)` 等价于 `(*account).Deposit(50.00)`，但前者更常用。

### 释放动态对象

使用 `new` 创建的单个对象应使用 `delete` 释放：

``` cpp
delete account;
account = nullptr;
```

`delete` 会调用对象的析构函数并释放对象占用的动态内存。释放后将指针置为 `nullptr`，可避免继续使用无效地址。

### 成员函数中的条件判断

取款操作需要判断余额是否足够。只有当取款金额大于 `0` 且不超过当前余额时，才允许减少余额。

## 编程要求

1. 使用 C++11 标准编写程序。
2. 在 `Account.h` 中声明 `Account` 类，私有数据成员为：

   ```cpp
   std::string id;
   double balance;
   ```

1. 声明并实现以下公有成员函数：

   ```cpp
   Account(const std::string& id, double balance);
   void Deposit(double amount);
   bool Withdraw(double amount);
   const std::string& GetId() const;
   double GetBalance() const;
   void Print() const;
   ```

1. `Deposit` 仅在 `amount > 0` 时增加余额；`Withdraw` 在取款成功时返回 `true` 并减少余额，否则返回 `false` 且余额不变。
2. `Print` 按以下格式输出账户信息，余额固定保留两位小数：

   ```text
   账号 A001
   余额 130.00
   ```

1. 在 `main.cpp` 中使用 `new` 创建 `Account` 对象，通过 `->` 调用成员函数；使用 `assert` 验证存取款结果和输出内容；最后使用 `delete` 释放对象并将指针设为 `nullptr`。

## 待完成代码

### Account.h

``` cpp
#ifndef ACCOUNT_H
#define ACCOUNT_H

#include <string>

// TODO：声明 Account 类

#endif
```

### Account.cpp

``` cpp
#include "Account.h"

// TODO：定义 Account 的成员函数
```

### main.cpp

``` cpp
#include "Account.h"

#include <cassert>
#include <iostream>
#include <sstream>

void test() {
    Account* account = new Account("A001", 100.00);

    account->Deposit(50.00);
    assert(account->GetBalance() == 150.00);

    assert(account->Withdraw(20.00));
    assert(account->GetBalance() == 130.00);

    assert(!account->Withdraw(200.00));
    assert(account->GetBalance() == 130.00);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    account->Print();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "账号 A001\n余额 130.00\n");

    delete account;
    account = nullptr;
    assert(account == nullptr);
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

测试先以 `100.00` 创建账户，存入 `50.00`，再成功取出 `20.00`，最后尝试取出超过余额的 `200.00`。预期余额为 `130.00`，且失败取款不会改变余额。

可使用以下命令编译三个文件：

``` bash
g++ -std=c++11 Account.cpp main.cpp -o account_pointer
```

---

开始你的任务吧，祝你成功！
