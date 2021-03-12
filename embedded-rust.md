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

PACs, on the other hand, are light wrappers that we don't have much examples for. With a PAC we have a single way to reach each register, but we still need to do the dance of register writes.