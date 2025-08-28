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

## Design 'Units'

- MCU / Control
- Solar Input / MPPT
- USB-C PD Input
- BMS / Battery
- Load Output / Regulation
- "Power Path" Switching
