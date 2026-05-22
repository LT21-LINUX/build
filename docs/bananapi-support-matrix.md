# Banana Pi support matrix

Last updated: 2026-05-22

This file tracks Banana Pi boards that are present in Banana Pi official
documentation or BPI-SINOVOIP BSP repositories, and whether this Armbian tree has
enough support to build release images.

Status values:

- `supported`: board config exists and is part of the normal Banana Pi image set.
- `bpi-added`: carried by the BPI release branch, but not necessarily present in
  upstream Armbian.
- `candidate-mainline`: kernel and U-Boot evidence exists; add Armbian board
  integration and validate on hardware.
- `candidate-legacy`: official/vendor BSP exists; integrate as a legacy Armbian
  family or branch before release use.
- `blocked`: do not publish images until bootloader, kernel, or hardware
  validation gaps are closed.

## Already tracked in this tree

| Board | Config | Family | Notes |
| --- | --- | --- | --- |
| Banana Pi M1 | `bananapi.conf` | `sunxi` | Active. |
| Banana Pi M1 Plus | `bananapim1plus.csc` | `sunxi` | Community support. |
| Banana Pi M2 Plus | `bananapim2plus.conf` | `sunxi` | Active. |
| Banana Pi M2 Ultra | `bananapim2ultra.csc` | `sunxi` | Community support. |
| Banana Pi M2 Zero | `bananapim2zero.csc` | `sunxi` | Community support. |
| Banana Pi M3 | `bananapim3.csc` | `sunxi` | Community support. |
| Banana Pi M64 | `bananapim64.csc` | `sunxi64` | Community support. |
| Banana Pi M2S | `bananapim2s.conf` | `meson64` | Active. |
| Banana Pi CM4IO | `bananapicm4io.conf` | `meson64` | Active; keep CM4 eMMC tuning separate from new bring-up work. |
| Banana Pi M2 Pro | `bananapim2pro.conf` | `meson64` | Active. |
| Banana Pi M5 | `bananapim5.conf` | `meson64` | Active. |
| Banana Pi M4 Berry | `bananapim4berry.conf` | `sunxi64` | Active. |
| Banana Pi M4 Zero | `bananapim4zero.conf` | `sunxi64` | Active. |
| Banana Pi M5 Pro | `bananapim5pro.conf` | `rockchip64` | Active. |
| Banana Pi M7 | `bananapim7.conf` | `rockchip64` | Active. |
| Banana Pi F3 | `bananapif3.conf` | `spacemit` | Active; RISC-V OS support matrix differs from ARM boards. |
| Banana Pi R2 | `bananapir2.csc` | `mt7623` | Community support. |
| Banana Pi R2 Pro | `bananapir2pro.csc` | `rockchip64` | Community support. |
| Banana Pi R4 | `bananapir4.csc` | `filogic` | Community support; useful reference for new Filogic boards. |
| Banana Pi Pro | `bananapipro.csc` | `sunxi` | Community support. |
| Lamobo R1 | `lamobo-r1.eos` | `sunxi` | End of support. |

## BPI branch additions to keep synchronized

| Board | Status | Source evidence | Next action |
| --- | --- | --- | --- |
| Banana Pi M2 | `bpi-added` | Restored from `.eos` to `.csc` in the BPI release branch. | Keep in BPI release set after server/desktop boot verification. |
| Banana Pi M2 Berry | `bpi-added` | Mainline/U-Boot `bananapi_m2_berry_defconfig` exists in cached U-Boot sources. | Keep build coverage for Debian 12/13 and Ubuntu 22.04/24.04/26.04. |
| Banana Pi M2 Magic | `bpi-added` | Added in BPI branch as `bananapim2magic.csc`. | Keep as community support unless hardware validation is repeated. |
| Banana Pi P2 Zero | `bpi-added` | Added through BPI kernel/U-Boot patches in the BPI branch. | Keep as community support; verify SD boot and Ethernet/Wi-Fi if available. |
| Banana Pi 6204 | `bpi-added` | Local board config and R40/V40 patches are present in this worktree. | Keep separate from upstream sync until boot logs are archived. |

## Missing or incomplete Banana Pi boards

