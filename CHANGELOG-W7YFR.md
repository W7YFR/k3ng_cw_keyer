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
| `feature/display-msg-delay-setting` | `master` | `default_display_msg_delay` can be set in a settings file |
| `feature/build-revision` | `master` | `FIRMWARE_GIT_REV`: the startup screen shows the commit a build came from |
| `fix/async-eeprom-write-overrun` | `master` | The background settings write no longer writes one byte past the settings |
| `fix/straight-key-echo-without-cli` | `master` | Straight key echo to the display builds without the CLI |
| `fix/prosign-sos-and-warnings` | `master` | SOS decodes to its letters; paddle reads always return; compiler warnings |
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
| `feature/farnsworth-effective-speed` | `feature/farnsworth-potentiometer` | `OPTION_FARNSWORTH_SETS_EFFECTIVE_SPEED`: the speed is the character speed, Farnsworth the effective speed |
| `feature/winkey-farnsworth` | `feature/farnsworth-effective-speed` | Winkey hosts' speed and Farnsworth translated under `OPTION_FARNSWORTH_SETS_EFFECTIVE_SPEED` |
| `feature/usb-port-mode` | `feature/settings-menu` | Device > USB port: CLI or Winkey, saved, switched without a power cycle |
| `fix/speed-pot-at-startup` | `master` | The speed pot's position sets the speed at power-on |
| `fix/farnsworth-pot-at-startup` | `feature/farnsworth-potentiometer` | The Farnsworth pot's position sets the Farnsworth speed at power-on |
| `fix/power-on-serial-button` | `master` | The command button held at power-on picks the other USB port mode again |
| `fix/prosign-forward-declaration` | `master` | Builds outside the Arduino IDE (`convert_prosign`) |
| `fix/platformio-forward-declarations` | `fix/prosign-forward-declaration` | More functions declared ahead, so other boards' profiles build outside the Arduino IDE |
| `build/platformio` | `master` | PlatformIO builds |
| `chore/build-warnings` | `master` | Compiler warning cleanup |

Below, newest first, each entry names its branch. Dates are when the work was merged into `w7yfr`.

## Unreleased

### Added
- `feature/usb-port-mode`: a USB port setting, CLI or Winkey (Device > USB port, `/KY USB WINKEY`,
  `\$ ky.usb winkey`), in builds with both `FEATURE_WINKEY_EMULATION` and `FEATURE_COMMAND_LINE_INTERFACE`.
  Changing it switches the port straight away: from the menu at once, from the CLI once its answer is out (at the
  old baud rate). Leaving Winkey mode stops a host's message and closes its session. It's saved (the top two bits of
  the extras byte after the configuration) and decides the mode at power-on; until it's first saved,
  `OPTION_PRIMARY_SERIAL_PORT_DEFAULT_WINKEY_EMULATION` decides, as before. The command button held at power-on
  still gives the other mode for that session. The branch adds the Device list to the menu's top level (the
  `w7yfr` build already has it, for Dit mash). Only on `w7yfr` (`b505cf1`): Winkey emulation enabled, and
  `OPTION_PRIMARY_SERIAL_PORT_DEFAULT_WINKEY_EMULATION` off, so the port starts as the CLI until set to Winkey.
- `feature/winkey-farnsworth`: with `OPTION_FARNSWORTH_SETS_EFFECTIVE_SPEED`, Winkey hosts' speeds are translated.
  A host means K1EL's speeds (its speed overall, a Farnsworth speed above it the character speed); the keyer keeps
  the character speed as its speed and the overall speed as Farnsworth. A host's "speed 10, Farnsworth 25" now sends
  characters at 25 and overall at 10, and get values reads back what the host set. Covered: unbuffered and buffered
  speed (and cancelling or clearing back to it), Farnsworth (0 turns it off), HSCW, load defaults, get values, the
  EEPROM download and every speed report. A host knows one speed, the overall speed, so that's what reports
  carry. With the option, both pots set the keyer's speeds while a host is open too, as a WinKeyer's pot does (the
  speed pot the character speed, the Farnsworth pot the overall speed), and the host is told the overall speed
  whenever it changes; a host's own speed commands apply until a pot moves. K3NG's speed pot only reported while a
  host was open and waited for the host to set the speed, which broke with the Farnsworth pot setting a speed of its
  own. Reports are kept within the pot's range (a speed below it read as a status byte). The host's Farnsworth speed
  is kept while it's at or below the speed, so lowering the speed below it turns Farnsworth back on, as on a
  WinKeyer, and a host's speed with Farnsworth off leaves the keyer's own Farnsworth setting alone when it's at or
  above that speed (off anyway). Without the option nothing changes, except that speed reports are kept within
  the pot's range.
