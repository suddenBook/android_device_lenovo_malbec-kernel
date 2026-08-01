# Kernel prebuilts for Lenovo Idea Tab Pro Gen 2 (malbec)

Prebuilt kernel, device tree blobs and kernel modules for `malbec`
(Lenovo TB390FU, Qualcomm SM8735P).

## SoC

```
soc_id      694
machine     TUNAP
family      Snapdragon
revision    1.1
hw_platform QRD
```

The silicon codename is **TunaP**; the matching upstream device tree source is
`qcom/tunap.dts` + `qcom/tunap.dtsi` (`qcom,msm-id = <694 0x10000>`). The kernel
build target and HAL platform family are both `sun` — that is a separate name and
both are correct.

## Kernel

An **unmodified Google GKI 2.0 image**:

```
6.6.87-android15-8-gc2569c3b141c-ab13768703-4k
  branch     android15-6.6
  KMI gen    8
  page size  4 KB
  builder    kleaf@build-host (ci.android.com build ab13768703)
```

Nothing is built from source. All hardware support arrives as loadable modules,
so the stock image is reused verbatim and only the modules are packaged.

## Contents

```
images/
├── kernel        GKI Image (35 MB)
├── dtbo.img      stock dtbo, used via BOARD_PREBUILT_DTBOIMAGE (48 MB)
└── dtbs/         19 device tree blobs
modules/
├── vendor_dlkm/  300 .ko + modules.load + modules.blocklist
├── vendor_boot/  322 .ko + modules.load + modules.load.recovery + modules.blocklist
└── system_dlkm/   96 .ko + modules.load
```

### Why 19 device tree blobs

The stock `vendor_boot` carries a multi-SoC device tree covering three families,
because Lenovo ships one image across several SKUs — the same reason
`device/qcom/malbec/` contains `manifest_kera.xml`, `manifest_sun.xml` and
`manifest_tuna.xml`:

| Family | `qcom,msm-id` |
|---|---|
| Kera | 659, 686, 720, 721, 731, 732 |
| Sun | 618, 639, 705, 706 (plus alternate thermal profiles) |
| **Tuna** | 655, 681, **694 ← this device (TunaP)** |

The bootloader selects by runtime `soc_id`. All 19 are kept rather than just
`19_dtbdump_..._TunaP_SoC.dtb`, so any other malbec SKU still boots. They cost
8 MB in a 96 MB partition.

### Module layout

Unlike most SM8750/SM8735 devices, `system_dlkm` and `vendor_dlkm` keep their
modules **flat** under `lib/modules`, not under `lib/modules/$(KERNEL_RELEASE)`.
Verified against both the factory image and a running device. `BoardConfig.mk`
copies them flat to match; using the versioned layout would leave modprobe
unable to find them at first stage.

`vendor_dlkm/modules.load` has 560 lines for 300 modules — 199 appear once, 57
twice, 5 three times, one four times and 38 six times. That is how the stock file
ships; the repeats come from modules being pulled in through several dependency
chains, and modprobe skips anything already loaded. It is left untouched.

## Source availability

About 93 % of the vendor modules have public sources, spread across the Qualcomm
`vendor/qcom/opensource/*` packages (audio-kernel, camera-kernel,
display-drivers, graphics-kernel, touch-drivers, wlan, …) and the msm-kernel
tree. The following have none and are used as prebuilts:

```
lenovo_keyboard.ko        hall_sensor.ko
lenovo_sys_temp.ko        lenovo_thermal_control.ko
afw.ko                    aw882xx_dlkm.ko
ps5169.ko                 r8125.ko
extend_reclaim.ko         oem_bm_adsp_ulog.ko
nvt_touch.ko              qca_cld3_*.ko
```

## References

- [`LineageOS/android_kernel_qcom_sm8750-devicetrees`](https://github.com/LineageOS/android_kernel_qcom_sm8750-devicetrees) — carries `qcom/tunap.dts` and `qcom/tunap.dtsi`
- [`LineageOS/android_kernel_qcom_sm8750-modules`](https://github.com/LineageOS/android_kernel_qcom_sm8750-modules)
- [`oppo-source/android_kernel_modules_and_devicetree_oppo_sm8735`](https://github.com/oppo-source/android_kernel_modules_and_devicetree_oppo_sm8735)
