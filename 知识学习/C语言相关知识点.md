# 一、printf()函数

printf()作为为 C 语言中的库函数，需要包含头文件 stdio.h，能够发送格式化输出到标准输出 stdout，printf() 函数的声明如下：

```c
int printf(const char *format, ...);
```

printf() 函数的格式控制字符串组成如下：

```c
%[flags][width][.precision][length]specifier
```

%[标志][最小宽度][.精度][长度]说明符，其中[]代表可选的。

参考： 1.[C 库函数-printf()](https://mp.weixin.qq.com/s/Sc9uW9aWrAfc2LJ1WSdjuQ) 2.[printf](https://cplusplus.com/reference/cstdio/printf/)

# 二、类型转换

两个不同类型的数进行运算，会进行隐式转换，“小”的会向“大”的转换。
同理，无符号整数和有符号进行整数运算时，有符号数会转换为无符号数。

参考：
[C 语言入坑指南-整型的隐式转换与溢出-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1497319)

# 三、输入一行整数数据，使用空格分开，回车结束

```cpp
// 1 2 3 4 5 6
for( i = 0; i < n; i++)
{
	scanf("%d",&cur);
	char c = getchar();
	if (c == '\n') {
		break;
	}
}
```

[\[C 语言\]输入一行整数，用空格分开，回车结束。-CSDN 博客](https://blog.csdn.net/weixin_30546933/article/details/96404738)

# 相关文章资料

[非常简洁的 C 语言知识汇总！](https://mp.weixin.qq.com/s/ZRnmkYE5ubg36JA6P5kvXA)
[C++ FAQ LITE](https://www.sunistudio.com/cppfaq/)
[C++ Cheat Sheets & Infographics | hacking C++](https://hackingcpp.com/cpp/cheat_sheets.html)
