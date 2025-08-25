# 资料网站

[SystemVerilog 教程 \| EasyFormal](https://easyformal.com/systemverilog/)
[SystemVerilog Tutorial](https://www.chipverify.com/tutorials/systemverilog)
[SV 学习笔记（一） – Wenhui's Rotten Pen](https://www.wenhui.space/docs/07-ic-verify/training/sv-study-note-one/)
[Welcome To SystemVerilog Central](https://www.asic-world.com/systemverilog/index.html)
[systemverilog.io - systemverilog.io](https://www.systemverilog.io/)
[UVM Systemverilog 国外学习网站 SemiWiki - All Things Semiconductor! - 掘金](https://juejin.cn/post/7024070651585511455)

# 相关知识点

[SystemVerilog 常用语法学习笔记](SystemVerilog常用语法学习笔记.md)
[SystemVerilog 验证学习笔记](SystemVerilog验证学习笔记.md)

### SV 参数传递

[systemverilog 参数传递 - 知乎](https://zhuanlan.zhihu.com/p/532237947)
[SystemVerilog 中的参数传递 - 李黑黑 - 博客园](https://www.cnblogs.com/holdendong/articles/18173760)
[system verilog 中的参数传递——ref，input，output_input output ref-CSDN 博客](https://blog.csdn.net/qq_41467882/article/details/121684326)

我们从函数或任务的形参类型来讨论，形参类型有 ref,input,output,inout。

形参类型为 ref 比较简单，可以认为函数或任务拿到这个变量就是实参变量本身。

形参类型为 input，会新建一个变量并且完成变量拷贝，也就把实参变量内容复制给新建变量。

形参类型为 output，只会新建一个变量，不会进行变量拷贝，这里里面有意思地方是，如果实参是数组队列，只会新建一个同等大小的空数组或空队列；而如果实参是类对象，那只会新建一个句柄，不会进行句柄的拷贝，网上有些例子报 error 啥的，就是因为只是这个新建的句柄没有指向任何对象，悬空状态。以上是 output 的第一个特性，它的第二个特性是函数或任务执行完后，实参原本的物理内存位置和内容会被释放掉，变为所创建形参的物理内存的位置和内容。这就可以实现实参句柄指向了形参句柄对象，因为实参句柄本身就已经变成了形参句柄。

形参类型为 inout，两个特性，一是新建变量和变量拷贝，二是函数或任务执行完后，实参原本的物理内存位置和内容会被释放掉，变为所创建形参的物理内存的位置和内容
