# Precision Multi-Timer Interrupt System with STM32F103C8T6

An STM32CubeIDE project demonstrating three independent hardware timer update interrupts on an STM32F103C8T6. Each interrupt toggles a separate GPIO output, allowing three timing signals to run without delay loops in the empty main loop.

## Repository contents

- `cubeIDE/` - complete STM32CubeIDE project named `te`, including HAL source, startup files, linker scripts, and the `cubeIDE/te.ioc` CubeMX configuration.
- `Rapport_Timer_Interrupt.pdf` - project report and timer-configuration discussion.
- `stm32.pdsprj` - Proteus simulation project.
- `test.pdsprj.DESKTOP-8TMAMQT.mohamed.workspace` - local Proteus workspace metadata.

## Target and I/O

- MCU: STM32F103C8T6
- System clock: 72 MHz from an 8 MHz HSE through PLL x9
- Timers: TIM2, TIM3, and TIM4 in up-counting base-timer mode
- Outputs: PA2, PA3, and PA4 configured as push-pull GPIO outputs

| Timer | Output toggled by callback | Prescaler (PSC) | Period (ARR) |
| --- | --- | ---: | ---: |
| TIM2 | PA2 | 7199 | 1999 |
| TIM3 | PA3 | 7199 | 500 |
| TIM4 | PA4 | 7199 | 700 |

The code starts all three timers with `HAL_TIM_Base_Start_IT()` and dispatches them in `HAL_TIM_PeriodElapsedCallback()`.

## Actual timing in the checked-in firmware

With a 72 MHz timer input and `PSC = 7199`, the counter frequency is 10 kHz, so one count is 0.1 ms. STM32 auto-reload counters include count zero, making the update interval:

```text
update interval = (ARR + 1) / 10,000 seconds
```

| Timer | Checked-in ARR | Interrupt/output-toggle interval | Full output cycle |
| --- | ---: | ---: | ---: |
| TIM2 | 1999 | 200 ms | 400 ms |
| TIM3 | 500 | 50.1 ms | 100.2 ms |
| TIM4 | 700 | 70.1 ms | 140.2 ms |

The full output cycle is twice the interrupt interval because each interrupt only toggles the pin once.

### Report discrepancy

The report discusses intended periods of approximately 200 ms, 500 ms, and 700 ms and shows timer values that are not fully consistent with that description. The authoritative checked-in CubeMX and C configuration is `ARR = 1999`, `500`, and `700`, which produces approximately 200 ms, 50.1 ms, and 70.1 ms between toggles at the configured 10 kHz counter rate. For 500 ms and 700 ms update intervals at that rate, ARR would need to be 4999 and 6999 respectively. This README documents the mismatch; it does not change the firmware.

## Build and run

1. Install STM32CubeIDE.
2. Import `cubeIDE/` as an existing STM32CubeIDE project.
3. Confirm the target board has the expected STM32F103C8T6 and an 8 MHz external clock source compatible with the project configuration.
4. Connect LEDs or measurement equipment to PA2, PA3, and PA4. Use a suitable series resistor for each LED and share ground.
5. Build the `te` project and flash it with a compatible debugger/programmer, such as an ST-LINK configured for the target board.
6. Observe the outputs with LEDs, an oscilloscope, or a logic analyzer. A scope or logic analyzer is recommended for verifying the shorter intervals.

The Proteus project can be opened separately in a compatible Proteus installation if simulation is preferred. The exact Proteus and STM32CubeIDE versions used to create the artifacts are not recorded.

## Notes and limitations

- Changing the system clock, APB1 prescaler, timer prescaler, or ARR changes the timing; recalculate from the actual timer clock rather than assuming it equals the CPU clock.
- LED visual behavior represents output toggles, not one interrupt per visible on/off cycle.
- Generated debug/build artifacts are present in the CubeIDE tree and may not be portable across tool versions.
- Verify voltage levels and board pin labels before connecting external equipment. STM32 GPIO is 3.3 V and is not 5 V power logic.
