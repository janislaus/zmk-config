# Skeletyl ZMK config — native switcher QC release (2026-08-30)

This repository targets current ZMK `main`, Zephyr 4.1 board IDs, and ZMK Studio
on the left/central half.

## Controller revision

The original config used `board: nice_nano`, which maps to **nice!nano V1**.
Therefore `build.yaml` uses:

    nice_nano@1//zmk

If the physical controllers are nice!nano V2, replace every occurrence with:

    nice_nano//zmk

## Thumb positions and modifiers

Physical thumb positions:

    left:   30 outer | 31 middle | 32 inner
    right:  33 inner | 34 middle | 35 outer

Current behavior:

- left outer hold (30): EXTRA; tap: Enter
- left middle hold (31): SIGNS; tap: Space
- left inner (32): sticky Ctrl
- right inner (33): sticky Shift
- right middle hold (34): NUMMY; tap: Esc
- right outer hold (35): NAV; tap: Tab
- 30+31: sticky Alt / macOS Option
- 31+32: sticky Shift
- D+F+left-middle-thumb (12+13+31): normal held GUI / Win / Command (not sticky)
- 34+35: one-shot POLISH layer
- 30+31+32: toggle GAME

Sticky Ctrl/Alt/Shift use a 500 ms release timeout. The GUI combo is intentionally non-sticky; combo timeout is 80 ms.

## OS modes

Windows is the baseline shortcut mapping. Linux and macOS activate transparent
state layers which only override shortcuts that differ.

Bluetooth host macros set the profile, BLE output, OS mode and Unicode mode:

- BT0 -> Windows
- BT1 -> Linux
- BT2 -> macOS
- BT3 -> Windows
- BT4 -> Linux

BLUE also contains a dedicated **USB Linux** key. It selects USB output and
Linux mode together so switching back from the Mac to the Ubuntu machine cannot
leave the keyboard in macOS shortcut mode.

Raw BLE and output-toggle keys remain available as advanced controls; they only
change the endpoint, not the OS mode. Prefer the host macros for normal switching.

## Entering layers

- EXTRA: hold left outer thumb
- SIGNS: hold left middle thumb
- NUMMY: hold right middle thumb
- NAV: hold right outer thumb
- BLUE: hold NAV, tap the top-left key; BLUE applies to the next key
- POLISH: press right middle + right outer thumbs together; POLISH applies to the next key
- GAME: press all three left thumbs together to toggle

## EXTRA — semantic commands

Unused positions are `&none` intentionally. EXTRA contains no media controls,
Polish hold-taps or duplicate HJKL navigation.

Physical base-key positions:

    Q       W             E             R             T       | Z      U      I       O      P
    none    Close Tab     App Switch    Tab Switch    New Tab | none   none   Zoom+   none   Print
            Shift:        Shift:        Shift:
            Reopen        reverse       reverse

    A       S             D             F             G       | H      J      K       L      BSPC
    All     Save          none          Find          Undo    | none   none   none    Lock   Backspace
                                                  Shift: Redo

    Y       X             C             V             B       | N      M      ,       .      -
    none    Cut           Copy          Paste         Shot    | none   none   none    none   Zoom-

OS-specific output:

| Action | Windows | Ubuntu / GNOME | macOS |
| --- | --- | --- | --- |
| Close tab | Ctrl+W | Ctrl+W | Cmd+W |
| Reopen tab | Ctrl+Shift+T | Ctrl+Shift+T | Cmd+Shift+T |
| App switch | Alt+Tab | Super+Tab | Cmd+Tab |
| Reverse app switch | Alt+Shift+Tab | Super+Shift+Tab | Cmd+Shift+Tab |
| Next tab | Ctrl+Tab | Ctrl+Tab | Ctrl+Tab |
| Previous tab | Ctrl+Shift+Tab | Ctrl+Shift+Tab | Ctrl+Shift+Tab |
| New tab | Ctrl+T | Ctrl+T | Cmd+T |
| Select all | Ctrl+A | Ctrl+A | Cmd+A |
| Save | Ctrl+S | Ctrl+S | Cmd+S |
| Find | Ctrl+F | Ctrl+F | Cmd+F |
| Print | Ctrl+P | Ctrl+P | Cmd+P |
| Copy | Ctrl+C | Ctrl+C | Cmd+C |
| Cut | Ctrl+X | Ctrl+X | Cmd+X |
| Paste | Ctrl+V | Ctrl+V | Cmd+V |
| Undo | Ctrl+Z | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Y | Ctrl+Shift+Z | Cmd+Shift+Z |
| Lock | Win+L | Super+L | Ctrl+Cmd+Q |
| Screenshot | Win+Shift+S | Print Screen | Cmd+Shift+3 |
| Zoom in | Ctrl++ | Ctrl++ | Cmd++ |
| Zoom out | Ctrl+- | Ctrl+- | Cmd+- |

