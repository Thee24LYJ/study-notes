# Windows 系统 文件 hash 值计算

```bash
$ certutil -hashfile  <文件名>  <hash类型>
# 例如:
$ certutil -hashfile  test.txt sha256
```

参考：

[使用 Windows 10 自带工具 校验 MD5 SHA1 SHA256 类型文件](https://blog.csdn.net/ThinkAboutLife/article/details/110384620)

# Windows 系统查看文件被哪个进程占用

资源监视器 -> CPU 选项卡 -> 关联的句柄(输入文件名进行搜索)

[windows 查看文件被哪个进程占用](https://zhuanlan.zhihu.com/p/158119424)

# Windows 包管理器

[Windows 系统缺失的包管理器：Chocolatey、WinGet 和  Scoop - 少数派](https://sspai.com/post/65933)

[Windows 下包管理器 Scoop 的安装与使用 - Muxiner's Blog](https://muxiner.github.io/using-scoop/)

# Windows 弹出移动硬盘提示“该设备正在使用中”

解决办法：

通过`事件查看器->自定义视图->管理事件`找到占用的进程，然后结束掉该进程

[弹出 USB 大容量存储设备时出问题“该设备正在使用中”](https://zhuanlan.zhihu.com/p/424874015)

# 路由追踪

```cmd
> tracert 8.8.8.8
```
