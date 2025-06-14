# ArtyS7-25-Petalinux

**A comprehensive, step-by-step guide to building and booting a custom Linux system on the Digilent Arty S7-25 board using only JTAG—no SD card or flash storage required.**

---

## Table of Contents

- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Vivado Project Setup](#vivado-project-setup)
  - [Downloading and Using the .tcl Script](#downloading-and-using-the-tcl-script)
  - [Generating the Bitstream](#generating-the-bitstream)
  - [Exporting Hardware for Petalinux](#exporting-hardware-for-petalinux)
- [Petalinux Project Setup](#petalinux-project-setup)
  - [Configuring the Project with Exported Hardware](#configuring-the-project-with-exported-hardware)
  - [Device Tree Customization](#device-tree-customization)
  - [Building the Petalinux System](#building-the-petalinux-system)
- [JTAG Boot: No SD Card, No Flash](#jtag-boot-no-sd-card-no-flash)
  - [Setting Up JTAG Boot Mode](#setting-up-jtag-boot-mode)
  - [Booting with JTAG](#booting-with-jtag)
- [Monitoring the Console](#monitoring-the-console)
- [Troubleshooting](#troubleshooting)
- [Project Commands Cheatsheet](#project-commands-cheatsheet)
- [Inserting Photographs](#inserting-photographs)
- [FAQs](#faqs)
- [Advanced: TFTP Boot (Optional)](#advanced-tftp-boot-optional)
- [License](#license)
- [Acknowledgments](#acknowledgments)
- [Contact](#contact)

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
- **Linux host machine** (Ubuntu 20.04/22.04 recommended)
- **JTAG programmer** (e.g., Digilent USB-JTAG cable)
- **USB cable** for UART communication
- **Terminal emulator** (e.g., `screen`, `minicom`)

---

## Project Structure

s7-linux/
├── .gitignore
├── README.md
├── src/
│ ├── design/ # HDL sources (VHDL, Verilog, SV)
│ ├── bd/ # Block design files (.bd)
│ ├── ip/ # IP core files (.xci, .xcix)
│ └── testbench/ # Testbenches
├── constraints/
│ └── *.xdc # Constraint files
├── hardware/
│ └── design_1_wrapper.xsa # Exported hardware for Petalinux
├── scripts/
│ └── s7-linux.tcl # Vivado project recreation script
├── arty/ # Petalinux project
└── LICENSE

text

---

## Getting Started

### Clone the Repository

git clone https://github.com/ishanurg/ArtyS7-25-Petalinux.git
cd ArtyS7-25-Petalinux

text

---

### Install Dependencies

On Ubuntu, install the following packages:

sudo apt update
sudo apt install -y build-essential git python3 python3-pip device-tree-compiler

text

---

### Set Up Vivado and Petalinux Environment

Source the Vivado and Petalinux environment scripts in your terminal:

source /tools/Xilinx/Vivado/2024.2/settings64.sh
source /tools/Xilinx/Petalinux/2024.2/settings.sh

text

*(Adjust paths according to your installation.)*

---

## Vivado Project Setup

### Downloading and Using the .tcl Script

You can use the provided `.tcl` script to recreate the Vivado project on any machine.  
**Download or clone the repository to get the script:**

git clone https://github.com/ishanurg/ArtyS7-25-Petalinux.git
cd ArtyS7-25-Petalinux

text

**To generate the Vivado project, run:**

vivado -mode batch -source scripts/s7-linux.tcl

text

This will create the Vivado project in your current directory.

---

### Opening the Project in Vivado

vivado s7-linux.xpr

text

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

### Navigate to the Petalinux Project

cd arty

text

---

### Configure the Project with Exported Hardware

petalinux-config --get-hw-description=../hardware

text

---

### Device Tree Customization

To ensure the Linux kernel loads properly, customize the device tree by editing:

arty/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi

text

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

text

After editing, rebuild the device tree:

petalinux-build -c device-tree

text

---

### Building the Petalinux System

petalinux-build

text

*(Optional: Package prebuilt images with `petalinux-package --prebuilt`.)*

---

## JTAG Boot: No SD Card, No Flash

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

text

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

text

*(Replace `/dev/ttyUSB0` with your actual UART device.)*

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

## Inserting Photographs

You can insert your photographs at any point in this guide.  
For example, after the JTAG boot section, add:

text

Or, after the Vivado project setup, add:

text

---

## FAQs

**Q: Can I boot without an SD card or flash?**  
A: Yes, you can boot directly via JTAG using the `petalinux-boot --jtag` command.

**Q: How do I monitor the console output?**  
A: Use `screen /dev/ttyUSB0 115200` or a similar terminal emulator.

**Q: What if the root filesystem is too large for JTAG?**  
A: Use TFTP boot to load the root filesystem over the network.  
*(See [Advanced: TFTP Boot](#advanced-tftp-boot-optional) below.)*

---

## Advanced: TFTP Boot (Optional)

If you want to load the kernel, rootfs, and device tree via TFTP (useful for large images or development), you can use U-Boot commands:

1. **Set up a TFTP server on your host.**
2. **Copy the following files to your TFTP directory:**
   - `image.ub` (kernel)
   - `rootfs.cpio.gz` (root filesystem)
   - `system.dtb` (device tree)
3. **In U-Boot, use the following commands:**

tftpboot 0x1000000 /tftpboot/image.ub
tftpboot 0x2000000 /tftpboot/rootfs.cpio.gz
tftpboot 0x3000000 /tftpboot/system.dtb
bootm 0x1000000 0x2000000 0x3000000

text

*(Adjust addresses as needed for your system.)*

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Acknowledgments

- **Digilent** for the Arty S7-25 board
- **Xilinx/AMD** for Vivado and Petalinux
- **The open source community** for support and inspiration

---

## Contact

For questions or feedback, open an issue on GitHub or contact the maintainer.

---

**Thank you for using ArtyS7-25-Petalinux!**  
Happy hacking!

---

