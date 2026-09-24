# Testing Report — EcoWorm Smart

## Status Summary

| Build | Status |
|---|---|
| Demo firmware (irrigation + SD + SMS + Sheets upload over GPRS) | Field-tested, demonstrated live |
| Latest firmware revision (fail-soft architecture, WiFi upload, RTC power-loss recovery) | Complete in code, not yet field-tested with pump/flow sensor attached |

## What Has Been Verified

**Sensor readings**
- Moisture sensors calibrated against dry/wet reference points for both the top and bottom probes, mapped to a 0–100% scale
- Temperature sensor confirmed reading correctly, with disconnected-sensor state detected and logged distinctly rather than reported as a false reading

**Irrigation**
- Flow-metered dosing confirmed: pump runs until the flow sensor reports the target volume delivered (with a maximum-runtime safety cutoff so a stuck/miscounting sensor can't run the pump indefinitely)

**Logging**
- CSV logging to microSD confirmed working across repeated cycles, including automatic header creation on first run

**Notifications and upload**
- SMS alerts to the farm owner's phone confirmed sending after each irrigation cycle, including sensor values and volume dispensed
- Data upload to a Google Sheets endpoint confirmed working over GPRS

**Real-world testing**
- Demonstrated live at Academy of Technology to invited delegates of IISc and Jadavpur University
- Tested once in an actual vermicompost farm setting, not just a bench setup

## What Is Not Yet Verified

- The latest firmware revision (fail-soft module handling, WiFi-based upload, RTC checkpoint recovery) has not been run end-to-end with the pump and flow sensor physically connected — those modules are currently disabled by a compile-time flag in that build, and irrigation falls back to a fixed-time estimate rather than measured volume until re-enabled and tested.
- Long-duration/unattended field reliability (multi-day, multi-week run without intervention) has not yet been tested.
- Behavior under a genuine multi-hour power outage in the field (as opposed to a bench power-cycle test) has not been separately verified.

## Known Issues

- DS3231 coin-cell backup is not always reliable on the specific breakout board in use — addressed in firmware via a flash-based time checkpoint, but the underlying hardware issue is worth resolving on the custom PCB.
- GPRS bearer setup on the SIM900A occasionally fails on the first attempt; a retry approach exists in one firmware branch and is planned to be merged into the primary build.
- The GSM module's own HTTP status read-back has been observed to be unreliable — requests can succeed even when the module reports a failure code, so upload success is not gated on that status.

## Next Testing Steps

1. Reconnect pump and flow sensor, re-enable those modules in the latest firmware revision, and run a full field cycle.
2. Merge the GPRS retry logic into whichever build becomes the primary firmware.
3. Run an extended (multi-day) unattended test to validate long-term stability.
