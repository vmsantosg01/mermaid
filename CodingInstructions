# Coding Styler — Unified Development & Documentation Rules

This document defines the **mandatory coding style**, documentation practices, and structural conventions that must be followed across the entire PAT Controller codebase.  
Its purpose is to ensure that every contributor — including automated coding agents — produces clean, consistent, maintainable, and well-documented code.

These rules apply to **all Python, C++, MCU firmware, scripts, utilities, and tests** unless otherwise stated.

---

# 1. General Principles

1. **Clarity over cleverness**  
   Code must be readable first, optimized second.

2. **Predictability**  
   Follow these conventions strictly so that every module looks and feels consistent.

3. **Explicitness**  
   Avoid hidden behaviors, implicit state, or magic values.

4. **Strict modularity**  
   All layers (application, devices, transports, MCU link) must remain decoupled.

5. **Documentation always required**  
   Every function, class, module, and file must include Doxygen-compatible doc comments.

---

# 2. File & Module Structure

### 2.1 Python File Structure (strict order)
Each Python file must follow this exact structure:

```text
# 1. Standard library imports
# 2. Third-party imports
# 3. Local project imports
# 4. Module-level documentation block (Doxygen)
# 5. Constants
# 6. Enums / NamedTuples / dataclasses
# 7. Classes (public -> private)
# 8. Functions (public -> private)
# 9. Main guard (if needed)
````

### 2.2 C++ File Structure

```text
// .hpp
// 1. Include guards or #pragma once
// 2. Includes
// 3. Namespaces
// 4. Forward declarations
// 5. Class/struct declarations with Doxygen
// 6. Inline functions (if any)

// .cpp
// 1. Includes (hpp first)
// 2. Namespaces
// 3. Function definitions
```

---

# 3. Naming Conventions

### 3.1 Python

* **Classes**: `PascalCase`
* **Functions**: `snake_case`
* **Variables**: `snake_case`
* **Constants**: `UPPER_CASE`
* **Private members**: `_leading_underscore`
* **Module names**: `snake_case`

### 3.2 C++ (Teensy Firmware)

* **Classes & structs**: `PascalCase`
* **Methods**: `camelCase`
* **Variables**: `camelCase`
* **Private attributes**: `_camelCase`
* **Constants**: `UPPER_CASE`
* **Namespaces**: `lowercase`

---

# 4. Doxygen Documentation Rules

All code must be fully documented using **Doxygen-compatible comments** and written **in English**.

### 4.1 File Documentation Template

```cpp
/**
 * @file camera_transport.py
 * @brief Implements the logical transport wrapper for the camera device.
 * @details
 *  - Handles frame retrieval from SharedTransport
 *  - Provides non-blocking access to camera readings
 */
```

### 4.2 Class Documentation Template

```cpp
/**
 * @class CameraTransport
 * @brief High-level logical transport for the camera process.
 * @details
 *  Provides:
 *  - receive() method for asynchronous frame retrieval
 *  - send_command() for future extensions
 */
```

### 4.3 Function Documentation Template (Python & C++)

```cpp
/**
 * @brief Short summary of what the function does.
 * @param param_name Description of the parameter.
 * @return Description of the returned value.
 * @throws (C++) Optional list of thrown exceptions.
 * @remarks Additional relevant information.
 */
```

### 4.4 Required Notes

Every public method **must** specify:

* What it does
* What it expects
* How it behaves in edge and error cases
* The units and coordinate system for numerical inputs (critical for OPAs)

---

# 5. Function Style Rules

### 5.1 Python Functions

* Must type annotate all parameters and return values.
* Must never return raw dicts for structured data — use:

  * `dataclasses`
  * `TypedDict`
  * or small custom classes

Example:

```python
def read(self) -> FourQDReading:
    """
    @brief Reads the latest 4QD snapshot.
    @return A structured FourQDReading object.
    """
```

### 5.2 C++ Functions (Teensy)

* Avoid dynamic memory
* Prefer references over pointers
* Never block in interrupt-level code
* All loops must be bounded (no unbounded while)

---

# 6. Error Handling

### 6.1 Python

* Never swallow exceptions silently.
* Use custom exception types per layer.
* Wrap hardware/transport reads in clearly scoped try/except blocks.

### 6.2 C++

* No exceptions inside real-time Teensy code.
* Use error codes or boolean return values.
* All MCU-host link decoding must validate buffer sizes.

---

# 7. Logging Rules

### 7.1 Python Logging

Use the project’s structured logging system:

```python
log_event("WARNING", "OPA deviation exceeded", {"opa_id": opa_id, "position": pos})
```

Never print to stdout.

### 7.2 Firmware Logging

Minimal and rate-limited.
Use lightweight diagnostic flags or counters.

---

# 8. Telemetry & Snapshot Formatting

### 8.1 Structured fields only

Telemetry must always be:

* structured
* validated
* versioned
* machine-parsable

Never send raw, ambiguous strings.

### 8.2 Naming

Keys must obey:

```
lowerCamelCase for all telemetry JSON fields
```

---

# 9. Command Formatting Rules

### 9.1 Python → Teensy Commands

All commands must follow:

* A single encoding method
* One function per semantic action
* No inline string building; use encoder utilities

Example:

```python
def set_opa_position(opa_id: int, x: float, y: float) -> None:
    """
    @brief Sends command to steer OPA to (x, y).
    """
```

### 9.2 Teensy → Python Snapshots

Must obey:

* exact ordering
* exact field naming
* exact schema version

---

# 10. Concurrency Rules

### 10.1 Python Main Loop

The main loop must:

* NEVER block
* NEVER wait on IO
* NEVER perform disk operations

All blocking work must be routed to background threads.

### 10.2 Shared Transports

* Must be thread-safe
* Must use queues for incoming data
* Must serialize outgoing commands

### 10.3 Locks

* Use locks only where absolutely required
* Keep lock scope extremely small

---

# 11. State Machine Rules

### 11.1 All states must:

* Be implemented as classes
* Derive from a common interface
* Have `enter()`, `execute()`, `exit()`
* Never perform blocking IO
* Never read raw transports

### 11.2 Transitions

* Must go exclusively through `StateManager`
* Must be logged
* Must be validated

---

# 12. Testing Rules

### 12.1 Unit Tests

* No real hardware
* Use `SharedMockTransport`
* Use deterministic inputs
* Validate behavior, not timing

### 12.2 Hardware-in-the-loop

Optional, but must follow:

* reproducible test cases
* snapshot comparison
* fault injection sequences

---

# 13. Folder & Path Conventions

```
project_root/
    application/
    devices/
    transports/
    mcu_link/
    states/
    managers/
    telemetry/
    telecontrol/
    logging/
    utils/
    tests/
    tools/
```

All files must follow this structure.

---

# 14. Commit Message Style

Must follow the conventional format:

```
<type>: <summary>

<body>
```

Where type is one of:

* feat
* fix
* refactor
* docs
* test
* build
* chore

---

# 15. Summary

This coding styler defines:

* Naming rules
* File structure
* Doxygen documentation requirements
* Telemetry and command formatting
* Threading and timing constraints
* State machine structure
* Transport and device abstraction rules
* Logging and testing standards

All contributors and coding agents must follow these guidelines to guarantee a uniform, maintainable, and safe codebase for the PAT Controller system.

---
