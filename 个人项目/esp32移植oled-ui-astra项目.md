> 前言：本项目的目的是为了在 esp32 平台上移植开源 OLED 项目 oled-ui-astra，详见[GitHub - AstraThreshold/oled-ui-astra: A smooth, easy-to-deploy, and easy-to-extend OLED UI framework, based on C++.](https://github.com/AstraThreshold/oled-ui-astra)，并添加旋转编码器 KY-040 实现 UI 界面的简单切换。
> 尽管笔者已经在本地环境测试没有问题，但是不能保证大家使用后也能正常工作，这里笔者尽量把移植 oled-ui-astra 项目的过程讲清楚，同时把已经测试过没有问题的整体工程放到这里[GitHub - Thee24LYJ/oled-ui-astra at esp32](https://github.com/Thee24LYJ/oled-ui-astra/tree/esp32)，仅供参考。下面正式开始。

# 1.硬件环境

### 1.1 硬件设备

`esp32 DevKit`、`7` 针 `SPI OLED` 和旋转编码器 `KY-040`

### 1.2 硬件连线

`ESP32` 采用硬件 `SPI` 与 `OLED` 进行通信

|      OLED      |           ESP32            |
| :------------: | :------------------------: |
| SCLK (CLK)/D0  |          GPIO 18           |
| MOSI (DIN)/D1  |          GPIO 23           |
|   CS (片选)    |     GPIO 5 (可自定义)      |
| DC (数据/命令) |     GPIO 16 (可自定义)     |
|   RES (复位)   | GPIO 17 (可自定义，可选接) |
|      VCC       |            3.3V            |
|      GND       |            GND             |

| KY-040 |       ESP32       |
| :----: | :---------------: |
|  CLK   | GPIO 4 (可自定义) |
|   DT   | GPIO 2 (可自定义) |
| VCC/+  |       3.3V        |
|  GND   |        GND        |

# 2.部署 oled-ui-astra 项目

官方部署教程：[部署教程 ‐ Deployment Tutorial · AstraThreshold/oled-ui-astra Wiki · GitHub](https://github.com/AstraThreshold/oled-ui-astra/wiki/%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B-%E2%80%90-Deployment-Tutorial#%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

> 由于笔者使用的开发环境是 `VScode+PlatformIO`，因此与官方部署教程存在差异

### 2.1 创建 PlatformIO 工程

这里开发板选择 `Espressif ESP32 Dev Module`，框架选择 `Arduino`，工程位置可默认也可自定义

![](https://s2.loli.net/2025/04/26/PqrlNcTSefJgOpV.png)

### 2.2 添加依赖库

在项目工程中的`platformio.ini`最后添加如下内容，其中`olikraus/U8g2@^2.35.4`是用于驱动`OLED`显示屏的`u8g2`库，`madhephaestus/ESP32Encoder@^0.11.7`是用于驱动`KY-040`的`ESP32Encoder`库

```ini
lib_deps =
    olikraus/U8g2@^2.35.4
    madhephaestus/ESP32Encoder@^0.11.7
```

### 2.3 添加 oled-ui-astra 源码到工程中

- 首先，从[Release astra UI v0.0.2-alpha · AstraThreshold/oled-ui-astra · GitHub](https://github.com/AstraThreshold/oled-ui-astra/releases/tag/v0.0.2-alpha)下载`astra-ui-v0-0-2-alpha.zip`源码，然后解压放到工程中的`lib`文件夹中，然后将`astra_rocket.h`  和  `astra_logo.h`移动到`include`文件夹中，移动`astra_rocket.cpp`和  `astra_logo.cpp`到`src`文件夹中，如下图所示

![](https://s2.loli.net/2025/04/26/7fwv8HENABnUrbM.png)

- 然后，修改一些头文件和源文件代码
  - 修改`include\astra_logo.h`中头文件包含`#include "../hal/hal.h"`为`#include "hal/hal.h"`
  - 修改`include\astra_rocket.h`中头文件包含`#include "../astra/ui/launcher.h"`为`#include "astra/ui/launcher.h"`，修改`#include "../hal/hal_dreamCore/hal_dreamCore.h"`为`#include "my_hal.h"`
  - 修改`lib\astra-ui-v0-0-2-alpha\astra\config\config.h`中头文件包含`#include "../../hal/hal_dreamCore/components/oled/graph_lib/u8g2/u8g2.h"`为`#include <U8g2lib.h>`

* 最后，在`include`文件夹中新建子类头文件`my_hal.h`，在`src`文件夹中新建子类源文件`my_hal.cpp`，定义继承自`HAL`类的子类为`MyHAL`，具体实现代码如下

```cpp
// MyHAL.h
#ifndef MYHAL_H_
#define MYHAL_H_

#include "hal/hal.h"

class MyHAL : public HAL
{
public:
    MyHAL() = default;
    void init() override
    {
        _u8g2_init();
    }

private:
    void _u8g2_init();

protected:
    u8g2_t canvasBuffer{};

public:
    void _screenOn() override;
    void _screenOff() override;

    void *_getCanvasBuffer() override;
    uint8_t _getBufferTileHeight() override;
    uint8_t _getBufferTileWidth() override;
    void _canvasUpdate() override;
    void _canvasClear() override;
    void _setFont(const uint8_t *_font) override;
    uint8_t _getFontWidth(std::string &_text) override;
    uint8_t _getFontHeight() override;
    void _setDrawType(uint8_t _type) override;
    void _drawPixel(float _x, float _y) override;
    void _drawEnglish(float _x, float _y, const std::string &_text) override;
    void _drawChinese(float _x, float _y, const std::string &_text) override;
    void _drawVDottedLine(float _x, float _y, float _h) override;
    void _drawHDottedLine(float _x, float _y, float _l) override;
    void _drawVLine(float _x, float _y, float _h) override;
    void _drawHLine(float _x, float _y, float _l) override;
    void _drawBMP(float _x, float _y, float _w, float _h, const uint8_t *_bitMap) override;
    void _drawBox(float _x, float _y, float _w, float _h) override;
    void _drawRBox(float _x, float _y, float _w, float _h, float _r) override;
    void _drawFrame(float _x, float _y, float _w, float _h) override;
    void _drawRFrame(float _x, float _y, float _w, float _h, float _r) override;

public:
    void _delay(unsigned long _mill) override;
    // unsigned long _millis() override;
    // unsigned long _getTick() override;
    unsigned long _getRandomSeed() override;

    // void _updateConfig() override;
};

#endif
```

```cpp
// my_hal.cpp
#include <U8g2lib.h>
#include "my_hal.h"

void MyHAL::_delay(unsigned long _mill)
{
    delayMicroseconds(_mill * 1000);
}

void MyHAL::_u8g2_init()
{
    // esp32 硬件SPI
    u8g2_Setup_ssd1306_128x64_noname_f(&canvasBuffer, U8G2_R0, u8x8_byte_arduino_hw_spi, u8x8_gpio_and_delay_arduino);
    u8x8_SetPin_4Wire_HW_SPI((u8x8_t *)&canvasBuffer, /* cs=*/5, /* dc=*/16, /* reset=*/17);
    u8g2_InitDisplay(&canvasBuffer);
    u8g2_ClearBuffer(&canvasBuffer);
    u8g2_SetPowerSave(&canvasBuffer, 0);
    u8g2_SetFontMode(&canvasBuffer, 1);      /*字体模式选择*/
    u8g2_SetFontDirection(&canvasBuffer, 0); /*字体方向选择*/
    u8g2_SetFont(&canvasBuffer, u8g2_font_wqy12_t_gb2312);
}

void MyHAL::_screenOn()
{
    u8g2_SetPowerSave(&canvasBuffer, 0);
}
void MyHAL::_screenOff()
{
    u8g2_SetPowerSave(&canvasBuffer, 1);
}
void *MyHAL::_getCanvasBuffer()
{
    return u8g2_GetBufferPtr(&canvasBuffer);
}
uint8_t MyHAL::_getBufferTileHeight()
{
    return u8g2_GetBufferTileHeight(&canvasBuffer);
}
uint8_t MyHAL::_getBufferTileWidth()
{
    return u8g2_GetBufferTileWidth(&canvasBuffer);
}

void MyHAL::_canvasClear()
{
    u8g2_ClearBuffer(&canvasBuffer);
}

void MyHAL::_canvasUpdate()
{
    u8g2_SendBuffer(&canvasBuffer);
}
void MyHAL::_setFont(const uint8_t *_font)
{
    u8g2_SetFontMode(&canvasBuffer, 1);      /*字体模式选择*/
    u8g2_SetFontDirection(&canvasBuffer, 0); /*字体方向选择*/
    u8g2_SetFont(&canvasBuffer, _font);
}

uint8_t MyHAL::_getFontWidth(std::string &_text)
{
    return u8g2_GetUTF8Width(&canvasBuffer, _text.c_str());
}
uint8_t MyHAL::_getFontHeight()
{
    return u8g2_GetMaxCharHeight(&canvasBuffer);
}

void MyHAL::_setDrawType(uint8_t _type)
{
    u8g2_SetDrawColor(&canvasBuffer, _type);
}

void MyHAL::_drawPixel(float _x, float _y)
{
    u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y));
}

void MyHAL::_drawEnglish(float _x, float _y, const std::string &_text)
{
    u8g2_DrawUTF8(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), _text.c_str());
    // u8g2_DrawStr(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), _text.c_str());
}

void MyHAL::_drawChinese(float _x, float _y, const std::string &_text)
{
    u8g2_DrawUTF8(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), _text.c_str());
}

void MyHAL::_drawBMP(float _x, float _y, float _w, float _h, const uint8_t *_bitMap)
{
    u8g2_DrawXBMP(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (int16_t)std::round(_w), (int16_t)std::round(_h), _bitMap);
}

void MyHAL::_drawBox(float _x, float _y, float _w, float _h)
{
    u8g2_DrawBox(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_w), (uint8_t)std::round(_h));
}

void MyHAL::_drawRBox(float _x, float _y, float _w, float _h, float _r)
{
    u8g2_DrawRBox(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_w), (uint8_t)std::round(_h), (uint8_t)std::round(_r));
}

void MyHAL::_drawFrame(float _x, float _y, float _w, float _h)
{
    u8g2_DrawFrame(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_w), (uint8_t)std::round(_h));
}

void MyHAL::_drawRFrame(float _x, float _y, float _w, float _h, float _r)
{
    u8g2_DrawRFrame(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_w), (uint8_t)std::round(_h), (uint8_t)std::round(_r));
}

void MyHAL::_drawHLine(float _x, float _y, float _l)
{
    u8g2_DrawHLine(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_l));
}

void MyHAL::_drawVLine(float _x, float _y, float _h)
{
    u8g2_DrawVLine(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), (uint8_t)std::round(_h));
}

void MyHAL::_drawVDottedLine(float _x, float _y, float _h)
{
    for (unsigned char i = 0; i < (unsigned char)std::round(_h); i++)
    {
        if (i % 8 == 0 | (i - 1) % 8 == 0 | (i - 2) % 8 == 0)
            continue;
        u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y) + i);
    }
}

void MyHAL::_drawHDottedLine(float _x, float _y, float _l)
{
    for (unsigned char i = 0; i < _l; i++)
    {
        if (i % 8 == 0 | (i - 1) % 8 == 0 | (i - 2) % 8 == 0)
            continue;
        u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x) + i, (int16_t)std::round(_y));
    }
}

unsigned long MyHAL::_getRandomSeed()
{
    return random(0, 100);
}
```

### 2.4 完善 main 函数

```
// main.cpp
#include <Arduino.h>
#include "my_hal.h"
#include "astra_rocket.h"
#include "astra_logo.h"

void setup()
{
 // put your setup code here, to run once:
  astraCoreInit();
}

void loop()
{
  // put your main code here, to run repeatedly:
  astraLauncher->update();
}
```

### 2.5 编译下载

通过 `USB` 线将 `ESP32` 连接到电脑上，点击 `upload` 编译固件上传到 `ESP32` 中，按下复位键重启开发板，一切顺利的话，就能够实现[【astra UI】v0.0.2-alpha 版本特性演示 - 超容易移植的 一行代码就可以创建的丝滑 OLED UI\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV16x421S7qc/?share_source=copy_web&vd_source=6edad9242d2299b41526c08a2751da39)中的丝滑效果

若是下载时遇到无法识别芯片等问题，可能需要手动进入下载模式：

- **拉低 BOOT 引脚**（按住开发板的 BOOT 按键）
- **短按 EN/RST 引脚**（复位开发板）
- **松开 BOOT 键**后立即开始烧录

# 3.添加旋转编码器到工程中

### 3.1 添加 OLED 对旋转编码器的响应函数

- 在`lib\astra-ui-v0-0-2-alpha\astra\ui\launcher.h`的`Launcher`类中添加公有成员函数`void update(int cnt, bool is_pressed);`
- 在`lib\astra-ui-v0-0-2-alpha\astra\ui\launcher.cpp`中添加该函数的视线，具体代码如下(**注意：这里只是简单实现了旋转编码器旋转切换 UI 界面和按下按键进入下一级界面(如果有的话)**)

```cpp
void Launcher::update(int cnt, bool is_pressed)
  {
    HAL::canvasClear();
    currentPage->render(camera->getPosition());
    selector->render(camera->getPosition());
    camera->update(currentPage, selector);

    uint8_t item_num = currentPage->getItemNum();
    uint8_t item_index = currentPage->selectIndex;
    if (cnt % item_num < 0)
      item_index = item_num - 1;
    else if (cnt % item_num > item_num - 1)
      item_index = 0;
    else
      item_index = cnt % item_num;
    if (item_index != currentPage->selectIndex)
    {
      selector->go(item_index);
    }
    if (is_pressed)
      open();
    // if (time == 2900)
    //   close();

    HAL::canvasUpdate();
  }
}
```

### 3.2 完善 main 函数

直接在`main.cpp`中添加`KY-040`的初始化代码及旋转和按下按键的简单处理操作，具体代码如下

```cpp
// main.cpp
#include <Arduino.h>
#include <ESP32Encoder.h>
#include "my_hal.h"
#include "astra_rocket.h"
#include "astra_logo.h"

ESP32Encoder encoder; // 创建编码器对象

void setup()
{

  Serial.begin(115200);
  // put your setup code here, to run once:
  // 初始化编码器（CLK=GPIO4, DT=GPIO2）
  encoder.attachHalfQuad(4, 2);
  // 配置硬件去抖动（必须设置）
  encoder.setFilter(1023); // 启用最大滤波强度
  encoder.clearCount();    // 计数器归零
  // 配置按键引脚（SW=GPIO15）
  pinMode(15, INPUT_PULLUP);

  astraCoreInit();
}

void loop()
{
  // put your main code here, to run repeatedly:
  static int lastCount = 0;
  int currentCount = encoder.getCount();
  bool isPressed = false;

  // 检测按钮按下（低电平触发）
  if (digitalRead(15) == LOW)
  {
    isPressed = true;
    // encoder.clearCount(); // 清除计数器
    Serial.println("Button Pressed!");
    delay(200); // 软件去抖动
  }

  astraLauncher->update(currentCount / 5, isPressed);
}
```

### 3.3 编译上传

点击`upload`将修改后的代码编译上传到`ESP32`中后复位，一切顺利的话应该就能通过旋转编码器切换`OLED`显示屏上的图标并进入下一级界面(如果有的话)