- `feature/vband-link`: the adapter's log on the keyer's USB port. With the adapter's `vb.log` on, each line it sends
  (`LG` / `LGP` frames) is printed with an `ESP> ` prefix, in any mode. Never waits on the port (a line that doesn't
  fit is dropped), and nothing is printed while the port is in Winkey mode.
- `feature/settings-menu`: each list starts with "< Back" ("< Exit" on the top level, which leaves the menu and
  command mode) and opens on its second row, with the first scrolled off the top when the rest fills the screen; a
  dah scrolls it into view.
- `feature/vband-link`: an interrupted sender's line repeats their message so far. When someone else's sending (or a
  join or leave line) cuts into a message, the sender's next line starts with what they'd sent, the last two display
  lines' worth, then carries on ("A>HEL", "B>HI", "A>HELLO"). Kept for the last 3 senders, your own keying included;
  5 s of quiet ends a message. Clearing the display forgets it. Normal sending checked on hardware; the repeat itself
  hasn't been seen on air yet.
- `feature/vband-link`: `STU` frames update the adapter's status screen while it's still showing (nothing else has
  taken the display, and you haven't keyed over it), so the adapter's connecting screen stays up with a clock.
- `feature/settings-menu`: VBand > Settings has an LED list. An adapter setting can sit in a Commands list (a
  lower-case group letter): the channel selector, in Commands > Rooms. R takes a command the adapter offers with an
  answer (`OF`): Who's here off VBand shows "R: Connect". In a list of options, dit goes to the next one and dah back,
  as rows do; numbers are unchanged.
- `feature/farnsworth-effective-speed`: `OPTION_FARNSWORTH_SETS_EFFECTIVE_SPEED` turns Farnsworth the other way round.
  The speed is the character speed, which the paddles always send at, and the Farnsworth setting (menu, `\M`, or
  its pot) a slower effective speed that memories and buffered sending stretch their spacing to; it's off at 0 or at
  or above the speed. Without the option, K3NG's way is unchanged (`farnsworth_speeds()` gives its answer). The CLI,
  help, web page and pot message name the speeds to match, and the pot shows "Farnsworth off" when it has no effect.
  The option is listed, off, in every `keyer_features_and_options` file. Winkey hosts are translated on
  `feature/winkey-farnsworth`. Only on `w7yfr`
  (`168aca8`): enabled.
- `feature/build-revision`: a build given `-D FIRMWARE_GIT_REV=<commit>` shows it on the startup screen in place of
  `CODE_VERSION` ("V: 4f3f238", with `+` for uncommitted changes) and after `CODE_VERSION` in the serial boot
  message; without it nothing changes. Only on `w7yfr` (`a4f836c`): `platformio.ini` passes it from
  `git describe --always --dirty=+`.
- `feature/display-msg-delay-setting`: a settings file can set `default_display_msg_delay`, how long a timed
  display message (a command mode confirmation, say) shows; the default stays 1000 ms. Only on `w7yfr` (`3fd2802`):
  set to 3000 ms.
- `feature/settings-menu`: the menu is grouped. The top level opens VBand, Keying, Sidetone and Transmit; Keying
  opens Speed (Character, Farnsworth, Cmd speed, Speed pot) above Mode, Weight and the rest; Transmit has the
  transmitter, PTT lead and tail and memory repeat. A row that opens a list shows `>`. The keyed and typed names
  don't change (`/KY WPM`, `ky.wpm`); the WPM setting is labelled "Character".
- `feature/settings-menu`: VBand's Commands and Settings show their most-used rows (Connect or Disconnect, Who's
  here) above the lists Rooms, Sound, Display and Device, which hold the rest; a list shows only when it has
  something in it, and back returns to your row at every depth. Link protocol 2 (the adapter's `SI` items carry the
  list they go in): update the adapter with it.
- `feature/settings-menu`: a colon after each setting's name ("Mode: Iambic B"), so a word value can't read as part of
  the name; it stands in for the space before the value, so no value gets shorter.
- Only on `w7yfr` (`9ba5316`): Device > Dit mash in the menu chooses what a run of dits does: Menu (open the settings menu),
  Clear (clear the display) or Off. Saved with paddle echo in the spare EEPROM byte; `/KY MASH`, `ky.mash`.
- `feature/settings-menu`: a list shows when anything under it does (one holding only another list no longer
  vanishes); keyed shortcut answers read "Name: value" like the menu.
