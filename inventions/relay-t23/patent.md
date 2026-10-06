# Firmware-Controlled Satellite Attitude Bias for Reduced Optical Signature

## Abstract
A firmware module in a low Earth orbit satellite computes an attitude bias angle from ephemeris data and observatory coordinates. The bias orients the satellite body so that its solar array normal deviates 8 to 22 degrees from the sun vector during predicted passes over ground observatories. Attitude thrusters execute the bias within 120 seconds. The method reduces specular reflection into ground telescopes while maintaining power generation above 92 percent of nominal.

## Problem
Starlink satellites produce streaks in astronomical images when sunlight reflects from flat surfaces toward ground-based telescopes. Existing brightness mitigations rely on fixed black coatings or dielectric mirrors that add mass and degrade thermal performance. Software updates alone have not eliminated streaks because they do not dynamically adjust attitude for specific observatory locations and pass geometries.

## Prior art
- JP2022042950A, Drift-based rendezvous control, uses state vectors over finite time horizons but does not bias attitude for optical signature reduction.
- CN116946392B, Control method for electric propulsion inclination, applies multidimensional attitude bias for orbit maintenance but does not incorporate ground observatory coordinates or optical constraints.
- US10224868B2, Solar focusing device and method, adjusts spacecraft orientation for energy collection but does not address ground-based telescope interference.

## Summary of the invention
The invention adds a firmware routine that receives ground station ephemeris updates containing observatory latitude, longitude, and altitude. The routine calculates the minimum solar phase angle during each predicted pass and commands a yaw or pitch bias of 8 to 22 degrees when the phase angle would otherwise produce specular reflection into the telescope. Bias commands are issued only when solar array power margin exceeds 8 percent. Attitude control uses existing reaction wheels and magnetorquers with a 0.05 degree per second slew rate limit.

## Claims
1. A method in satellite firmware comprising: receiving observatory coordinates and pass prediction data; computing a required attitude bias angle between 8 and 22 degrees when predicted solar phase angle is less than 25 degrees; commanding attitude control actuators to apply the bias for the duration of the pass; and restoring nominal attitude after the pass.
2. The method of claim 1 wherein the bias is applied only when predicted solar array output remains above 92 percent of nominal.
3. The method of claim 1 wherein the bias slew rate is limited to 0.05 degrees per second.
4. The method of claim 1 wherein the firmware stores up to 200 observatory locations updated via downlink every 24 hours.
5. The method of claim 1 further comprising logging bias events with timestamps and power margin values for downlink.
6. The method of claim 1 wherein failure to achieve commanded attitude within 120 seconds triggers a safe-mode return to sun-pointing without bias.

## Brief description of the drawings
FIG. 1 shows satellite body axes, solar array, and observer line of sight with bias angle applied. FIG. 2 shows control loop block diagram with ephemeris input and actuator output.

## Detailed description
Satellite body (10) maintains a reference frame with +Z toward nadir, +X along velocity vector, and +Y completing the right-hand triad. Solar array (12) is mounted on the -Y face with normal vector (14). Onboard computer (16) executes firmware (18) that receives ephemeris packet (20) containing observatory latitude, longitude, altitude, and predicted pass times. Firmware (18) computes sun vector (22) from onboard ephemeris and calculates phase angle phi between sun vector (22) and observer line of sight (24). When phi is predicted below 25 degrees and power margin exceeds 8 percent, firmware (18) issues bias command (26) of magnitude theta equal to 15 plus or minus 7 degrees in yaw about the Z axis. Reaction wheel assembly (28) slews at maximum 0.05 degrees per second until attitude error is below 0.2 degrees. Magnetorquer (30) provides desaturation. Pass duration is typically 180 to 420 seconds. After pass completion, firmware (18) commands return to nominal sun-pointing attitude. If attitude error exceeds 1 degree after 120 seconds, firmware (18) aborts bias and enters sun-pointing safe mode. All bias events are written to log buffer (32) with 1-second resolution timestamps and array current readings. Ephemeris updates arrive via S-band downlink every 24 hours and overwrite the 200-entry observatory table stored in non-volatile memory (34). Thermal sensors (36) on solar array (12) confirm temperature remains within  -20 C to +80 C during biased operation. The firmware handles single-event upset by triple modular redundancy on the bias calculation routine.