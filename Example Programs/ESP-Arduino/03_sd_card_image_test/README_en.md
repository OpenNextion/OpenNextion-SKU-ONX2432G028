# ESP32-S3 Arduino SD Card Image Test Example

## Overview

This is the ESP-Arduino version of `03_sd_card_image_test`. It tests SD-card image-file scanning and full-screen image browsing.

Libraries used:

- LCD: `GFX Library for Arduino`
- Touch: `Adafruit CST8XX Library`
- IO expander: `Adafruit PCF8574`
- SD card: `SD_MMC` from the ESP32 Arduino core
- UI: `lvgl`
- JPG/JPEG decoding: Espressif JPEG decoder bundled with the ESP32 Arduino core
- PNG decoding: LVGL bundled `lodepng`

The sketch scans `/images` for image files. When images are found, it opens a full-screen image viewer. Swipe left/right to switch to the previous or next image. If no SD card is inserted, no images are found, or an image cannot be decoded, the sketch shows a centered red prompt on a white background. Prompt text is wrapped for the 240-pixel display width.

## Notes

This version does not implement a custom image decoder. JPG/JPEG files are decoded with `esp_jpeg_decode()` from the ESP32 Arduino core, and PNG files are decoded with LVGL's bundled `lodepng`. Images are fully decoded into a PSRAM frame buffer before the LCD is updated in bands, which avoids visible top-to-bottom decode refresh on the panel.

Images use a fixed width-fit policy: regardless of the source orientation, the image is scaled to the 240-pixel screen width and vertically centered. Other extensions are scanned for compatibility with the ESP-IDF directory rules, but the current Arduino version directly renders JPG/JPEG/PNG only; unsupported formats show a decode-failed message.

This example uses the same verified display direction as `01_touch_test`:

```cpp
#define DISPLAY_SWAP_XY false
#define DISPLAY_MIRROR_X true
#define DISPLAY_MIRROR_Y false
```

`build_opt.h` enables `LV_USE_PNG` and `LV_MEM_CUSTOM` so LVGL builds PNG support and the PNG decoder can allocate from the configured external-memory path.

## SD Card Layout

Create this folder in the SD card root:

```text
/images
```

Then copy image files into it.

Recommended formats:

```text
.jpg
.jpeg
.png
```

If the SD card is missing, the screen shows `No SD card detected...`. If `/images` does not exist or contains no scannable image files, the screen shows `Image not found.`.

## Arduino IDE Settings

When opening this example in Arduino IDE, use:

- Board: `ESP32S3 Dev Module`
- Flash Size: `16MB (128Mb)`
- PSRAM: `OPI PSRAM`
- Partition Scheme: `16M Flash (3MB APP/9.9MB FATFS)`

Arduino CLI build/upload example:

```bash
arduino-cli compile --fqbn esp32:esp32:esp32s3:FlashSize=16M,PartitionScheme=app3M_fat9M_16MB,PSRAM=opi ONX2432G028/ESP-Arduino/03_sd_card_image_test
arduino-cli upload -p <your port> --fqbn esp32:esp32:esp32s3:FlashSize=16M,PartitionScheme=app3M_fat9M_16MB,PSRAM=opi ONX2432G028/ESP-Arduino/03_sd_card_image_test
```

## Files

- `03_sd_card_image_test.ino`: Main Arduino sketch, including board initialization, SD scanning, image decoding, scaling, and swipe handling.
- `BoardConfig.h`: ONX2432G028 LCD, I2C, PCF8574, and SD_MMC pin configuration.
- `build_opt.h`: LVGL/PNG build options.
- `sketch.yaml`: Target setting record for ESP32S3R8 + 16 MB flash.
