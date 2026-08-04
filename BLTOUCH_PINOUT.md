# BLTouch Pinout Instructions - LulzBot TAZ 6 (RAMBo)

## Images (start here)

- **[BLTOUCH_BOARD_PHOTO_ANNOTATED.png](BLTOUCH_BOARD_PHOTO_ANNOTATED.png)**
  - the real RAMBo board photo, fully labeled with every connector
  (mosfets, endstops, motors, thermistors, etc.), with the two BLTouch
  connectors additionally highlighted: **Z-MIN** (red) and **MX1** (blue).
  Start here for orientation.
- **[RAMBO_BOARD_FULL_REFERENCE.png](RAMBO_BOARD_FULL_REFERENCE.png)** -
  the same photo without the BLTouch-specific highlights, for a clean
  general-purpose reference of every connector on the board.
- **[MOTOR_EXT_MX1_MX2_MX3_CLOSEUP.png](MOTOR_EXT_MX1_MX2_MX3_CLOSEUP.png)**
  - a macro close-up of the "Motor Ext 1" header. Important: on this
  board revision it's a single 5-column x 3-row pin block, not three
  separate side-by-side headers. **Rows** are MX3 (top), MX2 (middle),
  MX1 (bottom) - MX1 is the **bottom row**, not a column.

All three images are real RAMBo board photos (not renders), cropped and
annotated from UltiMachine's official **RAMBo 1.3L manual** - see
"Reference" at the bottom for the exact source.

## This build no longer uses a Z-Home button at all

By explicit decision (not a firmware default): the physical mechanical
Z-Home button switch has been **removed** from this printer. The BLTouch
is now the sole Z reference for both homing and probing - it is wired to
the **Z-MIN** endstop header, exactly like a standard BLTouch conversion
on the Mini/TAZ Pro side of this same codebase. **Z-MAX is not used for
anything in this build** - nothing should be connected there.

This replaces an earlier version of this firmware/doc that routed the
BLTouch's trigger signal to Z-MAX instead, on the assumption that the
old home-button switch was still physically present on Z-MIN. That
assumption was wrong for this printer - if you built anything from that
earlier plan, rewire the trigger cable to Z-MIN, not Z-MAX.

## Corrected from an earlier version of this doc

An earlier version of this document and its images were also built from
the **RAMBo 1.1B manual** - the wrong board revision. Two things were
wrong there, independent of the Z-MIN/Z-MAX question above:

- This is a **RAMBo v1.3/1.4** board. Its "Motor Ext 1" header is a
  single 5x3 pin block where MX1/MX2/MX3 are **rows**, not separate
  side-by-side headers (as they are on 1.1B) - confirmed both from a
  real board photo and from the schematic in UltiMachine's own
  **RAMBo 1.3L manual**.
