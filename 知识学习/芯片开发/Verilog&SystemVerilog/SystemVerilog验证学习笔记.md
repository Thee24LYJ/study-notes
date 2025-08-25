参考：
[SystemVerilog 新手入门笔记 - 简书](https://www.jianshu.com/p/013c4066dd85)

### 变量

> logic:一位二进制
>
> logic 不能有多个结构性的驱动，只能有一个驱动

> verilog 中的 wire 类型(线网类型)表示导线，输出跟着输入变化；而 reg 类型(寄存器类型)表示储存单元，只有遇到相应触发信号其存储数据才会改变

```verilog
logic a;	//一位的二进制变量a
logic [3:0]b;	//4位(高位:低位)二进制变量b(类似C语言中的一维数组)
logic [31:0][31:0]c;//32位×32位二进制变量c(类似C语言中的二维数组)
```

# 一、数据类型

### 1.内建数据类型

|     数据类型     |           释意            |
| :--------------: | :-----------------------: |
|      bit b;      |      双状态，单比特       |
| bit [31:0] b32;  | 双状态，32 比特无符号整数 |
| int unsigned ui; | 双状态，32 比特无符号整数 |
|      int i;      | 双状态，32 比特有符号整数 |
|     byte b8;     | 双状态，8 比特有符号整数  |
|   shortint s;    | 双状态，16 比特有符号整数 |
|    longint l;    | 双状态，64 比特有符号整数 |
|   integer i4;    | 四状态，32 比特有符号整数 |
|     time t;      | 四状态，64 比特无符号整数 |
|     real r;      |   双状态，双精度浮点数    |

> 使用$isunknown()操作符，在表达式中任意位出现 X 或 Z 时返回 1
>
> 使用格式符%0t 和参数\$time 打印出当前仿真时间，打印格式在$timeformat()子程序中指定

### 2.定宽数组

##### 2.1 定宽数组声明和初始化

> verilog 要求在声明中必须给出数组上下界，但 systemverilog 允许只给出数组宽度来声明数组，跟 C 语言类似
>
> 若代码试图从越界地址读取数据，systemverilog 将返回数组元素类型缺省值(四状态返回 X，双状态返回 0)

```verilog
int a[0:15];	//16个整数
int c_style[16];	//16个整数
```

- 在变量名后指定维度创建多维定宽数组

```verilog
int array[0:7][0:3];	//完整声明
int array2[8][4];		//紧凑声明
```

##### 2.1 常量数组

> 使用一个单引号加大括号来初始化数组

```verilog
int ascend[4]='{0,1,2,3};	//初始化四个元素
int descend[5];

descend='{4,3,2,1,0};	//为五个元素赋值
descend[0:2]='{5,6,7};	//为前三个元素赋值
descend='{4{8}};		//四个元素全部为8
descend='{9,8,default:1};//{9,8,1,1,1}
```

##### 2.3 基本数组操作(for 和 foreach)

> 使用$size()函数返回数组宽度
>
> foreach 循环中，只需要指定数组名并在其后面方括号中给出索引变量，索引变量将自动声明并只在循环中有效，若需循环的数组为多维数组，则像[i,j]表示维度索引(不是用\[i][j]表示)

```verilog
initial begin
	bit [31:0] src[5],dst[5];
	for(int i=0;i<$size(src);i++)
		src[i]=i;
	foreach(dst[j])
		dst[j]=src[j]*2;
end
```

##### 2.4 基本数组操作(复制和比较)

> 不使用循环对数组进行聚合比较和复制(聚合操作适用于整个数组而不是单个元素，其中比较只适用于等于和不等于比较)
>
> 对数组的算术运算不能使用聚合操作而应该使用循环；对于逻辑运算，只能使用循环或合并数组

##### 2.5 同时使用位下标和数组下标

```verilog
bit [31:0] src[5]='{5{5}};
src[0];		//'b101或'd5
src[0][0];	//'b1
src[0][2:1];//'b10
```

##### 2.6 合并数组

> 对于某些数据类型，既想把它作为整体访问又想把它分解为更小的单元，此时使用合并数组，与合并数组不同的是它的存放方式是连续的比特集合，中间没有任何闲置的空间
>
> 合并数组声明时，合并的位和数组大小作为数据类型的一部分必须在变量名前面指定，数组大小定义格式必须是[msb:lsb]而不是[size]
>
> 需要和标量进行相互转换时，使用合并数组更为简便

```verilog
bit [3:0] [7:0] bytes;	//4字节组成32比特
```

### 3.动态数组

> 声明时使用空下标[]，数组在最开始时是空的，必须调用 new[]操作符来分配空间
>
> 基本数据类型相同，定宽数组可以和动态数组间相互赋值

```verilog
int dyn[];	//动态数组声明
dyn=new[5];	//分配5个元素
dyn=new[20](dyn);	//分配20个元素并将原来的值复制给新数组dyn的开始5个元素，释放原来5个元素的空间
```

### 4.队列

> 队列声明使用美元符号下标: [$]，编号从 0 到\$，它结合了链表和数组的优点，可以在队列中任何地方增加或删除元素，也可以通过索引访问任意元素，队列中的元素是连续存放的
>
> 若把$放到范围表达式左边则代表最小值，右边代表最大值
>
> 注意：队列的常量只有大括号而没有数组常量中开头的单引号，不要对队列使用构造函数 new[]

```verilog
int j = 1,
q[$] = {3,4};	//队列声明常量无需单引号

initial begin
    q.insert(1,j);	//在2(索引为1的值)之前插入j(即1) {1,1,2,3}
    q.delete(1);	//删除索引为1的元素 {1,2,3}
    q.push_front(6);//在q最前面插入6 {6,1,2,3}
    j = q.pop_back; //j=3 {6,1,2}
    q.push_back(8);	//{6,1,2,8}
    j = q.pop_front;//{1,2,8}
    q = {6,q};		//{6,1,2,8}
    q = {q,0};		//{6,1,2,8,0}
    q.delete();		//删除整个队列
    q = {};			//删除整个队列
end
```

### 5.关联数组

> systemverilog 提供关联数组用来保存稀疏矩阵的元素，这意味着当对一个非常大的地址空间寻址时，systemverilog 只对实际写入的元素分配空间
>
> 声明：
>
> type name[type];
>
> type: 数据类型
>
> name: 关联数组名

```verilog
initial begin
    //关联数组声明
    bit [63:0] assoc[bit[63:0]],idx = 1;
    //关联数组初始化
    repeat(64) begin
        assoc[idx] = idx;
        idx = idx << 1;
    end
end
```

### 6.链表

> 尽量避免使用链表，而是使用更为高效易用的队列

### 7.数组的方法

> 适用于任何一种非合并的数组类型，包括定宽数组、动态数组、队列和关联数组
>
> 这些方法如果不带参数则方法中的圆括号可以省略，但我觉得最好保留

- 数组缩减方法

> 基本数组缩减方法是把数组缩减成一个值
>
> 常用的是 sum()，对数组所有元素求和；其它方法有 product(乘积)、and(与)、or(或)和 xor(异或)

- 数组定位方法

> 数组定位方法返回值通常是一个队列
>
> min(): 数组最小值
>
> max(): 数组最大值
>
> unique(): 排除重复数值后返回的队列
>
> 表达式 with 指示 systemverilog 如何进行搜索，在条件语句 with 中，item 称为重复参数，代表数组中的单独一个元素，item 是缺省的名字，如果要指定其它名字则需在数组方法的参数列表中指定
>
> 返回值为索引的数组定位方法，其返回的队列类型是 int 而不是 integer

- 数组排序

> 与数组定位方法不同的是，排序方法更改了原来的数组，数组定位方法新建了一个队列来保存定位的结果
>
> reverse(): 倒转数组
>
> sort(): 从小到大排序
>
> rsort(): 从大到小排序
>
> shuffle(): 打乱顺序
>
> 注意：reverse 和 shuffle 方法不能带 with 条件语句，它们的作用范围是整个数组

### 8.选择存储类型

- 灵活性

> 数组索引为连续的非负整数时选择定宽或动态数组

- 储存器用量

> 使用双状态类型可以减小仿真时的储存器用量，为避免浪费空间，应该尽量选择 32 比特的整数倍作为数据位宽

- 速度

> 数组操作很频繁，则数组的宽度和类型就很关键，队列读写速度与定宽或动态数组基本相当

- 排序

> 元素一次性加入：选择定宽或动态数组
>
> 元素逐个加入：选择队列

- 选择最优数据结构

> - 网络数据包(长度固定，顺序存取): 定宽或动态数组
> - 保存期望值的计分板(仿真前长度未知，按值存取，长度经常变化): 一般使用队列，有时候使用关联数组
> - 有序结构：输出顺序确定采用队列，都则采用关联数组

### 9.使用 typedef 创建新类型

> 类似于 C 语言中的 typedef

```verilog
parameter OPSIZE = 8;
typedef reg [OPSIZE-1:0] opreg_t;

opreg_t op_a,op_b;
//定义新的数组类型
typedef int fixed_array5[5];
fixed_array5 f5;	//f5为数组即 int f5[5];
```

### 10.创建自定义结构

> struct 只是把数据组织到一起(即数据的集合，可综合)，且功能比类少

##### 10.1 使用 struct 创建新类型

> 这里只是创建了 pixel 变量，如果想在端口和程序中共享它则必须创建一个新的类型

```verilog
struct {bit [7:0] r,g,b} pixel;//这里的声明只是创建pixel变量
typedef struct {bit [7:0] r,g,b;} pixel_t;

pixel_t my_pixel;//创建pixel类型
```

##### 10.2 对结构进行初始化

> 在声明或过程赋值语句中把多个值赋给结构体，赋值时要把数值放到带单引号的大括号中

```verilog
initial begin
    typedef struct {
        int a;
        byte b;
        shortint c;
        int d;
    }my_struct_t;
    my_struct_t st = '{
        32'bhaaaa_aaaad,
        8'hbb,
        16'hcccc,
        32'hdddd_dddd
    };
```

##### 10.3 创建可容纳不同类型的联合

> 若需要以不同格式对同一寄存器进行频繁读写时，联合体相当有用

```verilog
typedef union{
    int i;
    real f;
}num_u;
num_u un;
un.f = 0.0;	//设置数值为浮点形式
```

##### 10.4 合并结构

> 合并结构允许对数据在寄存器中的排布方式有更多的控制，合并结构是以连续比特集的方式存放的，中间没有闲置空间
>
> 希望减少存储器的使用量或存储器的部分位代表数值时使用合并结构

```verilog
typedef struct packed {
    bit [7:0] r,g,b;
}pixel_p_t;
pixel_p_t my_pixel;
```

##### 10.5 合并结构和非合并结构的选择

> 若对结构操作很频繁，则使用合并结构效率较高
>
> 若操作经常是针对结构体内部的个体成员而非整体则使用非合并结构

### 11.类型转换

##### 11.1 静态转换

> 转换时指定目标类型并在需要转换的表达式前加上单引号

```verilog
int i;
real r;

i = int'(10.0-0.1);
r = real'(42);
```

##### 11.2 动态转换

> 动态转换函数$cast 允许我们对越界的数值进行检查
>
> $cast(value1,value2): 将 value2 赋值给 value1，如果赋值成功则返回 1，如果因数值越界导致失败则返回 0

##### 11.3 流操作符

> 流操作符<<和>>用在赋值表达式右边，后接表达式、结构或数组
>
> 流操作符用于将其后的数据打包成一个比特流，操作符>>把数据从左至右变成流，<<则把数据从右至左变成流，也可以指定一个片段宽度把源数据按照这个宽度分段以后再转变成流
>
> 流操作符可以完成具有不同尺寸元素的数组间的转换，也可以将结构打包或拆分到字节数组中

### 12.枚举类型

> 最简单的枚举类型声明包含一个常量名称列表以及一个或多个变量，但通过这种方式创建的是一个匿名的枚举类型，只能用于这个例子中声明的变量
>
> 创建署名的枚举类型有利于声明更多新变量，使用内建函数 name()可以得到枚举变量值对应的字符串

```verilog
enum {RED,BLUE,GREEN} color;//简单的枚举类型声明
typedef enum {INIT,DECODE,IDLE} fsmstate_e;
fsmstate_e pstate,nstate;	//声明自定义类型变量
```

##### 12.1 定义枚举值

> 枚举值缺省时是从 0 开始递增的整数，也可以指定枚举值

##### 12.2 枚举类型的函数

> first(): 返回第一个枚举常量
>
> last(): 返回最后一个枚举常量
>
> next(): 返回下一个枚举常量，若带参数 N，则返回以后第 N 个枚举常量
>
> prev(): 返回前一个枚举常量，若带参数 N，则返回以前第 N 个枚举常量
>
> 当到达枚举常量列表的头或尾部时，next()和 prev()会自动以环形方式绕回

##### 12.3 枚举类型转换

> 枚举类型缺省类型为双状态 int，可以使用简单赋值表达式把枚举变量的值直接赋值给非枚举变量如 int，但是不可以在没有进行显式类型转换的情况下把整型变量赋值给枚举变量(显示转换目的在于意识到可能存在的数值越界情况)

### 13.常量

> verilog 中创建常量最典型的方式是使用文本宏，好处是宏具有全局作用范围并且可以位于位段和类型定义，宏定义需要使用`·`符号
>
> systemverilog 中可以使用 typedef、parameter 来替换表示常量的宏，也支持使用 const 修饰符

### 14.字符串

> string 类型可以保存长度可变的字符串，单个字符是 byte 类型，{}用于串接字符串
>
> 与 C 语言字符串不同的是，systemverilog 中字符串结尾并不带标识符 null(\0)
>
> getc(N): 返回位置 N 上的字节
>
> toupper(): 返回大写字符串
>
> tolower(): 返回小写字符串
>
> putc(M,C): 把字节 C 写到字符串的 M 位上，0$\le$M$\le$len-1
>
> substr(start,end): 提取位置 start 到 end 的所有字符

```verilog
string s;
initial begin
    s = "IEEE ";
    s = {s,"P1800"}
end
```

> $psprintf()函数返回一个格式化的临时字符串并可以直接传递给其他子程序

### 15.表达式的位宽

> verilog 中表达式位宽是造成行为不可预知的主要源头之一
>
> 因此，需要使用临时变量、强制转换等方式以达到期望的精度

# 二、过程语句和子程序

### 1.过程语句

> `for`循环中定义的循环变量作用范围仅限于循环内部，在`begin`或`fork`语句中使用标识符，则在对应的`end`或`join`中放置相同标号，使得程序块收尾匹配更容易
>
> `systemverilog`的`continue`和`break`作用基本与`C`语言相同

### 2.任务、函数以及 void 函数

> `verilog`：任务可以消耗时间而函数不能消耗时间，函数中不能带阻塞语句，也不能调用任务，函数必须有返回值且必须被使用
>
> `systemverilog`：允许函数在`fork...join none`语句生成的线程中调用任务
>
> void 函数没有返回值，这使得它能够被任何任务或函数调用，从最大灵活性角度考虑，所有用于调试的子程序都应该被定义为`void`函数

```verilog
function void print_state(...);
    $display("@%0t:state=%s",$time,cur_state.name());
endfunction

//忽略函数返回值
void'($fscanf(file,"%d",i));
```

### 3.任务和函数概述

- 子程序中去掉`begin...end`

> `systemverilog`中的`begin...end`变成可选的了

### 4.子程序参数

##### 4.1 C 语言风格的子程序参数

> `verilog`中任务要求对一些参数进行两次声明(方向声明和类型声明)，`systemverilog`中可以采用简明的`C`语言风格，但必须使用通用输入类型`logic`

##### 4.2 参数方向

> 子程序参数缺省类型和方向是`logic input`，建议声明子程序参数时带上类型和方向

##### 4.3 高级参数类型

> - 参数传递方式指定为引用`ref`，此时不会将参数复制到本地变量，但`ref`参数只能被用于带自动存储的子程序中
>
> 向子程序中传递数组时尽量使用`ref`以获取最佳性能，若不希望子程序改变数组的值可以使用`const`确保不会被修改
>
> - `ref`参数可以让我们在任务中修改变量而且修改结果对调用它的函数随时可见

##### 4.4 参数缺省值

> 调用子程序时不指名参数则使用缺省值

##### 4.5 采用名字进行参数传递

> 对于带有多个参数的任务或函数，若只想对其部分参数修改，则采用类似于`port`的语法指定子程序的参数名字的方式进行参数传递

```verilog
task may(input int a=1,b=2,c=3,d=4);
    $display("%0d %0d %0d %0d",a,b,c,d);
endtask

initial begin
    many(6,7,8,9);	//6 7 8 9
    many();			//1 2 3 4
    many(.c(5));	//1 2 5 4
    mangy(,6,.d(8));//1 6 3 8
```

### 5.子程序的返回

##### 5.1 `return`语句

##### 5.2 函数中返回数组

> `verilog`中子程序只能返回一个简单值(比特、整数或向量)
>
> `systemverilog`中采用多种方式返回数组

- 首先定义一个数组类型，然后在函数声明中使用该类型
- 使用引用进行数组参数的传递
- 将数组包装到类中然后返回对象的句柄

### 6.局部数据存储

> `verilog`的所有对象都是静态分配的

##### 6.1 自动存储

> 指定任务、函数和模块使用自动存储(`automatic`)迫使仿真器使用堆栈区存储局部变量

##### 6.2 变量初始化

> `systemverilog`中模块和`program`块中的子程序缺省情况下仍然使用静态存储，当试图在声明中初始化局部变量时，会导致多次调用时该局部变量值被覆盖，这是因为局部变量实际上在仿真开始前就被赋了初值，常规解决办法是避免在变量声明中赋予除常数外的任何值，也可以将声明和初始化分离

### 7.时间值

> `timeunit`和`timeprecision`声明语句可以明确地为每个模块指明时间值
>
> `$time`：返回值是一个根据所在模块时间精度要求进行舍入的整数
>
> `$realtime`：返回值是带小数部分的完整实数

# 三、连接设计和测试平台

> 验证设计的步骤：
>
> - 生成输入激励
> - 捕获输出响应
> - 决定对错和衡量进度

### 1.将测试平台和设计分开

> `systemverilog`中引入程序块(`program block`)从逻辑和时间上将测试平台分开，避免`verilog`使用模块保存测试平台引起的驱动和采样时的时序问题
>
> 为解决模块复杂连接导致的问题，`systemverilog`使用接口(代表一捆连线的结构)，它是具有智能同步和连接功能的代码(接口既可以像模块一样例化也可以像信号一样连接到端口)

### 2.接口

```verilog
// 仲裁器简单接口：
interface arb_if(input bit clk);
    logic [1:0] grant,request;
    logic rst;
endinterface

// 使用简单接口的仲裁器
module arb(arb_if arbif);
    ...
    always @(posedge arbif.clk or posedge arbif/rst)
        begin
            if(arbif.rst)
                arbif.grant <= 2'b00;
            else
                arbif.grant <= next_grant;
            ...
        end
endmodule
```

> 测试平台通过实例名`arbif.request`来引用接口信号，接口信号必须使用非阻塞赋值来驱动，使用接口时需要确保在模块和程序块之外声明接口变量
>
> 在接口中使用`modport`结构能够将信号分组并指定方向
>
> 接口中不能例化模块但是能例化其他接口

```verilog
interface arb_if(input bit clk);
    logic [1:0] grant,request;
    logic rst;

    modport TEST (
        output request,rst,
        input grant,clk
    );
    modport DUT(
        input request,rst,clk,
        output grant
    );
    modport MONITOR(
        input request,grant,rst,clk
    );
endinterface

//接口中使用modport的仲裁器模型
module arb(arb_if.DUT arbif);
    ...
endmodule
//接口中使用modport的测试平台
module arb(arb_if.TEST arbif);
    ...
endmodule
```

### 3.激励时序

- 使用时钟块控制同步信号的时序

```verilog
interface arb_if(input bit clk);
    logic [1:0] grant,request;
    logic rst;
    //时钟块声明
    clocking cb @(posedge clk);
        output request;
        input grant;
    endclocking

    modport TEST (
        clocking cb,
        output rst
    );
    modport DUT(
        input request,rst,clk,
        output grant
    );
endinterface
//简单的测试平台
module test(arb_if.TEST arbif);
	initial begin
		arbif.cb.request <=0;
		@arbif.cb;
		$display("%0t: Grant = %b",$time,arbif.cb.grant);
	end
endmodule
```

### 4.接口驱动和采样

> 测试平台的驱动和采样设计的信号主要是通过带有时钟块的接口做到的
>
> 异步信号通过接口时没有任何时延，时钟块的信号将得到同步

##### 4.1 接口同步

> `verilog`中的`@`和`wait`来同步测试平台的信号

```verilog
program automatic test(bus_if.TB bus);
    initial begin
        @bus.cb;	//时钟块有效时钟沿继续
        repeat (3) @bus.cb;	//等待三个有效时钟沿
        @bus.cb.grant;	//在任何边沿继续
        @(posedge bus.cb.grant);//上升沿继续
        @(negedge bus.cb.grant);//下降沿继续
        wait (bus.cb.grant == 1);//等待表达式为真，已经是真则不做任何延时
        @(posedge bus.cb.grant or negedge bus.rst);//等待几个信号
    end
endprogram
```

##### 4.2 接口信号采样

> 当从时钟块中读取一个信号时是在时钟沿之前得到的采样值
>
> 在时钟块中使用`modport`时任何同步接口信号都必须加上接口名(`arbif`)和时钟块名(`cb`)

> 程序块中不允许使用`always`块，如果确实需要使用，可以使用`initial forever`

> 模块间的连接：可以使用快捷符号`.*`(隐式端口连接)，可以自动在当前级别自动连接模块实例的端口到具体信号，只要端口和信号的名字和数据类型相同

### 5.顶层作用域

> 任何`module、interface、program`边界外的作用域被称为编译单元作用域，也称为`$unit`，这个作用域内的任何成员都类似于全局变量，但与真正的全局变量不同
>
> 顶层作用域：块以外的作用域

### 6.程序-模块交互

> 程序块可以读写模块中的所有信号，可以调用模块中的所有例程，但是模块看不到程序块

### 7.断言

> 断言比`if`语句更加紧凑，断言括号内的表达式为真则正常运行，否则输出错误

```verilog
bus.cb.request <= 1;
repeat (2) @bus.cb;
a1:assert (bus.cb.grant == 2'b01);
```

- 定制断言行为

> 一个立即断言有可选的`then`和`else`分句，可以自定义输出消息
>
> `systemverilog`中有`$info、$warning、$error、$fatal`四个输出消息的函数，但仅允许在断言内部使用而不允许在过程代码中使用，后续将被允许

```verilog
a1:assert (bus.cb.grant == 2'b01)
    grants_received++;
else
    $error("Grant not asserted");
```

- 并发断言：连续运行的模块，为整个仿真过程检查信号的值

### 8.ref 端口的方向

> `inout`用于建模双向连接
>
> `ref`端口是对变量(不能是`net`)的引用，它的值是该变量最后一次赋的值

# 四、面向对象编程基础

> 面向对象编程(OOP)使用户能够创建复杂的数据类型并将它们跟使用这些数据类型的程序紧密结合在一起
>
> 发生器创建事务并将它们传给下一级，驱动器和设计进行会话，设计返回的事务将被监视器捕获，记分板将捕获的结果跟预期的结果进行对比

### 1.编写第一个类

> 类可以定义在 program、module、package 或这些块外的任何地方，类可以在程序和模块中使用
>
> 文件数量太大时使用 systemverilog 的包(package)将一组相关的类和类型定义捆绑在一起

```verilog
class Transaction;
    bit [31:0] addr, crc, data[8];

    function void display;
        $display("Transaction:%h",addr);
    endfunction : display

    function void calc_crc;
        crc = addr ^ data.xor;
    endfunction:calc_crc
endclass:Transaction
```

### 2.OOP 术语

> - 类(class)：包含变量和子程序的基本构建块
> - 对象(object)：类的实例
> - 句柄(handle)：指向对象的指针
> - 属性(property)：储存数据的变量
> - 方法(method)：任务或函数中操作变量的程序性代码
> - 原型(prototype)：程序的头、返回类型和参数列表

### 3.创建对象

##### 3.1 句柄的声明和使用

> 通过声明句柄来创建对象，而一个句柄可以指向很多对象但是同一时间只能指向一个对象
>
> 避免声明一个句柄的时候调用构造函数即 new 函数

```verilog
Transaction tr;	//声明句柄
tr = new();	//分配空间
```

##### 3.2 定制构造函数

> 构造函数除分配内存外还能初始化变量，默认情况下二值变量为 0，四值变量为 X，也可以自定义构造函数(new 函数)

##### 3.3 new()和 new[]的区别

> 调用 new()函数仅创建一个对象而 new[]操作则建立一个含有多个元素的数组
>
> new()可以使用参数设置对象的数值，而 new[]只需使用一个数值设置数组大小

### 4.对象的解除分配

> 当最后一个句柄不再引用某个对象时，systemverilog 会释放该对象的空间

```verilog
Transaction t;//创建句柄
t = new();//分配一个新的Transaction
t = new();//分配第二个且释放第一个t
t = null;//解除分配第二个
```

> 对对象使用`.`符号来引用变量和子程序
>
> 静态变量通常在声明时初始化，类中创建的静态变量使用范围仅限于这个类且只会存在唯一一个，不同对象共享这个变量；对于静态变量，直接使用类名加上`::`即类作用域操作符就可以引用静态变量
>
> 类中的方法默认使用自动存储，即采用堆栈区储存局部变量
>
> 在类之外定义方法(或函数)，与 C++不同的是，systemverilog 需要在方法前面添加 extern

- 当需要编译的类中包含一个尚未定义的类，声明这个包含的尚未定义的类的句柄将会引起错误，这时候需要使用 typedef 语句声明一个类名

```verilog
typedef class Statistics;//声明类名

class Transaction;
    Statistics states;//使用Statistics类
    ...
endclass

class Statistics;//定义Statistics类
    ...
endclass
```

### 5.对象的复制

##### 5.1 使用 new 操作符复制一个对象

> 创建新对象并复制现有对象的所有变量
>
> 若类中包含一个指向另一个类的句柄，则只有最高一级的对象被 new 操作符复制，下层对象都不会被复制

```verilog
Transaction src, dst;
initial begin
    src = new();//创建第一个对象
    dst = new src;//复制对象
end
```

- 使用流操作符从数组到打包对象或从打包对象到数组
