---
layout:	post
title:	Effective C++ Notes - 3
---

```
Tip 07 : Declare destructors virtual in polymorphic base classes.
```
（1）在多态（polymorphic）的基类中，只有将析构函数声明为虚析构函数才能正确地绑定到每一个派生类中。

（2）基类有非虚构（non-virtual）析构函数时，delete 指向派生类的基类指针是未定义的。
{% highlight c++ linenos %}
// .h
class A {
public: 
    A();
    ~A();
}
class B : public A { ... }
// .cpp
int main()
{
    A* pb = new B();
    delete pb;	// undefined behevior
}
{% endhighlight %}
（3）C++11 中新增了 final 和 override 关键字，final 可以禁止派生类覆盖函数，override 可以检查基类是否存在同名虚函数，更详细的细节可参考《 C++ Primer 5th 》 15.3。
{% highlight c++ linenos %}
class A {
public:
    virtual void f1() const;
    void f2();
    void f3();
    void f4() final;
}
class B : public A {
public:
    void f1() const override; // Pass
    void f2() override;       // Error : no virtual match
    void f3();                // Pass
    void f4();                // Error : final matched
}
{% endhighlight %}
（4）含有纯虚函数的抽象基类，不能直接创建对象。为抽象基类添加纯虚析构函数，可以一箭双雕，让编译器自动为派生类生成虚构函数。
{% highlight c++ linenos %}
class A {
public:
    virtual ~A() = 0;	// function with "= 0" is a pure virtual function
}
A::~A() {}    // important !!!
{% endhighlight %}
**注意：纯虚析构函数必须要在类外部为其提供一份定义，否则连接器会报错**

```
Tip 08 : Prevent exceptions from leaving destructors.
```
（1）析构函数中若要调用释放资源的操作，必须为其处理异常，否则可能会影响剩下的析构处理。

（2）释放资源比较好的处理机制：提供给客户使用释放资源操作的接口，但同时记录其标志位，若客户未调用，则有析构函数自行处理。
{% highlight c++ linenos %}
class A {
public:
    void close() {
        // free resourse
        closed = ture;
    }
    ~A() {
        if (!closed) {
            // free resourse
        } catch (..) {
        }
    }
private:
    bool closed;
};
{% endhighlight %}

```
Tip 09 : Never call virtaul functions during construction or destruction.
```
构造函数与析构函数都处于函数的构造期，在这个时期，virtual 处于未绑定状态，若在构造函数与析构函数中直接调用虚函数，编译器将报错；若声明了纯虚函数并间接调用，会导致派生类无法绑定对应的纯虚函数。
{% highlight c++ linenos %}
class A {
public:
    A() { init(); }
    virtual void vfunc() const = 0;
private:
    void init() {
        vfunc();
    }
};
{% endhighlight %}

```
Tip 10 : Have assignment operators return a reference to *this.
```
operator= 返回引用可以实现连锁解析。
{% highlight c++ linenos %}
// .h
A& operator=(const A& rhs) {
    ...
    return *this;
}
// .cpp
// A a1, a2, a3;
a1 = a2 = a3;	// a1 = (a2 = a3);
{% endhighlight %}

``` 
Tip 11 : Handle assignment to self in operator=.
```
operator= 中的相等校验，可以防止赋值前的删除操作导致的指针失效，检查方法主要有三种：判断来源和目标地址、设计实现顺序以及拷贝交换（copy-and-swap）。其中前两者速度较慢，因为在每次相同校验中都产生了不同数量的内存操作指令，拷贝交换将操作放在类的构造函数中，能让编译器产生更高效的代码。
{% highlight c++ linenos %}
class A {
    B* pb;
};
// version 1 : identity test
A& operator=(const A& rhs) {
    if (this == &rhs) return *this;
    
    delete pb;
    pb = new B(*rhs.pb);
    return *this; 
}
// version 2 : exception safety
A& operator=(const A& rhs) {
    A* pold = pb;
    pb = new B(*rhs.pb);
    delete pold;
    return *this;  
}
// version 3 : copy-and-swap
A& operator=(const A& rhs) { // call operator=(A rhs) inside
    A tmp(rhs);
    swap(tmp);
    return *this;
}
/*
A& operator=(A rhs) {
    swap(rhs);
    return *this;
}
*/
{% endhighlight %}

```
Tip 12 : Copy all parts of an object.
```
拷贝构造函数和拷贝赋值构造函数不会提示元素未被复制，当调用是应仔细确认每一个元素都被成功复制了，尤其是在**派生类**中。
{% highlight c++ linenos %}
class A {
public:
    A(int d) : data(d) {}
    A(const A& rhs) : data(rhs.data) {}
    A& operator=(const A& rhs) {
        A tmp(rhs);
        std::swap(data, tmp.data); // impliement of swap(rhs)
        return *this;
    }

    virtual void print() { std::cout << data << std::endl; }
protected:
    int data;
};

class B : public A {
public:
    B(int d, int e) : A(d), element(e)  {}
    B(const B& rhs) : A(rhs), element(rhs.element) {}
    B& operator=(const B& rhs) {
        B tmp(rhs);                       // call A(const A& rhs) here 
        std::swap(element, tmp.element);  // implement of swap rhs
        return *this;
    }

    void print() { std::cout << data << " " << element << std::endl; }
private:
    int element;
};
{% endhighlight %}

&copy; 2015 plinx