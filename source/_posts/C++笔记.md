---
title: C++学习笔记(完结)
abbrlink: 1
categories: 学习笔记
tags: C++
updated: 2023-2-8
date: 2023-02-08
---

## 1.第二章

### 1.1 函数定义

c++中，定义函数的时候可以让**最右边的连续若干个**参数有[缺省值](https://so.csdn.net/so/search?q=缺省值&spm=1001.2101.3001.7020)，在调用函数的时候，如果不写相应位置的参数，则调用的参数就为缺省值。

<font color=red>***注意是右往左连续参数**</font>

```C++
int func(int a=1,int b,int c=2){} //不合法
```

<!-- more -->

### 1.2 函数重载

C++ 允许多个函数拥有相同的名字，只要它们的参数列表不同就可以，这就是函数的重载（Function Overloading）。

*重载的规则：*

- 函数名称必须相同。
- 参数列表必须不同（个数不同、类型不同、参数排列顺序不同等）。
- 函数的返回类型可以相同也可以不相同。
- 仅仅返回类型不同不足以成为函数的重载。

**C++如何实现重载的**

C++代码在编译时会根据参数列表对函数进行重命名，例如void Swap(int a, int b)会被重命名为_Swap_int_int，void Swap(float x, float y)会被重命名为_Swap_float_float。当发生函数调用时，编译器会根据传入的实参去逐个匹配，以选择对应的函数，如果匹配失败，编译器就会报错，这叫做重载决议（Overload Resolution）。

```
不同的编译器有不同的重命名方式，这里仅仅举例说明，实际情况可能并非如此。
```

从这个角度讲，函数重载仅仅是语法层面的，本质上它们还是不同的函数，占用不同的内存，入口地址也不一样。



**多个参数时的二义性**

在设计重载函数时，参数类型过少或者过多都容易引起二义性错误，因为这些类型相近，彼此之间会相互转换。

例如我们要设计几个重载函数来处理数值，数值包括小数和整数，如果只提供了以下两个函数原型：

```c++
void func(int);
void func(double);
```

那么下面的函数调用就会产生二义性错误：

```c++
long n = 1000;func(n);
```

n 是 long 类型，转换为 int 或 double 的优先级都是一样的，编译器不知道如何抉择。

如果添加一个函数原型`void func(long);`，或者去掉一个函数原型`void func(int);`，无论再怎么调用也不会出错了。



### 1.3 函数传值

函数的形参与实参结合的两种方式（传值、传址），其本质区别在于形参与实参结合时，是否在内存空间临时的创建新的内存区域并进行赋值操作。由此，也引发出了两种参数结合方式各自的优缺点，例如传值方式，效率较低，但安全性高；传址方式，效率高，但形参可影响实参的值，故安全性欠佳。

- 当函数的返回值是数组类型的，传递的是数组第一个元素的地址。
- 决定C++语言中函数的返回值类型的是在定义该函数时所指定的数据类型



### 1.4 内联函数

为了消除函数调用的时空开销，C++ 提供一种提高效率的方法，即在编译时将函数调用处用函数体替换，类似于C语言中的宏展开。**这种在函数调用处直接嵌入函数体的函数称为内联函数（Inline Function），又称内嵌函数或者内置函数。**

**注意，要在函数定义处添加 inline 关键字，在函数声明处添加 inline 关键字虽然没有错，但这种做法是无效的，编译器会忽略函数声明处的 inline 关键字。**

- 函数代码少、频繁调用适合使用内联函数
- 使用内联函数的目是提高程序运行速度



## 2.第三四五章

### 2.1构造函数

一个类的对象被创建的时候，编译系统对象分配内存空间，并自动调用该构造函数，由构造函数完成成员的初始化工作。*因此，构造函数的核心作用就是，**初始化对象的数据成员***

- 名字与类名相同，可以有参数，但是不能有返回值（连void也不行）。
- 构造函数是在实例化对象时自动执行的，不需要手动调用。
- 作用是对对象进行初始化工作，如给成员变量赋值等。
- 如果定义类时没有写构造函数，系统会生成一个默认的无参构造函数，默认构造函数没有参数，不做任何工作。
- 如果定义了构造函数，系统不再生成默认的无参构造函数。
- 对象生成时构造函数自动调用，对象一旦生成，不能在其上再次执行构造函数。
- 一个类可以有多个构造函数，为重载关系。
- 构造函数可以是内联函数

**分类**

- 按参数种类分为：无参构造函数、有参构造函数、有默认参构造函数
- 按类型分为：普通构造函数、拷贝构造函数（赋值构造函数）

**拷贝构造函数**：

- 一种特殊的构造函数，当对象之间复制时会自动调用拷贝构造函数。
- 若类中没有显示定义拷贝构造函数，则系统会自动生成默认拷贝构造函数。

- 类中可以存在超过一个拷贝构造函数。

使用场合：

1. 使用一个已经创建完毕的对象来初始化一个新对象
2. 值传递的方式给函数参数传值(C++中,值传递是将实参复制一份传给形参)
3. 以值方式返回局部对象(C++中,值方式返回也是将局部对象复制一份返回)

```c++
MyClass func() {MyClass tmp; return tmp;}
void func(MyClass a){}
int main() {
    MyClass a1,a2(a1);  //(1)对象初始化对象
    MyClass x; func(x); //(2)值传递的方式给函数参数传值
    func();             //(3)以值方式返回局部对象
}
```

拷贝构造函数的判断

```c++
X::X(const X&);   //拷贝构造函数
X::X(X); 
X::X(X&, int a=1);   //拷贝构造函数
X::X(X&, int a=1, int b=2);  //拷贝构造函数
```

**对于一个类X, 如果一个构造函数的第一个参数是下列之一:
  a) X&
  b) const X&
  c) volatile X&
  d) const volatile X&
  且没有其他参数或其他参数都有默认值,那么这个函数是拷贝构造函数.**