- **Pin 1 = GND, pin 2 = VCC** (not the other way around) - confirmed by
  two independent schematics (the 1.3L manual's "Motor Extensions"
  block, and the 1.1B manual's own schematic diagram, which contradicts
  that same manual's prose text). The physical board photo's "-" / "+"
  silkscreen markings under the MX1 row also line up with GND-then-VCC
  reading left to right.

If you saved or printed an earlier version of this doc, discard it -
the underlying firmware pin *numbers* (D22 for the servo/control line)
were always correct (independently sourced from this repo's own
firmware, not either manual), only the physical layout, VCC/GND
ordering, and (see above) the Z-MIN/Z-MAX choice were wrong.

## Status

**Board-revision-correct, architecture matches explicit instructions,
still not physically confirmed on this specific printer.** Verify with
a multimeter and the bring-up checklist below before trusting it near
the bed. Do not home or probe on real hardware until it's confirmed.

## Safety first

- Power off and unplug the printer before connecting or disconnecting
  anything on the control board.
- Never connect or disconnect the BLTouch cables while the board is
  powered.
- Double-check polarity before powering on. A reversed 5V/GND connection
  on the servo/control cable can destroy the BLTouch and potentially the
  board's 5V rail.
- This is exactly the kind of change the repo's own `README.md` already
  warns about: "may damage your printer... use at your own risk."

## What you need

- A BLTouch (genuine or common clone - v3.0/3.1/most clones work with
  stock Marlin BLTouch settings).
- A BLTouch-to-RAMPS/RAMBo conversion cable. Most BLTouch kits ship with
  a 5-wire-to-2-connector adapter made for RAMPS/RAMBo-style boards - it
  splits into a 3-pin "servo" connector (GND/+5V/Signal) and a 2-pin
  "sensor" connector (GND/Signal), each keyed to only plug in one way.
  **Use that adapter rather than loose jumper wires** - it removes most
  of the risk of getting polarity backwards.

## Target pinout

| BLTouch cable | BLTouch wires (typical) | Connects to | RAMBo pin | Function in firmware |
|---|---|---|---|---|
| 3-pin servo/control | Black/Brown = GND, Red = +5V, Yellow/Orange = Signal | **MX1 row** (bottom row of the "Motor Ext 1" block), pins 1-2-3 left to right | `SERVO0_PIN`, Arduino digital pin **22** (pin 3 of the row) | Drives the BLTouch's PWM deploy/stow command. Board default in `pins_RAMBO.h` - not reassigned in firmware. |
| 2-pin sensor/trigger | White = Signal, Black = GND | **Z-MIN endstop header** | `Z_MIN_PIN`, Arduino digital pin **10** | Reads the BLTouch's trigger/alarm signal directly on the Z-Min endstop pin (`LULZBOT_Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN`) - the same mechanism the Mini/TAZ Pro already use, since this build no longer has a separate mechanical home button. |

MX1 row pin order, left to right: **1 = GND, 2 = VCC, 3 = Signal (D22),
4 = unused, 5 = unused.** Only use the first 3 pins.

Confirm the exact wire colors against the adapter cable you actually
have - kits vary. The pin *numbers* are cross-verified against
UltiMachine's RAMBo 1.3L manual schematic in addition to
`pins_RAMBO.h` - but still confirm the header *position* against your
physical board's silkscreen, since minor layout details can still vary
between 1.3 and 1.4.

## Two things to check before connecting

1. **The MX1 row may already be in use.** Stock (non-BLTouch) TAZ 6
   firmware uses this exact pin (`SERVO0_PIN`) as the *input* signal for
   the old electrical bed-washer probe (`LULZBOT_BED_WASHERS_PIN`). If
   that wiring is still connected, disconnect it first - the BLTouch
   needs this pin as a PWM *output*, and the two uses conflict
   electrically.
2. **The Z-MIN header should be free of the old home-button switch.**
   Confirm the mechanical Z-Home button has actually been physically
   disconnected before wiring the BLTouch's trigger cable there - the
   two can't share the pin. Z-MAX should have nothing connected to it
   at all in this build.

## Bring-up checklist (do this before your first real print)

1. **Before homing anything**, verify the BLTouch's own self-test: with
   the printer powered and idle, most BLTouch units do a brief
   deploy/retract "wiggle" and the LED changes from solid red to
   blinking red on power-up if wired correctly. If nothing happens,
   stop and recheck wiring before proceeding.
2. From the LCD or over serial, send `M280 P0 S10` (deploy) and
   `M280 P0 S90` (stow) and confirm the pin visibly moves each time,
   cleanly, with no grinding/binding.
3. Only after that works: try `G28` (home all axes), watching Z
   carefully the first time. Be ready to hit the emergency stop / power
   switch if it doesn't stop correctly - this is the highest-risk first
   test, since a wrong probe pin or offset could drive the nozzle into
   the bed.
4. Once homing works reliably, verify/calibrate these values (see
   commit `0c0f7e1`'s message for the bed-center/offset rationale, and
   `2af84a3`/`7a05509` for earlier history):
   - **Z probe offset** (`-1.0` - still an unmeasured placeholder): use
     `G29`/`M851` to measure the real difference between where the
     probe triggers and where the nozzle touches the bed, then
     `M851 Z<value>` and `M500`.
   - **X/Y probe offset** (`-4, -46` - user-measured from the physical
     mount, rounded to the nearest integer mm since Marlin requires
     integer values here): confirm on real hardware that the BLTouch is
     actually ~4mm left and ~46mm in front of the nozzle as expected: if
     `G29`/leveling behaves oddly, re-measure and update
     `X_PROBE_OFFSET_FROM_EXTRUDER`/`Y_PROBE_OFFSET_FROM_EXTRUDER` in
     `Marlin/Conditionals_LulzBot.h`, rebuild, reflash.
   - **Z_SAFE_HOMING X/Y point** (now `140, 140` - the physical center
     of the bed in machine coordinates, so the BLTouch lands at bed
     center on every Z home): confirm this is actually a safe,
     reachable point for the BLTouch on the real printer (no
     obstructions, within the printable area) - adjust and reflash if
     not.
5. Run a full `G29` bed mesh (LCD: Prepare > Level Bed, now visible for
   this BLTouch build - see `PROJECT.md`) and check the results look
   sane (no wildly outlying points, which usually means a wiring/trigger
   problem rather than a real bed issue).
6. If leveling still looks unchanged/flat after this, reset to firmware
   defaults first to rule out stale EEPROM data from a prior firmware:
   `M502` then `M500`.

## Troubleshooting

- **"Level Bed" on the LCD only does a Z-home, no 5x5 grid probing**:
  fixed in commit `c3517a1` - the back edge of the probe grid (291mm)
  was unreachable by the nozzle once the real BLTouch Y offset (probe
  46mm in front of the nozzle) was applied, so `G29` aborted instantly
  on its own reachability check, right after `G28`'s homing move -
  which from the LCD looked exactly like "only a Z-home happened."
  Pulled the grid's back boundary in to 250mm. If this recurs after
  changing the probe offset again, re-check that
  `LULZBOT_STANDARD_BACK_PROBE_BED_POSITION` (and left/right/front,
  though those have more margin) stays within
  `Y_MAX_POS + Y_PROBE_OFFSET_FROM_EXTRUDER` (currently `303 + (-46) =
  257`).
- **BLTouch LED solid red, never blinks / no self-test on power-up**:
  check the 3-pin servo/control cable's polarity and that it's on the
  MX1 row, not still on the old bed-washer probe wiring.
- **BLTouch deploys but G28/G29 never detects a trigger**: check the
  2-pin sensor cable is on the Z-MIN header (not Z-MAX) and its
  polarity - Marlin's BLTouch auto-configuration handles
  endstop-inverting/pullups automatically when `LULZBOT_USE_BLTOUCH` is
  enabled, so don't hand-edit those unless you know why.
- **Board won't boot / no LCD after flashing**: reflash the stock
  (non-BLTouch) config first (comment out `LULZBOT_USE_BLTOUCH` in
  `Marlin/Configuration_LulzBot.h`, rebuild, reflash) to isolate whether
  it's a wiring issue or a flashing issue.

## Reference

- Firmware pin assignment: `Marlin/Conditionals_LulzBot.h`, commits
  `7a05509` ("Add BLTouch support for TAZ 6") and `2af84a3` ("Route
  BLTouch trigger through Z-MIN instead of Z-MAX").
- Board pin source: `Marlin/src/pins/pins_RAMBO.h`.
- Board photos and pin mapping: *RAMBo 1.3L* connector diagram and
  schematic ("RAMBo-connectors.ai" / `RAMBo-manual.pdf`), downloaded
  from LulzBot's own TAZ 6 production-parts documentation:
  `download.lulzbot.com/TAZ/6.02/production_parts/electronics/RAMBo/docs/RAMBo-manual.pdf`.
  Board design and document copyright 2014 UltiMachine (Johnny, Britt,
  Dorothy, Lee, Bruce). Page 1 (main connectors photo) and page 3
  ("Motor Extensions" schematic block) were cropped and re-annotated
  with the Z-MIN/MX1 highlights for this project.
- Project background/open questions: `../PROJECT.md`.
