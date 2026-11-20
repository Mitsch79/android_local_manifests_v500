# LineageOS 21 sources for LG G Pad 8.3 Wi-Fi

The local manifest pins the V500 device configuration, vendor graphics sources, 6.1 board port and upstream Android Common Kernel. These projects target the Wi-Fi LG-V500. The bring-up build uses a test ramdisk and separate ext4 images.

## Existing LineageOS 21 checkout

Install the device manifest and the temporary launcher override under `.repo/local_manifests/`.

```sh
mkdir -p .repo/local_manifests
curl -fL https://raw.githubusercontent.com/Mitsch79/android_local_manifests_v500/lineage-21.0/v500.xml -o .repo/local_manifests/v500.xml
curl -fL https://raw.githubusercontent.com/Mitsch79/android_local_manifests_v500/lineage-21.0/v500-trebuchet.xml -o .repo/local_manifests/v500-trebuchet.xml
repo sync device/lge/v500 vendor/lge/v500 kernel/lge/v500-port kernel/google/android14-6.1 packages/apps/Trebuchet
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

## Temporary Trebuchet backport

`v500-trebuchet.xml` replaces only `packages/apps/Trebuchet`. The fork is based on LineageOS 21 at `08002b5483685be3457908456afb42277eed4e08` and backports the [upstream unlock callback fix](https://github.com/LineageOS/android_packages_apps_Trebuchet/commit/a9355d4dba3f0bf93cb1a28eb738c6c05dc43ca9). A destroyed Quickstep service could otherwise register the recents input consumer a second time when its user unlocks.

When the selected LineageOS 21 revision contains an equivalent fix, update the platform pin, remove the include from `release/pinned.xml` and remove `v500-trebuchet.xml` from existing local manifest installations. Sync the launcher and verify first-time setup and HOME on reboot.
