---
title: 写一个不会抛出异常的swap函数
poc: true
categories:
  - - C++简明进阶教程
tags:
  - C++简明进阶教程
date: 2023-06-18 13:16:00
---

首先需要注意的是成员版本的swap绝对不可以抛出异常, 他提供了不抛出异常的"赋值"操作; 但是非成员版本的swap 比如std::swap就可以一定程度上允许抛出, 因为其默认实现是基于构造函数的, 而构造函数是允许抛出异常的



首先是典型的std::swap实现

```cpp
namespace std{
    template<typename T>
    void swap(T& _Left, T& _Right){
	 T _Tmp = _STD move(_Left);
    _Left    = _STD move(_Right);
    _Right   = _STD move(_Tmp);           
    }
}

```

如果我们的类是`pImpl`实现方式(如下所示) 则可能会出一些问题

```cpp
template<typename T>
class WidgetImpl {
public:
	vector<T> publicContainer;


private:
	vector<T> privateContainer;
};

template<typename T>
class Widget {
//public:
//	WidgetImpl<T>* publicContainer;


private:
	WidgetImpl<T>* pImpl;
};

```



对于`Widget`对象, 我们`swap`只需要把内部的指针`swap`即可

因此,原始的实现不够完善, 所以我们需要自己特化一个swap

首先可以想到的是, 将swap在std空间中全特化, std空间中不允许添加新的函数, 但是允许全特化模板

```cpp
namespace std{
    template<>//意味着全特化
    void swap<Widget>(Widget& a,Widget& b){
        swap(a.pImpl,b.pImpl); //无法通过编译, 因为尝试访问类内private对象
    }
}
```

因此可以想到的改进是声明为友元函数, 但是为了与stl实现相一致, 这里我们采用非友元的实现, 如下所示

```cpp
//template<typename T> //修改为一般类 而非模板类
class Widget {
//public:
//	WidgetImpl<T>* publicContainer;
public:
    void swap(Widget& _Right){
        using std::swap;
        swap(pImpl, _Right.pImpl);       
    }
    

private:
	//WidgetImpl<T>* pImpl;
    WidgetImpl<int>* pImpl;
};


namespace std{
    template<>//意味着全特化
    void swap<Widget>(Widget& a,Widget& b){
        a.swap(b);
    }
}
```

到此为止， 我们已经完成了对**一般类**的swap函数编写，但是还有一个比较复杂的问题等待着我们，那就是为**模板类**编写swap



首先来看最直观的第一想法是否可以正确实现

```cpp
template<typename T>
class WidgetImpl {  //...
};
template<typename T>
class Widget {  //...
};

namespace std{
    template<typename T>
    void swap<Widget<T>>(Widget<T>&, Widget<T>&){//函数模板不允许偏特化 编译不通过
        
    }
    
}
```

上述内容不合法, 不允许偏特化函数模板

详情请参考https://valkierja.github.io/effective-c/pian-te-hua-mo-ban-han-shu-za-tan/ 等站内文章



当你打算偏特化模板函数时, 一个惯用做法是重载一般化版本的模板函数

```cpp
template<typename T>
class WidgetImpl {  //...
};
template<typename T>
class Widget {  //...
};

namespace std{
    template<typename T>
    void swap(Widget<T>&, Widget<T>&){//swap后面没有不完全类型和尖括号, 因此这个是一般化模板函数的重载版本,而不是偏特化
    //原始签名:
    // template<typename T>
    //void swap(T&, T&)
    }
    
}
```

一般来说, 上述做法是完全正确的, 但是有一点问题, 那就是: 向std空间中添加函数 或者重载函数之类的操作理论上是不被允许的

但是这个只是一个规范性的东西, 所以在大多数情况下, 上述代码不仅可以正常过编译, 还可以正常使用

但是为了完全贴合C++规范 我们需要改造一下上面的代码

改造的方法就是声明一个自己的命名空间,然后在这个空间中 重载一般化的模板函数

```cpp
namespace WidgetStuff{
    template<typename T>
	class WidgetImpl {  //...
	};
	template<typename T>
    class Widget {  //...
    };
    
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b){
        a.swap(b) //提供public成员函数 以操作private对象
    }
}
```

到这里为止, 我们已经写出了符合C++规范的swap函数(针对模板类对象)

但是还有一些小问题

1. 为什么放到命名空间`WidgetStuff`下, 编译器就可以找到这个重载版本的swap?
2. 我们前面讨论的全特化`std::swap`函数是否不再必要?(因为上述重载的方法可以同时应对模板类和一般类)



首先是第一个问题, 这是由于koemig 查找规则决定的https://zh.cppreference.com/w/cpp/language/adl

> Argument-dependent lookup (ADL) 是 C++ 中的一个重要机制，其规则如下：
>
> 1. 在命名空间中查找：如果使用了未限定的函数名（即没有指定命名空间），则编译器会在包含该函数的命名空间及其直接可见的命名空间中查找。
> 2. 参数类型匹配：如果函数名存在于多个命名空间中，则编译器会对每个命名空间中的所有函数进行参数类型匹配，并选择与调用中提供的参数最佳匹配的函数。
> 3. 模板参数匹配：如果有多个函数具有完全相同的名称和参数类型，则编译器会选择模板参数最匹配的函数。如果无法确定，则会发生编译错误。
>
> 需要注意的是，ADL 仅在函数模板实例化时起作用，而不是在定义时。此外，在某些情况下，ADL 可能会导致意外的行为或编译错误，因此需要谨慎使用。

也就是说, 对于实参类型所在的命名空间的同名函数, 如果可能的话, 会被优先使用



其次是第二个问题, 这个问题源于现实中许多不清楚命名空间到底是什么的C++程序员的一个常见的"错误", 或者说是, 不那么优秀的实现代码

```cpp
std::swap(a,b);
```

对的没错, 那就是无论什么时候都直接写死需要调用的函数所在的命名空间, 而不是先暴露特定命名空间中的函数接口,而后直接调用:

```cpp
using std::swap;
swap(a,b)
```

下面的这个写法才能让编译器找到`WidgetStuff`命名空间中的重载版本swap

但是介于第一种写法在现实中太过于普遍, 因此为了防止客户误用,我们还是需要全特化`std::swap`

但是需要注意的是, 全特化`std::swap`的做法只能当你实现一个一般类时使用, 而不能用于实现模板类时使用

总结:

1. 在类内提供一个public函数, 用于操作private成员
2. 为class或者是class template所在命名空间提供一个一般化模板函数swap的重载版, 其实现是调用上述public函数
3. 如果你在编写的是class需要额外做本步骤, 如果是template则不需要；在命名空间std中, 全特化swap函数,  其实现是调用上述public函数
4. 在任何地方为这个类调用swap函数时, 先用using暴露std空间中的版本, 再直接调用swap, 不要加限定符 `swap(a,b)`

但是我们还没有讲异常这部分!

(未完待续)

