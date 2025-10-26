参考：
[IC 研发常用英文术语缩写](https://mp.weixin.qq.com/s/z2_r7rY6eCO_ySgpF9HugQ)

### 软件篇

EDA Electronic Design Automation 电子设计自动化，IC 设计流程中需要使用非常多的 EDA 工具
VCS synopsys 公司的数字前端仿真工具
Verdi synopsys 公司的数字前端 debug 工具
FSDB 常用的波形文件格式，用 Verdi 打开
VCD value change dump 一个通用的波形文件格式，信息详细，但文件较大
DC design compiler synopsys 公司的数字综合工具
ICC IC Compiler synopsys 公司用于自动布局布线的一款软件，很多公司都在用
INNOVUS cadence 公司的数字版图实现工具、
GDSII 版图 layout 的文件格式
PT prime time synopsys 公司的静态时序分析工具
Vivado Vivado FPGA 厂商赛灵思公司 2012 年发布的集成设计环境
NCSIM cadence 公司的数字前端仿真工具
Modelsim mentor 公司的数字前端仿真工具，也叫 QUESTASIM
Tessent mentor 公司的 DFT 工具，市场占有率很高

### 语言篇

Shell 常用的一种脚本语言，和 linux 结合紧密
Python 常用的脚本语言，现在在人工智能方面使用很多，大受欢迎
Perl 常用的一种脚本语言，非常适合文本处理
TCL 工具命令语言，调度各个软件的脚本语言
Verilog 硬件描述语言
SystemVerilog 芯片验证语言

### 协议篇

APBAdvanced Peripheral BusARM 公司推出的 AMBA 总线规范之一，主要用于低带宽的外设之间的连接
AHB Advanced High-Performance Bus ARM 公司推出的 AMBA 总线规范之一，主要用于高性能模块之间的连接
AXI Advanced eXtensible Interface ARM 公司推出的 AMBA 总线规范之一，一种面向高性能、高带宽、低延迟的片内总线
GPIO General Purpose Input Output 通用输入/输出，总线扩展器
HDMI High Definition Multimedia Interface 高清晰度多媒体接口，是一种数字化视频/音频接口技术规范
SPI Serial Peripheral Interface 串行外设接口，是一种高速的，全双工，同步的通信总线
I2C Inter-Integrated Circuit I2C 是一种常用的多向控制总线，只有两根线
UART Universal Asynchronous Receiver/Transmitter 通用异步收发传输器，一种常见的 IP 模块
CANController Area NetworkISO 国际标准化的串行通信协议
MIPI Mobile Industry Processor Interface 移动产业处理器接口，为移动应用处理器制定的开放标准和一个规范
OCP Open Core Protocol 一个高效的、总线独立的、可配置和高度可扩展的接口协议
PCIe Peripheral Component Interconnect Express 外设组件互连标准，一种常见的总线标准
USB Universal Serial Bus 通用串行总线，一种高速的连接外设的总线协议

### 芯片篇

IC Integrated Circuit 集成电路
LSI Large-scale integrated circuit 大规模集成电路
VLSI Very-large-scale integration 超大规模集成电路
ASIC Application Special Integrated Circuit 专用集成电路，芯片设计公司的主流设计流程
FPGA Field Programmable Gate Array 现场可编程门阵列，与 ASIC 流程相对应
SoC System on Chip 片上系统，一般指规模比较大的芯片，大多含有 CPU/MCU 等
MCU Microcontroller unit 微控制器，主控模块
DSP Digital Signal Processing 数字信号处理模块， IC 设计公司的算法实现经常采用
CPLD Complex Programmable Logic Device 复杂可编程器件，和 FPGA 类似
IP Intellectual Property 知识产权
FE Front End 前端，IC 设计中的前端设计流程
DV Design Verification 验证，IC 设计中的验证流程
BE Back End 后端， IC 设计中的后端设计流程
FULLCHIP fullchip level 常用于数字前端设计和验证，指系统级和芯片级
GLS gate-level simulation 指数字验证中的门级仿真
LPS low power simulation 低功耗仿真，多用于低功耗设计验证中
FM formal 形式验证，网表与 verilog 进行比较
STA Static Timing Analysis 静态时序分析，数字 IC 设计流程中的重要环节
Netlist 门级网表，一般是 RTL Code 经过综合工具生成的网表文件
ECO Engineering Change Order 在项目后期，只能在门级对芯片设计进行修改
DFT Design for Test 为了增强芯片可测性而采用的一种设计方法，是数字 IC 流程中的重要步骤
ATPG Auto Test Pattern Generator 测试向量自动生成工具， DFT 中的常见流程
BIST Build in System Test 内建测试系统，DFT 中的常见流程
JTAG Joint Test Action Group 联合测试工作组，是一种国际标准测试协议，多用于芯片测试用
CTS Clock tree synthesis 时钟树综合，是数字后端实现中的重要流程
PD Physical design 物理设计，一般指数字后端的版图设计
PV Physical verification 物理验证，数字版图实现后需要做的验证
APR Auto place and route 自动布局布线，是数字后端版图实现的主要流程
NDR Non-Default Route 非默认连线规则，版图实现中的重要概念
Layout 版图，指芯片最终生成的版图，类似于建筑行业中的设计图纸
ERC Electronic Rule Check IC 设计经过 Layout 后检查其版图是否符合电气规则
LVS Layout versus Schematic 版图与电路图一致性检查，变成版图后检查其版图与门级电路是否一致
DRC Design Rule Check 生成版图后检查其是否符合工艺厂提供的设计规则,如宽度、间距、面积等
signoff 验收机制，验收标准
Tapout 流片，将最终的版图文件送到工艺厂去生产
DAC Digital to Analog Convert 数字信号到模拟信号的转换电路
ADC Analog to Digital Convert 模拟信号到数字信号的转换电路
CAD Computer-Aided Design 计算机辅助设计，专门帮助提供软件自动化
CDC clock domain crossing 异步时钟时序检查，是数字设计中的重要步骤
DMA Direct Memory Access 直接内存存取
RAM Random Access Memory 随机存储器
ROM Read Only Memory 只读存储器，具有非易失性
EEPROM Electrically Erasable Programmable Read-Only Memory 电可擦除只读存储器
DRAM Dynamic Random Access Memory 动态随机存取存储器,最为常见的系统内存
SRAM Static Random Access Memory 静态随机存取存储器
FLASH Flash EEPROM Memory 闪存，同时具有 RAM 快速读取数据的特点
LUT Look Up Table 查找表，用于存一些数据，本质就是一个 RAM
IEEE Institute of Electrical and Electronics Engineers 电气和电子工程师协会
SPEC specification 说明书，产品规范，每个岗位工程师都要写相应的 spec
RTL Register Transformation Level 寄存器传输级，多指使用 verilog 来描述的层次
DUT design under test 待测试的设计模块
DUV design under verification 和 DUT 的意思类似
Testbench 测试平台，数字验证搭建用来测试的平台
UVM Universal Verification Methodology 主流的数字验证方法学，基于 systemverilog
REGRESSION 回归测试，简单来说就是讲所有的测试用例不断的重复的跑，直到没有错误稳定一段时间
COVERAGE 覆盖率，数字验证常用术语，主要有代码覆盖率和功能覆盖率等
