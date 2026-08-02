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

headers_install:
	@mkdir -p $(O)/usr/include
	@rsync -mrq $(shell pwd)/* $(O)/usr/include

all:
	@true
