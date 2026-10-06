# Distributed Firmware Collision Avoidance and Deorbit Manager for LEO Constellations

## Abstract
A firmware module resident on each satellite processes DoD debris ephemeris packets received over the inter-satellite link. It runs a 10 Hz Kalman filter fusing local GPS, star-tracker attitude and ground uplink data to predict conjunctions within 2 km and 48 hours. When probability exceeds 5e-5 the module commands a 0.2 m/s delta-V burn using existing reaction wheels and cold-gas thrusters. End-of-life triggers a 90-day autonomous deorbit sequence lowering perigee to 300 km using stored propellant budget. All logic executes in 32-bit fixed-point arithmetic on the flight computer with 50 ms worst-case latency.

## Problem
Thousands of satellites at 550 km altitude receive periodic DoD debris catalogs yet must perform avoidance maneuvers without saturating the ground station network. Manual or centralized planning creates latency and single-point overload. Satellites also require guaranteed autonomous disposal within five to seven years to limit long-term debris risk. Current on-board software lacks integrated, deterministic prediction and execution for both tasks.

## Prior art
- US12187462B2 Using genetic algorithms for safe swarm trajectory optimization: central genetic computation for chaser swarms; this invention moves prediction and execution to per-satellite firmware with fixed-point Kalman filters.
- US20220081132A1 Satellite constellation forming system, debris removal scheme: ground-planned constellation maintenance; this invention adds autonomous onboard conjunction probability and deorbit triggers.
- JP7261312B2 Collision avoidance support device: trajectory forecast storage and ground assistance; this invention performs closed-loop 10 Hz filtering and burn commands locally.

## Summary of the invention
The invention is a firmware task running at priority 3 on the satellite flight computer. It ingests 1 kB DoD TLE packets every 6 hours via ISL, maintains a 500-object local catalog, and computes minimum approach distances. Exceeding thresholds initiates a two-burn avoidance sequence using pre-loaded attitude and propulsion tables. A separate watchdog timer starts the deorbit sequence at 6.5 years or when propellant falls below 8 kg.

## Claims
1. A method executed by firmware on a satellite flight computer comprising: receiving ephemeris data; maintaining a local catalog of 500 objects; running a 32-bit fixed-point Kalman filter at 10 Hz; computing conjunction probability; and commanding a 0.2 m/s delta-V maneuver when probability exceeds 5e-5.
2. The method of claim 1 further comprising storing a 90-day deorbit sequence that lowers perigee to 300 km using 12 kg cold-gas propellant when satellite age exceeds 6.5 years.
3. The method of claim 1 wherein the Kalman filter fuses GPS position (1 sigma 5 m), star-tracker attitude (0.01 deg) and ground uplink velocity corrections.
4. The method of claim 1 wherein worst-case execution latency is 50 ms and all arithmetic uses Q15.16 fixed-point format.
5. The method of claim 2 wherein the deorbit sequence includes three perigee-lowering burns spaced 30 days apart with 0.5 m/s each.
6. The method of claim 1 further comprising a failure-mode check that aborts a maneuver if reaction-wheel speed exceeds 4500 rpm or propellant pressure drops below 12 bar.

## Brief description of the drawings
FIG. 1 shows the firmware data flow and sensor interfaces inside the satellite bus. FIG. 2 shows the thrust-vector geometry during an avoidance burn relative to the orbital velocity vector.

## Detailed description
The flight computer (12) runs a real-time operating system with the collision task (14) scheduled every 100 ms. Ephemeris packets arrive on the ISL transceiver (16) and are parsed into a 500-entry RAM table (18) with 64-bit timestamps. The Kalman filter (20) state vector contains position (x,y,z) and velocity (vx,vy,vz) in ECI frame using Q15.16 arithmetic. Measurement updates occur at 10 Hz from GPS receiver (22) outputting 1-sigma 5 m accuracy and star-tracker (24) supplying quaternion at 0.01 deg. Ground uplink (26) injects velocity bias corrections every orbit. Conjunction search compares the host trajectory against each catalog entry over a 48-hour horizon using a 2 km miss-distance threshold. Probability is computed from the covariance trace; values above 5e-5 trigger the maneuver sequencer (28). The sequencer loads a pre-computed two-burn profile from flash (30): first burn 0.2 m/s at true anomaly 180 deg using cold-gas thrusters (32) mounted at 30 deg cant angle, second burn 24 hours later to restore the original semi-major axis. Attitude is held by reaction wheels (34) with speed limit 4500 rpm. If wheel speed or tank pressure (sensor 36) violates limits the maneuver aborts and logs fault code 0xA3. End-of-life logic (38) reads the mission timer (40) and propellant gauge (42). At 6.5 years or 8 kg remaining the sequencer executes three 0.5 m/s burns spaced 30 days to reach 300 km perigee. All parameters reside in non-volatile tables uploaded once per year. Total code size is 18 kB with 4 kB stack usage.