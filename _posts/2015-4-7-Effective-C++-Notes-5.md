---
layout:	post
title:	Effective C++ Notes - 5
---

## 4. Deigns and Declarations ##

```
Tip 18 : Make interfaces easy to use correctly and hard to use incorrectly.
```
设计接口的时候，需要尽可能考虑客户使用接口的可能性，包括错误使用的可能性，可以通过接口参数封装实现预防接口误用。
{% highlight c++ linenos %}
// a bad design
class Date {
public:
    Date(int month, int day, int year);
};
// a better design
struct Day { 
    explicit Day(int d) : val(d) {}
    int val;
};
struct Month {
    explicit Month(int m) : val(m) {}
    int val;
};
struct Year {
    explicit Year(int y) : val(y) {}
    int val;
};
class Date {
public:
    Date(const Month& m, const Day& d, const Year& y);
};
{% endhighlight %}

```
Tip 19 : Treat class design as type design.
```
定义一个新的 class，也就定义了一个新 type。想要高效地设计 classes，需要想清楚以下的问题：

（1）新 type 的对象应该如何创建和销毁；

（2）对象的初始化和对象的复制应该有怎样的差别；

（3）新 type 的对象如果被 passed by value 意味着什么；

（4）什么是新 type 的合法值；

（5）你的新 type 需要配合某个继承图系（inheritance graph）或设计模式吗；

（6）你的新 type 需要什么样的转换;

（7）什么样的操作符和成员函数应该/不应该暴露给使用者；

（8）谁会使用该 type，它是否具有权限；

（9）什么是新 type 的 “未声明接口”（undeclared interface）；

（10）考虑是否使用 template 避免重复实现；

（11）真的需要实现一个 type 来达到目标么。

```
Tip 20 : Prefer pass-by-reference-to-const to pass-by-value.
```

（1）使用传引用（pass-by-reference-to-const）可以避免额外的构造和析构，并且可以避免指向指向子类的父类指针被切割的问题；

（2）当使用系统内置类型时和 STL 迭代器时，使用传值（pass-by-value）更合适。

```
Tip 21 : Don't try to return a reference when you must return an object.
```
当实现 operator 操作时，可以通过是否有 “=” 来判断返回类型：

