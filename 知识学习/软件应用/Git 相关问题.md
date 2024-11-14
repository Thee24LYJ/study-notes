# 一、Git Bash 编码问题

[【问题记录】解决“命令行终端”和“Git Bash”操作本地 Git 仓库时出现 中文乱码 的问题！\_windows cmd git 乱码-CSDN 博客](https://blog.csdn.net/weixin_43729127/article/details/133185964)

- `Windows terminal`中使用`git bash`使用`git init`新建仓库后，再使用`git status`查看状态信息，输出中文全是**反斜杠加数字的乱码**
- 非`window terminal`的 `git bash`终端中，情况相同

### 解决方法

##### 方法 1：Windows terminal 中的 git bash

```bash
$ git config --global core.quotepath off
$ git config --global gui.encoding utf-8
$ git config --global i18n.commit.encoding utf-8
$ git config --global i18n.logoutputencoding utf-8
```

##### 方法 2 非 Windows terminal 中的 git bash

Options -> Text -> Locale(zh_CN) -> Character set(UTF-8)

# 二、git status 显示文件被修改单实际并未修改

[文件没有更改，但 git status 显示 modified-CSDN 博客](https://blog.csdn.net/Lekaor/article/details/132804005)
[多平台同步：Windows 与 Mac/Linux 中 GIT 管理冲突问题](https://zhuanlan.zhihu.com/p/5847530280?utm_psn=1838573586689961984)

- 使用 git 管理文件时发现对比修改前和修改后的文件实际并没有改动的地方，但使用 git status 命令显示文件已被修改(==Windows 和 Linux 不同系统换行符导致的问题==)

### 解决方法

##### 方法 1：若是因为文件权限导致的则可使用`git config core.filemode false`关闭文件权限检测功能

##### 方法 2：若是因为 Windows 和 Linux 平台的换行符 LF/CRLF 导致的则使用`git config core.autocrlf false`
