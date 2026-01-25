# Project: Driver Drowsiness Detection System

## Stakeholder Requirements

Stakeholders require a system that:
- **Ensures Driver Safety:** Provides timely warnings before cognitive fatigue results in an accident.
- **Responds to Physiological Events:** Detects shifts in EEG frequency bands within a strictly bounded time.
- **Handles Sensor Faults:** Safely manages scenarios where EEG electrodes lose contact or provide noisy data.
- **Operates Autonomously:** Functions entirely on the embedded edge without reliance on external cloud stability.
- **Supports Verification:** Allows for systematic testing via simulated data streams (UART) to prove safety logic before deployment.

---

## Functional Requirements

### FR-1 & FR-2: Timing and Performance
- **FR-1:** The system shall receive EEG data frames over UART at a minimum rate of 1 frame per 100 ms and process each frame within 50 ms of reception.
- **FR-2:** The system shall support EEG data acquisition over BLE with a minimum sustained throughput of ≥ 90% of UART frame rate when BLE mode is enabled.

### FR-3 & FR-4: Fault Detection
- **FR-3:** The system shall continue operating and updating internal state when up to 2 consecutive EEG frames are missing, delayed, or malformed.
- **FR-4:** The system shall treat EEG data as invalid and suppress safe-state declaration if ≥ 30% of channels are missing or if signal variance remains below a minimum threshold for 2 seconds.

### FR-5 & FR-6: Core Processing
- **FR-5:** The system shall compute EEG-derived metrics, including alpha-to-beta power ratio and signal variance, over sliding windows of 2 seconds updated every 500 ms.
- **FR-6:** The system shall maintain a buffer of the most recent EEG metrics to support multi-window trend analysis.

### FR-7: Alarms and Notifications
- **FR-7:** The system shall activate a driver alert within 500 ms of entering an unsafe (drowsy) state.

### FR-8 & FR-9: Power and Reset Behavior
- **FR-8:** The system shall perform a Power-On Self-Test (POST) to verify sensor connectivity and memory integrity prior to normal operation.
- **FR-9:** Upon detecting a low battery condition (<10%), the system shall prioritize Alert Notifications over BLE data transmission to conserve power for safety logic.

---

## Non-Functional Requirements

### Timing
- **NFR-T1:** The system shall guarantee a worst-case execution time (WCET) for the Alpha/Beta calculation that is less than the sampling window duration to prevent buffer overflows.
- **NFR-T2:** The system shall respond to a drowsiness event within 200ms of the window completion.
- **NFR-T3:** The system shall utilize prioritized task scheduling to ensure that the Alarm Task can preempt the Data Processing Task to meet latency deadlines.

### Reliability
- **NFR-R1:** In cases of high signal noise or uncertainty, the system must prefer triggering a false alert over failing to detect actual drowsiness (Fail-Safe/Conservative).
- **NFR-R2:** The system shall strictly follow static memory allocation for the core inference and signal processing modules to minimize runtime undefined behavior.

### Maintainability
- **NFR-M1:** The codebase shall strictly separate the communication layer (UART/BLE drivers) from the core safety logic to allow for independent testing and hardware portability.

### Recoverability
- **NFR-REC1:** The system shall resume operation in a safe state (Alert/Warning) within 500ms after a hardware reset.

### Portability
- **NFR-P1:** The core inference engine shall be written in platform-independent code to allow for compilation on both the target MCU and an x86 simulation environment.

### Usability
- **NFR-U1:** The alarm's intensity shall be determined solely by the current inferred risk level, independent of previous alert states (Stateless Alerting).