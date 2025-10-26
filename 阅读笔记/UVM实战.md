# 第一章 与 UVM 的第一次接触

### 验证在现代 IC 流程中的位置

![现代IC流程](https://s2.loli.net/2025/07/12/vclSYzQLqhMHCPF.jpg)

从需求说明书开始，IC 工程师会把它们细化为特性列表。设计工程师根据特性列表，将其转化为设计规格说明书，在这份说明书中，设计工程师会详细阐述自己的设计方案，描述清楚接口时序信号，使用多少 RAM 资源，如何进行异常处理等。验证工程师根据特性列表，写出验证规格说明书。在验证规格说明书中，将会说明如何搭建验证平台，如何保证验证完备性，如何测试每一条特性，如何测试异常的情况等

验证主要保证从特性列表到 RTL 转变的正确性，包括但不限于以下几点：

- DUT(Design Under Test)的行为表现是否与特性列表中要求的一致
- DUT 是否实现了所有特性列表中列出的特性
- DUT 对于异常状况的反应是否与特性列表和设计规格说明书中的一致，如中断是否置起
- DUT 是否足够稳健，能够从异常状态中恢复到正常的工作模式

SystemVerilog 是一门优秀的语言，但仅仅使用它来进行验证显然不够，有很多直接的问题需要考虑，例如：

- 验证平台中都有哪些基本的组件，每个组件的行为有哪些？
- 验证平台中各个组件之间是如何通信的？
- 验证中要组建很多测试用例，这些测试用例如何建立、组织的？
- 在建立测试用例的过程中，哪些组件是变的，哪些组件是不变的？

同时，也有一些更高层次的问题需要考虑：

- 验证平台中数据流与控制流如何分离？
- 验证平台中的寄存器方案如何解决?验证平台如何保证是可重用的?

# 第二章 一个简单的 UVM 验证平台

### 2.1 验证平台的组成

验证用于找出 DUT 中的 bug，这个过程通常是把 DUT 放入一个验证平台中来实现的，验证平台的基本功能如下：

- 验证平台要模拟 DUT 的各种真实使用情况，这意味着要给 DUT 施加各种激励，有正常的激励，也有异常的激励；有这种模式的激励，也有那种模式的激励。激励的功能是由 driver 来实现的
- 验证平台要收集 DUT 的输出并把它们传递给 scoreboard，完成这个功能的是 monitor
- 验证平台要能够给出预期结果。在记分板中提到了判断的标准，判断的标准通常就是预期，完成这个过程的是参考模型（reference model）

UVM 中验证平台的典型框图：
![](https://s2.loli.net/2025/07/13/cJwuGAeIDldSXkp.jpg)

### 2.2 只有 driver 的验证平台

- 使用 UVM 的第一条原则是：验证平台中所有的组件应该派生自 UVM 中的类
- 所有派生自 uvm_component 及其派生类的类都应该使用 uvm_component_utils 宏注册
- SystemVerilog 中使用 interface 来连接验证平台与 DUT 的端口

UVM 验证平台中的 driver 应该派生自 uvm_driver(uvm_driver 是一个派生自 uvm_component 的类)，所有派生自 uvm_driver 的类的 new 函数有两个参数，一个是 string 类型的 name，一个是 uvm_component 类型的 parent
driver 所做的事情几乎都在 main_phase 中完成。UVM 由 phase 来管理验证平台的运行，这些 phase 统一以 xxxx_phase 来命名，且都有一个类型为 uvm_phase、名字为 phase 的参数。main_phase 是 uvm_driver 中预先定义好的一个任务。因此几乎可以简单地认为，实现一个 driver 等于实现其 main_phase

==自动创建一个类的实例并调用其中的函数（function）和任务（task）==。要使用这个功能，需要引入 UVM 的 factory 机制，factory 机制的实现被集成在了一个宏中：uvm_component_utils；然后在 top_tb.sv 中使用 run_test 语句来创建一个 my_driver 的实例，并且自动调用 my_driver 的 main_phase

在每个 phase 中，UVM 会检查是否有 objection 被提起（raise_objection），如果有，那么等待这个 objection 被撤销（drop_objection）后停止仿真；如果没有，则马上结束当前 phase；读者可以简单地将 drop_objection 语句当成是 finish 函数的替代者，只是在 drop_objection 语句之前必须先调用 raise_objection 语句，raise_objection 和 drop_objection 总是成对出现

避免绝对路径的使用：

- 使用宏定义
- 使用 interface

对于 run_test 语句这种脱离了 top_tb 层次结构，同时又期望在 top_tb 中对其进行某些操作的实例，UVM 引进了 config_db 机制。在 config_db 机制中，分为 set 和 get 两步操作。所谓 set 操作，读者可以简单地理解成是“寄信”，而 get 则相当于是“收信”

与 main_phase 一样，build_phase 也是 UVM 中内建的一个 phase。当 UVM 启动后，会自动执行 build_phase。build_phase 在 new 函数之后 main\_ phase 之前执行。在 build_phase 中主要通过 config_db 的 set 和 get 操作来传递一些数据，以及实例化成员变量等；build_phase 是一个函数 phase，而 main_phase 是一个任务 phase，build_phase 是不消耗仿真时间的

### 2.3 为验证平台加入各个组件

post_randomize 是 SystemVerilog 中提供的一个函数，当某个类的实例的 randomize 函数被调用后，post_randomize 会紧随其后无条件地被调用

##### 2.3.1 加入 transaction

reference model、monitor、scoreboard 等验证平台的其他组件之间，信息的传递是基于 transaction 的

- 所有自定义的 transaction 派生类的基类是 uvm_sequence_item，由此才能使用 UVM 强大的 sequence 机制
- 不使用 uvm_component_utils 宏来实现 factory 机制，而是使用了 uvm_object_utils。从本质上来说，my_transaction 与 my_driver 是有区别的，在整个仿真期间，my_driver 是一直存在的，my_transaction 不同，它有生命周期。它在仿真的某一时间产生，经过 driver 驱动，再经过 reference model 处理，最终由 scoreboard 比较完成后，其生命周期就结束了。一般来说，这种类都是派生自 uvm_object 或者 uvm_object 的派生类，uvm_sequence_item 的祖先就是 uvm_object。UVM 中具有这种特征的类都要使用 uvm_object_utils 宏来实现

##### 2.3.2 加入 env

为解决验证平台各个组件的实例化问题，引入一个容器类，在这个容器类中实例化 driver、monitor、reference model 和 scoreboard 等。在调用 run_test 时，传递的参数不再是 my_driver，而是这个容器类，即让 UVM 自动创建这个容器类的实例。在 UVM 中，这个容器类称为 uvm_env，所有自定义 env 都派生自 uvm_env

与 my_driver 一样，容器类在仿真中也是一直存在的，使用 uvm_component_utils 宏来实现 factory 的注册

验证平台中的组件在实例化时都应该使用 type_name::type_id::create 的方式。这种方式就是 factory 机制带来的独特的实例化方式。只有使用 factory 机制注册过的类才能使用这种方式实例化；只有使用这种方式实例化的实例，才能使用后文要讲述的 factory 机制中最为强大的重载功能

通过 parent 的形式，UVM 建立起了树形的组织结构。在这种树形的组织结构中，由 run_test 创建的实例是树根（这里是 my_env），并且树根的名字是固定的，为 uvm_test_top，这在前文中已经讲述过；在树根之后会生长出枝叶（这里只有 my_driver），长出枝叶的过程需要在 my_env 的 build_phase 中手动实现

在 UVM 的树形结构中，build_phase 的执行遵照从树根到树叶的顺序，即先执行 my_env 的 build_phase，再执行 my_driver 的 build_phase。当把整棵树的 build_phase 都执行完毕后，再执行后面的 phase

##### 2.3.3 加入 monitor

验证平台中实现监测 DUT 行为的组件是 monitor。driver 负责把 transaction 级别的数据转变成 DUT 的端口级别，并驱动给 DUT，monitor 的行为与其相对，用于收集 DUT 的端口数据，并将其转换成 transaction 交给后续的组件如 reference model、scoreboard 等处理

注意：

- 所有的 monitor 类应该派生自 uvm_monitor
- 与 driver 类似，在 my_monitor 中也需要有一个 virtual my_if
- uvm_monitor 在整个仿真中是一直存在的，所以它是一个 component，要使用 uvm_component_utils 宏注册
- 由于 monitor 需要时刻收集数据，永不停歇，所以在 main_phase 中使用 while(1)循环来实现

##### 2.3.4 封装成 agent

driver 和 monitor 之间的联系：两者之间的代码高度相似。其本质是因为二者处理的是同一种协议，在同样一套既定的规则下做着不同的事情。由于二者的这种相似性，UVM 中通常将二者封装在一起，成为一个 agent。因此，不同的 agent 就代表了不同的协议

所有的 agent 都要派生自 uvm_agent 类，且其本身是一个 component，应该使用 uvm_component_utils 宏来实现 factory 注册；build_phase 中根据 uvm_agent 的成员变量 is_active 的值来决定是否创建 driver 的实例(在输出端口上不需要驱动任何信号，只需要监测信号。在这种情况下，端口上是只需要 monitor 的，所以 driver 可以不用实例化)

完成输入端口和输出端口 i_agt 和 o_agt 的声明后，在 my_env 的 build_phase 中对它们进行实例化后，需要指定各自的工作模式是 active 模式还是 passive 模式。UVM 中约定俗成的还是在 build_phase 中完成实例化工作

##### 2.3.5 加入 reference model

reference model 用于完成和 DUT 相同的功能。reference model 的输出被 scoreboard 接收，用于和 DUT 的输出相比较

在 UVM 中，通常使用 TLM（Transaction Level Modeling）实现 component 之间 transaction 级别的通信。在 UVM 的 transaction 级别的通信中，数据的发送有多种方式，其中一种是使用 uvm_analysis_port；UVM 的 transaction 级别通信的数据接收方式也有多种，其中一种就是使用 uvm\_ blocking_get_port

在 my_monitor 和 my_model 中定义并实现了各自的端口之后，通信的功能并没有实现，还需要在 my_env 中使用 fifo 将两个端口联系在一起，由于 analysis_port 是非阻塞性质的，ap.write 函数调用完成后马上返回，不会等待数据被接收。假如当 write 函数调用时，blocking_get_port 正在忙于其他事情，而没有准备好接收新的数据时，此时被 write 函数写入的 my_transaction 就需要一个暂存的位置，这就需要使用 fifo，fifo 的类型是 uvm_tlm_analysis_fifo，它本身也是一个参数化的类，其参数是存储在其中的 transaction 的类型

与 my_monitor 中的 uvm_analysis_port 句柄 ap 不同的是，不需要对 my_agent 中的 uvm_analysis_port 句柄 ap 进行实例化，而只需要在 my_agent 的 connect_phase 中将 monitor 的值赋给它，相当于 my_agent 中的 ap 是一个指向 my_monitor 的 ap 的指针

与 build_phase 及 main_phase 类似，connect_phase 也是 UVM 内建的一个 phase，它在 build_phase 执行完成之后马上执行。但是与 build_phase 不同的是，它的执行顺序并不是从树根到树叶，而是从树叶到树根——先执行 driver 和 monitor 的 connect_phase，再执行 agent 的 connect_phase，最后执行 env 的 connect_phase

##### 2.3.6 加入 scoreboard

my_scoreboard 要比较的数据一是来源于 reference model，二是来源于 o_agt 的 monitor。前者通过 exp_port 获取，而后者通过
act_port 获取。在 main_phase 中通过 fork 建立起了两个进程，一个进程处理 exp_port 的数据，当收到数据后，把数据放入
expect_queue 中；另外一个进程处理 act_port 的数据，这是 DUT 的输出数据，当收集到这些数据后，从 expect_queue 中弹出之前从
exp_port 收到的数据，并调用 my_transaction 的 my_compare 函数。采用这种比较处理方式的前提是 exp_port 要比 act_port 先收到数
据。由于 DUT 处理数据需要延时，而 reference model 是基于高级语言的处理，一般不需要延时，因此可以保证 exp_port 的数据在
act_port 的数据之前到来

##### 2.3.7 加入 field_automation 机制

调用 pack_bytes 将 tr 中所有的字段变成 byte 流放入 data_q 中，在 2.3.1 节中是手工地将所有字段放入 data_q 中的。pack_bytes 极大地减少了代码量。在把所有的字段变成 byte 流放入 data_q 中时，字段按照 uvm_field 系列宏书写的顺序排列。在上述代码中是先放入 dmac，再依次放入 smac、ether_type、pload、crc

使用 unpack_bytes 函数将 data_q 中的 byte 流转换成 tr 中的各个字段。unpack\_ bytes 函数的输入参数必须是一个动态数组，所以需要先把收集到的、放在 data_q 中的数据复制到一个动态数组中。由于 tr 中的 pload 是一个动态数组，所以需要在调用 unpack_bytes 之前指定其大小，这样 unpack_bytes 函数才能正常工作

### 2.4 sequence

##### 2.4.1 在验证平台中加入 sequencer

sequence 机制用于产生激励，它是 UVM 中最重要的机制之一。在一个规范化的 UVM 验证平台中，driver 只负责驱动 transaction，而不负责产生 transaction。sequence 机制有两大组成部分，一是 sequence，二是 sequencer

sequencer 的定义非常简单，派生自 uvm_sequencer，并且使用 uvm_component_utils 宏来注册到 factory 中。uvm_sequencer 是一个参数化的类，其参数是 my_transaction，即此 sequencer 产生的 transaction 的类型(定义 driver 时也需指定需要驱动的 transaction)

==**sequencer 产生 transaction，而 driver 负责接收 transaction**==

##### 2.4.2 sequence 机制

从本质上来说，sequencer 是一个 uvm_component，而 sequence 是一个 uvm_object。与 my_transaction 一样，sequence 也有其生命周期。它的生命周期比 my_transaction 要更长一些，其内的 transaction 全部发送完毕后，它的生命周期也就结束了；sequence 应该使用 uvm_object_utils 宏注册到 factory 中

每一个 sequence 都应该派生自 uvm_sequence，并且在定义时指定要产生的 transaction 的类型，这里是 my_transaction。每一个 sequence 都有一个 body 任务，当一个 sequence 启动之后，会自动执行 body 中的代码

uvm_do 个宏是 UVM 中最常用的宏之一，它用于：

- 创建一个 my_transaction 的实例 m_trans
- 将其随机化
- 最终将其送给 sequencer

一个 sequence 在向 sequencer 发送 transaction 前，要先向 sequencer 发送一个请求，sequencer 把这个请求放在一个仲裁队列中。作
为 sequencer，它需做两件事情：第一，检测仲裁队列里是否有某个 sequence 发送 transaction 的请求；第二，检测 driver 是否申请
transaction。

- 1\)如果仲裁队列里有发送请求，但是 driver 没有申请 transaction，那么 sequencer 将会一直处于等待 driver 的状态，直到 driver 申
  请新的 transaction。此时，sequencer 同意 sequence 的发送请求，sequence 在得到 sequencer 的批准后，产生出一个 transaction 并交给
  sequencer，后者把这个 transaction 交给 driver
- 2\)如果仲裁队列中没有发送请求，但是 driver 向 sequencer 申请新的 transaction，那么 sequencer 将会处于等待 sequence 的状态，
  一直到有 sequence 递交发送请求，sequencer 马上同意这个请求，sequence 产生 transaction 并交给 sequencer，最终 driver 获得这个 transaction
