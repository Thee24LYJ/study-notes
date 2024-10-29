[【C++】stringstream 类 最全超详细解析（什么是 stringstream？ stringstrem 有哪些作用? 如何在算法中应用？）-CSDN 博客](https://blog.csdn.net/weixin_45031801/article/details/136921743)

# 一、stringstream

> stringstream 是 C++中专门用于处理字符串输入输出流的类

### 1.1 stringstream 构造

##### 1.1.1 创建对象后使用<<输入字符串

```cpp
string s("hello world");
stringstream ss;

// 可以直接多次执行ss<<s;进行字符串拼接
ss << s;
cout << ss.str()<<endl; // 使用str()获取内部存储的字符串
```

##### 1.1.2 创建对象时初始化

```cpp
// 该种方式创建的对象执行ss<<s;进行字符串拼接时会先覆盖掉原本字符串后再进行拼接
stringstream ss("hello world");
cout << ss.str()<<endl;

// 若想拼接的时候不被覆盖而是直接添加到原本字符串后面，则采用如下初始化方式
stringstrem ss("hello world",std::ios_base::ate); // 追加的方式进行拼接
```

### 1.2 stringstream 内容修改

```cpp
// 构造对象
stringstream ss("hello world");
// 修改
ss.str("123456");
// 清空
ss.str("");
```

# 二、stringstream 使用

### 2.1 去除字符串中特定字符

```cpp
stringstream ss("hello world");
string s;
// 去除字符串中的空格,此时stringstream是一个单词一个单词的"流入"s中
while(ss >> s)
{
	cout << s <<endl;
}

stringstream ss2("hello,world");
string s2;
// 分隔指定字符,例如","
while(getline(ss2,s2,','))
{
	cout << s2 <<endl;
}
```

### 2.2 类型转换

##### 2.2.1 数字转字符串

```cpp
int num = 123;
stringstream ss;
ss << num;  // 将数字添加给ss对象
cout << ss.str()<<endl;  // 再调用str()即可获得该数字对应的字符串
```

##### 2.2.2 字符串转数字

```cpp
string s="123";
stringstream ss(s);
int num;
ss >> num;  // 将ss对象中的数赋值给num
cout << num <<endl;
```
