---
layout:	post
title:	Effective C++ Notes - 6
---

## 5. Implementations ##

```
Tip 26 : Postpone variable definitions as long as possible.
```

（1）考虑数据的定义时间；（2）考虑数据构造函数的消耗。

{% highlight c++ linenos %}
// version A1 : worst
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    string encrypted;       // too early to definition
    if (password.length() < MinimumPasswordLength) {
        throw logic_error("Password too short");
    }
    encrypted = password;
    return encrypted;
}

// version A2 : better
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    if (password.length() < MinimumPasswordLength) { ... }
    string encrypted;       
    encrypted = password;   // value assign slower than assignment constructor
    return encrypted.
}

// version A3 : best
std::string encryptPassword(const std::string& password)
{
    ...
    string encrypted(password); // use assignment constructor
    return encrypted;
}

// value definition for loop
// version B1 : worse
for (int i = 0; i < num; i ++) {
    Widget w;
    ...
}

// version B2 : better
Widget w;
for (int i = 0; i < num; i ++) {
    ...
}
{% endhighlight %}

```
Tip 27 : Minimize casting.
```

// 暂未深入理解

（1）尽量避免使用转型，特别是在注重效率的代码中避免 dynamic_casts，如果可以，试着用
无需转型的替代设计；

（2）如果转型是必要的，试着将它隐藏在某个函数背后。客户随后可以调用该函数而不需要将转型
放进他们的代码内；

（3）宁可使用 C++-style 转型，不要使用旧式 C-style 转型。

```
Tip 28 : Avoid returning "handles" to object internals.
```

尽量不要返回指向对象数据的指针，否则调用者可能跨过权限修改数据信息。

{% highlight c++ linenos %}
class Point {
public:
    Point(int x, int y);
    void setX(int x);
    ...
};

struct RectData {
    Point ulhc;     // upper left-hand corner
    Point lrhc;     // lower right-hand corner
};

// Version A
class Rectangle {
public:
    Point& upperLeft() const { return pData->ulhc; }    // !!! Warning
private:
    std::shared_ptr<RectData> pdata;
};

Point corrd1(0, 0);
Point corrd2(100, 100);
const Rectangle rect(coord1, coord2);

rect.upperLeft().setX(50);  // rect.upperLeft() return Point&

// Version B
class Rectangle {
public:
    const Point& upperLeft() cosnt { return pData->ulhc; }
    ...
};

rect.upperLeft().setX(50);  // compiler error with const
{% endhighlight %}

使用指向对象数据的指针还可能在对象析构后产生野指针。

```
Tip 29 : Strive for exception-safe code.
```

// Learn it later

```
Tip 30 : Understand the ins ans outs of inlining.
```

一个表面上看似 inline 的函数是否真的是 inline，取决于编译器。实际上，一个函数是否可以被
inline 是可以推测的。inline 是编译期行为，而编译期的函数必须具有确定性才可以被编译。

- 80-20 法则：一个程序 80% 的执行时间花费在 20% 的代码上。

（1）inline 会带来代码膨胀，只有找到最耗时的 20% 的代码，优化才是有意义的；

（2）不要因为 function templates 出现在头文件，就将他们声明为 inline。

```
Tip 31 : Minimize compilation dependecies between files.
```

// Pass



&copy; 2015 plinx