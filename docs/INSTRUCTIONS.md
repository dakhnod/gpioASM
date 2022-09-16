# GPIO opcodes (gpioASM)

Binary executables can be compiled from gpioASM for tasks like `power a motor until an endstop is reached`, `toggle an LED with a button press`, `turn a stepper motor until a hall sensor is triggered` and many more.

Here are the available instructions:

| command | opcode | arguments |
| ------- | ------- | --------- |
| [`write_digital`](GPIO_ASM.md#write-digitals) | `0b00000000` | [digital bits](#digital-bits) |
| [`write_analog_channel_0`](GPIO_ASM.md#write-analog-channel) | `0b00010000` | [uint16_t](#uint16t) |
| [`write_analog_channel_1`](GPIO_ASM.md#write-analog-channel) | `0b00010001` | [uint16_t](#uint16t) |
| [`write_analog_channel_2`](GPIO_ASM.md#write-analog-channel) | `0b00010010` | [uint16_t](#uint16t) |
| [`write_analog_channel_3`](GPIO_ASM.md#write-analog-channel) | `0b00010011` | [uint16_t](#uint16t) |
| [`sleep_ms`](GPIO_ASM.md#sleeping-for-constant-time) | `0b00100000` | [varint](#varint) |
| [`sleep_match_all`](GPIO_ASM.md#await-pin-states) | `0b00100001` | [digital bits](#digital-bits) |
| [`sleep_match_any`](GPIO_ASM.md#await-at-least-one-pin-state) | `0b00100010` | [digital bits](#digital-bits) |
| [`sleep_match_all_timeout`](GPIO_ASM.md#await-with-timeout) | `0b00100100` | [digital bits](#digital-bits) |
| [`sleep_match_any_timeout`](GPIO_ASM.md#await-with-timeout) | `0b00100101` | [digital bits](#digital-bits) |
| [`jump`](GPIO_ASM.md#jumping) | `0b01000000` | [varint](#varint) |
| [`jump_match_all`](GPIO_ASM.md#jumping-conditionally) | `0b01000001` | [varint](#varint), [digital bits](#digital-bits) |
| [`jump_match_any`](GPIO_ASM.md#jumping-conditionally) | `0b01000010` | [varint](#varint), [digital bits](#digital-bits) |
| [`jump_count`](GPIO_ASM.md#jumping-n-times) | `0b01001000` | [varint](#varint), [varint](#varint) |
| [`exit`](GPIO_ASM.md#terminating-code-execution) | `0b10000000` | |

## Datatypes

### digital bits

Digital bits represent logical states of digital GPIO pins.
Every digital pin can be in one of four states:

| state bits | state description |
| --- | --- |
| `0b00` | pin is LOW |
| `0b01` | pin is HIGH |
| `0b10` | pin is high-impedance |
| `0b11` | pin is not available or ignored |

Pin states are represented via bytes.
Every byte can house up to four pin states, unconfigured pin bits are set to `0b11`.
For example, five pins, states [HIGH, LOW, LOW, LOW, HIGH] are represented by two bytes, and look like this:

`0b01000000 01111111`

The amount of bytes to send depends on the amount of configured digital output pins.
If the chip boots with 6 digital output pins configured, it always expects a digital payload sized 2 bytes (`CEIL(pin_count / 4)`).

### varint

Varints are encoded integers that grow bytewise in size with higher integer values.
Best described [here](https://developers.google.com/protocol-buffers/docs/encoding).

### uint16_t

unsigned integers, range from 0 - 65535, encoded as 16-bit little-endian.

## Code structure

Every command is encoded as its opcode and the arguments.
No length or padding is encoded along.

`write_digital 1001` becomes `0b00000000 01000001`.
`write_analog_channel_1 1000` becomes `0b00000000 00000011 11101000`.

```
label start
write_digital 1
sleep_ms 100
write_digital 0
sleep_ms 100
jump start
```
becomes
```
wr.-dig.   pin#0 1  sleep    100ms    wr.-dit. pin#0 0  sleep    100ms    jump     addr 0
0b00000000 01111111 00100000 01100100 00000000 00111111 00100000 01100100 01000000 00000000
```