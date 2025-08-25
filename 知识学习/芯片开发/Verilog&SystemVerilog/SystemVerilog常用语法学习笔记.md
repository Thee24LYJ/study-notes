> systemverilog 的常用语法(可综合语法)

# 一、二进制

### 1.常量

> [width] ' (type)[value]
>
> width: (二进制的)位宽，可以为 1/8/16/32/64 等(未标明位宽，自动设置为 32 位？`a = 1;`<-->`a = 32'b1;`)
>
> type: 数据类型，b 表示二进制，d 表示十进制，h 表示十六进制
>
> value: 常量值

```verilog
1'b1
1'b0
```

### 2.变量

> logic:一位二进制
>
> logic 不能有多个结构性的驱动，只能有一个驱动
>
> [type] [high:low]name;
>
> type: 变量类型
>
> \[high:low\]: 高位和低位
>
> name: 变量名称

```verilog
logic a;	//一位的二进制变量a
logic [3:0]b;	//4位(高位:低位)二进制变量b(类似C语言中的一维数组)
logic [31:0][31:0]c;//32位×32位二进制变量c(类似C语言中的二维数组)
```

### 3.组合/位绑定

```verilog
{a, 1'b1}
//高位是a,低位是二进制常数1的2两位向量
{a, b}
//高位是a,低四位是b的五位向量
{16{1'b1}}
//16位二进制，全为1
{a, {16{b}}}
//高位为a,低位为16个重复的b
```

# 二、运算符

### 1.基本运算符

|        操作符        |      描述      |        使用方法(a,b)         |       输出值       |
| :------------------: | :------------: | :--------------------------: | :----------------: |
|          &           |     与运算     |     a[x-1:0] & b[x-1:0]      |      c[x-1:0]      |
|          \|          |     或运算     |     a[x-1:0] \| b[x-1:0]     |      c[x-1:0]      |
|          ~           |     非运算     |          ~a[x-1:0]           |      c[x-1:0]      |
|          ^           |    异或运算    |     a[x-1:0] ^ b[x-1:0]      |      c[x-1:0]      |
|          !           |     逻辑非     |          !a[x-1:0]           | ?(一位 0/需要测试) |
|         +, -         |   加,减运算    |     a[x-1:0] + b[x-1:0]      |    ?(需要测试)     |
|         ? :          |    条件运算    | a[0:0] ? b[x-1:0] : c[x-1:0] |      d[x-1:0]      |
|          >>          |    逻辑右移    |   a[$$2^x$$-1:0]>>b[x-1:0]   |   c[$$2^x$$-1:0]   |
|          <<          |    逻辑左移    |   a[$$2^x$$-1:0]<<b[x-1:0]   |   c[$$2^x$$-1:0]   |
|         >>>          |    算术右移    |  a[$$2^x$$-1:0]>>>b[x-1:0]   |   c[$$2^x$$-1:0]   |
|       \*, /, %       | 乘除、取模运算 |     运算慢，实际较少使用     |     ?(待测试)      |
| <, <=, !=, >, >=, == |    比较运算    |     a[x-1:0] op b[x-1:0]     |     ?(待测试)      |

### 2.缩位运算符

| 操作符 |         描述         |  使用方法  | 输出值 |
| :----: | :------------------: | :--------: | :----: |
|   &    | 向量所有位进行与运算 | &a[x-1:0]  | b[0:0] |
|   \|   | 向量所有位进行或运算 | \|a[x-1:0] | b[0:0] |

> 运算符不能作为向量取下标索引
>
> ```verilog
> a[3:0]	//合法
> b[3:0]	//合法
> (a + b)[3:0]	//不合法
> ```

### 3.赋值语句

> `variable = expression`
>
> ```verilog
> a = 32'b1;
> b = a;
> c = {a, b, a + b};
> {d, e} = {{16{a}}, b, c};
> ```
>
> - 等号左边变量可以是单个变量也可以是`{a, b}`位绑定，但不能带有常量或运算符
> - 等号右边表达式可以是单个变量或常量，也可以是`{a + b, 1'b1}`位绑定，但要注意位数
> - 若等号左右两边位数不同，则

# 三、电路语句

> 电路代码包括以下几部分：
>
> - 二进制信号声明
> - 数字元件
> - 电路连接

### 1. assign 语句

