"串口通信中 rts 和 dtr 分别是什么意思，..."点击查看元宝的回答
https://yuanbao.tencent.com/bot/app/share/chat/TtkIPSOjdkWK
"串口通信只连接了 TX 和 RX 信号线，此时有..."点击查看元宝的回答
https://yuanbao.tencent.com/bot/app/share/chat/RxDTsJJcAbh9
[理解 RTS 和 DTR：串口通讯中的重要概念 - 技象科技](https://www.techphant.cn/blog/76891.html)

# PlatformIO 下载 esp32-cam 网络摄像头程序后 monitor 无法接收数据

> 现象描述：esp32-cam 开发板下载完成网络摄像头程序并复位后，打开的 monitor 并没有打印出信息，但若是使用 arduino 的监视器或者其他串口通信软件在开发板复位时则能打印出相关信息

> 原因：在 PlatformIO monitor 中查看串口相关配置信息，发现其中 DTR 和 RTS 默认为 activate 即为高电平，而串口通信软件中默认未勾选 DTR 和 RTS 即为低电平，由于勾选了 DTR 和 RTS 后，软件会持续拉高对应信号线的电平。对于 RTS 来说，主机会持续向外设发送"我已准备好接收数据"的信号，对于 DTR 来说，主机向外设宣告“我已上电就绪，可通信”
>
> 对于很多 USB 转 TTL 串口线（例如常用的 CH340G、CP2102、FT232RL 等芯片的模块）虽然只引出了 TX、RX、GND 三根线，但其内部的 ​**​DTR/RTS​**​ 信号 ​**​ 被巧妙地用于自动控制复位电路 ​**​
> **​ 工作原理：​**​ 当用 Arduino IDE、STM32 的串口 ISP 工具等给单片机（如 Arduino, ESP8266, STM32）下载程序时，软件会 ​**​ 自动控制 DTR 和 RTS 的电平变化 ​**​，通过与门电路或电容耦合，在单片机复位引脚上产生一个特定的脉冲，使其自动进入 Bootloader 模式，从而实现“一键下载”

> 解决办法：在 platformio.ini 中添加如下两行代码(强制 RTS 和 DTR 端口为低电平，再次下载程序复位后就能在 monitor 中打印出相关信息)：
>
> ```
> monitor_rts = 0
> monitor_dtr = 0
> ```
