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
