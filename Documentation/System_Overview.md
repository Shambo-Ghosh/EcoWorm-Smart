# System Overview — EcoWorm Smart

## Purpose

EcoWorm Smart is designed to keep a vermicompost bed within a safe moisture and temperature range without a person needing to check on it manually every day, while keeping a full record of what happened even if the system is offline for stretches at a time.

## Subsystems

**Sensing layer**
Two capacitive soil moisture probes are placed at different depths in the bed (top and bottom), because moisture doesn't distribute evenly through the compost — the surface dries out faster than the base. A DS18B20 digital temperature sensor tracks bed temperature, since worm activity and compost breakdown rate are both temperature-sensitive.

**Timekeeping**
A DS3231 real-time clock, backed by a coin cell, keeps time independently of the ESP32's own clock. This matters because the ESP32's internal clock resets on every power cycle, but irrigation scheduling and log timestamps need to survive power loss.

**Actuation layer**
A relay-driven 12V diaphragm pump handles irrigation. A pulse-based water flow sensor sits inline, so the system doses by actual liters delivered rather than by a fixed pump run-time — this matters because pump output isn't perfectly consistent (voltage sag, line pressure, etc. all affect how much a fixed-time pulse actually delivers).

**Storage layer**
Every cycle's readings and every irrigation event are appended to a CSV file on a microSD card. This is the system's primary record — it doesn't depend on connectivity to function.

**Connectivity layer**
Two possible upload paths exist depending on firmware revision:
- SIM900A GSM/GPRS: sends SMS summaries to the farm owner's phone, and can push readings to a Google Sheets endpoint over a cellular data connection.
- ESP32 WiFi (newer revision): connects to a local hotspot and uploads to the same kind of Sheets endpoint over HTTPS, freeing GSM to be used only for SMS alerts.

## Operating Modes

- **Demo mode** — short cycle intervals (on the order of minutes) so a full sense → irrigate → log → notify cycle can be observed live during a demonstration, rather than waiting hours between events.
- **Production mode** — calibrated to a realistic schedule (on the order of twice a day), with fail-safe pump cutoffs so a stuck sensor reading can't run the pump indefinitely.

## Power-Loss Handling

If the system loses power, the DS3231's own battery backup normally preserves the correct time. As a second line of defense, the firmware periodically checkpoints the last known time into the ESP32's non-volatile storage. If the RTC itself reports a lost-power condition, the system restores time from that checkpoint instead of falling back to firmware build time, and flags this in the log so it's clear the value is a recovered estimate rather than a fully trusted reading.

On reboot, the system also checks how much time has elapsed and whether an irrigation cycle was missed during the outage, so a blackout doesn't silently skip a scheduled watering.

## Fault Tolerance

Every hardware module (RTC, SD card, GSM, WiFi, flow sensor) is checked at startup and re-checked periodically during operation. If a module is missing or fails, the system logs it once and continues running on whatever modules are still working, rather than halting. A module that comes back online later (for example, an SD card reinserted) is picked up automatically without needing a reboot.
