<div align="center">

<h1>🖥️ BootTerminal</h1>

<p><strong>Test bootable USB drives directly from your terminal — no VirtualBox, no GUI, no rebooting.</strong></p>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Open Source](https://img.shields.io/badge/Open%20Source-%E2%9D%A4-red)](https://github.com/NunesFilipe/BootTerminal)
[![Shell: Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Powered by QEMU](https://img.shields.io/badge/Powered%20by-QEMU-orange)](https://www.qemu.org/)
[![Platform: Linux](https://img.shields.io/badge/Platform-Linux-blue?logo=linux)](https://kernel.org/)

</div>

---

## Overview

**BootTerminal** is a lightweight, open-source Bash TUI tool that lets you boot and test any physical USB drive using **QEMU** — directly from your terminal, in seconds.

It automatically detects connected USB drives, lets you choose between **Legacy BIOS** (MBR) and **UEFI** (GPT/EFI) boot modes, and launches a QEMU window simulating a real boot from that device.

> No virtual machine setup. No rebooting. Just plug and test.

---

## Features

| Feature | Description |
|---|---|
| 🔍 Auto-detection | Automatically lists all connected USB block devices |
| 🖥️ Dual boot mode | Supports Legacy BIOS (MBR) and UEFI (OVMF/EFI) |
| ⚡ KVM acceleration | Near-native boot speed with hardware virtualization |
| 🎨 Colorful TUI | Intuitive terminal interface with keyboard navigation |
| 🔄 Rescan | Rescan USB devices without restarting the tool |
| 🛡️ Safe | Read-only raw disk access — no data is written |
| 📦 Minimal deps | Only requires QEMU and standard Linux tools |

---

## Requirements

| Dependency | Arch Linux | Debian / Ubuntu |
|---|---|---|
| `qemu-system-x86_64` | `qemu-full` | `qemu-system-x86` |
| `lsblk` | `util-linux` *(pre-installed)* | `util-linux` *(pre-installed)* |
| OVMF *(UEFI only)* | `edk2-ovmf` | `ovmf` |

> **KVM** is strongly recommended. Your CPU must support Intel VT-x or AMD-V, and the `kvm` kernel module must be loaded.

---

## Installation

### 1. Install dependencies

**Arch Linux / Manjaro:**
```bash
sudo pacman -S qemu-full edk2-ovmf
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
git clone https://github.com/NunesFilipe/BootTerminal.git
cd BootTerminal
```

### 3. Install globally

```bash
sudo cp bios /usr/local/bin/bios
sudo chmod +x /usr/local/bin/bios
```

### 4. (Recommended) Grant USB access without sudo

```bash
sudo usermod -aG disk $USER
```

> Log out and back in for the group change to take effect.

---

## Usage

```bash
bios
```

### Key bindings

| Key | Action |
|---|---|
| `1` – `N` | Select a USB drive |
| `L` | Boot in Legacy BIOS mode (MBR) |
| `U` | Boot in UEFI mode (GPT/EFI) |
| `R` | Rescan connected USB devices |
| `V` | Go back to the previous menu |
| `Q` | Quit |
| `Ctrl+C` | Force quit at any time |

To stop a boot session, close the QEMU window or press `Ctrl+Alt+Q` inside it.

---

## How It Works

1. **Detection** — Uses `lsblk -d -P -o NAME,SIZE,TRAN` to list USB block devices (whole disks only, no partitions).
2. **Model info** — Reads `/sys/block/<device>/device/model` for the human-readable drive name.
3. **QEMU launch**:
   - **Legacy BIOS**: `qemu-system-x86_64` with `-machine type=pc` and KVM acceleration.
   - **UEFI**: Same, but with `-machine type=q35` and `-bios /usr/share/edk2/x64/OVMF.4m.fd`.
4. **Permissions** — If the device isn't readable by the current user, `sudo` is invoked automatically.

---

## OVMF Firmware Path

By default, BootTerminal looks for OVMF at:

```
/usr/share/edk2/x64/OVMF.4m.fd
```

If your distro places it elsewhere, update `OVMF_PATH` at the top of the `bios` script:

```bash
OVMF_PATH="/usr/share/OVMF/OVMF_CODE.fd"  # Debian/Ubuntu example
```

---

## Troubleshooting

<details>
<summary><strong>Permission denied on /dev/sdX</strong></summary>

Add your user to the `disk` group:

```bash
sudo usermod -aG disk $USER
```

Then log out and back in. Verify with:

```bash
groups $USER
```

</details>

<details>
<summary><strong>QEMU window doesn't open</strong></summary>

Make sure your display server (X11 or XWayland) is running and QEMU was compiled with SDL support:

```bash
qemu-system-x86_64 -display help
```

</details>

<details>
<summary><strong>KVM not available</strong></summary>

Check if the module is loaded:

```bash
lsmod | grep kvm
```

Load it manually if needed:

```bash
sudo modprobe kvm_intel   # Intel
sudo modprobe kvm_amd     # AMD
```

</details>

<details>
<summary><strong>UEFI firmware not found</strong></summary>

Install the OVMF package for your distro and update `OVMF_PATH` in the script accordingly.

</details>

---

## Project Structure

```
BootTerminal/
├── bios        # Main executable script
└── README.md   # Documentation
```

---

## Contributing

BootTerminal is **open source** and contributions are welcome!

Whether it's a bug fix, a new feature, or a UI improvement — feel free to fork the repository, make your changes, and open a pull request.

**Ideas for contributions:**
- ARM / RISC-V architecture support
- Custom RAM and CPU configuration via flags
- Distribution-specific install scripts
- UI improvements and accessibility

If you fork or redistribute this project, please keep the original authorship notice in the script header:

```
Originally created by Filipe Nunes — https://github.com/NunesFilipe/BootTerminal
```

---

## License

Distributed under the **MIT License**. See [`LICENSE`](./LICENSE) for more information.

---

<div align="center">
Made with ❤️ by <a href="https://github.com/NunesFilipe">Filipe Nunes</a>
</div>
