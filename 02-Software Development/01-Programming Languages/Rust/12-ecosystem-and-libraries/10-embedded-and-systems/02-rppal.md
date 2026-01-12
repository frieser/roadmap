# Rppal (Raspberry Pi)
---
---

## Summary
**Rppal** (Raspberry Pi Peripheral Access Library) provides access to the Raspberry Pi's GPIO, I2C, SPI, and PWM interfaces. It is designed specifically for the Raspberry Pi (running Linux) and is optimized for user-level access via `/dev/gpiomem`.

## Detailed Explanation

### Core Philosophy
To provide a fast, safe, and easy-to-use interface for the Raspberry Pi hardware in Rust. It supports the `embedded-hal` traits, meaning you can use generic drivers with it.

### Key Features
*   **GPIO**: Control pins, read inputs, software PWM.
*   **Buses**: Hardware I2C and SPI support.
*   **Performance**: Uses memory-mapped I/O for GPIO where possible for speed.

### Use Cases
*   **IoT**: Connecting sensors to a Pi.
*   **Robotics**: Controlling motors and servos via PWM.

### Code Example
*Dependencies: `rppal`*

```rust
use rppal::gpio::Gpio;
use std::thread;
use std::time::Duration;

// GPIO 23 (Physical pin 16)
const GPIO_LED: u8 = 23;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let gpio = Gpio::new()?;
    let mut pin = gpio.get(GPIO_LED)?.into_output();

    // Blink
    pin.set_high();
    thread::sleep(Duration::from_secs(1));
    pin.set_low();

    Ok(())
}
```

## Interview Questions

1.  **Q: Can Rppal be used on a microcontroller like an Arduino?**
    *   **A:** No. Rppal is specifically designed for the Raspberry Pi running a Linux operating system. It relies on the Linux kernel's device drivers (`/dev/gpiomem`, `/dev/i2c-*`). For microcontrollers, you would use a "Bare Metal" HAL crate (like `avr-hal` or `stm32f4xx-hal`).

2.  **Q: Does Rppal support `embedded-hal`?**
    *   **A:** Yes, Rppal implements the `embedded-hal` traits. This means you can create an Rppal I2C struct and pass it to a platform-agnostic driver (like a display driver) that expects an `I2c` trait implementation.