| Board | SoC / family | Status | Evidence | Integration plan |
| --- | --- | --- | --- | --- |
| Banana Pi R3 | MediaTek MT7986A / `filogic` | `candidate-mainline` | U-Boot `mt7986a_bpir3_sd_defconfig`, `mt7986a_bpir3_emmc_defconfig`; kernel `mt7986a-bananapi-bpi-r3.dts` and storage overlays exist in cached sources. | Extend `filogic` for MT7986 ATF/FIP layout, then add SD and eMMC board configs as `.wip`. |
| Banana Pi R3 Mini | MediaTek MT7986A / `filogic` | `candidate-mainline` | Kernel and U-Boot DTS evidence exists as `mt7986a-bananapi-bpi-r3-mini.dts`. | Reuse the R3 family work; add a board config only after serial boot is checked. |
| Banana Pi R64 | MediaTek MT7622 | `candidate-mainline` | Kernel and U-Boot DTS evidence exists as `mt7622-bananapi-bpi-r64.dts`; official BSP repos include `BPI-R64-bsp-5.4`. | Decide whether to extend `mt7623` or create a small MT7622 family; validate SD/eMMC boot flow first. |
| Banana Pi P2 Pro | Rockchip RK3308 / `rockchip64` | `candidate-mainline` | Kernel and U-Boot DTS evidence exists as `rk3308-bpi-p2-pro.dts`; `.wip` board config and U-Boot defconfig are now staged in this tree. | Build one server image, then test serial console, eMMC, Ethernet, and Wi-Fi before moving from `.wip` to `.csc`. |
| Banana Pi M4 RTD1395 | Realtek RTD1395 | `candidate-mainline` / `candidate-legacy` | Mainline DTS evidence exists as `rtd1395-bpi-m4.dts`; official BSP repo is `BPI-M4-bsp` with kernel 4.9. | Start with legacy BSP only if mainline boot chain is incomplete; do not publish desktop until display and storage are verified. |
| Banana Pi W2 | Realtek RTD1296 | `candidate-legacy` | Official BSP repo is `BPI-W2-bsp` with RTD1296 branches. | Needs a Realtek legacy family or vendor branch; mainline support is likely incomplete for release quality. |
| Banana Pi W3 | Rockchip RK3588 | `candidate-legacy` | Official BSP repo is `BPI-W3-BSP`; vendor kernel/U-Boot must be mapped into Armbian. | Compare with existing `rockchip64`/RK3588 board configs and add only after bootloader blobs are identified. |
| Banana Pi F2S / F2P | Sunplus SP7021 | `candidate-legacy` | Official BSP repo is `BPI-F2S-bsp` with U-Boot 2019.04 and kernel 5.4.35. | New legacy family required; low priority unless hardware is available. |
| Banana Pi EAI80 | Edgeless EAI80 | `candidate-legacy` | Official BSP repo is `BPI-EAI80-bsp`. | Treat as vendor-only until kernel and bootloader licensing/redistribution are clear. |
| Banana Pi R4 Lite / R4 Pro | MediaTek Filogic | `blocked` | Official OpenWrt/BSP repos exist, but this tree only has R4 baseline. | Keep OpenWrt-style boards out of Debian/Ubuntu release until ATF/FIP and Ethernet firmware are verified. |
| Banana Pi RV2 | Siflower SF21H8898 | `blocked` | Official OpenWrt BSP repos exist. | Not a first-wave Debian/Ubuntu target. |
| Banana Pi CM5 / CM6 class boards | Mixed vendor BSP | `blocked` | Needs exact board/SOC mapping and bootloader source audit. | Add after official docs/BSP are mapped to concrete SoC families. |

## Release policy for newly added boards

1. Add only one server image target first, preferably Debian 13 or Ubuntu 24.04.
2. Require serial boot log, `lsblk`, `fdisk -l`, Ethernet, SD/eMMC read test, and
   a clean enough `dmesg` before expanding to the full OS matrix.
3. Keep unverified boards as `.wip`; move to `.csc` only after one real board
   boots consistently.
4. Do not publish desktop images for headless/router boards until display output
   or the lack of display is explicitly documented.
5. Legacy BSP images must include the kernel generation in the image name so
   users do not confuse them with mainline/current images.

## Reference sources

- Banana Pi official documentation: https://docs.banana-pi.org/
- BPI-SINOVOIP GitHub repositories: https://github.com/BPI-SINOVOIP
- BPI-SINOVOIP Armbian build branch set: https://github.com/BPI-SINOVOIP/armbian-build
- Upstream Armbian build: https://github.com/armbian/build
