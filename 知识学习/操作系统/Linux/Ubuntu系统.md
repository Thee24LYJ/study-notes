# Username is not in the sudoers file. This incident will be reported(用户名不在 sudoers 文件中。此事件将被报告)

### 解决办法 1：adduser添加用户到sudo用户组中

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
