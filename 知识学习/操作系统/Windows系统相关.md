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
