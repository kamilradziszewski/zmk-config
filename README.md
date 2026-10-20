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
   `cradio_left.uf2` and `cradio_right.uf2`.

`config/west.yml` and `.github/workflows/build.yml` are pinned to the same ZMK
`main` commit (`4dd95d7840eb3d5a3f59bb29fbf1ab054715ef85`) so the ZMK source
and build workflow stay on the same Zephyr 4.1 snapshot. This is a development
snapshot, not a stable release; update both pins together when choosing another
ZMK version. The nice!nano v2 target is `nice_nano@2.0.0//zmk`; the in-tree
`cradio` shield is unchanged.

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
  (web): authorize it against this `zmk-config` repo to edit ordinary key
  assignments and inspect the layers. This keymap also contains custom
  positional hold-taps and release-aware macros; review editor changes carefully
  to ensure those behavior definitions are preserved.
- **Optional — [ZMK Studio](https://zmk.studio)**: live key-swaps over USB, but it
  **only reassigns existing behaviors** — it can’t create combos / hold-taps /
  mod-morphs / macros. Enable it via the commented block in `build.yaml`.

---

## Layer reference

| # | Layer | Activation | What it does |
|---|-------|-----------|--------------|
| 0 | Base | always | Colemak-DH with bilateral opposite-hand home-row mods. Thumbs: LO=Backspace, LI=Space/Nav, RI=Enter/NumSym, RO=Tab/Forward Delete. Base-only modifier combos; **G+H = Esc**. `,`→⇧`'`, `.`→⇧`"`. |
| 1 | NumSym | hold RI from Base | Digits 1–0, coding symbols, and shifted punctuation forks. |
| 2 | Navigation | hold LI from Base | Window placement, browser history/tab shortcuts, arrows, display/media controls, Caps Word. Q enters FKeys. |
| 3 | Apps | hold physical M from Base | One-shot (3 s) layer: each letter sends `meh`+its Colemak-DH output. Thumbs: Leader (⌃F5), Raycast (⌃F6), ⌘. |
| 4 | FKeys | Q from Navigation | F1–F12 with modifier keys; an F-key returns to Base on release; the layer expires after 3 s idle. |
| 5 | Forward Delete | hold RO, then tap LI | Sends Forward Delete; all other keys pass through. |

Full per-key grids are in [`config/cradio.keymap`](config/cradio.keymap).

---

## Known differences from the kanata config

These are deliberate adaptations where ZMK and kanata don’t map 1:1. Test the
timing on the keyboard before relying on the new firmware.

- **Home-row mods.** ZMK uses balanced positional hold-taps with
  `hold-trigger-key-positions`, `hold-trigger-on-release`,
  `tapping-term-ms = 300`, and `require-prior-idle-ms = 150` to approximate
  Kanata’s opposite-hand-release behavior. Kanata’s `neutral-keys` rules have no
  exact ZMK equivalent, so mod rolls and same-hand typing need testing.
- **Thumb timing.** Kanata separates a 150 ms tap term from a 200 ms hold term.
  ZMK hold-taps use one term; the thumbs use 200 ms to preserve the hold timing.
- **One-shot layers and shift.** Apps uses ZMK's `&sl` sticky layer with a 3 s
  timeout; the A/semicolon shift holds use sticky Shift with a 500 ms timeout
  and quick-release.
- **FKeys idle timeout.** FKeys uses a 3 s sticky-layer timeout, re-armed by
  modifier and otherwise-unused key presses. Each F-key macro holds the key
  until physical release, then switches to Base. This approximates Kanata’s
  physical-idle timer; a modifier held for more than 3 s may still let the ZMK
  timeout expire.
- **Modifier combos.** The ten Base-only modifier pairs use ZMK combos with
  `slow-release`, so the modifiers remain held until both combo keys are
  released. Their 25 ms window matches Kanata’s configured chord timeout.
- **`next app window`** is mapped to **`Cmd+\``** (`LG(GRAVE)`), the real macOS
  shortcut. kanata sent `M-nubs`; if your setup expects the ISO key, change it to
  `LG(NON_US_BSLH)`.
- **Keyboard backlight keys dropped.** Kanata’s `bldn`/`blup` (MacBook internal
  keyboard backlight) have no external-HID equivalent; those Navigation-layer
  positions are blocked. Display brightness still uses `C_BRI_DN/UP`.
- **Right-hand modifiers** use ZMK’s `RSHFT`/`RCTRL`/`RALT`/`RGUI` variants,
  matching the Kanata profile.
- **Thumb order.** Bindings assume key-positions `30 31 32 33` =
  left-outer, left-inner, right-inner, right-outer (LO/LI/RI/RO). If a hand’s two
  thumb keys feel swapped, swap those two bindings in each layer (30↔31 and/or
  32↔33) — it’s the most build-dependent detail.
