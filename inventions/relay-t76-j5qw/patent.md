# Firmware-based adaptive frequency hopping for satellite terminal resilience

## Abstract
A firmware module in Starlink user terminals detects jamming signatures in the downlink spectrum and switches to pre-authorized backup frequency channels within 200 ms. The module uses on-board FFT analysis and a signed whitelist of 12 channels updated via authenticated satellite beacon. This prevents sustained outages during targeted interference.

## Problem
Adversaries can transmit narrowband or broadband jamming signals on the primary Ku-band downlink frequencies used by Starlink terminals. Once the terminal loses lock on the primary channel set, service remains unavailable until manual intervention or prolonged reacquisition, enabling deliberate blackouts.

## Prior art
- US20260019187A1 Communications System Using PAA and LMS Filter for Removing Jamming Signals: uses adaptive filtering on phased arrays but requires hardware modifications and does not address firmware channel switching.
- CN114978295B Cross-layer anti-interference method and system for satellite internet: describes protocol-level adjustments but lacks terminal-side spectrum monitoring and rapid authenticated handover.
- US10291347B2 Effective cross-layer satellite communications link interferences mitigation: covers interference classification yet relies on ground-station coordination with multi-second latency.

## Summary of the invention
The invention adds a jamming detection and autonomous channel-hopping routine to the terminal firmware. A 1024-point FFT runs continuously on the receive chain. Energy above threshold on more than 60 percent of bins within the active 250 MHz channel triggers a switch to the next channel in the signed whitelist. The switch occurs without loss of ephemeris or authentication state. Handover completes inside one superframe of 200 ms.

## Claims
1. A method in a satellite terminal comprising: continuously computing a 1024-point FFT on a 250 MHz downlink segment; declaring jamming when average bin energy exceeds -85 dBm on at least 60 percent of bins for three consecutive 50 ms windows; selecting the next frequency from a cryptographically signed list of twelve 250 MHz channels stored in non-volatile memory; retuning the local oscillator within 50 ms while preserving timing and authentication state; and resuming data reception on the new channel.
2. The method of claim 1 wherein the signed list is refreshed every 24 hours via a 128-byte beacon authenticated with ECDSA P-256.
3. The method of claim 1 wherein the FFT window overlap is 75 percent and a Hamming window is applied.
4. The method of claim 1 further comprising logging the jamming event with timestamp, center frequency, and duration to non-volatile memory for later telemetry upload.
5. The method of claim 1 wherein failure to acquire lock on the new channel within 300 ms causes reversion to the prior channel and a 10-second back-off before retry.
6. The method of claim 1 wherein the terminal reports detected jamming events to the constellation only after three successful handovers within any 60-minute interval.

## Brief description of the drawings
FIG. 1 shows the receive signal chain with FFT engine and channel selector logic. FIG. 2 shows the state machine for jamming detection and handover.

## Detailed description
The terminal baseband processor (12) receives digitized samples at 500 Msps from the analog front-end (14). A dedicated FFT engine (16) inside the processor computes a 1024-point transform every 1 ms using 75 percent overlap and a Hamming window. Bin magnitudes are compared against a programmable threshold of -85 dBm. When 614 or more bins exceed the threshold for three successive 50 ms observation windows, the jamming flag is asserted. The channel selector state machine (18) then reads the next entry from the 12-element whitelist table (20) stored in flash memory at address 0x0803C000. The local oscillator (22) is retuned to the new center frequency within 50 ms using a 12-bit DAC with 0.1 ppm settling accuracy. Timing recovery loops and the authentication session key remain intact. If lock is not achieved within 300 ms, the state machine reverts to the previous frequency and enforces a 10-second back-off. Every jamming event is written to a 4 kB circular buffer (24) containing UTC timestamp, center frequency, duration, and number of bins affected. The buffer is uploaded during the next authenticated telemetry window. The whitelist itself is replaced every 24 hours by a 128-byte ECDSA-signed beacon transmitted on the current channel; signature verification uses the public key burned at manufacture. All numeric thresholds and timeouts are stored in the same non-volatile region and may be updated only by signed firmware images.