> assign 语句是一种电路语句，它可以描述一部分电路，这些电路起到运算符的功能
>
> assign 语句基本语法：`assgin 赋值语句;`(阻塞赋值)
>
> 适用于简单组合电路、已有电路图(进包含基本元件)或逻辑表达式的组合电路

```verilog
logic [31:0] a, b, c;
assign c = a & b;	//按位与操作
```

### 2. 元件例化语句

> 作用：层次化设计(功能区分)、代码复用、黑盒(框架)设计

##### 2.1 模块声明

```verilog
module module_nam(
    input logic xx,	//输入端口名称
    output logic xx	//输出端口名称
);
	//实现代码
endmodule
//---

module adder(
	//输入输出声明
    input logic [3:0] a, b,
    output logic [3:0] c
);
    assign c = a + b;	//电路语句
endmodule
```

##### 2.2 模块使用(元件例化)

> `adder adder_inst0(.a(4'b0010), .b, .*);`
>
> `adder`: 模块名
>
> `adder_inst0`: 元件名
>
> **`.`端口名`(value)`**: `value`可以为常量、变量和位绑定等，端口名必须和声明的一致，不能取其下标等，如`.a[0]()`
>
> `.`端口名: 模块内外端口一致则可以这样使用，即`.端口名1(端口名1)`，例如`.b`和`.b(b)`一致，括号中的`b`是模块外的`b`，括号外的`b`是模块的端口名
>
> `.*`: 剩下所有端口接模块外部同名信号且两者位宽最好一致

```verilog
logic [3:0] b, c;	//声明语句
adder adder_inst(.a(4'b0010), .b, .*);	//元件例化
//上一个语句相当于下面这个语句
assign c = b + 4'b0010;
```

### 3. always_comb 语句

> 描述复杂电路的组合逻辑，内部允许 if、case 等控制语句，注意不要出现锁存器，采用阻塞赋值
>
> 其内部描述电路行为，内部每条语句都是赋值语句(不能出现电路语句)，，整个 always_comb 语句块是电路语句
>
> 对于 always_comb 语句块，相当于一个模块，输出是等号左边的集合，输入是除去输出其他组成的集合
>
> 性质：
>
> - 内部覆盖性: 允许对同一个输出多次赋值
> - 对外原子性: 相当于外面的 assign 语句会针对 always_comb 语句最后结果进行赋值，即对于 always_comb 语句对外面的其他语句具有原子性，不管内部存在多少个语句，对外表现都像是只有一个语句(执行时间最小，不可拆分)，不知道这样理解合适不 😂
> - 阻塞赋值：可以理解为从上到下执行语句

```verilog
always_comb begin	//顺序(串行)执行
    a = 1'b1;
    b = a;
    a = 1'b0;
    c = a;
end
```

```verilog
assign a = b;	//把b信号接给a信号
always_comb begin
    b = 1'b1;
    c = a;	// c = b;
    b = 1'b0;
end
// 最后b=1'b0, a=1'b0, c=1'b0
//若把c = a;换成c = b;则最后b=1'b0, a=1'b0, c=1b'1
```

##### 3.1 always_comb 语句中的控制语句: case

> case 语句常用于描述选择器和译码器，case 最好要有 default 语句

- `unique case`

> unique 表示在一系列选项中有且仅有一项符合条件，如果存在多个，则会发出警告，选项之间属于并行关系

```verilog
always_comb begin
    unique case(a[3:0])
        4'd1:begin
            ...
        end
        4'd0:begin
            ...
        end
        default:begin
            ...
        end
    endcase
end
```

- `priority case`

> 具有优先级的 case，设计者认为存在多个 case 语句的值与表达式相匹配，并且条件选项的顺序十分重要，当不存在任意一项满足表达式的值时，仿真器会发出警告

```verilog
always_comb begin
    priority case(1'b1)
        a[3]:begin
            ...
        end
        a[2]:begin
            ...
        end
        default:begin
            ...
        end
    endcase
end
```

- 上面代码和下面代码功能一样

```verilog
always_comb begin
    if(a[3]) begin
        ...
    end else if(a[2]) begin
        ...
    end else if(a[1]) begin
        ...
    end
end
```

##### 3.2 always_comb 语句中的控制语句: if 和 for

> if 和 for 是 always_comb 中的常用语法，它们可以表示行为，也可以生成电路

- `if`

> if 和 else 用于条件判断

```verilog
always_comb begin
    if(a[3]) begin
        b = 1'b1;
    end else if(a[2]) begin
        b = 1'b0;
    end else begin
        b = 1'b0;
    end
end
```

