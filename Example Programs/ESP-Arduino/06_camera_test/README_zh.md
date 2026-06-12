# ESP32-S3 Arduino 摄像头测试示例

## 概述

本示例是 `06_camera_test` 的 ESP-Arduino 版本，用于测试 ONX2432G028 的摄像头采集和 LCD 实时预览。

使用的库：

- LCD：`GFX Library for Arduino`
- 触摸：`Adafruit CST8XX Library`
- IO 扩展：`Adafruit PCF8574`
- UI：`lvgl`
- 摄像头：ESP32 Arduino core 自带 `esp_camera`

## 功能

- 使用 Arduino_GFX 驱动 ST7789U LCD。
- 使用 Adafruit CST8XX 读取 CST826 触摸。
- 使用 LVGL 绘制标题和状态提示。
- 使用 ESP32 Arduino core 自带的 `esp_camera` 获取 RGB565/HVGA 图像。
- 摄像头帧按宽度缩放为 240x160 后，通过 Arduino_GFX 直接刷新到屏幕预览区域，减少每帧复制。

## 运行流程

1. 初始化 I2C、PCF8574、LCD、背光和触摸。
2. 通过 PCF8574 拉低摄像头 `PWDN`，唤醒摄像头模组。当前会同时释放 ONX2432G028 的 CAM_PWDN pin 4 和 3.5 寸已验证路径使用的 pin 3，以兼容同款摄像头/转接方式。
3. 初始化 `esp_camera`，配置为 HVGA `480x320`、`RGB565` 输出。
4. 使用 LVGL 绘制 `Camera Test` 标题和状态文本。
5. 主循环中约每 80 ms 获取一帧摄像头图像，缩放为 `240x160` 后显示在屏幕中部。

## 摄像头配置

- 帧尺寸：`FRAMESIZE_HVGA`
- 分辨率：`480x320`
- 像素格式：`PIXFORMAT_RGB565`
- XCLK：`20 MHz`
- 帧缓冲：PSRAM 中 2 个 frame buffer。
- 预览显示：缩放到 `240x160`，避免超出 ONX2432G028 的 240 像素屏幕宽度。

初始化策略优先参考 ONX2432G028 ESP-IDF `06_camera_test`：

1. 第一轮使用 ESP-IDF SCCB 配置：`pin_sccb_sda=-1`、`pin_sccb_scl=-1`、`sccb_i2c_port=0`，复用 I2C0。
2. 如果 Arduino camera 驱动在该模式下初始化失败，则回退为直接使用 GPIO8/GPIO7 作为 SCCB。
3. 串口会打印每轮初始化尝试和错误码，方便定位硬件或连接问题。

## 显示与尺寸

- 屏幕方向与 `01_touch_test` 保持一致：`DISPLAY_SWAP_XY=false`、`DISPLAY_MIRROR_X=true`、`DISPLAY_MIRROR_Y=false`。
- 状态文字宽度限制为 216 像素并启用自动换行。
- 摄像头画面使用逐行缩放，只额外占用一行 240 像素的临时缓冲。

本例程行为与 ESP-IDF 版本保持一致：上电后持续显示摄像头预览，不提供暂停/恢复按钮。

## Arduino IDE 设置

使用 Arduino IDE 打开本例程时，开发板建议选择：

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## 文件

- `06_camera_test.ino`: Arduino 主程序。
- `BoardConfig.h`: LCD、触摸、PCF8574 和摄像头引脚配置。
- `build_opt.h`: LVGL 编译选项，当前仅跳过默认 `lv_conf.h`。
- `sketch.yaml`: 记录 ESP32S3R8 + 16MB Flash 的目标配置。
