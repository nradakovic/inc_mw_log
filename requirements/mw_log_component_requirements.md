# mw::log Component Requirements

## Log and Trace

*Log and Trace Framework consists of logging libraries and datarouter. The applications use logging libraries to write logs. datarouter implements logging daemon functionality, and can forward log messages from applications as DLT messages.*

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-001 | Valid | Use "Log & Trace" framework for logging and tracing | FFI, QM | No | mw::log, Datarouter | Derivation | All output from logging and instrumentation based logging shall go through "mw::log" framework. |

### Assumptions of Use

| ID | Rating | Type | Summary | Details |
|----|--------|------|---------|---------|
| REQ-002 | Approved | Requirement | Undocumented or private API from mw::log SHALL NOT be used. | (1) Threads:	mw::log API documented and tagged `\thread-safe` shall be thread safe. (2) Signal handler:	mw::log API SHALL NOT be called from signal handlers. (3) Interrupt handler:	mw::log API SHALL NOT be called from interrupt handlers. Note_1: The majority of the mw::log API is thread-safe. Thread-safe API can be recognized by the `\thread-safe` tag in the documentation. Note_2: LogStream classes are NOT thread-safe.|
| REQ-003 | Approved | Requirement | mw::log API SHALL NOT be used from unsupported contexts. | |
| REQ-004 | Approved | Requirement | Unbounded runtime behavior of mw::log initialization shall be mitigated or tolerated. | |
| REQ-005 | Approved | Requirement | mw::log SHALL NOT be used during the C++ static storage construction and destruction. | |
| REQ-006 | Content Review | Requirement | Console Logging shall not be used in production. | |
| REQ-007 | Approved | Requirement | Applications SHALL NOT rely on mw::log output for safety verification. | |

### mw::log Configuration

*Logging libraries use static configuration based on .json files.*

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-008 | Valid | mw::log shall be uniformly configured via an application-specific configuration file | QM | No | mw::log | Assumption | Configuration file location shall be relative to application binary. E.g.: `./<application name>/etc/logging.json`. |
| REQ-009 | Valid | Application log configuration shall provide information necessary for log message generation and filtering | QM | No | mw::log | Assumption | The provided parameter set includes, but is not limited to the following: application ID (appId), log mode for mw::log, default log level, log level assignments for individual context IDs. |
| REQ-010 | Valid | Global configuration shall be used as basis for all applications. | QM | No | mw::log | Assumption | Global configuration shall remain in the file `/etc/ecu_logging_config.json`. Application-specific configuration shall override the global defaults. |
| REQ-011 | Valid | mw::log shall fall back to a default value for a configuration item that is not available from the configuration files. | QM | No | mw::log | Assumption | mw::log shall fall back to a default value for a configuration item that is not available from the configuration files. |
| REQ-012 | Valid | mw::log shall discard configuration files with syntax errors. | QM | No | mw::log | Assumption | mw::log shall discard configuration files with syntax errors. |
| REQ-013 | Valid | mw::log shall discard configuration files that are not found. | QM | No | mw::log | Assumption | mw::log shall discard configuration files that are not found. |
| REQ-014 | Valid | mw::log shall discard invalid configuration settings. | QM | No | mw::log | Assumption | mw::log shall discard invalid configuration settings. |

### mw::log

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-015 | Valid | Avoid signal processing | ASIL B | Security relevant | mw::log | Derivation | The component shall not register any signal handler. |
| REQ-016 | Valid | File descriptors flags | ASIL B | Security relevant | mw::log | Derivation | The component shall set the `FD_CLOEXEC` (or `O_CLOEXEC`) flag on all the file descriptor it owns. Note: This requirement can be satisfied in unit-test, by checking `int flags = fcntl(fd, F_GETFD); assert((flags != -1) && (flags & FD_CLOEXEC));` for every file descriptor owned by the component. |
| REQ-017 | Valid | mw::log shall implement DLT verbose mode. | QM | No | mw::log | Assumption | Generation of messages shall be done in DLT-conform format, allowing messages to be sent by Datarouter without additional reprocessing. |
| REQ-018 | Valid | mw::log shall implement "Specification of Log and Trace for Adaptive Platform". | QM | Security relevant | mw::log | Assumption | Release of specification used shall be in agreement with the rest of Adaptive AUTOSAR stack. |
| REQ-019 | Valid | mw::log shall use logging library to send verbose messages to datarouter. | QM | No | mw::log | Assumption | Applications shall use mw::log interface to log verbose mode messages. Internally mw::log shall be using logging library. |
| REQ-020 | Valid | mw::log library shall not perform any useful activity if log level of LogStream object is insufficient for given context. | QM | No | mw::log | Assumption | LogStream object shall not generate serialized messages if the severity of the message does not allow it to be sent. |
| REQ-021 | Valid | Logging shared-memory files shall have read-only posix file permission for group and others | ASIL B | Security relevant | mw::log | Derivation | The shared-memory file created by mw::log shall be read-only for group and others. Notes: mw::log creates a shared memory file for each app. |

