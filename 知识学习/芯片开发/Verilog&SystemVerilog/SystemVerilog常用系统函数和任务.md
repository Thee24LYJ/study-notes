[SystemVerilog 的系统函数和任务](https://mp.weixin.qq.com/s/mk8nMXVTFTbAnjIsmdCozg)
[Systemverilog 系统函数-仿真时间](https://zhuanlan.zhihu.com/p/459171644)
[Verilog 的时间系统任务----\$time、\$stime、\$realtime-CSDN 博客](https://blog.csdn.net/wuzhikaidetb/article/details/125992226)
[SystemVerilog 中的 time、stime、realtime 的一些事儿](https://mp.weixin.qq.com/s/xae1KqTFDeF6_xdPZrtfYg)
[Time and Realtime - SystemVerilog Tutorial - Verification Studio](https://learn.verificationstudio.com/tutorials/1/systemverilog-tutorial/subcontents/8/time-and-realtime)
"Verilog 和 systemVerilo..."点击查看元宝的回答
https://yuanbao.tencent.com/bot/app/share/chat/STpeda5Xj63p

**1. Simulation control system tasks**

控制仿真的系统任务有三个：

- \$stop \[ ( n ) ]：\$stop 可以用于暂停仿真；
- \$finish \[ ( n ) ]：\$finish 会使仿真器退出并将控制权传递回主机操作系统；
- \$exit \[ ( ) ]：\$exit 会等待所有程序块(program block)完成，然后隐式调用\$finish；

\$stop 和\$finish 允许接收一个可选的参数（0,、1 或 3），该参数决定打印什么类型的诊断消息，如果没有提供参数，默认参数值为 1，如下表所示：

**2. Simulation time system functions**

提供对当前时间访问的系统函数有：

三个系统函数的返回值=当前仿真时间/当前作用域的 time unit

- \$time：返回一个 64-bit 时间的整数，按比例缩放(四舍五入)为调用它的模块的时间单位(time unit)；
- \$stime：返回一个 32-bit 时间的无符号整数，按调用它的模块的时间单位进行缩放。如果实际仿真时间不适合 32-bit，则返回当前仿真时间的低 32-bit；
- \$realtime：返回一个实数时间，与\$time 一样，它被缩放为调用它的模块的时间单位；
  此外可以使用\$printtimescale（hier）函数打印当前某个层次模块的时间单位和精度信息。如果\$printtimescale 传入的参数为空，则打印当前 scope 的时间设置信息

**3. Timescale system tasks**

显示和设置时间打印信息的系统任务有：

- \$printtimescale \[ ( hierarchical_identifier ) ]：用于显示特定模块的时间单位和时间精度；该系统任务可以指定或不指定参数：
  - 如果没有指定参数，它将显示当前作用域模块的时间单位和精度；
  - 如果指定参数，它将显示传递给它的模块的时间单位和精度；
- \$printtimescale 打印的 timescale 格式为：Time scale of (module_name) is unit / precision；
- \$timeformat \[ ( units_number , precision_number , suffix_string , minimum_field_width ) ]：它有以下两个功能：

  - 它指定了在\$write, \$display, \$strobe, \$monitor, \$fwrite, \$fdisplay, \$fstrobe 和\$fmonitor 中，用%t 打印时间信息的格式规范，直到另一个\$timeformat 被调用；
  - 它指定了进入交互方式延迟的时间单位；

- \$timeformat 中 units_number 参数应为 0~-15 范围内的整数，该参数不同取值表示的时间单位如下表所示：
  ![](https://mmbiz.qpic.cn/mmbiz_png/g0xxPdqzWAEM1m7BntVjWhJaNG7pRDFNwLHh36UO54pVZMeuAHOz9iaGy9Jgo0MnY3NPrs3zlXYjb3LwMSQYoVA/640?wx_fmt=png&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)
  precision_number 用于指定打印时间精度为多少，默认值为 0；suffix_string 用于指定时间后面跟的字符串，默认为空字符串；minimum_field_width 为时间格式最少占用的宽度。

**4. Conversion functions**

提供了系统函数在值和实数值之间进行转换，并将值转换为有符号值或无符号值。

以下系统函数用于处理实数值(real 和 shortreal 类型)，它们也可以用于处理常数表达式：

- Integer \$rtoi ( real_val )：\$rtoi 通过截断实数值将实数值转换为整数类型，比如 123.45 变成 123。这里要记住：\$rtoi 是直接截断小数点后面的值，而不会四舍五入的，像直接赋值或使用 cast 转换的场景，它们是会使用四舍五入的；
- Real \$itor ( int_val )：\$itor 将整数值转换为实数值，比如 123 变成 123.0；
- \[63:0] \$realtobits ( real_val )：\$realtobits 将 real 类型的值转换为实数的 64-bit 向量表示；
- Real \$bitstoreal ( bit_val )：\$bitstoreal 将\$realtobits 创建的 64-bit 向量模式转换为实数值；
- \[31:0] \$shortrealtobits ( shortreal_val )：\$shortrealtobits 将值从 shortreal 类型转换为实数的 32-bit 向量表示；
- Shortreal \$bitstoshortreal ( bit_val )：\$bitstoshortreal 将\$shortrealtobits 创建的 32-bit 向量模式转换为 shortreal 类型的值；

\$signed 和\$unsigned 系统函数可用于 强制转换表达式的符号，而不是类型。这些函数对输入表达式求值，并返回与输入表达式具有相同大小的值和函数定义的类型。 - \$signed：返回的值是有符号； - \$unsigned：返回的值是无符号；

需要说明的是：cast 也可以用于转换表达式的符号位，而是采用动态的 cast 改变表达式符号类型。

**5. Data query functions**

提供了查询表达式的一些系统函数，有：

- \$typename ( expression )或\$typename ( data_type )：\$typename 返回一个字符串，该字符串表示其参数的解析类型；
- \$bits ( expression ) 或\$bits ( data_type )：\$bits 返回参数的需要占据的位数，返回类型为 integer，4-state 值计为 1-bit；
- \$isunbounded ( constant_expression )：如果参数为\$，\$isunbounded 将返回 true；

**6. Array querying functions**

SV 提供了用于返回有关数组或整型数据类型或一个数据类型的数据对象的特定维度的信息，返回默认类型为 integer，可选维度表达式的默认值为 1。维度表达式可以指定任何固定大小的维度(packed 或 unpacked)或任何动态大小的维度(动态、关联或队列)。当用于动态数组或队列维度时，这些函数会返回有关数组当前状态的信息。对于除关联数组维度之外的任何维度：

- \$left：\$left 返回所选维度的左边界。对于 packed 维度，它会返回 most significant element 的索引；对于队列或动态数组维度，\$left 返回 0；
- \$right：\$right 返回所选维度的右边界。对于 packed 维度，它会返回 least significant element 的索引；对于当前大小为 0 的队列或动态数组维度，\$right 返回-1；
- \$increment：对于固定大小的维度，如果\$left 大于等于\$right，则\$increment 返回 1；如果\$left 小于\$right，则返回-1。对于队列或动态数组维度，\$increment 将返回-1；
- \$low：如果\$increment 返回-1，则\$low 将返回与\$left 相同的值；当\$increment 返回 1 时，\$low 返回与\$right 相同的值；
- \$high：如果\$increment 返回-1，则\$high 将返回与\$right 相同的值；当\$increment 返回 1 时，\$high 将返回与\$left 相同的值；
- \$size：返回所选维度中元素的个数，相当于\$high-\$low+1；
- \$dimensions：返回数组的总维度(packed 和 unpacked，动态或静态)。如果是 string 数据类型或任何其它等效于简单 bit vector 类型的非数组类型，则返回 1；其它类型返回 0。
- \$unpacked_dimensions：返回数组的总 unpacked 维度(动态或静态)；其它类型返回 0。

数组的维度按照如下方式编号：变化最慢的维度(packed 或 unpacked)为维度 1,。变化速度越快，维度越高。在对维度进行编号之前，首先展开中间类型的定义。

Array querying functions 在关联数组维度上的使用仅限于具有整数值的索引类型。如果使用整数索引，这些函数将返回以下内容：

- \$left 返回 0；
- \$right 返回最大可能的索引值；
- \$low 返回当前分配的最低索引值，但如果当前没有分配元素，则返回’x；
- \$hight 返回当前分配的最大索引值，但如果当前没有分配元素，则返回’x；
- \$increment 返回-1；
- \$size 返回当前分配的元素数量；

**7. Math functions**

SV 提供了整数算术函数和实数算术函数，这些算术系统函数可以在常量表达式中使用。

在整数算术函数中，\$clog2 系统函数将返回参数以 2 为基数的 log 上限(向上舍入为整数值)。参数可以是整数或任意大小的向量值。参数会被视为无符号数，如果参数为 0，那么结果为 0。\$clog2 可用于计算寻址戈丁大小的存储器所需的最小地址宽度，或表示给定数量的状态所需的最小位宽。

在实数算术函数中，传入的参数必须是实数值参数，并返回实数类型的结果。它们的行为和 C 语言标准库中对应函数的功能一致，如下表所示。

![](https://mmbiz.qpic.cn/mmbiz_png/g0xxPdqzWAEM1m7BntVjWhJaNG7pRDFNMkme2uMou5RWiaNlWLbAZKsdoI8jGYSTdGiciaicduQuWBUbZYiar2deotg/640?wx_fmt=png&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

**8. Bit vector system functions**

位向量系统函数可以对位向量进行操作，有如下：

- \$countbits ( expression , control_bit { , control_bit } )：\$countbits 计算位向量中一组特定值(例如：0、1、X、Z)的位的个数，它的参数 control_bit 可能是常数和变量都行，它的返回值是 int 类型，该值等于 expression 与 control_bit 相匹配的位数；比如：
  - \$countbits (expression, '1)返回 expression 中 bit 值为 1 的个数；
  - \$countbits (expression, '1, '0)返回 expression 中 bit 值为 1 或 0 的个数；
  - \$countbits (expression, 'x, 'z)返回 expression 中 bit 值为 x 或 z 的个数；
- \$countones ( expression )：等价于\$countbits(expression,'1)，返回类型是 int；
- \$onehot ( expression )：等价于\$countbits(expression,'1)\==1，返回类型是 bit；
- \$onehot0 ( expression )：等价于\$countbits(expression,'1)<=1，返回类型是 bit；
- \$isunknown ( expression )：等价于\$countbits(expression,'x,'z)!=0，返回类型是 bit；

**9. Severity tasks**

SV 提供了专门的文本消息处理任务，可用于标识各种异常情况。这些任务有：

- \$fatal \[ ( finish_number(0 | 1 | 2) \[ , list_of_arguments ] ) ]：\$fatal 将引发一个 run-time 致命错误，从而终止仿真并以错误代码结束；传递给\$fatal 函数的第一个参数与传递给\$finish 的参数含义一致，该参数用于设置工具所报告诊断信息级别。调用\$fatal 函数等同于隐式调用\$finish 函数。
- \$error \[ ( \[ list_of_arguments ] ) ]：\$error 用于上报一个运行错误；
- \$warning \[ ( \[ list_of_arguments ] ) ]：\$warning 用于上报一个运行警告；
- \$info \[ ( \[ list_of_arguments ] ) ]：\$info 表示消息没有特定的严重性；

**10. Coverage system functions**

SV 有几个内建的系统任务用于获取或设置覆盖率信息，有如下：

- \$coverage_control(control_constant, coverage_type, scope_def, modules_or_instance)：该函数用于控制或查询层次结构中指定部分的覆盖率是否可用；
- \$coverage_get_max(coverage_type, scope_def, modules_or_instance)：该函数获取在层次结构的指定部分上指定覆盖率类型的 100%覆盖率的值。在整个仿真过程中，它的值保持不变。该函数返回的值是 integer 类型，它的值与电路设计尺寸和结构成正比，在假设任何编译选项都不会修改覆盖支持或设计结构的情况下，理论上在多个独立的仿真和编译后，对于相同的设计，它的值保持不变；
- \$coverage_get(coverage_type, scope_def, modules_or_instance)：该函数在层次结构的给定部分上获得给定覆盖率类型的当前覆盖值。该函数的返回值是 integer 类型。这个数字可以通过公式(coverage% =(coverage_get() / coverage_get_max()) x 100)转换为覆盖率百分比；
- \$coverage_merge(coverage_type, "name")：该函数将指定覆盖率的数据加载并合并到仿真器中。”name”是工具使用的任意字符串，以特定于工具实现的方式。只要能定位到工具允许覆盖率存放的地方就行。如果用户通过该函数去检索名称不存在或覆盖率数据库不对应，就会报错。如果在加载过程中发生错误，那么该仿真生成的覆盖率数字可能没有意义了；
- \$coverage_save(coverage_type, "name")：该函数将覆盖率的当前状态保存到工具的覆盖率数据库中，并将其与给定名称相关联。该名称将以特定于实现的方式映射到覆盖率数据库中的某个文件或一组文件中。保存到数据库中的数据可以通过使用\$coverage_merge()用相同的名称检索到；在仿真中，保存覆盖率不会对覆盖率状态产生任何影响；
- \$set_coverage_db_name ( filename )：设置覆盖率数据库的文件名，在仿真运行结束时将覆盖率信息保存到该数据库中；
- \$load_coverage_db ( filename )：从给定的文件名加载所有 coverage group 类型的累积覆盖率信息；
- \$get_coverage：返回 0~100 范围内所有 coverage group 类型的总体覆盖率的实数值；

**11. Probabilistic distribution functions**

SV 提供一组标准概率分布函数用于返回整数值，有如下：

- \$random \[ ( seed ) ]：\$random 提供了一种生成随机数的机制。该函数每次被调用时都会返回一个新的 32-bit 有符号随机数。Seed 参数控制\$random 返回的数字，以便不同的种子生成不同的随机流。Seed 参数应该是一个整型变量，且在调用\$random 之前将种子值赋给这个变量；
- dist function：dist 系统函数的所有参数都是整数值。对于指数函数(exponential)、泊松函数(poisson)、卡方函数(chi-square)、t 函数和 erlang 函数来说，它们的参数 mean、degree_of_freedom 和 k_stage 要大于 0；对于每个系统函数，seed 参数都是 inout 参数；也就是说，将一个值传递给函数，然后返回一个不同的值。对于相同的种子，系统函数应该总是返回相同的值。这使得系统的操作可重复，从而便于调试。seed 参数应该是一个整型变量，由用户初始化，仅由系统函数更新，以便实现期望的分布。在\$dist_normal、\$dist_exponential、\$dist_poisson 和\$dist_erlang 中指定的整型输入 mean 参数，它使函数返回的平均值接近指定的 mean 值。与\$dist_chi_square 和\$dist_t 函数一起使用的 degree_of_freedom 参数是一个整型输入，有助于确定密度功能的形状，较大的数字将返回值分布在更大的范围内。
  - \$dist_uniform ( seed , start , end )：在\$dist_uniform 函数中，start 和 end 参数是绑定返回值的整数输入。\$dist_uniform 返回在其参数指定的间隔内均匀分布的随机数。起始值应小于结束值；
  - \$dist_normal ( seed , mean , standard_deviation )：在\$dist_normal 中使用的 standard_deviation 参数是一个整型输入，它有助于确定正态分布的密度形状，较大的 standard_deviation 值将返回值扩展到更大的范围；
  - \$dist_exponential ( seed , mean )
  - \$dist_poisson ( seed , mean )
  - \$dist_chi_square ( seed , degree_of_freedom )
  - \$dist_t ( seed , degree_of_freedom )
  - \$dist_erlang ( seed , k_stage , mean )

**12. Miscellaneous tasks and functions**

\$system ( \[ " terminal_command_line " ] )会调用 C 语言的函数 system()，该 C 函数执行传递给它的参数，就像从终端执行参数一样。\$system 可以作为任务或函数调用。当它作为函数调用时，它调用它返回数据类型为 int。如果调用\$system 时没有带字符串参数，则调用 c 函数 system()时将带 NULL 字符串
