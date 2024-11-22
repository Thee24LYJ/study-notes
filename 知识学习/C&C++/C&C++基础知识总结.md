> 参考：
> [GitHub - huihut/interview: 📚 C/C++ 技术面试基础知识总结，包括语言、程序库、数据结构、算法、系统、网络、链接装载库等知识及面试经验、招聘、内推等信息。This repository is a summary of the basic knowledge of recruiting job seekers and beginners in the direction of C/C++ technology, including language, program library, data structure, algorithm, system, network, link loading library, interview experience, recruitment, recommendation, etc.](https://github.com/huihut/interview)
>
> 声明：学习笔记，强化不熟悉的知识点

# 一、const 的指针与引用

详见[指针与引用](指针与引用.md)

> 被 const 修饰的值不可改变

### 1.1 const 的指针

- 指向常量的指针`const int *p;`
- 自身是常量的指针`int const *p;`

### 1.2 const 的引用

- 指向常量的引用`const int &q = a;`
- 没有自身是常量的引用，引用只是对象别名(不是对象，不能用 const 修饰)

# 二、#define 与 const 常量区别

| 宏定义 `#define`       | const 常量       |
| ---------------------- | ---------------- |
| 相当于字符替换         | 常量声明         |
| 预处理期处理           | 编译期处理       |
| 无类型安全检查         | 有类型安全检查   |
| 不分配内存             | 要分配内存       |
| 存储在**代码段**       | 存储在**数据段** |
| 可通过  `#undef`  取消 | 不可取消         |

# 三、static

详见[static 关键字](static关键字.md)

### 3.1 普通变量

- 修改普通变量储存区域为静态区，未初始化时默认初始化它，且在 main 函数运行前就已分配空间
- 修改生命周期为程序执行的整个周期

### 3.2 普通函数

- 函数作用范围为定义该函数的文件内才能使用，可避免与团队开发项目时函数重名

### 3.3 成员变量

- 所有该类的对象共享一个该成员变量，且可通过类名直接访问该成员变量而无需通过对象进行访问

### 3.4 成员函数

- 与 static 修饰的成员变量相同，但 static 修饰的成员函数不能访问静态成员

# 四、this 指针

- 隐含于每个**非静态成员函数**的特殊指针，指向调用该函数的对象地址
- 隐式声明为`className *const this`，className 为类的名称，即 this 指针本身为常量指针，不能对其进行赋值
- this 指针 为右值
- 显示使用 this 指针的场景：
  - 实现对象的链式引用
  - 避免对同一对象进行赋值操作
  - 实现一些数据结构(如 list)时

# 五、inline 内联函数

- 在调用内联函数的地方进行函数体的替换(函数代码展开)，有点像宏定义的替换，但比宏定义多了类型检查
- 优点：无需执行函数调用的流程，即压栈(保存当前函数地址，去执行对应调用函数)-出栈等操作(回到执行调用函数前的函数地址)，提高程序运行速度(内联函数在编译时会将函数体直接嵌入到每个调用点，‌ 减少函数调用的开销)
- 缺点：增加代码体积，滥用可能导致性能下降，对于较大函数，‌ 内联可能增加编译时间并降低程序性能 ‌
- 对于包含循环、递归、switch 等复杂操作的函数，编译器一般不进行内联
- 类声明中定义的函数，一般会**自动隐式的当做内联函数**(虚函数除外，但虚函数可以是内联函数-->编译器知道所调用类属于哪个类时才可以内联，只是当虚函数具有多态性时不能内联-->即使用对象指针或引用进行调用)
- 内联函数只是对编译器的建议，最终是否进行函数内联决定权在于编译器

# 六、volatile 关键字

- 类型修饰符，表示变量可以被操作系统，硬件等编译器未知的因素更改，使用该关键词就是告诉编译器不应该对这样的对象进行优化
- volatile 关键词修饰的变量每次进行访问时都必须从内存地址中取值，没有使用该关键词修饰的变量可能由于编译器优化而去 CPU 寄存器取值
- 指针和 const 可以使用 volatile 修饰

# 七、assert()断言

- 宏定义，若它的条件为假，终止程序执行

# 八、sizeof()函数

```cpp
int array[10];
int *p = nullptr;
sizeof(array);// 返回整个数组所占空间大小 10*sizeof(int)
sizeof(p);// 返回指针本身所占空间大小
```

# 九、位域

在一个位域中含有一定数量的二进制位，如`unsigned int x : 2;`表示 x 占用两位

# 十、extern "C"

详见[extern 关键字](extern关键字.md)

使用 C++编译器进行编译时，将采用 extern "C"声明的代码作为 C 语言代码进行编译，避免 C++编译器由于符号修饰导致不能与 C 语言头文件库中的符号进行链接的问题

# 十一、union 关键字

- 一个 union 可以有多个数据成员，但任意时刻只能有一个数据成员有值，其他数据成员未定义
- 与 struct 类似，默认访问控制符为 public，可以有构造函数和析构函数，但不能继承

# 十二、explicit 关键字

- 修饰构造函数，避免**隐式转换和赋值初始化**，但可以直接初始化、直接列表初始化、强制转换为对应类类型
- 修饰转换函数，避免隐式转换

```cpp

class c;

void do(c cc);

c c1(1); // 直接初始化 OK
c c2 = 1;// 赋值初始化 ERR
c c3{1}; // 直接列表初始化 OK
c c4 = {1}; // 赋值列表初始化 ERR
c c5 = (c)1;// static_cast进行初始化 OK
do(1); // 隐式转换 ERR
if(c1) // 被 explicit 修饰转换函数 c::operator bool() 的对象可以从 c 到 bool 的按语境转换 OK
bool c6 = c1; // 被 explicit 修饰转换函数 c::operator bool() 的对象不可以隐式转换 ERR
```

# 十三、friend 声明的友元类和友元函数

- 破坏封装性(可以访问私有成员)
- 友元关系不能传递(_你的友员不是我的友员_)、单向性(_我是你的友员但你不是我的友员_)
- 对于友元函数，友元函数不属于任何类，可以独立存在；一个类中可以声明多个函数为自己的友元函数

# 十四、using

### 14.1 using 声明

> 引入命名空间的一个成员

```cpp
// 声明1:
using namespace_name::name;
// 后续使用则直接使用name即可，示例如下：
using std::cout;
using std::endl;
cout<<"hello world"<<endl;

// 声明2(直接使用namespace_name::name)
// 示例如下：
std::cout<<"hello world"<<std::endl;

// 构造函数的using声明
// 下述using声明，对于基类的每个构造函数编译器都会生成一个与之对应(形参列表完全相同)的派生类构造函数
class Derived : Base
{
public:
	using Base::Base;
}
```

### 14.2 using 指示

> 使得某个特定命名空间中所有名字都可见，和 python 中的 from package_name import \*一样的作用
> 尽量少使用 using 指示污染命名空间，更不要直接在头文件中使用

```cpp
using namespace_name name;
// 例如:
using namespace std;
```

# 十五、:: 范围解析运算符

- 全局作用域符（`::name`）：用于类型名称（类、类成员、成员函数、变量等）前，表示作用域为**全局命名空间**
- 类作用域符（`class::name`）：用于表示指定类型的作用域范围是具体某个类的
- 命名空间作用域符（`namespace::name`）:用于表示指定类型的作用域范围是具体某个命名空间的

# 十六、enum 枚举类型

- 限定作用域(枚举类)

```cpp
enum class enum_name {red, yellow, green};
```

- 不限定作用域

```cpp
enum enum_name {red, yellow, green};
```

# 十七、decltype

> 关键字，检查实体的声明类型或表达式的类型及值分类

# 十八、引用

### 18.1 左值引用

> 对于一般变量(对象)的引用

### 18.2 右值引用

> 对于右值(临时对象，即将销毁的对象)的引用，即对象的值
> 使用右值引用目的：
> 避免两个对象不必要的拷贝，节省存储资源，提高效率
> 定义泛型函数更加简洁明确

> 引用折叠：
> `X& &`、`X& &&`、`X&& &`  可折叠成  `X&` > `X&& &&`  可折叠成  `X&&`

# 十九、成员初始化列表

> 高效性：减少一次调用默认构造函数的过程(类类型成员变量使用初始化列表效率提升较大，如果是内置类型成员变量则使用初始化列表和函数体中初始化效率差别不大)

必须使用初始化列表的地方：

> 常量成员：只能初始化而不能被赋值
> 引用类型：引用只能定义时初始化
> 没有默认构造函数的类

# 二十、面向对象三大特征 —— 封装、继承、多态

> [C++ 多态机制](C++%20多态机制.md)
> 多态是以封装和继承为基础的，分为静态多态(编译期/早绑定)和动态多态(运行期/晚绑定)
> ![](https://gitee.com/huihut/interview/raw/master/images/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E5%9F%BA%E6%9C%AC%E7%89%B9%E5%BE%81.png)

- 静态函数不能是虚函数
- 构造函数不能是虚函数(必须要调用构造函数后，对象的内存空间中才会存在虚表指针)
- 类的成员模板(本身是模板的成员函数)不能是虚函数

# 二十一、内存分配

详见[C&C++ 内存分配](C&C++%20内存分配.md)

### 21.1 定位 new

> 定位 new（placement new）允许我们向 new 传递额外的地址参数，从而在预先指定的内存区域创建对象
> place_address 为指针，`initializers`  提供一个（可能为空的）以逗号分隔的初始值列表

```cpp
new (place_address) type
new (place_address) type (initializers)
new (place_address) type [size]
new (place_address) type [size] { braced initializer list }
```

### 21.2 只能在堆或栈上分配对象空间的类

- 只能在堆上分配

  > 设置析构函数为 private

- 只能在栈上分配
  > 重载 new 和 delete 运算符为 private
