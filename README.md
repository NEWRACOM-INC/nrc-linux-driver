# NRC HaLow WiFi Driver for Linux Kernel

This repository contains the Newracom NRC HaLow (IEEE 802.11ah) WiFi driver for standalone Linux kernel builds.

## Purpose

This repository is specifically for building the NRC driver for standard Linux kernels (Raspberry Pi, Ubuntu, etc.), separate from the OpenWrt build system in the [newracom](https://github.com/NEWRACOM-INC/newracom) repository.

## Source

Driver source is derived from NRC_MACSW repository, branch `nrc7394-release-hm-nettop-8k`.

## Build Requirements

- Linux kernel headers for your target kernel version
- GCC cross-compiler (if cross-compiling)
- Make build tools

## Quick Build

### On Target System (e.g., Raspberry Pi)

```bash
cd nrc
make
sudo make modules_install
sudo insmod nrc.ko
```

### Cross-Compile Example (for Raspberry Pi from x86_64)

```bash
cd nrc
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- KDIR=/path/to/rpi/kernel/headers
```

## Testing

```bash
cd nrc
make test  # Builds, loads, and unloads the module
```

## Clean Build

```bash
cd nrc
make clean
make
```

## Configuration

The Makefile includes the following compile-time flags:
- `CONFIG_NRC_HIF_CSPI`: CSPI interface support
- `ENABLE_DYNAMIC_PS`: Dynamic power save
- `CONFIG_NRC_BADVA_FIX`: Bad virtual address fix
- `CONFIG_NRC_TRACING`: Kernel tracing support

## Directory Structure

```
nrc-linux-driver/
├── nrc/                    # Driver source code
│   ├── Makefile           # Build configuration
│   ├── nrc-mac80211.c     # Main MAC80211 implementation
│   ├── nrc-hif-cspi.c     # CSPI interface
│   └── ...                # Other driver components
└── README.md              # This file
```

## Supported Kernels

Tested on:
- Raspberry Pi kernel 6.6.74+rpt-rpi-v8 (aarch64)
- Other Linux kernels 4.15.0+

## License

GPL-2.0

## Related Repositories

- [newracom](https://github.com/NEWRACOM-INC/newracom) - OpenWrt-specific driver builds
