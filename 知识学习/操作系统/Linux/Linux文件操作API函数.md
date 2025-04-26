[Linux 系统文件 IO \| 爱编程的大丙](https://subingwen.cn/linux/file-io/)
[文件的属性信息 \| 爱编程的大丙](https://subingwen.cn/linux/stat/)
[文件描述符复制和重定向 \| 爱编程的大丙](https://subingwen.cn/linux/fcntl-dup2/)
[目录遍历 \| 爱编程的大丙](https://subingwen.cn/linux/directory/)

# 文件 IO 操作

### 打开文件

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

/*
open是一个系统函数, 只能在linux系统中使用, windows不支持
fopen 是标准c库函数, 一般都可以跨平台使用, 可以这样理解:
		- 在linux中 fopen底层封装了Linux的系统API open
		- 在window中, fopen底层封装的是 window 的 api
*/
// 打开一个已经存在的磁盘文件
int open(const char *pathname, int flags);
// 打开磁盘文件, 如果文件不存在, 就会自动创建
int open(const char *pathname, int flags, mode_t mode);
```

flags：使用什么方式打开指定的文件，这个参数对应一些宏值，需要根据实际需求指定

- 必须要指定的属性, 以下三个属性不能同时使用, 只能任选其一
  - O_RDONLY: 以只读方式打开文件
  - O_WRONLY: 以只写方式打开文件
  - O_RDWR: 以读写方式打开文件
- 可选属性, 和上边的属性一起使用
  - O_APPEND: 新数据追加到文件尾部, 不会覆盖文件的原来内容
  - O_CREAT: 如果文件不存在, 创建该文件, 如果文件存在什么也不做
  - O_EXCL: 检测文件是否存在, 必须要和 O_CREAT 一起使用, 不能单独使用: O_CREAT | O_EXCL
    - 检测到文件不存在, 创建新文件
    - 检测到文件已经存在, 创建失败, 函数直接返回-1（如果不添加这个属性，不会返回-1）

mode: 在创建新文件的时候才需要指定这个参数的值，用于指定新文件的权限，这是一个八进制的整数，最大值为 0777。创建的新文件对应的最终实际权限计算公式为 `(mode & ~umask)`，umask 掩码可以通过 umask 命令查看

### 关闭文件

```c
#include <unistd.h>
int close(int fd);
```

### 读写文件

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);

#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```

### 文件处理

```c
#include <sys/types.h>
#include <unistd.h>
// 移动文件指针或立即拓展文件大小
off_t lseek(int fd, off_t offset, int whence);
/*
whence: 通过这个参数指定函数实现什么样的功能
SEEK_SET: 从文件头部开始偏移 offset 个字节
SEEK_CUR: 从当前文件指针的位置向后偏移offset个字节
SEEK_END: 从文件尾部向后偏移offset个字节
*/
//文件指针移动到文件头部
lseek(fd, 0, SEEK_SET);
//得到当前文件指针的位置
lseek(fd, 0, SEEK_CUR);
//得到文件总大小
lseek(fd, 0, SEEK_END);
```

使用 lseek 函数进行文件拓展必须要满足一下条件：

- 文件指针必须要偏移到文件尾部之后， 多出来的就需要被填充的部分
- 文件拓展之后，必须要使用 write()函数进行一次写操作（写什么都可以，没有字节数要求）

```c
// 拓展文件或截断文件
#include <unistd.h>
#include <sys/types.h>

// 既可对文件进行拓展也可以截断文件
int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

# 文件属性信息

stat/lstat 函数的功能和 stat 命令的功能是一样的, 只不过是应用场景不同。这两个函数的区别在于处理软链接文件的方式上：

- lstat(): 得到的是软连接文件本身的属性信息
- stat(): 得到的是软链接文件关联的文件的属性信息

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *pathname, struct stat *buf);
int lstat(const char *pathname, struct stat *buf);
```

# 文件描述符复制与重定向

Linux 中除了使用 open()方式给被操作的文件分配一个文件描述符外还提供了一些其他的 API 用于文件描述符的分配：`dup, dup2, fcntl`

```c
#include <unistd.h>

// 复制文件描述符，这样就有多个文件描述符可以指向同一个文件(复制的新文件描述符与旧文件描述符相互独立)
int dup(int oldfd);

// 1. 文件描述符的复制, 和dup是一样的
// 2. 能够重定向文件描述符
// 	- 重定向: 改变文件描述符和文件的关联关系, 和新的文件建立关联关系, 和原来的文件断开关联关系
//		1. 首先通过open()打开文件a.txt , 得到文件描述符fd
//		2. 然后通过open()打开文件b.txt , 得到文件描述符fd1
//		3. 将fd1从定向到fd上:fd1和b.txt这磁盘文件断开关联, 关联到a.txt上, 以后fd和fd1都对用磁盘文件a.txt操作
int dup2(int oldfd, int newfd);

#include <unistd.h>
#include <fcntl.h>	// 主要的头文件

// fcntl()是变参函数且是多功能函数
int fcntl(int fd, int cmd, ... /* arg */ );

```

# 目录遍历

Linux 提供了相关的目录遍历的函数，分别为：`opendir(), readdir(), closedir()`

```c
#include <sys/types.h>
#include <dirent.h>
// 打开目录
DIR *opendir(const char *name);

// 读目录 遍历目录中的文件信息
#include <dirent.h>
struct dirent *readdir(DIR *dirp);

// 关闭目录, 参数是 opendir() 的返回值
int closedir(DIR *dirp);
```
