# Build Information - NRC HaLow Linux Driver

## Successful Builds

### Kernel 6.6.74+rpt-rpi-v8 (Raspberry Pi 4)

**Date**: 2026-01-09
**Branch**: `kernel-6.6`
**Architecture**: ARM64 (aarch64)
**Build Location**: Raspberry Pi 4 (192.168.10.97)
**Module Size**: 485 KB
**Status**: ✅ Successfully built and loaded

**Key Changes for Kernel 6.6**:
- Updated `file_operations` read/write function signatures from `int` to `ssize_t`
- Fixed `nrc-mac80211-twt.c` debugfs operations compatibility

**Test Results**:
- Driver loads without errors (`insmod nrc.ko`)
- wlan0 interface appears correctly
- No "Unknown symbol" errors
- SPI driver registered successfully

**Build Command**:
```bash
cd ~/nrc-linux-driver/nrc
make -j4
```

**Installation**:
```bash
sudo cp ~/nrc-linux-driver/nrc/nrc.ko ~/nrc_pkg/sw/driver/
sudo insmod ~/nrc_pkg/sw/driver/nrc.ko
```

**Verification**:
```bash
dmesg | grep nrc80211  # Should show "Succeed to register spi driver"
ip link show wlan0      # Should show wlan0 interface
```

## Branch Strategy

- **`master`**: Original driver source from NRC_MACSW
- **`kernel-6.6`**: Kernel 6.6+ specific modifications
- Additional kernel version branches can be created as needed

## Known Issues

None reported for kernel 6.6.74+rpt-rpi-v8

## Next Steps

- Test with actual HaLow hardware
- Verify sniffer mode functionality
- Test monitor mode and packet capture
