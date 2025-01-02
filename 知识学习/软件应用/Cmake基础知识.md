[Introduction \| CMake Cookbook](https://chenxiaowei.gitbook.io/cmake-cookbook)
[cmake 简明基础知识汇总！](https://mp.weixin.qq.com/s/XxBX242Jnq8jFmqw3E8oMw)
[看完这篇文章，我终于搞懂了 CMake，真香！(基础篇)](https://mp.weixin.qq.com/s/oBxPqAGTpPYoU2YEpoP_zA)
[看完这篇文章，我终于搞懂了 CMake，真香！(提升篇)](https://mp.weixin.qq.com/s/5L5_GMNXDlOLjvddFwzYwA)

# 一、Cmake 语法知识点

### 1.1 直译模式

使用参数`-P`指定 Cmake 以直译模式解析，直译模式解析 CMakeLists.txt 不会生成 Makefile 文件

### 1.2 变量定义

CMakeLists.txt 中只有字串和字串数组两种变量。定义变量通过  **`set命令`** (不区分大小写)来定义，使用变量时在外面加上 `${}` 符号即可

### 1.3 数学运算

```Cmake
# EXPR 是一款表达式计算工具
# math 是用于数学运算的命令

# 设置变量a、b的值
set(a "1")
set(b "2")

# 加减乘除求余
math(EXPR res "${a} + ${b}")
message("a + b : ${res}")
```

### 1.4 从命令行获取参数

```bash
# 使用-D 指定传递的变量及变量值
$ cmake -Da=2 -Db=3 -P CMakeLists.txt
# 配置工程为debug模式还是release模式
$ cmake -DCMAKE_BUILD_TYPE=Debug
```

### 1.5 流程控制

##### 1.5.1 if

```Cmake
set(ARCH "x86")
if(ARCH MATCHES "x86")
	message("ARCH is x86")
else()
	message("ARCH is arm")
endif()
```

##### 1.5.2 while

```Cmake
set(a "1")
while(${a} LESS "5")
	message("${a}")
	math(EXPR a "${a} + 1")
endwhile()
```

##### 1.5.3 foreach

```Cmake
message("for 1 =========")
foreach(i RANGE 1 5)
	message("${i}")
endforeach()

message("for 2 =========")
foreach(i 1 5 6 7 9 10)
	message("${i}")
endforeach()

message("for 3 =========")
foreach(str Linux C Cpp Python Shell)
	message("${str}")
endforeach()
```

### 1.6 自定义宏与函数

##### 1.6.1 宏

```Cmake
# 定义名为printf的宏
macro(printf str)
	message(${str})
endmacro()

# 使用
printf("hello macro")
# 输出hello macro
```

##### 1.6.2 函数

```Cmake
# 定义名为printf的函数
function(printf str)
	message(${str})
endfunction()

# 使用
printf("hello function")
# 输出hello function
```

- 函数中的变量是局部的，宏中的变量是全局的，宏中的变量在外面也可以被访问到

### 1.7 查看 Cmake 命令说明

```bash
# 查看所有Cmake命令
$ cmake --help-command-list
# 查看set命令说明
$ cmake --help-command set
```

# 二、Cmake 构建

详见[cmake 简明基础知识汇总！](https://mp.weixin.qq.com/s/XxBX242Jnq8jFmqw3E8oMw)
