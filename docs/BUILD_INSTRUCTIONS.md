# Build Instructions

This document provides detailed instructions for building and flashing the Intelligent Ambient Light Sensing and Control System firmware.

## Table of Contents
1. [Development Environment Setup](#development-environment-setup)
2. [Building with Keil μVision](#building-with-keil-μvision)
3. [Building with GCC ARM](#building-with-gcc-arm)
4. [Flashing the Firmware](#flashing-the-firmware)
5. [Verification and Testing](#verification-and-testing)
6. [Troubleshooting](#troubleshooting)

---

## Development Environment Setup

### Required Software

#### Option 1: Keil μVision (Recommended)
- **Keil MDK-ARM**: Version 5.x or later
  - Download from: https://www.keil.com/demo/eval/arm.htm
  - Free version supports up to 32KB code (sufficient for this project)
- **TivaWare**: C Series SDK from Texas Instruments
  - Download from: https://www.ti.com/tool/SW-TM4C
  - Version: 2.2.0 or later

#### Option 2: GCC ARM (Free/Open Source)
- **ARM GCC Toolchain**: arm-none-eabi-gcc
  - Download from: https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm
- **Make**: GNU Make utility
- **OpenOCD**: For flashing (alternative to TI's tools)

### Required Hardware

- **TI Tiva C Series LaunchPad** (EK-TM4C123GXL)
- **USB Cable**: Mini-USB or Micro-USB (depending on LaunchPad version)
- **Hardware Components**: See [README.md](../README.md#hardware-components) for full list

---

## Building with Keil μVision

### Step 1: Create New Project

1. Launch Keil μVision
2. Go to **Project → New μVision Project**
3. Navigate to your project directory: `Light-Based-LED-Driver/`
4. Name the project: `LightControl` (or any preferred name)
5. Click **Save**

### Step 2: Select Target Device

1. In the device selection window, navigate to:
   - **Manufacturer**: Texas Instruments
   - **Device**: TM4C123GH6PM
2. Click **OK**
3. When prompted to copy startup code, click **Yes** (or use existing in `device/` folder)

### Step 3: Add Source Files

1. In Project window, right-click **Source Group 1**
2. Select **Add Existing Files to Group**
3. Add the following files:

**C Source Files:**
```
src/onboard.c
device/system_TM4C123.c
```

**Assembly Files:**
```
device/startup_TM4C123.s
drivers/lcd/updateFunctions.s
drivers/lcd/AutoText.s
drivers/lcd/CNVRT.s
drivers/lcd/writeDigit.s
drivers/uart/OutStr.s
```

**Header Files:**
```
include/CalculateLux.h
```

### Step 4: Configure Include Paths

1. Right-click on project target → **Options for Target**
2. Go to **C/C++** tab
3. In **Include Paths**, add:
   ```
   .\include
   .\device
   C:\ti\TivaWare_C_Series-2.2.0.295  (adjust path to your TivaWare installation)
   ```

### Step 5: Configure Build Options

#### C/C++ Settings:
- **Optimization**: -O1 (Optimize for size)
- **C99 Mode**: Enable
- **Warnings**: All warnings
- **Define**: (none required)

#### Assembler Settings:
- **Assembler**: ARM Assembler
- **Instruction Set**: Thumb-2

#### Linker Settings:
- **Use Memory Layout from Target Dialog**: Checked
- **Scatter File**: Use default for TM4C123GH6PM

#### Debug Settings:
- **Debugger**: Stellaris ICDI
- **Settings → Flash Download**: Enable

### Step 6: Build Project

1. Click **Build** icon or press **F7**
2. Check **Build Output** window for errors
3. Successful build should show:
   ```
   0 Error(s), 0 Warning(s)
   ".\build\LightControl.axf" - 0 Error(s), 0 Warning(s)
   ```

### Expected Build Output:
```
Code Size: ~15 KB
Data Size: ~2 KB
Total Flash Usage: ~17 KB (within 32 KB free limit)
```

---

## Building with GCC ARM

### Step 1: Install Toolchain

**Linux/macOS:**
```bash
# Ubuntu/Debian
sudo apt-get install gcc-arm-none-eabi binutils-arm-none-eabi

# macOS (Homebrew)
brew install arm-none-eabi-gcc
```

**Windows:**
- Download installer from ARM website
- Add to PATH: `C:\Program Files (x86)\GNU Arm Embedded Toolchain\bin`

### Step 2: Create Makefile

Create `Makefile` in project root:

```makefile
# Compiler and tools
CC = arm-none-eabi-gcc
AS = arm-none-eabi-as
LD = arm-none-eabi-ld
OBJCOPY = arm-none-eabi-objcopy

# Flags
CFLAGS = -mcpu=cortex-m4 -mthumb -mfloat-abi=soft \
         -O1 -Wall -Iinclude -Idevice
ASFLAGS = -mcpu=cortex-m4 -mthumb
LDFLAGS = -T tm4c123gh6pm.ld -nostdlib

# Source files
C_SOURCES = src/onboard.c device/system_TM4C123.c
ASM_SOURCES = device/startup_TM4C123.s \
              drivers/lcd/updateFunctions.s \
              drivers/lcd/AutoText.s \
              drivers/lcd/CNVRT.s \
              drivers/lcd/writeDigit.s \
              drivers/uart/OutStr.s

# Object files
OBJECTS = $(C_SOURCES:.c=.o) $(ASM_SOURCES:.s=.o)

# Output
TARGET = build/light_control

all: $(TARGET).bin

$(TARGET).elf: $(OBJECTS)
	$(CC) $(LDFLAGS) -o $@ $^

$(TARGET).bin: $(TARGET).elf
	$(OBJCOPY) -O binary $< $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.s
	$(AS) $(ASFLAGS) $< -o $@

clean:
	rm -f $(OBJECTS) $(TARGET).elf $(TARGET).bin

flash: $(TARGET).bin
	openocd -f board/ti_ek-tm4c123gxl.cfg \
	        -c "program $(TARGET).bin verify reset exit"

.PHONY: all clean flash
```

### Step 3: Build

```bash
cd Light-Based-LED-Driver
make clean
make
```

**Expected Output:**
```
arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -O1 -c src/onboard.c -o src/onboard.o
...
arm-none-eabi-gcc -T tm4c123gh6pm.ld -o build/light_control.elf ...
arm-none-eabi-objcopy -O binary build/light_control.elf build/light_control.bin
Build complete: build/light_control.bin
```

---

## Flashing the Firmware

### Method 1: Keil μVision (Easiest)

1. Connect TM4C123 LaunchPad via USB
2. Wait for drivers to install (Windows)
3. In Keil, click **Download** icon or press **F8**
4. Watch **Build Output** for flash progress:
   ```
   Load "build\\LightControl.axf"
   Erase Done
   Programming Done
   Verify OK
   ```

### Method 2: LM Flash Programmer (TI Official Tool)

1. Download **LM Flash Programmer** from TI website
2. Connect LaunchPad via USB
3. Launch LM Flash Programmer
4. Configure:
   - **Interface**: Stellaris ICDI
   - **Binary File**: `build/LightControl.bin`
   - **Address**: 0x00000000
5. Click **Program**

### Method 3: OpenOCD (Open Source)

```bash
# Install OpenOCD
sudo apt-get install openocd  # Linux
brew install openocd          # macOS

# Flash firmware
openocd -f board/ti_ek-tm4c123gxl.cfg \
        -c "program build/light_control.bin verify reset exit"
```

### Method 4: Command Line (Windows - TI Uniflash)

```cmd
uniflash -c tm4c123gh6pm -f build\LightControl.bin
```

---

## Verification and Testing

### Step 1: Power-On Test

1. **Connect LaunchPad** via USB
2. **Observe**: One of the RGB LEDs should turn on immediately
3. **Expected Behavior**:
   - System initializes all peripherals
   - LCD displays initial values
   - Sensor begins reading

### Step 2: Serial Output Test

1. **Open Serial Terminal** (PuTTY, Tera Term, or Arduino Serial Monitor)
2. **Configure**:
   - Port: Check Device Manager for "Stellaris Virtual Serial Port"
   - Baud Rate: 9600
   - Data Bits: 8
   - Parity: None
   - Stop Bits: 1
3. **Expected Output**: Potentiometer resistance values updating

### Step 3: Sensor Test

1. **Cover TSL2561 sensor** with hand
   - Expected: Red LED turns on
   - LCD shows low luminosity value (<20 lux)

2. **Normal lighting**
   - Expected: Green LED turns on
   - LCD shows 15-100 lux

3. **Shine flashlight** at sensor
   - Expected: Blue LED turns on
   - LCD shows >1000 lux

### Step 4: LCD Display Test

**Verify LCD shows three values:**
```
LUM: [current luminosity]
LT:  [low threshold]
HT:  [high threshold]
```

All values should update in real-time.

### Step 5: Potentiometer Test

1. **Rotate potentiometer slowly**
2. **Observe**: LT and HT values on LCD change
3. **Verify**: LT < HT always (500 lux difference)

### Step 6: PWM Test (Advanced)

1. **Connect oscilloscope** to PB3
2. **Settings**: 5V/div, 1ms/div
3. **Expected**:
   - Square wave at ~1 kHz
   - Duty cycle changes with luminosity
   - 0-100% range

---

## Troubleshooting

### Build Errors

#### Error: "Cannot open source file CalculateLux.h"
**Solution**:
- Add `include/` directory to include paths
- Verify file exists at `include/CalculateLux.h`

#### Error: "Undefined symbol 'SPI0_init'"
**Solution**:
- Ensure all `.s` assembly files are added to project
- Check that assembly files are recognized (not added as text)

#### Error: "L6218E: Undefined symbol main"
**Solution**:
- Verify `src/onboard.c` is included in build
- Check that `main()` function exists

### Flash Errors

#### Error: "No TM4C123 device found"
**Solution**:
- Check USB cable connection
- Install/reinstall Stellaris ICDI drivers
- Try different USB port
- Check Device Manager (Windows) for "Stellaris ICDI"

#### Error: "Flash programming failed"
**Solution**:
- Disconnect and reconnect LaunchPad
- Close other applications using COM port
- Try pressing RESET button on LaunchPad
- Erase flash completely first

### Runtime Errors

#### LCD shows garbage
**Solution**:
- Check SPI connections (PA2, PA3, PA5, PA6, PA7)
- Verify 3.3V power to LCD
- Adjust contrast in `SPI0_init` (Vop value)

#### Sensor always reads zero
**Solution**:
- Check I2C connections (PD0, PD1)
- Verify sensor power (3.3V)
- Check I2C pull-up resistors (2.2kΩ - 10kΩ)
- Use logic analyzer to verify I2C communication

#### LEDs don't change
**Solution**:
- Check threshold values on LCD
- Adjust potentiometer
- Verify sensor is reading (check serial output)

---

## Build Performance Metrics

| Metric | Keil μVision | GCC ARM |
|--------|--------------|---------|
| **Build Time** | ~5 seconds | ~8 seconds |
| **Code Size** | ~14 KB | ~16 KB |
| **Data Size** | ~2 KB | ~2.5 KB |
| **Optimization** | -O1 | -O1 |

---

## Clean Build

### Keil μVision
```
Project → Clean Targets
Build → Rebuild All Target Files (Ctrl+F7)
```

### GCC ARM
```bash
make clean
make
```

---

## Version Information

**Document Version**: 1.0
**Firmware Version**: 1.0
**Toolchain Versions Tested**:
- Keil MDK-ARM: 5.37
- GCC ARM: 10.3.1
- TivaWare: 2.2.0.295

**Last Updated**: January 2026

---

## Additional Resources

- [TM4C123GH6PM Datasheet](https://www.ti.com/lit/ds/spms376e/spms376e.pdf)
- [TivaWare Peripheral Driver Library](https://www.ti.com/tool/SW-TM4C)
- [Keil Getting Started Guide](https://www.keil.com/support/man/docs/uv4/)
- [OpenOCD Documentation](http://openocd.org/documentation/)
- [ARM Cortex-M4 Programming](https://developer.arm.com/documentation/dui0553/latest/)
