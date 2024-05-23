---
title:       "C++强制类型转换"
subtitle:    ""
description: ""
date:        2024-05-23T9:38:45+08:00
author:      "tsand"
image:       ""
tags:        ["c++"]
categories:  ["Tech" ]
---
c/c++类型转换种分为两种：隐式类型转换（编译器）、显示类型转换（强制类型转换）

## c 类型转换
c语言的类型转换主要有两种：
* (new-type) expression
* new-type (expression)

### 局限性
* 转换随意，eg: 指向const对象的指针可随意换成指向非const对象的指针，指向基类对象的指针转换成指向派生类对象的指针。
* 缺乏关键字与标识符，不易代码排查

## c++ 强制类型转换
* static_cast: 
* dynamic_cast
* const_cast
* reinterpret_cast
### 用法
强制类型转换运算符<目标类型>（待转换表达式）
### static_cast

* 基本类型转换，例如 int 转换为 float。（向上转换）
* 类层次结构中上行转换（从派生类到基类，安全）。
* 类层次结构中下行转换（从基类到派生类，但不检查安全性，不安全不建议使用）。
```
// 上行转换，派生类→基类
Derive* d = new Derive();
Base* b = static_cast<Base*>(d);

// 下行转换，基类→派生类
Base* b = new Base();
Derive* d = static_cast<Derive*>(b);
```
* 移除指针的 const/volatile 修饰符（但不添加）。
* 把空指针转换成目标类型的空指针。
* 把任何类型的表达式转换成void类型。

示例：
```
int a = 10;
double b = static_cast<double>(n);
int* d = &a;
void* c = static_cast<void*>(d); //任意类型转换成void类型

Base* basePtr = new Derived();
Derived* derivedPtr = static_cast<Derived*>(basePtr);
```
不可使用
* 不同类型的指针之间互相转换
* 不同类型的引用之间的转换
* 整型和指针之间转换
  
优点

编译时转换，性能高。
可读性强，语法清晰。

缺点

无法进行运行时类型检查，可能导致不安全的类型转换
## dynastic_cast
只有在派生类之间转换时才使用dynamic_cast，type必须是类指针，类引用或者void*。

* 基类必须要有虚函数，因为dynamic_cast是运行时类型检查，需要运行时类型信息，而这个信息是存储在类的虚函数表中，只有一个类定义了虚函数，才会有虚函数表（如果一个类没有虚函数，那么一般意义上，这个类的设计者也不想它成为一个基类）。

* 对于下行转换，dynamic_cast是安全的（当类型不一致时，转换过来的是空指针），而static_cast是不安全的（当类型不一致时，转换过来的是错误意义的指针，可能造成踩内存，非法访问等各种问题）
  
```
Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
if (derivedPtr) {
    // 转换成功，可以安全使用 derivedPtr
} else {
    // 转换失败，处理错误情况
}
```

```
class BaseClass 
{
public:
　　int m_iNum;
　　virtual void foo(){};//基类必须有虚函数。保持多台特性才能使用dynamic_cast
};

class DerivedClass: public BaseClass 
{
public:
　　char *m_szName[100];
　　void bar(){};
};
　　
BaseClass* pb = new DerivedClass();
DerivedClass *pd1 = static_cast<DerivedClass *>(pb);//子类->父类，静态类型转换，正确但不推荐

DerivedClass *pd2 = dynamic_cast<DerivedClass *>(pb);//子类->父类，动态类型转换，正确

BaseClass* pb2 = new BaseClass();
//父类->子类，静态类型转换，危险！访问子类m_szName成员越界
DerivedClass *pd21 = static_cast<DerivedClass *>(pb2);

//父类->子类，动态类型转换，安全的。结果是NULL
DerivedClass *pd22 = dynamic_cast<DerivedClass *>(pb2);
```

优点

运行时类型检查，保证转换的安全性。
尤其适用于多态环境。

缺点

运行时开销较高。
需要启用 RTTI（运行时类型信息），增加了程序的复杂性。

## const_cast
用途(去除常量行为较危险)

* 添加或移除 const/volatile 修饰符。
* 常用于需要修改本应不可变的数据时，例如第三方库的 API 不接受 const 数据。
* 常量指针转换为非常量指针，并且仍然指向原来的对象
* 常量引用被转换为非常量引用，并且仍然指向原来的对象

```
struct SA 
{
　　int i;
};

const SA ra;
//ra.i = 10; //直接修改const类型，编译错误
SA &rb = const_cast<SA&>(ra);
rb.i = 10;

const int* constPtr = &i;
int* modifiablePtr = const_cast<int*>(constPtr);

```
优点

允许在非常特殊的场景下修改常量数据。

缺点

* 可能导致未定义行为，如果用于去除真正的常量属性。
* 易引入错误，应谨慎使用。

## reinterpret_cast（尽量不使用）

用途

* 强制转换指针类型，几乎可以在任何指针类型之间转换。
* 用于实现底层操作，例如位操作、系统调用、硬件编程等。

```
int* intPtr = new int(10);
char* charPtr = reinterpret_cast<char*>(intPtr);
```

优点

提供极大的灵活性，可以进行几乎任意的类型转换。
缺点

* 极易引入不安全的代码，可能导致未定义行为。
* 可读性差，难以维护和调试。


总结

1. static_cast 适用于大多数类型安全的显式转换，效率高。
2. dynamic_cast 适用于需要运行时类型检查的多态环境，确保类型安全。
3. const_cast 用于移除或添加常量属性，适用于特定场景。
4. reinterpret_cast 适用于底层操作，但容易引发错误，应谨慎使用。

## 引用

1. https://blog.csdn.net/chen134225/article/details/81305049


