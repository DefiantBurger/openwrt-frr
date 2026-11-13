# openwrt-frr

OpenWrt built from source with the full [FRR](https://frrouting.org/) routing suite, for the **Banana Pi BPI-R2 Pro** (Rockchip RK3568, target `rockchip/armv8`).

Prebuilt images are on the [Releases](../../releases) page.

## What's in the image

- The latest stable OpenWrt release, plus the LuCI web interface
- Every FRR daemon: bgpd, ospfd/ospf6d, isisd, ldpd, pimd/pim6d, babeld, bfdd, and the rest
- The kernel support FRR needs for MPLS, VRF, GRE and multicast routing

The full package selection is in [config/bpi-r2-pro.config](config/bpi-r2-pro.config).

## Building

Go to **Actions → Build OpenWrt + FRR (BPI-R2 Pro) → Run workflow**. You can set two inputs:

- `openwrt_version`: an OpenWrt tag such as `v25.12.6`. Leave it empty for the latest stable release.
- `publish_release`: turn this off to keep the build as a workflow artifact without publishing a release.

The first build takes about 1.5 hours on a GitHub runner. Later builds reuse cached downloads and compiler output.

## Flashing

To install fresh, flash `*-sysupgrade.img.gz` to an SD card with [balenaEtcher](https://etcher.balena.io/). Etcher reads the `.gz` directly, so you don't need to unzip it first.

To upgrade an existing OpenWrt install instead, use `sysupgrade`. Either way, check the image against `sha256sums` first.
