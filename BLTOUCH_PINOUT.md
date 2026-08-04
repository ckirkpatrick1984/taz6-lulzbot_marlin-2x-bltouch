# BLTouch Pinout Instructions - LulzBot TAZ 6 (RAMBo)

See also: [BLTOUCH_PINOUT_SKETCH.svg](BLTOUCH_PINOUT_SKETCH.svg) - a
labeled board diagram of the same information below.

**Status: cross-verified against official docs, still not physically
confirmed on this printer.** The pin numbers below match both this
repo's firmware (`Marlin/Conditionals_LulzBot.h`, commit `7a05509`)
*and* UltiMachine/RepRap Electro's own RAMBo 1.1B User Manual (pin
mapping table, p.49 - `D22 = MX1-3`, `D30 = Z-Max/MX3-4`), so the pin
numbers themselves are solid. What's still unconfirmed is purely
physical: whether *this specific* v1.3/1.4 board's silkscreen still
labels these headers "MX1"/"Z-MAX" the same way, and whether the actual
BLTouch has been wired to match. Verify both with a multimeter and the
bring-up checklist below before trusting it near the bed. Do not home or
probe on real hardware until it's confirmed.

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
| 3-pin servo/control | Red = +5V, Black/Brown = GND, Yellow/Orange = Signal | **"MX1" motor header** | `SERVO0_PIN`, Arduino digital pin **22** | Drives the BLTouch's PWM deploy/stow command. Board default in `pins_RAMBO.h` - not reassigned in firmware. |
| 2-pin sensor/trigger | White = Signal, Black = GND | **Z-MAX endstop header** | `Z_MIN_PROBE_PIN`, Arduino digital pin **30** | Reads the BLTouch's trigger/alarm signal when the probe touches down. Repurposed in firmware from the (unused) Z-max endstop. |

Confirm the exact wire colors against the adapter cable you actually
have - kits vary. The "MX1" and "Z-MAX" header names and pin numbers
are cross-verified against UltiMachine's official RAMBo 1.1B User
Manual (pin mapping table, p.49) in addition to `pins_RAMBO.h` - but
still confirm the header *positions* against your physical v1.3/1.4
board's silkscreen, since the manual's board photo is from the 1.1B
revision.

## Two things to check before connecting

1. **The MX1 header may already be in use.** Stock (non-BLTouch) TAZ 6
   firmware uses this exact header (`SERVO0_PIN`) as the *input* signal
   for the old electrical bed-washer probe (`LULZBOT_BED_WASHERS_PIN`).
   If that wiring is still connected, disconnect it first - the BLTouch
   needs this header as a PWM *output*, and the two uses conflict
   electrically.
2. **The Z-MAX header should be free.** TAZ 6 homes Z using a home
   button + probe, not a mechanical Z-max switch, so stock firmware
   never enables `USE_ZMAX_PLUG` and this header goes unused. It should
   be empty on the physical board - but verify nothing is connected
   there before plugging in the BLTouch's 2-pin trigger cable.

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
4. Once homing works reliably, calibrate the values this firmware left
   as explicit placeholders (see commit `7a05509`'s message for the
   full list):
   - **Z probe offset** (`-1.0` placeholder): use `G29`/`M851` to
     measure the real difference between where the probe triggers and
     where the nozzle touches the bed, then `M851 Z<value>` and `M500`.
   - **X/Y probe offset** (`0,0` placeholder): measure how far the
     probe tip is offset from the nozzle on the physical mount, update
     `X_PROBE_OFFSET_FROM_EXTRUDER`/`Y_PROBE_OFFSET_FROM_EXTRUDER` in
     `Marlin/Conditionals_LulzBot.h`, rebuild, reflash.
   - **Z_SAFE_HOMING X/Y point** (currently `-19, 258`, inherited from
     the old home-button config): confirm it's still a safe, reachable
     point for the BLTouch specifically, adjust and reflash if not.
5. Run a full `G29` bed mesh (LCD: Prepare > Level Bed, now visible for
   this BLTouch build - see `PROJECT.md`) and check the results look
   sane (no wildly outlying points, which usually means a wiring/trigger
   problem rather than a real bed issue).
6. If leveling still looks unchanged/flat after this, reset to firmware
   defaults first to rule out stale EEPROM data from a prior firmware:
   `M502` then `M500`.

## Troubleshooting

- **BLTouch LED solid red, never blinks / no self-test on power-up**:
  check the 3-pin servo/control cable's polarity and that it's on the
  MX1 header, not still on the old bed-washer probe wiring.
- **BLTouch deploys but G28/G29 never detects a trigger**: check the
  2-pin sensor cable is on the Z-MAX header and its polarity - Marlin's
  BLTouch auto-configuration handles endstop-inverting/pullups
  automatically when `LULZBOT_USE_BLTOUCH` is enabled, so don't
  hand-edit those unless you know why.
- **Board won't boot / no LCD after flashing**: reflash the stock
  (non-BLTouch) config first (comment out `LULZBOT_USE_BLTOUCH` in
  `Marlin/Configuration_LulzBot.h`, rebuild, reflash) to isolate whether
  it's a wiring issue or a flashing issue.

## Reference

- Firmware pin assignment: `Marlin/Conditionals_LulzBot.h`, commit
  `7a05509` ("Add BLTouch support for TAZ 6").
- Board pin source: `Marlin/src/pins/pins_RAMBO.h`.
- Project background/open questions: `../PROJECT.md`.
