[【问题记录】解决“命令行终端”和“Git Bash”操作本地 Git 仓库时出现 中文乱码 的问题！\_windows cmd git 乱码-CSDN 博客](https://blog.csdn.net/weixin_43729127/article/details/133185964)

# 一、问题描述

- `Windows terminal`中使用`git bash`使用`git init`新建仓库后，再使用`git status`查看状态信息，输出中文全是**反斜杠加数字的乱码**
- 非`window terminal`的 `git bash`终端中，情况相同

# 二、解决方法

### 2.1 Windows terminal 中的 git bash

```bash
$ git config --global core.quotepath off
$ git config --global gui.encoding utf-8
$ git config --global i18n.commit.encoding utf-8
$ git config --global i18n.logoutputencoding utf-8
```

### 2.2 非 Windows terminal 中的 git bash

Options -> Text -> Locale(zh_CN) -> Character set(UTF-8)
