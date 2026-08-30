# Skeletyl ZMK migration (2026)

This repository layout is prepared for current ZMK `main`, current Zephyr board
IDs, and ZMK Studio on the left/central half.

The repository is also declared as a Zephyr module in `zephyr/module.yml`, so
current ZMK discovers the custom shield under `boards/shields/skeletyl`.

## Important controller check

The old `build.yaml` used `board: nice_nano`.  In the current ZMK/Zephyr board
revision system that means **nice!nano V1**, so this generated config uses:

    nice_nano@1//zmk

If the physical controllers are actually **nice!nano V2**, change all three
occurrences in `build.yaml` to:

    nice_nano//zmk

Do this before the first build.

## Thumb modifiers

Physical thumb positions are:

    left:   30 outer | 31 middle | 32 inner
    right:  33 inner | 34 middle | 35 outer

The requested modifier setup is:

- Ctrl: left-inner thumb (32), sticky
- Shift: right-inner thumb (33), sticky
- Alt / macOS Option: outer-thumb combo 30+35, sticky
- GUI / Windows / macOS Command: middle-thumb combo 31+34, sticky

All four use the same 500 ms sticky timeout.  Modifier presses are ignored as
sticky-release triggers, so they can be rolled/chained.  `quick-release` makes
the chain release as soon as the first non-modifier key is pressed.

If 500 ms feels too short/long, change `STICKY_TIMEOUT_MS` near the top of
`config/skeletyl.keymap`.

## OS mode

The macOS state is a transparent layer plus conditional overlays.  It changes
only semantic shortcuts; real modifiers remain real HID modifiers:

- GUI = Windows/Super on Windows/Linux, Command on macOS
- Alt = Alt on Windows/Linux, Option on macOS
- Ctrl remains Ctrl on every OS

On EXTRA, the old duplicate modifier row is now a contiguous left-hand shortcut row:

    Select All | Cut | Copy | Paste | Undo

Hold the **left outer thumb** key (Enter when tapped, EXTRA when held), then use
the five left home-row positions. This means the complete shortcut set can be
used with the left hand only. NAV mirrors the same row as an optional second
access path.

The firmware emits Ctrl+A/X/C/V/Z in Win/Linux mode and Command+A/X/C/V/Z in
macOS mode. Real Ctrl remains Ctrl and real GUI remains Win/Command; only these
semantic shortcut keys are OS-aware.

### Opening Hosts / Settings and selecting a mode

The old BLUE-layer thumb chord used the two outer thumb keys. That now belongs
to the Alt/Option combo, so BLUE has a new conflict-free entry path:

    hold NAV (right outer thumb) -> tap the top-left key -> release NAV

The top-left key on NAV is `&sl BLUE`, so BLUE is armed for exactly the next
key press. On `Hosts / Settings` (BLUE), the lower row contains:

    BT_CLR | WIN/LINUX | MAC | USB | BLE | TOGGLE OUTPUT | STUDIO UNLOCK | ...

`WIN/LINUX` and `MAC` change only OS mode. `USB`, `BLE`, and `TOGGLE OUTPUT`
change only the output path.

### Bluetooth profile mapping

The five profile buttons on the BLUE layer are macros and also select BLE output:

- BT0 -> Win/Linux
- BT1 -> Win/Linux
- BT2 -> macOS
- BT3 -> Win/Linux
- BT4 -> macOS

Pair devices in that order, or edit the five macro bindings near the top of
`config/skeletyl.keymap`.

Important limitation: this is **profile-selection automation, not host OS
autodetection**.  Pressing one of the BT profile keys sets the correct OS state.
A normal active-layer state is not persisted across a complete power cycle. If
the keyboard powers up and auto-reconnects to a previously selected Mac profile
without you pressing its profile button, select that profile once (or press the
MAC mode button).  Implementing persistent automatic OS state would require
custom firmware/module code; it is intentionally not included in this robust
core-ZMK migration.

The output preference (USB vs BLE) *is* persisted by ZMK.

## GitHub Actions build

Commit and push the repository.  `.github/workflows/build.yml` calls ZMK's
current reusable user-config workflow.  Every push builds the entries in
`build.yaml` and publishes a `firmware` artifact containing:

- `skeletyl-left.uf2`
- `skeletyl-right.uf2`
- `settings-reset.uf2`

The left build enables Studio using `studio-rpc-usb-uart` and
`CONFIG_ZMK_STUDIO=y`; the right half intentionally does not.

## First flash after this migration

Because this is a major ZMK/split migration, do one clean settings reset on both
halves:

1. Download the `firmware` artifact from the successful GitHub Actions run.
2. Put the **left** controller into bootloader mode (usually double-tap reset).
3. Copy `settings-reset.uf2` to the mounted controller drive.
4. Repeat steps 2-3 for the **right** controller.
5. Put left into bootloader again and copy `skeletyl-left.uf2`.
6. Put right into bootloader again and copy `skeletyl-right.uf2`.
7. Remove/forget the old Skeletyl Bluetooth pairing on hosts and pair again.

After normal future keymap changes, you generally only need to flash the left
(central) half unless the change affects peripheral/hardware configuration.

## ZMK Studio

Connect the central/left half over USB, select USB output, open ZMK Studio, and
press `STUDIO UNLOCK` on the BLUE layer when requested.

Once Studio has saved runtime keymap changes, later edits to the stock
`config/skeletyl.keymap` will not appear until you use **Restore Stock Settings**
in ZMK Studio.

The physical layout coordinates in `skeletyl-layouts.dtsi` are schematic, not a
millimeter-accurate Skeletyl drawing.  The key ordering is exact; the schematic
coordinates affect only how Studio draws the keyboard.
