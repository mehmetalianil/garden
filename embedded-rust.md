# Embedded Rust

## Links

- [the Book](https://docs.rust-embedded.org/book/)
- [the Main STM32 crate](https://github.com/stm32-rs/stm32-rs)
- [the Book](https://docs.rust-embedded.org/book/) 

## Levels
There are different levels of how out code can be integrated into the hardware. 
-  through Peripheral Access Crates, PACs.
-  through Hardware Abstraction Layer crates, HAL crates.
-  through board crates. 

The STM32HAL code in C that most write on is a HAL.
The newer STM32 Cube MX also creates some board level code to work on, we can safely say that they are in higher level. 

PACs, on the other hand, are light wrappers that we don't have much examples for. With a PAC we have a single way to reach each register, but we still need to tap-dance on the registers.

The following is a code that failed to execute this morning, using the PAC. Somehow the registers did not even budge. 

> Warning: Bad code ahead.

```rust
#[entry]
fn start() -> ! {
    // Acquire the device peripherals. They can only be taken once ever.
    let device_peripherals = stm32l4x5::Peripherals::take().unwrap();
    let gpioa = device_peripherals.GPIOA; 
    let gpiob= device_peripherals.GPIOB;
    // let gpioc = device_peripherals.GPIOC;

    // Output speed is low
    gpioa.ospeedr.write(|w| w.ospeedr5().bits(0b00));
    gpiob.ospeedr.write(|w| w.ospeedr14().bits(0b00));
    // GPIOA05 and GPIOB14 are configured as outputs.
    gpioa.moder.write(|w| w.moder5().bits(0b00));
    gpiob.moder.write(|w| w.moder14().bits(0b00));
    // PUPDR
    gpioa.pupdr.write(|w| unsafe {w.pupdr5().bits(0b00)});
    gpiob.pupdr.write(|w| unsafe {w.pupdr14().bits(0b00)});
    // Output type is OD
    gpioa.otyper.write(|w| w.ot5().bit(true));
    gpiob.otyper.write(|w| w.ot14().bit(true));
    
    let mut counter:u32 = 0x0000_ffff;
    //gpioc.moder.write(|w| w.moder13().bits(0b01));

    loop {
        if counter < 0x0000_8fff {
            gpiob.bsrr.write(|w| w.bs14().bit(true));
            gpioa.bsrr.write(|w| w.bs5().bit(true));
        }
        else { 
            gpiob.bsrr.write(|w| w.br14().bit(true));
            gpioa.bsrr.write(|w| w.br5().bit(true));
        }
        counter = counter - 1;
        if counter == 0 {
            counter = 0x0000_ffff;
        }
    }
}
```

## Knurling session 2020 Q4

Knurling is a project by Ferrous systems aimed towards a better embedded rust experience. I tried to follow it to investigate a possible use of nRF52480 in embedded system projects.

## Overview of Embedded Rust

Embedded development has a lot of item in its development pipeline that we just had been taking for granted. We have the IDE, the debugging, drivers for the debuggers, the extensions of the IDE that talk with the debuggers, and sometimes extra glue logic for some of them to be compatible. This ecosystem was given by the vendor, and in many cases in C or in elementary C++. 

On the other hand, the Rust ecosystem is mainly supported by individual open-source maintainers. I will try to list here the tools that I tried out on my nRF52480 development efforts. 

### Debugging chain

There are three ways to debug an embedded microcontroller:

* [Cargo embed](https://github.com/probe-rs/cargo-embed) 
  * with [probe-rs](https://github.com/probe-rs/probe-rs) communicating with the debugger 
  * with winusb generic driver installed on the debugger with [Zadig](https://zadig.akeo.ie/)
  * using either RTT or GDB to communicate, not both. 
  * Has [probe-rs vscode extension](https://github.com/probe-rs/vscode), but seems to be not stable yet. 
* [probe-run](https://github.com/knurling-rs/probe-run) 
  * with [probe-rs](https://github.com/probe-rs/probe-rs) communicating with the debugger 
  * with winusb generic driver installed with [Zadig](https://zadig.akeo.ie/)
  * Using [defmt](https://github.com/knurling-rs/defmt) as a logging framework
* arm gdb server + open ocd
  * with [Open OCD](http://openocd.org/) communicating with the debugger
  * with mainstream drivers.
  * with [cortex-debug](https://github.com/Marus/cortex-debug) interfacing with the VSCode.
  * with [cortex-semihosting](https://github.com/rust-embedded/cortex-m) as logging framework. (albeit slow)


### Crates meant to go to nRF52480

These are meant to be the part of the code.

* [nrf-52840-hal](https://github.com/nrf-rs/nrf-hal)
  * This is the hardware abstraction layer for the nrf-52 series microcontrollers
* [nrf-softdevice](https://github.com/embassy-rs/nrf-softdevice) 
  * Wrapper for nrf softdevices that include the network stack, enabling high level applications use them.
* [embassy](https://github.com/embassy-rs/embassy) library for async/await functions in embedded Rust. 
* [embassy-nrf](https://github.com/embassy-rs/embassy)
* [RTIC](https://github.com/rtic-rs/cortex-m-rtic)