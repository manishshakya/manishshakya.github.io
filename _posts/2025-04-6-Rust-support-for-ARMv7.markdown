---
layout: post
title: "Linux kernel: Rust support for ARMv7"
date: 2025-04-06 10:38:00
categories: linux boot-analysis
published: true
---

I tried adding Rust support for ARMv7. See: [https://lore.kernel.org/all/20250123-rfl-arm32-v3-1-8f13623d42c5@gmail.com/](https://lore.kernel.org/all/20250123-rfl-arm32-v3-1-8f13623d42c5@gmail.com/)

## Documentation

Read `Documentation/rust/quick-start.rst`.
Read `Documentation/rust/testing.rst`.

## Install Clang

```sh
sudo apt install llvm clang clang-tools lld
```

## Install rustup and components

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add rust-src
rustup target add armv7-unknown-linux-gnueabihf
cargo install bindgen-cli
```

## Patch kernel

```sh
wget https://lore.kernel.org/all/20250123-rfl-arm32-v3-1-8f13623d42c5@gmail.com/raw -O patch-email.txt
git am patch-email.txt
git log -n 1
```

## Build Kernel

Enable RUST support from the menuconfig

```sh
export ARCH=arm
export CROSS_COMPILE=armv7-unknown-linux-gnueabihf
make multi_v7_defconfig
make menuconfig
make LLVM=1 rustavailable
make LLVM=1
```

## Output

```sh
# cat /proc/cpuinfo
processor       : 0
model name      : ARMv7 Processor rev 2 (v7l)
BogoMIPS        : 298.84
Features        : half thumb fastmult vfp edsp thumbee neon vfpv3 tls vfpd32
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc08
CPU revision    : 2

Hardware        : Generic AM33XX (Flattened Device Tree)
Revision        : 0000
Serial          : 4014BBBK0948
/bin # zcat /proc/config.gz | grep RUST
CONFIG_RUSTC_VERSION=108600
CONFIG_RUST_IS_AVAILABLE=y
CONFIG_RUSTC_LLVM_VERSION=190107
CONFIG_RUST=y
CONFIG_RUSTC_VERSION_TEXT="rustc 1.86.0 (05f9846f8 2025-03-31)"
CONFIG_HAVE_RUST=y
# CONFIG_RUST_FW_LOADER_ABSTRACTIONS is not set
CONFIG_TRUSTED_FOUNDATIONS=y
# CONFIG_BLK_DEV_RUST_NULL is not set
# CONFIG_RUST_PHYLIB_ABSTRACTIONS is not set
# CONFIG_HID_THRUSTMASTER is not set
# CONFIG_TRUSTED_KEYS is not set
CONFIG_SYSTEM_TRUSTED_KEYRING=y
CONFIG_SYSTEM_TRUSTED_KEYS=""
# CONFIG_SECONDARY_TRUSTED_KEYRING is not set
CONFIG_RUST_DEBUG_ASSERTIONS=y
CONFIG_RUST_OVERFLOW_CHECKS=y
CONFIG_RUST_BUILD_ASSERT_ALLOW=y
/bin # cat /proc/cmdline
console=ttyS0,115200n8 quiet initcall_debug log_buf_len=10M
/bin # cat /proc/version
Linux version 6.13.9-00001-g0308da26480b (manish@manish-GL703GE) (Ubuntu clang version 14.0.0-1ubuntu1.1, Ubuntu LLD 14.0.0) #6 SMP Sun Apr  6 08:20:43 EDT 2025
```
