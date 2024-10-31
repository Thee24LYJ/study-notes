# vivado 清理工程

vivado 工程结构：

- project_name.cache：Vivado 软件的运行缓存
- project_name.hw： 所有波形文件
- project_name.ip_user_files：用户关于 IP 的文件
- project_name.runs：编译与综合结果 ，impl_1 文件夹存取布线后结果
- project_name.sdk：SDK 环境代码，一般是 ZYNQ 设计中关于 PS 端的代码
- project_name.sim：仿真结果
- project_name.srcs：工程的源码、仿真文件与约束文件
- project_name.xpr： Vivado 工程启动文件

清理 vivado 工程的 TCL 命令：`reset_project` 和`reset_project -exclude ip`

`reset_project` 用于重置当前项目重置为初始状态，清除在综合，模拟，实现和 write_bitstream 过程中创建的所有输出文件,包括临时文件。但是要注意，这会清理所有的 IP 和缓存，如果是大工程的话，清理完后，第一次重新编译需要花费更多的时间

`reset_project -exclude ip` 可复位整个项目，但不会清理 IP 目录下的文件

`reset_sim` 清空项目的仿真目录下文件

[Vivado 如何清理工程，并避免缺失必要的文件？](https://zhuanlan.zhihu.com/p/622305660)
