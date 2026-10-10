# W7YFR fork changelog

Changes in this fork compared with upstream [K3NG](https://github.com/k3ng/k3ng_cw_keyer) `master`. Upstream's
own history is in the comments at the top of `k3ng_keyer/k3ng_keyer.ino`.

This fork is the keyer firmware for the [Stellaluna Keyer](https://github.com/W7YFR/stellaluna) (Arduino Mega 2560,
SSD1306 OLED, and an ESP32 VBand adapter).

## Using these changes in your own build

Each fix and feature is on its own branch, cut from upstream `master` (or stacked on another branch it needs), and
behind its own `FEATURE_` / `OPTION_` define where it adds behavior. Merge or cherry-pick the branch you want; you
don't need anything else from this fork.

`w7yfr` is this build's own branch: every branch below merged together, plus this build's configuration (pins,
which features are on) and a few changes made directly on it. Don't merge `w7yfr` itself unless you want this exact
build; the "only on `w7yfr`" entries list commit hashes to cherry-pick instead.

| Branch | Based on | What it adds |
|---|---|---|
| `fix/oled-nonblocking-redraw` | `master` | Display redraws no longer block paddle handling |
| `feature/oled-font-select` | `fix/oled-nonblocking-redraw` | Selectable SSD1306 font |
| `fix/i2c-wire-timeout` | `fix/oled-nonblocking-redraw` | A stuck I2C bus no longer freezes the keyer |
| `feature/vband-link` | `feature/oled-font-select` | `FEATURE_VBAND_LINK`: the link to the ESP32 VBand adapter |
| `feature/settings-menu` | `feature/vband-link` | `FEATURE_SETTINGS_MENU`: settings menu, shortcuts and CLI |
| `fix/exclamation-mark` | `master` | `!` sent and decoded as `-.-.--` |
| `fix/punctuation-decode` | `fix/exclamation-mark` | `;` `$` `'` `"` `_` `)` decode from the paddles |
| `feature/extra-button` | `master` | `EXTRA_BUTTONS`: buttons beyond the command and memory buttons |
| `feature/button-press-duration` | `master` | `button_hold_threshold_ms`: press vs hold on the buttons |
| `feature/oled-screen-controls` | `master` | SSD1306 display controls, scroll reset |
| `feature/memory-tweaks` | `master` | Memory recording and playback improvements |
| `feature/clear-display-after-memory` | `feature/memory-tweaks` + `feature/oled-screen-controls` | Clear the display after a memory plays |
| `feature/dit-hold-clear-reset` | `feature/memory-tweaks` + `feature/oled-screen-controls` | `FEATURE_DIT_HOLD_RESET`: clear or reset with a run of dits |
| `feature/paddle-echo-practice` | `master` | Echo practice from the paddles, no serial terminal |
| `fix/echo-practice` | `master` | `%` sent and tracked in echo practice |
| `feature/straight-key-paddle` | `master` | `FEATURE_STRAIGHT_KEY_PADDLE`: the dit paddle as a straight key |
| `feature/farnsworth-potentiometer` | `master` | `FEATURE_FARNSWORTH_POTENTIOMETER`: a pot for the Farnsworth speed |
| `fix/prosign-forward-declaration` | `master` | Builds outside the Arduino IDE (`convert_prosign`) |
| `build/platformio` | `master` | PlatformIO builds |
| `chore/build-warnings` | `master` | Compiler warning cleanup |

Below, newest first, each entry names its branch. Dates are when the work was merged into `w7yfr`.

## Unreleased

## 2026-10-05: settings menu and punctuation

### Added
- `feature/settings-menu`: `FEATURE_SETTINGS_MENU`, settings by name in three ways:
  - a menu in command mode (`/` and a pause): dit moves down, dah up, `R` selects, toggles or saves, `B` or
    "< Back" goes back, `X` leaves command mode;
  - keyed shortcuts such as `/KY WPM 22`;
  - the CLI: `\$ ky.wpm 22`.
- `feature/settings-menu`: keyer settings (`ky.*`, `st.*`): mode, speed, Farnsworth, weighting, ratio, paddle
  reverse, dit/dah memory, paddle echo, autospace, memory repeat, TX line, PTT lead/tail, sidetone pitch and mode.
  Each one appears only when the build has that feature or pin. Paddle echo is saved, without moving the memories.
- `feature/settings-menu`: the VBand adapter's settings and commands in the same menu, read from the adapter over
  the link (needs `FEATURE_VBAND_LINK`): Commands first, then Settings; Connect or Disconnect, whichever applies,
  first; running a command returns to the same row.

### Fixed
- `fix/exclamation-mark`: `!` is sent and decoded as `-.-.--` (it was sent as a comma).
- `fix/punctuation-decode`: `;` `$` `'` `"` `_` `)` decode from the paddles.

### Build config (`w7yfr`)
- `FEATURE_SETTINGS_MENU` on.

## 2026-10-04: VBand link

### Added
- `feature/vband-link`: `FEATURE_VBAND_LINK`, a serial link (Serial2, 38400 baud) to an ESP32 VBand adapter
  ([W7YFR/k3ng-vband-adapter](https://github.com/W7YFR/k3ng-vband-adapter)):
  - Keying moves to the VBand line (`tx_key_line_2`) while the adapter reports VBand is ready, and back to the radio
    when it isn't. It switches only between characters and isn't saved.
  - The adapter's status screens on the display (WiFi, portal, connecting, the channel and who's in it, OTA), and
    its system lines.
  - Other users' sending, decoded by the adapter, shown with a tag for each sender; your own sending shows under
    your tag.
  - Adapter commands from command mode (`/WHO`, `/CH`, `/OTA`, ...). Unknown words are checked with the adapter
    first and leave you in command mode to try again; answers show until they time out.
  - The link protocol version is checked both ways, with a "Link Mismatch" warning.
  - The link stays alive in command mode.

### Fixed
- `fix/i2c-wire-timeout`: a stuck I2C bus no longer freezes the keyer (Wire timeout).
- `feature/extra-button`: extra buttons are no longer treated as memory buttons (one said "memory empty").
- Only on `w7yfr` (`04048b1`): the next VBand line is re-tagged after the display is cleared.

### Build config (`w7yfr`)
- 6 memory buttons and 1 extra button.

## 2026-10-03: OLED and the VBand build setup

### Added
- `feature/oled-font-select`: selectable SSD1306 font.

### Fixed
- `fix/oled-nonblocking-redraw`: the display redraws without blocking paddle handling.

### Build config (`w7yfr`)
- `X11fixed7x14B` OLED font (18 x 4).
- `FEATURE_VBAND_LINK` on; VBand key line (`tx_key_line_2`) on D7.
- Speed pot on A2, Farnsworth pot on A0; speed pot range starts at 5 WPM; `FEATURE_FARNSWORTH_POTENTIOMETER` on.
- `FEATURE_ROTARY_ENCODER` and `FEATURE_CW_DECODER` off.

## 2026-09: straight key, Farnsworth pot, buttons and PlatformIO

### Added
- `feature/straight-key-paddle`: `FEATURE_STRAIGHT_KEY_PADDLE`, use the dit paddle as a straight key in straight
  mode (works with `FEATURE_STRAIGHT_KEY_ECHO`).
- `feature/farnsworth-potentiometer`: `FEATURE_FARNSWORTH_POTENTIOMETER`, a dedicated pot for the Farnsworth
  character speed (`farnsworth_potentiometer` in the pin settings); clearer effective vs character WPM labels.
- `feature/extra-button`: `EXTRA_BUTTONS` / `NUMBER_OF_EXTRA_BUTTONS`, buttons on the analog ladder beyond the
  command and memory buttons, with press, hold and command actions (see `check_buttons()`).
- `feature/button-press-duration`: `button_hold_threshold_ms`, telling a press from a hold.
- `build/platformio`: PlatformIO builds.

### Fixed
- `fix/prosign-forward-declaration`: `convert_prosign` is forward-declared, so builds outside the Arduino IDE
  compile.
- `build/platformio`: `lcd_center_print_timed_wpm` and `NETWORK_CLIENT_CLS` build errors outside the Arduino IDE.
- `feature/extra-button`, `feature/button-press-duration`: the new options are defined in every hardware board
  profile, so the other boards still compile.

### Build config (`w7yfr`)
- `FEATURE_FARNSWORTH`, `FEATURE_STRAIGHT_KEY_PADDLE` (+ echo) on; extra buttons enabled.

## 2026-08: echo practice without a terminal

### Added
- `feature/paddle-echo-practice`: paddle-only QSO echo practice, no serial terminal needed; the target word shows
  on the display and each attempt gets its own line.
- `feature/paddle-echo-practice`: `OPTION_ECHO_PRACTICE_DOUBLE_CORRECT_AFTER_MISS`.

### Fixed
- `fix/echo-practice`: `%` is actually sent and tracked.
- `feature/paddle-echo-practice`: unpassable `<` / `>` prosign entries removed from the QSO word lists.
- `chore/build-warnings`: compiler warnings cleaned up.
- Only on `w7yfr` (`b970e1e`): practice doesn't exit unless asked to.
- Only on `w7yfr` (`bbdc0e7`): the rotary encoder pins aren't configured when set to 0 (pin 0 conflicted with RX).

### Build config (`w7yfr`)
- Terminal-free echo practice with repeat on.

## 2026-07: buttons, display and memories

### Added
- `feature/memory-tweaks`: `OPTION_SKIP_SAVED_MEMORY_PLAYBACK` (skip playback after recording a memory); press the
  same memory button to restart recording; characters show while a memory plays.
- `feature/oled-screen-controls`: SSD1306 display controls; scroll position reset.
- `feature/clear-display-after-memory`: clear the display after a memory plays.
- `feature/dit-hold-clear-reset`: `FEATURE_DIT_HOLD_RESET`, clear or reset the display with a run of dits.
- Only on `w7yfr` (`bb64cea`): the extra button toggles or clears the display.

### Fixed
- `feature/memory-tweaks`: memories are saved at the configured speed.
- `feature/memory-tweaks`, `feature/dit-hold-clear-reset`: the new options are defined in every hardware board
  profile.
- Only on `w7yfr` (`7ba9f25`): only the first button triggers the display reset.

### Build config (`w7yfr`)
- Mega configuration and a PlatformIO Mega build target; paddle echo, skip playback and dit-hold reset on; a CW
  decoder tried and later turned off.

## 2025-11: first setup

### Build config (`w7yfr`)
- Mega configuration and a callsign greeting.
