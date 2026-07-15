# Roadmap and open work

Status of the open Radxa Cubie A7S (Allwinner A733) bring-up. The overall
goal is a fully open, blob-free, shippable software stack. This list is
honest about what is done, what is in progress, and where help is wanted.

Legend: [done] [wip] [help wanted]

## Boot chain

- [done] FEL bring-up characterized and board-verified. See
  [bringup/fel-a733.md](bringup/fel-a733.md).
- [done] First FEL code execution proven on the A733 (exec chain works).
- [wip] Bulk-transfer fix for `sunxi-fel` on Intel xHCI hosts. Patch written,
  board test pending, then upstream to sunxi-tools.
- [wip] Trivial bare-metal SPL over `sunxi-fel spl` to prove the full load and
  run chain end to end.
- [help wanted] Open DRAM init in boot0/SPL to replace the closed vendor
  libdram blob (LPDDR5). The controller bring-up stalls at config, it looks
  like the DesignWare umctl2 swctl handshake. Anyone with DesignWare umctl2
  DDR experience or open Allwinner DRAM init: this is the wall.

## Kernel and drivers

- [wip] Mainline A733 tracking. Clocks, DMA, RTC, and a first DT are on the
  lists. Pinctrl is the choke point.
- [help wanted] Ethernet PHY support upstream (MAE0621A) for a clean mainline
  path.
- [wip] GPU. Display works, but the GPU is capped at 600 MHz and the thermal
  cap is real. Going the open route (mesa, powervr) for a shippable image.
- [wip] NPU open path (etnaviv, Teflon).
- [wip] eDP output (DP over the combo PHY works on stock).

## Hardware documentation

- [wip] Hardware reference (rails, DVFS, pinout). Notable findings so far:
  rails are not clamped against absolute maximum in the device tree, and more
  power-on lanes are enabled than needed. To be published here after cleanup.

## What will not be here

This repository is blob-free by design. No OS images, no vendor blobs. Only
clean-room documentation, measurements, and 3D-printable accessories.
