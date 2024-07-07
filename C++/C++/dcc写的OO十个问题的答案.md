# OO的十个问题

## 1、当类中未自主定义构造函数时，compiler会提供默认构造函数，why？What does Compiler do for an "Empty" class？移动构造函数？

- 默认构造函数是为了初始化对象，而非对象的变量。因此此时成员变量的值是不确定的，由原来此处内存决定。

- ```cpp
  class A{
      A();//默认构造函数 A a;
      ~A();//默认析构函数 delete a;
      A(const A& a);//拷贝构造函数 A a=b;
      A& operator =(const A& );//拷贝赋值函数 a=b;
      const A* operator &()const;//const A a; const A* b= &a;
      A* operator &();//A a; A* b = &a;
  };
  ```

- 移动构造函数: 若没有自定义拷贝构造/拷贝赋值/析构函数，编译器会自动合成移动构造函数。

- ```cpp
  class A{
      A(A&& a);//移动构造函数，参数为右值
  };
  int x = 5;//x为左值，5为右值
  //左值引用
  int & y = x;//左值可以绑定到非const左值引用上
  const int & z = 5;//右值只能绑定到常量左值引用上
  //为了支持对右值的移动，建立了右值引用
  int && a = 10;a=100;//是对的，而且确实改变了右值
  int && a = num;//是错的，左值不能绑定右值引用
  ```

- 移动构造函数只做指针移动，不做拷贝，接受右值; 拷贝构造做浅拷贝，接受左值。

## 2、When构造函数、析构函数被定义为private？友元、const、static成员的使用，When？

- 单例模式时构造函数定义为private，有限对象时也可以，并用static计数

- 强制自主控制对象存储分配（在栈上建立对象时会报错，因此只能在堆上，显式调用析构）
  
  ```cpp
  static void free(A *p){
      delete p;
  }
  int main(){
      A *p = new A;
      A::free(p);//比起private来更好的办法
  }
  ```

- const对象只能调用被const修饰的类成员函数。

- const成员变量应该放在构造函数的成员初始化表中进行（除非是static const，只能在定义时初始化，不能在初始化表/构造函数内）

- 被const修饰的成员函数会对函数内容进行检查（逻辑const而非按位const），但是对于指针，并不能检查到指针指向的内容是否改变（虽然这种操作肯定是应当避免的），但可以修改static成员变量。

- static成员函数只能存取static成员变量，调用static成员函数。

- 友元的存在目的是"在封装时保证效率"，被friend修饰的。

- 友元：重定向操作符的重载（全局友元）

- ```cpp
  //先声明，后使用
  void func();
  class C;
  class B;
  class A{
      void f();
      friend void MATE(B &b,A& a);//注意这里是引用
                                  //因为值传递在编译时必须知道空间大小
  };
  class B{
      friend void func();//友元函数
      friend class C;//友元类
      friend void A::f();//友元类成员函数
      friend void MATE(B &b,A& a);
  };
  ```

## 3、Why引入成员初始化表？Why初始化表执行次序只与类数据成员的定义次序相关？

- 构造函数内顺序：1开辟空间2赋值；成员初始化表分配内存时即赋值，效率高；也可以在此处声明，以调用类内的其他类对象和基类的非默认构造函数；const成员变量的初始化也在此处。

- 成员初始化表根据类内成员变量内存位置的前后赋值，而定义次序决定了其在内存中存储的顺序。

- 基类的声明次序也决定了多继承中对基类构造/析构的调用顺序，但虚基类优先执行。

## 4、Why引入Copy构造函数、 = 操作符重载？

- 引入copy构造函数的目的是"用其它对象的数据来初始化当前对象"。

- 拷贝构造函数参数是const A& a，引用是为了避免循环调用，const是为了避免修改拷贝模板

- 默认拷贝构造函数会调用成员对象的拷贝构造函数，自定义拷贝构造函数会调用成员对象的默认构造函数

