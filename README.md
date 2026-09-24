# EcoWorm Smart

An embedded automation and data-logging platform for vermicompost bed monitoring and irrigation, built on ESP32.

> **Note on source code:** This project is not open-source. This repository documents the design, testing, and results only. The firmware source is maintained in a private repository. Feel free to reach out if you'd like to discuss implementation details.

## Problem Statement

Vermicompost beds need consistent moisture and temperature to keep the worm colony alive and productive. Manual monitoring is inconsistent, especially for farms without staff on-site around the clock, and over/under-watering is one of the most common causes of failed vermicompost setups.

## Objectives

- Continuously monitor bed moisture (top and bottom layers) and temperature
- Automate irrigation based on real thresholds instead of a fixed schedule
- Log every reading locally so no data is lost even without connectivity
- Notify the farm owner remotely when irrigation runs, and push data to the cloud when a network is available
- Keep the system running through power blackouts without losing track of time or missing a watering cycle

## Features

- Dual-layer capacitive soil moisture sensing (top + bottom of bed)
- DS18B20 digital temperature sensing
- Flow-metered irrigation (exact liters dispensed, not just pump on-time)
- Local CSV logging to microSD, so the system works fully off-grid
- Cloud upload to Google Sheets and SMS summaries to the farm owner's phone
- Battery-backed real-time clock (DS3231) with a software checkpoint, so the system can recover a sensible time even after a full power loss
- Configurable for demo mode (short cycles for live testing) or production mode (twice-daily irrigation schedule)

## Hardware Used

| Component | Role |
|---|---|
| ESP32 | Main controller |
| DS18B20 | Temperature sensing |
| Capacitive soil moisture sensor x2 | Top and bottom bed moisture |
| DS3231 RTC | Timekeeping, survives power loss on coin-cell backup |
| MicroSD module | Local data logging |
| Water flow sensor | Measured (not estimated) irrigation volume |
| 5V relay module | Pump/valve switching |
| 12V diaphragm pump | Irrigation actuation |
| SIM900A GSM module | SMS alerts, cellular data upload where WiFi isn't available |

## System Architecture

At a high level, the firmware runs a sensor-read → decision → actuate → log → upload loop:

1. **Sense** — read moisture (top/bottom) and temperature on a timer
2. **Decide** — compare against calibrated thresholds to decide if irrigation is needed
3. **Actuate** — run the pump until the flow sensor confirms the target volume has been delivered, not just for a fixed time
4. **Log** — write every cycle's data to microSD as CSV, so nothing is lost even with no network
5. **Notify/Upload** — send an SMS summary and push the record to a Google Sheets endpoint when connectivity is available

Every hardware module is treated as optional at runtime: if a sensor or module is missing or fails, the system logs a warning and keeps running on what's left, rather than halting. See `Documentation/System_Overview.md` for details.

## Current Status

- Electronics design: complete
- Firmware: developed and field-tested in demo configuration
- Presented at Academy of Technology to invited IIT Kharagpur delegates
- Tested once in a live vermicompost farm setting
- A more advanced firmware revision (fail-soft module handling, WiFi upload path, power-loss time recovery) is complete in code but not yet field-tested with the pump and flow sensor connected
- Custom PCB design in KiCad: planned

See `Documentation/Testing_Report.md` for what's verified and what's still pending.

## Future Scope

- Field re-test of the latest firmware revision with full hardware attached
- Custom PCB to replace the current breadboard/module wiring
- Local web dashboard served from the ESP32 (or a small backend) for live monitoring without depending on Google Sheets
- Scale-up path from single demo bed to multi-row commercial deployment

## Repository Contents

- `Documentation/` — system overview, design rationale, testing report
- `Circuit_Diagram/` — schematic and wiring diagrams
- `PCB_Design/` — KiCad schematic, layout, renders (once ready)
- `Results/` — calibration data, sensor readings, water usage measurements
- `Images/` — build photos
- `Demo/` — demonstration screenshots
