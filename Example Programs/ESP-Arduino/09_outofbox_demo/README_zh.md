# ESP32-S3 Arduino 开箱演示示例

## 概述

本示例是 `09_outofbox_demo` 的 ESP-Arduino 版本，用于运行 ONX2432G028 的开箱 LVGL 综合演示界面。

Arduino 版本已移植 ESP-IDF 例程中的自定义 `lv_demo_widgets` 源码和图片资源，界面效果与 ESP-IDF 版本保持一致。

使用的库：

- LCD：`GFX Library for Arduino`
- 触摸：`Adafruit CST8XX Library`
- IO 扩展：`Adafruit PCF8574`
- UI：`lvgl` 8.3.x

## 功能

- 使用 LVGL widgets demo 显示综合控件演示界面。
- 提供 `Profile`、`Analytics`、`Shop` 三个 Tab 页面。
- `Profile` 页面显示 Open Nextion 头像、说明、邮箱和电话信息。
- `Analytics` 页面显示 meter、chart 等 LVGL 控件效果。
- `Shop` 页面显示商品图片、列表和统计图表。
- 支持触摸滑动、Tab 切换、文本框、下拉框、日历和颜色主题切换。

## 显示方向

09 例程的显示方向已在 ONX2432G028 实机验证。当前方向宏为：

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

Arduino_GFX 的 rotation 编码与 ESP-IDF 的 `swap_xy` / `mirror_x` / `mirror_y` 不完全同名，主程序通过 `displayRotationFromConfig()` 做统一映射。

## 初始化流程

1. 初始化串口和 I2C 总线。
2. 通过 PCF8574 P6 复位 ST7789 LCD。
3. 初始化 Arduino_GFX SPI LCD，并设置实机验证后的显示方向。
4. 打开背光 GPIO6。
5. 初始化 CST826 触摸。
6. 注册 LVGL 显示 flush 回调和触摸输入回调。
7. 调用 `lv_demo_widgets()` 创建从 ESP-IDF 例程移植的开箱演示 UI。

## 文件

- `09_outofbox_demo.ino`：Arduino 主程序，负责 LCD、触摸、PCF8574 和 LVGL 初始化。
- `BoardConfig.h`：ONX2432G028 的引脚和 I2C 地址配置。
- `build_opt.h`：LVGL 和板型编译宏，包含 `NEXTION_2_8=1`。
- `src/widgets/lv_demo_widgets.c`：从 ESP-IDF 例程移植的自定义 widgets demo。
- `src/widgets/lv_demo_widgets.h`：widgets demo 的入口声明。
- `src/widgets/assets/`：ESP-IDF 例程使用的图片资源 C 文件。

## Arduino IDE 设置

使用 Arduino IDE 打开本例程时，开发板建议选择：

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

这些配置也可以通过 `arduino-cli` 的 FQBN 参数指定。

## 说明

本例程没有重新实现 UI，而是复用并移植 ESP-IDF 例程中的 LVGL widgets demo。Arduino 主程序只负责标准库驱动初始化和 LVGL 显示/触摸适配。

修改界面效果时应优先对照 ESP-IDF 版本的 `lv_demo_widgets` 源码，避免 Arduino 版本和 ESP-IDF 版本出现视觉差异。
