---
title: Klipper Tweaks
---

# Klipper Tweaks

Advanced experimental tweaks for Klipper stepper motor driver configuration. These settings can **only** be configured via the [Firmware Configuration](firmware_config.md) web interface under **Settings → Tweaks**.

> **Warning**: These are experimental features that modify low-level stepper driver parameters. Use with caution and monitor your printer carefully after enabling.

## TMC AutoTune

Applies optimized stepper motor driver settings for TMC2240 drivers.

**What it does:**
- Optimizes PWM settings for quieter operation
- Configures StallGuard and CoolStep parameters
- Adjusts timing parameters for better heat management
- Fine-tunes driver parameters for improved performance

**Risks:**
- May cause motors to overheat if cooling is insufficient
- Could result in reduced torque or skipped steps under heavy load
- Incorrect settings may affect print quality
- Changes low-level driver parameters that override defaults

**Recommendation:**
- Monitor motor temperatures during first use
- Test with simple prints before production work
- Revert to disabled if you experience issues

**Configuration:**
This feature can **only** be configured via Firmware Configuration web interface. Manual configuration is not supported.

## TMC Reduced Current

Lowers the stepper motor run current from 1.2A to 1.0A for X and Y axes.

**What it does:**
- Reduces X and Y axis motor current to 1.0A
- Lowers motor heat generation
- Results in quieter motor operation

**Risks:**
- May cause skipped steps under heavy load or fast movements
- Could result in layer shifts on demanding prints
- May reduce positioning accuracy under high acceleration

**Recommendation:**
- Monitor print quality after enabling
- Watch for layer shifts or positioning issues
- Disable if you experience motion problems

**Configuration:**
This feature can **only** be configured via Firmware Configuration web interface. Manual configuration is not supported.

## Max Speed

Raises the XY motion limits and speeds up tool changes, together with the TMC2240
chopper tuning that keeps the XY drivers stable at those speeds.