### System Backend

*System logger backend shall forward the logs to the native system logger.*

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-022 | Valid | Forward the logs to the native system logger | QM | No | mw::log | Assumption | The system logger backend shall forward the logs to the native system logger mechanism. Note: Under QNX, slogger2 shall be used. |
| REQ-023 | Valid | Activation of the system backend | QM | No | mw::log | Assumption | The system logger backend shall be enabled if and only if the log mode contains "kSystem". |

### Non-Functional Requirements

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-024 | Valid | Local allocation strategy | ASIL B | Security relevant | mw::log | Derivation | mw::log shall use local allocators to avoid using global heap. Global heap allocation (if any) shall be limited to initialization phase of application lifecycle. |
| REQ-025 | Valid | No endless loops | ASIL B | Security relevant | mw::log | Derivation | mw::log shall not contain unbound loops or loops with unchecked exit conditions. |
| REQ-026 | Valid | Avoid locks | ASIL B | Security relevant | mw::log | Derivation | mw::log shall be free of time-unbound locks and shall implement strategies to limit wait time. Time limit shall be defined based on requirements from functions on individual ECU. Mutual exclusion mechanisms shall include priority inversion protection. |
| REQ-027 | Valid | Cross-locking | ASIL B | Security relevant | mw::log | Derivation | Cross-application and cross-thread dependencies shall be avoided. A thread logging in a loop shall not block other thread's execution. |
| REQ-028 | Valid | Index and size checking | ASIL B | Security relevant | mw::log | Derivation | Indices in data structures and sizes of data accessed or copied shall be checked for plausibility to avoid high runtime utilization through long iterations or data corruption. |
| REQ-029 | Valid | Memory bound checking | ASIL B | Security relevant | mw::log | Derivation | Memory boundaries shall be explicitly checked when writing to shared memory. |

### Datarouter Services

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-030 | Valid | Datarouter shall implement DLT server component. | QM | No | datarouter | Assumption | Datarouter provides the functionality to send some of the messages using DLT protocol over UDP multicast transport. |
| REQ-031 | Valid | DLT server shall support multiple log channels as specified in AUTOSAR Diagnostic, Log and Trace Protocol Specification | QM | No | datarouter | Assumption | The following parameters shall be configurable statically via a deployment-specific configuration file (`<datarouter binary location>/./etc/log-channels.json`): Number and names of channels, Network configuration for channel output, Log level threshold per channel, Log channel assignments per application/context. |
| REQ-032 | Valid | DLT server shall support filtering of messages within each channel. | QM | No | datarouter | Derivation | The filter shall be parametrized with application ID, context ID, log level threshold. Initial filter set shall be part of channel configuration file, located in the following path: `<datarouter binary location>/../etc/log-channels.json`. |
| REQ-033 | Valid | DLT server shall provide support for verbose DLT messages | QM | No | datarouter | Assumption | Messages sent from mw::log library shall be sent via DLT using verbose mode. |

### DLT Quotas

| ID | Status | Summary | ASIL | Security | Satisfied by | Derivation | Description |
|----|--------|---------|------|----------|-------------|------------|-------------|
| REQ-034 | Valid | DLT Bandwidth Quota configuration | QM | No | datarouter | Derivation | The component shall have the DLT bandwidth quota configuration in the file located at `./etc/log-channels.json` relative to the application specific location `/opt/`. |
| REQ-035 | New | DLT Quota Bandwidth Reduction | QM | -- | -- | Derivation | It shall be possible to configure the action to drop DLT messages when a quota is exceeded. NOTE: The configuration shall be provided as an attribute as part of the DLT quota configuration file (REQ-034). |
