# CrisisSync – Assignment 1: Requirements, Traceability and Acceptance Tests

**Group number:** 8

| Full Name | ID Number | College Email |
|---|---|---|
| Maor Dalal | 324221233 | Maor.Dalal@e.braude.ac.il |
| Neta Baron | 213554512 | Neta.Baron@e.braude.ac.il |
| Segev Morag | 212981906 | Segev.Morag@e.braude.ac.il |
| Gal Florenthal | 212589469 | Gal.Florenthal@e.braude.ac.il |


## Scope

This document defines the requirements and acceptance tests for two operations:

1. Reserving a resource package for a task, including reporting the task's completion.
2. Generating the SLA compliance report.

## Functional Requirements (FR)

| ID | Requirement |
|---|---|
| FR-01 | The system shall reserve all resources of a task's package in a single transaction, so that either every resource is reserved or none is. |
| FR-02 | The system shall reject a reservation request for a task that is not in the Ready state. |
| FR-03 | The system shall reject a resource package that does not include exactly one Field Commander. |
| FR-04 | During automatic allocation, if a selected resource is reserved by a concurrent allocation, the system shall select the next best available resource, up to 2 alternative attempts, without a new request from the Dispatcher. |
| FR-05 | When the assigned Field Commander reports a task as complete, the system shall move the task to the Completed state and release all of its resources. |
| FR-06 | When a task is completed, the system shall move each dependent task that has no open prerequisites to the Ready state. |
| FR-07 | The system shall calculate, for each incident type, the average time from the opening of an incident until the closure of its last task, for a selected month. |
| FR-08 | The system shall generate the SLA compliance report automatically at the end of each calendar month. |
| FR-09 | The system shall allow the System Administrator to generate the SLA compliance report for a selected past month. |

## Non-Functional Requirements (NFR)

| ID | Category | Requirement |
|---|---|---|
| NFR-01 | Performance | The system shall display the result of a reservation request to the Dispatcher within 2 seconds, for a package of up to 10 resources, with up to 20 Dispatchers working concurrently. |
| NFR-02 | Performance | The system shall push every task or resource status change to all connected stations within 1 second. |
| NFR-03 | Performance | The system shall generate the SLA compliance report for one month within 30 seconds, for up to 10,000 incidents. |
| NFR-04 | Reliability | When 100 reservation requests for the same resource are submitted simultaneously, exactly 1 shall succeed and 0 double reservations shall occur. |
| NFR-05 | Reliability | If the server stops during a reservation transaction, 0 resources of that package shall remain reserved after the server restarts. |
| NFR-06 | Security | The system shall store user passwords only as bcrypt hashes with a cost factor of at least 12. |
| NFR-07 | Usability | The Field Commander shall be able to report a task as complete in at most 3 clicks from the incident status screen. |

## Requirements Traceability Matrix (RTM)

| Req ID | Type | Summary | Mapped Use Case | Test ID |
|---|---|---|---|---|
| FR-01 | FR | All-or-nothing package reservation | Reserve Resource Package | TEST-01 |
| FR-02 | FR | Reservation only for a Ready task | Reserve Resource Package | TEST-03 |
| FR-03 | FR | Exactly one Field Commander per package | Reserve Resource Package | TEST-04 |
| FR-04 | FR | Automatic fallback to the next best resource | Allocate Resources Automatically | TEST-05 |
| FR-05 | FR | Completion sets Completed and releases resources | Report Task Completion | TEST-06 |
| FR-06 | FR | Dependent tasks move to Ready | Report Task Completion | TEST-06 |
| FR-07 | FR | Average time per incident type | Generate SLA Compliance Report | TEST-08 |
| FR-08 | FR | Automatic report at the end of each month | Generate Monthly Reports | TEST-08 |
| FR-09 | FR | On-demand report for a past month | Generate Report for Selected Month | TEST-09 |
| NFR-01 | NFR | Reservation result within 2 seconds | Reserve Resource Package | TEST-01 |
| NFR-02 | NFR | Status pushed within 1 second | View Live Incident Status | TEST-06 |
| NFR-03 | NFR | Monthly report within 30 seconds | Generate SLA Compliance Report | TEST-08 |
| NFR-04 | NFR | No double reservations under concurrency | Reserve Resource Package | TEST-02 |
| NFR-05 | NFR | Full rollback after a server failure | Reserve Resource Package | TEST-07 |
| NFR-06 | NFR | Passwords stored as bcrypt hashes | Log In | TEST-10 |
| NFR-07 | NFR | Task completion in at most 3 clicks | Report Task Completion | TEST-06 |

