# 新建 Python 文件时自动添加相关注释

[pycharm 设置 python 文件模板，自动添加文件头注释](https://zhuanlan.zhihu.com/p/113896445)
[Python 的文件头与 Pycharm 模板 - 流光记 - 博客园](https://www.cnblogs.com/zhangyanlong/p/11297815.html)

```python
# -*- coding: utf-8 -*-#
"""
    @project: ${PROJECT_NAME}
    @Author：lyj
    @file： ${NAME}.py
    @date：${DATE} ${TIME}
    @Software: ${PRODUCT_NAME}
    @description:
"""
```

# pycharm 使用 ideavim 插件无法使用 Tab 进行多行缩进

[pycharm 中 ideavim 配置 Tab 缩进](https://zhuanlan.zhihu.com/p/622831329)

- 解决办法一

vim 中，“>”键可以实现单行缩进和多行缩进。

- 解决办法二

手动配置“Tab“键的映射，在.ideavimrc 中添加

```
vnoremap <Tab> ><CR>
nnoremap <Tab> ><CR>
```

# pycharm ideavim 插件启用相对行号

[Pycharm 启用相对行号（IdeaVim）\_idea 相对行号-CSDN 博客](https://blog.csdn.net/weixin_44627639/article/details/125961543)

在`.ideavimrc`文件中添加下面两行代码并保存(Windows 系统文件路径为`C:\用户\用户名\.ideavimrc`，Linux 和 MAC 系统路径为`~\.ideavimrc`)：

```
:set relativenumber
:set number
```

# pycharm 远程开发连接 WSL 项目报错如下：This IDE build has expired. Provide another build or select 'JetBrains Installer' from the installation options to install the latest version.

[【报错记录】this ide build has expired. provide another build or select 'jetbrains installer'](https://zhuanlan.zhihu.com/p/641334622)

建议重新安装其他版本的 IDE 并用该 IDE 打开

[Pycharm配置conda环境(解决新版本无法识别可执行文件问题)\_conda可执行文件-CSDN博客](https://blog.csdn.net/2401_84495872/article/details/139919853)
