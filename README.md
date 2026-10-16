PLC-Dual-Zone-Smart-Irrigation

> A fully autonomous irrigation control system for two agricultural zones based on real-time soil moisture measurement, implementing hysteresis logic, safe valve-pump sequencing, and multi-layer fault protection.

1. Overview
The system is designed to operate independently to minimize water consumption and prevent dry running of the pump. It relies on a PLC to read analog soil moisture sensors, process the signal, make irrigation decisions, and execute a time-protected irrigation cycle.

**Core Logic:** Analog Measurement -> Scaling & Validation -> Hysteresis Decision -> Sequential Execution -> Effectiveness Monitoring & Protection.

2. System Architecture - I/O List
| Type | Tag | Description |
| :--- | :--- | :--- |
| **Digital Input** | E-Stop, StartPB, StopPB | Safety and start-up circuit |
| **Selector** | AutoManualSel | Operation mode selection |
| **Analog Input** | SoilMoist_Z1/Z2_Raw | Soil moisture sensor 0-16383 |
| **Digital Input** | TankLowLevel | Tank low level sensor |
| **Manual Inputs** | ManualValve1/2PB, ManualPumpPB | Maintenance manual control |
| **Digital Output** | ValveZ1, ValveZ2, Pump | Final control elements |
| **Alarms** | AlarmLight, AlarmHorn, AnyAlarm | Alarm annunciation system |

3. Functional Description
3.1 Start-up and Mode Selection
- Latch/Unlatch for `SystemRun`: `(StartPB OR SystemRun) AND StopPB AND EstopOK`
- `AutoMode` follows `AutoManualSel`.

3.2 Signal Processing and Sensor Validation
- **SCP:** Scales 0-16383 to 0-100% moisture.
- **LIM:** Validates raw value within 300-15000. Out of range -> `Z_SensorFault`.

3.3 Hysteresis Control
- `SoilMoist < 35%` -> `OTL Z_Dry`
- `SoilMoist > 55%` -> `OTU Z_Dry`

3.4 Irrigation Cycle Sequence
1. Open zone valve `ValveZ`.
2. Delay `TON T5 (3s)` to ensure valve open.
3. Start pump on `PumpRequest AND T5.DN AND SystemRun AND NOT TankLow AND NOT AntiCycle`.
4. Interlock `XIO Z1Irrigating` in Z2 rung to prevent simultaneous zones.
5. Stop on target >55%.

4. Protection and Alarm System
| Alarm | Detection | Time | Action |
| :--- | :--- | :--- | :--- |
| `Z1/Z2 MaxRun` | `TON T0/T1` | 600s | Stop zone + Latch |
| `Z1/Z2 NoEffect` | `MOV StartMoist -> ADD 5%` with `TON T6/T7` | 300s | Latch - leak or dry-run |
| `Z1/Z2 SensorFault` | `LIM` | Instant | Inhibit zone |
| `TankLowAlarm` | `TankLowLevel` | Instant | Immediate pump stop |

5. No-Effect Detection Logic
```text
Rising Edge (Z_Irrig AND NOT Z_PrevIrr) -> MOV SoilMoistPct -> Z_StartMoist
Z_Irrig -> OTE Z_PrevIrr (State Memory)
Z_Irrig -> TON T6/T7 (300s)
ADD Z_StartMoist + 5.0 -> Z_TargetMoist
IF T6.DN AND SoilMoistPct < Z_TargetMoist -> OTL Z_NoEffectAlarm
This detects hidden faults like pump running dry, pipe leak, or closed valve.

6. Program Structure - 52 Rungs
- Rung 1-9: Safety, Scaling, Validation
- Rung 10-18: Hysteresis, Tank
- Rung 19-26: Requests, MaxRun
- Rung 27-32: Valve/Pump Sequencing
- Rung 33-44: No-Effect System
- Rung 45-52: Alarm Aggregation

7. Testing Protocol
1. Simulate moisture: 2000 (dry) to 10000 (wet).
2. Reduce T6/T7 Preset to 5s to test NoEffect alarm.
3. Activate TankLow during pump run to test immediate stop.

> **Note:** Logic validated in PLC Ladder Simulator. Field testing on physical PLC hardware is pending.