- `for`

> for 在 always_comb 中被解释为循环展开，循环变量上下界都应该为常数
>
> for 可以与 break 和 continue 结合使用

```verilog
logic [15:0] a;
logic [3:0] b;

//integer i;
always_comb begin
    b = '0;
    //for循环中的int i可以表示为在外面使用integer i;(但是这里i是全局变量，这里需要注意)
    for(int i = 15;i >= 0;i--) begin
        if(a[i]) begin
            b = i[3:0]
            break;
        end
    end
end
```

```verilog
always_comb begin
    for(int i = 0;i < 16;i++) begin
        a[i] = b[i] & (c[i] == d[i] | e[i]);//编译器认为i不是常数，a[i:i+3]非法			(不可以使用a[i:i+3])
    end
end
//----
电路生成的for
for(genvar i = 0; i < 16;i++) begin
    assign a[i] = b[i] & (c[i] == d[i] | e[i]);//编译器认为i是常数，a[i:i+3]合法	(可以使用a[i:i+3])
    always_comb begin
        ...
    end
end
```

### 4. always_ff 语句

> `always_ff`语句用于描述触发器，采用非阻塞赋值
>
> 写代码时参考状态方程来写
>
> `posedge clk`必须有，其他不是必须的
>
> @中的语句是敏感信号表
>
> 阻塞赋值和非阻塞赋值区别在于先后顺序，阻塞赋值必须等完成上一个赋值语句后才能接着完成下一个赋值语句，上一个赋值操作会影响下一个赋值操作；而非阻塞赋值相当于同时完成，并行处理。

```verilog
//典型always_ff语句
//常用的只有posedge clk
always_ff @(posedge clk, negedge resetn) begin
    if(~resetn) begin	//resetn 低电平有效，优先考虑
        q <= '0;
    end else if(en) begin
        q <= d;
    end
end
```

```verilog
logic [3:0] a, a_nxt;

always_ff @(posedge clk) begin	//描述并列触发器
    if(~resetn) begin
        a <= '0;
    end else if(en) begin
        a <= a_nxt;
    end
end

always_comb begin	//描述组合逻辑行为
    a_nxt = a;
    unique case(a)
        4'd3:begin
            a_nxt = 4'd2;
        end
        ...
        default:begin
            ...
        end
    endcase
end
```

# 四、高级语法

### 1. 自定义类型语法：typedef

> 基本格式：`typedef 已有类型 新类型`

```verilog
typedef logic[31:0] word_t;
typedef logic[5:0] entry_t;
typedef logic[31:0] table_t;
typedef entry_t[31:0] test;	//logic [31:0][5:0]

word_t a, b;
assign b = {a[15:0], a[31:16]};
```

### 2. struct

> 描述一组相关的数据，一般结合`typedef`和`packed`使用
>
> 以译码器为例：

```verilog
//以前的写法
logic [3:0] alufunc;
logic mem_read, mem_write,regwrite;

logic [6:0] control;
assign control = {alufunc, mem_read, mem_read, mem_write, regwrite};

//下面是结构体类型的写法
typedef struct packed{
    logic [3:0] alufunc;
    logic mem_read;
    logic mem_write;
    logic regwrite;
} control_t;

control_t control;
logic regwrite;
assign regwrite = control.regwrite;//等同于control[0]

//不使用typedef的struct(匿名格式)
struct packed{
    logic [3:0] alufunc;
    logic mem_read;
    logic mem_write;
    logic regwrite;
} control_without_typedef;//control_without_typedef是变量名，不是新类型名
```

### 3. enum

> 常用于编码(包括状态机的编码)
>
> enum 类型变量在 vivado 仿真中会显示枚举项，枚举项被视为常量

```verilog
typedef enum <datatype>{
    IDEN_1,//常量 枚举值为0
    IDEN_2//枚举值为1
} typename;
```

```verilog
typedef enum logic [3:0]{
    ALU_ADD, ALU_ADN, ALU_SUB
} alufunc_t;

alufunc_t alufunc;

enum logic [3:0]{
    ALU_ADD, ALU_ADN, ALU_SUB
} alufunc_without_typedef;//这里类似于不使用typedef的struct(匿名格式)
```

```verilog
typedef enum logic [2:0]{
    STATE_0, STATE_1, STATE_2
} state_t;

state_t state, state_nxt;
always_ff @(posedge clk) begin
    if(~resetn) begin
        state <= state_t'(0);//赋值时强制转换为枚举项
    end else begin
        state <= state_nxt;
    end
end
```

