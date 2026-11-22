# LineageOS 21 sources for LG G Pad 8.3 Wi-Fi

The local manifest pins the V500 device configuration, vendor sources, 6.1 board port and upstream Android Common Kernel. It also selects the V500 forks of SetupWizard, SetupCompat and Settings at their normal Android source paths. The bring-up build uses a test ramdisk and separate ext4 images.

## Existing LineageOS 21 checkout

Install the device manifest and the temporary launcher override under `.repo/local_manifests/`.

```sh
mkdir -p .repo/local_manifests
curl -fL https://raw.githubusercontent.com/Mitsch79/android_local_manifests_v500/lineage-21.0/v500.xml -o .repo/local_manifests/v500.xml
curl -fL https://raw.githubusercontent.com/Mitsch79/android_local_manifests_v500/lineage-21.0/v500-trebuchet.xml -o .repo/local_manifests/v500-trebuchet.xml
repo sync device/lge/v500 vendor/lge/v500 kernel/lge/v500-port kernel/google/android14-6.1 packages/apps/Trebuchet packages/apps/SetupWizard external/setupcompat packages/apps/Settings
```

Use a clean checkout or back up existing local sources in those paths before syncing.

## V500 setup wizard

The wizard loads languages and time zones asynchronously, preserves navigation history and remembers selections when users revisit a page. SetupCompat carries the wizard brightness into the delegated lock screen and provides visible button feedback. The Settings fork lets users open developer options on builds without a Bluetooth adapter.

## Fresh checkout with pinned platform revisions

The V500 setup wizard can reuse OpenGL programs compiled on a training tablet with the same HWUI and Mesa build. To include them in the system image, follow the [device shader training instructions](https://github.com/Mitsch79/android_device_lge_v500/tree/lineage-21.0#precompile-the-setup-wizards-graphics-programs).

```sh
repo init -u https://github.com/Mitsch79/android_local_manifests_v500 -b lineage-21.0 -m release/pinned.xml --git-lfs
repo sync -c
bash vendor/lge/v500/prepare-v500-vendor.sh "$PWD"
source build/envsetup.sh
lunch lineage_v500-ap2a-userdebug
m systemimage vendorimage ramdisk
```

The [kernel branch](https://github.com/Mitsch79/android_kernel_lge_v500/tree/lineage-21.0) documents the additional boot-image inputs, toolchain and kernel build. Firmware and the device-specific reference boot image must be supplied locally.

### Verify the M6 graphics profile

In the same shell used for `lunch`, check the vendor properties in the built product.

```sh
grep -E '^(ro\.hardware\.(egl|gralloc)|vendor\.hwc\.backend_override|vendor\.minigbm\.debug|ro\.sf\.lcd_density)=' "$ANDROID_PRODUCT_OUT/vendor/build.prop"
```

The native M6 profile selects Mesa, `minigbm_msm`, the `client` backend, linear scanout buffers (`nocompression`) and 280 logical dpi. An enabled `.v500-half-client` marker in the device checkout selects a different build profile. For a graphics comparison, keep the same system, vendor and boot inputs except for the component under test.

On the selected tablet, `adb shell dumpsys SurfaceFlinger --comp-displays` reports the framebuffer dimensions and the validated composition type of each visible layer. Check these in the scene under test. To time a transition inside the shared setup window, match the target page's draw token to its presented display frame. [Android FrameTimeline](https://perfetto.dev/docs/data-sources/frametimeline)

## Temporary Trebuchet backport

`v500-trebuchet.xml` replaces only `packages/apps/Trebuchet`. The fork is based on LineageOS 21 at `08002b5483685be3457908456afb42277eed4e08` and backports the [upstream unlock callback fix](https://github.com/LineageOS/android_packages_apps_Trebuchet/commit/a9355d4dba3f0bf93cb1a28eb738c6c05dc43ca9). A destroyed Quickstep service could otherwise register the recents input consumer a second time when its user unlocks.

When the selected LineageOS 21 revision contains an equivalent fix, update the platform pin, remove the include from `release/pinned.xml` and remove `v500-trebuchet.xml` from existing local manifest installations. Sync the launcher and verify first-time setup and HOME on reboot.
