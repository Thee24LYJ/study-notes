使用命令读取芯片信息报错如下：

```
A fatal error occurred: Failed to connect to Espressif device: No serial data received.
For troubleshooting steps visit: https://docs.espressif.com/projects/esptool/en/latest/troubleshooting.html

Command returned with error code 2
```

解决办法：**手动进入下载模式**

- **拉低 BOOT 引脚**（按住开发板的 BOOT 按键）
- **短按 EN/RST 引脚**（复位开发板）
- **松开 BOOT 键**后立即开始烧录

这是因为部分开发板自动下载电路时序不稳定，需手动触发进入下载模式

# ESP32-CAM 下载 micropython 固件

[ESP32-CAM 应用（Micropython+Thonny）-CSDN 博客](https://blog.csdn.net/qiaoen_python/article/details/142726613)

esp32-cam 连接方式：

|    USB 转 TTL 模块     | ESP32-CAM 模块 |
| :--------------------: | :------------: |
|           5V           |       5V       |
|        TXD/UOT         |      RXD       |
|        RXD/UOR         |      TXD       |
|          GND           |      GND       |
| ESP32-CAM 的 IO0(特殊) |      GND       |

下载完成后拔下 IO0 接 GND 的那根线，按下复位键或重新上电即可正常运行

ESP32-CAM 的 **IO0 引脚在下载时需接到 GND** 的原因与芯片的启动模式选择机制密切相关。

### **1. 启动模式与 GPIO0 的特殊功能**

ESP32 芯片在启动时会根据 **GPIO0 的电平状态** 决定进入哪种模式：

- **GPIO0 为高电平（默认）**：芯片进入**正常启动模式**，执行闪存中的已有程序。
- **GPIO0 为低电平（接 GND）**：芯片进入**下载模式**，允许通过串口烧录新固件

ESP32-CAM 的自动下载电路依赖这一机制，通过拉低 GPIO0 电平强制芯片进入固件上传状态

### **2. 硬件设计需求**

- **无内置 USB 接口**：ESP32-CAM 本身没有 USB 转串口芯片，需通过外部工具（如 FTDI 编程器或 CH340 模块）连接电脑。此时，**手动控制 GPIO0 电平是进入下载模式的唯一方式**
- **自动下载电路的局限性**：部分 ESP32-CAM 开发板的自动下载电路可能无法稳定触发时序，需手动干预 GPIO0 以确保可靠进入下载模式

### **3. 典型操作流程**

1. **烧录前**：将 GPIO0 短接到 GND，同时按下复位（RST）键，芯片进入下载模式。
2. **烧录时**：保持 GPIO0 接地，通过串口工具上传代码。
3. **烧录后**：断开 GPIO0 与 GND 的连接，复位芯片以正常启动新程序

若未正确接地，芯片会跳过下载模式，直接运行旧程序，导致烧录失败

### **4. 与 BOOT 引脚的关系**

- **BOOT 引脚定义**：不同 ESP32 型号的 BOOT 引脚可能对应不同 GPIO（如 ESP32-C3 为 GPIO9，ESP32-WROOM 为 GPIO0）。对于 ESP32-CAM（通常基于 ESP32-S 芯片），**BOOT 引脚即 GPIO0**，因此需通过该引脚控制启动模式
- **硬件设计冲突**：部分 ESP32-CAM 开发板的 GPIO0 与摄像头模块的时钟信号线复用，若未正确拉低，可能导致硬件冲突
