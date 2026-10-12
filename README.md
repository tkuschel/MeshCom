# MeshCom with the CI/CD

Build MeshCom through the github.com yaml file:
```text
build-fork-ci
```

## Legal Notice & Intellectual Property
This page (a fork of the MeshCom-Firmware by icssw-org, inclusive of build-fork-ci) is dedicated exclusively to the further development and debugging of the source code. Notice is hereby given that any supplementary code contained herein remains subject to my proprietary rights; no authorization is granted for the distribution or sharing thereof outside of this repository, whether in whole or in part. Upon submission and acceptance of a pull request to the original upstream repository, the respective rights for those submitted parts shall be granted to its operators under the project's standard MIT license terms, while copyright and authorship of the specific components remain credited to me.

## Workflow for Fixes, Issues, Features
Always open a fresh **branch** for each new code contribution (excluding any YAML files) from the official development branch `dev`, rather than from the local default testing branch.

---

## Preparation Local

### Local Git Exclude

To ignore specific development files only on your local machine without changing the project's main `.gitignore`, add these patterns to your local exclude file.

**File Path:** `.git/info/exclude`

Copy and paste the following content into that file:

```text
# git ls-files --others --exclude-from=.git/info/exclude
# Lines that start with '#' are comments.
# For a project mostly in C, the following would be a good set of
# exclude patterns (uncomment them if you want to use them):
# *.[oa]
# *~

# Codium / VS Code
# created through 'pio run -t compiledb'
compile_commands.json

# Local binaries
safeboot.bin
safeboot-s3.bin

# Additional Information:
# You can also ignore these files temporarily with:
#
#	git update-index --assume-unchanged safeboot.bin
#	git update-index --assume-unchanged safeboot-s3.bin
```

---

### CI Build & Synchronization Workflow

This fork uses `build-fork-ci` as a private testing environment. **Never create Pull Requests (PRs) from the `build-fork-ci` branch.** Its sole purpose is to trigger automated builds with your custom setup.

The official development branch of the MeshCom-Firmware remains `dev`.
All feature and issue branches must branch off from `dev`.

#### Initial Setup (One-time)
Add the official MeshCom repository as an upstream remote to pull the latest changes:
```bash
git remote add upstream https://github.com/icssw-org/MeshCom-Firmware.git
```

#### Step 1: Always Update the Local 'dev' Branch First
Before doing anything (testing or developing), always fetch the freshest updates from the official project into your local `dev` branch:
```bash
git checkout dev
git pull upstream dev
```

---

### Path A: Testing the Latest Code in your Private CI
If you want to sync the latest upstream changes into your private testing environment and trigger a build:

#### 1. Merge Updates into your CI Branch and Protect README.md
Switch to your testing branch, merge the new code from `dev`, and ensure your custom `README.md` is not overwritten:
```bash
git checkout build-fork-ci
git merge dev --no-commit --no-ff

# Force Git to keep our custom README.md instead of the upstream version
git checkout --ours README.md

# Complete the merge and push to trigger your private CI
git add README.md
git commit -m "Merge upstream updates and preserved custom CI README"
git push origin build-fork-ci
```

---

### Path B: Working on a GitHub Issue (Creating a PR)
If you want to fix a bug or add a feature to contribute back to the official project:

#### 1. Create your Issue Branch from 'dev'
Since your local `dev` branch is now perfectly up to date (from Step 1), safely branch off:
```bash
git checkout dev
git checkout -b issue-123
```

#### 2. Commit and Push your Changes
Work on your code, commit it, and push it to your GitHub fork to open a clean Pull Request:
```bash
git add changed_file.c
git commit -m "Fix bug described in issue 123"
git push origin issue-123
```

