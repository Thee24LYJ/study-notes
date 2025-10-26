# 使用 Vim 插件在 insert 模式下输入一次中文句号、逗号等会出现两个的问题

[VS Code Vim 模式出现的中文插入与标点符号错误\_vscode 1.101.2 配合 vscode vim 使用在中文输入法下异常-CSDN 博客](https://blog.csdn.net/loyangel/article/details/148811251)

原因：在 VS Code v1.101.0 后，更新了 editor.experimentalEditContextEnabled，并将它默认开启，虽然不知道有什么用，但是与我们的 Vim 插件产生了冲突。将它关掉即可

解决：设置 -> 搜索 'editor.experimentalEditContextEnabled' -> 取消勾选

# 容易遗忘的快捷键

[VSCode 快捷键大全 \| 菜鸟教程](https://www.runoob.com/vscode/vscode-shortcut-keys.html)

下面根据自身需求进行自定义的快捷键进行标注如：Alt+\`(Ctrl+\`)，其中括号中的为原本的快捷键

下面的快捷键均为个人不容易记住的快捷键：

|           快捷键            |    功能    |
| :-------------------------: | :--------: |
|           Ctrl+F            |    查找    |
|           Ctrl+H            |    替换    |
|       Alt+\`(Ctrl+\`)       |  打开终端  |
| Ctrl+Shift+\`(Alt+Shift+\`) |  新建终端  |
|    Ctrl+PageUp/PageDown     |  切换终端  |
|         Ctrl+/1/2/3         | 拆分编辑器 |
|  Ctrl+Alt+U(Ctrl+Shift+U)   | 小写转大写 |
