# SIRC Decoder

Captures and decodes Sony SIRC infrared remote control signals (12, 15 and 20-bit) on an STM32 Nucleo-F446RE, without using an IR library.

## Hardware
- STM32 Nucleo-F446RE
- VS1838B-type 38 kHz IR receiver module
- Receiver output connected to **D2 (PA10)**, configured as a GPIO input
- Output is printed over the ST-Link virtual COM port (USART2, 115200 baud)

## How it works
1. **Microsecond timer** – TIM2 runs from the 84 MHz APB1 timer clock with a prescaler of 83, so it counts once per microsecond.
2. **Waiting for a quiet line** – before each capture, the program waits until the receiver has been idle for 10 ms, so it never starts recording in the middle of a code.
3. **Capturing** – the program polls the receiver and records the time between every change of state (the length of each burst and gap). A capture ends when the line has been idle for more than 10 ms.
4. **Decoding** – SIRC uses pulse-width encoding: after a ~2400 µs start burst, each bit is a ~600 µs gap followed by a burst of ~600 µs (0) or ~1200 µs (1). Bursts longer than 900 µs are read as 1s. Bits are sent least significant bit first.
5. **Splitting the code** – the number of durations identifies the format:
   - 25 durations → 12-bit: 7-bit command + 5-bit device
   - 31 durations → 15-bit: 7-bit command + 8-bit device
   - 41 durations → 20-bit: 7-bit command + 5-bit device + 8-bit extended

## Example output
```
2376
619
1177
617
579
640
1157
639
558
639
559
639
558
639
559
619
1178
639
1157
619
1177
640
1156
641
1157
639
557
641
558
640
557
640
559
640
557
640
557
642
1156
640
556
--- 41 durations ---
20-bit: Extended --- 64 Device --- 31 Command --- 5

```

## Problems I solved
**Partial captures:** some buttons gave random numbers of durations. Sony remotes repeat each code about every 45 ms, and printing the results took longer than the gap between repeats, so the next capture started partway through a code. I fixed this by waiting for the line to be quiet for 10 ms before each capture.

## Building
The `Drivers/` folder is not included. Open `SIRCDecoder.ioc` in STM32CubeMX and click **Generate Code** to regenerate it, then import the project into STM32CubeIDE.

## Next steps
- Transmitting codes with an IR LED and a 38–40 kHz PWM carrier
- A learning remote that records and replays any IR signal