- 3\)如果仲裁队列中有发送请求，同时 driver 也在向 sequencer 申请新的 transaction，那么将会同意发送请求，sequence 产生
  transaction 并交给 sequencer，最终 driver 获得这个 transaction

driver 如何向 sequencer 申请 transaction 呢？在 uvm_driver 中有成员变量 seq_item_port，而在 uvm_sequencer 中有成员变量
seq_item_export，这两者之间可以建立一个“通道”，通道中传递的 transaction 类型就是定义 my_sequencer 和 my_driver 时指定的
transaction 类型,在 my_agent 中使用 connect 函数把两者联系在一起；当把二者连接好之后，就可以在 driver 中通过 get_next_item 任务向 sequencer 申请新的 transaction

通过 get_next_item 任务来得到一个新的 req，并且驱动它，驱动完成后调用 item_done 通知 sequencer。这里为什么会有一个
item_done 呢？当 driver 使用 get_next_item 得到一个 transaction 时，sequencer 自己也保留一份刚刚发送出的 transaction。当出现
sequencer 发出了 transaction，而 driver 并没有得到的情况时，sequencer 会把保留的这份 transaction 再发送出去。那么 sequencer 如何知道 driver 是否已经成功得到 transaction 呢？如果在下次调用 get_next_item 前，item_done 被调用，那么 sequencer 就认为 driver 已经得到了这个 transaction，将会把这个 transaction 删除。换言之，这其实是一种为了增加可靠性而使用的握手机制

uvm_do 宏产生了一个 transaction 并交给 sequencer，driver 取走这个 transaction 后，uvm_do 并不会立刻返回执行下一次的 uvm_do 宏，而是等待在那里，直到 driver 返回 item_done 信号。此时，uvm_do 宏才算是执行完毕，返回后开始执行下一个 uvm_do，并产生新的 transaction

sequence 如何向 sequencer 中送出 transaction 呢？前面已经定义了 sequence，只需要在某个 component（如 my_sequencer、my_env）的 main_phase 中启动这个 sequence 即可

除了 get_next_item 外，还有 try_next_item 来获取 transaction；get_next_item 是阻塞的，它会一直等到有新的 transaction 才会返回；try_next_item 则是非阻塞的，它尝试着询问 sequencer 是否有新的 transaction，如果有，则得到此 transaction，否则就直接返回

##### 2.4.3 default_sequence 的使用

在实际应用中，使用最多的还是通过 default_sequence 的方式启动 sequence；使用 default_sequence 的方式非常简单，只需要在某个 component（如 my_env）的 build_phase 中设置如下代码即可

    uvm_config_db#(uvm_object_wrapper)::set(this,
        "i_agt.sqr.main_phase",
        "default_sequence",
        my_sequence::type_id::get());

### 2.5 建造测试用例

在一个实际应用的 UVM 验证平台中，my_env 并不是树根，通常来说，树根是一个基于 uvm_test 派生的类，真正的测试用例都是基于 base_test 派生的一个类

base_test 派生自 uvm_test，使用 uvm_component_utils 宏来注册到 factory 中。在 build_phase 中实例化 my_env，并设置 sequencer 的
default_sequence。需要注意的是，这里设置了 default_sequence，其他地方就不需要再设置了

report_phase 也是 UVM 内建的一个 phase，它在 main_phase 结束之后执行

在 base_test 中做如下事情：第一，设置整个验证平台的超时退出时间；第二，通过 config_db 设置验证平台中某些参数的值