### 2.2 析构函数

- 析构函数名是 ~类名；
- 无参数无返回值；
- 一个类有且只有一个析构函数；
- 析构函数无法重载
- 析构函数的作用主要是释放对象占用的资源
- 程序执行结束前，调用析构函数将数组中各个对象元素释放，释放顺序与创建对象的顺序相反。



### 2.3 友元函数

**友元的目的** 就是让一个函数或者类 访问另一个类中的私有成员

**友元的关键字为** friend

**友元的三种实现**

- 全局函数做友元  
- 类做友元
- 成员函数做友元（包括构造和析构函数）

友元函数不是类的成员函数，只是独立于该类的一般函数。

### 2.4 友元类

友元类的所有成员函数都是另一个类的友元函数，都可以访问另一个类中的隐藏信息（包括私有成员和保护成员）。

定义方式 friend class 类名;

**注意事项**

- 友元关系不能被继承。 
- 友元关系是单向的，不具有交换性。若类B是类A的友元，类A不一定是类B的友元，要看在类中是否有相应的声明。
- 友元关系不具有传递性。若类B是类A的友元，类C是B的友元，类C不一定是类A的友元，同样要看类中是否有相应的申明

**设置友元可以提高程序运行的效率（减少了数据类型的判断）**



### 2.5常数据成员

**定义**  用const修饰的数据成员

```c++
const int  hour;//定义const为常数据成员
```

**初始化**  常数据成员必须被初始化，并且不能被更新。

不能采用在构造函数中对常数据成员进行赋初值的方法，因为常数据成员是不能被赋值的。

```c++
//构造函数只能用参数初始化列表对常数据成员进行初始化
Time ::Time (int h):hour(h){}//通过参数初始化表对常数据成员进行初始化
```

**PS：类的数据成员使用初始化列表进行的初始化才叫初始化，使用构造函数内部的语句初始化叫赋值**

**访问规则  **普通成员函数和常成员函数可以引用常数据成员，但是不能修改其值



### 2.6 常成员函数

**定义**  常成员函数是指由const修饰符修饰的成员函数

```c++
void getTime() const;//在声明和定义常成员函数时都要有const关键字，在调用时不必加
```

**访问规则** 

- 常成员函数可以引用常数据成员和普通数据成员，但是不能修改其值
- 常成员函数只能调用常成员函数，不能调用普通成员函数
- 普通对象可以调用常成员函数
- 可以重载

### 2.7 常对象

**定义** **两种方式1. const Point a 2.Point const a 两者效果一样**

**访问规则**

- 常对象可以调用常成员函数，不能调用非const成员函数；非const对象，可以调用普通成员函数和常成员函数
- 常对象成员函数不一定都是常成员函数，常对象的数据成员不一定都是数据成员
- 常对象一旦初始化，常对象的数据成员便不可允许修改



