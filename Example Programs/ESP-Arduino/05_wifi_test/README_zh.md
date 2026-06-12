# ESP32-S3 Arduino WiFi 测试示例

## 概述

本示例是 `05_wifi_test` 的 ESP-Arduino 版本，用于测试 WiFi 扫描和基础连接 UI。

使用的库：

- LCD：`GFX Library for Arduino`
- 触摸：`Adafruit CST8XX Library`
- IO 扩展：`Adafruit PCF8574`
- WiFi：ESP32 Arduino core 的 `WiFi`
- UI：`lvgl`

点击 `Scan` 按钮后，程序会扫描附近 WiFi AP，并显示 SSID、RSSI 和加密状态。点击某个 AP 后会弹出密码输入框，输入密码并点击 `Connect` 可尝试连接。

## 说明

WiFi 扫描和连接使用 ESP32 Arduino core 的标准 `WiFi.h`，没有自写 WiFi 管理栈。

ONX2432G028 屏幕宽度为 240 像素，本例程已按 2.8 寸屏幕重新约束 UI 尺寸：

- `WiFi Test` 标题靠左显示，右上角保留 `Scan` 按钮，避免两者重叠。
- 状态文字宽度限制为 216 像素，并启用自动换行。
- WiFi 列表宽度限制为 216 像素，列表项左对齐显示。
- SSID 过长时从右侧省略，避免左侧字符被裁掉。
- 连接面板、密码输入框和 LVGL 键盘均限制在屏幕边界内。

屏幕方向与 `01_touch_test` 一致：

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

## 操作流程

1. 上电后进入 WiFi Test 页面。
2. 点击右上角 `Scan`，程序切换到 STA 模式并扫描附近 AP。
3. 列表最多显示 20 个 AP，每一项包含 SSID、RSSI 和 `open` / `secure` 状态。
4. 点击 AP 后弹出密码输入面板。
5. 点击密码输入框会显示 LVGL 内置键盘。
6. 输入密码并点击 `Connect` 后开始连接。
7. 连接成功时显示当前 SSID 和 IP 地址；连接失败或断开时显示对应状态。

## 注意事项

- 本例程只演示扫描和基础连接，不保存 WiFi 配置。
- 开放网络可直接点击 AP 后点击 `Connect`，密码框可以留空。
- 扫描前会调用 `WiFi.disconnect(true)`，用于刷新扫描结果和连接状态。
- 后续修改 UI 时，不要使用超过 240 像素的固定宽度；文本建议使用 216 像素宽并开启换行或省略号。

## Arduino IDE 设置

使用 Arduino IDE 打开本例程时，开发板建议选择：

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## 文件

- `05_wifi_test.ino`: Arduino 主程序。
- `BoardConfig.h`: 板级引脚、分辨率和触摸/LCD 配置。
- `build_opt.h`: LVGL 编译选项，当前仅跳过默认 `lv_conf.h`。
- `sketch.yaml`: 记录 ESP32S3R8 + 16MB Flash 的目标配置。
