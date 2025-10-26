[Hello 系列 | Shell 编程必备简明基础知识](https://mp.weixin.qq.com/s?__biz=Mzg5ODYzNDU4Nw==&mid=2247484642&idx=1&sn=51579feb6febf3d9cb8c0379da17f703&chksm=c05ec2d6f7294bc0dd3cc5354d2702c07b8405dee54b4428425cc5b53f5630c53820cfb16282&token=835109125&lang=zh_CN&scene=21#wechat_redirect)
[Shell 工具和脚本](https://missing-semester-cn.github.io/2020/shell-tools/)
[Linux Bash 常用技巧汇总](Linux%20Bash常用技巧汇总.md)

# 一、hello shell

```bash
#!/bin/bash

# 1.单行注释：打印信息
echo "hello shell"

:<<!
多行注释1
多行注释2
!

# 2.变量定义
name="lyj"
echo "my name is ${name}!"

# 3.数组定义
# 3.1 方式1
array=(0 1 2 3)
# 3.2 方式2
array2=(
0
1
2
3
)

# 访问数组中索引为3的元素
echo "array[3]=${array[3]}"
# 访问数组中所有元素
echo "array[3]=${array[@]}"

# 获取数组长度
echo "array len=${#array[@]}"# 方式一
echo "array len=${#array2[*]}"# 方式二

# shell花括号{}
convert image.{png,jpg}
# 会展开为
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# 会展开为
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# 也可以结合通配使用
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件

# 创建 foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h 这些文件
touch {foo,bar}/{a..h}
```

- **`#!`**  是一个约定标记，指定这个脚本使用哪个 shell 解释器执行。这里指明使用 bash 解释器，这也是最常用的 shell 解释器
- 变量定义**等号两边不能有空格**

# 二、输入输出

```bash
#!/bin/bash

### 1.输入
read -p "input a num:" num
read -p "input a str:" str

### 2.输出
# 2.1 方式1
echo "num=${num},str=${str}"
# 2.2 方式2 测试发现这里\n无法换行
printf "num=%d,str=%s\n" ${num} ${str}

### 3.输入参数获取
echo "输入参数的个数: $#"

if [ $# != 2 ]
then
    echo "输入参数的个数不为2，请重新输入"
else
    echo "第 1 个参数：$1";
    echo "第 2 个参数：$2";
fi
```

# 三、流程控制

```bash
#!/bin/bash

# 1.if判断
a=1
b=2

if [ ${a} == ${b} ]
then
    printf "a == b\n"
elif [ ${a} -lt ${b} ] 
then 
    printf "a < b\n"   # \n可以正常换行
else
    printf "a > b\n"
fi

# 2.for循环
for i in 1 2 3 4 5 6
do
    echo "i = ${i}"
done

# 3.case
read -p "input a num(1~3):" num
case ${num} in
    1)  printf "num = 1\n"
    ;;
    2)  printf "num = 2\n"
    ;;
    3)  printf "num = 3\n"
    ;;
    *)  printf "error\n"
    ;;
esac

# 4.使用Linux命令
# 4.1 方式一：使用()
echo "方式一：使用()"
for file in $(ls /home)   
do
    echo ${file}
done

# 4.2 方式二：使用``  -> 反引号
echo "方式二：使用\`\`"
for file in `ls /home`
do
    echo ${file}
done
```

- 使用 if 判断时，需要严格输入空格。if、中括号 [] 与其中间的代码  **`应该有空格隔开`**

# 四、运算符

```bash
#!/bin/bash

### 1.基础运算符
# expr 是一款表达式计算工具
a=1
b=2

# 加减乘除取余 这里是加
res=`expr ${a} + ${b}`  # 符号两边的空格不能省
echo "a + b : ${res}"

### 2.关系运算符
# -eq：等于。EQUAL的缩写。
# -ne：不等于。NOT EQUAL的缩写。
# -gt：大于。GREATER THAN的缩写。
# -lt：小于。LESS THAN的缩写
# -ge：大于等于。GREATER THAN OR EQUAL的缩写。
# -le：小于等于。LESS THAN OR EQUAL的缩写。

a=1
b=2

# 等于
if [ ${a} -eq ${b} ]   # 注意空格
then
   echo "a 等于 b"
fi

### 3.逻辑运算符
a=1 # 等号两边不能有空格
b=2

# 逻辑与
if [[ ${a} -lt 6 && ${b} -gt 6 ]]
# 注意：1、使用[[]] 2、注意空格
then
   echo "true"
else
   echo "false"
fi

### 4.字符串运算符
a="abc"
b="xyz"

# 字符串相等判断
if [ ${a} = ${b} ]
then
   echo "a 等于 b"
fi

# 字符串不相等判断
if [ ${a} != ${b} ]
then
   echo "a 不等于 b"
fi

# 字符串长度是否为0
if [ -z ${a} ]
then
   echo "字符串 ${a} 长度为 0"
fi

# 字符串长度是否为0
if [ -n "${a}" ]
then
   echo "字符串 ${a} 长度不为 0"
fi

# 字符串是否为空
if [ $a ]
then
   echo "字符串 ${a} 不为空"
fi
```

# 五、函数

```bash
#!/bin/bash

### 1.无参无返回值函数定义使用
# 函数定义
hello_func()
{
	printf "hello func\n"
}
# 函数调用
hello_func

### 2.无参有返回值函数定义使用
# 函数定义
add()
{
	read -p "input a num: " num1
	read -p "input a num: " num2
	sum=`expr ${num1} + ${num2}`
	return ${sum}
}

# 函数调用
add
# 函数返回值使用$?获取
echo "sum=$?"

### 3.有参有返回值函数定义使用
# 函数定义
add()
{
	echo "第 1 个参数：$1";
    echo "第 2 个参数：$2";
    sum=`expr $1 + $2`
    return ${sum}
}
# 函数调用及参数传递
add 2 3
echo "sum=$?"
```

- `$0` - 脚本名
- `$1`  到  `$9` - 脚本的参数。 `$1`  是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值
- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用  `sudo !!`  再尝试一次
- `$_` - 上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下  `Esc`  之后键入 . 来获取这个值

# 六、文件判断

```bash
#!/bin/bash

# -e filename 如果 filename存在，则为真
# -d filename 如果 filename为目录，则为真
# -f filename 如果 filename为常规文件，则为真
# -L filename 如果 filename为符号链接，则为真
# -r filename 如果 filename可读，则为真
# -w filename 如果 filename可写，则为真
# -x filename 如果 filename可执行，则为真
# -s filename 如果文件长度不为0，则为真
# -h filename 如果文件是软链接，则为真



file="hello.txt"

if [ -e ${file} ]
then
	echo "文件存在"
else
	echo "文件不存在，创建文件"
	$(touch ${file})
fi
```

补充：
使用`(...)`语法会创建一个子 shell 执行其中的语句，不会影响外面的 shell 脚本

# 七、其他

### 命令自动扩展

花括号中的每个字符都可以和之后（或之前）的字符串进行组合拼接，**注意花括号和其中的逗号不可以用空格分隔，否则会被认为是普通的字符串对待**

```bash
$ echo {1,2,3}file
file1 file2 file3
$ echo {txt,csv}{1,2,3}
txt1 txt2 txt3 csv1 csv2 csv3
$ echo {1,2,3}file
1file 2file 3file
$ echo /very/long/path/file{,.bak}
/very/long/path/file /very/long/path/file.bak

cp /very/long/path/file{,.bak}
# 给 file 复制一个叫做 file.bak 的副本
rm file{1,3,5}.txt
# 删除 file1.txt file3.txt file5.txt
mv *.{c,cpp} src/
# 将当前目录所有 .c 和 .cpp 为后缀的文件移入 src 文件夹
```

### shell 脚本执行方式

[Linux 中 source xxx.sh 和 ./xxx.sh 的区别](https://mp.weixin.qq.com/s/5PzO1ouvIozHjs84wku0lg)

常见的两种方式：`source xx.sh`和`./xx.sh`，这两种执行 shell 脚本方式的区别如下(最主要的区别在于脚本是否在**当前 Shell 进程**中运行，以及其修改是否**影响当前 Shell 环境**)：

|      特性对比      |                     `source xxx.sh`                     |                `./xxx.sh`                |
| :----------------: | :-----------------------------------------------------: | :--------------------------------------: |
|    **执行环境**    |                   **当前 Shell 进程**                   |          **新建子 Shell 进程**           |
| **影响当前 Shell** |          **是**（变量、函数、环境更改会保留）           | **否**（脚本中的更改随子进程结束而消失） |
|    **权限要求**    |                     只需**读**权限                      |     需要**可执行**权限 (`chmod +x`)      |
|   **脚本解释器**   | 忽略 Shebang 行 (如  `#!/bin/bash`)，始终使用当前 Shell |       由脚本首行的 Shebang 行指定        |
|    **典型用途**    |            加载配置、刷新环境变量、定义函数             |            运行独立脚本或程序            |
