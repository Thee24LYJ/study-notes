参考：
[System Verilog 的概念以及与 verilog 的对比 - 永不止步，永无止境 - 博客园](https://www.cnblogs.com/youngforever/archive/2013/05/24/3104651.html)

# SystemVerilog

SystemVerilog 基于**IEEE1364-2001 Verilog 硬件描述语言（HDL），并对其进行了扩展，包括扩充了 C 语言数据类型、结构、压缩和非压缩数组、 接口、断言等等**

### 接口(Interface)

Interface 采用`interface`和`endinterface`关键字进行定义；SystemVerilog 的接口不仅仅可以表示信号的绑定和互连，还可以包含参数、常量、变量、结构、函数、任务、initial 块、always 块以及连续赋值语句

```systemverilog
interface bus_if;
	wire data;
	wire vld;
endinterface
```

### 全局声明和语句

SystemVeriog 增加了一个被称为`$root`的隐含的顶级层次。任何在模块边界之外的声明和语句都存在于`$root`空间中。所有的模块，无论它处于哪一个设计层次，都可以引用`$root`中声明的名字

### 时间单位和精度

- Verilog 中，表示时间的值使用不带时间单位的一个数来表示，Verilog 的时间单位和精度是作为每一个模块的属性，并使用编译器指令``timescale`来设置时间单位
- SystemVerilog 中可以显式指定时间单位，例如 ns，ps；其次，SystemVerilog 允许使用新的关键字（`timeunit`和`timeprecision`）来指定时间单位和精度。这些声明可以在任何模块中指定，同时也可以在`$root`空间中全局指定。时间单位和精度必须是 10 的幂，范围可以从 s 到 fs

```systemverilog
forever #5ns clock=~clock;

timeunit 1ns;
timeprecision 10ps;
```

### 抽象数据类型

- Verilog 提供了面向底层硬件的线网、寄存器和变量数据类型。这些类型代表了 4 态逻辑值，通常用来在底层上对硬件进行建模和验证。线网数据类型还具有多个强度级别，并且能够为多驱动源的线网提供解析功能
- SystemVerilog 允许在 Verilog 模型和验证程序中直接使用 C 和 C++代码，新加入的数据类型为`char/int/short int/long int/byte/bit/logic/short real/void`

缺省情况下，Verilog wire 和 reg 数据类型是无符号类型，integer 类型是一个有符号类型。Verilog-2001 标准允许使用 signed 关键字将无符号类型显式地声明成有符号类型。SystemVerilog 加入了相似的能力，它可以通过 unsigned 关键字将有符号数据类型显式地声明成有无符号数据类型

### 自定义数据类型、枚举/结构体/联合体

Verilog 不允许用户定义新的数据类型。SystemVerilog 通过使用`typedef`提供了一种方法来定义新的数据类型，这一点与 C 语言类似。用户定义的类型可以与其它数据类型一样地使用在声明当中

```systemverilog
typedef unsigned int uint;
uint a, b;
```

Verilog 语言中不存在枚举类型。标识符必须被显式地声明成一个线网、变量或参数并被赋值。SystemVerilog 允许使用类似于 C 的语法产生枚举类型。一个枚举类型具有一组被命名的值。缺省情况下，值从初始值 0 开始递增，但是我们可以显式地指定初始值

```systemverilog
enum {red, yellow, green} RGB;
enum {WAIT=2'b01, LOAD, DONE} states;

// 结合typedef
typedef enum {FALSE, TRUE} boolean;
boolean ready;
```

Verilog 语言中不存在结构体或联合体，而结构体或联合体在将几个声明组合在一起的时候非常有用；SystemVerilog 增加了结构体和联合体，它们的声明语法类似于 C

```systemverilog
struct {
  reg [15:0] opcode;
  reg [23:0] addr;
} IR;

union {
  int I;
  shortreal f;
} N;

// 结合typedef
typedef struct{
  reg [7:0] opcode;
  reg [23:0] addr;
} instruction; // 命名的结构体
instruction IR; // 结构体实例
```

### 数组

[System Verilog 中的 packed array 和 unpacked array_packed array type-CSDN 博客](https://blog.csdn.net/fangfanglovezhou/article/details/131550844)
[Systemverilog 压缩数组 packed-CSDN 博客](https://blog.csdn.net/qq_42043804/article/details/122529744)

- 在 Verilog 中可以声明一个数组类型，reg 和线网类型还可以具有一个向量宽度。在一个对象名前面声明的尺寸表示向量的宽度，在一个对象名后面声明的尺寸表示数组的深度 -在 SystemVerilog 中我们使用不同的术语表示数组：使用“压缩数组（packed array）”这一术语表示在对象名前声明尺寸的数组；使用“非压缩数组（unpacked array）”这一术语表示在对象名后面声明尺寸的数组。压缩数组可以由下面的数据类型组成：bit、logic、reg、wire 以及其它的线网类型。无论是压缩数组还是非压缩数组都可以声明成多维的尺寸

```systemverilog
bit [7:0] a; // 一个一维的压缩数组
bit b [7:0]; //一个一维的非压缩数组
bit [0:11] [7:0] c; //一个二维的压缩数组
bit [3:0] [7:0] d [1:10]; // 一个包含10个具有4个8位字节的压缩数组的非压缩数组
```

### 常量

Verilog 中有三种特性类型的常量：parameter、specparam 和 localparam；SystemVerilog 中，允许使用 const 关键字声明常量
在 Verilog 中，可以连接到模块端口的数据类型被限制为线网类型以及变量类型中的 reg、integer 和 time。而在 SystemVerilog 中则去除了这种限制，任何数据类型都可以通过端口传递，包括实数、数组和结构体

### 强制类型转换

Verilog 不能将一个值强制转换成不同的数据类型。SystemVerilog 通过使用`<type>'`操作符提供了数据类型的强制转换功能。这种强制转换可以转换成任意类型，包括用户定义的类型

```systemverilog
// 转换为int类型
int'(2.0*3.0)
// 转换成不同的向量宽度
18'(x-2)
```

### 唯一性和优先级决定语句

- 在 Verilog 中，如果没有遵循严格的编码风格，它的 if-else 和 case 语句会在 RTL 仿真和 RTL 综合间具有不一致的结果。如果没有正确使用 full_case 和 parallel_case 综合指令还会引起一些其它的错误
- SystemVerilog 能够显式地指明什么时候一条决定语句的分支是唯一的，或者什么时候需要计算优先级。我们可以在 if 或 case 关键字之前使用 unique 或 requires 关键字。这些关键字可以向仿真器、综合编译器、以及其它工具指示我们期望的硬件类型。工具使用这些信息来检查 if 或 case 语句是否正确建模了期望的逻辑。例如，如果使用 unique 限定了一个决定语句，那么在不希望的 case 值出现的时候仿真器就能够发布一个警告信息

### 对事件控制的增强

Verilog 使用@标记来控制基于特定事件的执行流，SystemVerilog 增强了@事件控制

SystemVerilog 在事件控制中加入了一个 iff 条件。只有 iff 条件为真的条件下，事件控制才会被触发

```systemverilog
// en无效时data变化也会触发
always @(data or en)
// 只有iff条件为真(即en等于1时data变化才触发)
always @(data or en iff en==1)
```

Verilog 允许在@事件控制列表中使用表达式，当@事件控制中包含表达式的时候，IEEE Verilog 标准允许仿真器进行不同的优化。这就可能导致在不同的仿真器间有不同的仿真结果，可能还会导致仿真与综合之间的结果不一致。SystemVerilog 加入了一个 changed 关键字，在事件控制列表中它被用作一个修饰符。@(changed (表达式))能够显式地定义只有当表达式的结果发生改变的时候才会触发事件控制

```systemverilog
always @((a * b))
always @(memory[address])
-->
always @(changed (a * b))
always @(changed memory[address])
```

### 新的过程语句

Verilog 使用 always 过程来表示时序逻辑、组合逻辑和锁存逻辑的 RTL 模型。综合工具和其它软件工具必须根据过程起始处的事件控制列表以及过程内的语句来推断 always 过程的意图。这种推断会导致仿真结果和综合结果之间的不一致
SystemVerilog 增加了三个新的过程来显式地指示逻辑的意图

- always_ff：表示时序逻辑的过程
- always_comb：表示组合逻辑的过程
- always_latch：表示锁存逻辑的过程

### 动态并发执行过程

- Verilog 通过使用 fork-jion 提供了一种静态的并发过程。每一个分支都是一个分离的、并行的过程。fork-jion 中任何语句的执行必须在组内的每一个过程完成后才会执行
- SystemVerilog 通过 process 关键字加入了一个新的、动态的过程。它为一个过程产生分支，然后继续执行而无需等待其他过程完成。过程不会阻塞过程或任务内的语句执行。这种方式能够建模多线程的过程