# 第三章 UVM 基础

### 3.1 uvm_component 与 uvm_object

uvm_component 派生自 uvm_object，uvm_object 是 UVM 中最基本的类

uvm_component 与 uvm_object 的区别：

- 一是 uvm_component 在 new 时可以指定 parent 参数来形成一种树形的组织结构
- 二是 uvm_component 有 phase 的自动执行特点

从 uvm_object 派生出了两个分支，所有的 UVM 树的结点都是由 uvm_component 组成的，只有基于 uvm_component 派生的类才可能成为 UVM 树的结点；最左边分支的类或者直接派生自 uvm_object 的类，是不可能以结点的形式出现在 UVM 树上的

UVM 常用类的继承关系：
![](https://s2.loli.net/2025/07/22/pE3QU2gMB5FC7cH.jpg)

在 UVM 中，不能从 uvm_transaction 派生一个 transaction，而要从 uvm_sequence_item 派生。事实上，uvm_sequence_item 是从 uvm_transaction 派生而来的，因此，uvm_sequence_item 相比 uvm_transaction 添加了很多实用的成员变量和函数/任务，从 uvm_sequence_item 直接派生，就可以使用这些新增加的成员变量和函数/任务

与 uvm_object 相关的宏：

- uvm_object_utils：它用于把一个直接或间接派生自 uvm_object 的类注册到 factory 中
- uvm_object_param_utils：它用于把一个直接或间接派生自 uvm_object 的参数化的类注册到 factory 中
- uvm_object_utils_begin：当需要使用 field_automation 机制时，需要使用此宏
- uvm_object_param_utils_begin：与 uvm_object_utils_begin 宏一样，只是它适用于参数化的且其中某些成员变量要使用 field_automation 机制实现的类
- uvm_object_utils_end：它总是与 uvm_object\_\*\_begin 成对出现，作为 factory 注册的结束标志

与 uvm_component 相关的宏：

- uvm_component_utils：它用于把一个直接或间接派生自 uvm_component 的类注册到 factory 中
- uvm_component_param_utils：它用于把一个直接或间接派生自 uvm_component 的参数化的类注册到 factory 中
- uvm_component_utils_begin：这个宏与 uvm_object_utils_begin 相似，它用于同时需要使用 factory 机制和 field_automation 机制注册的类
- uvm_component_param_utils_begin：与 uvm_component_utils_begin 宏一样，只是它适用于参数化的，且其中某些成员变量要使用 field_automation 机制实现的类
- uvm_component_utils_end：它总是与 uvm_component\_\*\_begin 成对出现，作为 factory 注册的结束标志

### 3.2 UVM 的树形结构

UVM 通过 uvm_component 来实现树形结构，所有的 UVM 树的结点本质上都是一个 uvm_component。每个 uvm_component 都有一个特点：它们在 new 的时候，需要指定一个类型为 uvm_component、名字是 parent 的变量

==UVM 的 build_phase 是自上而下执行的==

完整的 UVM 树：
![](https://s2.loli.net/2025/08/01/Yrocd4aBeX3gwTf.png)

- get_parent 函数，用于得到当前实例的 parent
- 与 get_parent 相对的就是 get_child 函数
- 为了得到所有的 child，可以使用 get_children 函数
- get_num_children 函数用于返回当前 component 所拥有的 child 的数量

### 3.3 field automation 机制

field automation 机制的常用函数：

- copy 函数：实例复制
- compare 函数：比较两个实例是否一样
- pack 函数：将所有的字段打包成 bit 流
- unpack 函数：将一个 bit 流逐一恢复到某个类的实例中
- pack_bytes 函数：将所有的字段打包成 byte 流
- unpack_bytes 函数：将一个 byte 流逐一恢复到某个类的实例中
- pack_ints 函数：将所有的字段打包成 int 流
- unpack_ints 函数：将一个 int 流逐一恢复到某个类的实例中

### 3.4 UVM 中打印信息的控制

在打印信息之前，UVM 会比较要显示信息的冗余度级别与默认的冗余度阈值，如果小于等于阈值，就会显示，否则不会显示。默认的冗余度阈值是 UVM_MEDIUM，所有低于等于 UVM_MEDIUM（如 UVM\_ LOW）的信息都会被打印出来

- `get_report_verbosity_level`函数得到某个 component 的冗余度阈值，返回整数
- `set_report_verbosity_level`函数设置某个 component 的默认冗余度阈值
- `set_report_verbosity_level_heir`函数递归设置某个 component 及其下所有 component 的默认冗余度阈值
- `set_report_max_quit_count`函数实现当 UVM_ERROR 达到一定数量时结束仿真
- `get_max_quit_count`函数用于查询当前的退出阈值

断点功能需要从仿真器的角度进行设置，不同仿真器的设置方式不同。为了消除这些设置方式的不同，UVM 支持内建的断点功能，当执行到断点时，自动停止仿真，进入交互模式

    // 将UVM_WARNING加入结束仿真的统计次数
    env.i_agt.drv.set_report_severity_action(UVM_WARNING, UVM_DISPLAY| UVM_COUNT);
    // 递归版本
    env.i_agt.set_report_severity_action_hier(UVM_WARNING, UVM_DISPLAY| UVM_COUNT);
    // 指定id
    env.i_agt.drv.set_report_id_action("my_drv", UVM_DISPLAY| UVM_COUNT);
    env.i_agt.set_report_id_action_hier("my_drv", UVM_DISPLAY| UVM_COUNT);
    env.i_agt.drv.set_report_severity_id_action(UVM_WARNING, "my_driver", UVM_
    DISPLAY| UVM_COUNT);
    env.i_agt.set_report_severity_id_action_hier(UVM_WARNING, "my_driver", UVM_
    DISPLAY| UVM_COUNT);

    // 只要将其中的UVM_COUNT替换为UVM_STOP，就可以实现相应的断点功能

### config_db 机制

config_db 机制用于在 UVM 验证平台间传递参数。它们通常都是成对出现的，但是在某些情况下，是可以只有 set 而没有 get 语句，即省略 get 语句。set 函数是寄信，而 get 函数是收信

UVM 中的路径：
![](https://s2.loli.net/2025/08/22/2YW5HQylRcebqw1.png)

    // 发端
    uvm_config_db#(int)::set(this, "env.i_agt.drv", "pre_num", 100);
    // 收端
    uvm_config_db#(int)::get(this, "", "pre_num", pre_num);

set 函数中第一个和第二个参数联合起来组成目标路径，与此路径符合的目标才能收信。第一个参数必须是一个 uvm_component 实例的指针，第二个参数是相对此实例的路径。第三个参数表示一个记号，用以说明这个值是传给目标中的哪个成员的，第四个参数是要设置的值

get 函数中的第一个参数和第二个参数联合起来组成路径。第一个参数也必须是一个 uvm_component 实例的指针，第二个参数是相对此实例的路径。一般的，如果第一个参数被设置为 this，那么第二个参数可以是一个空的字符串。第三个参数就是 set 函数中的第三个参数，这两个参数必须严格匹配，第四个参数则是要设置的变量

假如设置多次，而只获取一次，最终会得到哪个值呢？
主要是看调用 set()函数的层级，当同时发信时层级越高优先级就越高，当先后发信时则最近接收的信息优先级最高(当跨层次来看待问题时，是高层次的 set 设置优先；当处于同一层次时，上节已经提过，是时间优先)

UVM 规定层次越高，那么它的优先级越高。这里的层次指的是在 UVM 树中的位置，越靠近根结点 uvm_top，则认为其层次越高。uvm_test_top 的层次是高于 env 的，所以 uvm_test_top 中的 set 函数的优先级高

无论如何，在调用 set 函数时其第一个参数应该尽量使用 this。在无法得到 this 指针的情况下（如在 top_tb 中），使用 null 或者 uvm_root::get()

在上图 UVM 树中，driver 的路径为 uvm_test_top.env.i_agt.drv。在 uvm_test\_ top，env 或者 i_agt 中，对 driver 中的某些变量通过 config_db 机制进行设置，称为直线的设置。但是若在其他 component，如 scoreboard 中，对 driver 的某些变量使用 config_db 机制进行设置，则称为非直线的设置

在 my_driver 中使用 config_db::get 获得其他任意 component 设置给 my_driver 的参数，称为直线的获取。假如要在其他的 component，如在 reference model 中获取其他 component 设置给 my_driver 的参数的值，称为非直线的获取

要进行非直线的设置，需要仔细设置 set 函数的第一个和第二个参数-->==个人理解：其实就是让 set 设置的路径和 get 获取的路径一样==

在 UVM 树中，build_phase 是自上而下执行的，但是对于图 3-4 所示的 UVM 树来说，scb 与 i_agt 处于同一级别中，UVM 并没有明文指出同一级别的 build_phase 的执行顺序。所以当 my_driver 在获取参数值时，my\_ scoreboard 的 build_phase 可能已经执行了，也可能没有执行-->==这种非直线的设置，会有一定的风险，应该避免这种情况的出现==

非直线的获取可以在某些情况下避免 config_db::set 的冗余，因此非直线的获取可以在验证平台中多个组件（UVM 树结点）需要使用同一个参数时，减少 config_db::set 的冗余

config_db 机制功能非常强大，能够在不同层次对同一参数实现配置。但它的一个致命缺点是，其 set 函数的第二个参数是字符串，如果字符串写错，那么根本就不能正确地设置参数值，针对这种情况，UVM 提供了一个函数 check_config_usage，它可以显示出截止到此函数调用时有哪些参数是被设置过但是却没有被获取过。`由于config_db的set及get语句一般都用于build_phase阶段，所以此函数一般在connect_phase及其后任一phase被调用`

常见的枚举类型、virtual interface、bit 类型、队列等都可以成为 config_db 设置的数据类型

UVM 还提供了 print_config 函数，参数为 1 表示递归的查询，若为 0，则只显示当前 component 的信息

# 第四章

### 4.1 TLM 1.0

验证平台内部组件的通信->建立专门的通道进行通信(也有其他方法和技术实现，但会使得通信非常复杂)

TLM(transaction Level Modeling，事务级建模)，所谓 transaction level 是相对 DUT 中各个模块之间信号线级别的通信来说的。简单来说，一个 transaction 就是把具有某一特定功能的一组信息封装在一起而成为的一个类

TLM 通信常用术语(下面三个常用术语均有阻塞和非阻塞之分)：

- put 操作(A->B)
- get 操作(A<-B)
- transport 操作(put 操作+get 操作/类似于请求-应答操作)(A<->B)

无论是 get 还是 put 操作，其发起者拥有的都是 PORT 端口，而不是 EXPORT。作为一个 EXPORT 来说，只能被动地接收 PORT 的命令
PORT 和 EXPORT 体现的是一种控制流，在这种控制流中，PORT 具有高优先级，而 EXPORT 具有低优先级。只有高优先级的端口才能向低优先级的端口发起三种操作

UVM 提供对 TLM 操作的支持，在其中实现了 PORT 与 EXPORT。对应于不同的操作，有不同的 PORT，UVM 中常用的 PORT 类和 EXPORT 类有：

    来源：UVM源代码
    uvm_blocking_put_port#(T);
    uvm_nonblocking_put_port#(T);
    uvm_put_port#(T);
    uvm_blocking_get_port#(T);
    uvm_nonblocking_get_port#(T);
    uvm_get_port#(T);
    uvm_blocking_peek_port#(T);
    uvm_nonblocking_peek_port#(T);
    uvm_peek_port#(T);
    uvm_blocking_get_peek_port#(T);
    uvm_nonblocking_get_peek_port#(T);
    uvm_get_peek_port#(T);
    uvm_blocking_transport_port#(REQ, RSP);
    uvm_nonblocking_transport_port#(REQ, RSP);
    uvm_transport_port#(REQ, RSP);


    uvm_blocking_put_export#(T);
    uvm_nonblocking_put_export#(T);
    uvm_put_export#(T);
    uvm_blocking_get_export#(T);
    uvm_nonblocking_get_export#(T);
    uvm_get_export#(T);
    uvm_blocking_peek_export#(T);
    uvm_nonblocking_peek_export#(T);
    uvm_peek_export#(T);
    uvm_blocking_get_peek_export#(T);
    uvm_nonblocking_get_peek_export#(T);
    uvm_get_peek_export#(T);
    uvm_blocking_transport_export#(REQ, RSP);
    uvm_nonblocking_transport_export#(REQ, RSP);
    uvm_transport_export#(REQ, RSP);

### 4.2 UVM 中各种端口互连

如果需要在 A 和 B 间、C 和 D 间通信，必须在它们之间建立一种连接关系，UVM 中使用 connect 函数建立连接关系，例如：
A 作为发起者要和 B 通信，只能写成`A.port.connect(B.export)`，不能写成`B.export.connect(A.port)`，只有发起者才能调用 connect 函数，connect 函数一定要在 connect_phase 调用

当调用 connect 函数连接 A 的 port 对象和 B 的 export 对象后，还需要 B 的 export 对象后续的某个组件进行处理，因为 PORT 和 EXPORT 类只是作为门进行通行的作用，没有存储 transaction 的功能；UVM 中完成存储等处理的组件也是一种端口：IMP，它承担了 UVM 中 TLM 的绝大部分实现代码，UVM 中的 IMP 有：

    来源：UVM源代码
    uvm_blocking_put_imp#(T, IMP);
    uvm_nonblocking_put_imp#(T, IMP);
    uvm_put_imp#(T, IMP);
    uvm_blocking_get_imp#(T, IMP);
    uvm_nonblocking_get_imp#(T, IMP);
    uvm_get_imp#(T, IMP);
    uvm_blocking_peek_imp#(T, IMP);
    uvm_nonblocking_peek_imp#(T, IMP);
    uvm_peek_imp#(T, IMP);
    uvm_blocking_get_peek_imp#(T, IMP);
    uvm_nonblocking_get_peek_imp#(T, IMP);
    uvm_get_peek_imp#(T, IMP);
    uvm_blocking_transport_imp#(REQ, RSP, IMP);
    uvm_nonblocking_transport_imp#(REQ, RSP, IMP);
    uvm_transport_imp#(REQ, RSP, IMP);

IMP 定义中的 blocking、nonblocking、put、get、peek、get_peek、transport 等关键字只意味着它们可以和相应类型的 PORT 或者 EXPORT 进行通信，且通信时作为被动承担者

按照控制流的优先级排序，UVM 中三种端口顺序为：PORT、EXPORT、IMP。IMP 的优先级最低，一个 PORT 可以连接到一个 IMP，并发起三种操作，反之则不行；同样，EXPORT 也可以与 IMP 相连接，其连接方法与 PORT 和 IMP 的连接完全一样

在 UVM 中，只有 IMP 才能作为连接关系的终点。如果是 PORT 或者 EXPORT 作为终点，则会报错

前六个 IMP 定义中的第一个参数 T 是这个 IMP 传输的数据类型。第二个参数 IMP，UVM 文档中把其解释为实现这个接口的一个 component(与 PORT、EXPORT 和 IMP 无关)，即每一个 IMP 要和一个 component 相对应

而该 componentB 的关键是定义一个任务或函数 put/get/peek，对于所有 blocking 系列的端口来说，可以定义相应的任务或函数；但是对于 nonblocking 系列端口来说，只能定义函数

在 UVM 中，支持带层次的连接关系，像 PORT 和 PORT、EXPORT 和 EXPORT 的连接，例如，C 的 PORT 先连接到 A 的 PORT，最终连接到 B 的 IMP

在这些连接关系中，需要谨记的是连接的终点必须是一个 IMP

### 4.3 UVM 中的通信方式

除上述的 PORT、EXPORT 和 IMP 端口外，UVM 还有两种特殊的端口：analysis_port 和 analysis_export，它们其实与 put 和 get 系列端口类似，都用于传递 transaction，区别如下：

- analysis_port（analysis_export）更像是一个广播。默认情况下，一个 analysis_port（analysis_export）可以连接多个 IMP，也就是说，analysis_port（analysis_export）与 IMP 之间的通信是一对多的通信，但是 IMP 的类型必须是 uvm\_ analysis_imp，否则会报错；而 put 和 get 系列端口与相应 IMP 的通信是一对一的通信（除非在实例化时指定可以连接的数量）
- 由于 analysis_port 和 analysis_export 是广播，因此不存在阻塞和非阻塞

对于 put 系列端口，有 put、try_put、can_put 等操作，对于 get 系列端口，有 get、try_get 和 can_get 等操作。对于 analysis_port 和 analysis_export 来说，只有一种操作：write。在 analysis_imp 所在的 component，必须定义一个名字为 write 的函数

若存在两个 IMP 的话，则需要通过宏 uvm_analysis_imp_decl 解决

==其他的通信方式：FIFO==
FIFO 的本质是一块缓存加两个 IMP。在 monitor 与 FIFO 的连接关系中，monitor 中依然是 analysis\_ port，FIFO 中是 uvm_analysis_imp，数据流和控制流的方向相同。在 scoreboard 与 FIFO 的连接关系中，scoreboard 中使用 blocking_get_port 端口，而 FIFO 中使用的是一个 get 端口的 IMP。在这种连接关系中，控制流是从 scoreboard 到 FIFO，而数据流是从 FIFO 到 scoreboard

![](https://s2.loli.net/2025/08/23/DrPAlj1TchpCBbG.png)

peek 端口与 get 相似，其数据流、控制流都相似，唯一的区别在于当 get 任务被调用时，FIFO 内部缓存中会少一个 transaction，而 peek 被调用时，FIFO 会把 transaction 复制一份发送出去，其内部缓存中的 transaction 数量并不会减少

FIFO 的类型有两种，一种是上节介绍的 uvm_tlm_analysis_fifo，另外一种是 uvm_tlm\_ fifo。这两者的唯一差别在于前者有一个 analysis_export 端口，并且有一个 write 函数，而后者没有

# 第五章

### 5.1 phase 机制

UVM 中的 phase，按照其是否消耗仿真时间（\$time 打印出的时间）的特性，可以分成两大类，一类是 function phase，如 build_phase、connect_phase 等，这些 phase 都不耗费仿真时间，通过函数来实现；另外一类是 task phase，如 run_phase 等，它们耗费仿真时间，通过任务来实现

UVM 中的 phase(灰色背景为 task phase，其他为 function phase)，其中使用频率最高的是 build_phase、connect_phase 和 main_phase:
![](https://s2.loli.net/2025/08/24/5CJmptVgnS2Y8jw.png)

上述所有的 phase 都会按照图中的顺序自上而下自动执行，对于 function phase 来说，在同一时间只有一个 phase 在执行；但是 task phase 中，run_phase 和 pre_reset_phase 等 12 个小的 phase 并行运行

对于 build_phase 来说，UVM 是按照 UVM 树自上而下执行的，除了自上而下的执行方式外，UVM 的 phase 还有一种执行方式是自下而上。事实上，除了 build_phase 之外，所有不耗费仿真时间的 phase（即 function phase）都是自下而上执行的。无论是自上而下还是自下而上，都只适应于 UVM 树中有直系关系的 component

类似 run_phase、main_phase 等 task_phase 也都是按照自下而上的顺序执行的，但是将这些 run_phase 通过 fork…join_none 的形式全部启动。所以，更准确的说法是自下而上的启动，同时在运行

对于 build_phase 来说，uvm_component 对其做的最重要的事情就是 3.5.3 节所示的自动获取通过 config_db::set 设置的参数。如果要关掉这个功能，可以在自己的 build_phase 中不调用 super.build_phase

对于直接扩展自 uvm_component 的类，除 build_phase 外，在写其他 phase 时，完全可以不必加上 super.xxxx_phase 语句

phase 的跳转：jump 函数，其中 uvm_pre_reset\_ phase::get()后的所有 phase 作为该函数的参数都可以

    function void uvm_phase::jump(uvm_phase phase);

除了使用 uvm_info 在每个 phase 打印不同的信息外，UVM 提供命令行参数 UVM_PHASE_TRACE 来对 phase 机制进行调试

在 UVM 中通过 uvm_root 的 set_timeout 函数可以设置超时时间，默认的超时退出时间是 9200s，是通过宏 UVM_DEFAULT_TIMEOUT 来指定

### 5.2 objection 机制

在进入到某一 phase 时，UVM 会收集此 phase 提出的所有 objection，并且实时监测所有 objection 是否已经被撤销了，当发现所有都已经撤销后，那么就会关闭此 phase，开始进入下一个 phase。当所有的 phase 都执行完毕后，就会调用\$finish 来将整个的验证平台关掉

UVM 用户一定要注意：如果想执行一些耗费时间的代码，那么要在此 phase 下任意一个 component 中至少提起一次 objection，否则很有可能直接进入下一个 phase(注意：这只适用于 12 个 run-time 的 phase)

对于 run\_ phase 来说，有两个选择可以使其中的代码运行：第一是其他动态运行的 phase 中有 objection 被提起。在这种情况下，运行时间受其他动态运行 phase 中 objection 控制，run_phase 只能被动地接受。第二是在 run_phase 中 raise_objection。这种情况下运行时间完全受 run_phase 控制

phase 的引入是为了解决何时结束仿真的问题，它更多面向 main_phase 等 task phase，而不是面向 function phase，但是在 function phase 中提起和撤销 objection 也不会报错

在 monitor 和 reference model 中它们都是无限循环的。因此一般不会在 driver 和 monitor 中控制 objection

objection 的控制策略(即在哪里提起和撤销 objection)：

- 在 scoreboard 中进行控制
- 在 sequence 中提起 sequencer 的 objection，当 sequence 完成后，再撤销此 objection(使用最多的策略)

当 UVM 在 main_phase 检测到所有的 objection 被撤销后，接下来会检查有没有设置 drain_time(`phase.phase_done.set_drain_time(this,200)`)。如果没有设置，则马上进入到 post_main_phase，否则延迟 drain_time 后再进入 post_main_phase，每个 phase 对应着一个 drain_time，默认为 0

### 5.3 domain 的应用

domain 是 UVM 中一个用于组织不同组件的概念，在默认情况下，验证平台中所有 component 都位于一个名字为 common_domain 的 domain 中

domain 把两块时钟域隔开，之后两个时钟域内的各个动态运行（run_time）的 phase 就可以不必同步。注意，这里 domain 只能隔离 run-time 的 phase，对于其他 phase，其实还是同步的，即两个 domain 的 run_phase 依然是同步的，其他的 function phase 也是同步的

将某个 component 置于某个新的 domain 中：在 component 中新建一个 domain，并将其实例化。在 connect_phase 中通过 set\_ domain 将该 component 加入到此 domain 中

# 第六章 UVM 中的 sequence

### 6.1 sequence 基础

UVM 为了解决 DUT 存在多种激励扩展性差的问题，引入了 sequence 机制，在解决的过程中还使用了 factory 机制、config 机制。使用 sequence 机制之后，在不同的测试用例中，将不同的 sequence 设置成 sequencer 的 main_phase 的 default_sequence。当 sequencer 执行到 main_phase 时，发现有 default_sequence，那么它就启动 sequence

当完成一个 sequence 的定义后，可以使用 start 任务直接将其启动，除了直接启动之外，还可以使用 default_sequence 启动。事实上 default_sequence 会调用 start 任务

    # default_sequence启动方式1
    uvm_config_db#(uvm_object_wrapper)::set(this,
                                     "env.i_agt.sqr.main_phase",
                                     "default_sequence",
                                     case0_sequence::type_id::get());
    # default_sequence启动方式2(先实例化要启动的sequence，之后再通过default_sequence启动)

    41 function void my_case0::build_phase(uvm_phase phase);
    42   case0_sequence cseq;
    43   super.build_phase(phase);
    44 45   cseq = new("cseq");
    46   uvm_config_db#(uvm_sequence_base)::set(this,
    47                                   "env.i_agt.sqr.main_phase",
    48                                   "default_sequence",
    49                                   cseq);
    50 endfunction

**当一个 sequence 启动后会自动执行 sequence 的 body 任务**。其实，除了 body 外，还会自动调用 sequence 的 pre_body 与 post_body

### 6.2 sequence 的仲裁机制

**UVM 支持同一时刻在同一 sequencer 上启动多个 sequence，sequencer 根据 UVM 的 sequence 机制中的仲裁机制选择使用哪个 sequence 的 transaction**

对于 transaction 来说，存在优先级的概念，通常来说，优先级越高越容易被选中。当使用 uvm_do 或者 uvm_do_with 宏时，产生的 transaction 的优先级是默认的优先级，即-1。可以通过 uvm_do_pri 及 uvm_do_pri_with 改变所产生的 transaction 的优先级(数字越大，优先级越高)

默认情况下 sequencer 的仲裁算法是 SEQ_ARB_FIFO。它会严格遵循先入先出的顺序，而不会考虑优先级。SEQ_ARB_WEIGHTED 是加权的仲裁；SEQ_ARB_RANDOM 是完全随机选择；SEQ_ARB_STRICT_FIFO 是严格按照优先级的，当有多个同一优先级的 sequence 时，按照先入先出的顺序选择；SEQ_ARB_STRICT_RANDOM 是严格按照优先级的，当有多个同一优先级的 sequence 时，随机从最高优先级中选择；SEQ_ARB_USER 则是用户可以自定义一种新的仲裁算法

若想使优先级起作用，应该设置仲裁算法为 SEQ_ARB_STRICT_FIFO：`env.i_agt.sqr.set_arbitration(SEQ_ARB_STRICT_FIFO)`

除 transaction 有优先级外，sequence 也有优先级的概念。可以在 sequence 启动时指定其优先级，对 sequence 设置优先级的本质即设置其内产生的 transaction 的优先级

针对需要连续发送 transaction 的 sequence，则需要采用 lock 操作。所谓 lock，就是 sequence 向 sequencer 发送一个请求，这个请求与其他 sequence 发送 transaction 的请求一同被放入 sequencer 的仲裁队列中。当其前面的所有请求被处理完毕后，sequencer 就开始响应这个 lock 请求，此后 sequencer 会一直连续发送此 sequence 的 transaction，直到 unlock 操作被调用

与 lock 操作一样，grab 操作也用于暂时拥有 sequencer 的所有权，只是 grab 操作比 lock 操作优先级更高。lock 请求是被插入 sequencer 仲裁队列的最后面，等到它时，它前面的仲裁请求都已经结束了。grab 请求则被放入 sequencer 仲裁队列的最前面，它几乎是一发出就拥有了 sequencer 的所有权

两个 sequence 同时试图调用 lock 或 grab 函数，在先获得所有权的 sequence 执行完毕后才会将所有权交还给另外一个试图所有权的 sequence；但当调用 grab 任务时其他 sequence 已经调用 lock 锁定了获取 sequencer 的所有权则会等待 lock 的释放才会获得 sequencer 的所有权

sequencer 在仲裁时，会查看 sequence 的 is_relevant 函数的返回结果。如果为 1，说明此 sequence 有效，否则无效。因此可以通过重载 is_relevant 函数来使 sequence 失效，sequence 失效时不能参与仲裁

当 sequencer 发现在其上启动的所有 sequence 都无效时，此时会调用 wait_for_relevant 并等待 sequence 变有效；is_relevant 与 wait_for_relevant 一般应成对重载，不能只重载其中的一个，否则会报错

### 6.3 sequence 相关宏及其实现

- uvm_do 系列宏(uvm_do 系列的其他七个宏其实都是用 uvm_do_on_pri_with 宏来实现的)
  uvm_do 系列宏中，其第一个参数除了可以是 transaction 的指针外，还可以是某个 sequence 的指针

  来源：UVM 源代码
  `uvm_do(SEQ_OR_ITEM)o
`uvm_do_pri(SEQ_OR_ITEM, PRIORITY)
  `uvm_do_with(SEQ_OR_ITEM, CONSTRAINTS)
`uvm_do_pri_with(SEQ_OR_ITEM, PRIORITY, CONSTRAINTS)
  `uvm_do_on(SEQ_OR_ITEM, SEQR)
`uvm_do_on_pri(SEQ_OR_ITEM, SEQR, PRIORITY)
  `uvm_do_on_with(SEQ_OR_ITEM, SEQR, CONSTRAINTS)
`uvm_do_on_pri_with(SEQ_OR_ITEM, SEQR, PRIORITY, CONSTRAINTS)

除了使用 uvm_do 宏产生 transaction，还可以使用 uvm_create 宏与 uvm_send 宏来产生，uvm_create 宏的作用是实例化 transaction。当一个 transaction 被实例化后，可以对其做更多的处理，处理完毕后使用 uvm_send 宏发送出去。这种使用方式比 uvm_do 系列宏更加灵活(也完全可以不使用 uvm_create 宏，而直接调用 new 进行实例化)

- uvm_rand_send 系列宏

  来源：UVM 源代码
  `uvm_rand_send(SEQ_OR_ITEM)
`uvm_rand_send_pri(SEQ_OR_ITEM, PRIORITY)
  `uvm_rand_send_with(SEQ_OR_ITEM, CONSTRAINTS)
`uvm_rand_send_pri_with(SEQ_OR_ITEM, PRIORITY, CONSTRAINTS)

若不使用宏产生 transaction 的方式要依赖于两个任务：start_item 和 finish_item。在使用这两个任务前，必须要先实例化 transaction 后才可以调用这两个任务

    # 完整使用start_item和finish_item构建的一个sequence
    virtual task body();
      repeat(10) begin
        tr = new("tr");
        assert(tr.randomize() with {tr.pload_size == 200;});
        start_item(tr);
        finish_item(tr);
      end
    endtask

uvm_do 宏封装了从 transaction 实例化到发送的一系列操作，封装的越多，则其灵活性越差。为了增加 uvm_do 系列宏的功能，UVM 提供了三个接口：pre_do、mid_do 与 post_do

pre_do 是一个任务，在 start_item 中被调用，它是 start_item 返回前执行的最后一行代码，在它执行完毕后才对 transaction 进行随机化。mid_do 是一个函数，位于 finish_item 的最开始。在执行完此函数后，finish_item 才进行其他操作。post_do 也是一个函数，也位于 finish_item 中，它是 finish_item 返回前执行的最后一行代码

### 6.4 sequence 进阶应用

在一个 sequence 的 body 中，除了可以使用 uvm_do 宏产生 transaction 外，其实还可以启动其他的 sequence，即一个 sequence 内启动另外一个 sequence，这就是嵌套的 sequence；嵌套 sequence 的前提是，在套里面的所有 sequence 产生的 transaction 都可以被同一个 sequencer 所接受

除了 uvm_do 宏外，前面介绍的 uvm_send 宏、uvm_rand_send 宏、uvm_create 宏，其第一个参数都可以是 sequence 的指针。唯一例外的是 start_item 与 finish_item，这两个任务的参数必须是 transaction 的指针

在 transaction 的定义中，通常使用 rand 来对变量进行修饰，说明在调用 randomize 时要对此字段进行随机化；sequence 里可以添加任意多的 rand 修饰符，用以规范它产生的 transaction

若需要将两个截然不同的 transaction 交给同一个 sequencer，需要将 sequencer 和 driver 能够接受的数据类型设置为 uvm_sequence_item，由于 driver 中接收的数据类型是 uvm_sequence_item，如果它要使用 my_transaction 或者 your_transaction 中的成员变量，必须使用 cast 转换

### 6.5 virtual sequence 的使用

实现 sequence 之间同步的最好的方式就是使用 virtual sequence。从字面上理解，即虚拟的 sequence。虚拟的意思就是它根本就不发送 transaction，它只是控制其他的 sequence，起统一调度的作用

### 6.6 在 sequence 中使用 config_db

在 UVM 中使用 get_full_name()可以得到一个 component 的完整路径，同样的，此函数也可以在一个 sequence 中被调用

### 6.7 response 的使用

sequence 机制提供了一种 sequence→sequencer→driver 的单向数据传输机制。同时，sequence 机制允许 driver 将一个 response 返回给 sequence

通常来说，一个 transaction 对应一个 response，但是事实上，UVM 也支持一个 transaction 对应多个 response 的情况，在这种情况下，在 sequence 中需要多次调用 get\_ response，而在 driver 中，需要多次调用 put_response

response 机制的原理是 driver 将 rsp 推送给 sequencer，而 sequencer 内部维持一个队列，当有新的 response 进入时，就推入此队列。但是此队列的大小并不是无限制的，在默认情况下，其大小为 8

### 6.8 sequence library

sequence library 就是一系列 sequence 的集合，sequence library 派生自 uvm_sequence，从本质上说它是一个 sequence。它根据特定的算法随机选择注册在其中的一些 sequence，并在 body 中执行这些 sequence

定义 sequence library 时有三点要特别注意：一是从 uvm_sequence 派生时要指明此 sequence library 所产生的 transaction 类型，这点与普通的 sequence 相同；二是在其 new 函数中要调用 init_sequence_library，否则其内部的候选 sequence 队列就是空的；三是要调用 uvm_sequence_library_utils 注册

一个 sequence 在定义时使用宏 uvm_add_to_seq_lib 来将其加入某个 sequence library 中

sequence library 随机从其 sequence 队列中选择几个执行。这是由其变量 selection_mode 决定的；sequence library 会在 min_random_count 和 max_random_count 之间随意选择一个数来作为执行次数；UVM 提供了一个类 uvm_sequence_library_cfg 来对 sequence library 的迭代次数和选择算法进行配置

# 第七章 UVM 中的寄存器模型

### 7.1 寄存器模型简介

有了寄存器模型后，可以在任何耗费时间的 phase 中使用寄存器模型以前门访问或后门（BACKDOOR）访问的方式来读取寄存器的值，同时还能在某些不耗费时间的 phase（如 check_phase）中使用后门访问的方式来读取寄存器的值

前门访问与后门访问是两种寄存器的访问方式。所谓前门访问，指的是通过模拟 cpu 在总线上发出读指令，进行读写操作。在这个过程中，仿真时间（\$time 函数得到的时间）是一直往前走的

而后门访问是与前门访问相对的概念。它并不通过总线进行读写操作，而是直接通过层次化的引用来改变寄存器的值

- uvm_reg_field：这是寄存器模型中的最小单位
- uvm_reg：它比 uvm_reg_field 高一个级别，但是依然是比较小的单位。一个寄存器中至少包含一个 uvm_reg_field
- uvm_reg_block：它是一个比较大的单位，在其中可以加入许多的 uvm_reg，也可以加入其他的 uvm_reg_block。一个寄存器模型中至少包含一个 uvm_reg_block
- uvm_reg_map：每个寄存器在加入寄存器模型时都有其地址，uvm_reg_map 就是存储这些地址，并将其转换成可以访问的物理地址（因为加入寄存器模型中的寄存器地址一般都是偏移地址，而不是绝对地址）。当寄存器模型使用前门访问方式来实现读或写操作时，uvm_reg_map 就会将地址转换成绝对地址，启动一个读或写的 sequence，并将读或写的结果返回。在每个 reg_block 内部，至少有一个（通常也只有一个）uvm_reg_map

### 7.2 简单的寄存器模型

要为其建造寄存器模型，首先要从 uvm_reg 派生一个 invert 类。每一个派生自 uvm_reg 的类都有一个 build，这个 build 与 uvm_component 的 build_phase 并不一样，它不会自动执行，而需要手工调用，与 build_phase 相似的是所有的 uvm_reg_field 都在这里实例化。当 reg_data 实例化后，要调用 data.configure 函数来配置这个字段

定义好此寄存器后，需要在一个由 reg_block 派生的类中将其实例化，同 uvm_reg 派生的类一样，每一个由 uvm_reg_block 派生的类也要定义一个 build 函数，一般在此函数中实现所有寄存器的实例化

一个 uvm_reg_block 中一定要对应一个 uvm_reg_map，系统已经有一个声明好的 default_map，只需要在 build 中将其实例化，通过调用 uvm_reg_block 的 create_map 来实现，create_map 有众多的参数，其中第一个参数是名字，第二个参数是基地址，第三个参数则是系统总线的宽度，这里的单位是 byte 而不是 bit，第四个参数是大小端，最后一个参数表示是否能够按照 byte 寻址

随后实例化 invert 并调用 invert.configure 函数。这个函数的主要功能是指定寄存器进行后门访问操作时的路径

最后一步则是将此寄存器加入 default_map 中。uvm_reg_map 的作用是存储所有寄存器的地址，因此必须将实例化的寄存器加入 default_map 中，否则无法进行前门访问操作

寄存器模型的前门访问操作可以分成读和写两种。无论是读或写，寄存器模型都会通过 sequence 产生一个 uvm_reg_bus_op 的变量，此变量中存储着操作类型（读还是写）和操作的地址，如果是写操作，还会有要写入的数据。此变量中的信息要经过一个转换器（adapter）转换后交给 bus_sequencer，随后交给 bus_driver，由 bus_driver 实现最终的前门访问读写操作。因此，必须要定义好一个转换器(adapter)

一个转换器要定义好两个函数，一是 reg2bus，其作用为将寄存器模型通过 sequence 发出的 uvm_reg_bus_op 型的变量转换成 bus_sequencer 能够接受的形式，二是 bus2reg，其作用为当监测到总线上有操作时，它将收集来的 transaction 转换成寄存器模型能够接受的形式，以便寄存器模型能够更新相应的寄存器的值

adapter 写好之后，就可以在 base_test 中加入寄存器模型了，要将一个寄存器模型集成到 base_test 中，那么至少需要在 base_test 中定义两个成员变量，一是 reg_model，另外一个就是 reg_sqr_adapter。将所有用到的类在 build_phase 中实例化。在实例化后 reg_model 还要做四件事：第一是调用 configure 函数，其第一个参数是 parent block，由于是最顶层的 reg_block，因此填写 null，第二个参数是后门访问路径，请参考 7.3 节，这里传入一个空的字符串。第二是调用 build 函数，将所有的寄存器实例化。第三是调用 lock_model 函数，调用此函数后，reg_model 中就不能再加入新的寄存器了。第四是调用 reset 函数，如果不调用此函数，那么 reg_model 中所有寄存器的值都是 0，调用此函数后，所有寄存器的值都将变为设置的复位值

寄存器模型的前门访问操作最终都将由 uvm_reg_map 完成，因此在 connect_phase 中，需要将转换器和 bus_sequencer 通过 set_sequencer 函数告知 reg_model 的 default_map，并将 default_map 设置为自动预测状态

当一个寄存器模型被建立好后，可以在 sequence 和其他 component 中使用；对于寄存器，寄存器模型提供了两个基本的任务：read 和 write，这两个任务参数常用的只有前面三个，第一个为 uvm\_ status_e 型的变量，这是一个输出，用于表明写操作是否成功。第二个要写的值，是一个输入，第三个是写操作的方式，可选 UVM_FRONTDOOR 和 UVM_BACKDOOR

### 7.3 后门访问与前门访问

所谓前门访问操作就是通过寄存器配置总线（如 APB 协议、OCP 协议、I2C 协议等）来对 DUT 进行操作。无论在任何总线协议中，前门访问操作只有两种：读操作和写操作。前门访问操作是比较正统的用法

对于 UVM 来说，它是一种通用的验证方法学，所以要能够处理各种 transaction 类型。幸运的是，这些要处理的 transaction 都非常相似，在综合了它们的特征后，UVM 内建了一种 transaction：uvm_reg_item。通过 adapter 的 bus2reg 及 reg2bus，可以实现 uvm_reg_item 与目标 transaction 的转换

完整的读写操作流程如下：

- 参考模型调用寄存器模型的读任务
- 寄存器模型产生 sequence，并产生 uvm_reg_item：rw
- 产生 driver 能够接受的 transaction：bus_req=adapter.reg2bus（rw）
- 把 bus_req 交给 bus_sequencer
- driver 得到 bus_req 后驱动它，得到读取的值，并将读取值放入 bus_req 中，调用 item_done
- 寄存器模型调用 adapter.bus2reg（bus_req, rw）将 bus_req 中的读取值传递给 rw
- 将 rw 中的读数据返回参考模型

在 6.7.2 节中介绍 sequence 的应答机制时提到过，如果 driver 一直发送应答而 sequence 不收集应答，那么将会导致 sequencer 的应答队列溢出。UVM 考虑到这种情况，在 adapter 中设置了 provide_responses 选项，在设置了此选项后，寄存器模型在调用 bus2reg 将目标 transaction 转换成 uvm_reg\_ item 时，其传入的参数是 rsp，而不是 req

后门访问是与前门访问相对的操作，从广义上来说，所有不通过 DUT 的总线而对 DUT 内部的寄存器或者存储器进行存取的操作都是后门访问操作

所有后门访问操作都是不消耗仿真时间（即\$time 打印的时间）而只消耗运行时间的

后门访问操作存在的意义：

- 后门访问操作能够更好地完成前门访问操作所做的事情。后门访问不消耗仿真时间，与前门访问操作相比，它消耗的运行时间要远小于前门访问操作的运行时间
- 后门访问操作能够完成前门访问操作不能完成的事情

在 driver 等组件中也可以使用这种绝对路径的方式进行后门访问操作，但强烈建议不要在 driver 等验证平台的组件中使用绝对路径。这种方式的可移植性不强。如果想在 driver 或 monitor 中使用后门访问，一种方法是使用接口

UVM 中后门访问操作的实现：DPI+VPI
vpi_get_value 用于从 RTL 中得到一个寄存器的值。vpi_put_value 用于将 RTL 中的寄存器设置为某个值

    # 常用的VPI接口
    vpi_get_value(obj, p_value);
    vpi_put_value(obj, p_value, p_time, flags);

UVM 中使用 DPI+VPI 的方式来进行后门访问操作，它大体的流程是：

- 在建立寄存器模型时将路径参数设置好
- 在进行后门访问的写操作时，寄存器模型调用 uvm_hdl_deposit 函数，在 C/C++侧，此函数内部会调用 vpi_put_value 函数来对 DUT 中的寄存器进行写操作
- 进行后门访问的读操作时，调用 uvm_hdl_read 函数，在 C/C++侧，此函数内部会调用 vpi_get_value 函数来对 DUT 中的寄存器进行读操作，并将读取值返回

UVM 提供两类后门访问的函数：一是 UVM_BACKDOOR 形式的 read 和 write，二是 peek 和 poke。这两类函数的区别是，第一类会在进行操作时模仿 DUT 的行为，第二类则完全不管 DUT 的行为。如对一个只读的寄存器进行写操作，那么第一类由于要模拟 DUT 的只读行为，所以是写不进去的，但是使用第二类可以写进去

无论是 peek 还是 poke，其常用的参数都是前两个。各自的第一个参数表示操作是否成功，第二个参数表示读写的数据

### 7.4 复杂的寄存器模型

要将一个子 reg_block 加入父 reg_block 中，第一步是先实例化子 reg_block。第二步是调用子 reg_block 的 configure 函数。如果需要使用后门访问，则在这个函数中要说明子 reg_block 的路径，这个路径不是绝对路径，而是相对于父 reg_block 来说的路径。第三步是调用子 reg_block 的 build 函数。第四步是调用子 reg_block 的 lock_model 函数。第五步则是将子 reg_block 的 default_map 以子 map 的形式加入父 reg_block 的 default_map 中

一般将具有同一基地址的寄存器作为整体加入一个 uvm_reg_block 中，而不同的基地址对应不同的 uvm_reg_block

### 7.5 寄存器模型对 DUT 的模拟

对于任意一个寄存器，寄存器模型中都会有一个专门的变量用于最大可能地与 DUT 保持同步，这个变量在寄存器模型中称为 DUT 的镜像值（mirrored value）

除了 DUT 的镜像值外，寄存器模型中还有期望值（desired value）。如目前 DUT 中 invert 的值为'h0，寄存器模型中的镜像值也为'h0，但是希望向此寄存器中写入一个'h1，此时一种方法是直接调用前面介绍的 write 任务，将'h1 写入，期望值与镜像值都更新为'h1；另外一种方法是通过 set 函数将期望值设置为'h1（此时镜像值依然为 0），之后调用 update 任务，update 任务会检查期望值和镜像值是否一致，如果不一致，那么将会把期望值写入 DUT 中，并且更新镜像值

- read\&write 操作：无论通过后门访问还是前门访问的方式从 DUT 中读取或写入寄存器的值，在操作完成后，寄存器模型都会根据读写的结果更新期望值和镜像值（二者相等）
- peek\&poke 操作：在操作完成后，寄存器模型会根据操作的结果更新期望值和镜像值（二者相等）
- get\&set 操作：set 操作会更新期望值，但是镜像值不会改变。get 操作会返回寄存器模型中当前寄存器的期望值
- update 操作：这个操作会检查寄存器的期望值和镜像值是否一致，如果不一致，那么就会将期望值写入 DUT 中，并且更新镜像值，使其与期望值一致
- randomize 操作：寄存器模型提供 randomize 接口。randomize 之后，期望值将会变为随机出的数值，镜像值不会改变。但是并不是寄存器模型中所有寄存器都支持此函数。如果不支持，则 randomize 调用后其期望值不变

### 7.6 寄存器模型中一些内建的 sequence

UVM 提供了一系列的 sequence，可以用于检查寄存器模型及 DUT 中的寄存器：

- uvm_reg_mem_hdl_paths_seq 即用于检查 hdl 路径的正确性
- uvm_reg_hw_reset_seq 用于检查上电复位后寄存器模型与 DUT 中寄存器的默认值是否相同
- uvm_reg_access\_ seq 用于检查寄存器的读写(这个 sequence 会使用前门访问的方式向所有寄存器写数据，然后使用后门访问的方式读回，并比较结果；最后把这个过程反过来，使用后门访问的方式写入数据，再用前门访问读回)
- uvm_mem_access_seq 用于检查存储器的读写(这个 sequence 会通过使用前门访问的方式向所有存储器写数据，然后使用后门访问的方式读回，并比较结果。最后把这个过程反过来，使用后门访问的方式写入数据，再用前门访问读回)

### 7.7 寄存器模型的高级用法

更新寄存器模型的方法：

- 当 driver 将读取值返回后，寄存器模型会更新寄存器的镜像值和期望值。这个功能被称为寄存器模型的 auto predict 功能
- 由 monitor 将从总线上收集到的 transaction 交给寄存器模型，后者更新相应寄存器的值

UVM 提供 mirror 操作，用于读取 DUT 中寄存器的值并将它们更新到寄存器模型中，它有两种应用场景，一是在仿真中不断地调用它，使得到整个寄存器模型的值与 DUT 中寄存器的值保持一致，此时 check 选项是关闭的。二是在仿真即将结束时，检查 DUT 中寄存器的值与寄存器模型中寄存器的镜像值是否一致，这种情况下，check 选项是打开的

UVM 提供 predict 操作，用于人为地更新镜像值，但是同时又不要对 DUT 进行任何操作

避免一个 field 被随机化，可以在以下三种方式中任选其一：

- 当在 uvm_reg 中定义此 field 时，不要设置为 rand 类型
- 在调用此 field 的 configure 函数时，第八个参数设置为 0
- 设置此 field 的类型为 RO、RC、RS、WC、WS、W1C、W1S、W1T、W0C、W0S、W0T、W1SRC、W1CRS、W0SRC、W0CRS、WSRC、WCRS、WOC、WOS 中的一种

其中第一种方式也适用于关闭某个 uvm_reg 或者某个 uvm_reg_block 的 randomize 功能，randomize 会更新寄存器模型中的预期值

### 7.8 寄存器模型的其他常用函数

除了这种指针传递的形式外，UVM 还提供其他函数，使得可以在不使用指针传递的情况下得到寄存器模型的指针，在使用 get_root_blocks 函数得到 reg_block 的指针后，要使用 cast 将其转化为目标 reg_block 形式（示例中为 reg_model）。以后就可以直接使用 p_rm 来进行寄存器操作，而不必使用 p_sequencer.p_rm

在建立了寄存器模型后，可以直接通过层次引用的方式访问寄存器，但是出于某些原因，如果依然要使用地址来访问寄存器模型，那么此时可以使用 get\_ reg_by_offset 函数通过寄存器的地址得到一个 uvm_reg 的指针，再调用此 uvm_reg 的 read 或者 write 就可以进行读写操作

# 第八章 UVM 中的 factory 机制

### 8.1 SystemVerilog 对重载的支持

重载的最大优势是使得一个子类的指针以父类的类型传递时，其表现出的行为依然是子类的行为

### 8.2 使用 factory 机制进行重载

在实例化时，UVM 会通过 factory 机制在自己内部的一张表格中查看是否有相关的重载记录。set_type_override_by_type 语句相当于在 factory 机制的表格中加入了一条记录。当查到有重载记录时，会使用新的类型来替代旧的类型；有时候可能并不是希望把验证平台中的 A 类型全部替换成 B 类型，而只是替换其中的某一部分，这种情况就要用到 set_inst_override_by_type 函数

使用 factory 机制的重载是有前提的，并不是任意的类都可以互相重载。要想使用重载的功能，必须满足以下要求：

- 第一，无论是重载的类（parrot）还是被重载的类（bird），都要在定义时注册到 factory 机制中
- 第二，被重载的类（bird）在实例化时，要使用 factory 机制式的实例化方式，而不能使用传统的 new 方式
- 第三，最重要的是，重载的类（parrot）要与被重载的类（bird）有派生关系。重载的类必须派生自被重载的类，被重载的类必须是重载类的父类
- 第四，component 与 object 之间互相不能重载

无论是 set_type_override_by_type 还是 set_inst_override_by_type，它们的参数都是一个 uvm_object_wrapper 型的类型参数，这种参数通过 xxx::get_type()的形式获得。UVM 还提供了另外一种简单的方法来替换这种晦涩的写法：字符串

在有多个重载时，最终重载的类要与最初被重载的类有派生关系。最终重载的类必须派生自最初被重载的类，最初被重载的类必须是最终重载类的父类

factory 机制的重载功能很强大，UVM 提供了 print_override_info 函数来输出所有的打印信息，print_override_info 是一个 uvm_component 的成员函数，它实质上是调用 uvm_factory 的 debug_create_by_name。除了这个函数外，uvm_factory 还有 debug_create_by_type，除此之外，uvm_factory 还提供 print 函数；除了上述这些函数外，还有另外一个重要的工具可以显示出整棵 UVM 树的拓扑结构，这个工具就是 uvm_root 的 print_topology 函数。UVM 树在 build_phase 执行完成后才完全建立完成，因此，这个函数应该在 build_phase 之后调用

print_topology 这个函数非常有用，即使在不进行 factory 机制调试的情况下，也可以通过调用它来显示整个验证平台的拓扑结构是否与自己预期的一致。因此可以把其放在所有测试用例的基类 base_test 中

### 8.3 常用的重载

有了 factory 机制的重载功能后，可以不用重新写一个 abnormal_sequence，而继续使用 normal_sequence 作为新的测试用例的 default_sequence，只是需要将 my_transaction 使用 crc_err_tr 重载

transaction 可以重载，同样的，sequence 也可以重载。上节使用的 transaction 重载能工作的前提是约束也可以重载

8.3.1 节和 8.3.2 节分别使用重载 transaction 和重载 sequence 的方式产生异常的测试用例。其实，还可以使用重载 driver 的方式产生

除了 driver 可以重载外，scoreboard 与参考模型等都可以重载

### 8.4 factory 机制的实现

factory 机制提供了一系列接口来创建实例：

- create_object_by_name，用于根据类名字创建一个 object
- create_object_by_type，根据类型创建一个 object
- create_component_by_name，根据类名创建一个 component(该函数一般只在一个 component 的 new 或者 build_phase 中使用)
- create_component_by_type，根据类型创建一个 component

在没有 factory 机制之前，要创建一个类的实例，只能使用 new 函数；有了 factory 机制之后，除了使用 new 函数外，还可以根据类名创建这个类的一个实例；另外，还可以在创建类的实例时根据是否有重载记录来决定是创建原始的类，还是创建重载的类的实例

# 第九章 UVM 中代码的可重用性

### 9.1 callback 机制

在 UVM 验证平台中，callback 机制的最大用处就是提高验证平台的可重用性，除了提高可重用性外，callback 机制还用于构建异常的测试用例

randomize 是 SystemVerilog 提供的一个 callback 函数，同时 SystemVerilog 还提供了一个 post_randomize()函数，当 randomize()之后，系统会自动调用 post_randomize 函数

post_randomize 函数是 SystemVerilog 提供的广义的 callback 函数。UVM 也为用户提供了广义的 callback 函数/任务：pre_body 和 post_body，除此之外还有 pre_do、mid_do 和 post_do

callback 机制、sequence 机制和 factory 机制并不是互斥的，三者都能分别实现同一目的。当这三者互相结合时，又会产生许多新的解决问题的方式。如果在建立验证平台和测试用例时，能够择优选择其中最简单的一种实现方式，那么搭建出来的验证平台一定是足够强大、足够简练的

### 9.2 功能的模块化：小而美

在验证平台的设计中，要尽量做到小而美，避免大而全

### 9.3 参数化的类

UVM 的 config_db 机制可以用于传递 virtual interface。SystemVerilog 支持参数化的 interface。UVM 的 factory 机制不支持参数化类中的默认参数，声明 agent 时可以省略参数，但是在实例化时，必须将省略的参数加上

### 9.4 模块级到芯片级的代码重用

现代芯片的验证通常分为两个层次，一是模块级别（block level，也称为 IP 级别、unit 级别）验证，二是芯片级别（也称为 SOC 级别）验证

# 第十章 UVM 高级应用

### 10.1 interface

在 interface 中可以定义任务与函数。除此之外，还可以在 interface 中使用 always 语句和 initial 语句

interface 可以代替 driver 做很多事情，但是并不能代替 driver 做所有的事情。interface 只适用于做一些低层次的转换，如上述的 8b10b 转换、曼彻斯特编码等。这些转换动作是与 transaction 完全无关的

使用 interface 代替 driver 的第一个好处是可以让 driver 从底层繁杂的数据处理中解脱出来，更加专注于处理高层数据。第二个好处是有更多的数据出现在 interface 中，这会对调试起到很大的帮助

### 10.2 layer sequence

同一个 sequence 中产生了两种不同的 transaction，虽然这两种 transaction 之间有必然的联系（ip_transaction 作为 my_transaction 的 pload），但是将它们放在一起并不合适。最好的办法是将它们分离，一个 sequence 负责产生 ip_transaction，另外 sequence 负责产生 my_transaction，前者将产生的 ip_transaction 交给后者。这就是 layer sequence

### 10.3 sequence 的其他问题

在某些协议中，需要 driver 每隔一段时间向 DUT 发送一些类似心跳的信号。这些心跳信号的包与其他的普通的包并没有本质上的区别，其使用的 transaction 也都是普通的 transaction。发送这种心跳包有两种选择，一种是在 driver 中实现，driver 负责包的产生、发送；另外一种是在 sequence 中实现，这个 sequence 被做成一种无限循环的 sequence，这个 sequence 精确地计时，当需要发送心跳包时，生成一个心跳包并发送出去

### 10.4 DUT 参数的随机化

验证中有两大问题：一是向 DUT 灌输不同的激励，二是为 DUT 配置不同的参数

- 使用寄存器模型随机化参数
- 使用单独的参数类

### 10.5 聚合参数

在验证平台中用到的参数有两大类，一类是验证环境与 DUT 中都要用到的参数，这些参数通常都对应 DUT 中的寄存器；另外一类是验证环境中独有的，比如 driver 中要发送的 preamble 数量的上限和下限

对于一个大的项目来说，要配置的参数可能有千百个。如果全部使用 config_db 的写法就太重复了，一种比较好的方法就是将这 1000 个变量放在一个专门的类里面来实现

使用聚合参数后，可以将此参数类的指针放在 virtual sequencer 中，聚合参数方便了在 sequence 中改变验证平台参数

### 10.6 config_db

在本书前面的介绍中，使用 config*db 几乎都是在 build_phase 中。由于其 config* db::set 的第二个参数是字符串，所以经常出错。一个 component 的路径可以通过 get*full* name()来获得。要想避免 config_db::set 第二个参数引起的问题，一种可行的想法是把这个参数使用 get_full_name()
