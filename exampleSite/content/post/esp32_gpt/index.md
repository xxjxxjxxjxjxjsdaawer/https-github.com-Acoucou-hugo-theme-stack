+++
author = "coucou"
title = "esp_gpt_demo"
date = "2023-12-01"
description = "esp_gpt_demo"
categories = [
    ""
]
tags = [
    ""
]
+++

## 如何使用

> 你只需要更改 API_KEY 即可使用

## 代码如下

index.html

```c
/*
  ESP32 HTTPClient Jokes API Example

  https://wokwi.com/projects/342032431249883731

  Copyright (C) 2022, Uri Shaked
*/

#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <Adafruit_GFX.h>
#include <Adafruit_ILI9341.h>

const char* ssid = "Wokwi-GUEST";
const char* password = "";

#define BTN_PIN 5
#define TFT_DC 2
#define TFT_CS 15
Adafruit_ILI9341 tft = Adafruit_ILI9341(TFT_CS, TFT_DC);

const String url = "https://api.chatanywhere.com.cn/v1/chat/completions";
const String apiKey = "YOUR_API_KEY";  // Replace with your OpenAI API key

String prompt = "hello";

String getGPTAnswer() {
  HTTPClient http;
  http.begin(url);
  http.addHeader("Content-Type", "application/json");
  http.addHeader("Authorization", "Bearer " + apiKey);

  String data = "\{ \"model\": \"gpt-3.5-turbo\",\"messages\": \[{\"role\": \"user\",\"content\":\""  + prompt + "\"\}\]\}";

  int httpResponseCode = http.POST(data);

  if (httpResponseCode == 200) {
    String response = http.getString();
    DynamicJsonDocument doc(2048);
    DeserializationError error = deserializeJson(doc, response);

    if (error) {
      Serial.print("deserializeJson() failed: ");
      Serial.println(error.c_str());
      return "<error>";
    }

    String answer = doc["choices"][0]["message"]["content"].as<String>();;
    http.end();
    return answer;
  } else {
    Serial.print("HTTP POST request failed, error code: ");
    Serial.println(httpResponseCode);
    http.end();
    return "<error>";
  }
}

void setup() {
  pinMode(BTN_PIN, INPUT_PULLUP);

  WiFi.begin(ssid, password, 6);

  tft.begin();
  tft.setRotation(1);

  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(2);
  tft.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(100);
    tft.print(".");
  }

  tft.print("\nOK! IP=");
  tft.println(WiFi.localIP());

  tft.fillScreen(ILI9341_BLACK);  
  tft.setCursor(0, 0);
  String answer = getGPTAnswer();
  tft.setTextColor(ILI9341_GREEN);
  tft.println(answer);

  Serial.begin(115200);
  Serial.println("Enter a prompt:");
}

void loop() {
  if (Serial.available()) {
    tft.fillScreen(ILI9341_BLACK);  
    tft.setCursor(0, 0);

    tft.setTextColor(ILI9341_WHITE);
    tft.println("\nLoading...");

    prompt = Serial.readStringUntil('\n');
    prompt.trim();

    String answer = getGPTAnswer();
    Serial.println("Answer: " + answer);
    Serial.println("Enter a prompt:");

    tft.setTextColor(ILI9341_GREEN);
    tft.println(answer);
  }
}
```
