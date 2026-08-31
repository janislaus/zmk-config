# Switcher v6

The semantic EXTRA switchers intentionally use only normal ZMK hold-tap and key-press behaviors.

- EXTRA+E tap: one app switch (Windows Alt+Tab, Ubuntu/GNOME Super+Tab, macOS Cmd+Tab).
- EXTRA+E hold + right outer thumb: hold the OS modifier and send a *plain* Tab immediately; repeat the thumb to cycle.
- Hold Shift while pressing the Tab thumb to cycle backwards.
- Release E to release the app-switch modifier and select.
- EXTRA+R tap: one Ctrl+Tab.
- EXTRA+R hold + right outer thumb: Ctrl remains physically held; repeat Tab to cycle; release R to select.

The key detail versus v5 is that the right outer thumb on EXTRA is `&kp TAB`, not `&lt NAV TAB`, so the switcher path contains only one hold-tap decision (E or R) and cannot be blocked by a nested layer-tap on Tab.
