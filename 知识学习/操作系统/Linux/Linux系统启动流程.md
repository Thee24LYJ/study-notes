[Linux 启动流程 梳理| 思维导图 | 流程图 | 值得收藏](https://mp.weixin.qq.com/s/gq6qdfcu0G2Vu3HTc4AX5g)
[计算机是如何启动的？ - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2013/02/booting.html)
[Linux 的启动流程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html)
[BootLoader、Linux Kernel（linux 内核）、RootFile（根文件系统）-CSDN 博客](https://blog.csdn.net/zhangcanyan/article/details/81409053)

# 一、Linux 系统启动过程

- 读取 BIOS 加载硬件信息，对硬件系统进行自我测试(开机自检 POST)，取得 BIOS 中设置的第一个可启动设备
- 读取并执行第一个可启动设备内 MBR 的 boot loader(grub、spfdisk 等程序)
- 根据 boot loader 设置加载 kernel，在内核态运行并对内存进行划分，kernel 检测硬件并加载驱动程序
- 硬件驱动加载成功后，kernel 主动调用执行 init 进程，该进程会取得 run-level 信息(操作系统启动完成后，CPU就会切换到Ring 3级别上，同时操作系统进入用户态)
- init 执行各种程序准备软件执行操作环境、启动各种服务及开机自启程序
- init 执行终端机模拟程序 mingetty 启动 login 进程，最后等待用户登录

# 二、嵌入式 Linux 系统 boot loader 执行阶段

> Linux 系统 bootloader 由 bootstrap 和 uboot 组成，实际上 Linux 系统内核是由 uboot 装载到内存中去的

### 2.1 stage 1

- 硬件设备初始化
- 为加载 boot loader 的 stage 2 程序准备 RAM
- 复制内核映像和文件系统映像到 RAM
- 设置堆栈指针 sp
- 跳转到 stage 2 的执行入口处

### 2.2 stage 2

- 初始化当前阶段需使用的硬件设备
- 检测系统内存映像
- 加载内核映像和文件系统映像
- 设备内核启动参数
- 调用执行内核映像
