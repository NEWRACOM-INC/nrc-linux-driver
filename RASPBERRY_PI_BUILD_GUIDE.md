# Raspberry Pi Build Guide - NRC HaLow Linux Driver

## Prerequisites

### Hardware Requirements
- Raspberry Pi 4 (or compatible model)
- NRC HaLow WiFi module connected via SPI
- Stable power supply
- Internet connection for initial setup

### Software Requirements
- Raspberry Pi OS (Debian-based)
- Linux kernel headers installed
- Git installed
- Build tools (gcc, make)

## Step-by-Step Build Instructions

### Step 1: Install Required Packages

```bash
sudo apt-get update
sudo apt-get install -y \
    git \
    build-essential \
    linux-headers-$(uname -r) \
    raspberrypi-kernel-headers
```

### Step 2: Verify Kernel Version

```bash
uname -r
# Expected output: 6.6.74+rpt-rpi-v8 (or similar)
```

**Important**: Different kernel versions require different branches. See "Kernel Version Compatibility" section below.

### Step 3: Clone Repository

```bash
cd ~
git clone https://github.com/NEWRACOM-INC/nrc-linux-driver.git
cd nrc-linux-driver
```

### Step 4: Checkout Appropriate Branch

Choose the branch matching your kernel version:

**For Kernel 6.6.x**:
```bash
git checkout kernel-6.6
```

**For Other Kernel Versions**:
```bash
# Check available branches
git branch -a

# For kernel 4.x - 5.x, use master branch
git checkout master
```

### Step 5: Navigate to Driver Directory

```bash
cd nrc
```

### Step 6: Clean Previous Builds (if any)

```bash
make clean
```

### Step 7: Build Driver

```bash
make -j4
```

**Build time**: Approximately 2-3 minutes on Raspberry Pi 4

**Expected output**:
```
  CC [M]  /home/pi/nrc-linux-driver/nrc/nrc-mac80211.o
  CC [M]  /home/pi/nrc-linux-driver/nrc/nrc-trx.o
  ...
  LD [M]  /home/pi/nrc-linux-driver/nrc/nrc.ko
```

### Step 8: Verify Build Success

```bash
ls -lh nrc.ko
file nrc.ko
```

**Expected output**:
```
-rw-r--r-- 1 pi pi 485K Jan  9 10:14 nrc.ko
nrc.ko: ELF 64-bit LSB relocatable, ARM aarch64, version 1 (SYSV), not stripped
```

### Step 9: Install Driver (Two Options)

#### Option A: Manual Load (Temporary)
```bash
sudo insmod nrc.ko
```

#### Option B: Install to System (Persistent)
```bash
sudo make modules_install
sudo depmod -a
```

### Step 10: Verify Driver Loaded

```bash
# Check kernel messages
dmesg | tail -20 | grep nrc

# Expected output:
# "Succeed to register spi driver(nrc80211)"

# Verify wlan0 interface
ip link show wlan0
```

### Step 11: (Optional) Replace nrc_pkg Driver

If you have existing nrc_pkg installation:

```bash
sudo cp nrc.ko ~/nrc_pkg/sw/driver/nrc.ko
```

### Step 12: Test with Sniffer (if applicable)

```bash
cd ~/nrc_pkg/script
sniffer <channel> <remote_state> [port]

# Example:
sniffer 9 1 2002
```

## Kernel Version Compatibility

### Master Branch (Default)
- **Target Kernels**: 4.14.x - 5.x
- **Status**: Original source from NRC_MACSW
- **Modifications**: None (baseline)
- **Use Case**: Older Raspberry Pi kernels, legacy systems

### kernel-6.6 Branch
- **Target Kernels**: 6.6.x+
- **Status**: Tested on 6.6.74+rpt-rpi-v8
- **Key Modifications**:
  - `nrc-mac80211-twt.c`: Updated `file_operations` function signatures
    - Changed read/write return types: `int` → `ssize_t`
    - Functions affected (10 total):
      - `nrc_mac_twt_info_read/write`
      - `nrc_mac_twt_schedule_read/write`
      - `nrc_mac_twt_dump_read/write`
      - `nrc_mac_twt_mon_read/write`
      - `nrc_mac_twt_debug_read/write`
  - **Reason**: Kernel 6.6 changed file_operations API for large file support

**Changes Summary**:
```diff
- static int nrc_mac_twt_info_read(struct file *file, char __user *user_buf, ...)
+ static ssize_t nrc_mac_twt_info_read(struct file *file, char __user *user_buf, ...)
```

## Troubleshooting

### Build Errors

**Error**: `kernel headers not found`
```bash
# Solution:
sudo apt-get install raspberrypi-kernel-headers
# or
sudo apt-get install linux-headers-$(uname -r)
```

**Error**: `incompatible-pointer-types` in nrc-mac80211-twt.c
```bash
# Solution: Wrong branch for your kernel version
# Check kernel: uname -r
# For 6.6.x: git checkout kernel-6.6
# For 4.x-5.x: git checkout master
```

**Error**: `make: command not found`
```bash
# Solution:
sudo apt-get install build-essential
```

### Runtime Errors

**Error**: `Unknown symbol` errors when loading driver
```bash
# Solution: Kernel version mismatch
# Rebuild driver with correct kernel headers:
make clean
sudo apt-get install linux-headers-$(uname -r)
make -j4
```

**Error**: wlan0 interface doesn't appear
```bash
# Check if driver loaded:
lsmod | grep nrc

# Check kernel logs:
dmesg | grep nrc

# Verify SPI hardware connection
```

## Quick Reference Commands

```bash
# One-line build (fresh start)
cd ~/nrc-linux-driver/nrc && make clean && make -j4

# One-line install and load
cd ~/nrc-linux-driver/nrc && sudo make modules_install && sudo insmod nrc.ko

# Check driver status
lsmod | grep nrc
dmesg | grep -i nrc
ip link show wlan0

# Unload driver
sudo rmmod nrc

# Update from GitHub
cd ~/nrc-linux-driver && git pull && cd nrc && make clean && make -j4
```

## Build Information

### Verified Configurations

| Kernel Version | Branch | Architecture | Status | Date Tested |
|----------------|--------|--------------|--------|-------------|
| 6.6.74+rpt-rpi-v8 | kernel-6.6 | ARM64 (aarch64) | ✅ Working | 2026-01-09 |
| 4.14.x | master | ARMv7 | ⚠️ Untested | - |
| 5.x.x | master | ARM64 | ⚠️ Untested | - |

## Additional Resources

- **GitHub Repository**: https://github.com/NEWRACOM-INC/nrc-linux-driver
- **OpenWrt Build**: https://github.com/NEWRACOM-INC/newracom
- **NRC MACSW**: Internal Gerrit repository

## Notes

- Build artifacts (*.o, *.ko) are generated in the `nrc/` directory
- Clean builds recommended when switching kernel versions
- Driver requires SPI hardware interface configured in device tree
- For production deployment, consider using `make modules_install`

## Support

For issues specific to:
- **Driver bugs**: Create GitHub issue at nrc-linux-driver repository
- **Build problems**: Check "Troubleshooting" section above
- **Hardware setup**: Refer to NRC HaLow module documentation
