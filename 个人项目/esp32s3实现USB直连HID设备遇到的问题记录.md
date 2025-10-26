> 前言：最近购买了一块 esp32s3 devkitc-1 N16R8 开发板，想使用 esp32s3 自带的 USB-OTG 功能实现 USB 的 HID 设备，其中与串口芯片连接的 USB 口专门用作程序下载，另一个与 esp32s3 直连的 USB 口模拟 HID 设备，但实际测试发现并非想的那么简单，下面详细叙述相关测试记录

# 环境配置

### 硬件

esp32s3 devkitc-1 N16R8 开发板

### 软件

采用 arduino 框架开发，使用的库为 Adafruit_TinyUSB 库，使用的源码如下：

```cpp
#include "Adafruit_TinyUSB.h"

// Report ID
enum {
  RID_KEYBOARD = 1,
  RID_MOUSE,
  RID_CONSUMER_CONTROL, // Media, volume etc ..
};

// HID report descriptor using TinyUSB's template
uint8_t const desc_hid_report[] = {
    TUD_HID_REPORT_DESC_KEYBOARD(HID_REPORT_ID(RID_KEYBOARD)),
    TUD_HID_REPORT_DESC_MOUSE   (HID_REPORT_ID(RID_MOUSE)),
TUD_HID_REPORT_DESC_CONSUMER(HID_REPORT_ID(RID_CONSUMER_CONTROL))
};

// USB HID object.
Adafruit_USBD_HID usb_hid;

// the setup function runs once when you press reset or power the board
void setup() {
  // Manual begin() is required on core without built-in support e.g. mbed rp2040
  if (!TinyUSBDevice.isInitialized()) {
    TinyUSBDevice.begin(0);
  }

  Serial.begin(115200);

  // Set up HID
  usb_hid.setPollInterval(2);
  usb_hid.setReportDescriptor(desc_hid_report, sizeof(desc_hid_report));
  usb_hid.setStringDescriptor("TinyUSB HID Composite");
  usb_hid.begin();

  // If already enumerated, additional class driverr begin() e.g msc, hid, midi won't take effect until re-enumeration
  if (TinyUSBDevice.mounted()) {
    TinyUSBDevice.detach();
    delay(10);
    TinyUSBDevice.attach();
  }
  Serial.println("Adafruit TinyUSB HID Composite example");
}

void loop() {
  // not enumerated()/mounted() yet: nothing to do
  if (!TinyUSBDevice.mounted()) {
    Serial.println("Not Mounted!");
    return;
  }

  /*------------- Keyboard -------------*/
  if (usb_hid.ready()) {
    // use to send key release report
    static bool has_key = false;
      uint8_t keycode[6] = {0};
      keycode[0] = HID_KEY_A;
      usb_hid.keyboardReport(RID_KEYBOARD, 0, keycode);
      has_key = true;
      usb_hid.keyboardRelease(RID_KEYBOARD);
  }
  delay(1000);
}
```

# 开发平台

### arduino 软件

