[IO 多路转接（复用）之 select \| 爱编程的大丙](https://subingwen.cn/linux/select/)
[IO 多路转接（复用）之 poll \| 爱编程的大丙](https://subingwen.cn/linux/poll/)
[IO 多路转接（复用）之 epoll \| 爱编程的大丙](https://subingwen.cn/linux/epoll/)

IO 多路转接也称为 IO 多路复用，它是一种网络通信的手段（机制），通过这种方式可以同时监测多个文件描述符并且这个过程是阻塞的，一旦检测到有文件描述符就绪（ 可以读数据或者可以写数据）程序的阻塞就会被解除，之后就可以基于这些（一个或多个）就绪的文件描述符进行通信了。通过这种方式在单线程/进程的场景下也可以在服务器端实现并发。常见的 IO 多路转接方式有：`select`、`poll`、`epoll`

下面先对多线程/多进程并发和 IO 多路转接的并发处理流程进行对比（服务器端）：

- 多线程/多进程并发

  - 主线程/父进程：调用 accept()监测客户端连接请求
    - 如果没有新的客户端的连接请求，当前线程/进程会阻塞
    - 如果有新的客户端连接请求解除阻塞，建立连接
  - 子线程/子进程：和建立连接的客户端通信
    - 调用 read() / recv() 接收客户端发送的通信数据，如果没有通信数据，当前线程/进程会阻塞，数据到达之后阻塞自动解除
    - 调用 write() / send() 给客户端发送数据，如果写缓冲区已满，当前线程/进程会阻塞，否则将待发送数据写入写缓冲区中

- IO 多路复用
  - 使用 IO 多路复用函数委托内核检测服务器端所有的文件描述符（通信和监听两类），这个检测过程会导致进程/线程的阻塞，如果检测到已就绪的文件描述符阻塞解除，并将这些已就绪的文件描述符传出
  - 根据类型对传出的所有已就绪文件描述符进行判断，并做出不同处理
    - 监听的文件描述符：和客户端建立连接：此时调用 accept()是不会导致程序阻塞的，因为监听的文件描述符是已就绪的（有新请求）
    - 通信的文件描述符：调用通信函数和已建立连接的客户端通信
      - 调用 read() / recv() 不会阻塞程序，因为通信的文件描述符是就绪的，读缓冲区内已有数据
      - 调用 write() / send() 不会阻塞程序，因为通信的文件描述符是就绪的，写缓冲区不满，可以往里面写数据
  - 对这些文件描述符继续进行下一轮的检测（循环往复。。。）

**与多进程和多线程技术相比，I/O 多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销**

# select

使用 select 能够检测的最大文件描述符个数有上限，默认是 1024

```c
#include <sys/select.h>
struct timeval {
    time_t      tv_sec;         /* seconds */
    suseconds_t tv_usec;        /* microseconds */
};

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval * timeout);

// 将文件描述符fd从set集合中删除 == 将fd对应的标志位设置为0
void FD_CLR(int fd, fd_set *set);
// 判断文件描述符fd是否在set集合中 == 读一下fd对应的标志位到底是0还是1
int  FD_ISSET(int fd, fd_set *set);
// 将文件描述符fd添加到set集合中 == 将fd对应的标志位设置为1
void FD_SET(int fd, fd_set *set);
// 将set集合中, 所有文件文件描述符对应的标志位设置为0, 集合中没有添加任何文件描述符
void FD_ZERO(fd_set *set);
```

使用 select 并发处理流程：
![](https://subingwen.cn/linux/select/image-20210324235635304.png)

# poll

poll 的机制与 select 类似，与 select 在本质上没有多大差别，使用方法也类似，下面的是对于二者的对比：

- 内核对应文件描述符的检测也是以线性的方式进行轮询，根据描述符的状态进行处理
- poll 和 select 检测的文件描述符集合会在检测过程中频繁的进行用户区和内核区的拷贝，它的开销随着文件描述符数量的增加而线性增大，从而效率也会越来越低
- select 检测的文件描述符个数上限是 1024，poll 没有最大文件描述符数量的限制
- select 可以跨平台使用，poll 只能在 Linux 平台使用

```c
#include <poll.h>
// 每个委托poll检测的fd都对应这样一个结构体
struct pollfd {
    int   fd;         /* 委托内核检测的文件描述符 */
    short events;     /* 委托内核检测文件描述符的什么事件 */
    short revents;    /* 文件描述符实际发生的事件 -> 传出 */
};

struct pollfd myfd[100];
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

```

# epoll

epoll 全称 eventpoll，是 select 和 poll 的升级版，主要特点是：

- 对于待检测集合 select 和 poll 是基于线性方式处理的，epoll 是基于红黑树来管理待检测集合的
- select 和 poll 每次都会线性扫描整个待检测集合，集合越大速度越慢，epoll 使用的是回调机制，效率高，处理效率也不会随着检测集合的变大而下降
- select 和 poll 工作过程中存在内核/用户空间数据的频繁拷贝问题，在 epoll 中内核和用户区使用的是共享内存（基于 mmap 内存映射区实现），省去了不必要的内存拷贝。
- 程序猿需要对 select 和 poll 返回的集合进行判断才能知道哪些文件描述符是就绪的，通过 epoll 可以直接得到已就绪的文件描述符集合，无需再次检测
- 使用 epoll 没有最大文件描述符的限制，仅受系统中进程能打开的最大文件数目限制

当多路复用的文件数量庞大、IO 流量频繁的时候，一般不太适合使用 select()和 poll()，这种情况下 select()和 poll()表现较差，推荐使用 epoll()

```c
#include <sys/epoll.h>
// 创建epoll实例，通过一棵红黑树管理待检测集合
int epoll_create(int size);
// 管理红黑树上的文件描述符(添加、修改、删除)
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 检测epoll树中是否有就绪的文件描述符
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

```

select、poll 和 epoll 流程：
![](https://subingwen.cn/linux/epoll/image-20210403181746358.png)
