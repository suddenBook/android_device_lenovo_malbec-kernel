# SPDX-FileCopyrightText: The LineageOS Project
# SPDX-License-Identifier: Apache-2.0
#
# Stub kernel Makefile.
#
# malbec ships a stock Google GKI image and builds no kernel, but 69 modules in
# this product still compile against kernel UAPI headers -- the whole QTI
# display stack (libsdmcore, libsdedrm, libdrmutils, the composer service,
# gralloc) and parts of audio and IPA. vendor/lineage/build/soong's
# generated_kernel_includes runs
#
#     make -C $(TARGET_KERNEL_SOURCE) O=<gen> ARCH=arm64 headers_install
#
# unconditionally, so TARGET_KERNEL_SOURCE has to point at something that
# answers that target. This directory is that something: a pre-extracted UAPI
# header set plus this Makefile, which is the same arrangement the onyx kernel
# repo uses.
#
# VERSION/PATCHLEVEL are read by vendor/lineage/config/BoardConfigKernel.mk to
# derive TARGET_KERNEL_VERSION. 6.6 matches the running kernel
# (6.6.87-android15-8-gc2569c3b141c-ab13768703-4k).
#
# Provenance of the headers:
#
#   asm/ asm-generic/ drm/ linux/ misc/ mtd/ rdma/ scsi/ sound/ video/ xen/
#       `make ARCH=arm64 headers_install` on the LineageOS sm8750 msm-kernel
#       (6.6.139, BRANCH=android15-6.6, KMI_GENERATION=8).
#
#   audio/    qcom/opensource/audio-kernel/include/uapi/audio/
#   display/  qcom/opensource/display-drivers/include/uapi/display/
#       QTI-private UAPI, which is not part of the ACK tree. Same source repo.
#
#   linux/qti-smmu-proxy.h        securemsm-kernel/smmu-proxy/include/uapi/
#   linux/msm_ipa.h               dataipa/drivers/platform/msm/include/uapi/
#   linux/ipa_qmi_service_v01.h   likewise
#   linux/rmnet_ipa_fd_ioctl.h    likewise
#   linux/nfc/nfcinfo.h           nxp/opensource/driver/include/uapi/
#   linux/nfc/sn_uapi.h           likewise
#       Individual QTI/NXP-private headers that live in driver repos rather than
#       in either uapi tree above. qti-smmu-proxy.h is the load-bearing one:
#       libgralloccore is compiled with -DTARGET_USES_SMMU_PROXY and gr_dma_mgr.h
#       includes it unconditionally.
#
# The set was cross-checked against onyx's kernel-headers, which is the same
# arrangement for the same platform family. The only header onyx has that this
# one deliberately omits is display/drm/mi_disp.h, which is Xiaomi-specific.
#
# The device runs 6.6.87-android15-8; the headers come from 6.6.139-android15-8.
# Same branch, same KMI generation, and a frozen KMI generation is exactly the
# window inside which the kernel<->userspace interface is guaranteed not to
# change -- only LTS backports land. Do not substitute headers from a different
# android1X-6.Y branch or a different KMI generation.

VERSION = 6
PATCHLEVEL = 6

# ★ $(O) MUST BE ABSOLUTE, and this target refuses to run if it is not.
#
# `make -C <dir>` runs with the cwd set to <dir> -- i.e. to THIS directory. A
# relative $(O) is therefore resolved against the source tree, not the build
# top, and the whole UAPI header set is silently written to
#
#     device/lenovo/malbec-kernel/kernel-headers/<O>/usr/include/
#
# while the real genDir stays empty. Nothing fails at that moment. The build
# runs on for another hour and dies at 74% with
#
#     hardware/qcom-caf/sm8750/dataipa/ipanat/inc/ipa_nat_utils.h:71:10:
#         fatal error: 'linux/msm_ipa.h' file not found
#
# which names a file nobody edited, in a module nobody touched, and points at
# nothing. Session 30 lost a 1h13m build to exactly this and then had to delete
# 1003 stray headers out of this directory -- where, left in place, the `rsync`
# below would have copied them into the next build's genDir as well.
#
# Whether $(O) arrives absolute is decided by ONE line of upstream LineageOS,
# vendor/lineage/config/BoardConfigKernel.mk:273-276:
#
#     OUT_DIR_PREFIX := $(shell echo $(OUT_DIR) | sed -e 's|/target/.*$$||g')
#     KERNEL_BUILD_OUT_PREFIX :=
#     ifeq ($(OUT_DIR_PREFIX),out)
#         KERNEL_BUILD_OUT_PREFIX := $(BUILD_TOP)/
#     endif
#
# It prepends $(BUILD_TOP)/ only when OUT_DIR is literally `out`, on the
# assumption that any other OUT_DIR was given as an absolute path. A RELATIVE
# OUT_DIR that is not `out` -- `OUT_DIR=out-final-r1`, the obvious way to keep a
# release build beside a working one -- satisfies neither branch and produces a
# relative $(O). Upstream would be correct as
# `ifeq ($(filter /%,$(OUT_DIR)),)`, but this tree does not patch upstream, so
# the invariant is enforced here and in work/scripts/40-build.sh instead.
#
# The other half of the trap: OUT_DIR cannot simply be made absolute to dodge
# this, because Soong then rejects platform_testing/Android.bp:255
# ("continuous_native_tests ... Path is outside directory"). `out` is the only
# value that satisfies both. 40-build.sh pins it; this is the backstop.
headers_install:
	@case '$(O)' in \
	    /*) ;; \
	    *) echo "" >&2; \
	       echo "!! kernel-headers: O='$(O)' is RELATIVE and must be absolute." >&2; \
	       echo "!! Building with a relative OUT_DIR other than 'out' does this." >&2; \
	       echo "!! Fix: build via work/scripts/40-build.sh, or set OUT_DIR=out." >&2; \
	       echo "!! (see the comment above this target for the whole story)" >&2; \
	       echo "" >&2; \
	       exit 1 ;; \
	esac
	@mkdir -p $(O)/usr/include
	@rsync -mrq --exclude='out' --exclude='out-*' $(shell pwd)/* $(O)/usr/include

all:
	@true