### 4. union

> 对 union 类型变量进行赋值时需要注意多驱动(应该可以这样理解：相当于 union 中的变量都占用相同的地址空间)

```verilog
typedef union packed{
    struct packed{
        logic zero;
        logic [31:0] aluout;
    } alu;
    struct packed{
        logic zero;
        logic [31:0] pcbranch;
    } branch;
    struct packed{
        logic [31:0] addr;
        logic mem_read;
    } memory;
} result_t;

result_t res;

logic [31:0] addr, aluout;
assign addr = res.memory.addr;// assign addr = res[32:1]
assign aluout = res.alu.aluout;// assign aluout = res[31:0]
```

### 5. parameter

> 已有模块设计语法缺乏灵活性，为使模块代码具有更高的复用性，引入参数`parameter`
>
> `parameter`也可用于作用域内的全局常量声明，此时以分号结尾

```verilog
module adder #(
	parameter int N = 16,
    parameter logic [31:0] W = 32'd10000,
    parameter type element_t = logic [31:0]
)(
    input logic [N-1:0] a,b,
    output logic [N-1:0] c
);
    assign c = a + b;
    element_t x, y;//logic [31:0]
endmodule

module top #(
	parameter logic SIM = 1'b0//仿真标记，上板时设为0，仿真时设为1
)();
    logic [31:0] a, b, c;
    adder #(.N(32)) adder_inst(.a, .b, .c);//声明32位加法器 #(.N(32))

    logic [15:0] d, e, f;
    adder adder_inst2(.a(d), .b(e), .c(f));
endmodule

parameter logic[5:0] OP_ADDI = 6'b001011;
always_comb begin
    unique case(op)
        OP_ADDI: begin
            ...
        end
        ...
    endcase
end
```

### 6. 预编译命令

> systemverilog 中的预编译命令和 C 语言基本一致，但使用`(反引号)开头
>
> 预编译命令的作用：
>
> - 配置一些参数，类似于`parameter`语句
>
> - 根据不同参数生成不同电路，与`mux`不同
>
>   `generate if`语句粒度为电路语句，而`ifdef`之类的预编译命令可以是任意粒度的
>
>   ```verilog
>   assign a = b + c
>   	#ifdef D_INSIDE
>       + d
>   	#endif
>   ;
>
>   generate if(D_INSIDE) begin
>       assign a = b + c + d;
>   end else begin
>       assign a = b + c;
>   end
>   ```
>
> - 头文件类似于`package`语句
>
> - 使用宏封装一些功能，部分情况下可以使用`function`语句代替

```verilog
`include "mips.svh"	//sv头文件扩展名为.svh
// vivado软件把同一工程下所有文件视为同一目录
`ifndef __MIPS_SVH
`define __MIPS_SVH

`define LEN 0x10
logic a[`LEN-1:0];	//使用宏时也需要以`开头

`endif
```

### 7. 抽象接口 interface

> 元件例化语句存在的问题：
>
> - 元件例化语句不标明信号是输入还是输出
> - 信号相关模块的例化代码可能相隔很远
>
> 同时，不同模块可能复用一部分接口，添加一个接口需要修改多处代码
>
> `interface`语法可以解决以上问题，其语法形式与模块类似，`interface`可以多次例化
>
> `interface`声明放到头文件中可以大幅减少代码量
>
> ```verilog
> interface interface_name(/*input and output*/);
>     logic c;	//信号
>     //端口
>     modport modport_name1(input c);
>     modport modport_name2(output c);
> endinterface
>
> module module_name(
> 	interface_name.modport_name1 variable_name
> );
> ```
>
> `interface`里没有电路语句，只有信号(处理信号和信号间的连接关系)

```verilog
interface interface_name(input logic d, output logic e);
    logic c;	//信号
    //端口(包括输入输出信号)
    modport modport_name1(input c, d, output e);
    modport modport_name2(output c);
endinterface

module module_name(
	interface_name.modport_name1 variable_name
    // 相当于input logic c,d, output logic e
);
    logic d;
    assign d = variable_name.c;	//像struct一样使用variable_name
    assign variable_name.e = d;
endmodule

module top();
    logic a, b;
    interface_name intf_inst(.d(a), .e(b));	//interface例化(隐式声明信号)
    module_name instance_name(.variable_name(intf_inst.modport_name1));
endmodule
```
