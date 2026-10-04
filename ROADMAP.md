# Roadmap and open work

Status of the open Radxa Cubie A7S (Allwinner A733) work. This repository holds
the hardware side: notes, bring-up references, measurements and 3D prints.

The software moved to [a7s-build](https://github.com/DockSeed/a7s-build): one
`./build.sh` builds a Debian 13 image for the board from source (boot chain,
Linux 6.18 with the A733 patch series, root filesystem, optional Xfce desktop).
What the image offers and where help is wanted is kept there:
[how-we-built-it.md](https://github.com/DockSeed/a7s-build/blob/main/docs/how-we-built-it.md#what-the-image-offers)
and [help-wanted.md](https://github.com/DockSeed/a7s-build/blob/main/docs/help-wanted.md).

Legend: [done] [wip] [help wanted]

## In this repository

- [done] FEL bring-up characterized and board-verified, including the first
  proven FEL code execution on the A733. See [bringup/fel-a733.md](bringup/fel-a733.md).
- [done] Hardware reference: the A7S as populated - power rails and tree, the
  permitted DVFS voltage window, straps, pinout, and connectors, with board
  photos and full datasheet/schematic provenance. See
  [hardware/hardware-reference.md](hardware/hardware-reference.md).
- [done] Dev stand with NVMe adapter and fan, 3D-printable. See
  [3d-prints/dev-stand-nvme](3d-prints/dev-stand-nvme/).
- [wip] Bulk-transfer fix for `sunxi-fel` on Intel xHCI hosts. Patch written,
  board test pending, then upstream to sunxi-tools.

## Help wanted

- [help wanted] Open DRAM init in boot0/SPL to replace the closed vendor
  libdram blob (LPDDR5). Two separate things, do not confuse them:
  - The board-specific DRAM parameters can be obtained cleanly, without
    disassembling anything, by reading the documented parameter block in the
    boot0 image header with `sunxi-fw` (apritzel/sunxi-fw). The vendor boot0
    also prints its training summary on the serial console.
  - What does not exist for the A733 is the open init sequence that consumes
    those parameters. The controller is a DesignWare umctl2 with an LPDDR5
    PHY, and the bring-up stalls at config, most likely the umctl2 swctl
    handshake. The old sunxi Kconfig failsafe timings are DDR3 only and do
    not apply here. Good public references are the umctl2 init in mainline
    U-Boot for other SoCs (for example STM32MP1) and vendor TRMs that document
    the same controller.
  - So parameters are not the solution. The open umctl2 and PHY bring-up code
    is the actual work. Anyone with DesignWare umctl2 DDR experience or open
    Allwinner DRAM init on a recent SoC: this is the wall.
- [help wanted] Ethernet PHY support upstream (MAE0621A) for a clean mainline
  path.

## What will not be here

This repository is blob-free by design. No OS images, no vendor blobs. Only
clean-room documentation, measurements, and 3D-printable accessories.
