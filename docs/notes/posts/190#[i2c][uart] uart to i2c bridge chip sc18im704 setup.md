---
number: 190
title: "[i2c][uart] uart to i2c bridge chip sc18im704 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/190
author: konosubakonoakua
createdAt: 2025-06-26T07:36:36Z
updatedAt: 2025-06-26T07:40:42Z
labels: []
---

# 190 [i2c][uart] uart to i2c bridge chip sc18im704 setup

## datasheet
- [SC18IM704](https://www.nxp.com/products/interfaces/ic-spi-i3c-interface-devices/bridges/uart-to-ic-bus-bridge:SC18IM704)
- [ad5252](https://www.analog.com/en/products/ad5252.html)

---

## Comments

### konosubakonoakua · 2025-06-26T07:38:45Z

## wiring
- USB2UART module connect to sc18im704 (`baudrate=9600`)
- sc18im704 module connect to i2c chip AD5252 (`addr=0x2c`)


### konosubakonoakua · 2025-06-26T07:39:11Z

## driver

<details>

```python
"""
SC18IM704 UART to I2C Bridge Controller
- Controls AD5252 digital potentiometer via I2C
- Implements full logging for all serial transactions
"""

import serial
import time
from datetime import datetime

# ========== Configuration ==========
UART_PORT = 'COM8'       # Serial port for SC18IM704
BAUDRATE = 9600          # UART baud rate (default for SC18IM704 is 9600)
I2C_ADDR_AD5252 = 0x2C   # AD5252 default I2C address (7-bit)

# SC18IM704 command set (UART protocol) - UPDATED
CMD_STOP         = 0x50  # STOP
CMD_I2C_START    = 0x53  # I2C START
CMD_READ_REG     = 0x52  # Read internal register
CMD_WRITE_REG    = 0x57  # Write internal register
CMD_READ_GPIO    = 0x49  # Read GPIO
CMD_WRITE_GPIO   = 0x4F  # Write GPIO
CMD_POWER_DOWN   = 0x5A  # Power down

# SC18IM704 register addresses
REG_BRG0         = 0x00
REG_BRG1         = 0x01
REG_GPIO_CONF1   = 0x02
REG_GPIO_CONF2   = 0x03
REG_GPIO_STATE   = 0x04
REG_I2C_ADDR     = 0x06
REG_I2C_CLK_L    = 0x07
REG_I2C_CLK_H    = 0x08
REG_I2C_TIMEOUT  = 0x09
REG_I2C_STAT     = 0x0A

# I2C status codes
I2C_STAT_OK         = 0xF0
I2C_STAT_NACK_ADDR  = 0xF1
I2C_STAT_NACK_DATA  = 0xF2
I2C_STAT_TIMEOUT    = 0xF8

# ========== Helper Functions ==========
def log_transaction(direction, data):
    """Log serial transactions with timestamp"""
    timestamp = datetime.now().strftime("%H:%M:%S.%f")[:-3]
    hex_data = ' '.join(f"{b:02X}" for b in data)
    print(f"[{timestamp}] {direction}: {hex_data}")

def write_internal_reg(ser, reg_addr, value):
    """
    Write to SC18IM704 internal register.
    Protocol: [CMD_WRITE_REG, reg_addr, value, CMD_STOP]
    """
    cmd = bytes([CMD_WRITE_REG, reg_addr, value, CMD_STOP])
    log_transaction("TX", cmd)
    ser.write(cmd)
    time.sleep(0.01)

def read_internal_reg(ser, reg_addr):
    """
    Read from SC18IM704 internal register.
    Protocol: [CMD_READ_REG, reg_addr, CMD_STOP]
    Returns: value (int) or None
    """
    cmd = bytes([CMD_READ_REG, reg_addr, CMD_STOP])
    log_transaction("TX", cmd)
    ser.write(cmd)
    value = ser.read(1)
    if value:
        log_transaction("RX", value)
        return value[0]
    else:
        print(f"Failed to read register 0x{reg_addr:02X}")
        return None

def init_sc18im704(ser, i2c_freq=100_000, osc_freq=15_000_000):
    """
    Initialize SC18IM704 bridge
    - Sets I2C speed to desired frequency (Hz)
    - osc_freq: oscillator frequency in Hz (default 15MHz)
    - i2c_freq: desired I2C frequency in Hz (default 100kHz)
    """
    clk_div = int(osc_freq / (5 * i2c_freq))
    if clk_div > 0xFFFF:
        clk_div = 0xFFFF  # Clamp to 16-bit max
    clk_div_l = clk_div & 0xFF
    clk_div_h = (clk_div >> 8) & 0xFF
    write_internal_reg(ser, REG_I2C_CLK_L, clk_div_l)
    write_internal_reg(ser, REG_I2C_CLK_H, clk_div_h)
    # No response expected for register write

def write_i2c(ser, addr, data):
    """
    I2C write operation using SC18IM704 protocol:
    [CMD_I2C_START, addr_w, num_bytes, data..., CMD_STOP]
    """
    addr_w = (addr << 1) & 0xFE  # W=0
    num_bytes = len(data)
    cmd = bytes([CMD_I2C_START, addr_w, num_bytes] + list(data) + [CMD_STOP])
    log_transaction("TX", cmd)
    ser.write(cmd)
    time.sleep(0.05)
    # Read status
    status_str = read_i2c_status(ser)
    if status_str != "I2C OK":
        print(f"I2C Write Error: {status_str}")

def read_i2c(ser, addr, length):
    """
    I2C read operation using SC18IM704 protocol:
    [CMD_I2C_START, addr_r, num_bytes, CMD_STOP]
    """
    addr_r = (addr << 1) | 0x01  # R=1
    cmd = bytes([CMD_I2C_START, addr_r, length, CMD_STOP])
    log_transaction("TX", cmd)
    ser.write(cmd)
    time.sleep(0.05)
    response = ser.read(length)
    status_str = read_i2c_status(ser)
    if status_str != "I2C OK":
        print(f"I2C Read Error: {status_str}")
        return None
    if response:
        log_transaction("RX", response)
        return response
    else:
        print("I2C Read Error: No data received")
        return None

def read_after_write_i2c(ser, addr, write_data, read_length):
    """
    I2C repeated start (read after write) using SC18IM704 protocol:
    [CMD_I2C_START, addr_w, num_bytes, data..., CMD_I2C_START, addr_r, read_length, CMD_STOP]
    """
    addr_w = (addr << 1) & 0xFE
    addr_r = (addr << 1) | 0x01
    cmd = bytes(
        [CMD_I2C_START, addr_w, len(write_data)] +
        list(write_data) +
        [CMD_I2C_START, addr_r, read_length, CMD_STOP]
    )
    log_transaction("TX", cmd)
    ser.write(cmd)
    time.sleep(0.05)
    response = ser.read(read_length)
    status_str = read_i2c_status(ser)
    if status_str != "I2C OK":
        print(f"I2C Read-after-Write Error: {status_str}")
        return None
    if response:
        log_transaction("RX", response)
        return response
    else:
        print("I2C Read-after-Write Error: No data received")
        return None

def read_sc18im704_version(ser):
    """
    Reads the SC18IM704 firmware version string.
    Sends ASCII 'V' (0x56) and 'P' (0x50), expects a 16-byte null-terminated string.
    """
    cmd = bytes([0x56, 0x50])  # 'V', 'P'
    log_transaction("TX", cmd)
    ser.write(cmd)
    version = ser.read(16)
    if version:
        log_transaction("RX", version)
        # Remove null terminator and decode
        version_str = version.rstrip(b'\x00').decode(errors='replace')
        print(f"SC18IM704 Version: {version_str}")
        return version_str
    else:
        print("Failed to read SC18IM704 version.")
        return None

def read_i2c_status(ser):
    """
    Reads the SC18IM704 I2C status register and returns a human-readable status.
    """
    code = read_internal_reg(ser, REG_I2C_STAT)
    if code is None:
        print("Failed to read I2C status register.")
        return None
    if code == I2C_STAT_OK:
        return "I2C OK"
    elif code == I2C_STAT_NACK_ADDR:
        return "I2C NACK on address"
    elif code == I2C_STAT_NACK_DATA:
        return "I2C NACK on data"
    elif code == I2C_STAT_TIMEOUT:
        return "I2C TIMEOUT"
    else:
        return f"Unknown status: 0x{code:02X}"

# ========== AD5252 Operations ==========
def write_rdac3(ser, value):
    """
    Write to AD5252's RDAC3 register (address 0x03)
    - Performs I2C write: [0x03, value]
    """
    if not 0 <= value <= 255:
        raise ValueError("RDAC value must be 0-255")
    
    print(f"\n[AD5252] Writing RDAC3: 0x{value:02X}")
    write_i2c(ser, I2C_ADDR_AD5252, [0x03, value])

def read_rdac3(ser):
    """
    Read from AD5252's RDAC3 register using repeated start
    """
    print("\n[AD5252] Reading RDAC3...")
    data = read_after_write_i2c(ser, I2C_ADDR_AD5252, [0x03], 1)
    return data[0] if data else None

def read_whoami(ser):
    raise NotImplementedError

# ========== Main Program ==========
def main():
    print("===== SC18IM704 I2C Bridge Controller =====")
    print(f"Config: Port={UART_PORT}, Baud={BAUDRATE}")
    print(f"Target: AD5252 @ 0x{I2C_ADDR_AD5252:02X}\n")
    
    try:
        with serial.Serial(UART_PORT, BAUDRATE, timeout=1) as ser:
            # Read SC18IM704 firmware version
            read_sc18im704_version(ser)
            print("\n")

            # Initialize bridge
            print("[SC18IM704] Initializing...")
            init_sc18im704(ser)
            
            # 2. Write test value to RDAC3
            test_value = 0x80
            write_rdac3(ser, test_value)
            
            # 3. Verify write operation
            rdac_val = read_rdac3(ser)
            if rdac_val is not None:
                print(f"RDAC3 Value: 0x{rdac_val:02X}")
                print("Verification:", "PASS" if rdac_val == test_value else "FAIL")
            else:
                print("Error reading RDAC3!")

            # 4. Write test value to RDAC3
            test_value = 0xde
            write_rdac3(ser, test_value)
            
            # 5. Verify write operation
            rdac_val = read_rdac3(ser)
            if rdac_val is not None:
                print(f"RDAC3 Value: 0x{rdac_val:02X}")
                print("Verification:", "PASS" if rdac_val == test_value else "FAIL")
            else:
                print("Error reading RDAC3!")

    except serial.SerialException as e:
        print(f"\nSerial port error: {e}")
    except Exception as e:
        print(f"\nUnexpected error: {e}")

if __name__ == "__main__":
    main()
```
</details>

### konosubakonoakua · 2025-06-26T07:39:39Z

## log

<details>

```log
~\Desktop\sc18im704 via 🐍 v3.12.10 (py312) 
❯ python .\sc18im704_ad5252.py
===== SC18IM704 I2C Bridge Controller =====
Config: Port=COM8, Baud=9600
Target: AD5252 @ 0x2C

[15:30:12.005] TX: 56 50
[15:30:12.027] RX: 53 43 31 38 49 4D 37 30 34 20 31 2E 30 2E 32 00
SC18IM704 Version: SC18IM704 1.0.2


[SC18IM704] Initializing...
[15:30:12.027] TX: 57 07 1E 50
[15:30:12.039] TX: 57 08 00 50

[AD5252] Writing RDAC3: 0x80
[15:30:12.049] TX: 53 58 02 03 80 50
[15:30:12.100] TX: 52 0A 50
[15:30:12.106] RX: F0

[AD5252] Reading RDAC3...
[15:30:12.106] TX: 53 58 01 03 53 59 01 50
[15:30:12.157] TX: 52 0A 50
[15:30:12.163] RX: F0
[15:30:12.163] RX: 80
RDAC3 Value: 0x80
Verification: PASS

[AD5252] Writing RDAC3: 0xDE
[15:30:12.164] TX: 53 58 02 03 DE 50
[15:30:12.215] TX: 52 0A 50
[15:30:12.221] RX: F0

[AD5252] Reading RDAC3...
[15:30:12.221] TX: 53 58 01 03 53 59 01 50
[15:30:12.272] TX: 52 0A 50
[15:30:12.279] RX: F0
[15:30:12.279] RX: DE
RDAC3 Value: 0xDE
Verification: PASS
```
</details>
