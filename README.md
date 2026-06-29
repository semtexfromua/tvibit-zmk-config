# Tvibit

A custom **wireless split** keyboard running [ZMK](https://zmk.dev).
Two halves, 84 keys total (**42** per half / "6×7"), column-staggered, with a
3-key rotated thumb cluster per side and an OLED status display.

| | |
|---|---|
| **Controller** | nice!nano v2 clone (Pro Micro pinout, nRF52840) → built as `nice_nano_v2` |
| **Connection** | Bluetooth LE split. Left = central (USB / ZMK Studio), Right = peripheral |
| **Display** | SSD1306 OLED over I2C (`&pro_micro_i2c`, 128×32) |
| **Matrix** | 6 rows × 7 columns per half (`col2row`) |

## ⚠️ Before this works on hardware

There is **no PCB yet**, so the GPIO pins in
`boards/shields/tvibit/tvibit_left.overlay` and `tvibit_right.overlay` are
**placeholders**. Edit `col-gpios` / `row-gpios` to match your real hand-wiring
(`&pro_micro N` = Pro Micro logical pin N). The firmware compiles as-is, but the
keys won't scan correctly until the pins are right.

## Layout

```
LEFT (42)                                 RIGHT (42)
3 outer/function cols + 4 main cols        mirror of left, with the inverted-T
(column-staggered) + 3 rotated thumbs      arrow block (←↑↓→) + 3 rotated thumbs
```

Per-half electrical matrix: **6 rows × 7 columns** = 42 (`col2row`). Each key maps
to a `row,col` intersection — see the wiring diagram in chat. Row 5 (bottom) carries
the finger bottoms + the 3 thumbs + the right arrow block; thumbs reuse the column
wires freed by the dropped keys, so no 7th row is needed.

The physical→electrical mapping was generated 1:1 from the original
keyboard-layout-editor JSON and lives in
[`boards/shields/tvibit/tvibit.dtsi`](boards/shields/tvibit/tvibit.dtsi)
(`default_transform`). Each binding in the keymap lines up with that map.

## Keymap

[`config/tvibit.keymap`](config/tvibit.keymap) — a **starter** keymap (QWERTY):

| Layer | Name | Contents |
|---|---|---|
| 0 | BASE | letters, number row, modifiers, thumbs |
| 1 | LOWER | symbols (left) + numpad (right) — hold left-inner thumb |
| 2 | RAISE | F-keys (left) + arrows/navigation (right) — hold outer thumb |
| 3 | ADJUST | Bluetooth profiles, USB/BLE output, bootloader/reset — LOWER+RAISE |

Customize it directly, or live-edit over USB with
[ZMK Studio](https://zmk.dev/docs/features/studio) (already enabled on the central).

## Building

Pushing to GitHub triggers `.github/workflows/build.yml`, which produces three
`.uf2` files under the run's **Artifacts**:

- `tvibit_left.uf2` → flash to the left half
- `tvibit_right.uf2` → flash to the right half
- `settings_reset.uf2` → flash to either half to wipe BLE pairings if the
  halves won't connect, then re-flash the normal firmware

**Flashing:** double-tap reset on the nice!nano to enter the bootloader (a USB
drive appears), then copy the matching `.uf2` onto it.

## Project structure

```
build.yaml                         GitHub Actions build matrix
zephyr/module.yml                  marks this repo as a ZMK module (board_root)
config/
  west.yml                         pulls in zmkfirmware/zmk
  tvibit.keymap                    layers / bindings
  tvibit.conf                      BLE, sleep, battery, display, Studio
boards/shields/tvibit/
  Kconfig.shield                   SHIELD_TVIBIT_LEFT / _RIGHT
  Kconfig.defconfig                name, split role (left central), OLED defaults
  tvibit.dtsi                      kscan + matrix-transform + OLED
  tvibit_left.overlay              left GPIO pins  (TODO: real wiring)
  tvibit_right.overlay             right GPIO pins + col-offset (TODO)
  tvibit_left.conf / _right.conf   per-half overrides
  tvibit.zmk.yml                   hardware metadata
.github/workflows/build.yml        official ZMK user-config build
```
