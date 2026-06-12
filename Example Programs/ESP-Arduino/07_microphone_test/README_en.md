# ESP32-S3 Arduino Microphone Record/Playback Example

## Overview

This is the ESP-Arduino version of `07_microphone_test`. It tests PDM microphone recording and speaker playback on the ONX2432G028 board.

The interaction matches the ESP-IDF example: press and hold `MIC` to record, release it to loop-play the audio stored in RAM, and tap `Stop` to stop recording or playback.

Libraries used:

- LCD: `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- UI: `lvgl`
- Microphone: ESP-IDF `driver/i2s_pdm.h` from the ESP32 Arduino core
- Speaker: ESP-IDF `driver/i2s_std.h` from the ESP32 Arduino core

## Runtime Flow

1. Initialize LCD, CST826 touch, PCF8574, and PDM microphone input.
2. Allocate a 30-second recording buffer from PSRAM.
3. Press and hold `MIC`: recording starts and the button turns red.
4. Release `MIC`: recording stops, speaker I2S output is opened lazily, and loop playback starts; the button turns green.
5. Tap `Stop`: stop recording or playback, mute the speaker amplifier first, then write silence and disable I2S output.

## Audio Configuration

- Sample rate: `16000 Hz`
- Bit depth: `16-bit`
- Recording format: PDM RX converted to PCM mono
- Playback format: I2S STD TX mono
- Maximum record time: `30 s`
- Record buffer: PSRAM first, internal RAM fallback

Note: the ESP-IDF version includes the ESP-SR/AFE processing chain. The Arduino version keeps the core record/playback behavior and uses the standard I2S drivers bundled with the ESP32 Arduino core instead of porting ESP-SR/AFE.

## Speaker Mute Handling

The ONX2432G028 speaker amplifier enable pin is connected to `I2S_CTRL` on the PCF8574. This example uses the same mute strategy as the music example to avoid the boot pop and the idle hiss after stopping playback:

- The PCF8574 shadow byte keeps `I2S_CTRL` low by default.
- PCF8574 is initialized first at `100 kHz`, and the safe output state is written before LCD reset.
- `setup()` does not initialize speaker I2S TX, because switching BCLK/WS/DOUT into I2S peripheral mode at boot caused an audible transient on this board.
- Playback initializes and enables I2S TX only when needed, writes a short silence lead-in, then enables the speaker amplifier.
- `Stop` mutes the speaker amplifier first, then writes silence and disables I2S TX.

## Display Direction

The display direction macros match the other 2.8-inch examples:

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

The status label is limited to `216 px` and uses wrapping so messages stay inside the 240x320 screen.

## Arduino IDE Settings

When opening this example in Arduino IDE, use:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

## Files

- `07_microphone_test.ino`: Main Arduino sketch.
- `BoardConfig.h`: LCD, touch, PDM microphone, I2S speaker, and PCF8574 pin configuration.
- `build_opt.h`: LVGL build option; it currently skips the default `lv_conf.h`.
- `sketch.yaml`: Arduino CLI profile with the 16MB Flash, OPI PSRAM, and partition settings.
