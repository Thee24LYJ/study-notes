# 安装 putty

### 在线安装

```bash
$ sudo apt install putty
```

### 离线安装

下载地址：
[Download PuTTY: latest release (0.82)](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

参考 README 安装 putty 即可

```bash
$ cmake .
$ sudo cmake --build . --target install
```

# 运行 putty

```bash
$ sudo putty
```

直接使用命令`putty`显示界面并配置和打开串口时报错权限不够，而使用`sudo putty`直接报错且无法显示界面，报错如下：

> MoTTY X11 proxy: Unsupported authorisation protocol
>
> (putty:14837): Gtk-WARNING \*\*: 13:07:17.162: cannot open display: localhost:10.0

解决办法：将当前用户家目录下的.Xauthority 文件复制到 root 用户目录下

```bash
$ sudo cp ~/.Xauthority /root/
```

除了使用命令`sudo putty`，还可以把普通用户加入 dialout 组，然后打开 putty 软件，就不用再加 sudo 了，命令如下：

```bash
$ sudo adduser username dialout  # username是用户自己的名称
```

参考：
[Linux 远程图形化界面出错：MoTTY X11 proxy: Unsupported authorisation protocol - 时间的风景 - 博客园](https://www.cnblogs.com/create-serenditipy/p/15407983.html)
[Linux 下使用 putty 进行 UART 串口调试-CSDN 博客](https://blog.csdn.net/happygrilclh/article/details/79806932)
[Linux 下常用的串口助手 —— minicom、putty、cutecom-CSDN 博客](https://blog.csdn.net/Mculover666/article/details/87647810)
