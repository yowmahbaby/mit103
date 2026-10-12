# MIT 103 - Asynchronous Activity: Data Flow Diagrams

**Course:** MIT 103 - Advanced Systems Analysis, Design, and Implementation  
**Student:** Roffe R. Fajardo  
**Program:** Master of Information Technology (MIT 1A)  
**Project:** **KHub Cross-Section Engagement Analytics Module for PSHS-BRC**  
**Notation:** **Yourdon / DeMarco** throughout the six diagrams  

## 1. System description and diagram convention

The proposed module consolidates information on learner activity across KHub course-sections that a teacher is authorized to view. A teacher can select sections and a reporting period, obtain engagement summaries, see learner-level status and identify records that may require follow-up. KHub continues to own the course, enrollment, activity, completion and submission data. The module is **read-only with respect to these source records**. The ICT/LMS Administrator manages configuration and receives limited module-status and audit information.

The diagrams use the following **Yourdon notation** consistently:

| Symbol | Meaning | Naming rule |
|---|---|---|
| Oval process bubble | Transforms incoming data into outgoing data | Process number and **verb + noun**, e.g., `2.0 Retrieve KHub Records` |
| Rectangular external entity | Actor or other system outside the analytics module | Noun or system name, e.g., `Teacher` |
| Two parallel horizontal lines | Logical data store maintained by the analytics module | `D1`, `D2`, plus noun name |
| Arrow with text | Direction and content of a data flow | **Noun phrase**, e.g., `Enrollment Data` |
| Dashed rectangle in Level-1 diagrams | An *interface reference* to another Level-0 process, **not an additional external actor** | `Interface: 2.0 Retrieve KHub Records` |


## 2. Context diagram - Process 0

![Yourdon context-level DFD](diagrams/01_context.svg)

**Process 0:** KHub Cross-Section Engagement Analytics Module. Exactly three external entities interact with the one whole-system process; no data stores or internal subsystems are drawn at this level.

### Context-level external flow inventory

| External entity | Data flowing **into** Process 0 | Data flowing **out of** Process 0 |
|---|---|---|
| **Teacher** | Analytics Request; Course-Section Selection; Date Range and Activity Filters | Cross-Section Engagement Summary; Learner Engagement Status; Follow-Up List |
| **KHub / Moodle Core System** | Course-Section Data; Enrollment Data; Activity Log Data; Completion / Submission Status | Authorized Analytics Query Parameters |
| **ICT / LMS Administrator** | Role Permissions; Module Configuration | Access / Audit Information; Module Status |

**Count:** 15 separately named boundary flows: 3 + 3 for Teacher, 4 + 1 for KHub/Moodle, and 2 + 2 for the administrator. These names and directions are carried into the Level-0 diagram unchanged.

## 3. Level-0 DFD - Decomposition of Process 0

![Yourdon Level-0 DFD](diagrams/02_level0.svg)

The whole system is decomposed into **four** major processes.

| Process | Name | Purpose | Existing requirements supported |
|---|---|---|---|
| **1.0** | **Authorize Analytics Access** | Accept teacher scope and filters; apply configured role and section permissions; issue an access decision | FR-01 to FR-05; FR-14 |
| **2.0** | **Retrieve KHub Records** | Send authorized, read-only queries to KHub; obtain section, enrollment, activity, completion, and submission information | FR-06 to FR-08; FR-13 |
| **3.0** | **Compute Engagement Measures** | Produce section-level measures, learner-level information, and evidence-based possible follow-up cases | FR-09 to FR-12 |
| **4.0** | **Present Reports and Status** | Present the three teacher outputs and restricted administrative audit/status information | FR-09 to FR-12; FR-14 |

### Data stores

| Store | Name | Contents and purpose | Scope protection |
|---|---|---|---|
| **D1** | Module Rules | Approved module settings and role-access rules | Administrative configuration only; not permission to override KHub |
| **D2** | Audit Log | Access, retrieval, and report-access events; reporting status metadata | Avoid storing raw learner activity or grades in audit entries |

KHub's existing source database is **not** drawn as an internal data store. It belongs to the external **KHub / Moodle Core System**. The proposed analytics module does not create a separate persistent student database in this logical model.

### Main internal flow summary

