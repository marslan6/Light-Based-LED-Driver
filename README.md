# Intelligent Ambient Light Sensing and Control System

[![Microcontroller](https://img.shields.io/badge/MCU-TI%20Tiva%20TM4C123-red)](https://www.ti.com/product/TM4C123GH6PM)
[![Architecture](https://img.shields.io/badge/ARM-Cortex--M4-blue)](https://developer.arm.com/ip-products/processors/cortex-m/cortex-m4)
[![Language](https://img.shields.io/badge/Language-C%20%7C%20ARM%20Assembly-green)]()

An embedded firmware project for the Texas Instruments Tiva TM4C123GH6PM microcontroller that implements an adaptive lighting system with real-time ambient light sensing, threshold-based RGB LED control, and PWM-driven external LED brightness regulation.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Hardware Components](#hardware-components)
- [System Architecture](#system-architecture)
- [Technical Specifications](#technical-specifications)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Functionality](#functionality)
- [Performance Metrics](#performance-metrics)
- [Author](#author)

---

## Overview

This project demonstrates a complete embedded systems implementation featuring multi-protocol peripheral communication, real-time sensor data processing, and closed-loop feedback control. The system continuously monitors ambient light conditions using a TSL2561 dual-channel light sensor and automatically adjusts LED outputs based on user-configurable thresholds.

**Key Capabilities:**
- Real-time light intensity measurement (15-1700+ lux range)
- Threshold-based state machine with three operating modes
- Dynamic PWM brightness control proportional to luminosity
- User-adjustable sensitivity via potentiometer (±500 lux window)
- Real-time LCD display of luminosity and threshold values
- Serial debugging output via UART

---

## Features

### Core Functionality
- **Ambient Light Sensing**: TSL2561 dual-channel I2C sensor with 256-sample averaging for noise immunity
- **Multi-Color LED Indication**: RGB LED indicates low (red), normal (green), or high (blue) light conditions
- **PWM Brightness Control**: Timer-based PWM with interrupt-driven duty cycle modulation (0-100%)
- **LCD Display Interface**: Nokia 5110 compatible 84×48 pixel display via SPI
- **Configurable Thresholds**: Analog potentiometer sets sensitivity range via 12-bit ADC
- **Serial Communication**: UART output for debugging and monitoring (9600 baud)

### Technical Highlights
- **Mixed-Language Development**: C for control logic, ARM Thumb-2 assembly for performance-critical operations
- **Multi-Protocol Integration**: Simultaneous I2C, SPI, ADC, UART, and Timer peripherals
- **Interrupt-Driven Design**: Timer0A ISR for precise PWM generation
- **Custom LCD Driver**: Assembly-based bitmap rendering with predefined character sets
- **Sensor Calibration**: Proprietary lux calculation algorithm from TSL2561 datasheet

---

## Hardware Components

| Component | Model/Type | Interface | Function |
|-----------|-----------|-----------|----------|
| **Microcontroller** | TI Tiva TM4C123GH6PM | - | ARM Cortex-M4, 32-bit, 16 MHz |
| **Light Sensor** | TSL2561 | I2C3 (PD0/PD1) | Dual-channel ambient light sensing |
| **LCD Display** | Nokia 5110 / PCD8544 | SPI/SSI0 (PA2/3/5/6/7) | 84×48 pixel monochrome display |
| **Potentiometer** | 50kΩ Linear | ADC0 (PE3) | Threshold adjustment |
| **RGB LED** | On-board | GPIO (PF1/2/3) | Status indication |
| **External LED** | Generic | PWM via PB3 | Brightness control output |
| **Push Buttons** | On-board | GPIO (PB4-7) | User input (optional) |

### Pin Configuration

#### I2C3 (Light Sensor)
- **PD0**: SCL (Clock)
- **PD1**: SDA (Data)
- **Slave Address**: 0x39

#### SPI/SSI0 (LCD Display)
- **PA2**: CLK (Clock - 4 MHz)
- **PA5**: Din (MOSI)
- **PA7**: DC (Data/Command)
- **PA3**: CE (Chip Enable)
- **PA6**: RESET

#### ADC0 (Potentiometer)
- **PE3**: Analog Input (12-bit, 125 kSPS)

#### GPIO (LEDs & Outputs)
- **PF1**: Red LED (Low light indicator)
- **PF2**: Blue LED (High light indicator)
- **PF3**: Green LED (Normal light indicator)
- **PB3**: PWM Output (External LED driver)

#### UART0 (Serial Debug)
- **PA0**: RX
- **PA1**: TX (9600 baud, 8-N-1)

---

## System Architecture

### Data Flow Diagram

```
┌─────────────────┐
│  TSL2561 Sensor │ (I2C3)
│  Dual Channel   │
└────────┬────────┘
         │ Raw CH0/CH1 Data
         ↓
┌────────────────────────┐
│  256-Sample Averaging  │
│  CalculateLux()        │
└────────┬───────────────┘
         │ Luminosity (lux)
         ↓
    ┌────┴─────────────────────┬──────────────┐
    ↓                          ↓              ↓
┌─────────────┐      ┌──────────────┐   ┌─────────────┐
│ Threshold   │      │   RGB LED    │   │  PWM Timer  │
│ Comparison  │      │   Control    │   │  (Timer0A)  │
└─────┬───────┘      └──────────────┘   └──────┬──────┘
      │                                         │
      ↓                                         ↓
┌─────────────┐                        ┌────────────────┐
│ LCD Display │ (SPI0)                 │  External LED  │
│  (84×48)    │                        │  (Brightness)  │
└─────────────┘                        └────────────────┘

┌──────────────┐
│ Potentiometer│ (ADC0)
└──────┬───────┘
       │ Resistance → Voltage
       ↓
┌───────────────────┐
│ Threshold         │
│ Calculation       │
│ LT = (R×0.034)-500│
│ HT = (R×0.034)+500│
└───────────────────┘
```

### State Machine

The system operates in three states based on luminosity vs. thresholds:

```
         ┌─────────────────┐
         │  LOW LIGHT      │
Lux < LT │  Red LED ON     │
         │  PWM: Low Duty  │
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │  NORMAL LIGHT   │
LT ≤ Lux │  Green LED ON   │
    ≤ HT │  PWM: Med Duty  │
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │  HIGH LIGHT     │
Lux > HT │  Blue LED ON    │
         │  PWM: High Duty │
         └─────────────────┘
```

---

## Technical Specifications

### Microcontroller Configuration
- **Core**: ARM Cortex-M4 (32-bit RISC)
- **Clock**: 16 MHz system clock
- **Architecture**: Harvard architecture with Thumb-2 instruction set
- **Memory**: SRAM for data, Flash for program storage

### Communication Protocols

#### I2C3 (TSL2561 Sensor)
- **Speed**: 100 kHz (standard mode)
- **Addressing**: 7-bit (0x39)
- **Registers**:
  - 0x80: Power control (0x03 = power on)
  - 0x8C/0x8D: Channel 0 Low/High byte
  - 0x8E/0x8F: Channel 1 Low/High byte
- **Sampling**: 256 samples per cycle for averaging

#### SPI/SSI0 (LCD Display)
- **Speed**: 4 MHz clock
- **Mode**: SPI Mode 0 (CPOL=0, CPHA=0)
- **Data**: 8-bit transfers
- **Protocol**:
  - DC=0, CE=0: Command mode
  - DC=1, CE=1: Data mode

#### ADC0 (Potentiometer)
- **Resolution**: 12-bit (0-4095)
- **Sample Rate**: 125 kSPS
- **Sequencer**: SS3 (Sample Sequencer 3)
- **Reference**: 3.3V

#### Timer0A (PWM Generation)
- **Mode**: Periodic countdown (16-bit)
- **Prescaler**: 16 (1 μs resolution)
- **Frequency**: ~1 kHz
- **Duty Cycle**: Dynamic (0-100% based on luminosity)

#### UART0 (Serial Debug)
- **Baud Rate**: 9600 bps
- **Format**: 8 data bits, 1 stop bit, no parity (8-N-1)
- **FIFO**: Enabled

### Luminosity Calculation Algorithm

The TSL2561 lux calculation uses ratio-based compensation:

```c
ratio = CH1 / CH0;

if (ratio ≤ K1) lux = (CH0 × B1) - (CH1 × M1)
else if (ratio ≤ K2) lux = (CH0 × B2) - (CH1 × M2)
...
// 8 different constant sets for different ratio ranges
// Compensates for integration time and gain settings
```

**Typical Values:**
- Room lighting: 15-25 lux
- Bright desk lamp: 100-200 lux
- Flashlight (close): 1000-1700 lux

### Threshold Configuration

```
Potentiometer: 0-50kΩ → ADC: 0-4095
Voltage = (ADC / 4095) × 3.3V
Resistance = (Voltage / 3.3V) × 50kΩ

midValue = Resistance × 0.034
lowThreshold = midValue - 500 lux
highThreshold = midValue + 500 lux
```

**Scaling Factor (0.034)**: Normalizes 50kΩ resistance to ~1700 lux (max observed luminosity)

### PWM Duty Cycle Calculation

```
dutyCycle_High = luminosity × 0.0588  // Normalized to percentage
dutyCycle_Low = 100 - dutyCycle_High

Timer Period: HIGH duration
Next Period: LOW duration
```

**Example**: If luminosity = 1000 lux → 58.8% duty cycle

---

## Project Structure

```
Light-Based-LED-Driver/
├── README.md                       # This file
├── .gitignore                      # Git ignore rules
│
├── src/                            # Main application code
│   └── onboard.c                   # Main program (I2C, ADC, GPIO, Timer init & control loop)
│
├── include/                        # Header files
│   └── CalculateLux.h              # TSL2561 lux calculation algorithm
│
├── drivers/                        # Peripheral drivers (Assembly)
│   ├── lcd/                        # LCD display driver (SPI)
│   │   ├── updateFunctions.s       # SPI0 init, LCD commands, display update
│   │   ├── AutoText.s              # Predefined character rendering (L, U, M, T, H)
│   │   ├── CNVRT.s                 # Decimal-to-digit conversion
│   │   └── writeDigit.s            # Digit bitmap patterns (0-9)
│   │
│   └── uart/                       # UART driver
│       └── OutStr.s                # Serial string transmission
│
├── device/                         # Device-specific files
│   ├── startup_TM4C123.s           # Startup code & vector table
│   └── system_TM4C123.c            # System initialization
│
├── docs/                           # Documentation
│   ├── hardware/                   # Hardware documentation
│   │   └── TSL2561-Datasheet.pdf   # Light sensor datasheet
│   ├── implementation/             # Implementation details
│   │   └── project-report.pdf      # Detailed project report
│   └── images/                     # Images for documentation
│
└── build/                          # Build output directory (not tracked)
```

### File Descriptions

#### Source Files

**[src/onboard.c](src/onboard.c)**
- Main application entry point
- Peripheral initialization (GPIO, I2C3, ADC0, SPI0, Timer0A, UART0)
- Main control loop:
  - Reads TSL2561 sensor 256 times per cycle
  - Calculates average luminosity
  - Reads potentiometer resistance
  - Updates threshold values
  - Controls RGB LED based on thresholds
  - Updates LCD display
  - Generates PWM signal

**[include/CalculateLux.h](include/CalculateLux.h)**
- TSL2561 proprietary lux calculation function
- Implements ratio-based compensation algorithm from datasheet
- Uses 8 different constant sets (K, B, M values)
- Compensates for integration time and gain settings

#### Driver Files (Assembly)

**[drivers/lcd/updateFunctions.s](drivers/lcd/updateFunctions.s)**
- `SPI0_init`: Initializes SSI0 module, configures LCD contrast/bias
- `SPI1_Write_cmd`: Sends commands to LCD (DC=0)
- `SPI1_Write_data`: Sends data to LCD (DC=1)
- `updateLCD`: Main display refresh function

**[drivers/lcd/AutoText.s](drivers/lcd/AutoText.s)**
- Renders predefined characters: L, U, M, T, H
- Uses bitmap patterns for each letter
- Horizontal addressing mode

**[drivers/lcd/CNVRT.s](drivers/lcd/CNVRT.s)**
- Converts decimal numbers to individual digits
- Maps values to LCD memory positions
- Displays up to 3 digits per value

**[drivers/lcd/writeDigit.s](drivers/lcd/writeDigit.s)**
- Bitmap patterns for digits 0-9
- 5×8 pixel character font
- Hexadecimal pattern definitions

**[drivers/uart/OutStr.s](drivers/uart/OutStr.s)**
- UART0 initialization (9600 baud)
- String transmission function
- Serial debugging output

#### Device Files

**[device/startup_TM4C123.s](device/startup_TM4C123.s)**
- ARM Cortex-M4 startup code
- Vector table with interrupt handlers
- Stack initialization
- Calls main() function

**[device/system_TM4C123.c](device/system_TM4C123.c)**
- System clock configuration
- PLL setup (optional)
- Early hardware initialization

---

## Getting Started

### Prerequisites

**Hardware:**
- TI Tiva C Series LaunchPad (EK-TM4C123GXL)
- TSL2561 light sensor breakout board
- Nokia 5110 LCD display
- 50kΩ linear potentiometer
- External LED with NPN transistor driver circuit
- Breadboard and jumper wires

**Software:**
- Keil μVision IDE (or equivalent ARM compiler)
- TI TivaWare library
- Serial terminal (e.g., PuTTY, Tera Term)

### Hardware Setup

1. **Connect TSL2561 Light Sensor (I2C3):**
   - VCC → 3.3V
   - GND → GND
   - SCL → PD0
   - SDA → PD1

2. **Connect Nokia 5110 LCD (SPI):**
   - VCC → 3.3V
   - GND → GND
   - CLK → PA2
   - Din → PA5
   - DC → PA7
   - CE → PA3
   - RST → PA6
   - LIGHT → GND (backlight always on)

3. **Connect Potentiometer (ADC0):**
   - Terminal 1 → 3.3V
   - Wiper → PE3
   - Terminal 2 → GND

4. **External LED Circuit:**
   - PB3 → Base of NPN transistor (through 1kΩ resistor)
   - Collector → LED anode
   - Emitter → GND
   - LED cathode → 3.3V (through appropriate resistor)

5. **USB Connection:**
   - Connect LaunchPad to PC via USB (powers board and enables UART)

### Software Build

1. **Clone/Download Project:**
   ```bash
   git clone <repository-url>
   cd Light-Based-LED-Driver
   ```

2. **Open in Keil μVision:**
   - Create new project for TM4C123GH6PM
   - Add all `.c` files from `src/` and `device/`
   - Add all `.s` files from `drivers/` and `device/`
   - Include `include/` directory in compiler paths

3. **Build Configuration:**
   - Target: ARM Cortex-M4
   - Compiler: ARM Compiler v5 or v6
   - Optimization: -O1 or higher
   - Include TivaWare library paths if using CMSIS

4. **Compile:**
   ```
   Build → Build Target (F7)
   ```

5. **Flash to Board:**
   ```
   Flash → Download (F8)
   ```

### Running the System

1. **Power On:**
   - Connect LaunchPad via USB
   - Observe on-board LED lights up (initial state)

2. **Monitor Serial Output:**
   - Open serial terminal (9600 baud, 8-N-1)
   - Connect to appropriate COM port
   - View potentiometer resistance values

3. **Test Functionality:**
   - **Low Light**: Cover TSL2561 sensor → Red LED lights
   - **Normal Light**: Normal room lighting → Green LED lights
   - **High Light**: Shine flashlight at sensor → Blue LED lights
   - Observe LCD displaying LUM, LT, HT values
   - Adjust potentiometer to change thresholds
   - Watch external LED brightness change with ambient light

---

## Functionality

### Main Control Loop

The system operates in a continuous loop with the following sequence:

```c
while(1) {
    // 1. Sensor Reading (256 samples)
    for (i = 0; i < 256; i++) {
        I2C3_read_Multiple(SLAVE_ADDRESS, 0x8C, 2, CH0_data);  // Channel 0
        I2C3_read_Multiple(SLAVE_ADDRESS, 0x8E, 2, CH1_data);  // Channel 1
        CH0_sum += combine_bytes(CH0_data);
        CH1_sum += combine_bytes(CH1_data);
    }

    // 2. Calculate Average
    CH0_avg = CH0_sum / 256;
    CH1_avg = CH1_sum / 256;

    // 3. Convert to Lux
    luminosity = CalculateLux(1u, 2u, CH0_avg, CH1_avg, 1);

    // 4. Read Potentiometer
    potResistance = getPotResistance();

    // 5. Calculate Thresholds
    midValue = potResistance * 0.034;
    lowThreshold = midValue - 500;
    highThreshold = midValue + 500;

    // 6. Control RGB LED
    if (luminosity < lowThreshold) {
        // Red LED ON, others OFF
    } else if (luminosity > highThreshold) {
        // Blue LED ON, others OFF
    } else {
        // Green LED ON, others OFF
    }

    // 7. Update LCD Display
    updateLCD();  // Shows LUM, LT, HT values

    // 8. PWM Duty Cycle (handled by Timer0A ISR)
    dutyCycle = luminosity * 0.0588;
}
```

### LCD Display Layout

```
┌─────────────────────────┐
│ LUM: 0850              │  Row 0: Luminosity value (lux)
│                         │
│ LT:  0350              │  Row 2: Low threshold (lux)
│                         │
│ HT:  1350              │  Row 4: High threshold (lux)
│                         │
└─────────────────────────┘
  84×48 pixels, 6 rows
```

### Interrupt Service Routine (Timer0A)

```assembly
Timer0A_Handler:
    ; Clear interrupt flag
    ; Check current state
    ; If HIGH period just ended:
        ; Load LOW period duration
        ; Set PB3 = 0
    ; If LOW period just ended:
        ; Load HIGH period duration
        ; Set PB3 = 1
    ; Return from interrupt
```

---

## Performance Metrics

### Measured Performance

| Metric | Value |
|--------|-------|
| **Sensor Sampling Rate** | ~256 samples/cycle |
| **Luminosity Range** | 15 - 1700+ lux |
| **Threshold Adjustment Range** | 0 - 1700 lux (±500 window) |
| **PWM Frequency** | ~1 kHz |
| **PWM Resolution** | 8-bit equivalent (0-100%) |
| **LCD Update Rate** | Continuous (limited by sensor read time) |
| **ADC Conversion Time** | <10 μs (125 kSPS) |
| **I2C Transaction Time** | ~10 ms per read (100 kHz) |
| **System Response Time** | <3 seconds (256-sample averaging) |

### Accuracy

- **Sensor Accuracy**: ±15% (per TSL2561 datasheet)
- **ADC Accuracy**: ±1 LSB (12-bit)
- **PWM Linearity**: >95% (tested with oscilloscope)

---

## Demo & Testing

### Test Cases

1. **Low Light Detection:**
   - Cover sensor completely
   - Expected: Red LED ON, luminosity <20 lux, PWM duty cycle <5%

2. **Normal Room Lighting:**
   - Typical indoor lighting
   - Expected: Green LED ON, luminosity 15-100 lux, PWM 5-30%

3. **High Light Detection:**
   - Shine flashlight directly at sensor
   - Expected: Blue LED ON, luminosity >1000 lux, PWM >60%

4. **Threshold Adjustment:**
   - Rotate potentiometer fully CCW
   - Expected: LT ≈ 0, HT ≈ 500, narrow window
   - Rotate potentiometer fully CW
   - Expected: LT ≈ 1200, HT ≈ 1700, high threshold

5. **PWM Verification:**
   - Connect oscilloscope to PB3
   - Vary ambient light
   - Expected: Duty cycle proportional to luminosity

---

## Troubleshooting

### Common Issues

**LCD Display Not Working:**
- Check SPI connections (PA2, PA3, PA5, PA6, PA7)
- Verify 3.3V power supply
- Ensure contrast setting in `SPI0_init` is correct (Vop register)
- Check DC and CE signals with logic analyzer

**Sensor Reading Always Zero:**
- Verify I2C connections (PD0, PD1)
- Check TSL2561 power-on command (0x03 to register 0x80)
- Confirm slave address 0x39
- Test I2C bus with logic analyzer/oscilloscope

**RGB LED Not Changing:**
- Verify threshold values are reasonable (check LCD)
- Ensure GPIO Port F is initialized correctly
- Check if luminosity is within expected range
- Adjust potentiometer to change threshold window

**PWM Not Working:**
- Verify Timer0A initialization
- Check PB3 GPIO configuration (alternate function)
- Confirm ISR is being called (add breakpoint)
- Verify duty cycle calculation isn't resulting in 0% or 100%

**Serial Output Not Visible:**
- Confirm UART baud rate (9600)
- Check PA0/PA1 connections if using external USB-UART adapter
- On LaunchPad, USB connection should work automatically
- Verify correct COM port in terminal software

---

## Future Enhancements

Potential improvements for this project:

- **Data Logging**: SD card storage for long-term light level monitoring
- **Wireless Communication**: Bluetooth/WiFi for remote monitoring and control
- **Mobile App**: Android/iOS app for threshold configuration
- **Multi-Zone Control**: Multiple sensors controlling different LED zones
- **Advanced Algorithms**: Machine learning for adaptive threshold adjustment
- **Power Optimization**: Low-power sleep modes when light is stable
- **Additional Sensors**: Temperature, humidity for environmental monitoring
- **RTOS Integration**: Real-time operating system for task scheduling

---

## Author

**Embedded Software Engineer**
Specializing in ARM Cortex-M microcontrollers, peripheral driver development, and real-time embedded systems.

**Technical Skills Demonstrated:**
- ARM Cortex-M4 firmware development
- Multi-protocol communication (I2C, SPI, ADC, UART)
- Interrupt-driven programming
- PWM generation and control
- Sensor integration and calibration
- Mixed-language programming (C/Assembly)
- Real-time embedded systems
- Hardware/software co-design

---

## License

This project is available for educational and portfolio purposes.

---

## Acknowledgments

- Texas Instruments for TM4C123 microcontroller and TivaWare library
- TSL2561 datasheet for lux calculation algorithm
- Nokia 5110 LCD community documentation
- Embedded systems course instructors and lab resources

---

## Contact

For questions, suggestions, or collaboration opportunities, please reach out via:
- GitHub: [Your GitHub Profile]
- LinkedIn: [Your LinkedIn Profile]
- Email: [Your Email]

---

**Last Updated**: January 2026
**Version**: 1.0
