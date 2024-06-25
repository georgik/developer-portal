---
title: "Simplify Creating a New Project for ESP32-Based DevKits Using ESP-BSP Generic"
date: 2024-06-24
showAuthor: false
authors:
    - "juraj-michalek"
tags: ["Embedded Systems", "ESP32", "ESP32-C3", "Espressif", "BSP", "Developer Workflow"]
---

# Simplify Creating a New Project for ESP32-Based DevKits Using ESP-BSP Generic

## Introduction

The ESP Board Support Package (ESP-BSP) simplifies the development of applications for various Espressif SoCs, including the popular ESP32-C3. This guide will show you how to create a project for a custom board using the `esp_bsp_generic` package. This approach uses configuration through `sdkconfig` and does not require changes to C code, making it an ideal solution for simple boards like Espressif's DevKits.

## Prerequisites

Before you start, ensure you have the following:

- [**ESP-IDF**](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html): The official development framework for Espressif SoCs (ESP32, ESP32-S3, ESP32-C3, ESP32-P4, etc.), properly installed and sourced in your shell.

## Setting Up Your Project

### 1. Creating a New Project

Start by creating a new project with ESP-IDF. You can use `idf.py create-project` to create a new project directory.

```bash
idf.py create-project button_led_example
cd button_led_example
```

### 2. Adding `esp_bsp_generic` to Your Project

Add the `esp_bsp_generic` package to your project using the IDF Component Manager:

```bash
idf.py add-dependency "espressif/esp_bsp_generic^1.2.0"
```

### 3. Configuring the BSP with SDKCONFIG

Create a specific `sdkconfig` file for your custom DevKit. For instance, you can name it `sdkconfig.esp32_c3_devkit_rust_1` and configure the necessary settings.

Example of `sdkconfig.esp32_c3_devkit_rust_1`:

```text
# This file was generated using idf.py save-defconfig. It can be edited manually.
# Espressif IoT Development Framework (ESP-IDF) Project Minimal Configuration
#
CONFIG_IDF_TARGET="esp32c3"

# ESP32-C3-DevKit-RUST-1 Settings
# Buttons
CONFIG_BSP_BUTTONS_NUM=1
CONFIG_BSP_BUTTON_1_TYPE_GPIO=y
CONFIG_BSP_BUTTON_1_GPIO=9
CONFIG_BSP_BUTTON_1_LEVEL=0
# LEDs
CONFIG_BSP_LEDS_NUM=2
CONFIG_BSP_LED_1_TYPE_GPIO=y
CONFIG_BSP_LED_1_GPIO=7
CONFIG_BSP_LED_1_LEVEL=1
CONFIG_BSP_LED_2_TYPE_RGB=y
CONFIG_BSP_LED_RGB_GPIO=2
CONFIG_BSP_LED_RGB_BACKEND_WS2812=y
```

### 4. Writing the Example Code

Here's an example code that initializes buttons and LEDs using the `esp_bsp_generic` package:

```c
#include <stdio.h>
#include "bsp/esp-bsp.h"
#include "esp_log.h"
#include "led_indicator_blink_default.h"

static const char *TAG = "example";

#if CONFIG_BSP_LEDS_NUM > 0
static int example_sel_effect = BSP_LED_BREATHE_SLOW;
static led_indicator_handle_t leds[BSP_LED_NUM];
#endif

#if CONFIG_BSP_BUTTONS_NUM > 0
static void btn_handler(void *button_handle, void *usr_data)
{
    int button_pressed = (int)usr_data;
    ESP_LOGI(TAG, "Button pressed: %d. ", button_pressed);

#if CONFIG_BSP_LEDS_NUM > 0
    led_indicator_stop(leds[0], example_sel_effect);

    if (button_pressed == 0) {
        example_sel_effect++;
        if (example_sel_effect >= BSP_LED_MAX) {
            example_sel_effect = BSP_LED_ON;
        }
    }

    ESP_LOGI(TAG, "Changed LED blink effect: %d.", example_sel_effect);
    led_indicator_start(leds[0], example_sel_effect);
#endif
}
#endif

void app_main(void)
{
#if CONFIG_BSP_BUTTONS_NUM > 0
    /* Init buttons */
    button_handle_t btns[BSP_BUTTON_NUM];
    ESP_ERROR_CHECK(bsp_iot_button_create(btns, NULL, BSP_BUTTON_NUM));
    for (int i = 0; i < BSP_BUTTON_NUM; i++) {
        ESP_ERROR_CHECK(iot_button_register_cb(btns[i], BUTTON_PRESS_DOWN, btn_handler, (void *) i));
    }
#endif

#if CONFIG_BSP_LEDS_NUM > 0
    /* Init LEDs */
    ESP_ERROR_CHECK(bsp_led_indicator_create(leds, NULL, BSP_LED_NUM));

    /* Set LED color for first LED (only for addressable RGB LEDs) */
    led_indicator_set_rgb(leds[1], SET_IRGB(0, 0x00, 0x64, 0x64));

    /*
    Start effect for each LED
    (predefined: BSP_LED_ON, BSP_LED_OFF, BSP_LED_BLINK_FAST, BSP_LED_BLINK_SLOW, BSP_LED_BREATHE_FAST, BSP_LED_BREATHE_SLOW)
    */
    led_indicator_start(leds[0], BSP_LED_BREATHE_SLOW);
#endif
}
```

### 5. Building and Flashing the Project

You can build and flash your project with the predefined configuration:

```bash
idf.py -D "SDKCONFIG_DEFAULTS=sdkconfig.esp32_c3_devkit_rust_1" build flash monitor
```

If you need to adjust any settings using `menuconfig`, you can run:

```bash
idf.py menuconfig
idf.py build flash monitor
```

## Conclusion

Creating a project for custom boards using the `esp_bsp_generic` package simplifies the process and allows for easy configuration through `menuconfig`. This approach is ideal for simple boards and helps you quickly get your hardware supported by the ESP32 ecosystem. By following this guide, you can develop and submit new BSPs, making hardware integration easier for other developers.

## Useful Links

- [ESP-BSP GitHub Repository](https://github.com/espressif/esp-bsp)
- [ESP-BSP Documentation](https://github.com/espressif/esp-bsp/blob/master/README.md)
- [ESP-IDF Installation Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html)
- [BSP Development Guide](https://github.com/espressif/esp-bsp/blob/master/BSP_development_guide.md)
- [ESP Component Registry](https://components.espressif.com/)
- [IDF Component Manager Documentation](https://docs.espressif.com/projects/idf-component-manager/en/latest/)
- [ESP32-C3-DevKit-RUST-1](https://www.espressif.com/en/products/devkits)
- [AliExpress Espressif Official Store](https://www.aliexpress.com/item/1005004418342288.html)
- [Mouser Electronics](https://www2.mouser.com/ProductDetail/Espressif-Systems/ESP32-C3-DevKit-RUST-1?qs=4ASt3YYao0WvXOj9TGjU2A%3D%3D)
- [Training Material](https://esp-rs.github.io/std-training/)
- [ESP32-C3 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf)
