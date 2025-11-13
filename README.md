# PAT Controller System

This README provides a complete architectural overview of the **PAT Controller**, including runtime model, abstraction layers, processes, state machines, telemetry/telecontrol paths, and future migration strategy. It also includes **Mermaid diagrams** rendered natively by GitHub.

---

## 1. High-Level System Overview

The PAT Controller is a modular soft–real‑time supervision and control system running on a Raspberry Pi. A Teensy 4.1 microcontroller provides low‑level IO, sensor aggregation, and actuator control. External processes such as the GUI and Logger interact via network transports.

### High-Level Architecture Diagram

```mermaid
flowchart TD
    GUI[External GUI Process]
    LOGGER[External Logger Process]
    CAMERA[Camera Process]
    RPI[PAT Controller Process on Raspberry Pi]
    TEENSY[Teensy 4.1 Microcontroller]

    GUI <--> |Telecommands / Telemetry| RPI
    LOGGER <--> |Telemetry Stream| RPI
    CAMERA <--> |TCP Frames| RPI
    RPI <--> |UART Protocol| TEENSY
```

---

## 2. Layered Architecture

The system is structured into four main layers.

### Layer Diagram

```mermaid
flowchart TD
    A[Application Layer]
    B[Device & Domain Layer]
    C[Transport & MCU Link Layer]
    D[Hardware Layer]

    A --> B --> C --> D
```

---

## 3. Application Layer

Contains the main logic, managers, states, telemetry, telecontrol, configuration and safety.

### Application Layer Diagram

```mermaid
flowchart TD
    Controller[Controller]
    StateManager[State Manager]
    TelemetryManager[Telemetry Manager]
    TelecontrolManager[Telecontrol Manager]
    SensorsManager[Sensors Manager]
    ActuatorsManager[Actuators Manager]
    PeripheralsManager[Peripherals Manager]
    ConfigManager[Config Manager]
    SafetyManager[Safety Manager]

    Controller --> StateManager
    Controller --> SensorsManager
    Controller --> ActuatorsManager
    Controller --> PeripheralsManager
    Controller --> TelemetryManager
    Controller --> TelecontrolManager
    Controller --> ConfigManager
    Controller --> SafetyManager
```

---

## 4. Device & Domain Layer

This layer models every sensor, actuator, and peripheral using stable, high-level interfaces. All hardware interactions go through channel transports.

### Device Layer Diagram

```mermaid
flowchart LR
    subgraph Sensors
        PM[PowerMeterSensor]
        QD[FourQDSensor]
        TH[ThermalSensor]
        CAM[CameraSensor]
    end

    subgraph Actuators
        OPA[OPADevice]
        PS[PhaseShifterDevice]
    end

    subgraph Peripherals
        EDFA[EDFADevice]
        SFP[SFPDevice]
        EPS[EPSDevice]
    end
```

---

## 5. Transport & MCU Link Layer

This layer abstracts physical IO and routing of messages.

### Transports Diagram

```mermaid
flowchart TD
    Serial[SerialSharedTransport]
    Socket[SocketSharedTransport]
    Mock[SharedMockTransport]

    PMT[PMTransport]
    QDT[FourQDTransport]
    THT[ThermalTransport]
    CAMT[CameraTransport]
    OPAT[OPACommandTransport]
    PST[PhaseShifterTransport]
    EDFAT[EDFACommandTransport]
    SFPT[SFPTransport]
    EPST[EPSTransport]

    Serial --> PMT
    Serial --> QDT
    Serial --> THT
    Serial --> OPAT
    Serial --> PST
    Serial --> EDFAT
    Serial --> SFPT
    Serial --> EPST

    Socket --> CAMT
    Mock --> PMT
    Mock --> QDT
    Mock --> THT
    Mock --> CAMT
```

---

## 6. Hardware Layer

Represents physical devices: Teensy, sensors, actuators, and peripherals.

### Hardware Diagram