Parameters derived from [@JNP-1](https://github.com/JNP-1/Snapmaker-U1-Config);
the `Aggressive` values are the ones proposed by @justinh-rahb in
[#679](https://github.com/paxx12-snapmaker-u1/SnapmakerU1-Extended-Firmware/pull/679).

**Two tiers:**

| | Stock | Balanced | Aggressive |
|---|---|---|---|
| Max XY velocity | 500 mm/s | 600 mm/s | 750 mm/s |
| Max acceleration | 20000 mm/s² | 22000 mm/s² | 25000 mm/s² |
| Tool change speed | 400 mm/s | 550 mm/s | 700 mm/s |
| Tool change acceleration | 5000 mm/s² | 12000 mm/s² | 25000 mm/s² |
| Dock grab speed | 10 mm/s | 20 mm/s | 40 mm/s |

Both tiers also apply the same TMC2240 chopper tuning (`TBL`, `TOFF`, `HEND`,
`HSTRT`, `TPFD`, `VHIGHFS`, `VHIGHCHM`, `PWM_*`) and re-assert the stock 1.2A X/Y
run current.

**Which one:** start with `Balanced`. Most of the margin between the two tiers is on
the tool change side, because dock grab speed and tool change acceleration are
what drive the head into the dock and are not capped by anything in the slicer.
Move to `Aggressive` only once tool changes are reliable on `Balanced`.

Raising `max_velocity` on its own changes little in practice — the slicer caps
print moves by volumetric flow long before 750 mm/s, so this mostly affects
travel moves.

**Requirements:**
- Dock positions are calibrated correctly
- Belt tension is correct
- Input shaper is calibrated for your printer (run `SHAPER_CALIBRATE`)
- Tool changes are already reliable with the stock configuration

**Risks:**
- Crashes into the tool docks if dock positions are off
- Layer shifts and skipped steps under heavy load
- Higher driver and motor temperatures
- Ringing and other quality artifacts if input shaper is not calibrated

**Recommendation:**
- Calibrate the printer fully before enabling
- Start with `Balanced` and a small test print, and watch the first tool changes
- Disable if you see dock alignment problems or layer shifts

**Note:** This tweak cannot be combined with [TMC AutoTune](#tmc-autotune) or
[TMC Reduced Current](#tmc-reduced-current) — all three change the same TMC2240
driver settings, and Max Speed needs the full 1.2A X/Y current. Firmware Config
refuses to enable a conflicting combination and tells you which tweak to disable
first.

### Not ported from JNP-1's configuration

[@JNP-1's repo](https://github.com/JNP-1/Snapmaker-U1-Config) is a complete
`printer.cfg`. This tweak takes only the settings that are about speed and that
hold on any U1. These parts are deliberately left at stock:

| Not ported | Stock | JNP-1 | Why |
|---|---|---|---|
| `[input_shaper]` and the `[resonance_tester]` probe point | — | X 54 Hz, Y 47.5 Hz | The right values depend on your individual printer. Run `SHAPER_CALIBRATE`, then `SAVE_CONFIG`. |
| `rotation_distance` on all four extruders | 4.95 | 5.0147 | Extruder calibration, not a speed setting. |
| `fan_speed` on `e0`–`e3_nozzle_fan` | 1 | 0.8 | Cooling preference, unrelated to motion. |
| Faster homing: `[homing_xyz_override] speed`, the `M204` accel cap and the homing current | 300 mm/s, `S1000`, 0.650A | 800 mm/s, `S10000`, 0.900A | See below. |

Homing needs all three of those together. `speed` is an ordinary config option,
but the accel cap and the homing current live inside that section's `gcode:`
template, which can only be changed by restating the whole macro body. Raising
`speed` on its own would mean homing at 800 mm/s while still capped at
1000 mm/s² and still at 0.650A — an untested combination on sensorless homing,
with a failure mode (crashing into an endstop) that this tweak does not cover.

`square_corner_velocity` is also left alone: stock is already 8 mm/s.

**Configuration:**
This feature can **only** be configured via Firmware Configuration web interface. Manual configuration is not supported.

## Object Processing for Adaptive Mesh

Enables object processing in Moonraker's file manager to support adaptive mesh features.

**What it does:**
- Processes gcode files to extract object information
- Generates boundaries for adaptive mesh leveling
- Allows per-object print settings and controls

**Risks:**
- Can cause very long processing times for large gcode files (> 100MB)
- May result in extended delays when uploading files
- Snapmaker Orca may stay at 100% for a long time when sending prints
- High memory usage during file processing
- Can cause delays before prints can start

**Important:**
- Enabling this setting alone is not enough to use adaptive mesh
- You must also update your slicer start gcode to use: `BED_MESH_CALIBRATE ADAPTIVE=1`
- This tells Klipper to only mesh the area where objects will be printed

**Recommendation:**
- **Preferred approach:** Enable `Exclude Object` in your slicer settings instead of this option
- Slicer-generated object labels are more reliable and don't require server-side processing
- Only enable Moonraker object processing if your slicer doesn't support exclude object
- Disable if you frequently print large gcode files
- Monitor file upload times after enabling
- Consider splitting large models into smaller prints if processing is too slow

**Configuration:**
This feature can **only** be configured via Firmware Configuration web interface. Manual configuration is not supported.

## How to Configure

1. Open the printer's web interface (Fluidd or Mainsail)
2. Navigate to **Firmware Config** in the menu
3. Go to **Settings → Tweaks**
4. Select the desired option for each tweak
5. Confirm the warning dialog
6. Klipper will automatically restart to apply changes

Changes take effect immediately after Klipper restarts (no reboot required).

## Technical Details

These tweaks work by adding or removing configuration files from `/oem/printer_data/config/extended/`:
- `klipper/tmc_autotune.cfg` - TMC AutoTune parameters
- `klipper/tmc_current.cfg` - Reduced current settings
- `klipper/10_max_speed.cfg` - Max Speed motion limits, tool change speeds and driver tuning (either tier installs to this one file)
- `moonraker/object_processing.cfg` - Moonraker object processing settings

These files are automatically included by the main printer configuration if present. Manual editing of these files is not recommended as they will be overwritten by the Firmware Configuration interface.
