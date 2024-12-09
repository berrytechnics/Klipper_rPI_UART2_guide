# SKR Mini E3 V2 UART Setup Guide for Klipper

## Table of Contents
- [Migrating from USB to UART](#migrating-from-usb-to-uart)
- [Fresh Installation](#fresh-installation)
- [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)

## Migrating from USB to UART

### Prerequisites
- Raspberry Pi 3B already running MainsailOS
- SKR Mini E3 V2 board
- 3 jumper wires
- Working Klipper installation currently using USB

### Migration Steps

1. **Backup Your Configuration**
   ```bash
   cp ~/printer_data/config/printer.cfg ~/printer_data/config/printer.cfg.backup
   ```

2. **Stop Klipper Services**
   ```bash
   sudo service klipper stop
   sudo service moonraker stop
   ```

3. **Edit Raspberry Pi Configuration**
   ```bash
   sudo nano /boot/config.txt
   ```
   Add these lines at the end:
   ```
   enable_uart=1
   dtoverlay=disable-bt
   ```

4. **Make Physical Connections**
   - Power off both printer and Raspberry Pi
   - Connect SKR Mini E3 V2 TFT header to Pi:
     - TFT TX → Pi GPIO 15 (physical pin 10)
     - TFT RX → Pi GPIO 14 (physical pin 8)
     - GND → Any Pi GND pin

5. **Update printer.cfg**
   ```bash
   nano ~/printer_data/config/printer.cfg
   ```
   Find the [mcu] section and update:
   ```yaml
   [mcu]
   serial: /dev/ttyAMA0
   restart_method: command
   ```

6. **Reboot and Start Services**
   ```bash
   sudo reboot
   ```

## Fresh Installation

### Prerequisites
- Raspberry Pi 3B
- SKR Mini E3 V2 board
- MicroSD card (8GB or larger)
- 3 jumper wires
- Computer with SD card reader

### Installation Steps

1. **Install MainsailOS**
   - Download MainsailOS from official website
   - Flash to SD card using Raspberry Pi Imager
   - Enable SSH during flashing

2. **Initial Pi Setup**
   - Insert SD card and power on Pi
   - Connect to Pi via SSH
   - Run initial setup:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Configure UART**
   ```bash
   sudo nano /boot/config.txt
   ```
   Add:
   ```
   enable_uart=1
   dtoverlay=disable-bt
   ```

4. **Make Physical Connections**
   - Power off both devices
   - Connect SKR Mini E3 V2 TFT header to Pi:
     - TFT TX → Pi GPIO 15 (physical pin 10)
     - TFT RX → Pi GPIO 14 (physical pin 8)
     - GND → Any Pi GND pin

5. **Configure Klipper**
   - Create printer.cfg:
   ```bash
   nano ~/printer_data/config/printer.cfg
   ```
   - Add basic configuration:
   ```yaml
   [mcu]
   serial: /dev/ttyAMA0
   restart_method: command

   [printer]
   kinematics: cartesian
   max_velocity: 300
   max_accel: 3000
   max_z_velocity: 5
   max_z_accel: 100
   ```

6. **Flash Klipper to SKR Mini E3 V2**
   ```bash
   cd ~/klipper
   make menuconfig
   ```
   Settings:
   - Micro-controller: STM32F103
   - Boot Flash: 28KiB
   - Communication interface: Serial (UART)
   
   Then:
   ```bash
   make
   ```
   - Copy firmware to SD card
   - Insert SD into SKR board and power on

7. **Start Services**
   ```bash
   sudo service klipper start
   sudo service moonraker start
   ```

## Common Issues and Troubleshooting

### No Connection to MCU
1. Check physical connections
2. Verify UART is enabled:
   ```bash
   ls -l /dev/ttyAMA0
   ```
3. Check permissions:
   ```bash
   sudo usermod -a -G tty pi
   sudo usermod -a -G dialout pi
   ```

### Communication Errors
1. Verify TX/RX aren't swapped
2. Check ground connection
3. Verify baud rate matches in printer.cfg

### Bluetooth Issues
If you need Bluetooth, use alternate UART:
1. In `/boot/config.txt`:
   ```
   dtoverlay=uart2
   ```
2. In `printer.cfg`:
   ```yaml
   [mcu]
   serial: /dev/ttyAMA1
   ```