```mermaid
flowchart LR
    TEENSY[Teensy 4.1]
    PM[Power Meters]
    QD[4QD Sensors]
    TH[Thermal Sensors]
    OPA[OPAs]
    PS[Phase Shifters]
    EDFA[EDFAs]
    SFP[SFPs]
    EPS[EPS]
    CAMERA[Camera]

    TEENSY --> PM
    TEENSY --> QD
    TEENSY --> TH
    TEENSY --> OPA
    TEENSY --> PS
    TEENSY --> EDFA
    TEENSY --> SFP
    TEENSY --> EPS
    CAMERA -. TCP .- RPI
```

---

## 7. Runtime Model (Processes & Threads)

The PAT Controller separates deterministic logic from blocking IO.

### Processes Diagram

```mermaid
flowchart TD
    PAT[pat_controller Process (Raspberry Pi)]
    CAM[Camera Process]
    GUI[GUI Process]
    LOGP[Logger Process]
    MCU[Teensy 4.1]

    GUI <--> PAT
    LOGP <--> PAT
    CAM <--> PAT
    PAT <--> MCU
```

### Internal Threads

```mermaid
flowchart TD
    Loop[Main Loop Thread]
    SerialT[Serial Transport Thread]
    SocketT[Socket Transport Thread]
    TeleT[Telemetry Thread]
    LogT[Logging Thread]
    Tctrl[Telecontrol Thread or Poll]

    Loop --> TeleT
    Loop --> LogT
    SerialT --> Loop
    SocketT --> Loop
    Tctrl --> Loop
```

---

## 8. Teensy Firmware Architecture

The Teensy executes fast micro-loops and provides structured snapshots.

### Teensy Diagram

```mermaid
flowchart TD
    Idle[T_IDLE]
    Manual[T_MANUAL]
    Track[T_TRACKING]
    FastLoop[Fast 1 kHz Loop]
    Hill[Hill-Climb Optimizer]

    Idle --> Manual
    Manual --> Track
    Track --> Manual

    Track --> FastLoop
    Track --> Hill
```

---

## 9. Operational States

### State Machine Diagram

```mermaid
stateDiagram-v2
    Idle
    Manual
    Acquisition
    Tracking
    Error

    Idle --> Manual
    Idle --> Acquisition
    Idle --> Tracking

    Manual --> Idle
    Acquisition --> Tracking
    Tracking --> Acquisition
    Tracking --> Idle
    * --> Error
```

---

## 10. Main Control Loop

### Control Loop Diagram

```mermaid
flowchart TD
    TC[Fetch Telecommands]
    PC[Process Telecommands]
    SR[Acquire Sensor Data]
    ES[Execute Active State]
    AC[Issue Actuator Commands]
    TM[Build Telemetry]
    LG[Generate Logs]
    TS[Timing Sync]

    TC --> PC --> SR --> ES --> AC --> TM --> LG --> TS --> TC
```

---

## 11. Telecontrol Architecture

```mermaid
flowchart TD
    GUI[GUI / External Controller]
    TT[Telecontrol Transport]
    TM[Telecontrol Manager]
    SM[State Manager]
    CM[Config Manager]
    AM[Actuators Manager]
    PM[Peripherals Manager]

    GUI --> TT --> TM
    TM --> SM
    TM --> CM
    TM --> AM
    TM --> PM
```

---

## 12. Telemetry & External Integration

### Telemetry Pipeline

```mermaid
flowchart TD
    Loop[Main Loop]
    TM[Telemetry Manager]
    TThread[Telemetry Thread]
    GUI[GUI]
    LOG[Logger]

    Loop --> TM --> TThread
    TThread --> GUI
    TThread --> LOG
```

---

## 13. Logging Pipeline

```mermaid
flowchart TD
    ANY[Any Module]
    Q[Logging Queue]
    LT[Logging Thread]
    DISK[Disk or External Sink]

    ANY --> Q --> LT --> DISK
```

---

## 14. Migration Path

The design is compatible with future C++ Teensy logic and richer MCU protocols.

---

## 15. Summary

This README provides:

* A full architectural description
* Layer diagrams
* Runtime/process diagrams
* Firmware state diagrams
* Control loop flow
* Telemetry/telecontrol pipelines

The system is modular, testable, and future‑proof.

