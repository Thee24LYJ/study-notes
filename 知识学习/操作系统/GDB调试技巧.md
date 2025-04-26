[GDB 调试 \| 爱编程的大丙](https://subingwen.cn/linux/gdb/)

# 启动与退出 gdb 调试

### 启动 gdb

```bash
# gdb 可执行程序的名字
# 此时程序还未启动
$ gdb app
(gdb)
```

### 命令行传参

```bash
# 设置的时机: 启动gdb之后, 在应用程序启动之前
(gdb) set args 参数1 参数2 .... ...
# 查看设置的命令行参数
(gdb) show args
```

### gdb 中启动程序

```bash
# 方式1
(gdb) r
(gdb) run
#方式2
(gdb) start
```

- run: 可以缩写为 r, 如果程序中设置了断点会停在第一个断点的位置, 如果没有设置断点, 程序直接执行完毕
- start: 启动程序, 最终会阻塞在 main 函数的第一行，等待输入后续其它 gdb 指令
- 让程序 start 之后继续运行, 或者在断点处继续运行，可以使用 continue 命令, 可以简写为 c

### 退出 gdb

```bash
(gdb) q
(gdb) quit
```

# 查看代码

```bash
(gdb) l
(gdb) list

# 从第一行开始显示
(gdb) list
# 列值这行号对应的上下文代码, 默认情况下只显示10行内容
(gdb) list 行号
# 显示这个函数的上下文内容, 默认显示10行
(gdb) list 函数名

# 切换到指定的文件，并列出这行号对应的上下文代码, 默认情况下只显示10行内容
(gdb) l 文件名:行号
# 切换到指定的文件，并显示这个函数的上下文内容, 默认显示10行
(gdb) l 文件名:函数名


```

# 断点操作

断点的设置有两种方式一种是常规断点，程序只要运行到这个位置就会被阻塞，还有一种叫条件断点，只有指定的条件被满足了程序才会在断点处阻塞

```bash
# 在当前文件的某一行上设置断点
(gdb) b 行号
(gdb) b 函数名    # 停止在函数的第一行

# 在非当前文件的某一行上设置断点
(gdb) b 文件名:行号
(gdb) b 文件名:函数名		# 停止在函数的第一行

# 必须要满足某个条件, 程序才会停在这个断点的位置上
# 通常情况下, 在循环中条件断点用的比较多
(gdb) b 行数 if 变量名==某个值

# 查看设置的断点信息
(gdb) i b
(gdb) info break

# 删除断点 delete==del==d
(gdb) d 断点的编号1 [断点编号2 ...]

# 断点失效, gdb调试过程中程序不会停在该位置
# disable == dis
# 设置某一个或者某几个断点无效
(gdb) dis 断点1的编号 [断点2的编号 ...]
# 设置某个区间断点无效
(gdb) dis 断点1编号-断点n编号

# 断点生效
# enable == ena
# 设置某一个或者某几个断点有效
(gdb) ena 断点1的编号 [断点2的编号 ...]
# 设置某个区间断点有效
(gdb) ena 断点1编号-断点n编号
```

# 调试命令

- 和 print 命令一样，display 命令也用于调试阶段查看某个变量或表达式的值，它们的区别是，使用 display 命令查看变量或表达式的值，每当程序暂停执行（例如单步执行）时，GDB 调试器都会自动帮我们打印出来，而 print 命令则不会
- 对于使用 display 命令查看的目标变量或表达式，都会被记录在一张列表（称为自动显示列表）中。通过执行 info dispaly 命令，可以打印出这张表

```bash
# print == p
(gdb) p 变量名
# 如果变量是一个整形, 默认对应的值是以10进制格式输出, 其他格式参考C语言的printf()格式
(gdb) p/fmt 变量名

# 打印变量类型
(gdb) ptype 变量名

# 在变量的有效取值范围内, 自动打印变量的值(设置一次, 以后就会自动显示)
(gdb) display 变量名
# 以指定的整形格式打印变量的值, 关于 fmt 的取值, 请参考 print 命令
(gdb) display/fmt 变量名

# 删除不需要再打印值的变量或表达式
# 命令中的 num 是通过 info display 得到的编号, 编号可以是一个或者多个
(gdb) undisplay num [num1 ...]
# num1 - numN 表示一个范围
(gdb) undisplay num1-numN
(gdb) delete display num [num1 ...]
(gdb) delete display num1-numN

# 禁用不需要再打印值的变量或表达式
# 命令中的 num 是通过 info display 得到的编号, 编号可以是一个或者多个
(gdb) disable display num [num1 ...]
# num1 - numN 表示一个范围
(gdb) disable display num1-numN

# 启用禁用的变量或表达式
# 命令中的 num 是通过 info display 得到的编号, 编号可以是一个或者多个
(gdb) enable  display num [num1 ...]
# num1 - numN 表示一个范围
(gdb) enable display num1-numN
```

```bash
# 单步调试
# 从当前代码行位置, 一次调试当前行下的每一行代码
# step == s
# 如果这一行是函数调用, 执行这个命令, 就可以进入到函数体的内部
(gdb) step

# 如果通过 s 单步调试进入到函数内部, 想要跳出这个函数体
(gdb) finish

# next == n
# 如果这一行是函数调用, 执行这个命令, 不会进入到函数体的内部
(gdb) next

# 直接跳出某个循环体，但必须满足 要跳出的循环体内部不能有有效的断点，必须要在循环体的开始/结束行执行该命令
(gdb) until

# 设置变量值
# 可以在循环中使用, 直接设置循环因子的值
# 假设某个变量的值在程序中==90的概率是5%, 这时候可以直接通过命令将这个变量值设置为90
(gdb) set var 变量名=值
```