### 2.8 静态成员变量

在定义变量之前加 **static**关键字定义静态成员变量

**作用**  静态成员用以解决同一个类的不同对象之间成员，包括数据成员和函数成员的**共享问题**。

```c++
class test
{
private:
    static int my_value;		//定义类的静态成员变量

public:
 
    static int getValue()		//定义类的静态成员函数
    {
    	return my_value;
    }
};
 
int test::m_value = 12;		//类的静态成员变量需要在类外分配内存空间
 
int main()
{
    test t;
    cout << t.getValue() << endl;
}
```

**性质**

- 静态成员变量**属于整个类所有**
- 静态成员变量的**生命期不依赖于任何对象，为程序的生命周期**
- 可以**通过类名直接访问**公有静态成员变量
- **所有对象共享类的静态成员变量**
- 可以**通过对象名访问**公有静态成员变量
- 静态成员变量**需要在类外单独分配空间**



### 2.9 静态成员函数

**性质**

- 静态成员函数**是类的一个特殊的成员函数**
- 静态成员函数**属于整个类所有，没有this指针**
- 静态成员函数**只能直接访问静态成员变量和静态成员函数，不能访问非静态数据成员**
- **可以通过类名、对象名访问类的公有静态成员函数**

|                        |       静态成员函数        |       普通成员函数        |
| :--------------------: | :-----------------------: | :-----------------------: |
|      所有对象共享      |            YES            |            YES            |
|     隐含 this 指针     | <font color=red>NO</font> |            YES            |
| 访问普通成员变量(函数) | <font color=red>NO</font> |            YES            |
| 访问静态成员变量(函数) |            YES            |            YES            |
|    通过类名直接调用    |            YES            | <font color=red>NO</font> |
|   通过对象名直接调用   |            YES            |            YES            |



### 2.10 this指针

每个成员函数 (包括构造和析构）都有一个this指针。

在成员函数内部，this指针指向调用对象，即可以通过this关键字访问当前对象的成员。

```
访问成员变量：this->成员名;
访问成员函数：this->函数名();
```

**this是指向对象的指针，而*this则指向对象的本体**

- this指针在成员函数执行前就已经存在，在析构后清除。
- this指针的类型为类类型*const（类名 *const）为右值。
- this指针本身不占用大小，他并不是对象的一部分，不会影响sizeof()的结果。
- this的作用域在类成员函数的内部。
- this指针是类成员函数的第一个默认隐含参数（const *类名 this），编译器自动维护传递，类编写者不能显式传递。
- 只有在类的非静态成员函数中才可以使用this指针，其他任何函数都不可以。
- this指针作用：（1）解决名称冲突（2）可以用来返回当前对象的引用。
- 任何对类成员的直接访问（使用成员访问运算符::）都可被看作是对this的隐式引用。



### 2.11 其他知识点

类成员默认的访问属性为private

若new一个对象数组,那么用delete释放时应该写[]，否则只delete一个对象(调用一次析构函数)。



## 3.第六章

### 3.1 继承

**定义**： 继承(inheritance)机制是面向对象程序设计中使代码可以复用的最重要的手段，它允许程序员在保持原有类特性的基础上进行扩展，增加功能。这样产生的新类，称派生类（或子类），被继承的类称基类（或父类）。

**继承方式**  

```c++
class 新类的名字:继承方式 继承类的名字{};
```

默认继承方式为private

**访问权限** 

|   类成员\继承方式   | public继承 | protected继承 | private继承 |
| :-----------------: | :--------: | :-----------: | :---------: |
|  基类的public成员   |   public   |   protected   |   private   |
| 基类的protected成员 | protected  |   protected   |   private   |
|  基类的private成员  |   不可见   |    不可见     |   不可见    |

**注意事项**

- 基类的构造函数和析构不能被派生类继承
- 由多个基类派生时,基类构造函数的调用顺序,取决于定义派生类时,派生列表中的基类声明顺序
- 在创建派生类对象时，构造函数的执行顺序是基类构造函数、对象成员构造函数、派生类本身的构造函数
- 派生类的对象可以赋值给基类的对象，反之，则不可以





## 第七章

虚函数具有继承性。

（说好的不考呢）
