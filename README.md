# PAT Controller – Runtime & Architecture Overview

This document provides a consolidated engineering overview of the PAT Controller system, including runtime model, layered architecture, processes, threads, devices, transports, state-machine behavior, and control-loop execution. It includes diagrams to support system comprehension and future development.

---

# 1. System Overview

The PAT Controller orchestrates sensors, actuators, and peripherals participating in an optical pointing experiment. It runs on a Raspberry Pi and interacts with a Teensy 4.1 microcontroller, a camera process, a GUI, and a Logger.

```mermaid
flowchart LR
    GUI[GUI Process]
    LOGGER[Logger Process]
    CAM[Camera Process]
    PI[PI: PAT Controller]
    T4[Teensy 4.1]

    GUI <-- Telecommands / Telemetry --> PI
    LOGGER <-- Telemetry --> PI
    PI <-- TCP Frames --> CAM
    PI <-- UART --> T4
```

---

# 2. Runtime Architecture

## 2.1 Processes

### Diagram – System Processes

```mermaid
flowchart TB
    subgraph RP[Raspberry Pi]
        CTRL[PAT Controller Process]
        CAM[Camera Process]
    end
    GUI[External GUI Process]
    LOG[External Logger Process]
    MCU[Teensy 4.1]

    GUI -->|Telecommands| CTRL
    CTRL -->|Telemetry| GUI
    CTRL -->|Telemetry| LOG
    CAM -->|Frames (TCP)| CTRL
    CTRL -->|UART| MCU
```

---

## 2.2 Threads inside the PAT Controller

```mermaid
flowchart LR
    MAIN[Main Loop Thread]
    SERIAL[SerialSharedTransport]
    SOCKET[SocketSharedTransport]
    TEL[Telemetry Thread]
    LOG[Logging Thread]
    TC[Telecontrol Thread or Non-blocking Poll]

    MAIN --> SERIAL
    MAIN --> SOCKET
    MAIN --> TEL
    MAIN --> LOG
    MAIN --> TC
```

Main loop is strictly non-blocking and isolated from IO.

---

# 3. Layered Architecture

```mermaid
flowchart TB
    APP[Application Layer]
    DEV[Device & Domain Layer]
    TR[Transport & MCU Link Layer]
    HW[Hardware Layer]

    APP --> DEV --> TR --> HW
```

## 3.1 Application Layer

Components include:

* Controller
* StateManager and concrete states
* Sensors/Actuators/Peripherals Managers
* TelemetryManager
* TelecontrolManager
* ConfigManager

### Diagram – Application Layer Internals

```mermaid
flowchart LR
    SM[StateManager]
    SEN[SensorsManager]
    ACT[ActuatorsManager]
    PER[PeripheralsManager]
    TM[TelemetryManager]
    TCM[TelecontrolManager]
    CM[ConfigManager]

    SM --> ACT
    SM --> PER
    SM --> SEN
    SET((Telecommands)) --> TCM --> SM
    SEN --> TM
    ACT --> TM
    PER --> TM
    SM --> TM
```

---

## 3.2 Device & Domain Layer

Logical devices with high‑level APIs, independent of hardware.

### Example Diagram

```mermaid
flowchart TB
    subgraph Sensors
        PM[Power Meter]
        QD[4QD]
        TH[Thermal]
        CAM[Camera]
    end

    subgraph Actuators
        OPA[OPA Device]
        PS[Phase Shifter]
    end

    subgraph Peripherals
        EDFA[EDFA]
        SFP[SFP]
        EPS[EPS]
    end
```

---

## 3.3 Transport & MCU Link Layer

### Diagram – Transport Model

```mermaid
flowchart LR
    SHSER[SerialSharedTransport]
    SHSOC[SocketSharedTransport]
    SHMOCK[SharedMockTransport]

    CH1[PMTransport]
    CH2[FourQDTransport]
    CH3[ThermalTransport]
    CH4[CameraTransport]
    CH5[OPACommandTransport]
    CH6[PhaseShifterTransport]
    CH7[Peripherals Transports]

    CH1 --> SHSER
    CH2 --> SHSER
    CH3 --> SHSER
    CH5 --> SHSER
    CH6 --> SHSER
    CH7 --> SHSER
    CH4 --> SHSOC
    style SHMOCK stroke-dasharray: 5 5
```

---

## 3.4 Hardware Layer

```mermaid
flowchart TB
    T4[Teensy 4.1]
    PM[PM Sensors]
    QD[4QD Sensors]
    TH[Thermals]
    OPA[OPAs]
    PS[Phase Shifters]
    PER[Peripherals]
    CAM[Camera]

    T4 --> PM
    T4 --> QD
    T4 --> TH
    T4 --> OPA
    T4 --> PS
    T4 --> PER
    CAM -. TCP .-> PI
```

---

# 4. Teensy Firmware Architecture

```mermaid
stateDiagram-v2
    [*] --> T_IDLE
    T_IDLE --> T_MANUAL : set_manual
    T_MANUAL --> T_TRACKING : start_tracking
    T_TRACKING --> T_IDLE : stop

    state T_TRACKING {
        HC: Hill‑Climb (10–50 Hz)
        FL: Fast Loop (1 kHz)
        FL --> HC
        HC --> FL
    }
```

Teensy executes low‑level loops and remains autonomous on link loss.

---

# 5. Operational States (Pi Side)

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Manual : telecommand
    Idle --> Acquisition : telecommand
    Manual --> Idle : telecommand
    Acquisition --> Tracking : lock_acquired
    Tracking --> Acquisition : tracking_lost
    Tracking --> Idle : stop
```

---

# 6. Control Loop Execution Model

```mermaid
flowchart TB
    TC[Fetch Telecommands]
    PROC[Process Telecommands]
    READ[Acquire Sensor Data]
    EXEC[Execute Active State]
    CMD[Issue Actuator Commands]
    TEL[Build Telemetry]
    LOG[Generate Log Events]
    TIME[Loop Timing Sync]

    TC --> PROC --> READ --> EXEC --> CMD --> TEL --> LOG --> TIME --> TC
```

---

# 7. Telecontrol Architecture

```mermaid
flowchart LR
    GUI[GUI / Automation]
    TCT[Telecontrol Transport]
    TCM[TelecontrolManager]
    SM[StateManager]
    CM[ConfigManager]
    AM[ActuatorsManager]
    SAF[SafetyManager]

    GUI --> TCT --> TCM
    TCM --> SM
    TCM --> CM
    TCM --> AM
    TCM --> SAF
```

---

# 8. Telemetry & Logging Pipelines

### Telemetry

```mermaid
flowchart LR
    TM[TelemetryManager]
    TT[Telemetry Thread]
    GUI[GUI]
    LOG[Logger]

    TM --> TT --> GUI
    TT --> LOG
```

### Logging

```mermaid
flowchart LR
    EVT[Structured Log Events]
    LQ[logging_queue]
    LT[Logging Thread]
    FILES[Log Files]

    EVT --> LQ --> LT --> FILES
```

---

# 9. Migration Strategy

The architecture supports migration to a more MCU‑centric execution model without redesigning upper layers. Abstractions remain stable as Teensy responsibilities grow.

---

# 10. Summary

The PAT controller is modular, layered, soft–real-time, and future‑proof. It supports:

* Deterministic non-blocking main loop
* Multi-device, multi-transport operations
* External GUI and Logger integration
* Fully mocked environments for development
* Teensy‑accelerated execution with increasing autonomy

This README serves as the reference for developers implementing, testing, and extending the system.