- 1.函数返回类型是类的对象时
  
  2.函数的参数是类的对象，调用该函数时会调用该类的拷贝构造函数
  
  3.使用一个对象去初始化类的另一个对象时

## 5、What is Late Binding? How C++ implement vitural ？

- 运行时才决定对象调用哪个函数，而不是编译时。（因此无virtual的父/子类混用很容易出现切片，要注意）
- 实现方式：对象的内存空间中含有虚函数表指针，里面保存了可能调用的虚函数。因此值得注意的是：构造函数返回前，虚函数表并没有建好，因此没有虚函数特性。
- ```cpp
  class A{
  public:
      A(){f();}
      virtual void f();
      void g();
      void h(){f();g();}
      virtual void m();
      void n();
  };
  class B:public A{
  public:
      void f();
      void g();
      void m(){n();}
      void n();
  };
  int main(){
      B b;//A::A() A::f() B::B()
      A *p = &b;
      p->f();//B::f()
      p->g();//A::g()
      p->h();//A::h() B::f() A::g()
      p->m();//B::m() B::n()
  //造成p->h(),p->m()中非虚函数调用虚函数表现状态不同的原因是:
  //实际上在调用类的成员函数的时候，传递进去了一个指针常量 <该类名>* const this
  //因此在非虚函数内部也能正常按照一般规则调用虚函数
  }
  ```
- override 关键词强制要求基类中有对应的virtual函数
- final 关键词意味着该函数不能再由此派生类链式virtual下去

## 6、When we use virtual ？

- virtual是多态的实现方法

- 拥有纯虚函数的类为虚类，不能实例化，可作为纯虚接口。

- 类的成员函数才可以是虚函数。

- 静态函数（没有*this）、内联函数（编译时已确定）、构造函数（此时虚函数表没有构建好）不能是虚函数。

- 析构函数往往是虚函数（完全释放派生类与基类相比额外申请的内存）

- 抽象工厂模式

- 继承分为两种模式：复用接口；复用实现，复用接口较好。
  
  - 复用接口：纯虚实现（适用于每种类型差别较大）
  
  - 复用实现：目的是少写代码、提高可修改性，可以通过inline，template等实现。

## 7、What does public继承和non-public继承mean？

- private：只能由1.该类中的函数; 2.其友元函数访问。不能被任何其他访问，该类的对象也不能访问。

- protected：可以被1.该类中的函数、2.子类的函数、3.其友元函数访问。但不能被该类的对象访问。

- public：可以被1.该类中的函数、2.子类的函数、3.其友元函数访问，也可以由4.该类的对象访问。

- private 属性不能够被继承。（？是网上看到的，存疑）

- private继承意味着"implemented-in-terms-of"关系。使用private继承， 父类的protected和public属性在子类中变为private；子类也不能转换为对应的基类。

- 使用protected继承，父类的protected和public属性在子类中变为protected；

- public继承意味着"is-a"关系。使用public继承， 父类的protected和public属性不发生改变。

## 8、Why =  ()  []  -> 不能作为全局函数重载?

- 不满足交换律，全局无法保留顺序关系，索性要求不能全局重载。

- 易与原有产生冲突，造成二义性。

- 以下不能重载

- ```c
  // . 成员访问
  //.* 成员指针访问
  // 以上两个是因为要保证访问成员的功能不变
  // :: 域操作符,是因为参数并非变量表达式，而是名称
  // ?: 条件运算符，实际为流程控制，重载后两个都会执行，与原来作用不符
  ```

- 操作符重载

- ```cpp
  //++A or A++
  class A{
      int values;
      A(){values = 0;}
      A & operator ++(){//++A
          values ++;
          return *this;
      }//此处返回 &是为了++（++A）这种类型的出现，和++A.values
      A operator ++(int){//A++
          Counter temp =*this;
          values++;
          return temp;
      }//这里的int是dummy argument，不返回&的原因是temp是局部变量
  }
  ```

