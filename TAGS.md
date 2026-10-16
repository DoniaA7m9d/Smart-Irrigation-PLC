# TAG LIST - Allen-Bradley SLC500 / MicroLogix
PLC Dual-Zone Smart Irrigation - 2 Zones / 1 Pump / 1 Tank
Note: Logic validated in PLC Ladder Simulator. Field testing on physical PLC hardware is pending.

## 1. Digital Inputs - I:0 (Slot 0)
| Tag Name | Address | Type | Description |
| :--- | :--- | :--- | :--- |
| Start_PB | I:0/0 | NO | Start Push Button |
| Stop_PB | I:0/1 | NC | Stop PB - Fail-Safe NC |
| E_Stop | I:0/2 | NC | Emergency Stop - Fail-Safe NC |
| Auto_Manual_Sel | I:0/3 | Selector | Auto=1 / Manual=0 |
| Tank_Low_Level | I:0/4 | Sensor | Tank Low Level - NC |
| Manual_Pump_PB | I:0/5 | NO | Manual Pump for Maintenance |
| Manual_Valve1_PB | I:0/6 | NO | Manual Valve Z1 |
| Manual_Valve2_PB | I:0/7 | NO | Manual Valve Z2 |
| Alarm_Reset_PB | I:0/8 | NO | Reset All Latched Alarms |

## 2. Digital Outputs - O:0
| Tag Name | Address | Description |
| :--- | :--- | :--- |
| Pump | O:0/0 | Main Irrigation Pump Contactor |
| Valve_Z1 | O:0/1 | Zone1 Solenoid Valve |
| Valve_Z2 | O:0/2 | Zone2 Solenoid Valve |
| Alarm_Horn | O:0/3 | Alarm Buzzer - Silence with Reset |
| Alarm_Light | O:0/4 | Alarm Light - ON if Any_Alarm |
| System_Run_Lamp | O:0/5 | System Running Indicator |

## 3. Analog Inputs
| Tag Name | Address | Scaling |
| :--- | :--- | :--- |
| SoilMoist_Z1_Raw | AI:0 / N7:0 | 0 - 16383 Raw |
| SoilMoist_Z2_Raw | AI:1 / N7:1 | 0 - 16383 Raw |

## 4. Float Files - F8 (Real Values)
| Tag Name | Address | Value / Purpose |
| :--- | :--- | :--- |
| SoilMoist_Z1_Pct | F8:0 | 0.0 - 100.0% Scaled Z1 |
| SoilMoist_Z2_Pct | F8:1 | 0.0 - 100.0% Scaled Z2 |
| Z1_Low_SP | F8:2 | 35.0% - Start Irrigation |
| Z1_High_SP | F8:3 | 55.0% - Stop Irrigation |
| Z2_Low_SP | F8:4 | 35.0% |
| Z2_High_SP | F8:5 | 55.0% |
| Z1_Start_Moist | F8:6 | Moisture Saved at Start of Irrigation |
| Z2_Start_Moist | F8:7 | Moisture Saved at Start of Irrigation |
| Z1_Target_Moist | F8:8 | Start_Moist + 5% - For NoEffect Check |
| Z2_Target_Moist | F8:9 | Start_Moist + 5% |

## 5. Integer Files - N7 (Limits)
| Tag Name | Address | Value |
| :--- | :--- | :--- |
| Raw_Min | N7:2 | 0 |
| Raw_Max | N7:3 | 16383 |
| Valid_Min | N7:4 | 300 - Sensor Wire-Break Detection |
| Valid_Max | N7:5 | 15000 - Sensor Short Detection |

## 6. Binary Files - B3 (Status Bits)
| Tag Name | Address | Function |
| :--- | :--- | :--- |
| System_Run | B3:0/0 | System Latch (StartPB OR SystemRun) AND StopPB |
| Auto_Mode | B3:0/1 | Follows Auto_Manual_Sel |
| E_Stop_OK | B3:0/2 | E-Stop Healthy |
| Z1_Dry | B3:0/3 | OTL if <35% / OTU if >55% - Hysteresis |
| Z1_Wet | B3:0/4 | Above High SP |
| Z1_Irrigating | B3:0/5 | Zone1 Active Request |
| Z2_Dry | B3:0/6 | Hysteresis Memory Z2 |
| Z2_Wet | B3:0/7 | Above High SP Z2 |
| Z2_Irrigating | B3:0/8 | Zone2 Active Request |
| Z1_Sensor_Fault | B3:0/9 | LIM Check Failed Z1 |
| Z2_Sensor_Fault | B3:0/10 | LIM Check Failed Z2 |
| Tank_Low_Alarm | B3:0/11 | Immediate Pump Stop |
| Z1_No_Effect_Alarm | B3:0/12 | Latched - Pipe Leak / Dry Run |
| Z2_No_Effect_Alarm | B3:0/13 | Latched |
| Z1_MaxRun_Alarm | B3:0/14 | Latched - 10min Timeout |
| Z2_MaxRun_Alarm | B3:0/15 | Latched |
| Any_Alarm | B3:1/0 | OR of All Alarms Above |
| Pump_Request | B3:1/1 | Z1_Irrigating OR Z2_Irrigating |
| Zone_Priority_Z1 | B3:1/2 | Interlock - Only One Zone at a Time |
| Z1_Prev_Irr | B3:1/3 | Rising Edge Memory for MOV |
| Z2_Prev_Irr | B3:1/4 | Rising Edge Memory |

## 7. Timers - T4
| Tag Name | Address | Preset | Purpose |
| :--- | :--- | :--- | :--- |
| Z1_MaxRun_TMR | T4:0 | 600s | Max Irrigation Time Protection |
| Z2_MaxRun_TMR | T4:1 | 600s | |
| Z1_Soak_TMR | T4:2 | 900s | Anti-Cycle - Soil Soaking Time |
| Z2_Soak_TMR | T4:3 | 900s | |
| Pump_AntiCycle_TMR | T4:4 | 120s | Pump Protection |
| Valve_Open_Delay_TMR | T4:5 | 3s | Open Valve BEFORE Pump - Safe Sequence |
| Z1_NoEffect_TMR | T4:6 | 180s | Wait 3min then check if moisture increased |
| Z2_NoEffect_TMR | T4:7 | 180s | |

## 8. Counters - C5
| Tag Name | Address | Purpose |
| :--- | :--- | :--- |
| Z1_Irrigation_Count | C5:0 | Count Daily Irrigations Z1 |
| Z2_Irrigation_Count | C5:1 | Count Daily Irrigations Z2 |
