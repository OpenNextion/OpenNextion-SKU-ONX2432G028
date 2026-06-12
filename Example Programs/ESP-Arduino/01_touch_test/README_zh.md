# ESP32-S3 Arduino 触摸测试示例

## 概述

本示例是 `01_touch_test` 的 ESP-Arduino 版本，用于 ONX2432G028 开发板的 2.8 寸屏基础触摸测试。程序使用 Arduino 标准库生态完成硬件驱动和 UI 绘制：

- LCD：`GFX Library for Arduino` 的 `Arduino_ST7789`
- 触摸：`Adafruit CST8XX Library`
- IO 扩展：`Adafruit PCF8574`
- UI：`lvgl`

屏幕显示白色背景和灰色 `Touch Test` 文本。触摸屏幕时，程序会在触点位置绘制红色 4x4 像素方块，并在串口监视器输出触摸坐标。

UI 文本由 LVGL 绘制；触摸轨迹参考 ESP-IDF 例程的实现方式，使用 LCD 驱动直接刷 4x4 红色小块，避免连续触摸时创建大量 LVGL 对象导致内存增长。

## 文件

- `01_touch_test.ino`: Arduino IDE 可直接打开的主程序。
- `BoardConfig.h`: 板级引脚、分辨率和 I2C 地址配置。
- `build_opt.h`: Arduino ESP32 编译选项，用于让 LVGL 使用内置默认配置。

## 硬件引脚

| 功能 | ESP32-S3 引脚 |
| :--- | :--- |
| I2C SCL | GPIO7 |
| I2C SDA | GPIO8 |
| LCD SCLK | GPIO5 |
| LCD MOSI | GPIO1 |
| LCD DC | GPIO3 |
| LCD CS | GPIO2 |
| LCD BL | GPIO6 |
| LCD RST | PCF8574 P6 |

## Arduino IDE 设置

1. 安装 ESP32 Arduino core。
2. 安装库：`GFX Library for Arduino`、`Adafruit CST8XX Library`、`Adafruit PCF8574`、`lvgl`。
3. 打开 `ONX2432G028/ESP-Arduino/01_touch_test/01_touch_test.ino`。
4. 开发板选择 `ESP32S3 Dev Module`。
5. 建议设置：
   - USB CDC On Boot: Enabled
   - Flash Size: `16MB (128Mb)`
   - PSRAM: `OPI PSRAM`
   - Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`
6. 编译并烧录。
7. 打开 115200 波特率串口监视器查看触摸坐标。

## 屏幕方向

屏幕方向可在 `01_touch_test.ino` 顶部通过以下宏配置：

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

当前默认值为 ONX2432G028 实机验证后的方向配置。Arduino_GFX 的 rotation 编码与 ESP-IDF 的 `swap_xy` / `mirror_x` / `mirror_y` 不完全同名，因此程序通过 `displayRotationFromConfig()` 做一次映射。

## 实现说明

- LCD 复位通过 PCF8574 P6 控制，初始化时保持其它扩展 IO 为高电平，避免影响共用扩展 IO 的外设。
- LVGL 负责初始界面和文字显示，根页面禁用滚动，避免触摸边缘时页面被拖动。
- 红点使用 `Arduino_GFX` 直接刷屏绘制，行为接近 ESP-IDF 例程中的 `esp_lcd_panel_draw_bitmap()`。
- 本例程不包含自定义 Arduino 驱动库，LCD、触摸、IO 扩展和 UI 均使用标准 Arduino 库或第三方库。
