<div align="center">

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║          🖥   B I O S   T E R M I N A L                 ║
║              Bootable USB Drive Tester                   ║
║                       v1.0                               ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

**Test your bootable USB drives directly from the terminal — no VirtualBox, no GUI setup.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Shell: Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Powered by QEMU](https://img.shields.io/badge/Powered%20by-QEMU-orange?logo=qemu)](https://www.qemu.org/)
[![Platform: Linux](https://img.shields.io/badge/Platform-Linux-blue?logo=linux)](https://kernel.org/)

</div>

---

## What is BiosTerminal?

**BiosTerminal** is a lightweight Bash TUI (Terminal User Interface) tool that lets you boot and test any physical USB drive using **QEMU** — directly from your terminal, in seconds. No need to configure VirtualBox, no need to reboot your machine.

It automatically detects all connected USB drives, lets you choose between **Legacy BIOS** (MBR) and **UEFI** (GPT/EFI) boot modes, and launches a QEMU window simulating a real boot from that device.

Perfect for:
- Verifying that bootable USB drives actually boot
- Testing Linux/Windows ISOs written to USB
- Quickly checking if a pendrive was correctly flashed
- Developers and sysadmins who need a fast boot testing workflow

---

## Features

- 🔍 **Auto-detection** of all connected USB block devices
- 🖥️ **Dual boot mode**: Legacy BIOS (MBR) and UEFI (OVMF/EFI)
- ⚡ **KVM acceleration** for near-native boot speed
- 🎨 **Colorful TUI** with intuitive keyboard navigation
- 🔄 **Rescan** USB devices without restarting the tool
- 🛡️ **Safe**: read-only raw disk access, no data is written
- 📦 **Zero dependencies** beyond QEMU and standard Linux tools

---

## Requirements

| Dependency | Package (Arch) | Package (Debian/Ubuntu) |
|---|---|---|
| `qemu-system-x86_64` | `qemu-full` | `qemu-system-x86` |
| `lsblk` | `util-linux` *(pre-installed)* | `util-linux` *(pre-installed)* |
| OVMF firmware *(UEFI only)* | `edk2-ovmf` | `ovmf` |

> **KVM** is strongly recommended for performance. Your CPU must support hardware virtualization (Intel VT-x / AMD-V), and the `kvm` kernel module must be loaded.

---

## Installation

### 1. Install dependencies

**Arch Linux / Manjaro:**
```bash
sudo pacman -S qemu-full edk2-ovmf util-linux
```

**Debian / Ubuntu / Mint:**
```bash
sudo apt install qemu-system-x86 ovmf
```

**Fedora:**
```bash
sudo dnf install qemu-system-x86 edk2-ovmf
```

### 2. Clone the repository

```bash
git clone https://github.com/NunesFilipe/BiosTerminal.git
cd BiosTerminal
```

### 3. Install the script globally

```bash
sudo cp bios /usr/local/bin/bios
sudo chmod +x /usr/local/bin/bios
```

### 4. (Recommended) Grant USB device access without sudo

Add your user to the `disk` group so you can access block devices without `sudo`:

```bash
sudo usermod -aG disk $USER
```

> **Important:** Log out and log back in for the group change to take effect.

---

## Usage

Simply run:

```bash
bios
```

### Navigation

| Key | Action |
|-----|--------|
| `1`, `2`, ... `N` | Select a USB drive from the list |
| `R` | Rescan connected USB devices |
| `L` | Boot in **Legacy BIOS** mode (MBR) |
| `U` | Boot in **UEFI** mode (GPT/EFI) |
| `V` | Go back to the previous menu |
| `Q` | Quit |
| `Ctrl+C` | Force quit at any time |

### Stopping a boot session

Close the QEMU window directly, or press `Ctrl+Alt+Q` inside the QEMU window. You will be returned to the BiosTerminal menu automatically.

---

## How It Works

1. **Detection**: Uses `lsblk -d -P -o NAME,SIZE,TRAN` to list block devices. Filters only USB-connected whole disks (no partitions).
2. **Model info**: Reads `/sys/block/<device>/device/model` for the human-readable drive name.
3. **QEMU launch**:
   - **Legacy mode**: `qemu-system-x86_64` with `-machine type=pc`, KVM acceleration, and the USB drive as a raw virtio disk.
   - **UEFI mode**: Same as above but with `-machine type=q35` and `-bios /usr/share/edk2/x64/OVMF.4m.fd`.
4. **Permissions**: If the device is not readable by the current user, `sudo` is used automatically.

---

## OVMF Firmware Path

By default, BiosTerminal looks for the OVMF firmware at:

```
/usr/share/edk2/x64/OVMF.4m.fd
```

If your distribution places it elsewhere, edit the `OVMF_PATH` variable at the top of the `bios` script:

```bash
OVMF_PATH="/usr/share/OVMF/OVMF_CODE.fd"  # Debian/Ubuntu path example
```

---

## Troubleshooting

### "Permission denied" when accessing `/dev/sdX`

Make sure your user is in the `disk` group and you have logged out and back in:

```bash
groups $USER  # should include 'disk'
```

If not, run:
```bash
sudo usermod -aG disk $USER
```
Then log out and log back in.

### QEMU window doesn't open

Ensure your display server (X11 or XWayland) is running and `qemu-system-x86_64` was compiled with SDL support:

```bash
qemu-system-x86_64 -display help
```

### KVM not available

Check if the kvm module is loaded:

```bash
lsmod | grep kvm
```

Load it manually if needed:
```bash
sudo modprobe kvm_intel   # Intel CPUs
sudo modprobe kvm_amd     # AMD CPUs
```

### UEFI firmware not found

Install the OVMF package for your distribution and/or update `OVMF_PATH` in the script to match the correct path.

---

## Project Structure

```
BiosTerminal/
├── bios        # Main executable script
└── README.md   # This file
```

---

## Contributing

Contributions are welcome! Feel free to open issues or pull requests for:
- New features (e.g., ARM support, custom RAM configuration)
- Bug fixes
- Distribution-specific improvements
- UI enhancements

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2026 Filipe Nunes

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">
Made with ❤️ by <a href="https://github.com/NunesFilipe">Filipe Nunes</a>
</div>
