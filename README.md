# taz6-lulzbot_marlin-2x-bltouch

Marlin firmware for the LulzBot TAZ 6, on a **real Marlin 2.x core**,
intended to get BLTouch auto bed-leveling probe support added.

Board: RAMBo (v1.3 or v1.4 - exact revision not yet confirmed against the
physical board).

## Relationship to earlier TAZ 6 attempts

This supersedes two earlier, discontinued TAZ 6 efforts:

- An early `taz6-marlin` checkout on LulzBot's Marlin-1.1.9-based source
  (same core as `lulzbot-mini1-marlin-bltouch`) that never got BLTouch
  config added.
- A later `taz6-se-bltouch-marlin` attempt built on LulzBot's download-page
  "2.0.9" release, deleted at the user's explicit request as "the wrong
  build" - see below for why.

## Base source (verified, not guessed)

LulzBot's own download page labels its TAZ 6 release "2.0.9", but - as
already caught once on the Mini 1 rebuild - that label is LulzBot's own
release/software counter, not the underlying Marlin core version; past
"2.0.x"-labeled LulzBot downloads have turned out to actually be
re-badged Marlin 1.1.9 builds. To avoid repeating that mistake, this repo
uses the same independently-verified real Marlin 2.x core already used
for `mini1-marlin-2x`:

- **GitLab tag `v2.0.0.144`** on `gitlab.com/lulzbot3d/marlin` (tagged
  2019-07-12). `Marlin/src/inc/Version.h` defines
  `SHORT_BUILD_VERSION "2.0.0" LULZBOT_FW_VERSION`, with
  `LULZBOT_FW_VERSION ".144"` in `Marlin/Conditionals_LulzBot.h` - and the
  tree uses Marlin 2.x's reorganized `Marlin/src/` layout, not 1.x's flat
  `Marlin/*.cpp` layout. This is a genuine Marlin 2.x core.
- **Download mirror**:
  `download.lulzbot.com/Software/Marlin/2.0.0.144/Marlin-src_2.0.0.144_aded3b617.zip`
  (commit `aded3b617`, dated 2019-07-22) - the same zip already used for
  `mini1-marlin-2x`; confirmed byte-identical to that repo's copy of
  `README_Marlin.md` / `README_LulzBot.md`.
- `Marlin/Configuration_LulzBot.h` at this version lists `Oliveoil_TAZ6`
  (LulzBot TAZ 6) as a first-class, non-experimental printer model, and
  `Marlin/Conditionals_LulzBot.h` maps it to `BOARD_RAMBO` via
  `LULZBOT_IS_TAZ` - the correct board family for the TAZ 6's actual RAMBo
  controller (distinct from the Mini-RAMBo, EinsyRetro, and Archim2 boards
  used by other printers in this same source tree).

LulzBot's own build/dev-branch docs are preserved at
[README_LulzBot.md](README_LulzBot.md); stock upstream Marlin's README is
preserved at [README_Marlin.md](README_Marlin.md).

## Toolhead

The physical toolhead is Titan-Aero-based and not the stock TAZ 6 head,
but has no visible markings to confirm v1 vs v2. This source's only
TAZ-specific "Aero v1"-style option, `TOOLHEAD_Angelfish_Aerostruder`, is
explicitly commented `// Prototype Aero for TAZ` in
`Conditionals_LulzBot.h` - prototype-labeled code was ruled out. Selected
instead: `TOOLHEAD_CecropiaSilk_SingleExtruderAeroV2`, the non-prototype
universal SE/Aero toolhead (same E3D Titan Aero V6 block), already used
on the Workhorse build.

## Prebuilt firmware

A ready-to-flash **`firmware.hex`** is committed at the repo root, built
from the current `master` with `LULZBOT_USE_BLTOUCH` enabled
(`python3 -m platformio run -e rambo`; Flash 65.2%, RAM 68.5%).

**Read this before flashing it to your printer.** This build is not a
drop-in for a stock TAZ 6. It assumes a specific, physically modified
machine:

- **The mechanical Z-home button has been physically removed.** The
  BLTouch is the *sole* Z reference for both homing and probing. On a
  stock TAZ 6 that still has its home button, this firmware is not
  appropriate.
- **BLTouch wiring must match** `BLTOUCH_PINOUT.md`: trigger/sensor on
  `Z_MIN_PIN` (Arduino digital pin **10**, the Z-Min endstop header), and
  servo/control on `SERVO0_PIN` (pin **22**, the RAMBo MX1 row).
- **Toolhead is a non-stock Titan Aero**, configured as
  `TOOLHEAD_CecropiaSilk_SingleExtruderAeroV2`.
- **Probe offsets are specific to this BLTouch mount**: X `-4`, Y `-46`
  (measured 3.7 mm left / 46.4 mm in front, rounded - the 1.x/2.0 fork
  requires integers). Your mount will almost certainly differ.
- **The Z probe offset (`-3.0`) is a ruler measurement, not a calibrated
  one.** It has not yet been dialled in with an `M851` paper test, so
  first-layer height will need tuning on any machine.

If you flash this, **verify `M119` shows the probe actually toggling
before running `G28`** - an unconnected or miswired probe reads as
"never triggered," and Z homing will then drive the nozzle into the bed.

After flashing, run **`M502` then `M500`** to load and save the compiled
defaults. EEPROM values (probe offset, PID, steps/mm, mesh) survive a
flash and will otherwise silently shadow the firmware's settings.

To build it yourself instead, comment out `LULZBOT_USE_BLTOUCH` in
`Marlin/Configuration_LulzBot.h` for the stock configuration (Flash
63.9%, RAM 66.1%), or leave it enabled for the BLTouch build.

## Current state

Printer model and toolhead are selected in `Marlin/Configuration_LulzBot.h`
(`LULZBOT_Oliveoil_TAZ6` / `TOOLHEAD_CecropiaSilk_SingleExtruderAeroV2`)
and build-verified both ways: BLTouch enabled (Flash 65.2%, RAM 68.5%)
and stock/disabled (Flash 63.9%, RAM 66.1%).

**BLTouch probing and a full LCD-menu `G29` are confirmed working on real
hardware**, which retroactively validates the pin 10 / pin 22 wiring
documented in `BLTOUCH_PINOUT.md`.

Configuration reached its current state by removing the stock TAZ 6
bed-leveling behavior, which was built around the electrical bed-washer
probe (where the nozzle itself was the probe) and is wrong for a BLTouch:

- `G29_RETRY_AND_RECOVER` (reheat / wipe / retry) - disabled entirely
- Probe grid boundaries moved onto the physical bed (`10`/`270`/`10`/`245`);
  all four stock values sat on external washers, some off the bed
- `Z_SAFE_HOMING` moved from the old home-button position (`-19`, `258`)
  to bed center (`X_CENTER`, `Y_CENTER`), matching Marlin's own default
- `Z_PROBE_LOW_POINT` restored to Marlin's stock `-5` (was `0`, leaving no
  search margin past the expected trigger height)
- `Z_PROBE_OFFSET_RANGE_MIN`/`MAX` restored to Marlin's stock `-20`/`20`
  (was `-2`/`5`, which made `M851` reject the offset needed here)
- `BLTOUCH_FORCE_SW_MODE` enabled, so the probe holds its trigger output
  rather than pulsing ~10 ms - this fork has no `ENDSTOP_INTERRUPTS_FEATURE`,
  so endstops are polled and a short pulse can be missed

Remaining hardware-calibration TODO: the Z probe offset paper test
(`M851` + `M500`). See the umbrella `PROJECT.md` in the
`lulzbot-marlin-bltouch` workspace folder for goals, background, and open
questions across all repos in this fleet.

# Safety and warnings:

**This repository may contain untested software.** It has not been extensively tested and may damage your printer and present other hazards. Use at your own risk. Do not operate your printer while unattended and be sure to power it off when leaving the room. Please consult the documentation that came with your printer for additional safety and warning information.
