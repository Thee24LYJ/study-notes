# Username is not in the sudoers file. This incident will be reported(用户名不在 sudoers 文件中。此事件将被报告)(username 为待添加的用户名)

### 解决办法 1：adduser 添加用户到 sudo 用户组中

```bash
// root用户执行
# adduser username sudo
```

### 解决办法 2：编辑 sudoers 文件

```bash
# vim /etc/sudoers
```

为普通用户添加 root 权限

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
username  ALL=(ALL:ALL) ALL
```

参考：

- [How to Fix “Username is not in the sudoers file. This incident will be reported” in Ubuntu](https://www.tecmint.com/fix-user-is-not-in-the-sudoers-file-the-incident-will-be-reported-ubuntu/)
- [20.04 - "user is not in the sudoers file. this incident will be reported" when trying to run as an admin - Ask Ubuntu](https://askubuntu.com/questions/1304390/user-is-not-in-the-sudoers-file-this-incident-will-be-reported-when-trying-to)
- [修复"用户名不在 sudoers 文件中。此事件将被报告"的问题 | Baeldung 中文网](https://www.baeldung-cn.com/linux/username-not-in-sudoers-file)

# Ubuntu 系统挂载 FAT32 的 U 盘后显示中文乱码

解决办法：

- 添加`-o utf8=1`参数(推荐)

```
$ sudo mount -o utf8=1  /dev/sdb1 /mnt/test
```

- 添加`-o iocharset=utf8`参数

```
$ sudo mount -o iocharset=utf8  /dev/sdb1 /mnt/test
```

参考：
[解决 FAT32 文件系统分区中文文件名在 Linux 下显示乱码\_fat32 lfn unicode-CSDN 博客](https://blog.csdn.net/chinalinuxzend/article/details/4299224)

# Linux root 用户使用 chmod 错误递归修改根目录导致的问题

> 问题描述：
> 使用 `chmod` 错误递归修改`/usr` 文件夹权限为 `777`，导致用户无法正常使用`sudo`命令获取 `root` 权限
>
> ```bash
> $ sudo chmod -R 777 /usr
> ```

> 使用 `sudo apt update` 等命令获取 `root` 权限报错：
> sudo: /usr/bin/sudo 必须属于用户 ID 0(的用户)并且设置 setuid 位
>
> 对于一些其他可能的潜在问题，后续遇到再补充

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
