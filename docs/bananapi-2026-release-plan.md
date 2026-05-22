# Banana Pi 2026 Armbian release plan

Last updated: 2026-05-22

## Goal

Build a complete 2026 Banana Pi Armbian release set that keeps old Banana Pi
boards maintained while adding missing Banana Pi boards that official Banana Pi
supports but upstream Armbian does not yet cover.

The release target is:

- Debian 12 Bookworm server and desktop
- Debian 13 Trixie server and desktop
- Ubuntu 22.04 Jammy server and desktop
- Ubuntu 24.04 Noble server and desktop
- Ubuntu 26.04 Resolute server and desktop, where Armbian and architecture
  support are available

The output layout remains `output/images/bpi-*`, with `.img.xz`, checksum, and
metadata sidecars.

## Operating rules

1. Do not break boards that already boot and have been validated, especially
   Banana Pi CM4/CM4IO eMMC tuning and BPI-6204 local work.
2. Treat mainline and legacy/vendor BSP as separate release tracks.
3. Never promote a board from `.wip` to `.csc` or `.conf` without a real boot
   log and basic hardware validation.
4. Build one server test image first for every new board. Expand to the full OS
   matrix only after hardware validation.
5. For legacy/vendor BSP images, include the kernel family or legacy marker in
   image names and release notes so users do not confuse them with mainline.
6. Keep every board decision in `docs/bananapi-support-matrix.md`; no hidden
   ad-hoc board list.
7. Every release directory must be auditable: image, `.sha`, `.txt`, and build
   log or manifest must be present.

## Repositories and branches

Primary working tree:

- Local path: `/media/pi/SMCI/armbian/build`
- Upstream reference: `https://github.com/armbian/build`
- Banana Pi reference: `https://github.com/BPI-SINOVOIP/armbian-build`

Branch policy:

- Keep upstream sync work separate from board enablement work.
- Use one integration branch for this plan, for example
  `bananapi-2026-release-plan`.
- After the plan is reviewed, merge selected changes into the Banana Pi release
  branch such as `bpi-v26.8.0-trunk`.
- Do not push local experimental CM4/6204 changes unless they are explicitly
  part of the current task.

## Source inputs

Use these inputs for board inventory and porting:

- Banana Pi official docs: https://docs.banana-pi.org/
- BPI-SINOVOIP GitHub BSP repositories: https://github.com/BPI-SINOVOIP
- BPI-SINOVOIP Armbian fork: https://github.com/BPI-SINOVOIP/armbian-build
- Upstream Armbian build: https://github.com/armbian/build
- Cached Linux and U-Boot worktrees under `cache/sources`
- Existing verified image outputs under `output/images/bpi-*`

## Board classification

Each board must be classified in `docs/bananapi-support-matrix.md`:

- `supported`: already included in the normal release set.
- `bpi-added`: added by Banana Pi branch work but not necessarily upstream.
- `candidate-mainline`: kernel and U-Boot support exists; needs Armbian board
  integration and hardware validation.
- `candidate-legacy`: official/vendor BSP exists; needs a legacy Armbian family
  or branch.
- `blocked`: missing bootloader, kernel source, redistributable blobs, or
  hardware validation.

Current first-wave missing/incomplete candidates:

| Board | Priority | Track | Reason |
| --- | --- | --- | --- |
| Banana Pi P2 Pro | P0 | mainline candidate | RK3308 DTS exists in Linux and U-Boot; small Armbian integration needed. |
| Banana Pi R3 | P0 | mainline/Filogic | MT7986 kernel/U-Boot evidence exists; high user value. |
| Banana Pi R3 Mini | P0 | mainline/Filogic | Shares most R3 work; should follow R3. |
| Banana Pi R64 | P1 | mainline or legacy | MT7622 DTS exists; boot flow needs decision. |
| Banana Pi M4 RTD1395 | P1 | mainline or legacy | Mainline DTS exists, but boot chain may need vendor BSP. |
| Banana Pi W2 | P2 | legacy | Official RTD1296 BSP exists; likely not release-quality mainline. |
| Banana Pi W3 | P2 | legacy/RK3588 | Official BSP exists; needs bootloader and blob mapping. |
| Banana Pi F2S/F2P | P3 | legacy | SP7021 BSP exists; new family required. |
| Banana Pi router/OpenWrt-only boards | P3 | blocked until proven | Do not publish Debian/Ubuntu until boot chain and Ethernet are proven. |

## Execution phases

### Phase 0 - Planning and repository hygiene

Deliverables:

- `docs/bananapi-2026-release-plan.md`
- `docs/bananapi-support-matrix.md`
- A pushed branch that can be reviewed from GitHub

Actions:

1. Commit and push the plan documents first.
2. Keep unrelated local changes out of the plan commit.
3. Record branch name and remote URL in the final status.

Exit criteria:

- Plan is visible on GitHub.
- Matrix exists and can be updated by future commits.

### Phase 1 - Inventory and gap audit

Deliverables:

- Updated `docs/bananapi-support-matrix.md`
- Board gap list grouped by SoC family
- Output image completeness report for `output/images/bpi-*`

Actions:

1. Compare official Banana Pi docs with `config/boards`.
2. Compare Banana Pi BSP repos with current Armbian families.
3. Compare upstream Armbian `main`, `v26.02`, and BPI branches.
4. For every missing board, identify:
   - SoC
   - kernel DTS availability
   - U-Boot defconfig/DTS availability
   - ATF/FIP/blob requirement
   - likely Armbian family
   - required boot media
   - validation hardware status

