Automotive ECU State Machine (PIC32CM PL10)

Overview
This repository contains the firmware for a 32-bit Automotive Control Unit based on the PIC32CM PL10. The system is designed to provide a deterministic execution environment for monitoring critical vehicle subsystems. It utilizes a preemptive RTOS and a table-driven state machine to ensure that sensor processing and safety logic meet strict timing requirements.

System Architecture
1. Table-Driven State Dispatcher
To avoid the overhead and unpredictability of nested conditional branching, the application logic is encapsulated in a function-pointer-based dispatcher.
Logic: The dispatcher calls the current_state pointer at a fixed 100Hz interval.
Safety: If an invalid state transition is detected, the pointer is automatically redirected to a System_Hard_Fault handler to prevent undefined behavior.

2. Signal Processing (Fixed-Point Engine)
As the ARM Cortex-M0+ lacks a hardware FPU, this firmware implements a Q8.8 Fixed-Point math library for scaling analog inputs.
Operation: Analog-to-Digital (ADC) raw values are shifted and scaled using integer arithmetic to maintain high-speed throughput.
Accuracy: Maintains ±0.0039 precision across the operating range of the battery temperature sensors while reducing CPU utilization by ~60% compared to standard float libraries.

Task Scheduling (FreeRTOS)
The firmware is partitioned into three distinct priority levels to ensure safety-critical tasks are never blocked:
Task Name	     | Priority	| Period	|   Responsibility
Safety_Task	   | Highest	|  5ms	  |   Over-temperature/Voltage fault detection
Control_Task	 | Medium	  |  10ms	  |  State machine execution & Sensor scaling
Tele_Task	     | Lowest	  |  100ms	|  Diagnostics output via SERCOM (UART)

Hardware Peripheral Configuration
ADC0: Configured for 12-bit resolution, 3.3V reference.
SERCOM: Configured for 115200 baud to output real-time diagnostics.
WDT: Hardware Watchdog Timer enabled with a 1.2s timeout for system recovery.
