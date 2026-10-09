# QIDI Q1 Pro: stock MCU firmware

[Русский](README.ru.md)

The original QIDI firmware for both boards of the QIDI Q1 Pro: the mainboard
(STM32F401) and the toolhead (RP2040). Use it to go back to the stock firmware
after mainline Klipper, or to recover a board.

> It only works with the stock QIDI Klipper. Mainline Klipper will not connect
> to it.

| File | Board | How |
|---|---|---|
| `qd_mcu.bin` | mainboard | microSD |
| `stock-toolhead.uf2` | toolhead | USB cable, BOOT button |
| `qidi-bootloader-0x08000000.bin` | mainboard | usually not needed, see [below](#qidi-bootloader-qidi-bootloader-0x08000000bin) |

## Mainboard: `qd_mcu.bin`

You need a microSD card formatted as FAT32. Copy `qd_mcu.bin` to the root of
the card. Do not rename the file.

The mainboard has a microSD slot (marked 1) and the RESET button of the MCU
above it (marked 2).

![Mainboard](images/mainboard.jpg)

With the printer on:

1. Insert the card into the slot.
2. Press and release RESET. The MCU restarts and flashes the file from the card.
3. Wait a minute.
4. Remove the card. The printer does not need to be turned off.

![Flashing from microSD](images/mainboard-sdcard.jpg)

## Toolhead: `stock-toolhead.uf2`

You need:

* a USB cable with 2.54 mm Dupont pins on the other end. It is a plain USB
  cable, not a USB-UART adapter. A charge-only cable (red and black wires only)
  will not work;
* a zip tie;
* a computer.

![USB cable](images/usb-cable.svg)

### 1. Remove the toolhead back cover

Turn the printer off. Remove the 4 screws of the toolhead back cover and take
the cover off.

![Toolhead back cover](images/toolhead-cover.svg)

### 2. Connect the cable

On the right edge of the toolhead board there are 4 holes for USB (marked 1)
and the BOOT and RESET buttons below them (marked 2).

![Toolhead board](images/toolhead-board.jpg)

Insert the pins into the holes, top to bottom:

* **5V** (square pad, top): nothing. Leave the red wire unconnected.
* **GND**: black.
* **D+**: green.
* **D−**: white.

![Connecting the wires](images/toolhead-wiring.jpg)

The holes do not hold the pins. Tie the wires together with a zip tie so that
each one is slightly tensioned and its pin presses against the hole.

### 3. Put the toolhead into USB mode

1. Plug the USB end into the computer.
2. Turn the printer on.
3. Hold BOOT, press and release RESET, release BOOT.

![BOOT and RESET](images/bootsel.jpg)

The computer shows a new drive called `RPI-RP2`.

### 4. Flash

Copy `stock-toolhead.uf2` to the `RPI-RP2` drive. The drive disappears by itself
when the copy is complete, and the toolhead starts the new firmware.

Turn the printer off, disconnect the cable and put the cover back.

### If `RPI-RP2` does not show up

* Swap the green and white wires: the colors of some cables do not match the
  standard.
* Check that every pin presses against its hole. Pull the zip tie tighter.
* Repeat the BOOT and RESET sequence. You can also hold BOOT while turning the
  printer on.
* Try another cable: it must have 4 wires.

---

## Details

### QIDI bootloader: `qidi-bootloader-0x08000000.bin`

Usually not needed. This is the mainboard loader that flashes `qd_mcu.bin` from
microSD. Flashing from the card, Klipper and Katapult never touch it. It is only
useful if it gets damaged and the board no longer flashes from the card.

It cannot be written from microSD, only at `0x08000000` with the STM32 ROM
bootloader or over SWD.

It is byte for byte the same as the bootloader from another Q1 Pro, so it is
the same on every Q1 Pro.

### What gets overwritten

* `qd_mcu.bin` is written from `0x08008000`. If Katapult was installed, it is
  overwritten. The QIDI bootloader stays.
* `stock-toolhead.uf2` is written from the start of the RP2040 flash. QIDI has
  no bootloader of its own on the toolhead. The `RPI-RP2` mode is in the RP2040
  ROM and can not be erased, so the toolhead can always be flashed this way.

### Where the files come from

A flash dump of both boards of a stock Q1 Pro, taken on 2026-10-08 with the
Klipper `debug_read` command. Two passes matched byte for byte. Firmware
version: `v0.10.0-530-g3387a9c2-dirty`, QIDI build from 2022-12-22.

| File | Source |
|---|---|
| `qd_mcu.bin` | mainboard flash, `0x8000`-`0xDC4C` |
| `qidi-bootloader-0x08000000.bin` | mainboard flash, `0x0`-`0x8000` |
| `stock-toolhead.uf2` | toolhead flash, `0x0`-`0x5D00` (boot2 W25Q080 + Klipper) |

Check the files with `sha256sum -c SHA256SUMS`. None of them has been written
to hardware yet.

### Licenses

* `qd_mcu.bin` and `stock-toolhead.uf2` are QIDI's build of Klipper, licensed
  under the GNU GPLv3. QIDI's source code: https://github.com/QIDITECH/klipper
  (the build is marked `-dirty`, so it was made from a tree with uncommitted
  changes).
* `qidi-bootloader-0x08000000.bin` belongs to QIDI. It is published here only
  to recover printers.
* The board pictures are QIDI's product photos of the toolhead board
  (https://qidi3d.com/products/q1pro-adapter-plate) and the mainboard
  (https://qidi3d.com/products/q1pro-motherboard) with our marks.
