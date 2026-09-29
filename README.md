# Bare Metal Pong OS

This is a custom, hardware-accelerated Operating System written entirely from scratch in Rust, without any standard libraries, hardware abstraction crates, or API calls. It boots directly on an x86_64 PC via BIOS and runs a 2-player/Single-player game of Pong.

## Features (100% Raw Implementation)
- **Zero OS Dependencies:** Built using `#![no_std]` and `#![no_main]`. 
- **Raw Assembly Interrupts:** Custom written `global_asm!` naked wrappers to safely catch and handle CPU hardware interrupts via `iretq`, saving all System V ABI caller registers.
- **Hardware Graphics:** Direct memory mapped I/O rendering. The OS writes bytes directly into the physical `0xA0000` memory address of the VGA graphics card (Mode 13h, 320x200, 256 colors).
- **Custom Fonts:** Implements a custom rendering pipeline to draw 8x8 font glyphs scaled to 16x16.
- **Direct Port I/O:** Uses raw `in` and `out` assembly instructions to talk to motherboard chips.
- **Intel 8259 PIC:** Fully custom Programmable Interrupt Controller mapping to prevent IRQ collisions with CPU exception faults.
- **Interrupt Driven Input:** Instead of polling, the Intel 8042 PS/2 controller physically halts the CPU when a key is pressed, firing a custom IDT interrupt (IRQ1) to update keyboard state.
- **Hardware Timed Physics:** The game loop uses the CPU's `hlt` sleep instruction, perfectly syncing the physics engine to the ~18.2 Hz motherboard hardware clock timer (IRQ0).
- **PC Speaker Audio:** Generates square waves by programming the motherboard's Programmable Interval Timer (PIT) Channel 2 and toggling port `0x61`.

## How to Build
Due to the bleeding-edge nature of bare metal Rust, this requires a specific nightly compiler version to compile the bootable image correctly:

```bash
rustup override set nightly-2023-10-01-x86_64-pc-windows-msvc
cargo install bootimage --version 0.10.2
cargo bootimage
```

This will produce a raw `.bin` file in `target/x86_64-my_os/debug/bootimage-my_os.bin`. 

## How to Run
You can run this on any emulator (QEMU, VMware, VirtualBox) by mounting the generated `.bin` file as an IDE hard drive.

To run it on **real hardware**, you can use a tool like [Rufus](https://rufus.ie/en/) or `dd` to flash the generated `.bin` file to a USB flash drive, plug it into a laptop or PC, and boot from it!

## Controls
- **Human Player (Left):** `W` (Up) and `S` (Down)
- **AI Player (Right):** Automatically tracks the ball.

## Architecture Notes
The only dependencies used in this project are:
- `bootloader` (0.9.35): Required to write the initial 512-byte 16-bit real-mode assembly boot sector (which the Rust compiler physically cannot generate) to switch the CPU into 64-bit Long Mode and enable the VGA framebuffer.
- `font8x8`: Used strictly as a static data array (0s and 1s) representing the shapes of characters.

Everything else—from the memory layouts to the IDT configuration—is entirely handwritten.
