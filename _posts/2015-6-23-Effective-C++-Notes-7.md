---
layout:	post
title:	Effective C++ Notes - 7
---

## 6、Inheritance and Object-Oriented Design

```
Tip 32 : Make sure public inheritance models "is-a."
```

“public 继承”意味着 is-a。适用于 base classes 身上的每一件事情一定也适用于 
derived classes 身上，因为每一个 derived class 对象也是一个 base class 对象。

```
Tip 33 : Avoid hiding inherited names.
```

在不同的域中，变量的遮掩是从内到外的。例如：

{% highlight c++ linenos %}
int x = 1;
void inner() {
    double x = 2.0;
    cout << x;      // double x
}
{% endhighlight %}

在继承中，派生类对基类也相当于一个内部域，若在派生类中重载了基类中的一个函数，将遮掩基类中所有的同名函数。

{% highlight c++ linenos %}
class Base {
public:
    virtual void func1() = 0;
    virtual void func1(int);
};
class Derived : class Base {
    virtual void func1();
};

Derived d;
d.func1();      // call Derived::func1()
d.func1(5);     // error
{% endhighlight %}

为了让基类的函数暴露在派生类中，可以使用 using 实现。

{% highlight c++ linenos %}
class Base {
public:
    virtual void func1() = 0;
    virtual void func2(int);
    void func2();
    void func2(int);
};
class Derived : class Base {
    using Base::func1;
    using Base::func2;
    virtual void func1();
    void func2();
};

Derived;
d.func1();      // call Derived::func1()
d.func1(5);     // call Base::fun21(int)
d.func2();      // call Derived::func2()
d.func2(5);     // call Base::func2(int)
{% endhighlight %}

然而当我们不使用 public 继承的 is-a 关系时，使用 using 不一定能够正确指向函数的域，
此时可以使用转交函数（forwarding functions）实现。

{% highlight c++ linenos %}
class Base {
public:
    virtual void func1() = 0;
    virtual void func1(int);
};
class Derived : public Base {
    virtual void func1() { Based::func1(); }
};

Derived d;
d.func1();      // call Drived::func1()
d.func1(5);     // error
{% endhighlight %}
 
```
Tip 34 : Differentiate between inheritance of interface and inheritance of implementation.
```

在类的继承中，区分基类和派生类的中心在于：
函数接口（function interfaces）和函数实现（function implementations）。

作为类设计者，对类应该有三种期许：

- 派生类只能继承接口
- 派生类继承接口和实现，允许覆写（override）
- 派生类继承接口和实现，不允许覆写

这三种期许对应着 c++ 中三种函数声明方式：

- 纯虚函数 pure virtual
- （非纯）虚函数 impure virtual
- 非虚函数 non-virtual

pure virtual 意味着派生类只继承其接口，但实际上，纯虚函数也可以有自己的函数实现。

{% highlight c++ linenos %}
class Base {
public:
    virtual void func1() = 0;
};
void Base::func1() {...}
class Derived : public {
    void func1();
};

Base *pd = new Derived();
pd->func1();            // call Derived::func1()
pd->Based::func1();     // call Based::func1()
{% endhighlight %}

impure virtual 提供函数的缺省版本，当派生类不想实现该接口时，将调用基类的接口实现。

然而，当我们创建一个派生类，但类使用者可能会**忘记**去实现某个接口，从而导致结果与预期
不同。

有两种方法可以避免这种问题。
{% highlight c++ linenos %}
// method 1
class Base {
public:
    virtual void func2() = 0;
protected:
    void defaultfunc2();
};
class Derived : public Base {
    void func2() { defaultfunc2(); }
};
// method 2 : better
class Base {
public:
    virtual void func2() = 0;
};
void Base::func2() {...}
class Derived : public Base {
public:
    void func2() { Base::func2(); }
};
{% endhighlight %}

non-virtual 提供了持久化的接口，直到派生类想要制作特异化（specialization）的接口。

《 C++ Primer 5th 》15.3 介绍了 c++11 的关键字 final 和 override。
final 用于禁止覆盖，override 用于检查虚函数。

&copy; 2015 plinx