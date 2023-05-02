---
title: C++左值和右值:一文解决
poc: true
date: 2023-05-02 16:34:53
categories: 
- [C++简明进阶教程]
tags: [C++简明进阶教程]
---


(本节仍然处于草稿)




1. 左值右值实际上还需要更加清晰的定义生命周期这一概念
因此:

(这里需要写明xvalue不允许延长生命周期, 但是prvalue可以)

https://time.geekbang.org/column/article/169268

![image-20230501153305353](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202305011533431.png)





首先![image-20230428184043911](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304281840977.png)

![image-20230428192707304](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304281927332.png)

需要注意的是, xvalue和prvalue是不可以取地址的, 但是右值引用对象可以取地址

```cpp
int &&r = std::move(a);
int *p = &r;  // OK
```

因为右值引用对象`r`是一个r左值lvalue, 他所引用绑定的内容才是右值







需要注意一点, 我们说左值的时候, 一般只包括了"纯左值", 而不包括xvalue

但是我们说右值的时候, 往往是不仅仅指的是纯右值, 而是还包括了xvalue

当然, 这只是查阅资料的时候发现的一个约定俗成的东西, 不代表任何规范

(std::move的结果是一个xvalue, 可以绑定到右值引用但不能绑定到左值引用, 但是分类图上 一般把xvalue视作rvalue和glvaue的交集, 这一交集的描述似乎暗示了xvalue可以绑定到左值引用和右值引用,但是显示中xvalue只能绑定到右值引用和const左值引用上)

```cpp
    int temp = 1;
    int& a = temp;
    int& b = std::move(temp);//error
    int&& c = std::move(temp);
    int&& d = 1;
```

![image-20230428200128915](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304282001939.png)

这里继续拓展: 我们在网上的一些资料可以看到一种说法, 那就是xvalue has identity

以下是一些资料的例子

https://stackoverflow.com/questions/3601602/what-are-rvalues-lvalues-xvalues-glvalues-and-prvalues/38169963#38169963

![image-20230428201350079](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304282013145.png)

https://learn.microsoft.com/en-us/windows/uwp/cpp-and-winrt-apis/cpp-value-categories

![image-20230428202755863](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304282027910.png)我这边的现阶段的建议是忘掉这点

因为对这点的持续追问会最终会追溯到为什么下面的代码是错误的

```cpp
int main() {
    int a;
    int *b = &std::move(a); // ERROR
}
```

但是对上述代码的追问最终会指向:

 From [expr.unary.op]: https://stackoverflow.com/questions/50783525/what-does-it-mean-xvalue-has-identity

> The result of the unary `&` operator is a pointer to its operand. The operand shall be an lvalue [...]

语言硬性规定了这样的行为

所以我建议忘记identity这点

edit: 你可以把identity理解为脱离生成他的地方后, 仍然可以被二次使用的一种性质, 而不要像大多数资料那样, 理解成地址, 因为实际上xvalue不允许取地址操作



一个有意思的示例https://www.tutorialspoint.com/cplusplus/returning_values_by_reference.htm

```cpp
#include <iostream>
#include <ctime>

using namespace std;

double vals[] = { 10.1, 12.6, 33.1, 24.1, 50.0 };

double& setValues(int i) {
	return vals[i];   // return a reference to the ith element
}

double&& getValues(int i) {
    return std::move(vals[i]);   //
}
int main() {
    auto&& a = getValues(1);
    std::cout << a << std::endl;
    setValues(1) = 20.23; // change 2nd element
    return 0;
}
```

这个可以在Eigen矩阵库里面见到类似的操作

```cpp
Eigen::MatrixXf result = Eigen::MatrixXf::Zero(4, 4);
result.topLeftCorner(3, 3) = mat3x3;
```





关于函数的返回值, 需要注意的是

```cpp
int   prvalue();
int&  lvalue();
int&& xvalue();
```

正是因为如此, 上面的例子才得以实现





这里有一个小问题暂时笔者没有解决

```cpp
#include // std::move
#include <stdio.h>

class shape {
public:
  virtual ~shape() {}
};

class circle : public shape {
public:
  circle() { puts("circle()"); }
  ~circle() { puts("~circle()"); }
};

class triangle : public shape {
public:
  triangle() { puts("triangle()"); }
  ~triangle() { puts("~triangle()"); }
};

class result {
public:
  result() { puts("result()"); }
  ~result() { puts("~result()"); }
};

result
process_shape(const shape& shape1,
              const shape& shape2)
{
  puts("process_shape()");
  return result();
}

int main()
{
  puts("main()");
  process_shape(circle(), triangle());//good   
  result&& r = process_shape(circle(), triangle());//good
  const result& r = process_shape(circle(), triangle());//good
  result& r = process_shape(circle(), triangle());//cannot be compiled
  result&& r = std::move(process_shape( circle(), triangle()));// can be compiled, but wrong; out of lifetime
  puts("something else");
}
```

上述问题的参考资料:https://herbsutter.com/2008/01/01/gotw-88-a-candidate-for-the-most-important-const/

https://time.geekbang.org/column/article/169268

effective C++ 中文版p94





需要注意的是, 标准库里面的移动构造函数都会把原始对象的指针设置成nullptr, 因为移动的含义就是所有权转交, 那么原先的所有者(房客)持有的地址信息(房屋钥匙)就应当被销毁

例子如下

```cpp
string a="sdfasdfasdf";
string b(std::move(a));
//此时a的值为""
```





最后一个问题, 为什么  `const result&` 可以被绑定到右值上, 同时`result&&`也可以绑定右值? 

因为这是C++03历史弥留问题, 关于这一部分参考资料

 https://www.quora.com/In-C++-why-are-const-reference-types-allowed-to-bind-to-rvalues

https://herbsutter.com/2008/01/01/gotw-88-a-candidate-for-the-most-important-const/

https://changkun.de/modern-cpp/zh-cn/03-runtime/#3-3-%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8

那么为什么C++11之后不逐渐弃用左值常量引用的历史特殊性呢?

答案是为了区分拷贝构造函数和移动构造函数

请看这里https://www.zhihu.com/question/437590377/answer/2972237413
