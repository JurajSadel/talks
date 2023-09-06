---
marp: true
theme: espressif_training
headingDivider: 1
---

<!-- _class: lead -->
# esp-rs Introduction
## Juraj Sadel

# Goals of this session

- Overview about Rust team
- What do we focus on
- Hosted vs Bare Metal Rust environment

# About the team members

- started in summer 2021, request from customer
- 6 team members + a part-timers
     - Juraj Michálek - Rust project manager, Brno
     - Bjoern Quentin - Germany
     - Jesse Braham - Canada
     - Juraj Sadel - Brno
     - Sergio Gasquez - Spain
     - Scott Mabin - UK
     - Kirill Mikhailov - part-time, Brno
     - Samuel Benko - internship, Brno

# What we do
- enabling Rust language on ESP chips (compiler, tooling, peripheral/networking drivers)
- Xtensa and RISC-V architecture
- `std` and `no_std` approach
- exploring third option: ESP-IDF components written in Rust: https://github.com/espressif/rust-esp32-example

# Compiler

- 6 weeks release cycle
- RISC-V upstream, for Xtensa we have fork

## Tooling

- [espflash](https://github.com/esp-rs/espflash)
- [espup](https://github.com/esp-rs/espup)
- [esp-flasher-stub](https://github.com/esp-rs/esp-flasher-stub)

# Hosted Rust environment

- ESP-IDF provides a `newlib` environment, we build `standard library` on top of it
- Rust wrapper for the ESP-IDF
- [Using the std library](https://esp-rs.github.io/book/overview/using-the-standard-library.html#using-the-standard-library-std)
- Zephyr, Nuttx could be used instead of ESP-IDF
- https://github.com/esp-rs/std-training

# `std` blinky

```rust
use esp_idf_hal::delay::FreeRtos;
use esp_idf_hal::gpio::*;
use esp_idf_hal::peripherals::Peripherals;

fn main() -> anyhow::Result<()> {

    let peripherals = Peripherals::take().unwrap();
    let mut led = PinDriver::output(peripherals.pins.gpio4)?;

    loop {
        led.set_high()?;
        // we are sleeping here to make sure the watchdog isn't triggered
        FreeRtos::delay_ms(1000);

        led.set_low()?;
        FreeRtos::delay_ms(1000);
    }
}
```

# Bare Metal Rust environment

- no layer between HW and user's program
- pure Rust
- `core` library instead standard library
- most of the team works on no_std
- https://github.com/esp-rs/no_std-training

# `no_std` blinky

```rust
#![no_std]
#![no_main]

use esp32c3_hal::{clock::ClockControl, gpio::IO, peripherals::Peripherals, prelude::*, Delay};
use esp_backtrace as _;

#[entry]
fn main() -> ! {
    let peripherals = Peripherals::take();
    let system = peripherals.SYSTEM.split();
    let clocks = ClockControl::boot_defaults(system.clock_control).freeze();

    // Set GPIO5 as an output, and set its state high initially.
    let io = IO::new(peripherals.GPIO, peripherals.IO_MUX);
    let mut led = io.pins.gpio5.into_push_pull_output();

    led.set_high().unwrap();

    // Initialize the Delay peripheral, and use it to toggle the LED state in a
    // loop.
    let mut delay = Delay::new(&clocks);

    loop {
        led.toggle().unwrap();
        delay.delay_ms(500u32);
    }
}
```

# Links

- [esp-rs organisation](https://github.com/esp-rs)
- [The esp book](https://esp-rs.github.io/book/)
- [Rust on wokwi](https://wokwi.com/rust)
