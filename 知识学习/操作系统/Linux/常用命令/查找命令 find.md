[查找命令 \| 爱编程的大丙](https://subingwen.cn/linux/search/)

# 1.文件名 -name

# 2.文件类型 -type

# 3.文件大小 -size

-size 4k 表示的区间为 (4-1k，4k], 表示一个区间, 大于 3k,小于等于 4k
-size -4k: [0k, 4-1k], 表示一个区间, 大于等于 0 并且 小于等于 3k
-size +4k: (4k, 正无穷), 表示搜索大于 4k 的文件

# 4.同时执行其他操作

### 4.1 exec 参数

```bash
# 语法：
$ find 路径 参数 参数值 -exec shell命令 {} \;
```

### 4.2 ok 参数

该参数语法与 exec 相同，但是是交互式执行的

```bash
# 语法：
$ find 路径 参数 参数值 -ok shell命令 {} \;
```

### 4.3 xargs 命令

```bash
# 将 find 搜索的结果通过管道传递给后边的shell命令继续处理
$ find 路径 参数 参数值 | xargs shell命令
```