Note: Redo is application-dependent, especially on Windows/Linux. The mappings
above are the broadest conventional choices. Terminal applications can also
assign special meanings to Ctrl-based shortcuts; the semantic layer targets
normal desktop/application editing behavior.

## NAV

NAV remains the media/navigation layer and is intentionally not stripped of
media controls.

    BLUE   Vol-   Mute   Vol+   PgUp    | F1   F2   F3   F4   F5
    All    Cut    Copy   Paste  Undo    | Left Down Up   Right Delete
    Paste  Prev   Play   Next   Shot    | F6   F7   F8   F9   F10

On macOS, the NAV editing row is overridden to Cmd+A/X/C/V/Z. Linux inherits
normal Ctrl+A/X/C/V/Z. Media controls remain HID media controls on every OS.

## POLISH

POLISH is a true one-shot layer again; there are no Polish long-presses on EXTRA.
Press right-middle + right-outer thumbs (34+35), release, then press the letter.

    E -> ę    Z -> ż    O -> ó
    A -> ą    S -> ś    L -> ł
    X -> ź    C -> ć    N -> ń

Shift selects uppercase through the Unicode behavior.

Polish uses `urob/zmk-unicode` and the OS host macros select the matching mode:

- Windows: `UC_SET_WIN_COMPOSE` — requires WinCompose on the host
- Linux: `UC_SET_LINUX`
- macOS: `UC_SET_MACOS` — requires Unicode Hex Input to be enabled

## BLUE / Hosts

Enter BLUE with NAV + top-left. The relevant rows are:

    BT0 Win | BT1 Linux | BT2 Mac | BT3 Win | BT4 Linux | ...

    BT Clear | Windows mode | Linux mode | macOS mode | USB Linux |
    BLE raw  | Toggle raw   | Studio unlock | ...

Recommended normal workflow:

- Mac: use **BT2 Mac** — selects profile 2 + BLE + macOS mode
- Ubuntu by cable: use **USB Linux** — selects USB + Linux mode
- Windows Bluetooth: use **BT0 Win** or **BT3 Win**

## Build / Studio

`.github/workflows/build.yml` invokes ZMK's reusable user-config build workflow.
The left build enables Studio via `studio-rpc-usb-uart` and
`CONFIG_ZMK_STUDIO=y`.

After Studio stores runtime keymap changes, later stock `.keymap` edits may not
appear until **Restore Stock Settings** is used in ZMK Studio.

## First flash after a settings reset

1. Build the repository in GitHub Actions.
2. Flash `settings-reset.uf2` to both halves once.
3. Flash `skeletyl-left.uf2` to the left/central half.
4. Flash `skeletyl-right.uf2` to the right half.
5. Forget/re-pair Bluetooth hosts if the settings reset invalidated pairings.

For ordinary future keymap-only changes, flashing the central/left half is
normally sufficient.

## 2026-08-30 QC update: native app/tab switchers

- `EXTRA + C` is Copy (`Ctrl+C` on Windows/Linux, `Cmd+C` on macOS). `EXTRA + Y` is intentionally unused.
- `EXTRA + E` is implemented as a press/release macro, not a toggle. On key press it presses the OS app-switch modifier and taps Tab; on E release it releases the modifier. Therefore:
  - quick tap E -> switch one app and commit immediately;
  - hold E -> keep the native app switcher open;
  - while holding E, tap the physical right-outer Tab thumb to continue cycling;
  - hold physical Shift while cycling to go backwards;
  - release E -> commit/select because Alt/Super/Command is released.
- `EXTRA + R` uses the identical model with Ctrl for tab switching. Quick tap R switches once; hold R and tap the physical Tab thumb to cycle; release R to commit.
- No key-toggle state remains in the app/tab switchers, and EXTRA no longer has to clean up modifiers on release.
- The D+F+Space GUI combo is now a normal `LGUI` key instead of sticky GUI. Tap the chord to open Start/Activities immediately; keep the chord held if you need a physically held Win/Super/Command modifier. This removes the 500 ms sticky timeout from standalone GUI presses.
- Polish remains on the modifier-aware right-middle + right-outer one-shot combo. The right-inner Shift thumb remains tap=sticky Shift / hold=physical Shift, including the Polish uppercase handling.
