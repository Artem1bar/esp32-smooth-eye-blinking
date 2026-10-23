# esp32-smooth-eye-blinking

Smooth, natural-looking eye-blink animation for an ESP32 + ST7789 TFT display. Two white ellipses blink at random intervals with an ease-in/ease-out curve and a 10% chance of a quick double-blink. Rendered via a heap-allocated pixel buffer so only the eye region is redrawn — no full-screen flicker.

https://github.com/user-attachments/assets/1987799e-cd9d-4975-bf2d-db59a20a4e42

## Hardware

Tested on two boards:

| Board | PlatformIO env | Notes |
|---|---|---|
| **Adafruit Feather TFT ESP32-S3** | *(default)* | Built-in 240×280 ST7789; uses Adafruit GFX |
| **Waveshare ESP32-S3-LCD** | `waveshare_esp32_s3_lcd` | 240×280 ST7789; uses TFT_eSPI build flags |

Default pin definitions in `src/main.cpp` (Adafruit Feather):

| Signal | Pin |
|--------|-----|
| CS | 10 |
| DC | 6 |
| RST | — (internal) |
| Backlight | 46 |
| I2C power | 15 |

Any ESP32 board + ST7789 SPI display works — update the `#define` block at the top of `main.cpp` for your wiring.

## Setup

This project uses **PlatformIO**. The `platformio.ini` targets an ESP32-S3 at 240 MHz with the `huge_app` partition scheme.

1. Install [PlatformIO](https://platformio.org/) (VS Code extension or CLI).
2. Clone the repo and open it in VS Code (PlatformIO detects `platformio.ini` automatically), or use the CLI:

```bash
# Default (Adafruit Feather TFT ESP32-S3)
pio run --target upload

# Waveshare ESP32-S3-LCD
pio run -e waveshare_esp32_s3_lcd --target upload

pio device monitor --baud 115200
```

Dependencies (`lib_deps` in `platformio.ini`):
- `adafruit/Adafruit GFX Library`
- `bodmer/TFT_eSPI`

PlatformIO installs them automatically on first build.

## Configuration

Edit the `#define` block at the top of `src/main.cpp`:

| Constant | Default | Description |
|----------|---------|-------------|
| `EYE_WIDTH` | 30 | Ellipse horizontal radius (px) |
| `EYE_HEIGHT` | 50 | Ellipse vertical radius (px) |
| `EYE_SPACING` | 100 | Center-to-center distance between eyes |
| `EYE_COLOR` | White | Fill color (any `uint16_t` RGB565 value) |
| `BG_COLOR` | Black | Background color |
| `BLINK_FRAMES` | 20 | Total frames per blink (close + open) |
| `FRAME_DELAY` | 4 ms | Delay per blink frame |
| `OPEN_TIME_MIN` | 1000 ms | Minimum time eyes stay open |
| `OPEN_TIME_MAX` | 4000 ms | Maximum time eyes stay open |

## How it works

- A heap-allocated `uint16_t` buffer covers just the eye region (not the full screen), so only those pixels are pushed over SPI on every frame.
- The blink uses two phases: close (ease-in, quadratic) and open (ease-out, smooth step). `blinkProgress` drives the ellipse height each frame.
- A `random(10) == 0` check gives a 10% chance of a 300 ms double-blink after each blink completes.
- `bufferFillEllipse` is a software rasterizer that fills the buffer row by row using the standard ellipse equation — no GFX library call needed for the animated part.

## Related project

The eye animation is also part of [wndr-os](https://github.com/Artem1bar/wndr-os), a full Tamagotchi-style ESP32-S3 firmware with menus, touch input, IMU-driven eye drift, party games, and power management.

## License

MIT. See `LICENSE.md`.
