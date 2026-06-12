# ESP32-S3 Arduino WiFi Test Example

## Overview

This is the ESP-Arduino version of `05_wifi_test`. It tests WiFi scanning and a basic connection UI.

Libraries used:

- LCD: `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- WiFi: `WiFi` from the ESP32 Arduino core
- UI: `lvgl`

Tap `Scan` to scan nearby WiFi APs. The list shows SSID, RSSI, and encryption state. Tap an AP to open the password panel, then tap `Connect` to start the connection attempt.

## Implementation

WiFi scanning and connection use the standard ESP32 Arduino `WiFi.h` APIs. No custom WiFi manager stack is implemented.

The ONX2432G028 display is 240 pixels wide, so this example constrains the UI for the 2.8-inch screen:

- The `WiFi Test` title is aligned to the left, leaving the top-right area for the `Scan` button.
- The status label is limited to 216 pixels and uses word wrapping.
- The WiFi list is limited to 216 pixels and each row is left-aligned.
- Long SSIDs are truncated from the right so the first characters remain visible.
- The connection panel, password field, and LVGL keyboard stay inside the screen bounds.

Display direction matches `01_touch_test`:

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

## Operation Flow

1. Power on and enter the WiFi Test page.
2. Tap `Scan` in the top-right corner. The sketch switches to STA mode and scans nearby APs.
3. The list shows up to 20 APs. Each item includes SSID, RSSI, and `open` / `secure` state.
4. Tap an AP to open the password panel.
5. Tap the password field to show LVGL's built-in keyboard.
6. Enter the password and tap `Connect` to start connecting.
7. When connected, the status label shows the SSID and IP address. Failed or disconnected states are also shown.

## Notes

- This example demonstrates scanning and basic connection only; WiFi credentials are not saved.
- For open networks, tap the AP and then tap `Connect` with an empty password field.
- The sketch calls `WiFi.disconnect(true)` before scanning so scan results and connection state are refreshed.
- When modifying the UI later, avoid fixed widths of 240 px or more. Use around 216 px for text and enable wrapping or ellipsis.

## Arduino IDE Settings

When opening this example in Arduino IDE, use:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## Files

- `05_wifi_test.ino`: Main Arduino sketch.
- `BoardConfig.h`: Board pins, resolution, touch, and LCD configuration.
- `build_opt.h`: LVGL build option; it currently skips the default `lv_conf.h`.
- `sketch.yaml`: Target setting record for ESP32S3R8 + 16 MB flash.
