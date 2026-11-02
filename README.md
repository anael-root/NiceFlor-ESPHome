# NiceFlor-ESPHome

Emulates a **Nice Flor-s** remote (433.92 MHz, rolling code) with an ESP and [ESPHome](https://esphome.io), so you can control a Nice gate or garage door from Home Assistant.

> [!NOTE]
> This repository is a **fork** of [TimelessNL/NiceFlor-ESPHome](https://github.com/TimelessNL/NiceFlor-ESPHome).
> All credit for the Nice Flor-s encoding and the original configuration goes to its authors.
> This fork adds **ESP32 / ESP-IDF** support and updates the configuration for recent ESPHome versions.

## What's new in this fork

| | Original project | This fork |
|---|---|---|
| ESP8266 (Arduino) | ✅ | ✅ |
| ESP32 (ESP-IDF) | ❌ | ✅ |
| ESP32 (Arduino) | untested | untested |
| ESPHome ≥ 2024.6 (new `ota` syntax) | ❌ | ✅ |

Compilation verified with **ESPHome 2026.10.1** on `nodemcuv2` (ESP8266, Arduino) and `esp32dev` (ESP32, ESP-IDF).

### Changes

- **`niceflors/nice.h`**: the code no longer depends on Arduino.
  - `pgm_read_word_near` falls back to a direct memory read when it is not defined (as on ESP-IDF).
  - The Arduino `byte` type is replaced with `uint8_t`.
  - Timings are returned as `std::vector<int32_t>`, the type `remote_transmitter.transmit_raw` expects on every platform (with the ESP-IDF toolchain, `int32_t` is a `long`, not an `int`).
- **`niceflors.yaml`**:
  - The `ota:` block uses the current syntax (`- platform: esphome`).
  - Lambdas use `std::vector<int32_t>`.
  - The configuration now targets an ESP32 (`esp32dev`, ESP-IDF framework).
  - The ESP8266-only `adc` sensor on `VCC` ("Power Supply") has been removed.

## Hardware

- An ESP32 or ESP8266.
- A 433.92 MHz transmitter (OOK/ASK modulation).

| Board | Transmitter DATA pin |
|---|---|
| ESP32 (`esp32dev`) | `GPIO23` |
| ESP8266 (`nodemcuv2`) | `D6` |

## Installation

1. Copy `niceflors.yaml` and the `niceflors/` folder into your ESPHome configuration folder.
2. Create or update `secrets.yaml`:
   ```yaml
   api_key: "<base64 API encryption key>"
   ota_password: "..."
   wifi_ssid: "..."
   wifi_password: "..."
   web_password: "..."
   ```
3. Choose your platform in `niceflors.yaml`:

   **ESP32 (ESP-IDF)**, the default configuration:
   ```yaml
   esp32:
     board: esp32dev
     framework:
       type: esp-idf

   remote_transmitter:
     pin: GPIO23
   ```

   **ESP8266**:
   ```yaml
   esp8266:
     board: nodemcuv2
     restore_from_flash: true

   remote_transmitter:
     pin: D6
   ```
   You can also add the supply voltage measurement, which is only available on ESP8266:
   ```yaml
   sensor:
     - platform: adc
       pin: VCC
       name: "Power Supply"
   ```
4. Set the serial number of the emulated remote (`remote_serial`) and, if needed, the button IDs (`button_1` to `button_4`) in the `globals` section.
5. Compile and flash:
   ```bash
   esphome run niceflors.yaml
   ```

## Usage

Once the ESP is added to Home Assistant, the following buttons appear:

| Button | Action |
|---|---|
| **Pairing** | Transmits about 60 frames to pair the virtual remote with the receiver |
| **Step-by-Step** | Step-by-step command (button 1) |
| **AUX** | Button 2 |
| **OPEN** | Button 3 |
| **CLOSE** | Button 4 |

The rolling code is incremented on every press and saved to flash. The **Code** text sensor shows its current value.

To pair, put the Nice receiver into learning mode (see its manual), then press **Pairing**.

## License

Apache License 2.0, same as the original project. See [LICENSE](LICENSE).
