# 06 重载括号、new、delete

## 1、重载括号，可以把它当成仿函数（stl中常用）使用

**仿函数(functor) ，就是使一个类的使用看上去像一个函数一样，其实现是类中实现一个operator()，这个类就有了类似函数的行为，即实现了一个仿函数类**

```cpp
class:
    int operator()(int a1, int a2)
    {
        return a1 * a2;
    }
    重载小括号，将对象当成普通函数调用，将这种对象称为：仿函数
main:
	int ans = a(1, 2); // a对象仿函数调用
	也可以通过如下调用： int ans = a.operator()(1, 2);
```

`new int ：new接收的其实是长度 `

## 2、void 与 void * 区别

```
两者无关系，
void 表示无返回值
void * 表示返回万能指针
```

## 3、重载new、delete

**重载new**

```cpp
 void *operator new(size_t size)
    {
        cout << "重载new" << endl;
        return malloc(size);
    }
malloc函数接收的对象是sizeof(type * n),是一个无符号数
```

**重载delete**

```cpp
   // delete传入万能指针
    void operator delete(void *p)
    {
        cout << "重载delete" << endl;
        if (p != nullptr)
        {
            free(p);
            p = nullptr;
        }
        return;
    }
```

**main**

```cpp
    A *p = new A(10);
    delete p;
```

**重载后的new还是会调用构造函数，delete也是如此**

```
重载后的new{} 、 delete[]与原来的相同
调用时稍有区分：
    A *p = new A[10]; // 开辟十个A的无参构造的对象
    // p->operator new[](sizeof(A[10]));
```

## 4、不建议重载 并且、或者 操作符

**重载"&&" "||" 不会发生短路**

```cpp
在原来的语法里 "a && b" "a || b" 存在短路原则，一旦满足条件，后面半句就不执行。
而在类中重载"&&"操作符的话
eg: bool operator&&(Test & a) {
    if (this->a && a.a) return true;
    else return false;
}

A a(0), b(20);
if (a && (a + b)) // 理论上 a + b 不会被执行
    但传入时实则是：a.operator&&(a + b), 会先执行a + b， ”||“也一样
```

## 5、自定义string类

string不是普通数据类型，其为一个类

```cpp
Test(int a, char * name) {
    this->a = a;
    this->name = name;
}
要想支持this->name = name, 需要自定义string类
string name;

```

string类内部已经考虑了深拷贝
