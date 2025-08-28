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

- Hall effect current sensor: ACS712 (noisy!)

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

## MPPT Solar Charger

- https://www.instructables.com/DIY-1kW-MPPT-Solar-Charge-Controller/
- https://www.instructables.com/ARDUINO-SOLAR-CHARGE-CONTROLLER-Version-30/
- https://www.opengreenenergy.com/
- https://www.opengreenenergy.com/arduino-solar-charge-controller-v2-02/
- https://www.instructables.com/How-to-Design-and-Build-a-MPPT-Solar-Charger-Using/
- https://www.mathworks.com/discovery/mppt-algorithm.html
