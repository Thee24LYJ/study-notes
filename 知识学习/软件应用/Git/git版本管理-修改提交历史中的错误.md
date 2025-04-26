> 最近使用 git 管理项目时，使用`git commit`提交时写错了提交信息但是没注意就继续再此基础上继续提交，因此写这篇文章记录如何解决。

# 示例情况

使用`git log --graph`输出如下，其中第二次提交打错单词为 seconddd，因此需要将其修改为 second

```
* commit a24503f587900f95f22b32556e2b3cd3956792cc (HEAD -> master)
| Author: Thee24LYJ <1579290423@qq.com>
| Date:   Wed Mar 12 18:47:19 2025 +0800
|
|     thrid commit by lyj.
|
* commit 4e1ccab835389cdec1d1bfc90ae9dc0cd4f58e8a
| Author: Thee24LYJ <1579290423@qq.com>
| Date:   Wed Mar 12 18:46:35 2025 +0800
|
|     seconddd commit by lyj
|
* commit 6957deaf6af12c838fcb06db78e49541f363f5bc
  Author: Thee24LYJ <1579290423@qq.com>
  Date:   Wed Mar 12 18:44:11 2025 +0800

      first commit test
```

# 解决方法

### 使用交互式变基`git rebase -i`

```git
# 假设要修改最近的第3个提交（HEAD~3）
$ git rebase -i HEAD~3
# 然后将需要修改信息的提交前的pick改为edit，保存并退出
# 退出后git会停留在待修改目标提交处，此时可以修改文件或提交信息
$ git add <修改的文件> # 若未修改可以不用执行
$ git commit --amend # 修改提交信息
# 继续完成变基
$  git rebase --continue
```

还有一种更加简便的方法如下：

```git
# 假设要修改最近的第3个提交（HEAD~3）
$ git rebase -i HEAD~3
# 然后将需要修改信息的提交前的pick改为reword，保存并退出后会进入目标提交的界面，此时可以直接修改提交信息，保存并退出后就修改完成
```

除此之外，对于最近一次提交信息修改可以直接执行下面代码进行修改：

```
# 修改提交信息（不修改文件内容）
git commit --amend -m "新的提交信息"

# 添加漏提交的文件或移除误提交的文件：
git add <漏提交的文件>      # 添加漏掉的文件
git rm <误提交的文件>      # 移除误提交的文件
git commit --amend --no-edit  # 保持原提交信息，仅修改内容
```
