> 问题描述：
> 使用 `chmod` 错误递归修改`/usr` 文件夹权限为 `777`
>
> ```bash
> $ sudo chmod -R 777 /usr
> ```

# 1.root 权限问题

使用 sudo apt update 等命令获取 root 权限报错：
sudo: /usr/bin/sudo 必须属于用户 ID 0(的用户)并且设置 setuid 位

解决办法：

1.首先进入恢复模式

vmware 虚拟机重启后，等待 Vmware 加载进度条加载完后，按 esc 进入 GNU GRUB 界面，选择并进入**Ubuntu 的高级选项**，然后进入恢复模式，这里选择进入 Ubuntu, with Linux 5.4.0-196-generic (recovery mode)

2.然后进入 root，这里选择 root Drop to root shell prompt，然后就获得了 root 权限

3.在 root 权限下执行下面几行代码，修复 sudo 命令报错问题

```bash
# 恢复模式下 root权限
# chmod -R 755 /usr
# chmod 4755 /usr/bin/sudo
# reboot
```

参考：

[详解在 Ubuntu 中引导到救援模式或紧急模式](https://linux.cn/article-14709-1.html)

[linux sudo 命令失败 提示 sudo：/usr/bin/sudo 必须属于用户 ID 0(的用户)并且设置 setuid 位](https://www.cnblogs.com/chxwkx/p/10686864.html)
