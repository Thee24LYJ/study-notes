[tree 命令详解（输出目录树层结构，显示目录和文件）-CSDN 博客](https://blog.csdn.net/weixin_46034375/article/details/125403748)
[强大的命令行工具：tree](https://zhuanlan.zhihu.com/p/144616508)
[linux 下 tree 命令中文字符乱码解决方案\_tree 命令 中文编码-CSDN 博客](https://blog.csdn.net/cxrsdn/article/details/100006348)

以树状结构打印出指定目录下的文件和文件夹

```bash
# 显示当前目录结构
$ tree
# 指定遍历层级深度
$ tree -L 2
# 只显示文件夹
$ tree -d
# 过滤无需显示的文件或文件夹
$ tree -I files

# 若打印出来乱码如\320\305\265\300\274\306\313\343\317\356\304\277则添加-N参数
$ tree -N
```
