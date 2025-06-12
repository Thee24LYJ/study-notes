[Window 平台 Git-Bash 的配置](https://lhajh.github.io/windows/git/2019/05/05/Window-platform-Git-Bash-configuration.html)

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

# 三、git 关联多个远程仓库

[git 实现本地仓库同时关联多个远程仓库（Gitee 和 GitHub）-CSDN 博客](https://blog.csdn.net/xiecheng1995/article/details/106570059)
[【GitHub、Gitee】同时配置 SSH Keys 详细教程\_github 和 gittee 同时绑定密钥-CSDN 博客](https://blog.csdn.net/qq_44723773/article/details/121892147)

```
# 关联多个远程
$ git remote add GitHub git@github.com:Thee24LYJ/LearningCode.git
$ git remote add Gitee git@gitee.com:lyj_thee/learning-code.git

# 远程推送
$ git push GitHub main
$ git push Gitee main
```

# 四、git pull 报错

[解决 git@github.com: Permission denied (publickey). fatal: Could not read from remote repository. Pleas-CSDN 博客](https://blog.csdn.net/W_317/article/details/106518894?fromshare=blogdetail&sharetype=blogdetail&sharerId=106518894&sharerefer=PC&sharesource=qq_45237293&sharefrom=from_link)

> git@github.com: Permission denied (publickey). fatal: Could not read from remote repository.
>
> Please make sure you have the correct access rights  
> and the repository exists.

解决办法：在本地电脑和 github 网站上重新配置 SSH Key

# 五、git 修改提交历史中的错误

[git 版本管理-修改提交历史中的错误](git版本管理-修改提交历史中的错误.md)

# 六、git pull 报错

> $ git pull origin main
> From github.com:Thee24LYJ/oled-ui-astra
>
> - branch main -> FETCH_HEAD
>   fatal: refusing to merge unrelated histories

解决办法：强制允许合并无关历史
使用 `--allow-unrelated-histories` 参数强制合并：

```bash
$ git pull origin main --allow-unrelated-histories
```