（1）若不含 “=” 号（例如 operator+/-/*），返回对象可以避免修改原参数，或者返回对象在使用时已经被销毁导致的错误；

（2）若包含 “=” 号（例如 operator+=/-=/*=/=），返回引用可以实现递归调用，保证用户语义的正确性。

```
Tip 22 : Declare data members private.
```

（1）将成员变量声明为 private，可以赋予客户端访问数据的一致性、为细微划分访问控制、允许约束条件获得保证，并提供 class 作者以充分的实现弹性；

（2）对于派生类而言，protected 并不比 public 更有封装性，当数据改变时，同样会造成大量代码重写。

```
Tip 23 : Prefer non-member non-friend functions to member functions.
```

这里可以从两个角度去思考：

（1）当我们设计一个类的时候，我们通常将成员设置为 private（见 Tip 22），这样一来，我们可以说将类的内部封装起来了，外部成员只能通过我们预设的接口进行访问，此时我们可以认为封装是好的。假设在这个基础上，需要设计了一个 clearEverything 函数，这个函数可以清除类内部的所有成员信息。

|     | 成员函数 | 非成员函数 |
| --- | --- | --- |
| 优点 | 调用简单，操作彻底 | 包裹弹性（packaging flexibility）好，可定制 |
| 缺点 | 访问限制低        | 需要手工调用类接口 |

假设有一个类表示平面上的一点，包含坐标和颜色信息，可以用两种方式清楚其数据。

{% highlight c++ linenos %}
class Point2D {
public:
    void clearPoint() { x = y = 0; }
    void clearColor() { color = 0; }
    // member function clearEverything
    void reset() { zeroPoint(); zeroColor(); }
private:
    int x, y;
    int color;
};

// non-member function clearEverthing
void clear(Point2D& point) {
    point.clearPoint();
    point.clearColor();
}
{% endhighlight %}
从这个角度看，成员函数和非成员函数是没有什么区别的，作为单独的类，使用成员函数往往更能抽象操作的意义。

（2）从另外一个角度看，在我们的类 Point2D 创建后，虽然我们的成员是 private 类型的，但是对于 reset 操作来说，他们全部都暴露在类使用者（客户）面前，客户只要使用 reset 就会清除类内所有的信息。当类创建者和使用者对类不是同一个人时，对同一个操作的理解也会不同，这种时候，类创建者对类的封装就变得很“脆弱”了，或者说封装性实际上是很低的。在这个角度看，类创建者和使用者需要承担不同的责任，创建者需要保证类内部数据的正常，使用者需要保证自己使用方法是合理的。

在类创建者不提供便利函数的时候，使用者可以根据需要组合便利函数。

{% highlight c++ linenos  %}
// Point.h
namespace Point {
    class Point2D {
    public:
        void clearPoint() { x = y = 0; }
        void clearColor() { color = 0; }
        // void reset() { ... }
    private:
        ...
    };
}
// Coordinate.h
namespace Point {
    void clearPoint(Point2D& point) { point.clearPoint(); }
} 
// Color.h
namespace Color {
    void clearColor(Point2D& point) { point.clearColor(); }
}
{% endhighlight %}

当类不仅仅是作为单独的类，而是作为基类时，从这个角度往往发现更多的问题。所以一个 clearEverything 函数，究竟是应该作为成员函数还是应该作为非成员函数应该取决于类的用途。

```
Tip 24 : Declare non-member functions when type conversions should apply to all parametes.
```

当类操作可预见地会出现隐式类型转换时，需要为其提供一个非成员函数的转换函数。

{% highlight c++ linenos %}
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);
    int numerator() const;
    int denominator() const;
private:
    ...
};

Rational oneHalf(1, 2);
Rational result;
result = oneHalf * 2;   
// Pass : oneHalf.operator*(2) => oneHalf.operator*(Rational tmp(2))
result = 2 * oneHalf;   
// Error : 2.operator*(oneHalf) => int(2).operator*(oneHalf)
{% endhighlight %}

遇到这种情况，我们可以提供一个支持混合运算的非成员函数。

{% highlight c++ linenos %}
class Rational { ... };

const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.numerator() * rhs.numerator(),
                    rhs.denominator() * rhs.denominator());
}

Rational result;
Rational oneHalf(1, 2);
result = oneHalf * 2;   
// Pass : oneHalf.operator*(2) => oneHalf.operator*(Rational tmp(2))
result = 2 * oneHalf;   
// Pass : 2.operator*(oneHalf) => Rational tmp(2).operator*(oneHalf)

{% endhighlight %}

``` 
Tip 25 : Consider support for a non-throwing swap.
```

STL 中 swap 的算法实现如下。

{% highlight c++ linenos %}
namespace std {
    template<typename T>
    void swap(T& a, T& b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
{% endhighlight %}

在这个算法实现中，a 和 b 的交换通过中间变量 temp 来传递。若 a, b 是内置变量类型，那么用 temp 的传递效率可以接受。若 a, b 为类时，temp 需要使用类拷贝构造函数。当 a, b 为类，且内部包含其他的类指针时，实现如下。

{% highlight c++ linenos %}
class Widget {
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        *pImpl = *(rhs.pImpl);
    }
private:
    WidgetImpl* pImpl;
};
{% endhighlight %}

当我们对 Widget 执行 swap 时，我们不仅构造了临时变量 temp，还对 WidgetImpl 指针进行了三次复制，非常缺乏效率。想要高效地交换指针，我们需要“特化” STL 中的 swap 函数，实现如下。

{% highlight c++ linenos %}
class Widget {
public:
    void swap(Widget& rhs)
    {
        using std::swap;
        swap(pImpl, rhs.pImpl);
    }
};

namespace std {
    template<>      // std::swap 全特化版本(total template specialization)
    void swap<Widget>(Widget& a, Widget& b)
    {
        a.swap(b);
    }
}
{% endhighlight %}

当我们使用的 Widget 是一个类时，以上的做法是可行的，但当 Widget 是一个模板类时，我们需要“偏特化” STL 中的 swap 函数，以下为错误代码。

{% highlight c++ linenos %}
namespace std {
    template<typename T>    // std::swap 偏特化版本(partially specialize)
    void swap<Widget<T>>(Widget<T>& a, Widget<T>& b)    
    // 编译出错，C++ 不允许在 function template 上偏特化
    {
        a.swap(b);
    }

    template<typename T>    // 偏特化重载 std::swap
    void swap(Widget<T>& a, Widget<T>& b)               
    // 编译正常，但往 STL 填充新东西是“禁止”的行为
    {
        a.swap(b);
    }
}
{% endhighlight %}

要解决这个问题，最好的方法是使用命名空间管理 Widget 或 Wiget<T> 的 swap 行为。

{% highlight c++ linenos %}
namespace WidgetSpace {
    template<typename T>
    class Widget { ... };       // 与前面的实现相同

    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b)
    {
        a.swap(b);
    }
}
{% endhighlight %}

此后，在任何地方使用 swap 交换 Widget，都将使用 WidgetSpace 中的 swap，因为 C++ 遵循名称查找法则（name lookup rules；更具体地说是 argument-dependent lookup 或 Koenig lookup 法则）。

&copy; 2015 plinx

