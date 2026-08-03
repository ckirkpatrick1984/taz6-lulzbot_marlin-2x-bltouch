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
`Conditionals_LulzBot.h` - prototype-labeled code was ruled out. The
intended choice is `TOOLHEAD_CecropiaSilk_SingleExtruderAeroV2`, the
non-prototype universal SE/Aero toolhead (same E3D Titan Aero V6 block),
already used on the Workhorse build.

## Current state

This is currently a **stock, unmodified 2.0.0.144 checkout** - no printer
model, toolhead, or BLTouch config has been selected/added yet. That's
tracked as follow-up work in the same session; see the umbrella
`PROJECT.md` in the `lulzbot-marlin-bltouch` workspace folder for goals,
background, and open questions across all repos in this fleet.

# Safety and warnings:

**This repository may contain untested software.** It has not been extensively tested and may damage your printer and present other hazards. Use at your own risk. Do not operate your printer while unattended and be sure to power it off when leaving the room. Please consult the documentation that came with your printer for additional safety and warning information.
