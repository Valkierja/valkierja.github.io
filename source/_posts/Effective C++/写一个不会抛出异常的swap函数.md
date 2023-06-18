---
title: 写一个不会抛出异常的swap函数
poc: true
categories:
  - - C++简明进阶教程
tags:
  - C++简明进阶教程
date: 2023-06-18 13:16:00
---

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
    void swap<Widget<T>>(Widget<T>&, Widget<T>&){//函数模板不允许偏特化
        
    }
    
}
```

上述内容不合法, 不允许偏特化函数模板

详情请参考