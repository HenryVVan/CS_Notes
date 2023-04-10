## 04、构造函数初始化列表与new、delete、static...

## 1、构造函数初始化列表

```cpp
class A {
public:
	A(int a) {
		m_a = a;
	}
	~A() {
		cout << "析构" << endl;
	}
	
private: int m_a;
}
class B {
public:
B(A &a1, A &a2, int b) {
		m_b = b;
		// m_a1(a1); // 错误，此处将m_a1误当构造函数使用，实际上应该是 A m_a1(a1); 但是private中已经初始化过了a1
	}

private: int m_b;
			A a1;
			A a2;
}
// 该操作在C++中不被允许
// 所以需要构造函数的初始化列表
修改后的代码如下：
B(A &a1, A &a2, int b): m_a1(a1), m_a2(a2) {
	m_b = b;
}
上面是调用A中的析构: A m_a1(a1); = A m_a1 = a1;
```

```cpp

class A
{
public:
    A(int a)
    {
        cout << "A 构造" << endl;
        m_a = a;
        cout << m_a << endl;
    }
    void show()
    {
        cout << m_a << endl;
    }
    ~A()
    {
        cout << "A 析构" << endl;
    }

private:
    int m_a;
};

class B
{
public:
    B(A &a1, A &a2, int b) : m_a1(a1), m_a2(a2)
    {
        cout << "B 构造" << endl;
        m_b = b;
    }
    // 构造对象成员的顺序跟初始化列表的顺序无关
    // 与成员对象的定义顺序有关
    B(int a1, int a2, int b) : m_a1(a1), m_a2(a2)
    {
        cout << "赋值" << endl;
        m_b = b;
    }
    void showB()
    {
        cout << "b = " << m_b << endl;
        m_a1.show();
        m_a2.show();
    }
    ~B()
    {
        cout << "B 析构" << endl;
    }

private:
    int m_b;
    A m_a2;
    A m_a1;
};

void test()
{
    // A a1(1), a2(2);
    B b(1, 2, 3);
    b.showB();
}

```

**构造对象成员初始化的顺序跟初始化列表的顺序无关**
**与成员对象的定义顺序有关**

进阶，一个类的多参数作函数初始化列表

```cpp
class ABC
{
public:
    ABC(int a, int b, int c)
    {
        m_a = a;
        m_b = b;
        m_c = c;
    }
    void show()
    {
        cout << m_a << endl;
    }

private:
    int m_a, m_b, m_c;
};

class ABCD
{
public:
    ABCD(int a1, int b1, int c1, int d) : m_abc(a1, b1, c1)
    {
        m_d = d;
    }
    void show()
    {
        cout << m_d << endl;
        m_abc.show();
    }

private:
    int m_d;
    ABC m_abc;
};
D类的初始化顺序，先m_d 后 m_abc
```

在类的private声明变量时，一定不能赋值，const int a（在类里不开空间，创建对象时才开）; 都不能在那赋值，需要在构造函数里赋值。

```cpp
常量赋值只能写在初始化列表里

private:
const int a;

x(int .... int a) :x(a)
只能写在初始化列表中，不可以写在函数里
```

## 2、在构造函数中调用构造函数是一个危险的行为

```cpp
class T
{
public:
    T(int a, int b, int c)
    {
        a = m_a;
        b = m_b;
        c = m_c;
    }
    T(int a, int b)
    {
        m_a = a;
        m_b = b;
        T(a, b, 10); // 调用三形参的构造，会产生一个匿名的新对象
        // 匿名创建完没有载体，立刻被析构
    }
    int get_c()
    {
        return m_c;
    }

private:
    int m_a, m_b, m_c;
};

main: T t1(1, 2); cout <<  t1.getc(); 输出 0，两参构造函数中创建的匿名对象没有载体，直接就被析构。
```

## 3、new、delete