#### 3. Branch Naming Conventions
To maintain a clean and structured repository overview, please use the following prefixes when naming your issue branches:
* **`bugfix/issue-123-short-description`** – Use this for fixing broken code, crashes, or unintended firmware behavior.
* **`feature/issue-456-new-sensor`** – Use this for introducing brand new functionalities, supporting new hardware modules, or adding protocol improvements.
* **`docs/issue-789-update-info`** – Use this exclusively for modifying or expanding code documentation, comments, or project guides without touching executable logic.
* **`chore/issue-999-cleanup-config`** – Use this for routine tasks, build system maintenance, updating dependencies, or refactoring code structure without changing internal logic or adding features.

---

## Statement
This README is subject to change and serves as the definitive terms of use for this repository fork.

-- October 2026, Klosterneuburg

---
---

## 📑 Condensed Project Overview (As of October 2026)

# MeshCom
MeshCom is an exciting project of the Institute of Citizen Science for Space & Wireless communication (www.icssw.org) aimed at creating a resilient, text-based communication tool for amateur radio operators.
It utilizes LORA™ modulation technology and the APRS protocol to establish a mesh network in the 70cm band.

## Basic functions:
- Each Node is identified by an Amateur Radio Callsign (with optional SSID)
- Short test messages can be sent to ALL (everybody), with ACK from Server/Gateway
- Short text messages can be sent DIRECTLY to other callsigns, with End-to-End Acknowledgment
- Nodes automatically send STATUS, POSITION, and WX/TELEMETRY data periodically
- Messages will be displayed on small OLED Display, via Bluetooth apps, or Serial Console

## Frequency in Region:
EU / USA / Africa: 433.175 MHz | UK: 439.9125 MHz (BW:125k SF:10) | Norway: 433.925 MHz (BW:125k SF:10)

## LoRa parameters:
SF: 11 | Bandwidth: 250kHz | CR: 4/6

## MeshCom 4.0 Firmware & Protocol
MeshCom 4.0 uses the APRS PROTOCOL REFERENCE for the source, destination, Digipeater, and payload data as defined for AX.25 APRS packets.

---

## Preparations for platform.io VSCode plugin
Install the needed frameworks under Platforms: 
+ Espressif 32
+ Nordic nRF52

---

## Flashing Firmware

### 1. Smart Desktop Flashing via CI Artifacts (Recommended for TTGO T-Beam)
When you build your code using the **Automated Fork CI**, it generates a specialized, unzipped artifact package named `ttgo-tbeam-desktop-flasher`.
This package contains the binaries and plug-and-play desktop helper scripts.

1. Download the `ttgo-tbeam-desktop-flasher` artifact directory from your GitHub Action run.
2. Connect your T-Beam board via a **USB Data Cable** to your computer.
3. Run the script matching your operating system:
   * **Linux:** Run `./flash_tbeam.sh` via your favorite shell (`bash`/`zsh`).
   * **Windows:** Double-click `flash_tbeam.bat`.

#### 💥 Core Features of the Desktop Flasher Scripts:
* **Auto-Port Detection:** Automatically scans your connected hardware nodes for active USB-to-UART bridges (CP210x, CH340, CH9102) so you don't have to specify serial ports manually.
* **Smart Bootloader Detection:** If `bootloader.bin` is present in the folder, the script automatically switches strategies to flash **both** the bootloader (at `0x1000`) and the firmware (at `0x10000`) sequentially. If missing, it flashes the firmware standalone at maximum safe speed (`921600` baud).
* **Dynamic Permission Validation (Linux):** Dynamically probes which system group owns the hardware serial port (e.g., `uucp` on Arch Linux, `dialout` on Debian/Ubuntu), checks if your user is a member, and outputs exact correction commands if access is denied.

---

### 2. Manual ESP32 Flash via Command Line:
If you prefer to flash everything manually using an installed `esptool.py` binary, navigate to your compiled `.pio/build/<board>` directory and utilize the following memory layouts:

#### Classic ESP32 Boards (e.g., TTGO T-Beam V1.0 / V1.1)

