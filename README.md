# BL602 Level 2 Proxy

[![Build & Release](https://img.shields.io/badge/Build-GitHub%20Actions-blue.svg)](https://github.com)
[![Platform](https://img.shields.io/badge/Platform-BL602%20RISC--V-orange.svg)](https://en.bouffalolab.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A specialized, high-performance firmware implementation for the **Bouffalo Lab BL602 RISC-V Wi-Fi & BLE SoC** that operates as a transparent **Layer 2 (Data Link) Network Proxy**.

---

## 📖 Overview & Architecture

### What is a Level 2 Proxy?

In traditional networking, proxy servers and routers operate at **Layer 3 (Network Layer)** or **Layer 7 (Application Layer)** of the OSI model. They route packets based on IP addresses, inspect headers, perform Network Address Translation (NAT), and establish socket connections.

A **Level 2 (L2) Proxy**, by contrast, operates directly at the **Data Link Layer**. It intercepts, processes, and forwards raw **Ethernet frames (802.3 / 802.11)** based on MAC addresses and physical frame headers without traversing an IP stack.

```
       +-------------------------------------------------------+
       |                  OSI Network Stack                    |
       +-------------------------------------------------------+
       | Layer 7: Application (HTTP, MQTT, SSH)                |
       | Layer 3: Network     (IP, ICMP, Routing)              |
  =====|=======================================================|=====
  ====>| Layer 2: Data Link   (MAC Frames, Ethernet, IEEE 802) |<==== BL602 L2 PROXY OPERATES HERE
  =====|=======================================================|=====
       | Layer 1: Physical    (RF, 2.4GHz Wi-Fi Radio)         |
       +-------------------------------------------------------+
```

### Why Use a Layer 2 Proxy on BL602?

* **Might be useful to bridge some distant iot devices

---

## ✨ Key Features

### 1. Initial Access Point (`bl602proxy`) Provisioning
When unconfigured, out of the box, or following a factory reset, the device automatically initializes a dedicated SoftAP (Access Point) for network setup.

* **SSID:** `bl602proxy`
* **Default IP / URL:** `http://192.168.4.1/`
* **Security:** Open by default during provisioning (configurable in settings).
* **Built-in Web Portal:** Light-weight HTTP server providing standard network configuration fields (Station SSID, WPA2/WPA3 password, static IP vs. DHCP, L2 pass-through filters).

### 2. Dual-Access Web Management
Unlike basic IoT provisioning flows that disable the configuration interface once connected to Wi-Fi, the BL602 Level 2 Proxy keeps the administration portal active across both network contexts:

* **SoftAP Mode (`192.168.4.1`):** Always accessible when connected directly to the device's setup Wi-Fi.
* **Station Mode (Local Network IP):** Accessible on your home or enterprise local network via the DHCP-assigned or static IP address (e.g., `http://192.168.1.150/`).
* **Live System Telemetry:** View frame counters, dropped packet metrics, signal strength (RSSI), uptime, and memory status.

### 3. 5-Cycle Power-Toggle Hardware Recovery (Fail-Safe)
If the Wi-Fi router password changes, network configurations become invalid, or the device is assigned an unreachable IP address, you can perform a factory reset without needing physical button hardware or UART flashing tools.

* **Recovery Procedure:**
  1. Turn the power **ON**.
  2. Wait **2 to 3 seconds**.
  3. Turn the power **OFF**.
  4. Repeat this sequence **5 consecutive times**.
* **Result:** On the 5th reboot, the firmware detects the rapid power-cycle boot flag stored in RTC memory, clears invalid Wi-Fi configuration, resets to default parameters, and restores the `bl602proxy` setup Access Point at `http://192.168.4.1/`.

---

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── build.yml          # Automated CI/CD pipeline for tag releases
├── src/                       # Custom L2 proxy application source files
├── overrides/                 # Patch files (0001-xxx.patch) applied to SDK
├── bl_iot_sdk/                # Bouffalo Lab IoT SDK (Git Submodule)
└── README.md                  # Project documentation
```

---

## 🛠️ Building & Compilation

### Prerequisites

To compile the firmware locally, ensure you have the following toolchain installed:
* **RISC-V Toolchain:** `riscv64-unknown-elf-gcc` (Bouffalo Lab customized toolchain)
* **Build Utilities:** `make`, `bash`, `git`, `python3`

### Step-by-Step Local Build

1. **Clone the repository with submodules:**
   ```bash
   git clone --recursive https://github.com/your-username/bl602_level2_proxy.git
   cd bl602_level2_proxy
   ```

2. **Apply SDK Patches:**
   Navigates to the SDK submodule and applies custom project overrides:
   ```bash
   cd bl_iot_sdk
   for patch in ../overrides/*.patch; do
     if [ -f "$patch" ]; then
       echo "Applying $patch..."
       git apply "$patch"
     fi
   done
   cd ..
   ```

3. **Stage Source Files & Compile:**
   ```bash
   # Copy customer application into SDK directory structure
   mkdir -p bl_iot_sdk/customer_app/bl602_level2_proxy
   cp -r src/* bl_iot_sdk/customer_app/bl602_level2_proxy/

   # Run build generator script
   cd bl_iot_sdk/customer_app/bl602_level2_proxy
   chmod +x ./genromap
   ./genromap
   ```

4. **Output Binary:**
   Upon successful compilation, the final flashable binary is created at:
   ```
   bl_iot_sdk/customer_app/bl602_level2_proxy/build_out/bl602_level2_proxy.bin
   ```

---

## ⚡ Flashing the Firmware

You can flash `bl602_level2_proxy.bin` to your BL602 module (e.g., Pine64 PineCone, DT-BL10, DoIt ESP-M2) using **`blflash`** or Bouffalo Lab's official **`DevCube`** software.

### Flash via `blflash` (CLI)

1. Put the BL602 into bootloader mode (pull `BOOT2` / `GPIO8` to High/3.3V while powering on or resetting).
2. Execute the flash command:
   ```bash
   blflash flash bl_iot_sdk/customer_app/bl602_level2_proxy/build_out/bl602_level2_proxy.bin --port /dev/ttyUSB0
   ```
3. Remove the BOOT jumper and power cycle the board.

---

## 🤖 Continuous Integration & Releases

This repository includes an automated **GitHub Actions** workflow (`.github/workflows/build.yml`) that triggers on tag pushes (`v*`).

The CI pipeline automatically:
1. Checks out the repository and `bl_iot_sdk` submodules recursively.
2. Applies sequential patches from the `overrides/` folder (`0001-*.patch`, `0002-*.patch`, etc.).
3. Stages the `./src` contents into `bl_iot_sdk/customer_app/bl602_level2_proxy`.
4. Executes `./genromap` to compile the application.
5. Publishes a new GitHub Release containing `bl602_level2_proxy.bin` as a downloadable release asset.

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
