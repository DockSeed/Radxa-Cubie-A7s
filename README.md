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
