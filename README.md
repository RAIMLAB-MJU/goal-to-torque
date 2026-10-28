# Goal-to-Torque

**An Open-Source Reference Architecture for Real-Time Robot Control**

*From asynchronous high-level decisions to deterministic joint-torque execution.*

Goal-to-Torque is an independent, research-oriented software project designed to bridge high-level robotic applications and real-time, model-based joint-torque control using open-source technologies.

The architecture addresses a fundamental challenge in intelligent robotics: perception, planning, and learning algorithms operate with variable computational demands and asynchronous execution, whereas low-level robot controllers require predictable periodic operation.

Rather than coupling high-level applications directly to actuator control, Goal-to-Torque introduces explicit communication and control interfaces that preserve temporal separation, data consistency, and command validity throughout the execution pipeline.

The project is designed around four architectural principles:

1. **Temporal decoupling:** Asynchronous ROS 2 applications exchange information with real-time tasks without introducing application-level blocking into the periodic control path.
2. **Thread-safe communication:** Atomic, single-writer data slots provide consistent snapshots with bounded read retries and freshness validation.
3. **Time-explicit torque control:** Fixed-priority scheduling separates fieldbus transmission, reference generation, and computed-torque control.
4. **Cross-layer verification:** Operating-system timing, fieldbus communication, robot dynamics, reference constraints, and physical motion are evaluated as connected parts of one architecture.

The intended reference platform combines PREEMPT_RT Linux, the IgH EtherCAT Master, RBDL, Ruckig, ROS 2, and MuJoCo. Initial development targets the Neuromeka Indy7 while keeping robot-specific hardware interfaces separate from the reusable controller core.

## Real-Time Execution

Three periodic tasks operate on a common 1-ms release grid:

| Task | Priority | Responsibility |
|---|---|---|
| EtherCAT | 97 | Acquire drive feedback and transmit the previously computed torque |
| Trajectory | 96 | Process goals, solve inverse kinematics, and generate constrained references |
| Dynamics | 95 | Compute the next joint-torque command using the robot's dynamic model |

The send-before-compute schedule prioritizes host-side transmission timing while introducing an explicitly modeled additional cycle of command age.

## Communication Interfaces

The controller uses four logical data-exchange slots:

| Slot | Publisher | Consumer |
|---|---|---|
| Goal | Application bridge | Trajectory task |
| State | EtherCAT task | Trajectory and Dynamics tasks |
| Reference | Trajectory task | Dynamics task |
| Torque | Dynamics task | EtherCAT task |

Each interface is intended to provide consistent data publication, bounded reading, source-period identification, and defined behavior when information becomes invalid or stale.

These interfaces prevent asynchronous application execution from directly blocking the periodic torque controller. They do not eliminate computational delays within the periodic tasks themselves.

## Research and Reproducibility

The project aims to provide a reusable implementation of the architecture presented in:

*From High-Level Goals to Joint Torques: A Reference Software Architecture for Real-Time Robot Control Using Open-Source Components.*

J. H. Yoon, K. W. Kim, J. E. Kim, and R. Delgado.

*IEEE Access manuscript — publication details pending.*

A future verified release will include source-code revisions, configuration files, simulation examples, synchronization tests, timing measurements, and experimental data necessary to reproduce its reported behavior.

**Development status:** Independent implementation in progress. The performance results reported in the research manuscript are not automatically attributable to this new codebase until reproduced and verified.

**Safety notice:** Research software only. Direct joint-torque operation requires validated hardware configurations, independent protective equipment, and appropriate safety procedures. The architecture is not a safety-certified robot controller.
