> 前言：
> 最近想使用 esp32 系列芯片实现对 HID 设备(如键盘、鼠标)的模拟，但是使用 PlatformIO 进行开发过程中遇到了一些问题，下面对其进行记录

# 环境配置

### 硬件

- esp32 devkit 开发板，芯片型号 ESP32-D0WD-V3

### Platform IO 配置

```
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
lib_deps = adafruit/Adafruit TinyUSB Library@^3.7.2
```

# 添加 TinyUSB 库然后编译 TinyUSB 例程报错

```
src/main.cpp:23:5: error: 'TUD_HID_REPORT_DESC_KEYBOARD' was not declared in this scope
     TUD_HID_REPORT_DESC_KEYBOARD()};
     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/main.cpp:23:5: note: suggested alternative: 'HID_USAGE_DESKTOP_KEYBOARD'
     TUD_HID_REPORT_DESC_KEYBOARD()};
     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
     HID_USAGE_DESKTOP_KEYBOARD
src/main.cpp:27:1: error: 'Adafruit_USBD_HID' does not name a type; did you mean 'Adafruit_USBH_CDC'?
 Adafruit_USBD_HID usb_hid;
 ^~~~~~~~~~~~~~~~~
 Adafruit_USBH_CDC
```

# 原因

ESP32 Devkit 开发板板载芯片为 esp32-D0WD 系列芯片，不支持 USB 模拟 HID 设备，只能支持蓝牙模拟 HID 设备，ESP32-S2 和 ESP32-S3 支持 USB 模拟 HID 设备

因此 ESP32 Devkit 开发板不支持直接模拟 USB 的 HID 设备，所以该开发板相关头文件中没有定义`CONFIG_USB_OTG_SUPPORTED`，导致在 TinyUSB 库的头文件 tusb_config_esp32.h 中只能定义`CFG_TUD_ENABLED`宏为 0，这就导致在 Adafruit_TinyUSB.h 中无法包含 HID 相关头文件，从而报错

![](https://s2.loli.net/2025/09/29/aljOPb4QdXHBFLG.png)
![](https://s2.loli.net/2025/09/29/5bRnXMH98gI37dS.png)

修改 esp32dev 开发板为支持直接模拟 HID 设备的其他开发板后，相关的宏都能找得到，编译没问题就没问题了

![](https://s2.loli.net/2025/09/29/Hclvn3sjQGUerL9.png)
![](https://s2.loli.net/2025/09/29/rfpTHF3ylsWGZBm.png)
