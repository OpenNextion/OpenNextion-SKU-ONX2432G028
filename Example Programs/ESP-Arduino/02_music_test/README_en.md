# ESP32-S3 Arduino Music Test Example

## Overview

This is the ESP-Arduino version of `02_music_test` for the ONX2432G028 board.

It uses standard Arduino/ESP32 libraries and third-party libraries:

- LCD: `Arduino_ST7789` from `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- SD card: `SD_MMC` from the ESP32 Arduino core
- UI: `lvgl`, with the Music Demo UI assets ported from the ESP-IDF `components/gui`
- Audio playback: `ESP32-audioI2S-master`

The sketch mounts the SD card, scans `/music` for `.mp3` files, and displays the same LVGL Music Demo style as the ESP-IDF example: cover art, spectrum, track metadata, playback controls, and the ALL TRACKS list. Tapping a track plays it through the onboard NS4168 speaker amplifier.

## Notes

This Arduino version matches the ESP-IDF UI and uses a proven third-party library for MP3 decoding and I2S playback. It does not implement a custom audio decoder.

For the ONX2432G028 240 x 320 display, this sketch keeps the ESP-IDF Music Demo assets and only applies the required small-screen adaptation:

- Display direction matches the verified `01_touch_test` configuration: `DISPLAY_SWAP_XY=false`, `DISPLAY_MIRROR_X=true`, `DISPLAY_MIRROR_Y=false`.
- Extra scrollbars are disabled so the main page stays inside the screen bounds.
- Previous, play/pause, and next buttons are spaced farther apart to reduce accidental taps on the 2.8-inch panel.
- Cover art is scaled for the 2.8-inch screen while keeping the ESP-IDF layout and visual assets.

## Audio And Pop Suppression

- I2S BCLK: GPIO14
- I2S LRCLK/WS: GPIO16
- I2S DOUT/SDATA: GPIO15
- NS4168 enable: PCF8574 `I2S_CTRL`

PCF8574 ports default high after power-up. If `I2S_CTRL` is enabled too early, the speaker may pop before I2S output is stable. This sketch follows the same mitigation used by the tested 3.5-inch Arduino example:

- After PCF8574 initialization, the sketch immediately writes the shadow register with `I2S_CTRL=LOW`.
- LCD reset, SD CS, and other PCF8574 output changes all go through the shadow register so the amplifier enable bit is not changed accidentally.
- The NS4168 amplifier is enabled only after MP3 decoding and I2S output have started and settled.
- The amplifier is muted before pause, track switching, and playback restart to reduce switching transients.

## ESP32S3R8 + 16 MB Flash Settings

This sketch ports the complete LVGL Music UI images and fonts from the ESP-IDF example, so it is larger than Arduino's default `1.2 MB APP` partition. For ESP32S3R8 with external 16 MB flash, use these Arduino IDE settings:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

The project includes `sketch.yaml` as a record of the target board settings. With the current Arduino CLI, `--profile` may skip the locally installed library search path, so use the verified full FQBN:

```bash
arduino-cli compile --fqbn esp32:esp32:esp32s3:FlashSize=16M,PartitionScheme=app3M_fat9M_16MB,PSRAM=opi ESP-Arduino/02_music_test
arduino-cli upload --fqbn esp32:esp32:esp32s3:FlashSize=16M,PartitionScheme=app3M_fat9M_16MB,PSRAM=opi -p <your port> ESP-Arduino/02_music_test
```

## Files

- `02_music_test.ino`: Main Arduino sketch.
- `BoardConfig.h`: Board pins, resolution, I2C addresses, SD pins, and I2S pins.
- `build_opt.h`: LVGL build option.
- `sketch.yaml`: Target board setting record for ESP32S3R8 + 16 MB flash.
- `src/gui`: LVGL Music UI source and assets ported from the ESP-IDF example.
- `src/gui/gui_hal.cpp`: Arduino adapter for SD_MMC scanning, MP3 playback, volume mapping, and amplifier timing.

## SD Card Layout

Create this folder in the SD card root:

```text
/music
```

Then copy `.mp3` files into it.

If no SD card is detected, or if `/music` does not exist or contains no `.mp3` files, the screen shows a red hint instead of entering the player UI.

## Dependencies

Install these libraries from Arduino IDE Library Manager:

- `GFX Library for Arduino`
- `Adafruit CST8XX Library`
- `Adafruit PCF8574`
- `lvgl` 8.3.x
- `ESP32-audioI2S-master`
