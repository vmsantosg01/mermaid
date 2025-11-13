# mermaid
```mermaid
flowchart LR

    %% ========== External Processes ==========
    subgraph GUI[External GUI Process]
        GUI_TC[Telecommands]
        GUI_TLM[Telemetry Viewer]
    end

    subgraph LOGGER[External Logger Process]
        LOGGER_SUB[Telemetry/Log Subscriber]
    end

    subgraph CAMPROC[Camera Process]
        CAM_CAP[High-Frame-Rate Capture]
        CAM_TCP[Frame Server (TCP)]
        CAM_CAP --> CAM_TCP
    end

    %% ========== Raspberry Pi Controller ==========
    subgraph PI[Raspberry Pi — PAT Controller Process]

        subgraph THREADS[Internal Threads]
            ML[Main Loop Thread]
            SERIAL[Serial Transport Thread]
            SOCK[Socket Transport Thread]
            TELE[Telemetry Thread]
            LOGTH[Logging Thread]
        end

        subgraph MANAGERS[Application Layer]
            SM[StateManager]
            SEN[SensorManager]
            ACT[ActuatorsManager]
            PER[PeripheralsManager]
            SAF[SafetyManager]
            CONF[ConfigManager]
            TC[TelecontrolManager]
            TLMGR[TelemetryManager]
        end

        subgraph TRANS[Transport Layer]
            ST[SerialSharedTransport]
            CT[SocketSharedTransport]
            MT[MockTransport]
        end
    end

    %% ========== Teensy ==========
    subgraph TEENSY[Teensy 4.1 MCU]
        T_SENS[Sensors: PM, 4QD, Thermal]
        T_ACT[Actuators: OPA, Phase Shifter]
        T_PER[Peripherals: EDFA, SFP, EPS]
        T_PROTO[MCU Protocol Engine]
    end


    %% ========== Connections ==========

    %% Telecommand Path
    GUI_TC --> TC
    TC --> ML

    %% Telemetry Path
    TELE --> GUI_TLM
    TELE --> LOGGER_SUB
    ML --> TLMGR --> TELE

    %% Camera Path
    CAM_TCP -->|TCP Frames| SOCK --> CT --> SEN

    %% Serial/MCU Path
    SERIAL --> ST --> TEENSY
    TEENSY --> ST --> SERIAL

    %% Teensy Internal Data Flow
    T_SENS --> ST
    ST --> SEN
    ACT --> ST --> T_ACT
    PER --> ST --> T_PER

    %% Main Loop Interactions
    ML --> SM
    ML --> SEN
    ML --> ACT
    ML --> PER
    ML --> SAF
    ML --> CONF

    %% Logging
    ML --> LOGTH
