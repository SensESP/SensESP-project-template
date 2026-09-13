# SensESP Project Template

This repository provides a template for [SensESP](https://github.com/SignalK/SensESP/) projects.
Fork, clone or download the repository and try building and uploading the project to an ESP32 device.
You should immediately see output on the serial monitor! Similarly, you should be able to connect to
the WiFi access point with the same name as the device. The password is `thisisfine`.

To customize the template for your own purposes, edit the `src/main.cpp` and `platformio.ini` files.

Comprehensive documentation for SensESP, including how to get started with your own project, is available at the [SensESP documentation site](https://signalk.org/SensESP/).

## Which environment to build

`platformio.ini` defines two kinds of build environments for each board:

| Board | Flash this (ESP-IDF from source) | Compile check only (precompiled Arduino) |
|-------|----------------------------------|------------------------------------------|
| Generic ESP32 | `esp32dev_espidf` | `pioarduino_esp32` |
| Generic ESP32-C3 | `esp32c3_espidf` | `pioarduino_esp32c3` |
| SH-ESP32 | `shesp32_espidf` | `shesp32` |
| HALMET | `halmet_espidf` | `halmet` |
| HALSER | `halser_espidf` | `halser` |

The default environment is `esp32dev_espidf`; change `default_envs` in `platformio.ini` to match your board, or pass `-e <env>` to `pio run`.

**Flash a `<board>_espidf` environment on any device that talks to a Signal K server over TLS (`https://`, `wss://`).** These environments build ESP-IDF from source, which makes `sdkconfig.defaults` authoritative and enables the dynamic mbedTLS buffers a TLS connection needs to fit in the ESP32's memory. The `esp_websocket_client` component for these environments is declared in `src/idf_component.yml` and pinned in `dependencies.lock`.

The plain Arduino environments use precompiled libraries that ignore `sdkconfig.defaults`. They build in a fraction of the time and are fine for compile checks and for servers on plain HTTP, but do not flash them onto a device that connects to a TLS server. The symptom is easy to mistake for a network fault: the device boots, joins WiFi, and Signal K stays Disconnected, with `mbedtls_ssl_setup` failing with `-0x7F00` in the log.

`arduino_esp32` and `arduino_esp32c3` use the official PlatformIO `espressif32` platform (Arduino Core 2.x). They cannot build the ESP-IDF environments or the TLS support at all and exist only for users who cannot use the pioarduino platform.

Things to know about the `<board>_espidf` environments:

- **The first build downloads ESP-IDF** (several hundred megabytes) and compiles it from source, which takes several minutes. Later builds only recompile what changed.
- **On Windows, keep the project in a short path without spaces** (for example `C:\p\my-project`). ESP-IDF builds fail on long or space-containing paths, and the first build takes 10 to 20 minutes.
- **A device running an Arduino build needs one USB flash** before it can take an ESP-IDF build over the air. Flashing the ESP-IDF build over OTA onto an Arduino-built device reports success but the device falls back to the old application.
- Every environment builds for 4 MB of flash with the `min_spiffs.csv` partition table, HALMET included. A HALMET flashed from an earlier version of this template used the 8 MB `default_8MB.csv` table; the first flash of the new layout moves the settings partition, so the device loses its saved WiFi and Signal K settings once and needs to be set up again through its web UI. For an 8 MB layout, see [HALMET-example-firmware](https://github.com/hatlabs/HALMET-example-firmware), which carries a matching `sdkconfig.defaults`.
- The build generates `sdkconfig.<env>` files and a `managed_components/` directory; both are ignored by git. `sdkconfig.defaults` (shared) and `sdkconfig.defaults.esp32c3` (appended automatically for the C3) are the tracked configuration. Once a `sdkconfig.<env>` file exists in the project root, its values override `sdkconfig.defaults` on later builds, and it survives `pio run -t fullclean`. After changing `sdkconfig.defaults`, delete every `sdkconfig.<env>` file so the next build regenerates them. After changing `sdkconfig.defaults.esp32c3`, delete only the C3 environment files (for example `sdkconfig.esp32c3_espidf`).
