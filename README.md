# clox-pico

A port of clox from Crafting Interpreters to the Raspberry Pi Pico Microcontroller Family

 - Crafting Interpreters: https://craftinginterpreters.com
 - Raspberry Pi Pico: https://www.raspberrypi.com/products/raspberry-pi-pico/
 - Raspberry Pi Pico 2: https://www.raspberrypi.com/products/raspberry-pi-pico-2/

CMakeLists.txt is intended to be used to create a clox executable via cmake.

CMakePresets.json contains configurations for a host PC ('host') and to build
UF2 format firmware images for the Pico family ('pico', 'pico2' and 'pico2r').

e.g on my (Mac OS) system configured with xcode build tools and cmake:

```
% cmake --preset host .
% cmake --build out/build/host
% ./out/build/host/clox
>
```

## Pico Support

###

There are two changes to the clox from the book. Firstly, a zero parameter main() is provided (HOST_REPL) so that clox runs on a system with no filesystem. Secondly, STACK_MAX is redefined to a smaller size, in order to fit within the 264k RAM on the RP2040.

### Build and Run

The Pico family can host C software built using the PICO SDK. I followed the '[Getting Started](https://www.raspberrypi.com/documentation/microcontrollers/c_sdk.html)' advice and installed it via a VS Code extension. This copied the SDK and toolchains to ~/.pico-sdk on my machine.

There are two Pico boards (Pico and Pico 2). The Pico has an RP2040 microconroller, and the Pico 2 has the RP2350.

The Pico 2's RP2350 contains a pair of ARM cores, and a pair of Risc-V cores. Firmware selects which to use at boot. The cmake configuration files can create three UF2 firmware images to support the two boards:

Building each in turn:

```
// Pico
% cmake --preset pico .
% cmake --build out/build/pico

// Pico 2 ARM
% cmake --preset pico2 .
% cmake --build out/build/pico2

// Pico 2 Risc-V
% cmake --preset pico2r .
% cmake --build out/build/pico2r
```

In each case, you will get a clox.uf2 firmware image. These are installed as usual:

 - Hold down 'bootsel' while connecting the board over USB
 - Copy 'clox.uf2' to the Pico external drive that is mounted in this mode

...the pico will reboot...

 - Use a terminal program to connect to a USB serial port. On my Mac this appeared as '/dev/tty.usbmodem1101', with a baud rate of 115200.

I used the 'serial monitor' in VSCode, and found I needed to set 'Line ending' to either LF or CRLF to interact with the repl. The scripts in specimen/script-one-liners.txt are formatted for use on the repl, which requires one line at a time. Given the time needed to connect the serial monitor, I needed to press enter to get a fresh prompt.

### Language Additions

There are three additions to the Lox language:

 - Support for the notation 0x prefixing a base 16 hex number, wherever numbers can be used.
 - Bitwise operations `<<`, `>>`, `|`, `&`, and `^`, similar to C. They truncate (reporting an error for out of range) a Number to an unsigned 32 bit integer before performing the operation.
 - `peek()` and `poke address, value`. `peek()` reads a 32bit uint from the specified memory address. `poke` writes the value (truncating it to a 32bit uint) to the specified memory address. Intended for manipulating peripheral registers.

 These additions are excercised in specimen/led-on.lox, which will turn on the built in LED on a Pico or Pico 2.

### Performance

This code includes the two optimisations from Chapter 30, and is otherwise unoptimised for these microcontroller cores. On an RP2040, the spec says we have two ARM Cortex-M0+ cores at 133MHz (200MHz after SDK 2.1.1). On an RP2350 we have dual Cortex-M33 or Hazard3 processors at 'up to' 150 MHz. In each case, clox only runs on one core. My host PC (for comparison) runs on an Apple Silicon M2 Max.

Running the fibbonacci micro-benchmark from the book:

```
fun fib(n) {
  if (n < 2) return n;
  return fib(n - 2) + fib(n - 1);
}

var start = clock();
print fib(35);
print clock() - start;
```

Apple Silicon: 2.36sec

| SDK | RP2040 | RP2350 ARM | RP2350 Risc-V |
| --- | --- | --- | --- |
| 2.1.0 | 232 | 95.2 | 122 |
| 2.1.1 | 145 | 94.8 | 122 |

Times are as reported by clox from one run. Units are seconds. The upgrade to SDK 2.1.1 bumped the RP2040 clock speed to 200MHz, and updated the ARM toolchain. The Risc-V toolchain was unchanged.