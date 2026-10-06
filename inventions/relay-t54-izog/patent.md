# Firmware-Based Terminal Authentication and Frequency Agility for Satellite Networks

## Abstract
A firmware method in a satellite terminal receives signed commands over the downlink, verifies them with a pre-stored public key, and switches frequency bands or applies adaptive modulation only on valid commands. The method logs invalid attempts and falls back to last known good parameters after three failures within 60 seconds.

## Problem
External actors transmit spoofed disable commands or jamming signals that cause terminals to cease uplink during national blackouts. Terminals lack onboard verification of command origin and cannot autonomously select alternate frequencies when primary downlink is disrupted.

## Prior art
- No patents returned from searches on Starlink terminal anti jamming firmware or satellite terminal signal authentication software update.

## Summary of the invention
The invention adds firmware logic that authenticates every received command packet using ECDSA signatures before executing frequency changes or power adjustments. It maintains a table of three fallback frequency pairs and switches after detecting sustained packet error rate above 30 percent for 10 seconds. Sensors monitor SNR and BER; on threshold breach the firmware initiates a 5-second listen window on alternate band before resuming.

## Claims
1. A method in satellite terminal firmware comprising: receiving a command packet on downlink (12); verifying an ECDSA signature using a 256-bit public key stored in read-only memory (14); executing the command only if verification succeeds; otherwise incrementing a failure counter (16) and reverting to last authenticated frequency pair after three failures in 60 seconds.
2. The method of claim 1 further comprising measuring signal-to-noise ratio on primary band (18) and switching to a stored fallback band when SNR drops below 8 dB for more than 10 seconds.
3. The method of claim 2 wherein the firmware selects among three preloaded frequency pairs with 50 MHz separation and applies QPSK modulation on fallback.
4. The method of claim 1 further comprising logging invalid signature attempts with timestamp and RSSI value to non-volatile memory for later uplink when connectivity restores.
5. The method of claim 3 wherein the switch occurs only after a 5-second listen period confirming absence of valid beacon on the new band.
6. The method of claim 1 wherein the public key is updated only via signed over-the-air packets that themselves pass verification.

## Brief description of the drawings
FIG. 1 shows the terminal receiver chain with signature verification block and frequency table.

## Detailed description
The satellite terminal (20) contains a receiver front-end (22) tuned to the primary Ku-band downlink at 11.7 GHz with 500 MHz bandwidth. Incoming packets pass to a demodulator (24) that extracts the command field and attached 64-byte ECDSA signature. The firmware running on processor (26) compares the signature against the command hash using the public key held in OTP memory (14). Valid commands update the local oscillator (28) to a new frequency within ±2 MHz tolerance. When packet error rate exceeds 30 percent over a 10-second window measured by CRC failures in the baseband chip (30), the firmware (32) selects the next entry from the fallback table stored in flash (34). Each fallback pair is separated by 50 MHz. After switching, the terminal listens for 5 seconds for a valid beacon packet before transmitting. Invalid signature attempts increment counter (16); at three counts within 60 seconds the terminal reverts to the last verified parameters and raises an alert flag for ground reporting. All timing uses a 10 MHz TCXO reference (36) with 0.5 ppm stability. On power cycle the firmware boots from the last authenticated configuration to prevent persistent disable. Failure modes include key compromise, handled by requiring dual signatures for key rotation, and table corruption, detected by CRC-32 on the flash region with automatic reload from factory image. Dimensions of the frequency table allow storage of exactly three pairs at 8 bytes each. The mechanism tolerates 2 dB additional path loss on fallback bands by increasing transmit power up to +3 dB within regulatory limits.