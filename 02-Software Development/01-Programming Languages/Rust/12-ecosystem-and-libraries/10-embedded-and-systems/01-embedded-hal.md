# Embedded HAL
---
---

## Summary
**embedded-hal** is the foundation of the Rust embedded ecosystem. It defines a set of traits (interfaces) for common hardware peripherals like GPIO, UART, I2C, and SPI. This allows drivers (e.g., for a temperature sensor) to be written once and run on any microcontroller that implements the HAL traits.

## Detailed Explanation

### Core Philosophy
"Write once, run on any chip." By abstracting the hardware details behind traits, the community can build platform-agnostic drivers. A driver for an LCD display written using `embedded-hal` will work on an STM32, an Arduino (AVR), or a Raspberry Pi without modification.

### Key Features
*   **Traits**: `InputPin`, `OutputPin`, `SpiDevice`, `I2c`.
*   **Platform Agnostic**: Zero dependencies on specific hardware.
*   **Typestate Programming**: Often used to enforce hardware configuration at compile time.

### Use Cases
*   **Driver Development**: Writing libraries for sensors and actuators.
*   **Firmware**: Writing portable application code.

### Code Example (Conceptual)
```rust
use embedded_hal::digital::OutputPin;

// This function works with ANY pin on ANY chip
fn blink_led<P: OutputPin>(pin: &mut P) {
    pin.set_high().unwrap();
    // delay...
    pin.set_low().unwrap();
}
```

## Interview Questions

1.  **Q: What is the main benefit of using `embedded-hal` over vendor SDKs (like C HALs)?**
    *   **A:** Vendor SDKs lock you into that specific manufacturer's chips. `embedded-hal` allows you to reuse drivers. If you switch from an STM32 chip to an nRF52 chip, your sensor drivers and application logic often don't need to change, only the low-level initialization code does.

2.  **Q: Explain the "Typestate" pattern in embedded Rust.**
    *   **A:** It uses the type system to encode the state of the hardware. For example, a GPIO pin might be type `Pin<Input>`. You cannot call `.set_high()` on it because the `Input` type doesn't implement `OutputPin`. You must explicitly consume the pin and convert it: `.into_output()`, which returns a `Pin<Output>`. This prevents configuration errors at compile time.
