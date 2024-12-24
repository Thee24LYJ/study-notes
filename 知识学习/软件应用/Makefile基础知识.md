[Makefile 基础学习 | Thee](https://www.theelyj.com/2022/12/28/Makefile%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0/)
[Makefile 基础 - Makefile 教程 - 廖雪峰的官方网站](https://liaoxuefeng.com/books/makefile/makefile-basic/index.html)
[不懂Makefile的程序员还敢写代码吗？](https://mp.weixin.qq.com/s/tKYuRfHfgnWvcq5VcDnrQw)

Makefile 是用来描述哪些文件需要编译、哪些文件需要重新编译的文件(实现文件编译和链接)，其格式如下：

```Makefile
目标... : 依赖文件集合...
	命令1
	命令2
	...
```