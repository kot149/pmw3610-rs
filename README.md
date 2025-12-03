# pmw3610-rs

PixArt PMW3610 Low-Power Mouse Sensor Driver for Rust/Embassy/RMK.

Ported from the Zephyr driver implementation:
https://github.com/zephyrproject-rtos/zephyr/blob/d31c6e95033fd6b3763389edba6a655245ae1328/drivers/input/input_pmw3610.c

> [!Warning]
> This implementation is very experimental and only implemented/tested with nRF52840.
> The custom input device interface in RMK is still in early stages and may be subject to changes in future versions of RMK.

## Usage (with RMK and nRF52840)

### 1. Add to your Cargo.toml dependencies

```toml
[dependencies]
pmw3610-rs = { git = "https://github.com/kot149/pmw3610-rs", branch = "main", features = ["embassy-nrf", "rmk"] }
```

### 2. Initialize the sensor

```rust
use pmw3610_rs::{Pmw3610Config, Pmw3610Device};
use embassy_nrf::gpio::{Flex, Input, Output, Level, Pull, OutputDrive};

// Initialize PMW3610 mouse sensor
let pmw3610_config = Pmw3610Config {
    res_cpi: 800,
    smart_mode: false,
    force_awake: false,
    swap_xy: false,
    invert_y: false,
    invert_x: false,
    ..Default::default()
};
let pmw3610_sck = Output::new(p.P0_05, Level::High, OutputDrive::Standard);
let pmw3610_sdio = Flex::new(p.P0_04);
let pmw3610_cs = Output::new(p.P0_09, Level::High, OutputDrive::Standard);
let pmw3610_irq = Input::new(p.P0_02, Pull::Up);
let mut pmw3610_device = Pmw3610Device::new(
    pmw3610_sck, pmw3610_sdio, pmw3610_cs, Some(pmw3610_irq), pmw3610_config
);

// Add to the run_devices! macro
run_devices! (
    (matrix, pmw3610_device) => EVENT_CHANNEL,
),
```

### 3. Add an InputProcessor to handle the events

`Pmw3610Device` returns `Event::Joystick` events. You need an `InputProcessor` to convert these into `MouseReport`.

For simple mouse movement, use `JoystickProcessor`:

```rust
use rmk::input_device::joystick::JoystickProcessor;
let mut joystick_proc = JoystickProcessor::new([[1, 0], [0, 1]], [0, 0], 4, &keymap);

// Add processor to the chain
run_processor_chain! {
    EVENT_CHANNEL => [joystick_proc],
},
```

## Custom HAL Implementation

To use with a different HAL, implement the `BidirectionalPin` trait:

```rust
use pmw3610_rs::BidirectionalPin;

struct MyFlexPin {
    // Your pin implementation
}

impl BidirectionalPin for MyFlexPin {
    fn set_as_output(&mut self) {
        // Configure pin as output
    }

    fn set_as_input(&mut self) {
        // Configure pin as input
    }

    fn set_high(&mut self) {
        // Set pin high
    }

    fn set_low(&mut self) {
        // Set pin low
    }

    fn is_high(&self) -> bool {
        // Read pin state
        true
    }
}
```

## License

Apache-2.0 (derived from Zephyr driver)
