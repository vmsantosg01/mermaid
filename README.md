# README

```md
Project Objective
The PAT controller project aims to implement a modular, soft–real-time control system running on a Raspberry Pi, responsible for orchestrating sensors, actuators, and peripheral devices involved in an optical pointing experiment. The controller must execute deterministic state-driven logic cycles, manage telecommands, and maintain coherent telemetry, while ensuring that the same core software can operate seamlessly on real hardware or in fully mocked environments for development and validation.

## Architecture Overview (Mermaid Diagram)

```mermaid
graph TD
    A[Application Layer] --> B[Device & Domain Layer]
    B --> C[Transport & MCU Link Layer]
    C --> D[Hardware Layer]
```

## Runtime Processes (Mermaid Diagram)

```mermaid
graph LR
    P1[Raspberry Pi - PAT Controller]
    P2[Camera Process]
    P3[GUI Process]
    P4[Logger Process]
    P5[Teensy 4.1]

    P1 --- P2
    P1 --- P3
    P1 --- P4
    P1 --- P5
```

## Control Loop Pipeline (Mermaid Diagram)

```mermaid
sequenceDiagram
    participant T as Telecontrol
    participant C as Controller
    participant S as SensorsManager
    participant ST as StateManager
    participant A as Actuators/Peripherals
    participant TM as TelemetryManager

    T->>C: Fetch Telecommands
    C->>C: Process Telecommands
    C->>S: Acquire Sensor Data
    S-->>C: Sensor Snapshot
    C->>ST: Execute Active State
    ST->>A: Issue Commands
    C->>TM: Build Telemetry Snapshot
    TM-->>C: Telemetry Enqueued
```

