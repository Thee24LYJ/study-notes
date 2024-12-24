> git 提交代码时需要写 commit message(提交说明)，通过该提交说明来说明本次提交的目的

[How to Write a Git Commit Message](https://cbea.ms/git-commit/)

# 1.提交说明作用

- 提供更多历史信息，方便快速浏览
- 通过`grep`命令筛选特定提交，便于快速查找信息
- 通过 commit 直接生成 change log

# 2.提交说明格式

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

> 其中第一行为 header(不可省略)，其他 body 和 footer 均可省略，`<BLANK LINE>`为空行(分隔行)

### 2.1 header

> header 包括 type(必需)、scope(可选)和 subject(必需)三个字段

##### 2.1.1 type

用于说明 commit 的类别，只允许使用下面 7 个标识：

- feat：新功能（feature）
- fix：修补 bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改 bug 的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动

如果 type 为`feat`和`fix`，则该 commit 将肯定出现在 Change log 中。其他情况建议不放入 Change log 中

##### 2.1.2 scope

用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。如果修改影响了不止一个`scope`，可以使用`*`代替

##### 2.1.3 subject

对该提交的简短描述

### 2.2 body

对本次 commit 的详细描述，可以分成多行

- 使用第一人称现在时
- 应该说明代码变动的动机，以及与以前行为的对比

### 2.3 footer

只用于以下两种情况：

- 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 BREAKING CHANGE 开头，后面是对变动的描述、以及变动理由和迁移方法。

- 关闭 Issue

如果当前 commit 针对某个 issue，那么可以在 Footer 部分关闭这个 issue

参考：[git commit 规范指南](https://segmentfault.com/a/1190000009048911)
