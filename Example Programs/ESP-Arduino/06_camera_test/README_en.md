# ESP32-S3 Arduino Camera Test Example

## Overview

This is the ESP-Arduino version of `06_camera_test`. It tests camera capture and live LCD preview on the ONX2432G028 board.

Libraries used:

- LCD: `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- UI: `lvgl`
- Camera: `esp_camera` from the ESP32 Arduino core

## Features

- ST7789U display through Arduino_GFX.
- CST826 touch through Adafruit CST8XX.
- LVGL title and status label.
- RGB565/HVGA camera capture through the ESP32 Arduino core camera driver.
- Camera frames are scaled to 240x160 and pushed directly to the LCD preview area with Arduino_GFX to reduce per-frame copies.

## Runtime Flow

1. Initialize I2C, PCF8574, LCD, backlight, and touch.
2. Pull camera `PWDN` low through PCF8574 to wake the camera module. The sketch releases both ONX2432G028 CAM_PWDN pin 4 and the pin 3 path used by the verified 3.5-inch camera example, so the same camera module/adapter can be detected.
3. Initialize `esp_camera` with HVGA `480x320` and `RGB565` output.
4. Draw the `Camera Test` title and status text with LVGL.
5. In the main loop, capture one frame about every 80 ms, scale it to `240x160`, and draw it in the middle of the screen.

## Camera Configuration

- Frame size: `FRAMESIZE_HVGA`
- Resolution: `480x320`
- Pixel format: `PIXFORMAT_RGB565`
- XCLK: `20 MHz`
- Frame buffers: two frame buffers in PSRAM.
- Preview: scaled to `240x160` so it stays inside the ONX2432G028 240-pixel screen width.

Initialization follows the ONX2432G028 ESP-IDF `06_camera_test` first:

1. First try the ESP-IDF SCCB configuration: `pin_sccb_sda=-1`, `pin_sccb_scl=-1`, `sccb_i2c_port=0`, reusing I2C0.
2. If the Arduino camera driver fails in that mode, fall back to direct GPIO8/GPIO7 SCCB pins.
3. The serial log prints each initialization attempt and error code for hardware/debug checks.

## Display And Size

- Display direction matches `01_touch_test`: `DISPLAY_SWAP_XY=false`, `DISPLAY_MIRROR_X=true`, `DISPLAY_MIRROR_Y=false`.
- The status label is limited to 216 pixels and uses word wrapping.
- Preview scaling is line-based and only allocates one temporary 240-pixel line buffer.

This example matches the ESP-IDF behavior: after boot, it continuously displays the camera preview and does not provide a pause/resume button.

## Arduino IDE Settings

When opening this example in Arduino IDE, use:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## Files

- `06_camera_test.ino`: Main Arduino sketch.
- `BoardConfig.h`: LCD, touch, PCF8574, and camera pin configuration.
- `build_opt.h`: LVGL build option; it currently skips the default `lv_conf.h`.
- `sketch.yaml`: Target setting record for ESP32S3R8 + 16 MB flash.
