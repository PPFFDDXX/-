### 1.简答

##### c和c++

- 超集

- c++支持c所支持的全部编程技巧

- 任何c程序都能被c++用基本相同的方法编写，并具备同等开销（时间、空间）

##### c的理念

- 程序员需要被信任

- 注重效率

- 实用性>艺术性严谨性

##### c++的原则

尽量使用const

使用析构防止内存泄漏

基类为抽象类

避免可能出现的模糊性

exception-safe code

不要多态地使用数组

##### 影响表达式求值的因素

操作符，操作数，优先级，结合性，求值顺序，类型转换约定

表达式副作用：如x=a++;破坏可移植性，降低可读性

##### 多态

同一个元素在不同的论域中有不同的解释

- 模板
  
  - 是一种源代码复用机制

- 结构化：
  
  - union
  
  - 函数指针
    
    - 额外参数
    
    - 大量指针运算
    
    - 实现起来复杂
    
    - 可读性差

- OO：
  
  - virtual
  
  - 重载：
    
    - 需要定义的重载函数太多
    
    - 定义不全

##### 代码重用

继承

inline

宏的缺陷

- 没有类型检查

- 只能实现简单的功能

- 重复计算

（Template、）

##### 函数传参

call by value

call by referrence

call by name

call by value-result

cdecl

stdcall

fastcall

##### 头文件中该写什么

- extern 接口

- const float pi=3.14 常量定义

- inline函数整体

- typedef 类型的的定义

- 模板的完整定义

- 其他一些不会产生内存分配的样式

### 2.小型知识点

###### 模板类：

- 如果在模块A中使用模块B定义的某模板实例，在模块B中该实例未使用的话，则模块A无法使用。

- 原因：实例化是在编译阶段，连接是在link阶段，已经无法实例化

###### 函数重载匹配机制：

- 1.选出候选函数集。满足：
  
  - 该集合中的函数的声明,在该调用点可见。也就是说,和背调函数在同一作用域下
  
  - 该集合中的函数,与被调函数同名

- 2.选出可行函数集。满足：
  
  - 参数个数与被调函数必须相同
  
  - 每个实参的类型要与对应的形参类型相同,或者是能够转换成形参的类型（c++支持隐式类型转换）

- 3.选出最佳匹配。满足：
  
  - 该函数的每个参数的匹配都不必别的差
  
  - 该函数存在一个参数匹配比别的好

- 4.若最佳匹配没有，或者有多个，则会因为二义性报错。ambiguous
  
  - 匹配的规则是：
    
    - 完美匹配（int配int，double配double，bool配bool，子类到基类，非常量到常量，函数名、数组名到指针）
    
    - 整型提升（指 char, 单位int(如bool), short int转为int，或接下来继续转为unsigned int的行为）
    
    - 其余匹配（double配int，int配double，int配long，long配int是一样的）

###### smartpointer（RAII）

```cpp
Template <class T>
class SmartPointer{
    T* t;
  public:
    SmartPointer(T* t=0):t(t){}
    ~SmartPointer(){delete t;}
    T& operator * ()const{return *t;}
    T* operator ->()const{return t;}
};
```

###### 不要多态地对待数组：

基类&子类占用内存大小不一致，数组的[]实际上是`*(array+i)`实现的。偏移量的大小由编译器自己决定（根据数组的类型声明）。

###### 使用非虚接口

```cpp
class CPoint2D{
    double x,y;
  public:
    virtual void display(ostream& out){out<<x<<","<<y<<endl;}
};

ostream &operator <<(ostream& out CPoint2D& a){
    a.display(out);
    return out;
}
class CPoint3D:public CPoint2D{
    double z;
  public:
    void display(ostream& out){
        CPoint2D::display();
        out<<","<<z<<endl;
    }
}
```

###### 前期&动态绑定

- 前期绑定
  
  - 编译时刻
  
  - 依据对象静态类型
  
  - 效率高、灵活性差

- 动态绑定（virtual才行）
  
  - 运行时刻
  
  - 依据实际类型
  
  - 灵活性高，效率低

###### 虚继承、多继承

```plantuml
B -|> A
C -|> A
D -|> B
D -|> C
```

虚继承是为了防止如上的情况下，A的代码在D中有两份的情况

```cpp
class A    // 声明基类A
{
    int a;
public:
    A(int i):a(i){}
    virtual void f();
};
class C: virtual public A    // 声明类 C 是类 A 的公有派生类，A 是 C 的虚基类
{
    int c;
  public:
    C(int i):A(i),c(i){}
    void f();
};
class B: virtual public A    // 声明类 B 是类 A 的公有派生类，A 是 B 的虚基类
{
    int b;
  public:
    B(int i):A(i),b(i){}
    void f()
};

class D: public B, public C    // 类 D 中只有一份 A 的数据
{
    int d;
  public:
    D(int i):A(i),B(i),C(i),d(i){}
                        //注意这里必须要给出A的，若不是virtual则不用，A由BC分别创建
                        //由于virtual，只有一份A，A要由D创建，BC的构造函数不会创建A
    using C::f();//B，C中都有重写的f()，只希望重写C中的f，要这样做
    void f();
};

int main(){
    D d(1);//构造顺序 A C B D //非virtual时 A C A B D
        //析构顺序 D B C A  
}
```

###### inline优&缺

- 优点
  
  - 加快效率
  
  - 提高可读性
  
  - 弥补宏无法进行类型检查的缺陷

- 导致的问题
  
  - 增大目标代码
  
  - 病态换页
  
  - 降低指令快取装置的命中率

- 使用建议：
  
  - 整体放在头文件中
  
  - 使用频率较高、结构不复杂的小段代码使用内联

- 缺点：
  
  - 不能递归
  
  - 由编译系统决定是否真的inline

###### 构造&析构顺序

```cs
class C{
    int values;
public:
    C(int i):values(i){}
    C():values(-1){}
};
class A{};
class B:public A,public C{
    A a_in_b;
    C c_in_b;
    C *p;
public:
    B():c_in_b(2){p=new C[2];p[0]=C(0);p[1]=C(1);}
};
int main(){
    B b;
    delete [] b.p;
}
```

构造顺序：

C(values=-1)

A

A

C(values=2)

C(values=0)

C(values=1)

B

析构顺序：

C（values=1）//意思是delete也是从后往前delete

C（values=0）

B

C（values=2）

A

A

C（values=-1）

###### A类的内存池管理

- 内存池类，总体管理

- 节点链表类，链表形式串联内存结点

- 内存结点类，用于管理内存池申请到的空间，从效率的角度考虑，决定一次申请多个

- Object类，用于包装A，内存节点中形成链表

- A类运算符重写

###### 函数重载&重写

重写又叫覆盖：函数名、函数列表、函数返回值必须都一样。

静态重载又叫"函数隐藏"：是函数名相同，函数列表不同，函数返回值相不相同无所谓

有virtual的

###### 函数传参规则：

从右往左依次传参，缺省参数匹配从左往右，缺省参数设置只能从右往左设置（除了函数模板）
