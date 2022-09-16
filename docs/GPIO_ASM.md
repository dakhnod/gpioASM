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

Multiple pin states are represented by concatenating , for instance `1001-0`.

### uint16_t

a two-byte, little-endian-encoded integer. Should be exactly two bytes in size.
Written as a decimal number, ranging from 0 - 65535.

### varint

an unsigned integer, encoded as varint during compilation.
Written as a decimal integer.

### string

a combination of characters, written without quotes.

## Instructions

| instruction | arguments | description |
| ----------- | --------- | ----------- |
| [`write_digital`](#write-digitals) | [pin states](#pin-states) | write states to digital output pins |
| [`write_analog_channel_0`](#write-analog-channel) | [uint16_t](#uint16_t) | write argument to analog channel 0 |
| [`write_analog_channel_1`](#write-analog-channel) | [uint16_t](#uint16_t) | write argument to analog channel 1 |
| [`write_analog_channel_2`](#write-analog-channel) | [uint16_t](#uint16_t) | write argument to analog channel 2 |
| [`write_analog_channel_3`](#write-analog-channel) | [uint16_t](#uint16_t) | write argument to analog channel 3 |
| [`sleep_ms`](#sleeping-for-constant-time) | [varint](#varint) | sleep for n milliseconds |
| [`sleep_match_all`](#await-pin-states) | [pin states](#pin-states) | sleep until all input [pin states](#pin-states) match `arg0` |
| [`sleep_match_any`](#await-at-least-one-pin-state) | [pin states](#pin-states) | sleep until at least one input input matches `arg0` |
| [`sleep_match_all_timeout`](#await-with-timeout) | [pin states](#pin-states), [varint](#varint) | sleep until all input [pin states](#pin-states) match `arg0`, at most `arg1`  milliseconds |
| [`sleep_match_any_timeout`](#await-with-timeout) | [pin states](#pin-states), [varint](#varint) | sleep until at least one input input matches `arg0`, at most `arg1` milliseconds |
| [label](#jumping) | [string](#string) | create a new [label](#string) at this code point |
| [`jump`](#jumping) | [label](#string) | jump to code location |
| [`jump_match_all`](#jumping-conditionally) | [label](#string), [pin states](#pin-states) | jump to `arg0` only of all input pins match `arg1` |
| [`jump_match_any`](#jumping-conditionally) | [label](#string), [pin states](#pin-states) | jump to `arg0` if any of the requested input pins match `arg1` |
| [`jump_count`](#jumping-n-times) | [label](#string), [uint](#varint) | jump to `arg0` max. `arg1` times |
| [`exit`](#terminating-code-execution) | | terminate code execution

### Write digitals

`write_digital` reads in an arbitary number of [pin states](#pin-states) and applies them to configured pins.
If fewer [pin states](#pin-states) are given than output pins configured, remaining states get filled up with '-'.
If too many [pin states](#pin-states) are given, excess states get cut off.
Let's assume you have this configuration:
```
pin 10: output digital
pin 11: output digital
pin 13: input
pin 15: output digital
```
The code `digital_write 1` would turn pin 10 HIGH.
`digital_write -0` would leave pin 10 untouched, turn pin 11 LOW.
`digital_write ---` would not change any pin states.
`digital_write ---1` would also not have any effect, since the last `1` gets cut off
due to only three configured output pins.

### Write analog channel

`write_analog_channel_0` outputs a PWM signal on a configured analog output pin.
Let's assume the following configuration:
```
pin 10: output analog
pin 11: output digital
pin 12: input
pin 13: output analog
```
`analog_write_channel_0 1500` would output a duty cycle of 1500us on pin 10.
Since the chip is configured with a servo friendly frequency of 50Hz, 1500us equals a duty cycle of 1.5%, or 90° of a standard servo.

`analog_write_channel_1 1000` would output a duty cycle of 1500us on pin 13.
This would turn a standard servo to around 0°.

`analog_write_channel_2 1000` ...I don't really know what would happen here since there are only two analog outputs configured...

### Sleeping for constant time

`sleep_ms 1000` would just block code execution for 1000ms.
The next command, if there is any, would be executed with a delay.
Knowing everything we know so far, we can make an LED blink in the following pattern:
- 1000ms off
- 100ms on
```
write_digital 0
sleep_ms 1000
write_analog 1
sleep_ms 100
```
As you might notice this pattern does not repeat itself.
For repetitions we need to introduce [jumping](#jumping).

### Await pin states

`sleep_match_all` makes the code block until certain criteria are met on input pins.
This is handy if you want to wait for a button press, for instance.
The code takes in [pin states](#pin-states) and creates a filter out of those states.
This call returns immediately if all criteria are already met.

Examples:

Here's our given setup:

```
pin 20: input
pin 21: output
pin 22: input
```

| code | description |
| ---- | ----------- |
| `sleep_match_all 1` | waits for pin 20 to become HIGH |
| `sleep_match_all 0` | waits for pin 20 to become LOW |
| `sleep_match_all 11` | ignores pin 20 __and__ 22 to become HIGH |
| `sleep_match_all -1` | ignores pin 20, waits for pin 22 to become HIGH |
| `sleep_match_all --` | ignores both pins, returns immediately |
| `sleep_match_all 1-` | ignores pin 22, same as `sleep_match_all 1` |

If you want to have all of your criteria to match `sleep_match_all` is your call.
If one matching input is enough, we need to introduce `sleep_match_any`.

### Await at least one pin state

`sleep_match_any` has the same syntax and behaviour as `sleep_match_all`,
with the difference that is does not wait for all input states to match.
It continues code execution when at least one [pin states](#pin-states) matches.
This is handy if you have multiple buttons, and need to react to any of those buttons being pressed, for instance.
Examples:

Here's our given setup:

```
pin 20: input
pin 21: output
pin 22: input
```

| code | description |
| ---- | ----------- |
| `sleep_match_any 1` | waits for pin 20 to become HIGH |
| `sleep_match_any 0` | waits for pin 20 to become LOW |
| `sleep_match_any 11` | waits for pin 20 __or__ pin 22 to become HIGH |
| `sleep_match_any --` | ignores both pins, returns immediately |
| `sleep_match_any 10` | waits for pin 20 to become HIGH __or__ pin 22 to become LOW |

In our example we light up an LED until any button is pressed:

```
write_digital 1
sleep_match_any 11111
write_digital 0
```

### Await with timeout

Sometimes, you want to wait for a certain input state, but don't want to wait indefinetly.
In this case, `sleep_match_all_timeout` and `sleep_match_any_timeout` enter the stage.
They take an additional parameter `timeout` and wait for either the inputs to fulfil conditions __or__ the timeout to expire.

Here's our given setup:

```
pin 20: input
pin 21: output
pin 22: input
```

| code | description |
| ---- | ----------- |
| `sleep_match_any_timeout 1 20000` | waits for pin 20 to become HIGH. Returns if nothing happened after 20s |
| `sleep_match_any_timeout 0 1000` | waits for pin 20 to become LOW. Returns if nothing happened after 1s |
| `sleep_match_any_timeout 11 500` | waits for pin 20 __or__ pin 22 to become HIGH. Returns after 500ms |
| `sleep_match_all_timeout 11 500` | waits for pin 20 __and__ pin 22 to become HIGH. Returns after 500ms |

In our example we light up an LED until any button is pressed. After 60s we turn the led off regardless:

```
write_digital 1
sleep_match_any_timeout 11111 60000
write_digital 0
```

### Jumping

Often times you want to repeat your code or just run another piece of code without having to execute everything in between.
That is where _jumping_ comes in handy.
The `jump` code reads in a string that you have defined earlier as a [label](#string).
Code execution will then continue after the [label](#string).
Let's return to our [`write_digital`](#write-digitals) example, but make it repeat this time:

```
[label](#string) start
write_digital 0
sleep_ms 1000
write_analog 1
sleep_ms 100
jump start
```

As you can see we define the [label](#string) start at the beginning, so our `jump`-call in the last line knows what code to continue with.
Labels are replaced with executable offsets during compilation.

### Jumping n times

In our previous example we got out led to blink indefinetly.
Let's now say that we want out led to only blink 10 times.
This is where `jump_count` enters the stage.
The internal runtime counter starts off disabled.
When a `jump_count`-call is first encountered, the internal counter takes in `arg1` and ties itself to that `jump-count` command.

As long as no other jump is encountered, the following encounters of that `jump-count`-call count down the counter and jump to [label](#string) `arg0`, unless the counter is 0.
When the counter becomes 0, code execution continues after the `jump-count` call.

Now we want out LED to only blink 10 times:

```
[label](#string) start
write_digital 0
sleep_ms 1000
write_analog 1
sleep_ms 100
jump_count start 10
```

### Jumping conditionally

`jump_match_any` and `jump_match_all` are used when you only want to perform a jump when certain input criteria are met.
They take in a jump [label](#string) and [pin states](#pin-states) to compare to.

Let's say we only want out blinking pattern to repeat if no button is pressed:

```
[label](#string) start
write_digital 0
sleep_ms 1000
write_analog 1
sleep_ms 100
jump_match_all start 0000
```

### Terminating code execution

`exit` takes in no arguments and prevents any more code from being executed.