```cpp
与C中的malloc、free类似
但new、delete是关键字，不是函数

// malloc 分配十个int空间给数组
malloc: (int *) *p = (int *) malloc (sizeof(int) * 10);

new:
int *p = new int;
*p = 10;

// 删完空间要记得指空
if (p != nullptr) {
	delete p;
	p = nullptr;
}

// new 分配10个int空间
int *p = new int[10]; // 支持[]、{}，不支持()
// new int(10); 这个不是开辟空间， 是把那个int赋值为10
// int a(10); == int a = 10; int 本身也是一个类

if(p != nullptr) {
	delete[] p; // delete后面要加对应符号
	p = nullptr;
}

```

**new在堆上开辟空间时，可以触发对象的构造函数，而malloc不可以**

**free 和 delete在成员不分配新内存时，可以互相混用，但不推荐**

**free 不会触发析构函数， delete会触发**

```cpp
// 分配空间函数与C++中关键字的区别
eg1:new 在分配时可以触发构造函数，malloc不可以
void t()
{
    // int *p = new int[10];
    T *p = new T(1, 2, 3);
    p->print();
    if (p != nullptr)
    {
        delete p;
        p = nullptr;
    }
}

eg2: delete 会触发析构，free不会（可能存在内存泄漏），且关键字会比函数调用效率稍微高一点
T *p = new T(10, "wxy")
在内存中分布
栈			堆			堆
p ->		10
			char *p		 "wxy"
析构会释放char *所指向的内存
然后delete可以触发析构，而free不会，他俩只能删除栈中对象所指向的内容，在这种情况下，使用free会造成内存泄漏。

```

## 4、静态成员变量和静态成员函数

```cpp
// 静态成员变量与静态成员函数都属于类，不属于具体对象
void test() {
	static int a = 1;
	a ++;
}
test(); -> 11
test(); -> 12, static C中只初始化一次，静态变量区

static void test() {

} // 该函数只在当前文件有效，在其他文件中不可以调用这个方法

class T 
private:
	static int a;
	int b;
静态变量a是在静态区分配的，不属于某一对象，创建新对象时，不会为其再开辟空间，而b就会继续开空间

静态成员变量初始化，需要在类外如下赋值：int T::a = 10;

注意静态成员变量static不是const

    // 想返回局部变量，需要加上&，将其变为左值
    static int &get_c()
    {
        m_c++;
        return m_c;
    }
	private:
    	int m_a;
    	static int m_c;
    在类外先赋初值：int T::m_c = 1;
    
    
    T t(1);
    // t.get_c() = 3;
    T::get_c() = 3;
    // cout << t.get_c() << endl;
    cout << T::get_c() << endl; // 4
    
```

## 5、static练习

```cpp
1、static 成员变量实现了同类对象间信息共享
2、static 成员是在类外存储，不计算在类的大小里，一开始就在常量区为其开辟了空间
3、static 成员是命名空间，属于类的全局变量，存储在data区
4、static 成员只能类外初始化
5、可以通过类名直接访问（不生成对象也可），也可以通过对象去访问
```

**统计学生人数与均分**

```cpp

#include <iostream>
using namespace std;

void test()
{
    static int a = 1;
    a++;
    cout << a << ' ';
}

class T
{

public:
    T(int a)
    {
        m_a = a;
    }

    static int &get_c()
    {
        m_c++;
        return m_c;
    }

private:
    int m_a;
    static int m_c;
};
int T::m_c = 2;

class Student
{
public:
    Student(int id, double score)
    {
        m_id = id;
        s_score += score;
        m_num++;
    }
    static int num()
    {
        return m_num;
    }
    static double score()
    {
        return s_score;
    }

private:
    int m_id;
    static int m_num;
    static double s_score;
};
int Student::m_num = 0;
double Student::s_score = 0.0;

int main()
{
    Student s1(1, 70);
    Student s2(2, 90);
    Student s3(3, 100);
    cout << Student::num() << endl;
    cout << Student::score() / Student::num() << endl;
    return 0;
}
```

## 6、static占用大小

```
类的大小 只有 普通成员变量有关，与函数，静态成员变量均无关
函数也不属于对象
class C:
	get() // 不占内存，不属于对象
	
main: 
	C c1, c2;
	c1.get(); c2.get(); // 为什么可以get到对应的值
	
	c1 占的内存				c2 占的内存
				
				get()：与他俩无关
	
```

