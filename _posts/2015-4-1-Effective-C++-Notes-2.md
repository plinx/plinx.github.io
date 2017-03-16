---
layout:	post
title:	Effective C++ Notes - 2
---

## 2、Constructors，Destructors，and Assignment Operators

```
Tip 05 : Know what functions C++ silently writes and calls.
```
（1）在未声明任何构造函数时，编译器会为类合成默认构造函数（constructor）/析构函数（Destructor）/拷贝构造函数（copy constructor）/拷贝赋值操作符（operator=）；当类声明了含/不含实参的构造函数，编译器不会再为类合成默认构造函数/析构函数，但仍能合成默认拷贝构造函数/拷贝赋值构造函数。
{% highlight c++ linenos %}
template <typename T>
class A {
public:
    A(const char* v, const T& o) : value(v), obj(o) {}
    A(const std::string& v, const T& o) : value(v), obj(o) {}

    std::string get_value() const { return value; }
    T get_obj() const { return obj; }

private:
    std::string value;
    T obj;
};

int main()
{
    A<int> a1("test", 1);
    A<int> a2(a1);
    //A<int> a3; // Error : no default constructor exists for class A<T>
    A<int> a4 = a1;
    a4 = a2;

    return 0;
}
{% endhighlight %}
（2）当类中含有引用（reference）或常量（const）时，编译器无法合成拷贝赋值操作符；同样的，若在基类（base class）中声明拷贝赋值运算符为 private ，编译器将无法为派生类（derived class）合成拷贝赋值运算符号。
{% highlight c++ linenos %}
class A {
public:
    A(std::string& v) : value(v) {}
    std::string get_value() const { return value; }
private:
    std::string& value;
};
template <typename T>
class B {
public:
    B(const T& o) : obj(o) {}
    T get_obj() const { return obj; }
private:
    const T& obj;
};

int main()
{
    std::string str("test");
    A a1(str);
    A a2(a1);
    //A a3; // Error : no default constructor exists for class A
    A a4 = a1;
    //a4 = a2; // error C2582: 'operator =' function is unavailable in 'A'
    B<int> b1(1);
    B<int> b2(b1);
    //B<int> b3; // Error : no default constructor exists for class B<T>
    B<int> b4 = b1;
    //b4 = b2; // error C2582: 'operator =' function is unavailable in 'B<T>'

    return 0;
}
{% endhighlight %}
（3）当不使用引用或常量时，C++11 可以用 =default 来声明构造函数/析构函数/拷贝操作运算符，让编译器合成默认构造函数，区别于 Tip05-（1）。
{% highlight c++ linenos %}
template <typename T>
class A {
public:
    A() = default;
    ~A() = default;

    A(std::string& v, const T& o) : value(v), obj(o) {}
    std::string get_value() const { return value; }

    A& operator=(const A& rhs) = default;
private:
    std::string value;
    T obj;
};

int main()
{
    std::string str("test");
    A<int> a1(str, 1);
    A<int> a2(a1);
    A<int> a3;
    A<int> a4 = a1;
    a4 = a2; 

    return 0;
}
{% endhighlight %}

**关于 =default 的更多内容请查看 《 C++ Primer 5th 》 13.1.5。**

```
Tip 06 : Explicitly disallow the use of compiler-generated functions you do not want.
```
（1）如果想禁止拷贝构造函数/拷贝赋值运算符，可以将其声明在 private 中，这样的类可以限制数据拷贝，保持数据唯一性；若想在每个类中都使用这样的限制，可以将基类设计成无参数基类，每一类对该基类可以通过继承实现禁止。
{% highlight c++ linenos %}
class A {
public:
    A() {}
    ~A() {}
private:
    A(const A&);
    A& operator=(const A&);
};

int main()
{
    A a1;
    //A a2(a1); // A::A(const A&) is inaccessible
    //A a3 = a1; // A::A(const A&) is inaccessible
    //A a4;
    //a4 = a1; // A& A::operator=(const A&) is inaccessible 

    return 0;
}
{% endhighlight %}
（2）C++11 可以用 =delete 实现禁止（删除），详细介绍参考 《 C++ Primer 5th 》 13.1.6。
{% highlight c++ linenos %}
class A {
public:
    A() {}
    ~A() {}
    A(const A&) = delete;
    A& operator=(const A&) = delete;
};

int main()
{
    A a1;
    //A a2(a1); // A::A(const A&) cannot be referenced 
    //A a3 = a1; // A::A(const A&) cannot be referenced
    //A a4;
    //a4 = a1; // A& A::operator=(const A&) cannot be referenced
    return 0;
}
{% endhighlight %}

&copy; 2015 plinx