参考：
[ESP32-S3 not appearing as a MIDI device · Issue #279 · adafruit/Adafruit_TinyUSB_Arduino](https://github.com/adafruit/Adafruit_TinyUSB_Arduino/issues/279#issuecomment-1692804299)
[panic at \`usb.begin()\` · Issue #394 · adafruit/Adafruit_TinyUSB_Arduino · GitHub](https://github.com/adafruit/Adafruit_TinyUSB_Arduino/issues/394#issuecomment-1960124957)
[Esp32-S3 TinyUSB config not working - Development Platforms - PlatformIO Community](https://community.platformio.org/t/esp32-s3-tinyusb-config-not-working/34209)
[Arduino IDE 中 ESP32S3 运行参数意义 - 哔哩哔哩](https://www.bilibili.com/opus/833077798602014787)
[解释 ESP32S3 红框配置 - 豆包](https://www.doubao.com/thread/w3728a7bfc412975d)

##### 配置 1(见下图)

采用两个 USB 端口，一个 USB 作为串口通信，一个 USB 作为 HID 设备，编译下载上述代码后实际发现两个 USB 都被电脑识别成串口，因为作为 HID 设备的 USB 进行 mount 操作时失败
![](https://s2.loli.net/2025/10/18/KPDUkxmai9Q18uo.png)
编译信息如下：

```
xtensa-esp32s3-elf-g++ -c -w -Os -Werror=return-type -w -x c++ -E -CC -DF_CPU=240000000L -DARDUINO=10607 -DARDUINO_ESP32S3_DEV -DARDUINO_ARCH_ESP32 -DARDUINO_BOARD="ESP32S3_DEV" -DARDUINO_VARIANT="esp32s3" -DARDUINO_PARTITION_default -DARDUINO_HOST_OS="windows" -DARDUINO_FQBN="esp32:esp32:esp32s3:UploadSpeed=921600,USBMode=hwcdc,CDCOnBoot=default,MSCOnBoot=default,DFUOnBoot=default,UploadMode=default,CPUFreq=240,FlashMode=qio,FlashSize=16M,PartitionScheme=default,DebugLevel=none,PSRAM=disabled,LoopCore=1,EventsCore=1,EraseFlash=none,JTAGAdapter=default,ZigbeeMode=default" -DESP32=ESP32 -DCORE_DEBUG_LEVEL=0 -DARDUINO_RUNNING_CORE=1 -DARDUINO_EVENT_RUNNING_CORE=1 -DARDUINO_USB_MODE=1 -DARDUINO_USB_CDC_ON_BOOT=0 -DARDUINO_USB_MSC_ON_BOOT=0 -DARDUINO_USB_DFU_ON_BOOT=0
```

##### 配置 2(见下图)

只采用与 esp32s3 直连的那个 USB 端口，通过该 USB 编译下载程序后显示有一个串口，同时也有 HID 设备
![](https://s2.loli.net/2025/10/18/8GJrb4RIlYzPQc5.png)
编译信息如下：

```
xtensa-esp32s3-elf-g++ -c -w -Os -Werror=return-type -w -x c++ -E -CC -DF_CPU=240000000L -DARDUINO=10607 -DARDUINO_ESP32S3_DEV -DARDUINO_ARCH_ESP32 -DARDUINO_BOARD="ESP32S3_DEV" -DARDUINO_VARIANT="esp32s3" -DARDUINO_PARTITION_default -DARDUINO_HOST_OS="windows" -DARDUINO_FQBN="esp32:esp32:esp32s3:UploadSpeed=921600,USBMode=default,CDCOnBoot=cdc,MSCOnBoot=default,DFUOnBoot=default,UploadMode=cdc,CPUFreq=240,FlashMode=qio,FlashSize=16M,PartitionScheme=default,DebugLevel=none,PSRAM=disabled,LoopCore=1,EventsCore=1,EraseFlash=none,JTAGAdapter=default,ZigbeeMode=default" -DESP32=ESP32 -DCORE_DEBUG_LEVEL=0 -DARDUINO_RUNNING_CORE=1 -DARDUINO_EVENT_RUNNING_CORE=1 -DARDUINO_USB_MODE=0 -DARDUINO_USB_CDC_ON_BOOT=1 -DARDUINO_USB_MSC_ON_BOOT=0 -DARDUINO_USB_DFU_ON_BOOT=0
```

##### 总结

两者区别是配置 1 目的是想采用与 USB 转串口芯片连接的 USB 进行串口通信，另外一个与 esp32s3 直连的 USB 作为 HID 设备，配置 2 目的是只采用与 esp32s3 直连的 USB 同时进行串口通信与 HID 设备，但实际情况发现配置 1 并未达到目的，目前暂不清楚为什么，不过配置 2 只通过一个 USB 进行串口与 HID 设备倒是可以

### VScode+PlatformIO 平台

参考：
[VS Code+platformio 配置 ESP32-S3-N16R8（8MB PSRAM + 16MB FLASH）工程 - Macrored - 博客园](https://www.cnblogs.com/macrored/p/17357581.html)
[arduino-esp32/boards.txt at 72c41d09538663ebef80d29eb986cd5bc3395c2d · espressif/arduino-esp32 · GitHub](https://github.com/espressif/arduino-esp32/blob/72c41d09538663ebef80d29eb986cd5bc3395c2d/boards.txt#L144C1-L145)

##### Platform 配置

```ini
[env:4d_systems_esp32s3_gen4_r8n16]
platform = espressif32
board = 4d_systems_esp32s3_gen4_r8n16
framework = arduino
monitor_speed = 115200
; 配置1
build_flags =
	-DARDUINO_USB_MODE=0
	-DARDUINO_USB_CDC_ON_BOOT=0
; 配置2
;build_flags =
;	-DARDUINO_USB_MODE=0
;	-DARDUINO_USB_CDC_ON_BOOT=1
; 配置3
;build_flags =
;	-DARDUINO_USB_MODE=1
;	-DARDUINO_USB_CDC_ON_BOOT=0
; 配置4
;build_flags =
;	-DARDUINO_USB_MODE=1
;	-DARDUINO_USB_CDC_ON_BOOT=1
lib_deps = adafruit/Adafruit TinyUSB Library@^3.7.3
```

- 对于 Platform 平台，不添加 build_flags 时默认为配置 4(`-DARDUINO_USB_MODE=1和-DARDUINO_USB_CDC_ON_BOOT=1`)，默认情况编译没有警告和报错；配置 1 和 3 编译不会报错(但有些宏会出现重定义警告)，通过与 USB 转串口芯片连接的 USB 接口上传程序，再把另一个直连 esp32s3 芯片的 USB 口接上电脑复位芯片后发现一直无法挂载，`TinyUSBDevice.mounted()`返回均为 0，这时设备管理器显示存在两个串口，HID 设备无法被识别到；对于配置 2，编译则会报重定义错误，即`.platformio\packages\framework-arduinoespressif32\tools\sdk\esp32s3\lib\libarduino_tinyusb.a(usbd.c.obj)`和 Adafruit TinyUSB Library 库的 `src/device/usbd.c` 有同名的函数或宏定义
- 前面都是采用的 Adafruit TinyUSB Library 库实现 HID 设备的模拟，若令`-DARDUINO_USB_MODE=0` 表示使用 USB-OTG (TinyUSB)内置的 tinyUSB，但是实际测试发现不起作用，若使用 esp32s3 的 arduino 框架自带的 tinyUSB 就没有问题，相关配置如下：

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200

; 关键配置：启用 USB 模式为 "Hardware CDC and JTAG"
board_build.usb_mode = hardware_cdc
; 或（如果需更灵活控制） 启用 USB 模式为 "TinyUSB"
; board_build.usb_mode = tinyusb
; 启用 USB HID 功能
board_build.usb_hid = enabled
```

# 总结

- 使用 arduino 软件时，目前测试发现只能通过与 esp32s3 直连的 USB 端口同时模拟串口和 HID 设备(即只使用一个 USB 端口)
- 使用 VScode+Platform 平台时，目前测试发现只能通过两个 USB 端口分别实现串口与 HID 设备的模拟(必须使用两个 USB 端口)

可能还有需要配置的地方没有配置好导致上述问题，但现在可以正常使用 esp32s3 devkitc-1 N16R8 开发板通过 USB 模拟 HID 设备了，后续有机会再来探索吧
