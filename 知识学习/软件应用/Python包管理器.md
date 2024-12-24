# conda 相关命令

[Conda 常用命令合集](https://zhuanlan.zhihu.com/p/363904808)

```bash
# 创建环境
$ conda create -n env_name
# 创建指定Python版本的环境
$ conda create -n env_name python=3.7
# 进入环境
$ conda activate env_name
# 退出环境
$ conda deactivate
# 删除环境
$ conda remove -n env_name --all
# 列出所有环境
$ conda env list
```

# pip 相关命令

[【Python】pip 安装库时存在缓存（及清除方法）-CSDN 博客](https://blog.csdn.net/qq_44921056/article/details/129857659)

```bash
# 清除pip所有缓存，包括已下载但未安装的软件包和已安装但未被使用的缓存
$ pip cache purge
```
