# 练习8：消解多继承中的同名成员二义性

---

## 任务描述

设计 `Painter`、`Musician` 和 `Artist` 类。两个基类都具有名为 `level` 的私有成员，`Artist` 必须使用基类名限定访问正确的接口。

## 相关知识

### 同名成员二义性

多继承中，若多个基类提供同名成员，派生类对象直接调用该名称会产生二义性。可使用 `基类名::成员名` 明确指定：

``` cpp
Painter::setLevel(3);
Musician::setLevel(5);
```

## 编程要求

1. `Painter` 和 `Musician` 都私有保存 `int level`，并提供小驼峰形式的 `setLevel`、`getLevel`。
2. `Artist : public Painter, public Musician` 实现：

   ```cpp
   void setLevels(int paintingLevel, int musicLevel);
   void printLevels() const;
   ```

1. `setLevels` 和 `printLevels` 必须分别使用 `Painter::`、`Musician::` 消解二义性。
2. 在 `test` 中断言两个等级及输出。

## 待完成代码

``` cpp
#include <cassert>
#include <iostream>
#include <sstream>

class Painter { /* TODO */ };
class Musician { /* TODO */ };

class Artist : public Painter, public Musician {
public:
    void setLevels(int paintingLevel, int musicLevel){
        /* TODO */
    }
    void printLevels() const{
        /* TODO */
    }
};

void test() {
    Artist artist;
    artist.setLevels(3, 5);
    assert(artist.Painter::getLevel() == 3);
    assert(artist.Musician::getLevel() == 5);

    std::ostringstream output;
    std::streambuf* oldBuffer = std::cout.rdbuf(output.rdbuf());
    artist.printLevels();
    std::cout.rdbuf(oldBuffer);
    assert(output.str() == "绘画等级：3\n音乐等级：5\n");
}

int main() {
    test();
    std::cout << "本关测试通过" << std::endl;
    return 0;
}
```

## 测试说明

预期输出：

``` text
绘画等级：3
音乐等级：5
```

---

开始你的任务吧，祝你成功！
