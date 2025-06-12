参考：

[CUDA 编程基础入门系列（持续更新）](https://www.bilibili.com/video/BV1sM4y1x7of/?vd_source=306f4736c1d442055b9fb0141a07c927)
[CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)
[初识多线程并行计算 \| Notebook](https://cuda.keter.top/intro_parallel/#%E7%BA%BF%E7%A8%8B%E7%B4%A2%E5%BC%95)

### CUDA 环境搭建

[CUDA 相关](CUDA相关.md)

![](https://s2.loli.net/2025/01/15/9JPxCIp4HdTkhAb.png)

查询 GPU 详细信息：`nvidia-smi -q`

查询某块 GPU 详细信息：`nvidia-smi -q -i 0`

查询 GPU 特定信息：`nvidia-smi -q -i 0 -d MEMORY`

核函数中只能使用 printf 打印信息，host 端(CPU)可以使用 cout 打印

NVCC 编译：`nvcc hello.cu -o hello`

### CUDA 核函数（Kernel function）

核函数在 GPU 上并行执行，使用`__global__`修饰，返回值必须为 void

```c++
__global__ void kernel_function(argument)
{
	printf("hello world from GPU!\n");
}
// 相同作用
void __global__ kernel_function(argument)
{
	printf("hello world from GPU!\n");
}

// CUDA程序编写流程
int main(void)
{
    // 主机代码 CPU执行
    // 核函数代码 GPU执行
    // 主机代码 CPU执行
    return 0;
}
```

注意事项：

- 核函数**只能**访问 GPU 的内存即显存
- 核函数不能使用变长参数
- 核函数不能使用静态变量
- 核函数不能使用函数指针
- 核函数具有异步性(由 GPU 并行执行)
- 核函数不支持 C++的 iostream 库

### CUDA 线程模型

![](https://s2.loli.net/2025/01/15/VTnZNwzj5DJBM49.png)

grid_size 表示一个 grid 中有多少个线程块，block_size 表示一个 block 中有多少个线程即：

$$
grid\_size = gridDim.x \\
block\_size = blockDim.x
$$

![](https://s2.loli.net/2025/01/15/JfgqHy3L6dKxr1c.png)

一维线程全局唯一标识 Idx：

$$
Idx = threadIdx.x + blockIdx.x*blockDim.x
$$

- CUDA 可以组织三维的网格和线程块
- blockldx 和 threadldx 是类型为 uint3 的变量，该类型是一个结构体，具有 x,y,z 三个成员(3 个
  成员都为无符号类型的成员构成)

$$
\left\{\begin{array} {l}
\text { blockIdx.x } \\
\text { blockIdx.y } \\
\text { blockIdx.z }
\end{array} \quad
\left\{\begin{array}{l}
\text { threadIdx.x } \\
\text { threadIdx.y } \\
\text { threadIdx.z }
\end{array}\right.\right.
$$

- gridDim 和 blockDim 是类型为 dim3 的变量，该类型是一个结构体，具有 x,y,z 三个成员

$$
\left\{\begin{array} {l}
\text { gridDim.x } \\
\text { gridDim.y } \\
\text { gridDim.z }
\end{array} \quad
\left\{\begin{array}{l}
\text { blockDim.x } \\
\text { blockDim.y } \\
\text { blockDim.z }
\end{array}\right.\right.
$$

- 非一维网格时，gridDim 和 blockDim 没有指定的维度默认为 1：

$$
\left\{\begin{array} {l}
\text { gridDim.x = grid\_size } \\
\text { gridDim.y = 1 } \\
\text { gridDim.z = 1}
\end{array} \quad
\left\{\begin{array}{l}
\text { blockDim.x = block\_size } \\
\text { blockDim.y = 1} \\
\text { blockDim.z = 1}
\end{array}\right.\right.
$$

- 以上变量无需定义，属于内建变量，只在核函数内有效

```
// 定义多维网格和线程块(类似于C++构造函数语法)
dim3 grid_size(Gx,Gy,Gz);
dim3 block_size(Bx,By,Bz);

// 定义2*2*1的网格和5*3*1的线程块
dim3 grid_size(2,2); // 等效于dim3 grid_size(2,2,1);
dim3 block_size(5,3); // 等效于dim3 block_size(5,3,1);
```

二维线程中，每个线程块中线程唯一标识 tid，每个网格中线程块的唯一索引 bid，网格中每个线程的唯一索引 id

$$
tid = threadIdx.x + threadIdx.y*blockDim.x\\
bid = blockIdx.x + blockIdx.y*gridDim.x\\
id = tid + bid * (blockDim.x*blockDim.y)
$$

三维线程中，每个线程块中线程唯一标识 tid，每个网格中线程块的唯一索引 bid，网格中每个线程的唯一索引 id

$$
tid = (threadIdx.z*(blockDim.x*blockDim.y)) + threadIdx.y*blockDim.x + threadIdx.x\\
bid = blockIdx.z*gridDim.x*gridDim.y + blockIdx.y*gridDim.x + blockIdx.x\\
id = bid * (blockDim.x * blockDim.y * blockDim.z) + tid
$$

- 网格和线程块大小限制(线程块总的大小最大为 1024)

$$
max(gridDim.x) = 2^{31}-1\\
max(gridDim.y) = 2^{16}-1\\
max(gridDim.z) = 2^{16}-1\\
max(blockDim.x) = 1024\\
max(blockDim.y) = 1024\\
max(blockDim.z) = 64\\
\\
max(blockDim.x\times blockDim.y\times blockDim.z) = 1024
$$

### NVCC 编译流程

![](https://s2.loli.net/2025/01/15/Wqs1t4RjknPFzJi.png)

[NVIDIA CUDA Compiler Driver NVCC](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html)

![](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/_images/cuda-compilation-from-cu-to-executable.png)

[Parallel Thread Execution ISA](https://docs.nvidia.com/cuda/parallel-thread-execution/index.html)

![](https://s2.loli.net/2025/01/15/vOubtLcxB8E7kK6.png)

# CUDA 程序基本框架

```c++
#include <头文件>

__global__ void 函数名(参数...)
{
    // 核函数内容
}

int main()
{
    // 设置GPU设备
    // 分配主机和设备内存
    // 初始化主机中的数据
    // 将数据从主机复制到设备
    // 调用核函数在GPU设备中进行计算
    // 将计算得到数据从设备传给主机
    // 释放主机与设备内存
}
```

CUDA 通过内存分配、数据传递、内存初始化和内存释放进行内存管理

运行时 API

```
__host__ __device__ cudaError_t cudaGetDeviceCount(int *count);	// 获取GPU设备数量
__host__ cudaError_t cudaSetDevice(int device);	// 设置GPU执行时使用的设备

__host__ __device__ cudaError_t cudaMalloc(void **devPtr, size_t size);	// cuda内存分配函数
__host__ cudaError_t cudaMemcpy(void *dst, const void *src, size_t count, cudaMemcpyKind kind);	// cuda数据拷贝
__host__ cudaError_t cudaMemset(void *devPtr, int value, size_t count);	// cuda内存初始化
__host__ __device__ cudaError_t cudaFree(void *devPtr);	// cuda内存释放
__host__ __device__ const char *cudaGetErrorName(cudaError_t error);	// 获取错误代码对应名称
__host__ __device__ const char *cudaGetErrorString(cudaError_t error);	// 获取错误代码描述信息
__host__ __device__ cudaError_t cudaGetLastError();	// 返回最后一个错误

__host__ cudaError_t cudaGetDeviceProperties(cudaDeviceProp *prop, inde device);	// 查询GPU信息的运行时API
```

| 标准 C 语言内存管理函数 | CUDA 内存管理函数 |
| :---------------------: | :---------------: |
|        malloc()         |   cudaMalloc()    |
|        memcpy()         |   cudaMemcpy()    |
|        memset()         |   cudaMemset()    |
|         free()          |    cudaFree()     |

![](https://s2.loli.net/2025/01/15/M6oSTdKnGgrs9Y3.png)

```c++
cudaError_t ErrorCheck(cudaError_t error_code, const char *fileName, int lineNumber)
{
    if (error_code != cudaSuccess)
    {
        printf("%s:%d: CUDA ERROR Code=%d, name=%s, description=%s", fileName, lineNumber, error_code, cudaGetErrorName(error_code), cudaGetErrorString(error_code));
        return error_code;
    }
    return error_code;
}
// 捕捉核函数可能产生的错误
ErrorCheck(cudaGetLastError(), __FILE__, __LINE__);
ErrorCheck(cudaDeviceSynchronize(), __FILE__, __LINE__);
```

# CUDA 计时(事件计时)

CUDA 事件计时可以为 host 和 device 代码计时

```c++
cudaEvent_t start, stop;
ErrorCheck(cudaEventCreate(&start), __FILE__, __LINE__);
ErrorCheck(cudaEventCreate(&stop), __FILE__, __LINE__);
ErrorCheck(cudaEventRecord(start), __FILE__, __LINE__);
cudaEventQuery(start);

// 需计时代码

ErrorCheck(cudaEventRecord(stop), __FILE__, __LINE__);
ErrorCheck(cudaEventSynchronize(stop), __FILE__, __LINE__);
float elapsed_time = 0.0;
ErrorCheck(cudaEventElapsedTime(&elapsed_time, start, stop), __FILE__, __LINE__);
printf("Time elapsed: %f ms\n", elapsed_time);

ErrorCheck(cudaEventDestroy(start), __FILE__, __LINE__);
ErrorCheck(cudaEventDestroy(stop), __FILE__, __LINE__);
```

# GPU 硬件资源

- GPU 并行性依靠流多处理器 SM(streaming multiprocessor)完成
- GPU 中的每个 SM 都可以支持数百个线程**并发**执行
- 以线程块 block 为单位，向 SM 分配线程块，多个线程块可以同时被分配到一个可用的 SM 上(一个线程块只能分配到一个 SM 上，线程块中的所有线程分配到同一个 SM 上执行)
- 线程块分配到 SM 中，会以 32 个线程为一组进行分割形成一个 warp
- 每个线程束只能包含同一线程块中的线程，线程束在 GPU 硬件上真正做到了并行，线程束数量=ceil(线程块中的线程数/32)

![](https://s2.loli.net/2025/01/15/fsoHDeh8ckqiB5j.png)

# CUDA 内存模型

![](https://s2.loli.net/2025/01/15/6HwLNoWZ4XTVUqz.png)

![](https://s2.loli.net/2025/01/15/bTqFBGtzYoH3AlM.png)

![](https://s2.loli.net/2025/01/15/McTXdzg3QvtAySe.png)

# 寄存器和本地内存(局部内存)

寄存器内存在片上(on-chip)，具有 GPU 上最快的访问速度，但是数量有限，属于 GPU 的稀缺资源
寄存器仅可在线程内可见，生命周期也与所属线程一致
核函数中定义的不加任何限定符的变量一般存放在寄存器中(限定符类似于\_\_global\_\_)
内建变量存放于寄存器中，如 gridDim、blockDim、blockldx 等
核函数中定义的不加任何限定符的数组有可能存在于寄存器中，但也有可能存在于本地内存中

寄存器都是 32 位的，保存 1 个 double 类型的数据需要两个寄存器，寄存器保存在 SM 的寄存器文件
计算能力 5.0~9.0 的 GPU，每个 SM 的寄存器数量都是 64K 个，Fermi 架构只有 32K 个
每个线程块使用的最大数量不同架构是不同的
每个线程的最大寄存器数量是 255 个，Fermi 架构是 63 个

寄存器放不下的内存会存放在本地内存：

1、索引值不能在编译时确定的数组存放于本地内存
2、可能占用大量寄存器空间的较大本地结构体和数组
3、任何不满足核函数寄存器限定条件的变量

每个线程最多可使用 512KB 的本地内存
本地内存从硬件角度看只是全局内存的一部分，延迟也很高，本地内存的过多便用，会减低程序的性能
对于计算能力 2.0 以上的设备，本地内存的数据存储在每个 SM 的一级缓存和设备的二级缓存中

核函数所需的寄存器数量超出硬件设备支持，数据则会保存到本地内存(local memory)中：
1、一个 SM 运行并行运行多个线程块/线程束，总的需求寄存器容量大于 64KB
2、单个线程运行所需寄存器数量超过 255 个
寄存器溢出会降低程序运行性能:
1、本地内存只是全局内存的一部分，延迟较高
2、寄存器溢出的部分也可进入 GPU 的缓存中

# 全局内存

全局内存在片外，即 GPU 内存大小
特点：容量最大，延迟最大，使用最多
全局内存中的数据所有线程可见，Host 端可见且具有与程序相同的生命周期

全局内存初始化：

- 动态全局变量：`cudaMalloc/cudaFree`
- 静态全局变量：使用`__device__`关键词声明(不能在函数中定义)

```c++
// 将host数据复制到静态全局内存
__host__ cudaError_t cudaMemcpyToSymbol(const void *symbol, const void *src, size_t count, size_t offset, cudaMemcpyKind kind);
// 将静态全局内存数据复制到host(CPU端)
__host__ cudaError_t cudaMemcpyFromSymbol(const void *dst, const void *symbol, size_t count, size_t offset, cudaMemcpyKind kind);
```

# 共享内存

- 共享内存在片上(on-chip)，与本地内存和全局内存相比具有更高的带宽和更低的延迟
- 共享内存中的数据在线程块内所有线程可见，可用于线程间通信，共享内存的生命周期也与所属线程块一致
- 使用 `__shared__` 修饰的变量存放于共享内存中，共享内存可定义为动态与静态两种
- 每个 SM 的共享内存数量是一定的，也就是说，如果在单个线程块中分配过度的共享内存，将会限制活跃线程束的数量
- 访问共享内存必须加入同步机制：线程块内使用 `void __syncthreads();`进行同步，不同线程块不能同步

![](https://s2.loli.net/2025/01/15/an28Usj4ifB6ebO.png)

共享内存作用：

- 经常访问的数据存放位置可以由全局内存(global memory)更换到共享内存(shared memory)，提高访问效率
- 改变全局内存的访问内存的内存事务方式，提高数据访问的带宽

静态共享内存&动态共享内存：

```c++
// 静态共享内存
__shared__ float tile[size];
__shared__ float tile[size,size];

// 动态共享内存
extern __shared__ float arr[];// 声明
// 核函数
__global__ void kernel_func();

dim3 block(32);
dim3 grid(2);
// 传递共享内存大小为32
kernel<<<grid,block,32>>>();
```

静态共享内存编译时就需要确定内存大小

# 常量内存

常量内存是有常量缓存的全局内存，数量有限大小仅为 64KB，由于有缓存，线程束在读取相同的常量内存数据时，访问速度比全局内存快
常量内存中的数据对同一编译单元内所有线程可见;
使用 constant 修饰的变量存放于常量内存中，不能定义在核函数中，且常量内存是静态定义的;定义在函数外
常量内存仅可读，不可写;
给核函数传递数值参数时(非指针类型的参数)，这个变量就存放于常量内存。

常量内存必须在主机端使用 cudaMemcpyToSymbol 进行初始化:
线程束中所有线程从相同内存地址中读取数据时，常量内存表现最好，例如数学公式中的系数，因为线程束中所有的线程都需要读取同一个地址空间的系数数据，因此只需要读取一次，广播给线程束中的所有线程。

# GPU 缓存

一级缓存(L1)
二级缓存(L2)
只读常量缓存
只读纹理缓存

![](https://s2.loli.net/2025/01/15/ofBy2RqLUmkOigj.png)

GPU 缓存是不可编程的内存
每个 SM 都有一个一级缓存，所有 SM 共享一个二级缓存
L1 缓存和 L2 缓存用来存储局部内存(local memory)和全局内存(global memory)的数据，也包括寄存器溢出的部分
在 GPU 上只有内存加载可以被缓存，内存存储操作不能被缓存(即从 GPU 内存中读取的数据可以被缓存，而往 GPU 内存中写入数据不可以被缓存)
每个 SM 有一个只读常量缓存和只读纹理缓存，它们用于在设备内存中提高来自各自内存空间内的读取性能

![](https://s2.loli.net/2025/01/15/hIf1stxlQC3594a.png)

- 以计算能力为 8.9 的显卡为例:
  - 统一数据缓存大小为 128KB，统一数据缓存包括共享内存、纹理内存和 L1 缓存
  - 共享内存从统一的数据缓存中分区出来，并且可以配置为各种大小，共享内存容量可设置为 0，8，16，32，64 和 100KB，剩下的数据缓存用作 L1 缓存，也可由纹理单元使用
  - L1 缓存与共享内存大小是可以进行配置的，但不一定生效，GPU 会自动选择最优的配置

伏特架构(计算能力 7.0)：统一数据缓存大小为 128KB 共享内存容量可以设置为 0、8、16、32、64 或 96KB
图灵架构(计算能力 7.5)：统一数据缓存大小为 96KB，共享内存容量可以设置为 32KB 或 64KB
安培架构(计算能力 8.0)：统一数据缓存力小为 192KB，共享内存容量可以设置为 0、8、16、32、64、100、132 或 164KB

# 计算资源分配

线程束本地执行上下文主要资源组成：

- 程序计数器
- 寄存器
- 共享内存

SM 处理的每个线程束计算所需的计算资源属于片上(on-chip)资源，因此从一个执行上下文切换到另一个执行上下文是没有时间损耗的

对于一个给定的内核，同时存在于同一个 SM 中的线程块和线程束数量取决于在 SM 中可用的内核所需寄存器和共享内存数量

- 寄存器对线程数目的影响：

> SM 寄存器总数固定，比如计算能力 8.9，每个 SM 寄存器数量为 64KB

每个线程消耗的寄存器越多，则可以放在一个 SM 中的线程束就越少:
如果减少内核消耗寄存器的数量，SM 便可以同时处理更多的线程束;

- 共享内存对线程块数量的影响：

> SM 共享内存大小固定，比如计算能力 8.9，每个 SM 共享内存大小为 100KB

每个线程消耗的寄存器越多，则可以放在一个 SM 中的线程束就越少:
如果减少内核消耗寄存器的数量，SM 便可以同时处理更多的线程束;

![](https://s2.loli.net/2025/01/15/uqo19pAKYGiLnec.png)

网格和线程块大小的准则:

- 保持每个线程块中线程数量是线程束大小的倍数
- 线程块不要设计的太小
- 根据内核资源调整线程块的大小
- 线程块的数量要远远大于 SM 的数量，保证设备有足够的并行

# 延迟隐藏

指令延迟：在指令发出和完成之间的时钟周期被定义为指令延迟

当每个时钟周期中所有线程束调度器都有一个符合条件的线程束时，可以达到计算资源的完全利用

GPU 的指令延迟被其他线程束的计算隐藏称为延迟隐藏

指令可以被分为两种基本类型： 1)算数指令 2)内存指令

### 算术指令隐藏

算数运算指令延迟是从开始运算到得到计算结果的时钟周期，通常为 4 个时钟周期

满足延迟隐藏所需的线程束数量，利用利特尔法则可以合理提供一个估计值：所需线程束数量 = 延迟 x 吞吐量

带宽-理论情况/吞吐量-实际情况

吞吐量是 SM 中每个时钟周期的操作数量确定的

提升算数指令并行性方法：

1)线程中更多独立指令 2)更多并发线程

### 内存指令隐藏

内存访问指令延迟是从命令发出到数据到达目的地的时钟周期，通常为 400~800 个时钟周期

对内存操作来说，其所需的并行可以表示为在每个时钟周期内隐藏内存延迟所需的字节数

# 避免线程束分化

GPU 支持传统的、C/C++风格的显式控制流结构，如 if...else、for 和 while

GPU 是相对简单的设备，没有复杂的分支预测机制

一个线程束中的所有线程在同一个周期中必须执行相同的指令->避免线程束分支

如果同一个线程束中的线程执行不同分支的指令，则会造成线程束分支

线程束分支会降低 GPU 的并行计算能力，条件分支越多，并行性削弱越严重

线程束分支只发生在同一个线程束中，不同线程束不会发生线程束分化

为获取最佳性能，应避免在同一个线程束中有不同的执行路径

---

在向量中满足交换律和结合律的运算，称为规约问题，并行执行的规约计算称为并行规约计算
