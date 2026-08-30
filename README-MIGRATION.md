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

Current modifier setup:

- Ctrl: left-inner thumb (32), sticky
- Shift: right-inner thumb (33), sticky (original dedicated thumb access)
- Alt / macOS Option: left outer + left middle thumbs (30+31), sticky
- Shift alternate combo: right middle + right outer thumbs (34+35), sticky
- GUI / Windows / macOS Command: D + F + left-middle thumb/Space (12+13+31), sticky

The earlier cross-hand thumb combos (30+35 for Alt and 31+34 for GUI) were removed.
The original D+F+Space GUI combo was restored, but now uses the same sticky
modifier behavior as the other modifier combos.

All sticky modifiers use the same 500 ms timeout. Modifier presses are ignored
as sticky-release triggers, so they can be rolled/chained. `quick-release` makes
the chain release as soon as the first non-modifier key is pressed.

The 30+31 Alt combo intentionally overlaps the 30+31+32 Game combo, and the
34+35 Shift combo intentionally overlaps the 33+34+35 Polish one-shot combo.
ZMK supports fully overlapping combos; these overlapping combos all use an 80 ms
timeout.

If 500 ms feels too short/long, change `STICKY_TIMEOUT_MS` near the top of
`config/skeletyl.keymap`.

## OS modes and semantic shortcut keys

The firmware now has three explicit OS modes: **Windows**, **Linux**, and
**macOS**. Windows is the baseline state; Linux and macOS use transparent state
layers plus conditional overlays. Real modifiers remain real HID modifiers:

- GUI = Windows/Super on Windows/Linux, Command on macOS
- Alt = Alt on Windows/Linux, Option on macOS
- Ctrl remains Ctrl on every OS

On EXTRA, the five left home-row positions are the semantic shortcut row:

    A          S     D      F      G
    Select All Cut   Copy   Paste  Undo

Hold the **left outer thumb** key (Enter when tapped, EXTRA when held), then use
these five positions. The complete shortcut set therefore works with the left
hand only. NAV mirrors the same row as an optional second access path.

The emitted keys depend on OS mode:

| Action | Windows | Linux | macOS |
| --- | --- | --- | --- |
| Select All | Ctrl+A | Ctrl+A | Command+A |
| Cut | Ctrl+X | `K_CUT` | Command+X |
| Copy | Ctrl+Insert | `K_COPY` | Command+C |
| Paste | Shift+Insert | `K_PASTE` | Command+V |
| Undo | Ctrl+Z | `K_UNDO` | Command+Z |

Why Windows Copy/Paste use Insert: `Ctrl+Insert` and `Shift+Insert` are standard
Windows clipboard shortcuts and are also handled as Copy/Paste by Windows
Terminal, without reusing Ctrl+C as the copy gesture.

Why Linux uses `K_*`: these are dedicated HID editing keycodes rather than
Ctrl-based terminal control sequences. ZMK currently marks `K_CUT`, `K_COPY`,
`K_PASTE`, and `K_UNDO` as supported on Linux. Application and terminal support
can still vary: a host program has to bind/handle the HID editing key. This is
therefore the safest generic firmware-level Linux choice, not a mathematical
guarantee for every terminal emulator. `Select All` has no corresponding
keyboard editing key in ZMK, so Linux keeps Ctrl+A; in a shell this commonly
means "beginning of line" rather than "select all".

### Opening Hosts / Settings and selecting a mode

The old BLUE-layer thumb chord used the two outer thumb keys. Those now belong
to the Alt/Option combo, so BLUE uses a conflict-free entry path:

    hold NAV (right outer thumb) -> tap the top-left key -> release NAV

The top-left key on NAV is `&sl BLUE`, so BLUE is armed for exactly the next key
press. The bottom row of `Hosts / Settings` now contains:

    BT_CLR | WINDOWS | LINUX | MAC | USB | BLE | TOGGLE OUTPUT | STUDIO UNLOCK | ...

The three OS buttons are deterministic: selecting one first disables the other
state layer(s), so Windows/Linux/macOS cannot intentionally remain active at the
same time. USB/BLE selection is independent of OS mode.

### Bluetooth profile mapping

The five Bluetooth profile keys also select BLE output and set an OS mode. The
default generated mapping is:

- BT0 -> Windows
- BT1 -> Linux
- BT2 -> macOS
- BT3 -> Windows
- BT4 -> Linux

Pair devices in that order, or edit the five `bt*_...` macros near the top of
`config/skeletyl.keymap`. This is profile-selection automation, not host OS
autodetection.

The manual WINDOWS/LINUX/MAC buttons remain available for USB use or for
overriding the mode after a Bluetooth selection. The output preference
(USB/BLE) is persisted by ZMK; ordinary layer state is not relied on as a
persistent OS database.

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


## Polish layer compatibility note

The legacy config defined Polish characters as bare `Uxxxx` values (for example `U0105` and `U0119`). Current ZMK does not expose arbitrary Unicode code points as keycodes, so those tokens fail during devicetree parsing. They have been migrated to Polish Programmer-style Right-Alt/AltGr combinations (`RA(A)`, `RA(E)`, etc.). This keeps the firmware core-only and compile-safe, but requires a host keyboard layout that interprets those AltGr combinations as Polish characters. If host-layout-independent Unicode output is required later, use a dedicated module such as `urob/zmk-unicode` and make its host method OS-aware.


## Polish Unicode input

Polish special characters now use the `urob/zmk-unicode` module and therefore do not depend on the host keyboard layout. The EXTRA long-press keys activate the Polish layer for exactly one key. On that one-shot layer, A/C/E/L/N/O/S/X/Z emit ą/ć/ę/ł/ń/ó/ś/ź/ż; holding Shift emits the uppercase variants.

The OS selector also selects the matching Unicode input method:
- Windows: WinCompose mode (`UC_SET_WIN_COMPOSE`). Install WinCompose on Windows.
- Linux: Linux/IBus Unicode mode (`UC_SET_LINUX`). On Ubuntu/GNOME this normally works with the standard Ctrl+Shift+U Unicode input path.
- macOS: Unicode Hex Input mode (`UC_SET_MACOS`). Add/enable the `Unicode Hex Input` input source in macOS.

The regular outer-thumb Alt combo remains `LALT` (Alt / Option). No physical `RALT` key is required for Polish Unicode input; the Unicode module can generate any required Right-Alt sequence internally.

## Build compatibility note (2026-08-30)

For the current ZMK `main` checkout used by GitHub Actions, custom deterministic
layer-toggle behaviors use `compatible = "zmk,behavior-toggle-layer"`.
The generated config uses that spelling for `tog_on` and `tog_off`.

## Latest ergonomic changes

- Sticky Alt: left outer + left middle thumb (`30 + 31`).
- Sticky Shift: left middle + left inner/right thumb (`31 + 32`).
- Sticky GUI/Win/Command: `D + F + Space` (`12 + 13 + 31`).
- The old three-right-thumb Polish combo has been removed.
- Polish letters are now long-press actions on the EXTRA layer (180 ms):
  `A/C/E/L/N/O/S/X/Z` -> `ą/ć/ę/ł/ń/ó/ś/ź/ż`; Shift selects uppercase.
  A short tap still performs that key's normal EXTRA-layer action.

