PAT Controller
A Modular Soft–Real-Time Supervisory System for Optical Pointing Experiments
Table of Contents
Project Overview
High-Level Description
Core Goals and Purpose
Architecture Summary
Architectural Style
Technologies Used
Hardware Elements
Requirements
Functional Requirements
Non-Functional Requirements
Constraints & Assumptions
System Components
Application Layer
Device & Domain Layer
Transport & MCU Link Layer
Hardware Layer
Diagrams
Technical Decisions & Rationale
Development Guidelines
Code Structure
Naming Conventions
Branching Strategy
Testing Approach
Environment Setup
Project Overview
The PAT Controller is a modular, soft–real-time supervision system running primarily on a Raspberry Pi. It orchestrates sensors, actuators, and optical peripherals for an optical pointing experiment. The controller operates as a deterministic loop driven by state machines, telecommands, and telemetry pipelines—all while maintaining the ability to run on real hardware or in a fully simulated environment.
The system supervises a Teensy 4.1 microcontroller responsible for fast actuation and sensor acquisition. Additional devices—such as cameras—may run in separate processes and communicate over TCP.
High-Level Description
The controller executes a periodic supervision loop that processes telecommands, reads sensors, applies state logic, commands actuators, and publishes telemetry. All hardware is abstracted behind clean interfaces, ensuring that components can be replaced, mocked, or extended without architectural changes.
The system supports multiple operational states used in optical experiments:
Manual control
Acquisition routines
Closed-loop tracking
Scanning patterns
Experiment-specific behavior
The design supports future migration toward tighter real-time constraints and partial C++ reimplementation.
Core Goals and Purpose
Provide a deterministic, soft–real-time control loop for an optical pointing system.
Cleanly separate application logic, device models, transports, and hardware.
Support telecommands for control, configuration, and safety.
Publish structured telemetry to GUI and logging processes.
Enable full simulation without hardware.
Remain portable toward future C++ MCU-centric designs.
Architecture Summary
The system follows a four-layer architecture:
Application Layer
Device & Domain Layer
Transport & MCU Link Layer
Hardware Layer
Interprocess communication is used for cameras, GUI, and logging components.
Architectural Style
Modular, layered design
Interface-driven device abstraction
Soft–real-time event-driven loop
Multi-threading with strict isolation of blocking IO
Microservice-like external processes (GUI, Logger)
Protocol-based MCU link
Technologies Used
Python 3.x — Primary implementation language
ZeroMQ or TCP sockets — Telemetry and telecommand transport
UART (Serial) — Raspberry Pi to Teensy communication
Raspberry Pi OS / Linux — Deployment platform
Teensy 4.1 (C++ firmware) — Low-level IO and fast control loops
Optional: Protobuf / JSON for structured telemetry
Hardware Elements
Raspberry Pi
Teensy 4.1 (MCU backend)
Optical hardware
OPA arrays
Phase Shifters
EDFAs
SFP modules
EPS units
Sensors
Power Meters
4QD sensors
Thermal sensors
Cameras (separate process)
Requirements
Functional Requirements
Execute deterministic supervision loop.
Acquire sensor data (PM, 4QD, Thermal, Cameras).
Control actuators and peripherals (OPA, Phase Shifters, EDFA, SFP, EPS).
Support multiple operational states (Idle, Manual, Acquisition, Tracking).
Process telecommands without blocking the control loop.
Publish telemetry asynchronously.
Log diagnostic events.
Support mock transports and simulated sensors.
Non-Functional Requirements
Soft real-time loop with bounded jitter.
High modularity and hot-swappable components.
Thread-safe isolation of blocking IO.
Portability toward C++ implementation.
Robustness under missing devices or delayed transports.
Constraints & Assumptions
No hard real-time guarantees.
Teensy firmware may evolve into more autonomous control.
System must run without hardware using mocks.
System Components
Application Layer
This layer coordinates control logic, states, telecommands, sensors, actuators, and telemetry.
Controller
Central orchestrator of the system.
class Controller:
    """
    @brief Main orchestrator of the PAT control system.
    @details Executes the loop, handles telecommands, manages states,
    aggregates sensors, commands actuators, and publishes telemetry.
    """

    def run(self):
        """@brief Starts the deterministic control loop."""
        pass
State Manager and States
Implements Idle, Manual, Acquisition, Tracking, and Error states.
class State:
    """
    @brief Base class for all controller states.
    @details Each state implements its per-loop behavior.
    """

    def execute(self, ctx):
        """@brief Execute one iteration of the state."""
        raise NotImplementedError
Sensors Manager
Aggregates all sensors and returns typed snapshots.
Actuators Manager
Controls OPAs and Phase Shifters.
Peripherals Manager
Controls EDFA, SFP, EPS.
Telemetry Manager
Builds telemetry snapshots.
Telecontrol Manager
Routes telecommands and performs validation.
Config Manager
Stores thresholds, gains, scan ranges, etc.
Device & Domain Layer
Defines high-level sensor/actuator objects with stable APIs.
Example device interface:
class PowerMeterSensor:
    """
    @brief Logical model of a power meter.
    @details Provides structured readings independent of the transport type.
    """

    def read(self):
        """@brief Returns latest power reading."""
        pass
