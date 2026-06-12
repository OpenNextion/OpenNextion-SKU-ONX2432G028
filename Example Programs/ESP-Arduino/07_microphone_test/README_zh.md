# ESP32-S3 Arduino 麦克风录音回放示例

## 概述

本示例是 `07_microphone_test` 的 ESP-Arduino 版本，用于测试 ONX2432G028 的 PDM 麦克风录音和喇叭回放。

界面和 ESP-IDF 例程保持一致：长按 `MIC` 按钮录音，松开后循环播放刚才录到 RAM 中的音频，点击 `Stop` 停止录音或播放。

使用的库：

- LCD：`GFX Library for Arduino`
- 触摸：`Adafruit CST8XX Library`
- IO 扩展：`Adafruit PCF8574`
- UI：`lvgl`
- 麦克风：ESP32 Arduino core 自带 ESP-IDF `driver/i2s_pdm.h`
- 喇叭：ESP32 Arduino core 自带 ESP-IDF `driver/i2s_std.h`

## 运行流程

1. 初始化 LCD、CST826 触摸、PCF8574 和 PDM 麦克风。
2. 从 PSRAM 申请 30 秒录音缓存。
3. 长按 `MIC`：开始录音，按钮变红。
4. 松开 `MIC`：停止录音，延迟打开喇叭 I2S 输出并开始循环回放，按钮变绿。
5. 点击 `Stop`：停止录音或回放，先关闭喇叭功放，再写入静音数据并关闭 I2S 输出。

## 音频配置

- 采样率：`16000 Hz`
- 位深：`16-bit`
- 录音格式：PDM RX 转 PCM mono
- 回放格式：I2S STD TX mono
- 最大录音时长：`30 s`
- 录音缓存：优先使用 PSRAM，失败时尝试内部 RAM

说明：ESP-IDF 版本中包含 ESP-SR/AFE 处理链路；Arduino 版本使用 ESP32 Arduino core 提供的标准 I2S 驱动完成录音与回放，不额外移植 ESP-SR/AFE 组件。

## 喇叭静音处理

ONX2432G028 的喇叭功放使能脚连接到 PCF8574 的 `I2S_CTRL`。本例程使用和音乐播放例程一致的静音策略，避免上电爆破声和停止播放后的电流声：

- PCF8574 的影子寄存器默认让 `I2S_CTRL` 保持低电平。
- 上电后先以 `100 kHz` 初始化 PCF8574，并在 LCD 复位前写入安全输出状态。
- `setup()` 阶段不初始化喇叭 I2S TX，避免 BCLK/WS/DOUT 切换到 I2S 外设时产生瞬态声音。
- 开始回放时才初始化并使能 I2S TX，先写入一小段静音数据，再打开喇叭功放。
- 点击 `Stop` 时先关闭喇叭功放，再写入静音数据并关闭 I2S TX。

## 显示方向

显示方向宏定义和本系列其它 2.8 寸例程保持一致：

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

状态文本宽度限制为 `216 px` 并启用自动换行，避免在 240x320 屏幕上超出边界。

## Arduino IDE 设置

使用 Arduino IDE 打开本例程时，开发板建议选择：

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## 文件

- `07_microphone_test.ino`: Arduino 主程序。
- `BoardConfig.h`: LCD、触摸、PDM 麦克风、I2S 喇叭和 PCF8574 引脚配置。
- `build_opt.h`: LVGL 编译选项，当前仅跳过默认 `lv_conf.h`。
- `sketch.yaml`: Arduino CLI profile，记录 16MB Flash、OPI PSRAM 和分区配置。
