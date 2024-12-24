[Git 基础 - git tag 一文真正的搞懂 git 标签的使用-CSDN 博客](https://blog.csdn.net/qq_39505245/article/details/124705850)
[Pro Git 中文版](https://git-scm.com/book/zh/v2)
[版本控制(Git)](https://missing-semester-cn.github.io/2020/version-control/)

# 一、git tag

- 查看已有的标签

```bash
$ git tag
```

- 查看标签提交信息

```bash
$ git show <标签名>
```

- 创建标签

```bash
# 直接给当前提交版本创建轻量标签
$ git tag <标签名>
# 给指定提交版本创建轻量标签
$ git tag <标签名> <提交版本号>
# 给当前提交版本创建附注标签
$ git tag -a <标签名> -m <附注信息>
# 给指定提交版本创建附注标签
$ git tag -a <标签名> <提交版本号> -m <附注信息>
```

- 删除标签

```bash
$ git tag -d <标签名>
```

# 二、git reset/revert

[Git 恢复之前版本的两种方法 reset、revert（图文详解）-CSDN 博客](https://blog.csdn.net/yxlshk/article/details/79944535)

- `git reset`：恢复之前某个提交的版本且该版本之后的提交都不需要时使用
- `git revert`：恢复之前某个提交的版本且该版本之后的提交仍需要时使用
