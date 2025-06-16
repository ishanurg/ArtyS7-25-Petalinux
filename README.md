# ArtyS7-25-Petalinux
**A comprehensive, step-by-step guide to building and booting a custom Linux system on the Digilent Arty S7-25 board using only JTAG**

---

## Table of Contents

- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Vivado Project Setup](#vivado-project-setup)
  - [Downloading and Using the .tcl Script](#downloading-and-using-the-tcl-script)
  - [Generating the Bitstream](#generating-the-bitstream)
  - [Exporting Hardware for Petalinux](#exporting-hardware-for-petalinux)
- [Petalinux Project Setup](#petalinux-project-setup)
  - [Device Tree Customization](#device-tree-customization)
  - [Building the Petalinux System](#building-the-petalinux-system)
- [JTAG Boot: No SD Card, No Flash](#jtag-boot-no-sd-card-no-flash)
  - [Setting Up JTAG Boot Mode](#setting-up-jtag-boot-mode)
  - [Booting with JTAG](#booting-with-jtag)
- [Monitoring the Console](#monitoring-the-console)
- [Troubleshooting](#troubleshooting)
- [Project Commands Cheatsheet](#project-commands-cheatsheet)
- [License](#license)
- [Acknowledgments](#acknowledgments)


---

## Introduction

Welcome to the **ArtyS7-25-Petalinux** project!  
This guide is designed for users who want to build and boot a custom Linux system on the Digilent Arty S7-25 board **using only JTAG**.  
No SD card, no QSPI flash—just connect your board to your host computer via JTAG and UART, and boot directly into Linux.

---

## Project Overview

- **Target Board:** Digilent Arty S7-25
- **Tools:** Vivado 2024.2, Petalinux 2024.2
- **Boot Method:** JTAG only (no SD card, no QSPI flash)
- **Purpose:** Build and boot a custom Linux system for FPGA development and testing

---

## Prerequisites

- **Vivado 2024.2** (or newer) installed
- **Petalinux 2024.2** (or newer) installed
- **Digilent Arty S7-25 board**
- **Linux host machine** (Ubuntu 22.04 recommended)
- **JTAG programmer** (e.g., Digilent USB-JTAG cable)
- **USB cable** for UART communication
- **Terminal emulator** (e.g., `putty`, `minicom`)

---

## Getting Started

### Clone the Repository

git clone https://github.com/ishanurg/ArtyS7-25-Petalinux.git
cd ArtyS7-25-Petalinux



---

### Install Dependencies

On Ubuntu, install the following packages:

sudo apt update
sudo apt install -y build-essential git python3 python3-pip device-tree-compiler



---

### Set Up Vivado and Petalinux Environment

Source the Vivado and Petalinux environment scripts in your terminal:

source /tools/Xilinx/Vivado/2024.2/settings64.sh
source /tools/Xilinx/Petalinux/2024.2/settings.sh



*(Adjust paths according to your installation.)*

---

## Vivado Project Setup

### Downloading and Using the .tcl Script

You can use the provided `.tcl` script to recreate the Vivado project on any machine.  
**Download or clone the repository to get the script:**

git clone https://github.com/ishanurg/ArtyS7-25-Petalinux.git
cd ArtyS7-25-Petalinux



**To generate the Vivado project, run:**

vivado -mode batch -source scripts/s7-linux.tcl



This will create the Vivado project in your current directory.

---

### Opening the Project in Vivado

vivado s7-linux.xpr


![Screenshot from 2025-06-14 22-24-13](https://github.com/user-attachments/assets/54cd9ab1-dec2-4652-8b6f-84afd28282bf)


---

### Exploring the Project Structure

- **HDL sources:** `src/design/`
- **Block designs:** `src/bd/`
- **IP cores:** `src/ip/`
- **Constraints:** `constraints/`

---

### Modifying the Project

- **Add or edit HDL sources** in `src/design/`
- **Add or edit block designs** in `src/bd/`
- **Add or edit IP cores** in `src/ip/`
- **Add or edit constraints** in `constraints/`

---

### Generating the Bitstream

In Vivado, click **Generate Bitstream** to synthesize and implement your design.

---

### Exporting Hardware for Petalinux

After generating the bitstream, export the hardware for Petalinux:

1. **File → Export → Export Hardware...**
2. **Include bitstream** and **Local to Project**
3. **Export to `hardware/design_1_wrapper.xsa`**

---

## Petalinux Project Setup
To create the project - 

petalinux-create --type project --template microblaze --name <project-name>

cd <project-name>

### Navigate to the Petalinux Project

cd arty



---

### Configure the Project with Exported Hardware

petalinux-config --get-hw-description=(XSA-file)
(No changes just save and exit)



---

### Device Tree Customization
You can see the error so accordingly go to address editor of Vivado of AXI Timer and accordingly see that you might see the errors 
![Screenshot from 2025-06-14 03-01-27](https://github.com/user-attachments/assets/32be6699-6ccb-4d08-ba3b-40a4b859813c)


To ensure the Linux kernel loads properly, customize the device tree by editing:

arty/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi



**Example:**  
Add the following to `system-user.dtsi` to include a timer:

/include/ "system-conf.dtsi"
/ {
timer@41c00000 {
compatible = "xlnx,xps-timer-1.00.a";
reg = <0x41c00000 0x10000>;
interrupt-parent = <&microblaze_0_axi_intc>;
interrupts = <1 2>;
clock-frequency = <100000000>;
};
};



After editing, rebuild the device tree:

petalinux-build -c device-tree



---

### Building the Petalinux System

petalinux-build

---

## JTAG Boot:

### Setting Up JTAG Boot Mode

- **Power off the board.**
- **Connect the JTAG port on the board to your host computer using the JTAG cable.**
- **Connect the UART port on the board to your host computer for console output.**
- **Set the boot mode switches on the board to JTAG mode.**  
  *(Refer to your board documentation for the correct switch settings.)*

---

### Booting with JTAG

Use the following command to boot the system directly via JTAG:

petalinux-boot --jtag --fpga --u-boot --kernel --rootfs


**Explanation:**
- `--jtag`: Boot via JTAG
- `--fpga`: Download the FPGA bitstream
- `--u-boot`: Download and run U-Boot
- `--kernel`: Download and boot the Linux kernel
- `--rootfs`: Download and use the root filesystem

This will download the bitstream, U-Boot, kernel, and root filesystem directly to the board via JTAG.

---

## Monitoring the Console

Open a terminal emulator to monitor the console output:

screen /dev/ttyUSB0 115200


*(Replace `/dev/ttyUSB0` with your actual UART device.)*

## U-Boot
![WhatsApp Image 2025-06-15 at 01 08 12](https://github.com/user-attachments/assets/4aeebc7b-5f28-4b2a-a6b1-637365f534ec)


## Linux-Booting
![Screenshot from 2025-06-15 00-15-44](https://github.com/user-attachments/assets/bb1bdce0-215e-44cc-be23-b381238ea0ec)
![Screenshot from 2025-06-15 00-17-52](https://github.com/user-attachments/assets/3dcdc6c9-529b-4cd3-9aa8-6756b899b458)
![Screenshot from 2025-06-15 00-20-05](https://github.com/user-attachments/assets/c1c8af60-b115-4e89-ab8f-6b90e0acf722)


---

## Troubleshooting

- **JTAG not detected:**  
  - Ensure the JTAG cable is properly connected.
  - Check that the board is powered on and in JTAG boot mode.
- **U-Boot or kernel does not boot:**  
  - Double-check the Petalinux configuration and build process.
  - Ensure the correct bitstream and images are being used.
- **No console output:**  
  - Check the UART connection and terminal settings (baud rate 115200).
- **Root filesystem issues:**  
  - If you use `--rootfs` with JTAG boot, ensure the rootfs image is valid.
  - For large root filesystems, consider using TFTP boot (see [Advanced: TFTP Boot](#advanced-tftp-boot-optional) below).

---

## Project Commands Cheatsheet

| Command | Description |
|---------|-------------|
| `git clone https://github.com/ishanurg/ArtyS7-25-Petalinux.git` | Clone the repository |
| `vivado -mode batch -source scripts/s7-linux.tcl` | Recreate the Vivado project |
| `vivado s7-linux.xpr` | Open the Vivado project |
| `petalinux-config --get-hw-description=../hardware` | Configure Petalinux with hardware |
| `petalinux-build -c device-tree` | Build device tree |
| `petalinux-build` | Build the Petalinux project |
| `petalinux-boot --jtag --fpga --u-boot --kernel --rootfs` | Boot via JTAG (no SD card, no flash) |
| `screen /dev/ttyUSB0 115200` | Monitor the console |

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Acknowledgments

- **Digilent** for the Arty S7-25 board
- **Xilinx/AMD** for Vivado and Petalinux
- **The open source community** for support and inspiration


---

**Thank you for using ArtyS7-25-Petalinux!**  
Happy hacking!

---

