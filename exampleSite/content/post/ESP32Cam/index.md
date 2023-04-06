+++
author = "coucou"
title = "Esp32cam显示到tft屏"
date = "2022-12-25"
description = "esp32cam + st7735 + ov2604"

categories = [
    "ESP32Cam"
]

tags = [
    "ESP32Cam"
]

+++

![](1.jpg)

## 开发环境

```ini
[env:esp32cam]
platform = espressif32
board = esp32cam
framework = arduino
monitor_speed = 115200
upload_speed = 921600
lib_deps = 
	bblanchon/ArduinoJson@^6.19.4
	espressif/esp32-camera@^2.0.0
	bodmer/TFT_eSPI@^2.4.79
	bodmer/TFT_eFEX@^0.0.8
	bodmer/JPEGDecoder@^1.8.1
```

## 代码

```c
#include "esp_camera.h"
#include <WiFi.h>
#include <SPI.h>
#include <TFT_eSPI.h>      // Hardware-specific library
TFT_eSPI tft = TFT_eSPI(); // Invoke custom library
#include <TFT_eFEX.h>
TFT_eFEX fex = TFT_eFEX(&tft);

#define PWDN_GPIO_NUM 32
#define RESET_GPIO_NUM -1
#define XCLK_GPIO_NUM 0
#define SIOD_GPIO_NUM 26
#define SIOC_GPIO_NUM 27

#define Y9_GPIO_NUM 35
#define Y8_GPIO_NUM 34
#define Y7_GPIO_NUM 39
#define Y6_GPIO_NUM 36
#define Y5_GPIO_NUM 21
#define Y4_GPIO_NUM 19
#define Y3_GPIO_NUM 18
#define Y2_GPIO_NUM 5
#define VSYNC_GPIO_NUM 25
#define HREF_GPIO_NUM 23
#define PCLK_GPIO_NUM 22

void camera_init()
{
    camera_config_t config;
    config.ledc_channel = LEDC_CHANNEL_0;
    config.ledc_timer = LEDC_TIMER_0;
    config.pin_d0 = Y2_GPIO_NUM;
    config.pin_d1 = Y3_GPIO_NUM;
    config.pin_d2 = Y4_GPIO_NUM;
    config.pin_d3 = Y5_GPIO_NUM;
    config.pin_d4 = Y6_GPIO_NUM;
    config.pin_d5 = Y7_GPIO_NUM;
    config.pin_d6 = Y8_GPIO_NUM;
    config.pin_d7 = Y9_GPIO_NUM;
    config.pin_xclk = XCLK_GPIO_NUM;
    config.pin_pclk = PCLK_GPIO_NUM;
    config.pin_vsync = VSYNC_GPIO_NUM;
    config.pin_href = HREF_GPIO_NUM;
    config.pin_sscb_sda = SIOD_GPIO_NUM;
    config.pin_sscb_scl = SIOC_GPIO_NUM;
    config.pin_pwdn = PWDN_GPIO_NUM;
    config.pin_reset = RESET_GPIO_NUM;
    config.xclk_freq_hz = 20000000;
    config.pixel_format = PIXFORMAT_JPEG;

    config.frame_size = FRAMESIZE_HQVGA; // FRAMESIZE_ + QVGA|CIF|VGA|SVGA|XGA|SXGA|UXGA
    config.jpeg_quality = 12;            // 0-63 lower number means higher quality
    config.fb_count = 2;

    // camera init
    esp_err_t err = esp_camera_init(&config);
    if (err != ESP_OK)
    {
        Serial.printf("Camera init failed with error 0x%x", err);
        return;
    }
}

void setup()
{
    Serial.begin(115200);

    camera_init();

    tft.begin();
    tft.fillScreen(TFT_BLACK);
    fex.listSPIFFS();
}

void loop()
{
    camera_fb_t *fb = esp_camera_fb_get();

    fex.drawJpg((const uint8_t *)fb->buf, fb->len, 0, 0, 240, 240);

    tft.setCursor(10, 230, 1); //设置起始坐标(10, 10)，2 号字体
    tft.println("TFT_Text");  //显示文字

    esp_camera_fb_return(fb);
}
```