- ```cpp
  // = ,赋值操作符重载不能继承，因为派生类中有基类没有的成员
  //    对于这些成员，继承后的赋值操作符会造成"部分赋值"的后果
  // 同时要避免自我赋值，三种方法：Content,Same memory Location,Object identifier
  class A{
      int x,y;
      char *p;//有指针，需要赋值操作符的重载
              //否则赋值之后一对象delete，会悬垂指针
              //需要析构的重载，否则p指向的内容不会自动delete，内存泄漏
     public:
      A(int i,int j,char *s):x(i),y(j){
          p=new char[strlen(s)+1];
          strcpy(p,s);//深拷贝
      }
      virtual ~A(){delete []p};
      A & operator=(A& a){
          x=a.x;y=a.y;
          char *p_orig= p;
          p = new char[strlen(a.p)+1];
          strcpy(p,a.p);
          delete [] p_orig;//这里也有考量：
      //若先delete p，那么内存不够时p就未能重新定义。
      //因此基本顺序为：先赋值，再释放
          return *this;
      }//返回 A& 是为了 a=b=c
       //若返回 const A&，则在(a=b).f()的例子中，f也必须为常成员函数
  }
  ```

- ```cpp
  // []
  class String{
      char *p;
    public:
      String(char *p1){//拷贝构造函数，深拷贝
          p=new char[strlen(p1)+1];strcp(p,p1);
      }
      virtual ~String(){delete[] p;}
      char & operator [](int i){return p[i];}
      const char operator [](int i)const{return p[i];}
      // 一个版本是string s("1234") s[1]='m',另一个是为了const string cs("cons")
      // 无const的必须返回引用，否则无法对其修改
      // 为什么第二个一定要加const呢？既然返回char也不能修改。
      //原因是返回值不能作为区分函数的条件.
      //而被const修饰后，第一个被省略的变量从 string *this变成了const string *this
  }
  ```

- ```cpp
  // -> 是二元操作符，但按照一元操作符格式重载, 要返回A*
  A* operator ->(){return &A}
  ```

- ```cpp
  // << 流操作符 只能重载为类的友元函数
  class A{
      int valueA,valueB;
      A(int a,int b):valueA(a),valueB(b){}
      friend ostream& operator <<(ostream&,A&);
  };
  ostream& operator <<(ostream& out, A& a){
      out<<a.valueA<<","<<a.valueB;
  }
  ```

- ```cpp
  //类型转换操作符的重载：注意：没有返回值，（实际上有，就是转换后的类型）
  //作用：可以隐式转换,当class B中定义了 operator A() ,main()中A a; B b;时：
  // a=b; 可以，还有诸如a<=b
  //void f(A a); f(b); 可以
  //目的是
  class Rational{//分数
    public:
      Rational(int n1,int n2):n(n1),d(n2){}
      operator double() {return (double)n/d;}
    private:
      int n,d;
  };
  int main(){
      Rational r(1,2);
      double x=r;//x=0.5
      x = x+r;//x=1
  }
  ```

- ```cpp
  // new/delete：可以实现
  //
  
  //new:第一个参数：unsigned int，表示大小，默认new时系统自动计算
  void* operator new(size_t size,...){}
  
  //delete：第一个参数：首地址，第二个参数，delete的大小
  void operator delete(void *p,size_t size){}
  
  //new的重载可以有多个，但delete只能有一个
  ```

- ```cpp
  // +/-重载
  friend A operator+(A &a1,A &a2);
  //或
  class A{
      A operator +(A& a){}
  };
  ```

- 

## 9、When成员函数能返回& ？

- 返回值不是局部变量。
- 支持链式调用（a=b=c，cout<<a<<b）
- 希望能避免拷贝构造，提高效率/希望返回原值

## 10、When and How to 重载new、delete？

- 对同一块小内存频繁new&delete

- 程序自身管理内存，提高效率

- how to见上
