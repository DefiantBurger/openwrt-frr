# openwrt-frr

OpenWrt built from source with the full [FRR](https://frrouting.org/) routing suite, for the **Banana Pi BPI-R2 Pro** (Rockchip RK3568, target `rockchip/armv8`).

Prebuilt images are on the [Releases](../../releases) page.

## What's in the image

- The latest stable OpenWrt release, plus the LuCI web interface
- Every FRR daemon: bgpd, ospfd/ospf6d, isisd, ldpd, pimd/pim6d, babeld, bfdd, and the rest
- The kernel support FRR needs for MPLS, VRF, GRE and multicast routing

The full package selection is in [config/bpi-r2-pro.config](config/bpi-r2-pro.config).

## Network setup

Each build produces two images, one for each end of the radio link. They come up routing with OSPF on first boot, with no setup needed:

```
base computers ── base-router ══ antennas A & B ══ robot-router ── robot computers
```

| Port | Use | `base-*` image | `robot-*` image |
|---|---|---|---|
| lan0, lan1 | Computers (DHCP served) | 10.0.1.1/24 | 10.0.2.1/24 |
| lan2 | Antenna link A | 10.0.101.1/24 | 10.0.101.2/24 |
| lan3 | Antenna link B | 10.0.102.1/24 | 10.0.102.2/24 |
| WAN | Unused | | |

- **OSPF** shares each side's computer subnet with the other router, so computers on either side can reach each other.
- **BFD** checks each antenna link every 300 ms. If a link stops responding, traffic moves to the other link in about 1.5 s.
- **Both links working:** they share traffic. Each connection sticks to one link.
- **Antennas:** they must be in transparent bridge mode. If you give them management IPs, put them in their link's subnet, e.g. 10.0.101.10 and 10.0.101.11.

The configuration is in [files/](files/):
- `common/` goes into both images. Its first-boot script sets up IPs, interfaces and the firewall.
- `base/` and `robot/` hold each router's FRR config and role.

To check that routing works, run these on either router:

```sh
vtysh -c 'show ip ospf neighbor'   # two neighbors (one per link), state Full
vtysh -c 'show bfd peers brief'    # two peers, status up
vtysh -c 'show ip route ospf'      # the other side's 10.0.x.0/24
```

## Building

Go to **Actions → Build OpenWrt + FRR (BPI-R2 Pro) → Run workflow**. You can set two inputs:

- `openwrt_version`: an OpenWrt tag such as `v25.12.6`. Leave it empty for the latest stable release.
- `publish_release`: turn this off to keep the build as a workflow artifact without publishing a release.

The first build takes about 1.5 hours on a GitHub runner. Later builds reuse cached downloads and compiler output.

## Flashing

Use `base-*` on the base station router and `robot-*` on the robot router.

To install fresh, flash the `sysupgrade.img.gz` to an SD card with [balenaEtcher](https://etcher.balena.io/). Etcher reads the `.gz` directly, so you don't need to unzip it first.

To upgrade an existing OpenWrt install instead, use `sysupgrade`. It keeps your settings, including any FRR changes saved with `write memory`. If you're switching a board to the other role, use `sysupgrade -n` so it starts fresh.

Either way, check the image against `sha256sums` first.
