# orrious' ESPHome Components

This repository contains my personal and some customized default components for [ESPHome](https://esphome.io/).

## 1. Usage

Add the repository to [`external_components`](https://esphome.io/components/external_components.html) in your configuration like this.

```yaml
external_components:
  - source:
    type: git 
    url: https://github.com/orrious/esphome-components.git
    ref: add-uart-mode
```

## 2. Components

### 2.1. `uart`

This is a customized version of the [`uart`](https://esphome.io/components/uart.html) component which allows the use of hardware flow control (thanks to shawly) and allows uart mode to be set for ESP32 boards.  This enables the ability to use ADM483 / MAX485 chips directly.  
**You can only use this with the [ESP-IDF framework](https://esphome.io/components/esp32.html#esp32-espidf-framework) and ESP32 boards!**

#### 2.1.1. Example
```
        VCC ---------------+
                           |
                   +-------x-------+
        RXD <------| R             |
                   |              B|-----------<> B
        TXD ------>| D    ADM483   |
ESP                |               |     RS485 bus side
        RTS --+--->| DE            |
              |    |              A|-----------<> A
              +----| /RE           |
                   +-------x-------+
                           |
                          GND
```

```yaml
# required
external_components:
  - source:
      type: git 
      url: https://github.com/orrious/esphome-components.git
      ref: add-uart-mode
    components: [uart]

esp32:
  board: esp32-s3-devkitc-1
  framework:
    # this is also required
    type: esp-idf
    version: recommended
    sdkconfig_options:
      CONFIG_COMPILER_OPTIMIZATION_SIZE: y

uart:
  baud_rate: 9600
  tx_pin: GPIO17
  rx_pin: GPIO18
  rts_pin: GPIO19
# cts_pin: GPIO20

  # possible values are
  # - DISABLE = disable hardware flow control
  # - RTS = enable RX hardware flow control (rts)
  # - CTS = enable TX hardware flow control (cts)
  # - CTS_RTS = enable hardware flow control
  # - MAX = ?
  hw_flowctrl: DISABLE

  # UART mode selection - Values:
  # - UART     :regular UART mode
  # - RS485_HD :half duplex RS485 UART mode control by RTS pin
  # - RS485_CD :RS485 collision detection UART mode (used for test purposes)
  # - RS485_AC :application control RS485 UART mode (used for test purposes)
  # - IRDA     :IRDA UART mode
  mode: RS485_HD
  
  debug:
    # this is just for debugging
    direction: BOTH
    dummy_receiver: true
    after:
      delimiter: "\n"
    sequence:
      - lambda: UARTDebug::log_string(direction, bytes);

switch:
  - platform: uart
    name: "UART Test"
    data: "Hello World\n"
```

#### 2.1.2. Notes

To test hardware flow control you need a USB to TTL serial adapter with an FTDI FT232RL chip which allows you to use hardware flow control. It should have pins for RTS and CTS.  
These ones should work: [[1](https://www.amazon.com/dp/B07BBPX8B8)] (confirmed working) [[2](https://www.amazon.com/dp/B07XF2SLQ1)] (this one requires soldering pin headers for RTS/CTS)  
Remember to enable hardware flow control on your host with `stty -F /dev/ttyUSB0 crtscts` and don't forget to set the correct baudrate with `stty -F /dev/ttyUSB0 57600` (replace 57600 with your required baudrate)
