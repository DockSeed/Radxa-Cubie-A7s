# Roadmap and open work

Status of the open Radxa Cubie A7S (Allwinner A733) bring-up. The board already
runs a full graphical Linux desktop. The mission is to make that stack fully
open, blob-free, and shippable.

Legend: [done] [wip] [help wanted]

Note on "works today": the current working images boot and run well, but still
lean on some patched vendor blobs (GPU, DRAM training, and others). Replacing
those with open code is what the rest of this list is about.

## What works today

- [done] Fedora aarch64 port with a GPU-accelerated LXQt desktop booting on the
  board. A pioneering distro port for this SoC.
- [done] GPU bring-up on the open Mesa driver (Imagination powervr), with
  benchmarks. Stable at a pinned clock (performance governor).
- [done] Armbian build track for the board.
- [done] A single kernel base (mainline 6.18.x LTS plus BSP patches) feeding
  both the Fedora and Armbian tracks.
- [done] DisplayPort output over the USB-C combo PHY (works on stock too).
- [done] NVMe over the PCIe FPC link.
- [done] Gigabit Ethernet.
- [done] eMMC and SD boot.
- [done] FEL bring-up characterized and board-verified, including the first
  proven FEL code execution on the A733. See [bringup/fel-a733.md](bringup/fel-a733.md).
- [done] Thermal and DVFS behavior characterized.
- [done] Hardware reference: the A7S as populated - power rails and tree, the
  permitted DVFS voltage window, straps, pinout, and connectors, with board
  photos and full datasheet/schematic provenance. See
  [hardware/hardware-reference.md](hardware/hardware-reference.md).

## In progress

- [wip] GPU DVFS (dynamic clocking). The GPU is stable at pinned clocks, but
  certain clock points hang it, and part of that is a Mesa driver issue. The
  thermal cap is real.
- [wip] Remove the remaining GPU firmware blob dependency so a shippable image
  is fully blob-free. The open Mesa driver already runs the GPU.
- [wip] Open NPU path (etnaviv, Teflon) as an alternative to the vendor VIPLite.
- [wip] eDP output (DisplayPort already works).
- [wip] Mainline upstreaming of the A733. Clocks, DMA, RTC, and a first device
  tree are on the lists. Pinctrl is the choke point.
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
