Open Source Kernel for Xiaomi 12 and Xiaomi 12 Pro
=========================================

The Xiaomi 12 and Xiaomi 12 Pro were released at the 31st December 2021.
Fortunately Xiaomi has released the kernel sources with the release of the devices but unfortunately these released sources are incomplete. This Project aims at providing the missing sources and enables developers and enthusiats to get an easy start to compile a kernel for the Xiaomi 12 and/or Xiaomi 12 Pro.

## How to build the kernel
1. Intitalize your local repository using this manifest:
```
repo sync --force-sync kernel_platform/build
```
2. Then to sync up:
```
repo sync
```
3. Run the kernel build for your device. E.g for cupid:
```
./build.sh cupid
```
4. Find the build artifacts in `device/qcom/cupid-kernel/`. This includes the kernel (Image), the kernel modules, dtb and dtbo, kernel headers and some host utilities.

## What is missing in the official sources?
The official kernel sources insist of the following repositories:
- [Xiaomi_Kernel_Opensource](https://github.com/MiCode/Xiaomi_Kernel_OpenSource/commits/zeus-s-oss)
This repository contains xiaomis fork of the msm-kernel and the camera-kernel and display-drivers techpack repositories.
- [vendor_qcom_opensource_audio-kernel](https://github.com/MiCode/vendor_qcom_opensource_audio-kernel/tree/zeus-s-oss)
This repository contains xiaomis fork of the qualcomm audio-kernel-ar repository.
- [vendor_qcom_opensource_wlan](https://github.com/MiCode/vendor_qcom_opensource_wlan/tree/zeus-s-oss)
This repository contains xiaomis forks of the qualcomm qcacld-3.0, qca-wifi-host-cmn, fw-api and sigma-dut repositories.
- [kernel_devicetree](https://github.com/MiCode/kernel_devicetree/tree/zeus-s-oss)
This repository contains xiaomis forks of the proprietary qualcomm devicetree, camera-devicetree and display-devicetree.

Starting with the msm-kernel xiaomi has added their own hwid driver, which is used in kernel to differentiate between different xiaomi devices, to the .gitingore file in the repository and hence the whole source code of this driver is missing. [reference](https://github.com/MiCode/Xiaomi_Kernel_OpenSource/blob/7d10d738b2c4299f3b8549c61a01ece39f19c788/.gitignore#L169)
Xiaomi is using an own driver for their touchfeature implementation and they are using a custom fts_spi kernel driver for the touchscreen but the inclusion of these drivers is commented. This indicates that xiaomi has moved these drivers to an external repository which they did not provide the source for. Both drivers source code is present in the kernel tree (although it's not being used due to the commented out inclusion) but unfortunately it is not the source being used on MIUI releases. The xiaomi touch driver on the [oldest miui i could find](https://bigota.d.miui.com/V13.0.12.0.SLCCNXM/cupid_images_V13.0.12.0.SLCCNXM_20211229.0000.00_12.0_cn_6c8fd3679d.tgz) is already significantly ahead of the public drivers state (E.g. /sys/devices/virtual/touch/touch_dev/fod_press_status is provided by the xiaomi touch driver on MIUI but the public driver does not provide a similar functionality). The FTS touchscreen driver is also behind the one shipped on MIUI and lacks some basic functionality (E.g. double tap to wake is broken). [reference](https://github.com/MiCode/Xiaomi_Kernel_OpenSource/blob/7d10d738b2c4299f3b8549c61a01ece39f19c788/drivers/input/touchscreen/Makefile#L118-L119)

Xiaomi does not provide the source code of many used qualcomm external kernel module repositories. Namely these are: [mmrm-driver](https://git.codelinaro.org/clo/la/platform/vendor/opensource/mmrm-driver), [mmrm-driver-test](https://git.codelinaro.org/clo/la/platform/vendor/opensource/mmrm-driver-test), [video-driver](https://git.codelinaro.org/clo/la/platform/vendor/opensource/video-driver), [video-driver-test](https://git.codelinaro.org/clo/la/platform/vendor/opensource/video-driver-test), [cvp-kernel](https://git.codelinaro.org/clo/la/platform/vendor/opensource/cvp-kernel), [dataipa](https://git.codelinaro.org/clo/la/platform/vendor/opensource/dataipa), [eva-kernel](https://git.codelinaro.org/clo/la/platform/vendor/opensource/eva-kernel), [touch-drivers](https://git.codelinaro.org/clo/la/platform/vendor/opensource/touch-drivers), [datarmnet](https://git.codelinaro.org/clo/la/platform/vendor/qcom/opensource/datarmnet) and [datarmnet-ext](https://git.codelinaro.org/clo/la/platform/vendor/qcom/opensource/datarmnet-ext).
These repositories might not have been changed by xiaomi but since their kernel is based on a pre-public caf tag one has to guess the used revision of all these repositories.

Xiaomi does not provide the source code of the following kernel modules which are not publicised by qualcomm: binder_prio.ko, sla.ko. It does not seem like they have a meaningful function.

Xiaomis proviced devicetree is incomplete and does not work because it misses the video, mmrm and audio devicetree techpacks. These techpack devicetrees are not publicised by qualcomm which makes it harder to get this working.

## Fixes for the mentioned issues

### The hwid driver
Xiaomi has used a similar hwid driver on older device generations already, so it was possible to import an older driver and update the defines of the internal codenames for Xiaomi 12 and Xiaomi 12 Pro. [commit](https://github.com/xiaomi-sm8450-kernel/android_kernel_platform_msm-kernel/commit/17deedc1d673d707672cb426036623f8f2216262)

### The touchscreen drivers
The public fts and xiaomi touch drivers can be used and the basic functionality (E.g. touchscreen input) works fine. However double tap to wake and the under display fingerprint sensor do not work on MIUI with these drivers. Double tap to wake is to be fixed and the under display fingerprint sensor can be fixed with a custom implementation on custom ROMs or extensive reversing of the MIUI module.

### The missing external kernel module repositories
As mentioned above xiaomi might not have touched these modules so i was able to guess revisions of the modules and include the repositories from qualcomm and it seems to work fine.

### The mysterious binder_prio and sla kernel modules
The system seems to run fine without them.

### The missing devicetree techpacks
Fortunately ASUS has properly released these sources for their Snapdragon 8G1 devices ([link](https://dlcdnets.asus.com/pub/ASUS/ZenFone/AI2201/ASUS_AI2201-32.2810.2205.63-kernel-src.tar.gz)) and i was able to use them as a base to apply xiaomis changes on top of. The video and mmrm devicetrees do not have any changes and i have reversed their audio devicetree changes for Xiaomi 12 and Xiaomi 12 Pro. [commit](https://github.com/xiaomi-sm8450-kernel/android_vendor_qcom_proprietary_audio-devicetree/commit/b44fdf9e6568219cca4a04a5f39bda495088f4dc)

## Other issues
On qualcomm 5.10 devices the devicetree is built from multiple techpack repositories as multiple dtb(o)s which will be merged later. Xiaomi has introduced the `xiaomi,miboard-id` property to mark split devicetrees which are (in)compatible to merge. The public qualcomm script to merge dtbs does not respect this property and merges incompatible split devicetrees together which results in a broken devictree. This can be fixed by respecting the miboard-id in the merge dts script. [commit](https://github.com/xiaomi-sm8450-kernel/android_kernel_platform_build/commit/8488cc7b670607aa652310cf8eac6a92047b77f1)
