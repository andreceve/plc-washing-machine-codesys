# PLC Control of a Washing Machine — CODESYS (IEC 61131-3)

Course project for **Control Systems Technologies**, University of Bologna.

PLC application that controls an automatic washing machine, developed in **CODESYS V3** and
tested on the **CODESYS Control Win V3** soft-PLC in simulation, with an HMI built with
CODESYS Visualization.

---

## Features

- **Washing programs** — three selectable programs, with prewash, wash and rinse phases
  shown on the HMI.
- **Water temperature control** — selectable reference of 30 / 60 / 90 °C, controlled by
  on/off (hysteresis) switching of the heating resistor around the reference.
- **Sequential control** of door lock, soap dosing, water intake and drain, and drum motor.
- **Input validation** before every cycle: a valid program and temperature must be
  selected, the door must be closed and there must be enough soap for one cycle.
- **Fault detection and alarms**:
  - heating resistor fault (temperature stays too low with the resistor on, or too high
    with it off);
  - water intake valve fault and drain valve fault (the door stays locked if the water
    cannot be drained);
  - missing soap, wrong program, wrong temperature.
- **Fault injection for testing** — simulation flags let the user force a resistor,
  intake-valve or drain-valve failure to check the alarm handling.

---

## Software architecture

The application follows the **Generalized Actuator (GA)** pattern: every physical device
is wrapped in a function block that exposes a common state machine
(`S_Init → S_Ready → S_Busy → S_Fault`), and a supervisory policy coordinates them through
a request / acknowledge handshake (`Start_Action → Wait_Comp → Set_Ok`).

| POU | Type | Role |
|---|---|---|
| `Policy` | Program (SFC) | Supervisory sequence of the washing cycle and alarm branches |
| `Protocol` | Function block | Validation of user inputs and sensors before starting |
| `DoorLockGA` | Function block | Door lock / unlock |
| `SoapGA` | Function block | Soap dosing |
| `WaterGA` | Function block | Water intake and drain with fault detection |
| `TemperatureGA` | Function block | Hysteresis temperature control and heater fault detection |
| `MotorGA` | Function block | Drum motor |
| `GAInstances` | Program | Cyclic execution of the GA instances |
| `Simulation_Program` | Program | Plant simulation (sensors, drum position, fault injection) |

Data types: `State_GA` (GA states), `BasicActions` (commands sent to the GAs),
`S_Action` (handshake states). Sensors, actuators, HMI signals and simulation flags are
declared in a global variable list.

Languages used: **Sequential Function Chart (SFC)** for the supervisory policy and
**Structured Text (ST)** for the function blocks.

---

## Repository structure

```
project/
  WashingMachine.project   % CODESYS V3.5 project (POUs, GVL, visualization, device config)
```

## How to run

1. Open `project/WashingMachine.project` with **CODESYS V3.5** (SP17 or later).
2. Select the device **CODESYS Control Win V3** and start the soft-PLC.
3. Build, log in, download the application and start it.
4. Open the visualization to select program and temperature, close the door, add soap and
   press *run*. Use the fault flags to test the alarms.

## Tools

CODESYS V3.5 · IEC 61131-3 (SFC, ST) · CODESYS Visualization · CODESYS Control Win V3

## Author

Andrea Cevenini — BSc in Automation Engineering, University of Bologna.
