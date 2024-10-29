- [一文讲明白 C++中的结构体 Struct 和类 Class 的区别以及使用场景 -CSDN 博客](https://blog.csdn.net/qq_39350172/article/details/132523467)
- [C++ 中 class 和 struct 区别 | 编程指北](https://csguide.cn/cpp/basics/class_and_struct.html#c-%E4%B8%AD-class-%E5%92%8C-struct-%E5%8C%BA%E5%88%AB)

# 一、结构体 struct

- C++中的**结构体**是用来组合不同类型数据成员的用户自定义数据结构
- 结构体中数据成员默认访问权限是 public 的，但可以修改访问权限(C++11 引入)
- struct 中没有默认无参构造函数，只能声明有参构造函数
- 结构体中可以声明和定义成员函数，但仅限于简单操作而非丰富行为

注：C 语言中结构体不能定义函数

```cpp
struct struct_test{
	struct_test(int abc){}
	~struct_test(){}

	int public_int;
private:
	int private_int;

public:
	void func_test()
	{
		int test = 0;
	}
}
```

# 二、类 class

- C++中的**类**用于创建用户自定义数据类型(数据成员和成员函数)，实现数据的封装
- 类是面向对象编程的核心，允许将数据和操作封装在一起，创建更加模块化和可维护的代码
- 类具有构造函数初始化对象，析构函数释放对象占用的空间
- 类中成员默认访问权限是 private 的，但也可以修改其访问权限
- 类支持继承和多态行为，实现代码扩展和重用，可以用于定义模版参数

```cpp
class class_test{
	int private_int;
public:
	int public_int;

	class_test(){}
	~class_test(){}

private:
	int age;
	double radius;

	void func_test()
	{
		int test = 1;
	}
}
```

# 三、struct 和 class 的区别

- struct 中成员默认访问权限为 public，class 中成员默认访问权限为 private
- struct 继承默认是 public 继承，class 继承默认是 private 继承
- struct 不能定义模板参数，class 可以定义模板参数
