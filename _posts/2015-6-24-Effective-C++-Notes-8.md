---
layout:	post
title:	Effective C++ Notes - 8
---

```
Tip 35 : Consider alternatives to virtual functions.
```

NVI（Non-Virtual Interface）实现 Template Method 模式。

{% highlight c++ linenos %}
class Painter {
public:
    void draw() const { doDraw(); }     // wrapper
private:
    virtual void doDraw() const {}
};
{% endhighlight %}

NVI 允许派生类重新实现虚函数，但调用虚函数需要权限，若兄弟派生类想要调用 draw，则需要声明为
protected.

// pass Strategy 模式

```
Tip 36 : Nerver redefine an inherited non-virtual function.
```

non-virtual 函数是静态绑定的，所以基类指针和派生类指针会产生不同的结果。

{% highlight c++ linenos %}
class Base {
public:
    void func1();
    virtual void func2(); 
};
class Derived : public Base {
public:
    void func1();
    virtual void func2();
};

Derived d;
Base *pb = &d;
Derived *pd = &d;

pb->func1();    // call Base::func1()
pd->func1();    // call Derived::func1()

pb->func2();    // call Derived::func2()
pd->func2();    // call Derived::func2() 
{% endhighlight %} 

所以，任何情况下，不该重新定义一个继承而来的 non-virtual。解决该问题的方法参考 Tip 34。

```
Tip 37 : Never redefine a function's inherited default parameter value.
```

{% highlight c++ linenos %}
class Base {
public:
    enum Order { one, two, three };
    virtual void getOrder(Order order = one) const = 0;
};
class D1 : public Base {
public:
    virtual void getOrder(Order order = two) const;     // default change
};
class D2 : public Base {
public:
    virtual void getOrder(Order order) const;           // no default
};

Base *pb;
D1 d1;
D2 d2;
pb = &d1;
pb->getOrder(Order::three);     // D1::getOrder(Order::three)
pb->getOrder();                 // D1::getOrder(Order::two)     !!

pb = &d2;       
pb->getOrder(Order::three);     // D2::getOrder(Order::three)
pb->getOrder();                 // it depends on compiler.      !!
{% endhighlight %}

要避免这个问题，可以使用 Tip 35 的 NVI 方法。

{% highlight c++ linenos %}
class Base {
public:
    enum Order { one, two, three };
    void retOrder(Order order = one) const { getOrder(order); }
private:
    virtual void getOrder(Order order) const = 0;
};

class Derived : public Base {
private:
    virtual void getOrder(Oreder order) const;
};
{% endhighlight %}

在 VS2013 community 中，编译器将派生类中的默认参数都使用基类的默认参数。

```
Tip 38 : Model "has-a" or"is-implemented-in-terms-of" through composition.
```

复合（composition）的意义和 public 继承完全不同。

在应用域（application domain），复合意味着“有一个”（"has-a"）。
在实现域（implementation domain），复合意味着“根据某物实现”（is-implemented-in-terms-of）。

```
Tip 39 : Use private inheritance judiciously.
```

// pass

```
Tip 40 : Use multiple inheritance judiciously.
```

// pass



&copy; 2015 plinx