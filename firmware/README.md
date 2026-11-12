# ESC Firmware Starter & Target Definition

Bring-up, peripheral mapping, and target configuration for the **Custom 35A 3S-6S Single BLDC ESC** powered by the **STM32G071KBT6** 64MHz 32-bit ARM Cortex-M0+ microcontroller and **FD6288Q** 3-phase gate driver.

This configuration is compatible with open-source ESC firmware including **AM32**, **BLHeli_32**, or bare-metal custom BLDC motor control pipelines.

---

## Hardware Pinout & Peripheral Mapping

| MCU Pin | Pin Name | Peripheral Signal | Connected Net | Description / Function |
| :---: | :---: | :---: | :---: | :--- |
| **19** | `PA8` | `TIM1_CH1` | `/HIN_A` | Phase A High-Side Gate Drive Input |
| **20** | `PA9` | `TIM1_CH2` | `/HIN_B` | Phase B High-Side Gate Drive Input |
| **21** | `PA10` | `TIM1_CH3` | `/HIN_C` | Phase C High-Side Gate Drive Input |
| **26** | `PB13` | `TIM1_CH1N` | `/LIN_A` | Phase A Low-Side Gate Drive Input |
| **27** | `PB14` | `TIM1_CH2N` | `/LIN_B` | Phase B Low-Side Gate Drive Input |
| **28** | `PB15` | `TIM1_CH3N` | `/LIN_C` | Phase C Low-Side Gate Drive Input |
| **6** | `PA0` | `ADC1_IN0` | `/BEMF_A` | Phase A Back-EMF Voltage Divider (10k / 1.2k) |
| **7** | `PA1` | `ADC1_IN1` | `/BEMF_B` | Phase B Back-EMF Voltage Divider (10k / 1.2k) |
| **8** | `PA2` | `ADC1_IN2` | `/BEMF_C` | Phase C Back-EMF Voltage Divider (10k / 1.2k) |
| **11** | `PA5` | `ADC1_IN5` | `/BEMF_NEUTRAL` | Virtual Star Neutral Voltage Reference (10k x3) |
| **10** | `PA4` | `ADC1_IN4` | `/CURRENT_SENSE` | INA180 Current Sense Amp Output (Gain 20V/V across 0.5mΩ shunt) |
| **9** | `PA3` | `ADC1_IN3` | `/VBAT_SENSE` | Battery Voltage Divider (10k / 1.2k) |
| **18** | `PB7` | `TIM17_CH1` / EXTI | `/PWM_IN` | DShot150/300/600 or PWM Motor Signal Input |
| **12** | `PA6` | `USART1_TX` | `/TELEMETRY` | Bidirectional ESC Telemetry (Current, Voltage, RPM, Temp) |
| **14** | `PB0` | `GPIO_Output` | `/LED_STATUS` | Active-High Green Status Indicator LED |
| **23** | `PA13` | `SWDIO` | `/SWDIO` | Serial Wire Debug Data Pin |
| **24** | `PA14` | `SWCLK` | `/SWCLK` | Serial Wire Debug Clock Pin |
| **4** | `NRST` | `RESET` | `/NRST` | Hardware Microcontroller Reset with 100nF filter |

---

## Timer Configuration (TIM1 Motor Commutation)

* **Clock:** SystemCoreClock = 64 MHz
* **Prescaler:** `0` (running at full 64 MHz timer clock)
* **PWM Frequency:** 24 kHz to 48 kHz (Center-Aligned Mode `TIM_COUNTERMODE_CENTERALIGNED1`)
* **Dead-Time Insertion:** Configured in `TIM1->BDTR` register (hardware dead-time = ~150ns) to match the FD6288Q gate propagation delays and prevent bridge shoot-through.

---

## AM32 / BLHeli Target Header (`target.h`)

See [`target.h`](target.h) for compile-time macro definitions ready for AM32 build systems:
- Timer assignments
- Inverting vs non-inverting driver polarity
- BEMF comparator channels
- Current shunt milliohm value (0.5 mΩ)
- Voltage divider ratio (10k / 1.2k)

---

## Flashing & Debugging via SWD

Use OpenOCD or ST-Link CLI with the exposed programming test pads (`TP1: SWDIO`, `TP2: SWCLK`, `TP3: NRST`, `TP4: +3V3`, `TP5: GND`):

```bash
# Flash AM32 firmware binary using openocd
openocd -f interface/stlink.cfg -f target/stm32g0x.cfg -c "program custom_esc_35a_am32.bin verify reset exit 0x08000000"
```
