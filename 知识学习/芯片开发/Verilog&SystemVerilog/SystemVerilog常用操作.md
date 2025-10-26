# 文件读写操作

### 一、文件操作基础(打开关闭文件)

- **`$fopen`**：返回文件描述符（整数句柄），模式包括：
  - `"r"`：只读
  - `"w"`：写入（覆盖原文件）
  - `"a"`：追加
- **`$fclose`**：释放文件资源，避免句柄泄露。

```systemverilog
initial begin
    int fd = $fopen("data.txt", "r");  // 打开文件
    if (!fd) $error("文件打开失败");
    // ... 操作文件
    $fclose(fd);  // 关闭文件
end
```

### 二、文件读取方法

#### 1. **逐行读取：`$fgets`**

- 按行读取文本，适合处理配置文件或测试向量：

```systemverilog
initial begin
    int fd = $fopen("config.txt", "r");
    string line;
    while ($fgets(line, fd)) begin
        line = line.substr(0, line.len()-2); // 去除换行符（如存在）
        $display("Line: %s", line);
    end
    $fclose(fd);
end
```

#### 2. **格式化读取：`$fscanf`**

- 解析结构化数据（如 CSV）：

```systemverilog
initial begin
    int fd = $fopen("samples.csv", "r");
    int addr, data;
    while (!$feof(fd)) begin
        int code = $fscanf(fd, "%d,%d", addr, data);
        if (code < 2) break; // 检查成功读取的字段数
        $display("Addr=%0d, Data=%0d", addr, data);
    end
    $fclose(fd);
end
```

#### 3. **批量读取：`$readmemh`/`$readmemb`**

- 一次性加载十六进制/二进制数据到存储器数组：

```systemverilog
reg [7:0] mem [0:255]; // 256字节存储器
initial begin
    $readmemh("data.hex", mem); // 加载十六进制数据
    foreach (mem[i]) $display("mem[%0d] = %h", i, mem[i]);
end
```

### 三、文件写入方法

#### 1. **`$fdisplay` 与 `$fwrite`**

- `$fdisplay`：自动换行，适合结构化输出。
- `$fwrite`：不自动换行，需手动添加`\n`：

```systemverilog
initial begin
    int fd = $fopen("log.txt", "w");
    $fdisplay(fd, "Time: %0t", $time); // 带换行
    $fwrite(fd, "Data: ");
    for (int i=0; i<4; i++) $fwrite(fd, "%h ", i);
    $fdisplay(fd, ""); // 手动换行
    $fclose(fd);
end
```

#### 2. **二进制数据写入：`$fwrite` 二进制模式**

```systemverilog
bit [31:0] packet = 32'hDEADBEEF;
int fd = $fopen("packet.bin", "wb");
$fwrite(fd, "%u", packet); // 无格式二进制写入
$fclose(fd);
```

### 四、异常处理与高级技巧

1. **错误检测**

   - 检查`$fopen`返回值是否为 0（失败）。
   - 使用`$ferror`获取错误码：
     ```systemverilog
     if ($ferror(fd)) $error("Error code: %0d", $ferror(fd));
     ```

2. **缓冲区管理**

   - 为`$fgets`分配足够缓冲区（如 `string line = new[256]`）。

3. **性能优化**
   - 大型文件用`$readmemh`替代逐行读取。
   - 二进制模式比文本模式更快。

### 五、并发文件操作

多线程下需同步访问共享文件，防止数据竞争：

```systemverilog
semaphore sem = new(1); // 互斥锁
initial begin
    fork
        begin
            sem.wait();
            $fwrite(fd, "Thread1 Data");
            sem.post();
        end
        begin
            sem.wait();
            $fwrite(fd, "Thread2 Data");
            sem.post();
        end
    join
end

```

### 六、方法对比与选择指南

| **方法**          | **适用场景**          | **数据格式**    | **读取方式**  | **性能特点**   |
| ----------------- | --------------------- | --------------- | ------------- | -------------- |
| **`$fgets`**      | 配置文件/日志逐行解析 | 文本            | 逐行          | 中等           |
| **`$fscanf`**     | CSV 等结构化数据      | 格式化文本      | 逐行+字段解析 | 中等           |
| **`$readmemh/b`** | 测试向量/存储器初始化 | 十六进制/二进制 | 批量加载      | 高（大数据量） |
| **`$fdisplay`**   | 生成带换行的日志      | 文本            | 写入          | 中等           |
| **`$fwrite`**     | 无格式/二进制数据写入 | 二进制或文本    | 写入          | 高             |

### 总结

SystemVerilog 文件操作的核心要点：

1. **基础操作**：`$fopen`→ 读写 →`$fclose`流程是基础。
2. **读取选择**：逐行用`$fgets`，结构化解析用`$fscanf`，批量加载用`$readmemh`。
3. **写入控制**：换行需求选`$fdisplay`，无格式用`$fwrite`。
4. **并发安全**：信号量/互斥锁同步多线程写入。
5. **健壮性**：检查返回值、处理异常、管理缓冲区大小避免截断。

> 示例完整代码可参考：
> [深入 SystemVerilog：彻底理解文件读写机制的 4 个步骤 - CSDN 文库](https://wenku.csdn.net/column/67memza89s) > [SystemVerilog 文件操作：多线程并发读写的解决方案与最佳实践 - CSDN 文库](https://wenku.csdn.net/column/5bxt2dfzht)
