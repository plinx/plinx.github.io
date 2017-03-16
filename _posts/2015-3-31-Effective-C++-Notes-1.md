---
layout: post
title: Effective C++ Notes - 1
---

《 Effective C++ 》 学习笔记用于记录一些自己遇到了，读后深有感触的条目。

## 1. Accustoming Youself to C++ ##

```
Tip 01 : View C++ as a fedoration of languages
```

C++ 是一种多范式（multiparadigm）的编程语言：

- 过程形式 procedural
- 面向对象形式 object-oriented
- 函数形式 functional
- 泛型形式 generic
- 模板形式 template

在不同的应用场景，使用不同的范式进行编程能实现兵来将挡、水来土掩的神奇效果。


```
Tip 02 : Prefer consts, enums, and inlines to #defines
```

区别：#define 作用在预处理期；const，enum 和 inline 作用在编译期。

在预处理期中，#define 主要起到替换的作用，将宏替换成声明的内容。
在程序中动态使用 #define 的宏时，可能会出现分编译错误的信息，但此时只能查看到被替换的信息而无法知道宏是谁。
使用 const 替换 #define 可以在整个编译期都保持有效。

在类（class）内部使用常量时，若使用 const 需要在实现文件中再次声明，
若使用 enum 可以避免重复类外再声明，enum 的另外一个好处是，可以隔离 pointer 和 reference 的指向：

{% highlight c++ linenos %}
// const version
// .h
class A {
    static const int Num = 5;
    int value[Num];
};
// .cpp
const int A::Num;

// inline version
// .h
class A {
    enum { Num = 5 };
    int Value[Num];
};
{% endhighlight %}

在 #define 中调用函数可能会受实参影响，而 inline 可以很好地替换它：
{% highlight c++ linenos %}
// #define version
#define MAX_FUNC(a, b) f((a) > (b) ? (a) : (b))

int a = 5, b = 0;
MAX_FUNC(++a, b);			// a = 7
MAX_FUNC(++a, b + 10);		// a = 6

// inline version
template <typename T>
inline void max_func(const T& a, cosnt T& b) {
    f(a > b ? a : b);
}
{% endhighlight %}

```
Tip 03 : Use const whenever possible
```
const 指针常用用法区别：

（1）char str[] = "test";

（2）char* p = str;		// non-const pointer/data

（3）const char* p = str;	// const data, non-const pointer

（4）char const* p = str;	// const data, non-const pointer

（5）char* const p = str;	// const pointer, non-const data

（6）const char* const p = str;	// const pointer/data

```
Tip 04 : Make sure the objects are initialized before they're used.
```
类的构造函数应尽可能使用初始化列表来构造，而避免使用赋值：
{% highlight c++ linenos %}
// Use member initialization list
// better version
class A {
    int data;
    std::string name;

    A(int d, std::string n) : data(d), name(n) {}
};

// Use copy assignment
// worse version
class B {
    int data;
    std::string name;

    B(int d, std::string n) {
        data = d; name = n;
    }
};
{% endhighlight %}

当跨单元编译时，若存在一个类成员是另外一个类，或者类成员使用另外一个类的函数/参数进行构造，可以使用 **Singleton-Pattern** 进行设计：
{% highlight c++ linenos %}
class A {
    int data;
public:
    A(...) {
        // don't call createB()!
    }
    static A& createA() {
        static A tmp;
        return tmp;
    }
    int data() { return data; }
};
class B {
    ...
    int data;
    B(...) {
        ...
        data = createA().data();
    }
    static B& createB() {
        static B tmp;
        return tmp;
    }
};
{% endhighlight %}
**Singleton-Pattern** 通过一个函数返回新创建的类，可以防止类在被调用时未初始化。

**注意：不能在 B 构造函数调用 createA() 时，A 构造函数调用 createB()，否则将造成 AB 锁**

&copy; 2015 plinx
