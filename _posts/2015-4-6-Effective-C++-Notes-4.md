---
layout:	post
title:	Effective C++ Notes - 4
---

## 3. Resourse Management ##

```
Tip 13 : Use objects to manage resources.
```

C++ 允许动态管理内存，若资源分配后未进行释放，将造成内存泄露。即使在程序中使用了 delete 操作，仍可能因为程序的其他异常导致释放操作未执行。
{% highlight c++ linenos %}
void func() {
    Factory* factory = new concreteFactory();
    ...    // 若抛出异常，可能会造成 delete 操作未执行
    delete factory;
}
{% endhighlight %}
《 C++ Primer 5th 》 12.1 介绍了 shared_ptr 、unique_ptr 和 weak_ptr，使用 shared_ptr 可以对程序进行如下修改。
{% highlight c++ linenos %}
void func() {
    shared_ptr<Factory*> factory(ConcreteFactory());
    ...
} // 即使函数内部抛出异常，shared_ptr 的析构函数也将自动删除 factory
{% endhighlight %}
关于 unique_ptr 和 weak_ptr　见书。

```
Tip 14 : Think carefully about copying behavior in resource-managing classes.
```
对带有资源的类进行拷贝时，将会改变资源的状态。
{% highlight c++ linenos %}
// .h
void lock(Mutex* pm);    // 锁定互斥锁
void unlock(Mutex* pm);  // 接触互斥锁
class Lock {
public:
    explict Lock(Mutex* pm) : mutexPtr(pm) { lock(mutexPtr); }
    ~Lock() { unlock(mutexPtr); }
private:
    Mutex* mutexPtr;
};
// .cpp
int main()
{
    ...
    Lock mlock1(&m);        // lock mutex
    Lock mlock2(mlock1);    // copy mutex and don't know what would happen
    ...
}
{% endhighlight %}
控制带有资源的类的拷贝行为，有两种方法：

（1）禁止复制（2）使用智能指针进行管理

禁止复制可以参考条款 6，使用指针可以参考条款 13。
{% highlight c++ linenos %}
// prohibit copying version
class Uncopyable {
public:
    Uncopyable(const Uncopyable&) = delete;
    Uncopyable& operator=(const Uncopyable&) = delete;
};
class Lock : public Uncopyable {
};

// smart point reference-counting version
class Lock {
public:
    Lock(Mutex* pm) : mutexPtr(pm, unlock) { lock(mutexPtr.get()); }
private:
    shared_ptr<Mutex*> mutexPtr;
}
{% endhighlight %}

``` 
Tip 15 : Provide access to raw resources in resource-managing classes.
```
使用智能指针时，若函数 API 需要调用原始资源（raw resources），可以使用两种方法：

（1）显示转换（2）隐式转换

shared\_ptr 和 auto\_ptr 都提供了 get 成员函数，显示地返回指针对象。
{% highlight c++ linenos %}
shared_ptr<Factory*> factory(ConcreteFactory());
// int getProductId(const Factory* f);
int ProductId = getProductId(factory.get()); 
{% endhighlight %}
若使用自定义的类进行资源调用时，可以通过实现 get 进行显示转换，实现 operator() 进行隐式转换。
{% highlight c++ linenos %}
class Font {
public:
    explict Font(FontHandle f) : fh(f) {}
    ~Font() { releaseFont(f); }
    
    FontHandle get() const { return f; }      // better idea
    operator FontHandle() const { return f; } // not a good idea
private:
    FontHandle fh;
}
{% endhighlight %}

```
Tip 16 : Use the asme form in corresponding uses of new and delete.
```

在 new 表达式中使用了 []，在 delete 中也应该对应使用 []，反之亦然。
{% highlight c++ linenos %}
std::string* strPtr1 = new std::string;
std::string* strPtr2 = new std::string[100];

delete strPtr1;
delete [] strPtr2;
{% endhighlight %}
更详细的细节可参考 《 C++ Primer 5th 》 12.2。

```
Store newed objects in smart pointers in standalone statements.
```
若给一个函数传入即时动态申请的智能指针，可能将造成资源泄漏。
{% highlight c++ linenos %}
// .h
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);

// .cpp
int main()
{
    // compile error.
    processWidget(new Widget, priority());
    // compile success but not safe
    processWidget(std::shared_ptr<Widget>(new Widget), priority());
    // better choice
    std::shared_ptr<Widget> pw(new Widget);
    processWidget(pw, priority());
}
{% endhighlight %}

&copy; 2015 plinx