| Address | File |
| --- | ----------- |
| 0x1000 | bootloader.bin |
| 0x8000 | partitions.bin |
| 0xE000 | otadata.bin |
| 0x10000 | safeboot.bin |
| 0xC0000 | firmware.bin |

```bash
esptool.py -p <SERIAL_PORT> write_flash 0x1000 bootloader.bin 0xE000 otadata.bin 0x8000 partitions.bin 0x10000 safeboot.bin 0xC0000 firmware.bin
```

#### ESP32-S3 Boards (e.g., Heltec V3, E290, T-Beam Supreme)

| Address | File |
| --- | ----------- |
| 0x0000 | bootloader-s3.bin |
| 0x8000 | partitions.bin |
| 0xE000 | otadata.bin |
| 0x10000 | safeboot-s3.bin |
| 0xC0000 | firmware.bin |

```bash
esptool.py --chip esp32s3 -p <SERIAL_PORT> write_flash 0x0000 bootloader-s3.bin 0xE000 otadata.bin 0x8000 partitions.bin 0x10000 safeboot-s3.bin 0xC0000 firmware.bin
```

*Alternatively, ready-built firmware images can be flashed inside a web browser using the [OEVSV Web Flasher](https://esptool.oevsv.at).*

---

### OTA-Update:
The `safeboot.bin` / `safeboot-s3.bin` images contain a lightweight factory firmware specifically for Over-The-Air web upgrades. 

1. Trigger OTA mode on your running node by entering the `--ota-update` CLI command over serial connection, or by clicking **OTA-Update** inside the integrated webserver or phone app.
2. The node will reboot into safety mode. Connect to its local web panel either via `<YOUR-CALLSIGN>.local` (if router WiFi credentials were saved beforehand) or connect to the local Access Point hosted by the module named `MeshCom-OTA` (Password-free, accessible at `192.168.4.1` or `MeshCom-OTA.local`).
3. Upload your binary directly through the browser interface.

#### Erasing the Firmware completely: 
```bash
esptool.py --port <SERIAL-PORT> erase_flash
```
#### Erasing exclusively saved Node Settings (NVS Region): 
```bash
esptool.py --port <SERIAL-PORT> erase_region 0x009000 0x005000
```

---

### RAK4631 via CLI:
Requires `adafruit-nrfutil` (`pip3 install adafruit-nrfutil`). Uses the `.zip` archive instead of raw binaries:
```bash
adafruit-nrfutil --verbose dfu serial --package wiscore_rak4631.zip -p <SERIAL_PORT> --singlebank --touch 1200
```

### RAK4631 via UF2 File (Mass Storage Emulation):
The WisBlock module supports a user-friendly USB mass storage bootloader for dragging and dropping firmware files.

1. Connect the RAK4631 module to your computer via a USB cable.
2. Double-click the physical reset button on the module. The board will mount on your operating system as a standard USB drive named `RAK4631`.
3. Drag and drop your compiled `.uf2` file directly onto the mounted drive. The module will automatically flash itself and reboot.

#### Generating a UF2 File from a compiled HEX File:
If you need to manually convert a compiled `.hex` payload into a flashable `.uf2` image, use Microsoft's conversion utility:

1. Download the official conversion script:
   ```bash
   wget https://raw.githubusercontent.com/microsoft/uf2/refs/heads/master/utils/uf2conv.py
   ```
2. Convert your firmware hex file using the specialized Nordic nRF52 family ID (`0xADA52840`):
   ```bash
   python3 uf2conv.py .pio/build/wiscore_rak4631/firmware.hex -c -o .pio/build/wiscore_rak4631/wiscore_rak4631.uf2 -f 0xADA52840
   ```

## Icon Licensing
MeshCom utilizes a GPS symbol as well as menu (green/red) and settings symbols designed by [WpZoom](https://www.wpzoom.com), provided under the terms of the [Creative Commons Attribution-Share Alike 3.0 License](https://creativecommons.org/licenses/by-sa/3.0/).
