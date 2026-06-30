# zmk-config — Ferris Sweep (Colemak-DH)

ZMK firmware for a **Ferris Sweep** (34 keys, Bluetooth, **nice!nano v2** per half),
porting the kanata Colemak-DH config from
[`kamilradziszewski/dotfiles`](https://github.com/kamilradziszewski/dotfiles) (`dot_config/kanata/`).

The **keyboard emits Colemak-DH letters directly** — keep macOS on **US-QWERTY**,
exactly like the kanata setup. The macOS side (Raycast, Leader-key, all shortcuts)
is unchanged; the keyboard just sends the same chords.

> **Shield = `cradio`.** In ZMK the classic Ferris Sweep is the `cradio` shield.
> If you actually have a **splitkb.com “Aurora Sweep”**, change `cradio_left` /
> `cradio_right` to `splitkb_aurora_sweep_left` / `_right` in
> [`build.yaml`](build.yaml) and rename `config/cradio.*` to `config/splitkb_aurora_sweep.*`.

---

## 1. Build (GitHub Actions — no local toolchain)

1. Create a new GitHub repo named `zmk-config` (empty, no README).
2. Push this folder to it:
   ```sh
   cd ~/zmk-config
   git init -b main
   git add .
   git commit -m "feat: initial Ferris Sweep Colemak-DH keymap"
   git remote add origin git@github.com:kamilradziszewski/zmk-config.git
   git push -u origin main
   ```
   *(Use the `kamilradziszewski` account — switch with `gh auth switch -u kamilradziszewski` / `ghsw` if needed.)*
3. Open the repo’s **Actions** tab. Each push runs **“Build ZMK firmware”** and
   produces a `firmware` artifact containing **two** files:
   `cradio_left-nice_nano_v2-zmk.uf2` and `cradio_right-nice_nano_v2-zmk.uf2`.

## 2. Flash (drag-and-drop, no CLI)

For **each half**:
1. Plug the half into USB.
2. Double-tap the **reset** button → it mounts as a USB drive named `NICENANO`.
3. Drag the matching `*_left*.uf2` / `*_right*.uf2` onto the drive. It reboots automatically.

**First time only:** after flashing both halves, also flash ZMK’s
[`settings_reset`](https://zmk.dev/docs/troubleshooting/connection-issues#reset-procedure)
firmware to each half if the halves don’t pair, then re-flash the normal firmware.
Pair to macOS via **Bluetooth** (the **left** half is the central that talks to the Mac).

## 3. Edit with a GUI

- **Primary — [Nick Coutsos keymap-editor](https://nickcoutsos.github.io/keymap-editor/)**
  (web): authorize it against this `zmk-config` repo and it edits `config/cradio.keymap`
  visually — **including** combos, hold-taps, mod-morphs, and macros — committing
  straight back to GitHub (which re-triggers the build). This is the recommended tool.
- **Optional — [ZMK Studio](https://zmk.studio)**: live key-swaps over USB, but it
  **only reassigns existing behaviors** — it can’t create combos / hold-taps /
  mod-morphs / macros. Enable it via the commented block in `build.yaml`.

---

## Layer reference

| # | Layer | Activation | What it does |
|---|-------|-----------|--------------|
| 0 | Base | always | Colemak-DH. Top-row mod-taps (⌃⌘⌥⇧ mirrored). Thumbs: LO=⌫/NumSym, LI=Space/⌘, RI=⏎/⇧, RO=Tab/Nav. Holds: A→Arrows, V→Func, M→Apps, ;→Digits. **H+J = Esc**. `,`→⇧`'`, `.`→⇧`"`. |
| 1 | NumSym | hold LO (⌫) | Digits 1–0 (mirrored mod-taps), `[ ]` `( )`(⇧`< >`), `` ` `` , `;`, `= -`(⇧`_ +`), `\`. |
| 2 | Nav | hold RO (Tab) | Left = Raycast window mgmt (meh/hyper combos). Right = vim arrows + Home/End/PgUp/PgDn + browser history/tab nav. |
| 3 | Func | hold V | F1–F12 (mirrored mod-taps), display + ext-display brightness, play/pause, volume. |
| 4 | Arrows | hold A | H J K L = ←↓↑→; browser history/tab nav + Esc on the left. |
| 5 | Apps | hold M | Every alpha = `meh`+letter → Raycast app launch. Thumbs: Leader (⌃F5), Raycast (⌃F6), ⌘. |
| 6 | Digits | hold ; | Number row over the home block. |

Full per-key grids are in [`config/cradio.keymap`](config/cradio.keymap).

---

## Known differences from the kanata config

These are deliberate adaptations where ZMK and kanata don’t map 1:1. Tune in the
web editor.

- **Hold-tap feel.** kanata `tap-hold` → ZMK `tap-preferred` (`&ht`); kanata
  `tap-hold-release` → ZMK `balanced` (`&lb`). Timings start at
  `tapping-term-ms = 200`, `quick-tap-ms = 175`, `require-prior-idle-ms = 150`.
  Top-row mods are sensitive during fast typing — raise `require-prior-idle-ms`
  if you get accidental modifiers. The **alpha layer-holds** (A→Arrows, V→Func,
  M→Apps, ;→Digits) use the `&lb` balanced flavor, so typing a word with a quick
  key right after `a`/`d`/`h`/`o` can occasionally trigger the layer. If that
  bothers you, give those four keys a *separate* hold-tap with
  `require-prior-idle-ms` and/or `hold-trigger-key-positions` (positional
  hold-tap) — but keep the **thumb** `&lb` without prior-idle so thumb layers
  still fire mid-burst.
- **Apps / Digits are momentary, not one-shot.** kanata used a 3 s one-shot
  layer; ZMK sticky-layers only last for the *next single key*, which breaks
  multi-digit entry. So these are plain **hold-while-typing** layers (hold M or
  `;` while pressing the target key). If you prefer tap-to-activate for Apps,
  swap `&lb APPS H` for a `&sl`-based behavior in the keymap.
- **`next app window`** is mapped to **`Cmd+\``** (`LG(GRAVE)`), the real macOS
  shortcut. kanata sent `M-nubs`; if your setup expects the ISO key, change it to
  `LG(NON_US_BSLH)`.
- **`` ` `` on NumSym phys-L** is `GRAVE` (the documented intent). kanata sent
  `nubs`; switch to `&kp NON_US_BSLH` if you really want the ISO key.
- **Keyboard backlight keys dropped.** kanata’s `bldn`/`blup` (MacBook internal
  keyboard backlight) have **no external-HID equivalent** — those two Func-layer
  keys are `&none`. (Display brightness *does* work via `C_BRI_DN/UP`.)
- **Right-hand mods use R-variants** (`RSHFT`/`RCTRL`/…) per ZMK convention;
  functionally identical to the left-hand mods on macOS.
- **Arrows layer** keeps kanata’s `phys P = ⌃`, `phys E/I = ⌥` workaround. The
  Sweep matrix has no conflict, so you can simplify these if you like.
- **Thumb order.** Bindings assume key-positions `30 31 32 33` =
  left-outer, left-inner, right-inner, right-outer (LO/LI/RI/RO). If a hand’s two
  thumb keys feel swapped, swap those two bindings in each layer (30↔31 and/or
  32↔33) — it’s the most build-dependent detail.
