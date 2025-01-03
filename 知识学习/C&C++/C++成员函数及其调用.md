- [C++成员函数调用 - MalcolmMeng - 博客园](https://www.cnblogs.com/MalcolmMeng/p/10151960.html)
- [C++语言学习（十四）——C++类成员函数调用分析](https://blog.51cto.com/quantfabric/2148695)
- [深入理解类成员函数的调用规则（理解成员函数的内存为什么不会反映在 sizeof 运算符上、类的静态绑定与动态绑定、虚函数表） - 拾月凄辰 - 博客园](https://www.cnblogs.com/FengZeng666/p/9362759.html)

# 一、成员函数

### 1.1 成员函数编译

C++中的函数在编译时会根据命名空间、类、参数签名等信息重新命名形成新的函数名，所使用的算法是 Name Mangling(不同编译器具体实现不同，产生的函数名也不同)；该算法可逆且能确保新函数名唯一(原有函数名<--> 新函数名)

### 1.2 this 指针

this 指针不是对象的一部分，所以使用 sizeof 求出来的类对象大小不会包括 this 指针的内存大小，this 指针类型取决于使用 this 指针的成员函数类型及类类型，this 在成员函数开始执行前构造，执行结束后销毁

this 指针属性：

> 1.名称属性：标识符 this 表示
>
> 2.类型属性：classname const
>
> 3.值属性：表示当前调用该函数对象的首地址
>
> 4.作用域：this 指针是编译器默认传给类中非静态函数的隐含形参，其作用域在非静态成员函数的函数体内
>
> 5.链接属性：在类作用域中，不同类的非静态成员函数中，this 指针变量的链接属性是内部的，但其所指对象是外部的，即 this 变量是不同的实体，但指向对象是同一个
>
> 6.存储类型：this 指针是由编译器生成，当类的非静态成员函数的参数个数一定时，this 指针存储在 ECX 寄存器中；若该函数参数个数未定（可变参数函数），则存放在栈中

### 1.3 成员函数不储存在对象分配空间中

- 成员函数可以被看作类作用域中的全局函数，不位于类对象分配的空间中，而虚函数会使类对象中存储一个指向虚函数表的指针
- 类对象调用成员函数时，通过传入 this 指针和其他相关参数，实现对成员函数的调用，因此类对象中不必存储成员函数的信息
- this 指针也不是类对象的一部分

# 二、成员函数指针

### 2.1 简介

- C++中的成员函数指针具有 contravariance 特性(**当参数类型随着子类型关系的逆转而变化时，参数的类型关系也随之逆转的特性**)，也即基类成员函数指针可以赋值给派生类的成员函数指针
- C++编译器在代码编译阶段就对类对象调用的成员函数进行静态绑定(虚函数动态绑定)，即类成员函数地址在代码编译阶段就已经确定(类成员函数指针可以保存类成员函数地址)
- 成员函数指针不是常规指针，成员函数指针保存的是成员函数在类对象布局中的相对地址

```cpp
class test{
public:
	void print()
	{
		cout<<"test::print"<<endl;
	}
}

// test类的成员函数指针声明
void (test::*pfunc)();
// test类成员函数指针初始化
// pfunc = &类作用域限定符::成员函数名
pfunc = &test::print;

/* 使用typedef 下面这种写法与上面相同 */
typedef void (test::*pfunc)();
// test类成员函数指针定义
pfunc p= &test::print;
// 获取类成员函数地址: &test::print
```

### 2.2 类成员函数地址

- C++中的类成员函数使用 thiscall 函数调用约定
- 静态成员函数、普通成员函数的地址位于代码区，虚成员函数地址是相对地址
- C++规定，可以实现基类成员函数指针到派生类成员函数指针的转换，但不允许派生类成员函数指针到基类成员函数指针的转换(违反多态性和类型安全)

##### 2.2.1 静态成员函数

- 函数体内不存在 this 指针，静态成员函数不属于类的一部分(与常规全局函数一样)，普通成员函数指针语法针对静态成员函数并不成立
- 静态成员函数指针与普通成员函数指针不同之处在于：静态成员函数指针声明时不用添加**类作用域标识符**

```cpp
class test{
public:
	static void print()
	{
		cout<<"test::print"<<endl;
	}
}

// test类的静态成员函数指针声明
void (*pfunc)();
// test类静态成员函数指针初始化
// pfunc = &类作用域限定符::成员函数名
pfunc = &test::print;
```

##### 2.2.2 普通成员函数指针

- 普通成员函数(非静态、非虚函数)指针需要绑定类对象才能调用
- 普通成员函数指针指向代码区中实际函数地址，若强制转换为普通函数指针而非类成员函数指针的行为未定义，该成员函数中的 this 指针对应的成员变量会是垃圾值

##### 2.2.3 虚成员函数指针

- 虚成员函数->实现运行时多态特性
- 虚成员函数指针是相对地址，表示该虚函数在虚函数表中离表头的偏移量，当类对象调用虚函数时，首先找到虚函数表地址，再在此基础上添加距离表头的偏移量即为虚函数实际地址

# 三、成员函数的调用分析

### 3.1 成员函数调用介绍

- 类的成员函数地址位于代码区。调用成员函数时，类对象地址作为参数隐式传递给成员函数，成员函数通过传递进来的对象地址隐式访问对象的成员变量，由于成员函数内部存在 this 指针，因此该 this 指针会赋值为被调用的类对象地址
- 若类对象地址为空(NULL、nullptr)，且当调用的成员函数无需访问成员变量也能成功调用；静态成员函数无需类对象即可进行调用

### 3.2 普通成员函数调用分析

普通成员函数通过地址直接调用，对于`p->print()`的调用(即使 p 为空指针，由于没有对 p 指针解引用，函数也能正常执行，C++编译器生成代码如下：

```cpp
class test{
public:
	void print()
	{
		cout<<"test::print"<<endl;
	}
}

test t;
test *p = &t;
p->print(); // OK
p = nullprt;
p->print(); // OK

// p->print();
test* const this = p;
void test::print(test* const this)
{
	cout<<"test::print"<<endl;
}
```

### 3.3 静态成员函数调用分析

调用方式同全局函数

### 3.4 虚成员函数调用分析

==类对象内部各自维护属于自己的虚函数表==，确保运行时对象调用正确的虚函数

> 1.先通过运行时对象获取虚函数表的地址(虚函数表指针指向的值)
>
> 2.再将虚函数表地址加上虚函数离表头的偏移量得到虚函数地址
