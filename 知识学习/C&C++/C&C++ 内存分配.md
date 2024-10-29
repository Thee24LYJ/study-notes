- [c++ new 与 malloc 有什么区别](https://www.cnblogs.com/ywliao/articles/8116622.html)
- [C++经典内存管理之 new,malloc 的区别及底层原理](https://blog.csdn.net/weixin_53344209/article/details/127533351)
- [malloc 底层实现及原理](https://www.cnblogs.com/ssezhangpeng/p/10808969.html)
- [linux 环境内存分配原理--虚拟内存 mallocinfo](https://www.cnblogs.com/dongzhiquan/p/5621906.html)
- [深入理解 malloc | 寒枫的博客](https://hanfeng.ink/post/understand_glibc_malloc/)
- [4.2 malloc 是如何分配内存的？ | 小林 coding](https://xiaolincoding.com/os/3_memory/malloc.html#linux-%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%86%85%E5%AD%98%E5%88%86%E5%B8%83%E9%95%BF%E4%BB%80%E4%B9%88%E6%A0%B7)
- [C/C++｜ malloc 底层原理与内存分配详解-CSDN 博客](https://blog.csdn.net/caiziming_001/article/details/139213036)
- [c 语言 malloc、calloc 和 realloc 动态分配内存函数的区别 - 琨为玉也 - 博客园](https://www.cnblogs.com/zkbklink/p/18277009)
- [C++中 new 一个对象的时候加括号和不加括号的区别-CSDN 博客](https://blog.csdn.net/ywending/article/details/51096547)
- [c++ new 带括号和不带括号 - youxin - 博客园](https://www.cnblogs.com/youxin/p/3735064.html)

# 一、malloc 与 new

### 1.1 malloc()函数

##### 1.1.1 函数声明

```c
void *malloc(int size);
```

> C 语言中，库函数 malloc()向系统申请从堆上分配 size 字节的内存空间，返回类型为 void \*

- malloc()函数返回类型为 void \*，需要在使用的时候将其强制转换
- malloc()函数申请内存成功返回指向该内存地址的 void \*指针；否则返回 NULL
- malloc()函数只分配内存而不进行初始化
- malloc()函数需要搭配 free()函数释放该指针指向的内存空间，但不销毁指针本身

> malloc()/calloc()/realloc()
> malloc()只分配空间而不初始化
> calloc()在 malloc()分配内存空间基础上进行初始化操作(初始化为 0)
> realloc()调整已分配内存大小，调整前后内存地址不一定一样

调用 malloc()函数时，将从堆的空闲链表中寻找足够用于分配的空闲区域，若寻找到则将需要分配的内存空间地址返回，剩下的空间继续加到空闲链表中；若因为内存碎片较多找不到能用于分配的空闲区域则：首先对内存空间进行整理，再对整理后的空闲空间进行寻找，若仍没有找到则返回 NULL 指针

##### 1.1.3 malloc 底层实现内存分配方式

> 下面两种实现内存分配的方式所分配的内存都是虚拟内存，并没有分配物理内存；而是在第一次访问该分配的虚拟地址空间时，发生缺页中断，才切换到内核态由操作系统分配物理内存，然后才建立起虚拟内存与物理内存间的映射关系
> malloc() 在分配内存的时候，**会预分配更大的空间作为内存池**

- 申请内存小于 128KB 时，采用 brk()系统调用从堆分配内存，将数据段（.data）的最高地址指针 \_edata 往高地址移动，获得新的内存空间；当使用 free()释放内存时，并不是直接将该内存归还给操作系统而是缓存到 malloc 的内存池中，便于下次使用(内存不会被真正释放掉)
- 申请内存大于 128KB 时，通过 mmap()文件调用在文件映射区分配内存；使用 free()释放内存时，会直接将该内存归还给操作系统(内存会被真正释放掉)

### 1.2 new 操作符

##### 1.2.1 操作符声明

```cpp
type_name * p = new type_name(初始化参数列表)
```

> C++语言中，new 是操作符而非库函数，通过 new 操作符从自由存储区(不一定是堆)上为对象内存分配内存空间，返回类型为对象类型的指针
> 对于有构造函数的类，不论有没有括号，都用构造函数进行初始化；如果没有构造函数，则不加括号的 new 只分配内存空间，不进行内存的初始化，而加了括号的 new 会在分配内存的同时初始化为 0，因此推荐使用括号

- new 操作符申请内存无需指定内存大小，编译器会根据类型信息自动计算
- new 申请内存空间并对其初始化成功，返回指向该内存空间对应对象类型的指针；否则抛出 bad_alloc 异常
- new 操作符能被重载

##### 1.2.2 new 底层原理

对于自定义类型，new 操作符首先调用 operator new 函数来申请足够内存空间(底层使用 malloc()实现)，再调用自定义类型的构造函数来初始化内存，最后返回指向该内存空间自定义类型的指针

### 1.3 malloc 与 new 的区别

|            特征            |               new 操作符               |       malloc 库函数        |
| :------------------------: | :------------------------------------: | :------------------------: |
|        分配内存位置        |               自由存储区               |             堆             |
|    分配内存成功时返回值    |           相应对象类型的指针           |          void \*           |
|    分配内存失败时返回值    |              默认抛出异常              |            NULL            |
|       分配内存字节值       |         编译器根据类型计算得到         |    显式指定需分配字节数    |
|        对数组的处理        | `new[]`(例如`int *ptr = new int[10];`) | 显式指定该数组需分配字节数 |
|       扩充已分配内存       |              无法直接处理              |         realloc()          |
|        是否嵌套调用        |           底层调用 malloc()            |        不能调用 new        |
|  分配内存时内存不足的处理  |        可指定自定义对应处理函数        |   不可指定自定义处理函数   |
|        是否允许重载        |                允许重载                |         不允许重载         |
| 是否调用构造函数进行初始化 |                  调用                  |           不调用           |

# 二、free 与 delete

### 2.1 free()函数

##### 2.1.1 函数声明

```c
void free(void *ptr);
```

> 释放使用 malloc、calloc 或 realloc 进行动态分配的内存空间，若传递的指针 ptr 是空指针，则 free 不做任何操作
> malloc 返回给用户态的内存起始地址比进程的实际堆空间起始地址多了 16 字节，这个多出来的 16 字节就是保存了该内存块的描述信息，比如有该内存块的大小等信息

### 2.2 delete 操作符

```cpp
delete ptr; // 删除分配给非数组对象的内存空间
delete [] ptr; // 删除分配给数组对象的内存空间
```

> delete 首先调用对象类型的析构函数(对于类类型的指针)，再调用 operator delete 释放内存(底层通常使用 free 实现)，若传递的指针 ptr 是空指针，delete 也能正常运行
