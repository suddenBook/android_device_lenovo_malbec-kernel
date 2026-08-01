# Kernel prebuilts for Lenovo Idea Tab Pro Gen 2 (malbec)

Prebuilt kernel and kernel modules for `malbec` (Lenovo TB390FU, Qualcomm SM8735P / Kera).

## What this contains

The device ships an **unmodified Google GKI 2.0 kernel**:

```
6.6.87-android15-8-gc2569c3b141c-ab13768703-4k
  branch     android15-6.6
  KMI gen    8
  page size  4 KB
  builder    kleaf@build-host (ci.android.com build ab13768703)
```

All hardware support is provided as loadable modules rather than being built into
the kernel image, so the stock kernel is reused as-is and only the modules need to
be packaged:

| Location | Modules |
|---|---|
| `vendor_dlkm` | 300 |
| `system_dlkm` | 96 |
| `vendor_boot` ramdisk (first stage) | 322 (115 loaded via `modules.load`) |

## Source availability

Roughly 93 % of the vendor modules have public sources, distributed across the
Qualcomm `vendor/qcom/opensource/*` module packages (audio-kernel, camera-kernel,
display-drivers, graphics-kernel, touch-drivers, wlan, …) and the msm-kernel tree.

The following have no public source and are used as prebuilts:

```
lenovo_keyboard.ko        hall_sensor.ko
lenovo_sys_temp.ko        lenovo_thermal_control.ko
afw.ko                    aw882xx_dlkm.ko
ps5169.ko                 r8125.ko
extend_reclaim.ko         oem_bm_adsp_ulog.ko
nvt_touch.ko              qca_cld3_*.ko
```

## References

- Device tree for this SoC family: [`LineageOS/android_kernel_qcom_sm8750-devicetrees`](https://github.com/LineageOS/android_kernel_qcom_sm8750-devicetrees) — contains `qcom/kera-iot.dts` (`qcom,msm-id = <731 0x10000>`), which matches this device's DTB exactly
- Vendor modules: [`LineageOS/android_kernel_qcom_sm8750-modules`](https://github.com/LineageOS/android_kernel_qcom_sm8750-modules)
- Closest production SM8735 tree: [`oppo-source/android_kernel_modules_and_devicetree_oppo_sm8735`](https://github.com/oppo-source/android_kernel_modules_and_devicetree_oppo_sm8735)