Actuators use semantic APIs:
class OPADevice:
    """
    @brief Logical model of an OPA actuator.
    @details Exposes semantic steering commands.
    """

    def steer(self, x, y):
        """@brief Commands OPA to steer to (x, y)."""
        pass
All device drivers rely on ChannelTransports, not hardware.
Transport & MCU Link Layer
Handles communication with Teensy and external processes.
Shared Transports
SerialSharedTransport (UART → Teensy)
SocketSharedTransport (Camera frames)
SharedMockTransport (simulation)
ChannelTransports
Each device has its own logical channel:
PMTransport
FourQDTransport
ThermalTransport
CameraTransport
OPACommandTransport
PhaseShifterTransport
EDFACommandTransport
SFPTransport
EPSTransport
MCU Link
Defines high-level commands and snapshots.
class MCUCommand:
    """
    @brief High-level command sent to the Teensy.
    @param opcode Type of command.
    @param payload Structured command data.
    """
Hardware Layer
Teensy 4.1 running fast tracking and IO
All optical sensors and actuators
Camera process
UART and TCP transports
Diagrams
High-Level System Architecture
flowchart LR

GUI[GUI Process] <-- Telecommands --> PAT[PAT Controller]
Logger[Logger Process] <-- Telemetry --> PAT

PAT -->|UART| Teensy
PAT -->|TCP| CameraProc[Camera Process]
CameraProc --> CameraHW[Camera Hardware]

Teensy --> OPAs
Teensy --> PhaseShifters
Teensy --> PMs
Teensy --> FourQD
Teensy --> Thermals
Teensy --> EDFA
Teensy --> SFP
Teensy --> EPS
PAT Controller Internal Threads
flowchart TB

MainLoop[Main Loop Thread]

SerialT[Serial Shared Transport]
SocketT[Socket Shared Transport]
TelemetT[Telemetry Thread]
LogT[Logging Thread]
TelecmdT[Telecommand Thread]

MainLoop --> SerialT
MainLoop --> SocketT
MainLoop --> TelemetT
MainLoop --> LogT
MainLoop --> TelecmdT
Control Loop Sequence Diagram
sequenceDiagram
    participant TC as Telecontrol Manager
    participant SM as State Manager
    participant SEN as Sensors Manager
    participant ACT as Actuators Manager
    participant TEL as Telemetry Manager
    participant LOG as Logging Thread

    loop Each Loop
        TC->>TC: Fetch telecommands
        TC->>SM: Process transitions/commands
        SEN->>SEN: Acquire sensor data
        SM->>ACT: Run state logic and produce actuator commands
        ACT->>ACT: Send commands to transports
        SM->>TEL: Build telemetry snapshot
        TEL->>LOG: Queue structured logs
    end
Technical Decisions & Rationale
Layered Architecture
Decouples application logic from hardware, ensuring simulation capability and future firmware evolution.
Soft Real-Time Main Loop
Provides deterministic behavior without blocking on IO.
Dedicated IO Threads
Prevents telemetry, logging, serial, and socket operations from impacting timing.
Device Abstractions
Stable interfaces allow migration to C++ or alternate transports transparently.
Teensy Fast Loop
Delegates high-frequency tracking where Python cannot meet timing.
Mock Transports
Enable testing and CI without any hardware dependency.
Development Guidelines
Code Structure
pat_controller/
│
├── app/                # Application Layer
│   ├── controller.py
│   ├── state_manager.py
│   ├── states/
│   ├── telemetry_manager.py
│   ├── telecontrol_manager.py
│   ├── config_manager.py
│   └── safety_manager.py
│
├── devices/           # Device Layer
│   ├── sensors/
│   ├── actuators/
│   └── peripherals/
│
├── transports/        # Shared & Channel Transports
│   ├── serial/
│   ├── socket/
│   └── mock/
│
├── mcu_link/          # MCU protocol definitions
│
├── utils/             # Logging, timing, helpers
│
└── tests/             # Unit & simulation tests
Naming Conventions
Classes: PascalCase
Methods/variables: snake_case
Modules: short and descriptive
Telemetry keys: lower_snake_case
Branching Strategy
Recommended Git workflow:
main → stable releases
dev → active development
Feature branches: feature/<name>
Bugfix branches: bugfix/<name>
Use PRs with code review
Testing Approach
Mock transports for full pipeline testing
Replay tests using logged sensor data
Unit tests for device logic and state transitions
Integration tests for timing and telemetry
CI pipeline executing simulation scenarios
Environment Setup
Requirements
Python 3.x
ZeroMQ or equivalent transport library
Access to Raspberry Pi UART (for hardware mode)
Installation
pip install -r requirements.txt
Running on Raspberry Pi
python -m pat_controller
Running in Simulation
python -m pat_controller --mock
Final Notes
This README provides a full technical specification and onboarding reference for developers. The system architecture is modular, testable, and designed for long-term evolution toward MCU-centric control without redesigning upper layers.
If you need additional documentation (API docs, sequence charts, or firmware README), just tell me.