- Only on `w7yfr` (`ed77945`): Dit mash is a list: Command (Menu, Clear, Off) and Qty, how many dits make the run (6-12,
  default 10, in place of `dit_hold_reset_count`); `/KY DITS`, `ky.dits`.
- Only on `w7yfr` (`d55352e`): holding the dit paddle for 10 dits opens the settings menu (command mode with the menu
  open) instead of clearing the display; the extra button still clears it. The dits are still sent.

### Fixed
- `fix/speed-pot-at-startup`: at power-on the speed is the speed pot's position (when the pot is in use). The pot
  was read before the saved settings were, so the keyer started at the saved speed and ignored the pot until it was
  turned. The pot's first reading also always reads it: from a static 0, a pot at the bottom read as 0 wpm.
  Upstream `master` has the same bug.
- `fix/farnsworth-pot-at-startup`: at power-on the Farnsworth speed is the Farnsworth pot's position. The pot was
  read before the saved settings were, so the saved Farnsworth speed replaced it, and the pot counted as already
  applied, so it had no effect until turned. Found through Winkey testing: a host connecting resets the Mega, and
  read back the saved speed instead of the pot's.
- `fix/power-on-serial-button`: holding the command button at power-on picks the other USB port mode (CLI or
  Winkey) again. `initialize_serial_ports()` read the button before `initialize_analog_button_array()` had set up
  the ladder, so the press was never seen; the ladder is now set up first. Upstream `master` has the same bug.
- `feature/settings-menu`: in command mode, the adapter's connecting screen kept up: a Connect from the menu froze
  its clock and missed the screen that ends it.
- `feature/vband-link`: command mode notices the adapter going (its goodbye, or the link timeout) instead of only
  keeping the link alive, without drawing "VBand Off" over its screens. Received text and join/leave lines go on
  behind command mode and show on leaving it, so the display has the whole conversation.
- `feature/settings-menu`: the keyer's settings table, its labels and the mode names live in flash (`PROGMEM`),
  about 340 bytes of RAM back. A pause after "/KY" or "/KY WPM" ends it after 3 word spaces instead of a fixed 5 s,
  and shortcut answers and the menu's and VBand's notices show for 3 s (`SETTINGS_ANSWER_MS`) instead of 1. B on
  the menu's top level closes the menu and stays in command mode.
- `feature/settings-menu`: the menu follows the adapter coming and going while it's open. The VBand pages close back
  to the top level with "Adapter off" as the title, and the top level's VBand row appears and disappears, with the
  cursor kept on its row.
- `feature/dit-hold-clear-reset`: in straight mode, the display keeps the running text. A held straight key counted
  as a dit on every loop pass, so each release set off the dit-hold gesture and cleared the display.
- `fix/prosign-sos-and-warnings`: decoding SOS no longer sends SOS. `convert_prosign()` keyed it and returned
  nothing, so the echo printed from a garbage pointer; it now returns "SOS", shown in full by the straight key echo
  and memory playback. `paddle_pin_read()` returned nothing for a pin the direct-read options don't cover; it now
  falls back to `digitalRead()` (or reads as not pressed with `OPTION_SAVE_MEMORY_NANOKEYER`). The remaining compiler
  warnings in the boards' builds are fixed (unused variables, signed/unsigned comparisons, including the button
  library's). Upstream `master` has all of these.
- `fix/async-eeprom-write-overrun`: K3NG's background settings write (`service_async_eeprom_write`) wrote one byte
  too many, putting whatever follows the configuration in RAM into the EEPROM byte after it. Upstream `master` has
  the same bug.
- `feature/settings-menu`: settings changed from the menu or the CLI survive a power cycle. They're saved as soon as
  the keyer is idle, instead of waiting until 30 s after the last save (`eeprom_write_time_ms`), so a change made
  just before power-off was lost. Paddle echo moved to the next spare EEPROM byte, away from the one the
  background write overran, which turned it off after each save.
- `fix/straight-key-echo-without-cli`: builds with `FEATURE_STRAIGHT_KEY_ECHO` and a display but without
  `FEATURE_COMMAND_LINE_INTERFACE` no longer fail on `cli_straight_key_echo` (it was declared only with the CLI).
  Upstream `master` has the same bug.
- `fix/platformio-forward-declarations`: `service_beacon`, `service_straight_key`, `so2r_switches` and
  `sidetone_adj` are declared ahead of their first use, so builds outside the Arduino IDE no longer fail with
  "not declared in this scope" (seen with K3NG's `test-everything` and `megakeyer` profiles).
- Only on `w7yfr` (`a50434a`): builds without `FEATURE_OLED_SSD1306` link again; the extra button's display toggle and
  clear are only built with the OLED.

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
