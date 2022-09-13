# gpioASM

gpioASM is an assembler made for creating simple IO-dependant programs to run on microcontrollers.
Examples would be `blink an led`, `power a motor until an endstop is reached`, `toggle an LED with a button press`, `turn a stepper motor until a hall sensor is triggered` and many more.

This document provides a reference for the available human-readable instructions that are compiled into executable code.

## Datatypes

### Pin states

pin states represent states of pins.
These can be HIGH, LOW or ignore(d).

For `HIGH` valid values are `h`, `1`.

For `LOW` valid values are `l`, `0`.

For `ignore` valid values are `i`, `x`.

Multiple pin states are represented by concatenating with zero or more spaces, for instance `1 0 0 1 x 0`.

### uint16_t

a two-byte, little-endian-encoded integer. Should be exactly two bytes in size.

### uint

an unsigned integer, encoded as varint during compilation

### label

a label attached to one of the commands

## Instructions

| instruction | arguments | description |
| ----------- | --------- | ----------- |
| write_digital | pin states | write states to digital output pins |
| write_analog_channel_0 | uint16_t | write argument to analog channel 0 |
| write_analog_channel_1 | uint16_t | write argument to analog channel 1 |
| write_analog_channel_2 | uint16_t | write argument to analog channel 2 |
| write_analog_channel_3 | uint16_t | write argument to analog channel 3 |
| label | string | create a new label at this code point |
| sleep_ms | uint | sleep for n milliseconds |
| sleep_match_all | pin states | sleep until all input pin states match `arg0` |
| sleep_match_any | pin states | sleep until at least one input input matches `arg0` |
| jump | label | jump to code location |
| jump_match_all | label, pin states | jump to `arg0` only of all input pins match `arg1` |
| jump_match_any | label, pin states | jump to `arg0` if any of the requested input pins match `arg1` |
| jump_count | label, uint | jump to `arg0` max. `arg1` times |
| exit | | terminate code execution