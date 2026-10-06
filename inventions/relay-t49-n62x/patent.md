# Firmware-managed satellite terminal power cycling for outage-resilient last-mile links

## Abstract
A method and system in satellite user terminals for detecting grid power loss via voltage sensing and executing timed low-power cycling of the modem and phased array to preserve battery runtime during extended outages. The firmware monitors input voltage at 1 Hz, initiates 30-120 s sleep intervals when below 10.5 V, and resumes with beam reacquisition only after voltage recovery above 11.5 V for 60 s. Outage statistics are logged to non-volatile memory and a low-rate beacon mode activates after 72 h.

## Problem
Hawaii island last-mile deployments experience frequent multi-day grid outages from hurricanes and wildfires. Standard Starlink terminals draw 50-100 W continuously, exhausting user-provided batteries within hours and leaving residents without any satellite link. No existing firmware logic adapts power draw to measured voltage or outage duration, causing complete loss of service when terrestrial backhaul is also down.

## Prior art
No matching prior-art patents were located in searches for satellite terminal voltage-sensing sleep logic or phased-array power scaling under outage conditions.

## Summary of the invention
The invention adds firmware routines in the terminal modem that read the DC input voltage sensor every second. When voltage drops below 10.5 V the modem enters a low-power sleep state for a duration between 30 s and 120 s. Upon wake the terminal performs a limited beam search only if voltage has recovered above 11.5 V for a hold time of 60 s. Outage statistics are logged to non-volatile memory for post-event analysis and a beacon mode activates after 72 h.

## Claims
1. A method in a satellite user terminal comprising: sensing input voltage at a rate of 1 Hz; entering a sleep state of duration between 30 s and 120 s when sensed voltage is below 10.5 V; waking and performing beam reacquisition only after sensed voltage exceeds 11.5 V for at least 60 s; and logging each sleep-wake cycle with timestamp and voltage values to non-volatile memory.
2. The method of claim 1 further comprising scaling transmit power of the phased array to 25 % of nominal during the first 30 s after wake when voltage remains between 11.0 V and 11.5 V.
3. The method of claim 1 wherein the sleep duration is computed as 30 s plus 5 s per 0.1 V below 10.5 V measured at entry.
4. The method of claim 1 further comprising transmitting a low-rate status packet containing the last three voltage samples upon successful beam reacquisition.
5. The method of claim 1 further comprising entering a permanent low-power beacon mode after 72 h of continuous outage, transmitting one status packet every 15 min until voltage falls below 9.5 V.
6. A satellite user terminal comprising a voltage sensing circuit coupled to the DC input, a modem processor executing the method of claim 1, and non-volatile memory storing the logged cycles.

## Brief description of the drawings
FIG. 1 shows the terminal power path and firmware control loop with reference numerals for battery, voltage sensor, modem processor, power amplifier, transmitter, scaling register and flash memory.

## Detailed description
Referring to FIG. 1, the satellite user terminal receives DC power from an external battery (12) through input connector (14). Voltage sensor (16) measures the voltage at the input with accuracy ±0.05 V and supplies digitized values to modem processor (18) at 1 Hz. When the processor detects voltage below 10.5 V it disables the phased-array power amplifier (20) and places the modem into a sleep interval of 30 s to 120 s calculated from the measured voltage. After the sleep interval the processor wakes, checks voltage again, and only enables the phased-array transmitter (22) and begins beam search if voltage has exceeded 11.5 V continuously for 60 s. During the initial 30 s after wake, if voltage is between 11.0 V and 11.5 V, transmit power is limited to 25 % of nominal via power scaling register (24). Each cycle is written to non-volatile flash (26) with timestamp, entry voltage, exit voltage, and sleep duration. Failure mode of sensor drift is handled by a 0.2 V hysteresis band and periodic self-calibration against a 5 V reference inside the modem. Outage duration exceeding 72 h triggers a permanent low-power beacon mode that transmits one status packet every 15 min until battery voltage falls below 9.5 V. All numeric thresholds and timing values stated in the claims are implemented exactly as described.
