# PMS-Prototype
Power Management System that includes a MPPT solar input, USB-C PD input, BMS w/ active balancing, load output, power path control, and power monitoring.


## Goals

- Solar input with MPPT
- USB-C PD input for charging
- BMS with active cell balancing
- Regulated load output
- "Power path" control (Select power source for load [battery, solar, USB], select battery charging/discharging/disconnected, and select source for battery charging)
- Power monitoring (Current, voltage, power input/output for each sink/source)
- Protections (Temperature, OVP, UVP, OCP, RVP, RCP, etc)
- Programmable control (eg. SPI, CAN Bus, RS232/TTL, etc)

## V1.0 Scope

- One solar panel input (12v 100W - 24v Vos 6A Isc)
  - 6A limit (19v Vmp: 100W / 19v = 5.26A)
  - 24v limit
- One load output (12v @ 3A = 36W)
- 1S battery (no balancing)
- Solar (~18v-20v) -> MPPT Buck -> Battery (~13v-14.5v) -> Buck -> Load (12v)
- Battery current/voltage sensing and MOSFET disconnect switch
- Control battery charge current by subtracting load current from total current draw? (removes the need for battery current sensing)
  - Or, use battery current and total current to determine load current (better safety for battery)

## Design 'Units'

- MCU / Control
- Solar Input / MPPT
- USB-C PD Input
- BMS / Battery
- Load Output / Regulation
- "Power Path" Switching

## Planning / Parts

### MCU / Control

Main contender is the STM32G4x4:

- Cortex-M4 (with FPU and DSP instructions) running at 170 MHz
- Rich advanced analog peripherals (comparator, op-amps, DAC)
- ADC with hardware oversampling (16-bit resolution)
- Dual-bank Flash memory with error-correcting code (ECC) (supports in-field firmware upgrades)
- High-resolution timer version 2 (HRTIM for digital power conversion)
- USB Type-C interface with Power Delivery including physical layer (PHY)

### Solar Input / MPPT

- Synchronous Buck Topology
- 48v input (to support 2x series 12v 100W panels with an Voc of ~24v each)
- 15A input (to support 2x series 12v 100W panels with an Isc of ~6A each)
- (Idea) Output is programmable: Can be set to match load output or controlled to charge the battery (CV/CC)

### Power Path

- Individual Enable/Disable for Solar/USB/Battery/Load
- Input priority control
- Output priority control

- Solar/USB/Battery/Load all in parallel
- Each sink/source either takes from or contributes to the pool of energy
- The battery has a disconnect, but can't regulate
- If the system voltage drops, you can either:
  - Increase the power input (switching battery from sink to source?)
  - Decrease load (Limit the battery charge current or send a signal to )
- The PMB should be able to identify when the maximum solar power available isn't being used
  - Increase battery charging speed
  - Signal to the main vessel controller to use more power (speed up, perform comms transmissions, etc)

- USB-C PD || Solar || Battery || 
- Only one power input (Solar or USB-C) 
  - USB->Buck (USB supply must be at least load voltage)
  - Solar->MPPT->Sys
- Battery in parallel with load
- Sys->

## Other Parts

- Hall effect current sensor:
  - ACS712 (popular but noisy!)
  - LEM HLSR50 (expensive - $16 - but 50A, shielded, internal voltage ref, can be used with differential ADC eliminating need for calibration)
- MOSFET Driver for synconous buck converter:
  - IR2101 and IR2110 (seperate PWM inputs for high and low side MOSFETS)
  - IR2104 (One PWM input drives both high and low side MOSFETS)
    - Be careful!!! Setting the PWM too low causes the high side to stay off and the low side to stay on, resulting in a short circuit if a battery is attached and potentially burning out the low-side MOSFET.
    - `PWM Floor Duty Cycle = (Output Voltage / Input Voltage) * 100)`
    - When PWM signal has a lower duty cycle than the computed PWM floor value, the current flows in reverse and causes the low-side MOSFET to conduct when it isn't supposed to be conducting.
- Isolated DC-DC Converter:
  - B1212S
- ADC:
  - ADS1115 (16bit 860sps) / ADS1015 (12bit 3300sps)
  - (the STM32G4x4's internal ADC might be good enough?)
- MOSFET:
  - CSD19505

# Resources

## Relion LiFePO3 Battery Specs

**RB20-LT:**

- Recommended Charge Voltage: 14.2 V - 14.6 V
- BMS Charge Voltage Cut-Off: 15.6 V (3.9 ±0.025 vpc) (1 ±0.3 s)
- Reconnect Voltage:          14.6 V (3.8 ±0.050 vpc)
- Balancing Voltage:          14.4 V (3.6 ±0.025 vpc)
- Recommended Low Voltage Disconnect: 11.0 V
- Discharge Under-Voltage Protection: 8.0 V (2.0 ±0.08 vpc) (20 ±8 ms)
- Reconnect Voltage:                  9.04V (2.26 ±0.34 vpc)

## Battery Charging

- https://en.wikipedia.org/wiki/Battery_balancing

## MPPT Solar Charger

- https://www.instructables.com/DIY-1kW-MPPT-Solar-Charge-Controller/
  - https://drive.google.com/drive/folders/1Sd2jWAb-F8NAXlJ6PLdhcnPDQV0alD15
- https://www.instructables.com/ARDUINO-SOLAR-CHARGE-CONTROLLER-Version-30/
  - https://microcontrolere.wordpress.com/2016/12/16/mppt-solar-charger/
- https://www.opengreenenergy.com/
  - https://www.opengreenenergy.com/arduino-solar-charge-controller-v2-02/
- https://www.instructables.com/How-to-Design-and-Build-a-MPPT-Solar-Charger-Using/
- https://www.mathworks.com/discovery/mppt-algorithm.html
- In an MPPT buck converter, when the PV voltage is lower than the battery voltage, the high-side MOSFET body diode causes current leakage from the batteries into the solar panels.
  - Solution: Add reverse blocking MOSFET
  - Drive the MOSFET with and isolated DC-DC converter (B1212S)
