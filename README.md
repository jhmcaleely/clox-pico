# clox-pico

A port of clox from Crafting Interpreters to the Raspberry Pi Pico Microcontroller

Crafting Interpreters: https://craftinginterpreters.com

CMakeLists.txt is intended to be used to create a clox executable via cmake.

e.g on my (Mac OS) system configured with xcode build tools and cmake:

```
% cmake -B out/build/host .
% cmake --build out/build/host
% ./out/build/host/clox
>
```