## Acceptance Tests (BDD)

### Acceptance Test [TEST-01]: Successful Resource Package Reservation (Happy Path)
* **Given:** A task is in the Ready state and its package of 4 resources, including one Field Commander, is all available.
* **When:** The Dispatcher submits the reservation for the package.
* **Then:** The system must reserve all 4 resources for the task and move the task to the Active state.
* **And:** The Dispatcher must see the confirmation within 2 seconds.

### Acceptance Test [TEST-02]: Simultaneous Reservation of the Same Resource (Exception)
* **Given:** A bulldozer is available, and 100 reservation requests that include it are prepared for different tasks.
* **When:** All 100 requests are submitted at the same moment.
* **Then:** Exactly 1 request must succeed, and the other 99 must be rolled back with no resource reserved.
* **And:** Each failed Dispatcher must receive a clear failure message, and the bulldozer must not be reserved twice.

### Acceptance Test [TEST-03]: Reservation for a Task That Is Not Ready (Exception)
* **Given:** A task is in the Blocked state because one of its prerequisite tasks is still open.
* **When:** The Dispatcher submits a reservation for the task's package.
* **Then:** The system must reject the reservation and display the task's current state.
* **And:** No resource may be reserved.

### Acceptance Test [TEST-04]: Package Without Exactly One Field Commander (Exception)
* **Given:** A task is in the Ready state and its package does not include a Field Commander.
* **When:** The Dispatcher submits the reservation for the package.
* **Then:** The system must reject the reservation and notify the Dispatcher.
* **And:** No resource may be reserved.

### Acceptance Test [TEST-05]: Automatic Allocation Falls Back to an Alternative (Exception)
* **Given:** The Dispatcher requests automatic allocation by geographic proximity, and the closest ambulance is reserved by a concurrent allocation.
* **When:** The system tries to reserve the closest ambulance and fails.
* **Then:** The system must automatically reserve the next closest available ambulance, within at most 2 alternative attempts.
* **And:** The Dispatcher must not be asked to repeat the request.

### Acceptance Test [TEST-06]: Field Commander Reports Task Completion (Happy Path)
* **Given:** A task is Active, its reserved resources are a bulldozer and a medic, and a dependent task waits only for this task.
* **When:** The assigned Field Commander reports the task as complete, using at most 3 clicks from the incident status screen.
* **Then:** The task must move to the Completed state, and the bulldozer and the medic must become available.
* **And:** The dependent task must move to the Ready state, and all connected stations must show the change within 1 second.

### Acceptance Test [TEST-07]: Server Failure During a Reservation (Exception and Recovery)
* **Given:** A reservation transaction for a package of 4 resources has started.
* **When:** The server stops before the transaction is committed, and is then restarted.
* **Then:** 0 resources of that package may be reserved after the restart.
* **And:** The task must remain in the Ready state.

### Acceptance Test [TEST-08]: Automatic Monthly SLA Report (Happy Path)
* **Given:** The month has 10,000 incidents of different types.
* **When:** The end of the month arrives.
* **Then:** The system must generate the SLA compliance report automatically, within 30 seconds.
* **And:** The report must show, for each incident type, the average time from opening until the last task was closed.

### Acceptance Test [TEST-09]: SLA Report for a Selected Past Month (Happy Path)
* **Given:** The System Administrator is logged in, and incidents exist for a past month.
* **When:** The System Administrator requests the SLA compliance report for that month.
* **Then:** The system must generate the report for the selected month.
* **And:** The report must be available for the System Administrator to view.

### Acceptance Test [TEST-10]: Password Storage (Security)
* **Given:** A new user account is created with a password.
* **When:** The user's record is read directly from the database.
* **Then:** The password must appear only as a bcrypt hash with a cost factor of at least 12.
* **And:** The plain-text password must not appear anywhere in the database.
