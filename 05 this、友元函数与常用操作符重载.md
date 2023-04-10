## 05 this、友元函数与常用操作符重载

## 1、this**指针**

**this就是指向调用该成员函数方法的非静态对象的地址**

`this` 指针不是`const Test *`，它是`Test* const `

```cpp
用对象的get方法时，编译器翻译时会传入对象地址，进而可以直接找到

类的成员函数 尾部的const 修饰this指针，表示指针指向内容也不可修改

一般而言：this指针是一个对象的常指针，指向不可变，指向内容可改
class T
{
public:
    T(int a)
    {
        this->m_a = a;
    }
    int get_a() const // 在类成员函数后+const，this指向内容也不可修改
    this相当于： Test * const // 指向不可变，内容可变
    {
        // this ++; //
        // this->m_a = 10;
        return this->m_a;
    }

private:
    int m_a;
};

```

## 2、对象返回本身

**静态成员函数只能访问静态数据成员。因为非静态成员函数，在调用this指针被当成参数传递，而静态成员函数属于类，不属于对象，所以没有this指针**

```cpp
static int s_getK() {
	return k; // k需要是静态成员变量，static成员函数没法返回非静态的变量，因为只有对象才有this指针，而static属于类
	return m; // 错误
}
static int k;
int m;
非静态成员函数可以返回类中所有变量，静态成员函数只能返回静态变量
```

**在一个成员方法中，如果想返回一个对象本身，用*this实现***

```
    Test &add(Test &another)
    {
        this->b += another.b;
        this->c += another.c;
        return *this; // 返回对象本身
    }
    // 用this来区分成员函数
    Test(int b, int c)
    {
        this->b = b;
        this->c = c;
    }
    对一个对象连续调用成员函数，每次需要修改对象，需要将成员方法定义成引用类型
    t1.add(t2).add(t2);
```

## 3、自定义数组类

````cpp
A.h
#pragma once

class A
{
public:
    A(int len);
    A(const A &another);
    void set();
    void show();
    ~A();

private:
    int len;
    int *arr;
};
```

```cpp
A.cpp
#include "A.h"
#include <iostream>

using namespace std;
A::A(int len)
{
    this->len = len;
    this->arr = new int[this->len];
}
A::A(const A &another)
{
    this->len = another.len;
    this->arr = new int[another.len];
    for (int i = 0; i < this->len; i++)
    {
        this->arr[i] = another.arr[i];
    }
}
void A::set()
{
    for (int i = 0; i < len; i++)
    {
        arr[i] = i;
    }
}
void A::show()
{
    for (int i = 0; i < len; i++)
    {
        cout << arr[i] << ' ';
    }
    cout << endl;
}
void A::operator=(const A& another) {
    this->len = another.len;
    this->arr = new int[another.len];
    for (int i = 0; i < this->len; i++)
    {
        this->arr[i] = another.arr[i];
    }
}
A::~A()
{
    if (this->arr != nullptr)
    {
        delete (this->arr);
        this->arr = nullptr;
    }
}

```

## 4、友元函数

**友元提高了程序的运行效率（减少了类型检查和安全性访问，这些原来都需要时间开销），但它破坏了类的封装性和隐藏性，使得非成员函数可以访问类的私有成员变量**

```cpp
频繁需要访问类中私有成员变量的函数，可以在类中将其声明为友元函数，友元函数可以访问类中的私有变量，而不需要用类提供的接口中拿数据（减少压栈、出栈的过程）
    private:
    	int a, b;  

    friend int dis(B &a, B &b);
    
	int dis(B &a, B &b)
    {
        int x = a.a - b.a;
        int y = a.b - b.b;
        return x * x + y * y;
    }
```

**不用轻易的去使用friend去引用另一个类的成员方法**

**不建议**

```
// 先声明需要被访问的类
class B;

// 声明访问类及其内部函数
class C
{
public:
    int get(B &a);
};
 
class B
{
public:
    // 友元函数声明
    friend int C::get(B &a);
    B(int a, int b)
    {
        this->a = a;
        this->b = b;
    }

private:
    int a, b;
};