| Source | Destination | Data flow(s) |
|---|---|---|
| 1.0 | 2.0 | Authorized Analytics Scope; Effective Analytics Criteria |
| 1.0 | 4.0 | Access Decision |
| 1.0 | D1 | Approved Module Rules |
| D1 | 1.0 | Active Module Rules |
| 1.0 | D2 | Access Audit Entry |
| 2.0 | 3.0 | Retrieved Engagement Records; Effective Analytics Criteria |
| 2.0 | 4.0 | Retrieval Status |
| 2.0 | D2 | Retrieval Audit Entry |
| 3.0 | 4.0 | Cross-Section Metrics; Learner Engagement Records; Follow-Up Candidates; Indicator Availability Information |
| 4.0 | D2 | Report Audit Entry |
| D2 | 4.0 | Audit Entries |

> Only permitted, available measures are used. An absent event is not automatically treated as a learner's lack of participation. When completion tracking or other measures are disabled, the report must indicate their unavailability.

## 4. Level-1 DFD - Process 1.0: Authorize Analytics Access

![Yourdon Level-1 DFD of process 1.0](diagrams/03_level1_p1.svg)

| Subprocess | Description |
|---|---|
| **1.1 Capture Analytics Request** | Receive the teacher's analytics request, chosen sections, dates, and activity filters |
| **1.2 Maintain Module Rules** | Receive authorized administrator configuration and role rules; update `D1 Module Rules` |
| **1.3 Evaluate Authorized Scope** | Compare the requested scope against active rules and permitted KHub course-section access |
| **1.4 Record Access Decision** | Pass permitted scope and criteria to 2.0; relay access status to 4.0; record audit metadata in D2 |

**Parent-boundary balance:** This decomposition retains the same five incoming external flows from Teacher and ICT/LMS Administrator, the two inputs/outputs with D1, and the four outgoing flows to other processes/D2 as Process 1.0 in Level-0. Rejected access must not be forwarded as a usable authorized scope.

## 5. Level-1 DFD - Process 2.0: Retrieve KHub Records

![Yourdon Level-1 DFD of process 2.0](diagrams/04_level1_p2.svg)

| Subprocess | Description |
|---|---|
| **2.1 Prepare Authorized Query** | Turn authorized scope and filters into a retrieval specification |
| **2.2 Issue KHub Data Query** | Submit `Authorized Analytics Query Parameters` through an approved read-only KHub interface |
| **2.3 Collect Source Records** | Receive course-section, enrollment, activity-log, and available completion/submission data |
| **2.4 Validate Retrieved Scope** | Check returned information against the approved request; provide usable records and retrieval status; write audit metadata |

**Parent-boundary balance:** Inputs from 1.0 and KHub, the query output to KHub, outputs to 3.0 and 4.0, and the audit entry written to D2 match Process 2.0 in Level-0.

## 6. Level-1 DFD - Process 3.0: Compute Engagement Measures

![Yourdon Level-1 DFD of process 3.0](diagrams/05_level1_p3.svg)

| Subprocess | Description |
|---|---|
| **3.1 Organize Activity Records** | Align permitted records by learner, course-section, activity, reporting interval, and supported indicators |
| **3.2 Calculate Section Measures** | Build consistent section-level counts and comparisons from recorded events |
| **3.3 Identify Review Candidates** | Find learner records matching a teacher-selected observable rule, such as an applicable missing submission |
| **3.4 Consolidate Report Results** | Produce metrics, learner status, candidate records, and indicator-availability explanations |

**Parent-boundary balance:** Two incoming flows from 2.0 and four outgoing flows to 4.0 match Process 3.0 in Level-0. This process **does not** grade learners, make disciplinary decisions, or write completion changes back to KHub.

## 7. Level-1 DFD - Process 4.0: Present Reports and Status

![Yourdon Level-1 DFD of process 4.0](diagrams/06_level1_p4.svg)

| Subprocess | Description |
|---|---|
| **4.1 Assemble Teacher Views** | Prepare the section summary, permitted learner information, and candidate details from calculated results |
| **4.2 Deliver Engagement Reports** | Send the three teacher-facing outputs and record report-access metadata in D2 |
| **4.3 Compile Administrative Status** | Combine access, retrieval, report-delivery, and permitted audit data |
| **4.4 Deliver Administrator Information** | Send `Access / Audit Information` and `Module Status` to the ICT/LMS Administrator |

**Parent-boundary balance:** The incoming information from 1.0, 2.0, 3.0 and D2, the three teacher outputs, two administrator outputs, and one audit-store write match Process 4.0 in Level-0.



**Use of AI:** An AI assistant was used to organize the source submissions, prepare the draft process decompositions and render the diagrams. I have inspected the diagrams, confirm that the models match the actual KHub environment and assume editorial responsibility.
