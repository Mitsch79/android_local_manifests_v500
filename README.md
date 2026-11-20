# LineageOS 21 sources for LG G Pad 8.3 Wi-Fi

The local manifest pins the V500 device configuration, vendor graphics sources, 6.1 board port and upstream Android Common Kernel. These projects target the Wi-Fi LG-V500. The bring-up build uses a test ramdisk and separate ext4 images.

## Existing LineageOS 21 checkout

Install `v500.xml` under `.repo/local_manifests/`. To get just this file while preserving any existing local manifests:

```sh
mkdir -p .repo/local_manifests
curl -fL https://raw.githubusercontent.com/Mitsch79/android_local_manifests_v500/lineage-21.0/v500.xml -o .repo/local_manifests/v500.xml
repo sync device/lge/v500 vendor/lge/v500 kernel/lge/v500-port kernel/google/android14-6.1
```

Use a clean checkout or back up existing local sources in those paths before syncing.

## Fresh checkout with pinned platform revisions

```sh
repo init -u https://github.com/Mitsch79/android_local_manifests_v500 -b lineage-21.0 -m release/pinned.xml --git-lfs
repo sync -c
bash vendor/lge/v500/prepare-v500-vendor.sh "$PWD"
source build/envsetup.sh
lunch lineage_v500-ap2a-userdebug
m systemimage vendorimage ramdisk
```

The [kernel branch](https://github.com/Mitsch79/android_kernel_lge_v500/tree/lineage-21.0) documents the additional boot-image inputs, toolchain and kernel build. Firmware and the device-specific reference boot image must be supplied locally.
