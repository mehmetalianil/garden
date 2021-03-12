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