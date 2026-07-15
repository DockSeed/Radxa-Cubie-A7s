# FEL on the Allwinner A733 (Radxa Cubie A7S)

Board-verified reference for the FEL bring-up chain on the Allwinner A733
(sun60iw2). Everything here was measured on real hardware over FEL, not
guessed. FEL is Allwinner's BROM USB recovery mode; `sunxi-fel` is the open
client from `linux-sunxi/sunxi-tools` (GPL-2.0).

## Core facts (all read over FEL)

| Item | Value |
|------|-------|
| SoC ID | `0x1903` (sun60iw2). `sunxi-fel version` reports `soc=00001903(A733)` |
| FEL USB ID | `1f3a:efe8` (Allwinner FEL/OTG) |
| FEL port | USB-C DRD port (USB0). Power and FEL on one cable, the host USB powers the board in FEL |
| FEL entry | Remove the SD card, or hold the boot button while applying power |
| FEL scratchpad (BROM) | `0x000ee800` |
| Chip ID (SID) | lower 16 bit of each word match the kernel boot log |

## SRAM map (raw reads, verified)

| Region | Status |
|--------|--------|
| `0x0`, `0x20000` | Not FEL-readable, a read faults and wedges the session |
| `0x40000`..`0x73FFF` | SRAM_A2 (first 16 K is CPUS) |
| `0x74000`..`0xF3FFF` | Shared SRAM |
| around `0xee800` | BROM scratchpad (see above) |

About 720 K contiguous from `0x40000`. Usable for an SPL, with the BROM
sitting at the top of the range.

## soc_info entry for A733

sunxi-tools did not have an A733 entry. It is now going upstream in
`linux-sunxi/sunxi-tools` PR #223 (author dlan17). Treat that PR as the
canonical source and verify any exact address against it before use. The
sketch below shows the fields, with the confidence of each value called out:

```c
    .soc_id       = 0x1903,     /* Allwinner A733 (sun60iw2). Verified: sunxi-fel version */
    .name         = "A733",
    .spl_addr     = 0x47000,    /* SD boot load address, hard requirement */
    .sid_base     = 0x03006000, /* Verified via `sunxi-fel sid`, key at +0x200 */
    .sid_offset   = 0x200,
    .sid_sections = generic_2k_sid_maps,
    .sram_size    = 180 * 1024,
    .icache_fix   = true,

    /* scratch/thunk: NOT finalized upstream. The values below are FEL-tested
     * and used by the PR #223 diff, but the PR discussion also proposes a low
     * variant (scratch 0x46000, thunk 0x45E00) placed under spl for SD image
     * compatibility. Confirm the final choice against PR #223 before relying
     * on it. */
    .scratch_addr = 0x71000,
    .thunk_addr   = 0x72000, .thunk_size = 0x200,
    .swap_buffers = /* buffers {0x47000, 0x72400, 0x1C00} */

    /* reset-related fields: inherited from the A523 entry and used only for
     * exec-reset and the reset command. Not separately verified on A733. */
    .rvbar_reg    = 0x08001004,
    .watchdog     = &wd_a523_compat,
```

CCU base is `0x02002000` (board-verified). The region `0x46E00`..`0x48C00`
is FEL-corrupting and must be avoided, this matches the PR #223 discussion.

## Cheat sheet

```bash
FEL=sunxi-fel

$FEL version                 # read SoC ID (safe)
$FEL sid                     # read chip ID / eFuse (safe)
$FEL hex 0x47000 0x40        # raw SRAM read (safe only on valid SRAM)
$FEL readl 0x47000           # 32-bit read (needs a correct soc_info scratch)
$FEL writel 0x47000 0xdead   # 32-bit write (needs soc_info)
$FEL spl u-boot-sunxi-with-spl.bin   # load and run an SPL
$FEL exe  0x48000            # call a function, code must return with `bx lr`
```

## Gotchas (learned the hard way)

1. `readl` and `writel` upload and execute a stub at `scratch_addr`. Without
   a correct soc_info the stub runs into nothing and the session wedges. Set
   up soc_info first, then use readl/writel.
2. A raw `hex`/`read` on an unmapped address faults and wedges. Raw reads are
   only safe on known-valid SRAM. Blind scanning wedges the session.
3. Wedged is not fatal. Replug the FEL cable for a clean session. FEL does
   not brick the board, the BROM is in ROM and a power cycle resets it.
4. The BROM is not directly readable from FEL. Dumping it needs a running
   helper SPL that copies BROM into SRAM to read it back. That is an exec
   chain, not a plain read.

## Exec chain proven (2026-07-15)

First verified FEL code execution on the A733. A small function at
`0x48000` writes `0xCAFEBABE` to `0x50000` and returns with `bx lr`. The six
instruction words were poked with `writel`, read back exactly, then run with
`exe 0x48000`. FEL returned and the session stayed alive, and `readl 0x50000`
returned `0xcafebabe`.

`exe <address>` is a function call. The code must return with `bx lr`, or FEL
stays dead until a power cycle.

## Bulk transfer bug on Intel xHCI, and the fix

`sunxi-fel write` (bulk) can fail with `ERROR -8: Overflow` on an Intel xHCI
root hub. Root cause, from `fel_lib.c`: `usb_bulk_recv` requests IN transfers
with a non packet aligned length (for example a 13 byte `AWUS` status). On
Intel xHCI, libusb raises `LIBUSB_ERROR_OVERFLOW` when the length is not a
multiple of the full speed max packet size (64). Other ports and cables do not
help if they are all on the same xHCI.

Two ways around it:

- Small payloads: poke word by word with `writel` (stable, no bulk).
- Proper fix: patch `usb_bulk_recv` to read into a bounce buffer rounded up to
  a multiple of 512, then copy back only the wanted bytes. This is a clean,
  self contained candidate for an upstream sunxi-tools PR (not part of PR #223).
  Board test still pending before submission.

An EHCI path (a USB 2.0 hub between host and the FEL port) also avoids the
xHCI bulk bug.

## UART for SPL bring-up

To get prints out of an SPL over the UART:

```c
set_wbit(ccu_base + 0xe000 + uart_idx * 4, 0x10001);  /* CCU 0x0200e000 = 0x10001 enables UART0 */
```

UART0 base is `0x02500000` (16550 compatible), pin PB9 mux 2. This is the
recipe used by the sunxi-tools uart-hello example.

## Next step

Build a trivial bare-metal SPL (UART output or a memcopy) and run it with
`sunxi-fel spl` to prove the full exec chain end to end. That unlocks BROM
dump, open DRAM init, and everything else that depends on running code rather
than reads on this SoC.

## Upstream pointers

- `linux-sunxi/sunxi-tools` PR #223 adds the A733 soc_info and a uart-hello
  example. It does not include DRAM init. Open A733 DRAM training is still
  the missing piece.
- The xHCI bounce buffer fix above is a separate upstream candidate.