// 实现访问类中具体函数
int C::get(B &a)
{
    return a.a;
}
```

## 5、友元类和友元关系性质

**友元是单向性的，不具有交换性、传递性、也不能被继承**

a是b的友元，c是a的友元，c并不是b的友元

```
class B
{
public:
    //友元类声明
    friend class D;


class D
{
public:
    D(int d)
    {
        this->d = d;
        B b(10);
        cout << b.a << ' ' << this->d << endl;
    }

private:
    int d;
};

```

````

## 4、操作符重载

```cpp
'=' 重载： 全局操作符重载、类中操作符重载留一个即可，否则需要使用显式调用
class Complex
{
public:
    friend Complex operator+(Complex &a, Complex &b); // 1
    Complex(int a, int b)
    {
        this->a = a;
        this->b = b;
    }
    void print()
    {
        cout << this->a << " + " << this->b << 'i' << endl;
    }
    // 类中
    Complex & operator+(Complex &another) // 2
    {
        this->a + another.a;
        this->b + another.b;
        return *this;
    }

private:
    int a, b;
};

// 全局
Complex operator+(Complex &a, Complex &b)
{
    Complex tmp(a.a + b.a, a.b + b.b);
    return tmp;
}

    Complex z = x + y;
    // 这样隐式重载，编译器不知道调用 1 还是 2
    // 应该用显式的方式去调用
    eg:
    全局调用方式： operator+(x, y);
    类中函数调用方式： x.operator(y);
    
```

## 5、操作符重载的规则

**不允许重载的操作符：**

```
'.'：.成员选择符
'.*'：eg *p,成员对象选择符
'::'：域解析作用符
'?:'：三目运算符 
```

**重载不能改变运算符运算对象**（**即操作数**)**个数**

 **重载不能改变运算符的优先级别**

**重载不能改变运算符的结合性**

**操作符重载不能有默认参数**：**有默认参数会改变操作数个数**

 **重载的运算符必须和用户自定义类型的对象一起使用**，**其参数至少应有一个是类对象**（**或为类对象的引用**）

```
即参数不能全部是C++的标准类型，防止用户修改用于标准数据成员的运算符的性质，eg：
int operator+(int a, int b) {
	return a - b;
} // 此类行为在C++种绝对是禁止的
如想实现，需要使用自定义类型
```

 **类对象的运算符一般都需要重载**，**有两个特例**，一般"**=**"和"****&****"不需要用户重载

 **应该让重载运算符的功能与该操作符作用于标准数据类型时所实现的类似**

 **运算符重载函数可以是类的成员函数**，**也可以是类的友元函数**，**还可以是既非类的成员函数也不是友元的普通函数**

## 6、单目、双目运算符重载

```cpp
前++与后++ 重载的区别在于，是否用int占位
重载 前++： 可以连加
类中:
	Complex & operator++()
    {
        this->a++;
        this->b++;
        return *this;
    }
	
类外： 需先声明为友元函数
    Complex &operator++(Complex &a)
    {
        a.a++;
        a.b++;
        return a;
    }
    
重载 后++ 不可以连加，使用返回的对象要用const修饰，防止被修改
类中： 需用哑元加以区分
	const Complex & operator++(int) 
    {
        Complex tmp(this->a, this->b);
        this->a++;
        this->b++;
    	return tmp;
    }
    
类外：
	const Complex & operator++(Complex &a, int)
    {
        Complex tmp(a.a++, a.b++);
        return tmp;
    }
```

## 7、左移、右移操作符重载

**类中常用**

**<<**

```cpp
在类中重载
// 重载 <<
    ostream &operator<<(ostream &os)
    {
        os << this->a << " + " << this->b << " i " << endl;
    }
调用时：a.operator<<(cout); 
与cout常见用法不同，故不应该将其重载为类的成员函数
在全局重载
    ostream &operator<<(ostream &os, Complex &c)
{
    os << c.a << " + " << c.b << " i" << endl;
}
```

**>>**

```cpp
istream &operator>>(istream &is, Complex &c)
{
    cout << "c.a:" << endl;
    is >> c.a;
    cout << "c.b" << endl;
    is >> c.b;
}
```

## 8、等号操作符重载

```cpp
需要重载为成员函数，不能是全局函数
   Student &operator=(Student &s)
    {
        // 传入自身，不用赋值，此外赋值可以连赋，所以需要引用
        if (this == &s)
        {
            return *this;
        }
        // 将原来指针所指向的内存释放
        if (this->name != nullptr)
        {
            delete[] this->name;
            this->id = 0;
            this->name = nullptr;
        }
        // 深拷贝
        int len = strlen(s.name);
        this->name = new char[len + 1];
        strcpy(this->name, s.name);
        this->id = s.id;
    }
```