Exit criteria:

- No board is left in an unknown state.
- Every candidate has a next action or a blocker.

### Phase 2 - Mainline quick-win boards

Initial boards:

- Banana Pi P2 Pro
- Banana Pi R3
- Banana Pi R3 Mini
- Banana Pi R64

Actions for each board:

1. Add board config as `.wip`.
2. Add only the minimum required U-Boot/kernel patches.
3. Build one server image first.
4. Save build logs and image metadata.
5. Boot on hardware before expanding OS matrix.

Validation commands on target:

```bash
uname -a
cat /proc/device-tree/model
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
fdisk -l
dmesg | grep -Ei 'error|fail|mmc|emmc|eth|phy|firmware'
ip addr
dd if=/dev/mmcblk0 of=/dev/null bs=1M count=1000 status=progress
```

For eMMC boards, repeat the read test on the eMMC device. If the device names
differ, record the mapping in the matrix.

Exit criteria:

- One server image boots to login.
- Network works.
- Storage is stable enough for a 1 GB read test.
- No fatal bootloader/kernel storage errors remain.

### Phase 3 - Legacy/vendor BSP integration

Candidate BSP sources:

- `BPI-M4-bsp` for RTD1395
- `BPI-W2-bsp` for RTD1296
- `BPI-W3-BSP` for RK3588 vendor tree
- `BPI-F2S-bsp` for SP7021
- `BPI-R64-bsp-5.4` if mainline boot flow is insufficient

Actions:

1. Audit source license, kernel version, U-Boot version, and required blobs.
2. Decide whether to create a new family or extend an existing family.
3. Build only one server image per BSP family first.
4. Clearly mark image names and release notes as `legacy`.
5. Do not publish desktop images until basic display, input, and package
   compatibility are verified.

Exit criteria:

- Legacy builds are reproducible inside Armbian build.
- Kernel packages install cleanly on Debian/Ubuntu target rootfs.
- Boot logs are archived.

### Phase 4 - Release image matrix expansion

Eligible boards:

- Existing supported boards already in the BPI release set
- New boards that passed Phase 2 or Phase 3 validation

Actions:

1. Build server images first for all eligible boards and OS targets.
2. Build desktop images only after server images complete.
3. Keep RISC-V and legacy boards on the OS matrix they actually support.
4. Archive images under `output/images/bpi-*`.
5. Generate or verify `.sha` and `.txt` sidecars.
6. Run completeness audit after every batch.

Exit criteria:

- Every intended board/OS/type combination is present or explicitly skipped.
- Skipped combinations have a written reason.
- No directory contains stale image names from an older release date.

### Phase 5 - Hardware validation and promotion

Promotion rules:

- `.wip` to `.csc`: one real board boots reliably and has basic hardware tested.
- `.csc` to `.conf`: repeatable validation, maintainer assigned, release notes
  written, and no known critical boot/storage/network issue.
- Keep `.eos` only for boards that cannot reasonably be maintained.

Required hardware validation:

- Serial boot log
- Login prompt
- Network DHCP
- SD/eMMC detection
- 1 GB storage read test
- Reboot test
- `dmesg` review
- For desktop images: display manager start, keyboard/mouse, basic GPU/display

Exit criteria:

- Release notes include validated and unvalidated hardware features.
- Matrix status is updated.

### Phase 6 - Release packaging

Deliverables:

- Final `output/images/bpi-*` image set
- Checksums
- Build manifest
- Board support matrix
- Release notes

Actions:

1. Rename files with final release date.
2. Compress final raw images as `.img.xz`.
3. Verify checksums.
4. Generate per-board manifest.
5. Copy or publish final images to the expected release location.

Exit criteria:

- The release can be audited without rebuilding.
- Every image has a matching checksum and metadata file.

## Immediate task queue after this plan is pushed

1. Finish Phase 0 by pushing this plan branch.
2. Complete the support matrix with the first-wave candidate evidence.
3. Build one `bananapip2pro.wip` server image.
4. Start Filogic R3/R3 Mini boot-flow work:
   - identify MT7986 ATF target
   - identify FIP offsets
   - map SD and eMMC variants
   - create `.wip` board configs only after boot artifacts are understood
5. Investigate R64 MT7622:
   - decide `mt7622` family vs extending existing MediaTek family
   - identify bootloader write offsets
6. Audit M4/W2/W3 legacy BSP viability.
7. Build only validated boards into the full release matrix.

## Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| A board builds but does not boot | Keep new boards as `.wip`; require serial logs before release. |
| Legacy kernels fail on newer Debian/Ubuntu | Start with one server image; mark unsupported OS combinations explicitly. |
| Bootloader blobs are missing or not redistributable | Mark board `blocked`; do not publish images. |
| Existing CM4 eMMC stability regresses | Keep CM4 changes isolated and do not touch known-good tuning unless requested. |
| Output folders contain stale or incomplete images | Run completeness audit after each build batch. |
| Upstream Armbian changes conflict with BPI patches | Sync by family, not by bulk overwrite; keep patch series small and documented. |

## Definition of done

This plan is complete when:

1. The support matrix is current.
2. Every official Banana Pi board is classified.
3. Every release image in `output/images/bpi-*` is complete or explicitly
   skipped with a reason.
4. New boards have boot logs before promotion.
5. Debian 12/13 and Ubuntu 22.04/24.04/26.04 server/desktop images are generated
   for all validated boards where the OS and architecture are supported.
