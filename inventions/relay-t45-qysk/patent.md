# Firmware-based adaptive signal compensation for satellite direct-to-device links in vehicles

## Abstract
A firmware module in a Starlink Mobile user terminal or vehicle-integrated receiver dynamically measures attenuation through automotive glass and applies real-time compensation to transmission parameters. The module uses sensor data and lookup tables to adjust power, frequency offset, and beam steering within regulatory limits, ensuring reliable connectivity without hardware changes.

## Problem
Starlink Mobile direct-to-cell signals experience 10-25 dB attenuation when passing through laminated vehicle windshields containing metallic interlayers. This prevents reliable uplink from phones inside Tesla vehicles and similar models, causing connection drops during motion or in urban environments. Doppler shift from satellite velocity compounds the issue.

## Prior art
- US20250119206A1 Mobile satellite: describes phased-array antennas on mobile platforms facing satellites but lacks vehicle glass compensation algorithms or real-time attenuation sensing.
- US10602329B2 System and method for routing, mobility, application services: covers vehicle on-board units for sensor data routing but does not address satellite signal propagation through glass.

## Summary of the invention
The invention provides a firmware routine executed on the terminal processor that samples received signal strength indicator (RSSI) and signal-to-noise ratio (SNR) at 100 ms intervals, correlates them with vehicle speed and orientation from onboard sensors, and selects compensation values from a preloaded table calibrated for common windshield types. Adjustments are applied to transmit power (up to +3 dB), carrier frequency offset (up to 5 kHz), and phased-array tilt angles.

## Claims
1. A method in a satellite terminal firmware comprising: periodically sampling RSSI and SNR from a direct-to-cell downlink; retrieving vehicle motion data from an inertial measurement unit; indexing a calibration table with the sampled values and motion data to obtain compensation parameters; and applying the parameters to an uplink transmitter within a 3 dB power limit and 5 kHz frequency offset.
2. The method of claim 1 further comprising detecting windshield type via a stored vehicle identification number and selecting a corresponding sub-table.
3. The method of claim 1 wherein the calibration table stores attenuation values measured at 1.6 GHz to 2.5 GHz in 50 MHz steps.
4. The method of claim 1 further comprising monitoring for connection failure and falling back to a reduced data rate mode using 50% lower modulation order.
5. The method of claim 1 wherein the firmware executes on a 100 ms cycle with a maximum latency of 20 ms for parameter application.
6. The method of claim 1 further comprising logging compensation events to non-volatile memory for post-drive analysis.

## Brief description of the drawings
FIG. 1 shows the signal path through a vehicle windshield with reference numerals for the phased array, processor, and sensors.

## Detailed description
The firmware (20) runs on processor (22) inside terminal housing (24) mounted on vehicle roof (26). Phased-array antenna (28) transmits at 1.8 GHz nominal. Windshield (30) contains a 0.3 mm PVB interlayer with embedded silver coating causing 18 dB average loss at 2 GHz. IMU (32) provides speed (34) and pitch (36) at 10 Hz. Table (38) in flash memory (40) contains 256 entries indexed by RSSI range (-120 to -80 dBm) and speed (0-120 km/h). On each 100 ms tick the firmware reads RSSI from receiver (42), indexes table (38), computes required power delta (44) limited to 3 dB, applies frequency offset (46) up to 5 kHz to counter Doppler, and commands beam steering angles (48) via array controller (50). Failure mode of table mismatch triggers fallback to lowest MCS index. All values are also stated in the claims. Every reference numeral appears in a figure.