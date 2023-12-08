# gpioASM

gpioASM is an assembler made for creating simple IO-dependant programs to run on microcontrollers.
Examples would be `blink an led`, `power a motor until an endstop is reached`, `toggle an LED with a button press`, `turn a stepper motor until a hall sensor is triggered` and many more.

Alternatively to
1. sending an ON signal
2. waiting for a certain amount of time
3. sending an OFF signal hoping for the connection to still be instact and the delay being precise enough

you can, using this, compile these steps into a tiny program and send the program to the uC.

TOC:

- [Examples](examples/)
- [Instructions reference](docs/GPIO_ASM.md)
- [Executable reference](docs/INSTRUCTIONS.md)

## As seen on

- https://github.com/dakhnod/nRF51-GPIO-BLE-Bridge

## Runtimes

- [C/C++](https://github.com/dakhnod/gpioASM-C-runtime/tree/main)
