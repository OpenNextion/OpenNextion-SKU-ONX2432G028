# ESP32-S3 Arduino Out-of-Box Demo

## Overview

This is the ESP-Arduino version of `09_outofbox_demo`. It runs the ONX2432G028 out-of-box LVGL showcase.

The Arduino version ports the customized `lv_demo_widgets` source and image assets from the ESP-IDF example, so the UI matches the ESP-IDF version.

Libraries used:

- LCD: `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- UI: `lvgl` 8.3.x

## Features

- Shows the LVGL widgets demo as the main out-of-box UI.
- Provides the `Profile`, `Analytics`, and `Shop` tabs.
- The `Profile` tab shows the Open Nextion avatar, description, email, and phone number.
- The `Analytics` tab demonstrates meters, charts, and related LVGL widgets.
- The `Shop` tab shows product images, list items, and chart widgets.
- Supports touch scrolling, tab switching, text areas, dropdowns, calendar input, and theme-color switching.

## Display Direction

The 09 example display direction has been verified on ONX2432G028 hardware. The current direction macros are:

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

Arduino_GFX rotation IDs do not map one-to-one to ESP-IDF `swap_xy` / `mirror_x` / `mirror_y`, so the sketch translates them in `displayRotationFromConfig()`.

## Initialization Flow

1. Initialize serial output and the I2C bus.
2. Reset the ST7789 LCD through PCF8574 P6.
3. Initialize the Arduino_GFX SPI LCD and apply the hardware-verified display direction.
4. Enable the backlight on GPIO6.
5. Initialize CST826 touch.
6. Register the LVGL display flush callback and touch input callback.
7. Call `lv_demo_widgets()` to create the out-of-box UI ported from the ESP-IDF example.

## Files

- `09_outofbox_demo.ino`: Main Arduino sketch for LCD, touch, PCF8574, and LVGL initialization.
- `BoardConfig.h`: ONX2432G028 pin and I2C address configuration.
- `build_opt.h`: LVGL and board build macros, including `NEXTION_2_8=1`.
- `src/widgets/lv_demo_widgets.c`: Customized widgets demo ported from the ESP-IDF example.
- `src/widgets/lv_demo_widgets.h`: Entry declaration for the widgets demo.
- `src/widgets/assets/`: Image asset C files used by the ESP-IDF demo.

## Arduino IDE Settings

When opening this example in Arduino IDE, use:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

The same settings can also be supplied through the `arduino-cli` FQBN options.

## Notes

This example does not recreate the UI from scratch. It reuses the ESP-IDF example's LVGL widgets demo and keeps the Arduino sketch focused on standard-library driver initialization and LVGL display/touch bridging.

When changing the UI, compare against the ESP-IDF `lv_demo_widgets` source first so the Arduino version remains visually aligned with the ESP-IDF version.
