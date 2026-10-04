# Radxa Cubie A7S

Community information hub for the Radxa Cubie A7S single-board computer (Allwinner A733). Hardware notes, documentation, and 3D-printable accessories. Work in progress.

Looking for software? [a7s-build](https://github.com/DockSeed/a7s-build) builds a Debian 13 image for the board from source with one `./build.sh`.

## Contents

### 3D prints

| Model | Description |
|-------|-------------|
| [Dev-Stand with NVMe](3d-prints/dev-stand-nvme/) | Open bench stand that holds the board flat with a Waveshare M.2 NVMe adapter, a rear 40x10 mm fan, an SMA antenna hole, and a pass-through for the 30-pin GPIO header |

### Documentation

| Doc | Description |
|-----|-------------|
| [FEL on the A733](bringup/fel-a733.md) | Board-verified FEL bring-up reference: SoC ID, SRAM map, soc_info entry, cheat sheet, gotchas, and the xHCI bulk fix |
| [Hardware reference](hardware/hardware-reference.md) | The A7S as populated: power rails and tree, the permitted DVFS voltage window, straps, pinout, connectors, and board photos. Every claim is traced to datasheet, schematic, or measurement. |

More bring-up notes will be added under their own folders.

## Related projects

Open work on the board and the A733 by others:

| Project | Description |
|---------|-------------|
| [radxa-build/radxa-a733](https://github.com/radxa-build/radxa-a733) | Radxa's official system images for its A733 boards |
| [radxa-pkg/u-boot-dlan17](https://github.com/radxa-pkg/u-boot-dlan17) | Radxa's packaging of the A733 boot chain, based on [U-Boot](https://github.com/dlan17/u-boot) and [TF-A](https://github.com/dlan17/trusted-firmware-a) by Yixun Lan |
| [alexcaoys/allwinner-bsp](https://github.com/alexcaoys/allwinner-bsp/tree/linux-6.18.y) | Radxa's Allwinner BSP drivers ported to Linux 6.18 |
| [QinCai-rui/allwinner-bsp](https://github.com/QinCai-rui/allwinner-bsp/tree/linux-7.1.y) | The same BSP port carried on to Linux 7.1 |
| [NickAlilovic/build](https://github.com/NickAlilovic/build/tree/Radxa-mainline-WIP-a7s) | Armbian build for the A7S on top of those drivers, work in progress; discussed in the [Armbian forum](https://forum.armbian.com/topic/56130-radxa-cubie-a7aa7z-allwinner-a733/) |
| [fuhuasxflwb/allwinner-bsp](https://github.com/fuhuasxflwb/allwinner-bsp) | Fork of NickAlilovic/allwinner-bsp with USB gadget endpoint changes for audio |
| [chainsx/build](https://github.com/chainsx/build/tree/a733) | Armbian build fork with an early A733 branch for Radxa boards |
| [a7s-linux-drivers](https://github.com/skitzo2000/a7s-linux-drivers) | Drivers, patches and device-tree overlays for the Armbian 6.18 edge kernel: GMAC, DisplayPort over USB-C, NPU, AXP8191 CPU-rail fix, AIC8800 |
| [esp32-a7s-fel](https://github.com/skitzo2000/esp32-a7s-fel) | ESP32-S3 firmware that FEL-boots an A7S without boot media and starts an OS from a USB stick |

## Status and open work

See [ROADMAP.md](ROADMAP.md) for what is done, what is in progress, and where
help is wanted. The overall goal is a fully open, blob-free, shippable stack.

## License

Content is released under [CC BY 4.0](LICENSE), an open license for everyone.
Use it, print it, remix it, share it, commercial use included. The only
condition is attribution.

## Contributing

Pull requests are welcome. Board owners, printers, and tinkerers: open an
issue or a [pull request](https://github.com/DockSeed/Radxa-Cubie-A7s/pulls)
with fixes, additions, remixes, or notes. The [roadmap](ROADMAP.md) lists the
spots where help is wanted most.
