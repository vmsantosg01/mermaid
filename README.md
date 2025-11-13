# mermaid
# PAT Controller System

## Overview
A modular soft–real-time control system for orchestrating sensors, actuators, and peripherals in an optical pointing experiment.

## High-Level System Architecture
```mermaid
flowchart LR

    subgraph GUI["External GUI Process"]
        GUI_TC["Telecommands"]
        GUI_TLM["Telemetry Viewer"]
    end

    subgraph LOGGER["External Logger Process"]
        LOGGER_SUB["Telemetry/Log Subscriber"]
    end

    subgraph CAMPROC["Camera Process"]
        CAM_CAP["High-Frame-Rate Capture"]
        CAM_TCP["Frame Server (TCP)"]
        CAM_CAP --> CAM_TCP
    end

    subgraph PI["Raspberry Pi — PAT Controller"]

        subgraph THREADS["Controller Threads"]
            ML["Main Loop Thread"]
            SERIAL["Serial Transport Thread"]
            SOCK["Socket Transport Thread"]
            TELE["Telemetry Thread"]
            LOGTH["Logging Thread"]
        end

        subgraph MANAGERS["Application Layer"]
            SM["StateManager"]
            SEN["SensorsManager"]
            ACT["ActuatorsManager"]
            PER["PeripheralsManager"]
            SAF["SafetyManager"]
            CONF["ConfigManager"]
            TCTL["TelecontrolManager"]
            TLMGR["TelemetryManager"]
        end

        subgraph TRANS["Transport Layer"]
            ST["SerialSharedTransport"]
            CT["SocketSharedTransport"]
            MT["MockTransport"]
        end
    end

    subgraph TEENSY["Teensy 4.1 MCU"]
        T_SENS["Sensors: PM, 4QD, Thermal"]
        T_ACT["Actuators: OPA, Phase Shifter"]
        T_PER["Peripherals: EDFA, SFP, EPS"]
        T_PROTO["MCU Protocol Engine"]
    end

    GUI_TC --> TCTL
    TCTL --> ML

    ML --> TLMGR
    TLMGR --> TELE
    TELE --> GUI_TLM
    TELE --> LOGGER_SUB

    CAM_TCP --> SOCK
    SOCK --> CT
    CT --> SEN

    SERIAL --> ST
    ST --> TEENSY
    TEENSY --> ST
    ST --> SERIAL

    T_SENS --> ST
    ACT --> ST
    PER --> ST
    ST --> SEN

    ML --> SM
    ML --> SEN
    ML --> ACT
    ML --> PER
    ML --> SAF
    ML --> CONF

    ML --> LOGTH
```

## Control Loop Sequence
```mermaid
sequenceDiagram
    participant TC as TelecontrolManager
    participant ML as MainLoop
    participant SEN as SensorsManager
    participant SM as StateManager
    participant ACT as ActuatorsManager
    participant PER as PeripheralsManager
    participant SAF as SafetyManager
    participant TM as TelemetryManager
    participant LOG as LoggingThread

    TC->>ML: Poll telecommands
    ML->>TC: Process telecommands

    ML->>SEN: Read sensors
    SEN-->>ML: SensorsSnapshot

    ML->>SM: Execute state logic
    SM-->>ML: Outputs

    ML->>SAF: Safety checks
    SAF-->>ML: Status

    ML->>ACT: Actuator cmds
    ML->>PER: Peripheral cmds

    ML->>TM: Build telemetry
    TM-->>ML: Queued

    ML->>LOG: Log events

    ML->>ML: Timing sync
```

## Layered Architecture
```mermaid
flowchart TB

    subgraph APP["Application Layer"]
        CTRL["Controller"]
        SM["StateManager"]
        SENM["SensorsManager"]
        ACTM["ActuatorsManager"]
        PERM["PeripheralsManager"]
        SAFM["SafetyManager"]
        TCTL["TelecontrolManager"]
        TLM["TelemetryManager"]
        CONF["ConfigManager"]
    end

    subgraph DEV["Device & Domain"]
        DEV_SENS["Sensor Devices"]
        DEV_ACT["Actuator Devices"]
        DEV_PER["Peripheral Devices"]
    end

    subgraph TRANS["Transport & MCU Link"]
        CH_TRANS["ChannelTransports"]
        SH_TRANS["SharedTransports"]
        MCU_PROTO["MCU Protocol"]
    end

    subgraph HW["Hardware"]
        HW_TEENSY["Teensy 4.1"]
        HW_CAM["Camera Process"]
        HW_OPTICS["Optical Hardware"]
        HW_PER["Peripheral Hardware"]
        HW_SENS["Sensors"]
    end

    APP --> DEV
    DEV --> TRANS
    TRANS --> HW
```

## MCU Communication
```mermaid
sequenceDiagram
    participant PI as RaspberryPi
    participant MCU as Teensy41
    participant SENS as MCU_Sensors
    participant ACT as MCU_Actuators
    participant PER as MCU_Peripherals

    loop Cycle
        PI->>MCU: MCUCommand
        MCU-->>PI: ACK
    end

    MCU->>SENS: Poll sensors
    SENS-->>MCU: Data

    MCU->>ACT: Update actuators
    MCU->>PER: Update peripherals

    MCU-->>PI: MCUSnapshot
```

## State Machine
```mermaid
stateDiagram-v2
    Idle --> Manual
    Idle --> Acquisition
    Idle --> Tracking
    Idle --> Calibration

    Manual --> Idle
    Manual --> Tracking
    Manual --> Error

    Acquisition --> Tracking
    Acquisition --> Idle
    Acquisition --> Error

    Tracking --> Idle
    Tracking --> Error
    Tracking --> Manual

    Calibration --> Idle
    Calibration --> Error

    Error --> Idle
```

## Thread Architecture
```mermaid
flowchart TB

    subgraph MAIN["PAT Controller Threads"]
        ML["Main Loop"]
        SERIAL["Serial Transport"]
        SOCK["Socket Transport"]
        TELE["Telemetry"]
        LOGTH["Logging"]
    end

    ML --> SERIAL
    ML --> SOCK
    ML --> TELE
    ML --> LOGTH

    SERIAL --> ML
    SOCK --> ML
    TELE --> ML
    LOGTH --> ML
```
