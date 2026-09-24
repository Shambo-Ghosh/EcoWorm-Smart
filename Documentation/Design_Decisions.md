# Design Decisions — EcoWorm Smart

This document explains the reasoning behind the major design choices, not just what was built.

## Why ESP32 over a simpler microcontroller

Needed enough I/O for two analog moisture sensors, a 1-Wire temperature sensor, SD/SPI, I2C for the RTC, and two serial channels (GSM + debug), plus native WiFi for the newer connectivity path. A bare Arduino Uno-class board would have run out of resources and required an added WiFi/GSM shield anyway.

## Why a hardware RTC instead of relying on NTP

The system is meant to run in field conditions without guaranteed connectivity. An NTP-only approach would lose correct time entirely whenever the network is down, which is exactly when accurate logging matters most (to know when an irrigation cycle was missed). A coin-cell-backed DS3231 keeps time independently, with NTP/network sync treated as optional rather than required.

## Why dual moisture probes instead of one

A single probe at one depth doesn't represent the whole bed. Vermicompost beds dry from the top down, so a top-only reading over-triggers irrigation, and a bottom-only reading risks surface drying going unnoticed. Reading both lets the irrigation logic use whichever threshold is more conservative.

## Why flow-metered irrigation instead of timed pulses

A fixed pump on-time is a poor proxy for volume delivered — supply voltage, tubing kinks, and pump wear all change actual output over time. A flow sensor gives a real liters-delivered figure, so irrigation events are measurable and comparable across time, and so a partially-blocked line doesn't silently under-water the bed while the log claims a normal watering.

## Why the WiFi/GSM split in the newer firmware revision

The original design used the SIM900A for both SMS and data upload. In practice, GPRS bearer setup on that module is the least reliable part of the whole system (see `Bug_Fixes.md` in the private log). Splitting responsibilities — ESP32 WiFi for bulk data upload, GSM reserved only for short SMS alerts — reduces how often the flakiest subsystem is on the critical path, since SMS is a much smaller, simpler operation than opening an HTTP session over GPRS.

## Why fail-soft rather than fail-stop

An earlier version would effectively wait or hang if a module (SD card, GSM, RTC) failed to initialize. For a system meant to run unattended in the field, a hang is worse than degraded operation — a missing SD card shouldn't stop moisture readings and irrigation from happening; it should just mean that cycle isn't logged. The newer firmware treats every module as optional and probes/re-probes it independently, so a single failed component degrades the system instead of stopping it.

## Why NVS (flash) backup of RTC time

Coin-cell RTC modules are a known point of failure — some breakout boards (in particular, "ZS-042" style DS3231 boards) are wired with a charging circuit meant for a rechargeable cell, which slowly damages or fails to properly back up a standard non-rechargeable CR2032. Rather than relying purely on the coin cell, the firmware also checkpoints the last known good time into the ESP32's own flash storage, so a power loss with a failing coin cell still recovers to a recent known time instead of resetting to firmware build time.

## Demo mode vs. production timing

Testing an irrigation + logging + notification cycle on a real twice-a-day schedule would make a live demonstration impractical. Cycle timing is a single configurable constant, letting the same firmware logic run on a multi-minute demo interval or a realistic twice-daily production interval without changing any of the actual decision logic.
