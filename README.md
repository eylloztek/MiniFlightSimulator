# STM32 ADXL345 Mini Flight Simulator

## 1. Project Description

This project implements a simple flight-direction indicator using:

- STM32 Nucleo-F446RE
- ADXL345 3-axis accelerometer (I2C mode)
- 4 LEDs arranged in a square layout

The system reads real-time acceleration data from the ADXL345 and turns on specific LEDs depending on tilt direction (forward, backward, left, right).

The logic mimics a simplified flight attitude indicator.

---

## 2. System Behavior

| Tilt Direction | Condition | LED Activated |
|---------------|------------|---------------|
| Forward       | Y > +50    | Green         |
| Backward      | Y < -50    | Yellow        |
| Right         | X > +50    | Blue          |
| Left          | X < -50    | Red           |

- Threshold: 50 (raw accelerometer units)
- Sampling delay: 100 ms
- X and Y are evaluated independently (diagonal tilt may activate two LEDs)

---

## 3. Hardware Components

- STM32 Nucleo-F446RE
- ADXL345 accelerometer module
- 4 × LEDs
- 4 × 220Ω resistors

---

## 4. Hardware Connections

### 4.1 ADXL345 → STM32 (I2C Mode)

| ADXL345 Pin | STM32 Pin | Description |
|-------------|-----------|------------|
| VCC | 3.3V | Power supply |
| GND | GND | Common ground |
| SDA | PB9 | I2C1_SDA |
| SCL | PB8 | I2C1_SCL |
| CS | 3.3V | Forces I2C mode |
| SDO | GND | Sets address to 0x53 |

I2C 7-bit address: `0x53`  
HAL uses left-shifted address: `(0x53 << 1)`

---

### 4.2 LED Connections

Each LED is connected in series with a 220Ω resistor.

GPIO HIGH → LED ON

| LED Color | STM32 Pin | Port | Function |
|------------|-----------|------|----------|
| Green | PA0  | GPIOA | Forward |
| Yellow | PA1 | GPIOA | Backward |
| Blue | PA4 | GPIOA | Right |
| Red | PA10 | GPIOA | Left |

---

## 5. STM32 Configuration

### 5.1 Clock Settings

| Parameter | Value |
|-----------|--------|
| Oscillator | HSI |
| PLL Source | HSI |
| SYSCLK | 84 MHz |
| AHB Divider | 1 |
| APB1 Divider | 2 |
| APB2 Divider | 1 |
| Flash Latency | 2 |

---

### 5.2 I2C1 Configuration

| Parameter | Value |
|------------|--------|
| Mode | I2C |
| Clock Speed | 100 kHz |
| Addressing Mode | 7-bit |
| Dual Address | Disabled |
| General Call | Disabled |
| No Stretch | Disabled |

---

### 5.3 GPIO Configuration

| Parameter | Value |
|-----------|--------|
| Mode | Output Push-Pull |
| Pull | No Pull |
| Speed | Low Frequency |
| Initial State | RESET |

<img width="1065" height="1032" alt="image" src="https://github.com/user-attachments/assets/40b1ca0b-cbe8-4e78-b5ff-50c7138aa1f0" />

---

## 6. Software Architecture

### 6.1 ADXL345 Initialization

Registers configured:

| Register | Address | Value | Description |
|----------|----------|--------|-------------|
| POWER_CTL | 0x2D | 0x08 | Measurement mode |
| DATA_FORMAT | 0x31 | 0x00 | ±2g range |

---

### 6.2 Data Reading

Acceleration data is read starting from register `0x32`.

| Axis | Registers |
|------|-----------|
| X | 0x32–0x33 |
| Y | 0x34–0x35 |
| Z | 0x36–0x37 |

Data conversion:

```c
X = (int16_t)(data[1] << 8 | data[0]);
Y = (int16_t)(data[3] << 8 | data[2]);
Z = (int16_t)(data[5] << 8 | data[4]);
```
---
### 6.3 Control Algorithm

- Read accelerometer data
- Turn off all LEDs
- Compare X and Y values with threshold
- Activate corresponding LED(s)
- Delay 100 ms

---

## 7. Build & Flash
### 7.1 Requirements
- STM32CubeIDE
- ST-Link
- STM32CubeProgrammer (optional)

### 7.2 Steps
1. Open project in STM32CubeIDE
2. Build
3. Flash via ST-Link
> If unexpected GPIO behavior occurs: Use STM32CubeProgrammer → Full Chip Erase (Mass Erase). Then re-flash firmware.

---
## 8. Libraries Used
* **STM32 HAL Drivers** (via STM32CubeIDE)

---

## 9. Others

This section contains a general overview of the project on a breadboard and a sample video of the project in action.
![flightsim1](https://github.com/user-attachments/assets/fca3b6c1-db6b-4251-bdfb-aba9983d89ad)

![flightsim2](https://github.com/user-attachments/assets/6a5de3aa-24f1-470c-8c33-6e958735168d)

https://github.com/user-attachments/assets/f7d0d34f-8389-48dd-b9fb-fbc31d1c702f

