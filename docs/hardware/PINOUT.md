# TM4C123GH6PM Pin Configuration

## Complete Pin Assignment Table

| Pin | Function | Peripheral | Direction | Description |
|-----|----------|------------|-----------|-------------|
| **PA0** | UART RX | UART0 | Input | Serial receive |
| **PA1** | UART TX | UART0 | Output | Serial transmit |
| **PA2** | SSI0 CLK | SSI0 | Output | SPI clock (4 MHz) |
| **PA3** | GPIO | - | Output | LCD Chip Enable (CE) |
| **PA5** | SSI0 TX | SSI0 | Output | SPI MOSI/Din |
| **PA6** | GPIO | - | Output | LCD Reset |
| **PA7** | GPIO | - | Output | LCD Data/Command (DC) |
| **PB3** | Timer PWM | Timer0A | Output | External LED PWM control |
| **PB4** | GPIO | - | Input | Push button (with pull-up) |
| **PB5** | GPIO | - | Input | Push button (with pull-up) |
| **PB6** | GPIO | - | Input | Push button (with pull-up) |
| **PB7** | GPIO | - | Input | Push button (with pull-up) |
| **PD0** | I2C SCL | I2C3 | Bidir | I2C clock line |
| **PD1** | I2C SDA | I2C3 | Bidir | I2C data line |
| **PE3** | ADC | ADC0 | Analog | Potentiometer input |
| **PF1** | GPIO | - | Output | On-board Red LED |
| **PF2** | GPIO | - | Output | On-board Blue LED |
| **PF3** | GPIO | - | Output | On-board Green LED |

## Port Configuration Details

### Port A (GPIO + SSI0 + UART0)
- **PA0-PA1**: UART0 (alternate function)
  - Configure AFSEL, enable digital, set PCTL for UART
- **PA2, PA5**: SSI0 (alternate function)
  - Configure AFSEL, enable digital, set PCTL for SSI
- **PA3, PA6, PA7**: GPIO (LCD control signals)
  - Configure as outputs, enable digital

### Port B (GPIO + Timer)
- **PB3**: Timer0A (PWM output)
  - Configure as output, enable alternate function for timer
- **PB4-PB7**: GPIO (buttons - optional)
  - Configure as inputs with pull-up resistors

### Port D (I2C3)
- **PD0-PD1**: I2C3 (alternate function)
  - Configure AFSEL, enable digital, set PCTL for I2C
  - Enable open-drain for I2C operation

### Port E (ADC)
- **PE3**: ADC0 Channel 0
  - Disable digital function
  - Enable analog mode
  - Configure ADC sample sequencer

### Port F (On-board RGB LED)
- **PF1**: Red LED (active high)
- **PF2**: Blue LED (active high)
- **PF3**: Green LED (active high)
- Configure as outputs, enable digital

## Peripheral Module Assignments

### I2C3 Module
- **Base Address**: 0x40023000
- **Clock Gate**: SYSCTL_RCGCI2C_R3
- **Pins**: PD0 (SCL), PD1 (SDA)
- **Speed**: 100 kHz standard mode
- **Devices**: TSL2561 (address 0x39)

### SSI0 Module (SPI)
- **Base Address**: 0x40008000
- **Clock Gate**: SYSCTL_RCGCSSI_R0
- **Pins**: PA2 (CLK), PA5 (TX/MOSI)
- **Speed**: 4 MHz
- **Devices**: Nokia 5110 LCD (PCD8544)

### ADC0 Module
- **Base Address**: 0x40038000
- **Clock Gate**: SYSCTL_RCGCADC_R0
- **Sample Sequencer**: SS3
- **Input**: PE3 (AIN0)
- **Resolution**: 12-bit
- **Reference**: 3.3V

### Timer0 Module
- **Base Address**: 0x40030000
- **Clock Gate**: SYSCTL_RCGCTIMER_R0
- **Mode**: Timer0A in periodic countdown
- **Output**: PB3 (CCP0)
- **Function**: PWM generation via ISR

### UART0 Module
- **Base Address**: 0x4000C000
- **Clock Gate**: SYSCTL_RCGCUART_R0
- **Pins**: PA0 (RX), PA1 (TX)
- **Baud**: 9600 bps
- **Format**: 8-N-1

## External Connections

### TSL2561 Light Sensor
```
TM4C123        TSL2561
---------      --------
3.3V      -->  VCC
GND       -->  GND
PD0 (SCL) <--> SCL (with pull-up)
PD1 (SDA) <--> SDA (with pull-up)
```

**I2C Pull-ups**: 2.2kΩ - 10kΩ to 3.3V (often built into breakout board)

### Nokia 5110 LCD
```
TM4C123        LCD
---------      -----
3.3V      -->  VCC
GND       -->  GND
PA2       -->  CLK
PA5       -->  Din
PA7       -->  DC
PA3       -->  CE
PA6       -->  RST
GND       -->  LIGHT (backlight)
```

### Potentiometer (50kΩ)
```
TM4C123        Potentiometer
---------      --------------
3.3V      -->  Terminal 1
PE3       <--  Wiper
GND       -->  Terminal 2
```

### External LED Circuit
```
TM4C123        Circuit
---------      ---------
PB3       -->  1kΩ --> NPN Base
                       NPN Collector --> LED Anode
                       NPN Emitter --> GND
3.3V      -->  Resistor --> LED Cathode
```

**Recommended NPN**: 2N3904, 2N2222, or equivalent
**LED Current Limiting Resistor**: Calculate based on LED forward voltage

## Register Configuration Summary

### GPIO Initialization Pattern
```c
// 1. Enable clock
SYSCTL_RCGCGPIOx_R |= bit_mask;

// 2. Wait for clock
while ((SYSCTL_PRGPIOx_R & bit_mask) == 0);

// 3. Configure direction
GPIOx_DIR_R |= output_pins;  // 1 = output, 0 = input

// 4. Enable digital (if not analog)
GPIOx_DEN_R |= digital_pins;

// 5. For analog (ADC)
GPIOx_AMSEL_R |= analog_pins;

// 6. For alternate function
GPIOx_AFSEL_R |= alt_function_pins;
GPIOx_PCTL_R = (GPIOx_PCTL_R & mask) | pctl_value;

// 7. For pull-up/pull-down
GPIOx_PUR_R |= pullup_pins;    // Pull-up
GPIOx_PDR_R |= pulldown_pins;  // Pull-down
```

## Power and Ground Distribution

**Critical**: All peripherals must share common ground with TM4C123

- **System Power**: 3.3V from LaunchPad regulator
- **Maximum Current per Pin**: 2 mA (standard), 8 mA (high drive - not used)
- **Total GPIO Current**: Do not exceed 100 mA total

**Note**: External LED uses transistor driver to avoid exceeding pin current limits.

## Debugging Tips

1. **I2C Issues**: Use logic analyzer on PD0/PD1 to verify SCL/SDA signals
2. **SPI Issues**: Check PA2 clock with oscilloscope, verify chip select timing
3. **ADC Issues**: Measure PE3 voltage directly with multimeter
4. **PWM Issues**: Oscilloscope on PB3 to verify frequency and duty cycle
5. **UART Issues**: Loopback test (connect PA0 to PA1 temporarily)

## Reference Voltage Levels

- **Logic High (VOH)**: 2.4V minimum
- **Logic Low (VOL)**: 0.4V maximum
- **Input High (VIH)**: 2.0V minimum
- **Input Low (VIL)**: 0.8V maximum
- **ADC Reference**: 3.3V ±5%

---

**Document Version**: 1.0
**Last Updated**: January 2026
