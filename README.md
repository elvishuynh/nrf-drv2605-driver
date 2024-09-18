# DRV2605 Haptic Driver for nRF Connect SDK 2.7.0

This driver integrates support for the Texas Instruments DRV2605 Haptic Feedback Motor Driver into the nRF SDK 2.7.0. Due to the the nRF Connect SDK not utilizing the latest version of upstream Zephyr, it lacks native support for the DRV2605. The driver is adapted from [badjeff's implementation for ZMK](https://github.com/badjeff/zmk-drv2605-driver.git) and made compatible with the Zephyr build system in nRF SDK v2.7.0.

## Features

* Supports ERM (Eccentric Rotating Mass) and LRA (Linear Resonant Actuator) motor control
* Configurable haptic feedback library 
* Standby mode for power optimization
* Uses Zephyr's I2C interface to communicate with the DRV2605

## Getting Started

### Installation
* Clone or copy the repository into your nRF application root directory. For example if using a west workspace in nRF, update your manifest.yml:
```yml
manifest:
  self:
    path: application
  remotes:
    - name: nrf
      url-base: https://github.com/nrfconnect
    - name: elvishuynh
      url-base: https://github.com/elvishuynh
  projects:
    - name: sdk-nrf
      remote: nrf
      path: nrf
      revision: v2.7.0
      import:
        path-prefix: external
      clone-depth: 1
    - name: nrf-drv2605-driver
      remote: elvishuynh
      revision: main
```

* Append the `/path/to/your/nrf-drv2605-driver/` to the `ZEPHYR_EXTRA_MODULES` list within CMakeLists.txt

```cmake
list(APPEND ZEPHYR_EXTRA_MODULES ${CMAKE_CURRENT_SOURCE_DIR}/nrf-drv2605-driver)
```

### Configuration
To use the DRV2605, you need to configure your device tree with the appropriate I2C pins and settings. This can be done by adding an app.overlay file to your project.

Here’s an example app.overlay for the Actinius Icarus IoT Board v2 you can use as a template. Replace the pin numbers with the correct values for your board:

```overlay
dts
Copy code
&pinctrl {
    i2c2_default: i2c2_default {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 27)>,
                    <NRF_PSEL(TWIM_SCL, 0, 26)>;
        };
    };

    i2c2_sleep: i2c2_sleep {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 27)>,
                    <NRF_PSEL(TWIM_SCL, 0, 26)>;
            low-power-enable;
        };
    };
};

&i2c2 {
    status = "okay";
    compatible = "nordic,nrf-twim";
    pinctrl-0 = <&i2c2_default>;
    pinctrl-1 = <&i2c2_sleep>;
    pinctrl-names = "default", "sleep";

    drv2605_0: drv2605@5a {
        compatible = "ti,drv2605";
        reg = <0x5a>;

        /* Library to use, 0 = Empty, 1-5 are ERM, 6 is LRA. */
        library = <2>;

        /* switch to standby mode after idling for 1000ms */
        standby-ms = <1000>;
    };
};
```

### Notes
* This driver was developed and tested using:
  * [Adafruit DRV2605L](https://www.adafruit.com/product/2305) (voltage input range is lower in this "L" variant when compared to the DRV2605)
  * [Actinius Icarus IoT Board v2](https://www.actinius.com/icarus)
  * [Vibrating Mini Motor Disc](https://www.adafruit.com/product/1201)
* The default library for the DRV2605 driver is set to ERM mode (library 2 AKA library B)
* (See the [DRV2605 data sheet](https://www.ti.com/product/DRV2605) for more details on the libraries and waveforms available to use.)
* The driver switches to standby mode after 1000ms of idling to save power, configurable via the standby-ms property in the overlay file.

### Driver is adapted from [badjeff's DRV2605 driver for ZMK](https://github.com/badjeff/zmk-drv2605-driver.git)

