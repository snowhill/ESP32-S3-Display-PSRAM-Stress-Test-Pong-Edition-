# ESP32-S3 Display & PSRAM Stress Test (Pong Edition)

This repository contains an ESPHome configuration that turns an ESP32-S3 and an ST7789V TFT display into a playable, retro-style Pong game using a rotary encoder.

While fully playable, **the primary purpose of this project is a hardware stress test and validation tool.** It was built to verify the stability of the microcontroller, external memory, and SPI bus before deploying this hardware as the main control interface for a larger lab-grade incubator simulator project.

## 🎯 The "Why": Stress Testing the Hardware

Driving a 240x320 high-resolution color display requires a massive amount of memory and processing power. This game engine acts as a dynamic diagnostic tool to validate several critical hardware components simultaneously:

1. **PSRAM Bandwidth & Stability:** By forcing the display to update at 30 Frames Per Second (`update_interval: 33ms`), the game constantly writes to and reads from the 2MB Quad PSRAM buffer. If the PSRAM is misconfigured or unstable, the fast-moving ball and paddle will cause visual tearing, or the board will experience a fatal crash.
2. **SPI Bus Speed:** Continuously pushing full-screen redraws validates the integrity and speed of the SPI wiring (CLK/MOSI pins) between the ESP32-S3 and the ST7789V display.
3. **Input Latency:** Using a rotary encoder to control the paddle tests the hardware interrupt speed of the ESP32-S3 under heavy CPU load. If the processor is overwhelmed by rendering graphics, the paddle controls will feel sluggish or skip steps.
4. **Power Delivery:** Sustained 30 FPS rendering combined with an active Wi-Fi connection causes power spikes. If the board does not reboot during gameplay, the power supply and USB connections are proven stable.

## 🛠️ Hardware Specifications

* **Microcontroller:** ESP32-S3 DevKitC-1 (N8R2 variant)
* **Flash Memory:** 8MB
* **External RAM:** 2MB Quad PSRAM @ 40MHz
* **Display:** 240x320 TFT Color Display (ST7789V Controller)
* **Inputs:** 1x Rotary Encoder (Paddle Control), 1x Encoder Push Button (Start/Restart), 1x Tactile Button (Pause/Resume)

## 🔌 Wiring & Pinout

| Component | Pin Function | ESP32-S3 GPIO |
| --- | --- | --- |
| **ST7789V Display** | CLK (Clock) | GPIO12 |
|  | MOSI / SDI (Data) | GPIO11 |
|  | CS (Chip Select) | GPIO8 |
|  | DC (Data/Command) | GPIO9 |
|  | RST (Reset) | GPIO10 |
|  | BLK (Backlight) | GPIO7 |
| **Rotary Encoder** | DT / A | GPIO4 |
|  | CLK / B | GPIO5 |
|  | SW (Button) | GPIO6 |
| **Extra Button** | K0 (Pause) | GPIO3 |

*(Note: The rotary encoder and buttons utilize the ESP32's internal `INPUT_PULLUP` resistors. The other side of the switches should be wired directly to GND).*

## ⚠️ Important Display Configuration Notes

If you are replicating this build, pay close attention to the `display:` block in the ESPHome YAML:

* **Color Fix:** The ST7789V often renders a negative image with swapped Red/Blue channels. This code uses `invert_colors: false` and `color_order: BGR` to correct the hardware-level color mapping.
* **Orientation:** The game expects a vertical "Portrait" orientation so the ball can travel the full 320-pixel height. Ensure `rotation: 0°` (or `180°`) is set. If the screen is set to `90°` (Landscape), the paddle and floor will be drawn "off-screen" and remain invisible.

## 🚀 How to Play

1. Flash the provided YAML configuration to your ESP32-S3 via ESPHome.
2. The screen will boot to the Pong title menu.
3. **Press the Rotary Encoder button** to start the game.
4. **Turn the Rotary Encoder** to slide the paddle left and right.
5. Prevent the ball from hitting the bottom of the screen. The ball speed increases slightly with every successful hit!
6. **Press the K0 button** to pause or resume the game at any time.

---
