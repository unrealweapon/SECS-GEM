# SEMI E94 — Control Job Management Standard (CJM)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Control Job Concept](#control-job-concept)
4. [Control Job State Model](#control-job-state-model)
5. [Control Job Attributes](#control-job-attributes)
6. [Control Job Commands](#control-job-commands)
7. [SECS-II Messages](#secs-ii-messages)
8. [Material Routing](#material-routing)
9. [Scenarios & Message Flows](#scenarios--message-flows)
10. [Relationship to Other Standards](#relationship-to-other-standards)
11. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E94** defines the **Control Job Management Standard (CJM)**, which provides lot-level job management and material routing coordination on semiconductor equipment. It operates at a higher level than Process Jobs (E40), orchestrating the overall flow of material through equipment.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E94 |
| Full Name | Control Job Management Standard |
| Abbreviation | CJM |
| Primary Purpose | Lot-level job orchestration and material routing |
| Dependencies | SEMI E5, E30, E39, E40, E87 |
| Related | E40 (PJM), E87 (CMS), E90 (STS) |
| Level | Higher than Process Job, lower than MES scheduling |

---

## Purpose & Scope

### What E94 Defines

1. **Control Job** — A lot-level unit of work that coordinates carriers, recipes, and process jobs
2. **State Model** — Lifecycle of a control job
3. **Material Routing** — How substrates route through equipment modules
4. **Job Coordination** — How control jobs interact with process jobs
5. **Priority Management** — Job scheduling and prioritization
6. **Carrier Association** — Linking carriers to control jobs

### Control Job vs. Process Job

```
┌─────────────────────────────────────────────────────────────────┐
│ MES / Factory Host                                               │
│   "Process LOT_A using recipe ETCH_01 at Equipment TOOL_1"      │
└──────────────────────────────────┬──────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│ CONTROL JOB (E94) — CJ001                                       │
│   Lot: LOT_A                                                     │
│   Carrier: FOUP001                                               │
│   Recipe: ETCH_01                                                │
│   Routing: LP1 → PM1 → LP1                                     │
│                                                                  │
│   Creates and manages:                                           │
│   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│   │ Process Job     │  │ Process Job     │  │ Process Job     │  │
│   │ PJ001 (W01-W05)│  │ PJ002 (W06-W10)│  │ PJ003 (W11-W15)│  │
│   └────────────────┘  └────────────────┘  └────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Hierarchy

| Level | Standard | Scope | Example |
|-------|----------|-------|---------|
| Factory | MES | Multi-tool scheduling | "Run LOT_A on TOOL_1" |
| **Control** | **E94** | **Single-tool lot management** | **"Route FOUP001 through TOOL_1"** |
| Process | E40 | Module-level processing | "Run ETCH_01 on W01-W05 in PM1" |
| Equipment | E30 | Basic equipment control | Status, alarms, events |

---

## Control Job Concept

### What is a Control Job?

A Control Job represents the complete processing plan for material at a single equipment:

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONTROL JOB: CJ001                             │
├─────────────────────────────────────────────────────────────────┤
│  Job ID:             "CJ001"                                     │
│  Lot ID:             "LOT_A"                                     │
│  Input Carriers:     ["FOUP001"]                                 │
│  Output Carriers:    ["FOUP001"] (return to same)                │
│  Starting Ports:     [LP1]                                       │
│  Recipe:             "ETCH_01"                                   │
│  Material:           All wafers in FOUP001 (25 wafers)           │
│  Process Order:      FIFO (first in, first out)                  │
│  Priority:           Normal (50)                                 │
│  Auto-Start:         TRUE                                        │
├─────────────────────────────────────────────────────────────────┤
│  Status:             EXECUTING                                   │
│  Progress:           15/25 wafers processed                      │
│  Active Process Jobs: PJ004 (W16-W20 in PM1)                    │
└─────────────────────────────────────────────────────────────────┘
```

### Control Job Responsibilities

| Responsibility | Description |
|---------------|-------------|
| Carrier management | Link carriers to job, coordinate access |
| Recipe selection | Specify recipe for processing |
| Material selection | Define which substrates to process |
| Routing | Define path through equipment modules |
| Process job creation | Create process jobs as needed |
| Sequencing | Order of substrate processing |
| Completion tracking | Monitor progress to completion |
| Error handling | Respond to processing failures |

---

## Control Job State Model

### State Diagram

```
                    ┌─────────────────────┐
         Create    │                     │
   ───────────────→│     QUEUED           │
                    │  (Waiting to start) │
                    └──────────┬──────────┘
                               │ Resources ready
                    ┌──────────┴──────────┐
                    │     SELECTED         │
                    │  (Resources claimed, │
                    │   ready to execute) │
                    └──────────┬──────────┘
                               │ Start (auto or command)
                    ┌──────────┴──────────┐
                    │     EXECUTING        │◄────────────┐
                    │  (Actively running   │             │
                    │   process jobs)      │             │ RESUME
                    └──────────┬──────────┘             │
                               │                        │
              ┌────────────────┼────────────────┐       │
              │                │                │       │
              ▼                ▼                ▼       │
   ┌──────────────┐  ┌──────────────┐  ┌─────────────┐│
   │  COMPLETED    │  │  STOPPING    │  │  PAUSED     ││
   │  (All done)  │  │  (Graceful)  │  │  (Held)     │┘
   └──────────────┘  └──────┬───────┘  └─────────────┘
                             ▼
                    ┌──────────────┐
                    │   STOPPED    │
                    └──────────────┘

   Any Active State ──ABORT──→ ┌──────────────┐
                               │  ABORTING    │
                               └──────┬───────┘
                                      ▼
                               ┌──────────────┐
                               │   ABORTED    │
                               └──────────────┘
```

### Control Job States

| State | Value | Description |
|-------|-------|-------------|
| **QUEUED** | 1 | Job created, waiting for resources (carrier, port) |
| **SELECTED** | 2 | Resources reserved, ready to begin |
| **WAITING FOR START** | 3 | Ready for host/operator START command |
| **EXECUTING** | 4 | Actively processing material |
| **PAUSED** | 5 | Execution suspended |
| **COMPLETED** | 6 | All material successfully processed |
| **STOPPING** | 7 | Graceful stop in progress |
| **STOPPED** | 8 | Gracefully terminated |
| **ABORTING** | 9 | Emergency stop in progress |
| **ABORTED** | 10 | Abnormally terminated |

### State Transition Rules

| From | Event/Command | To | Notes |
|------|--------------|-----|-------|
| — | Create | QUEUED | New job enters queue |
| QUEUED | Resources available | SELECTED | Carrier at port, module free |
| SELECTED | Auto-start or START cmd | EXECUTING | Begin processing |
| EXECUTING | All material done | COMPLETED | Normal completion |
| EXECUTING | PAUSE | PAUSED | Suspend after current substrate |
| EXECUTING | STOP | STOPPING | Finish current, stop new |
| EXECUTING | ABORT | ABORTING | Immediate termination |
| PAUSED | RESUME | EXECUTING | Continue processing |
| PAUSED | STOP | STOPPING | Graceful stop from paused |
| PAUSED | ABORT | ABORTING | Emergency from paused |
| STOPPING | All active PJs done | STOPPED | Graceful complete |
| ABORTING | All active PJs aborted | ABORTED | Emergency complete |
| QUEUED | CANCEL/DELETE | (removed) | Job removed from queue |

---

## Control Job Attributes

### Required Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| **ObjID (CtrlJobID)** | ASCII | Unique control job identifier |
| **CtrlJobState** | U1 | Current state |
| **MtlCarrierList** | List | Input carrier(s) for this job |
| **ProcessingCtrlSpec** | List | Recipe/routing specification |
| **StartMethod** | U1 | How job starts (auto/manual) |
| **CurrentPRJob** | ASCII | Currently active process job(s) |

### Optional Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| Priority | U1 | Job priority (1=highest) |
| DataCollectionPlan | ASCII | Which data to collect |
| OutputCarrierList | List | Output carrier(s) |
| ProcessStartPort | List | Which port(s) to load from |
| ProcessEndPort | List | Which port(s) to unload to |
| EstimatedCompletionTime | ASCII | Predicted finish time |
| CtrlJobCreateTime | ASCII | When job was created |
| CtrlJobStartTime | ASCII | When execution began |
| CtrlJobEndTime | ASCII | When job finished |
| MtlProcessOrder | U1 | Processing sequence rule |
| MaxProcessCount | U4 | Max simultaneous process jobs |
| PauseEvent | List | Events that auto-pause |
| MtlOutSpec | List | Material output routing |
| ObjType | ASCII | "ControlJob" |

### StartMethod Values

| Value | Method | Description |
|-------|--------|-------------|
| 0 | Manual | Wait for explicit START command |
| 1 | Auto (immediate) | Start as soon as SELECTED |
| 2 | Auto (carrier arrives) | Start when carrier detected |

### MtlProcessOrder Values

| Value | Order | Description |
|-------|-------|-------------|
| 0 | FIFO | First in, first out (slot 1 first) |
| 1 | LIFO | Last in, first out (slot 25 first) |
| 2 | Optimized | Equipment determines optimal order |
| 3 | HostDefined | Host specifies exact sequence |

### ProcessingCtrlSpec Format

```
{L:n
  {L:3
    <A ProcessStepID>     ← Step identifier
    <A RecipeID>          ← Recipe for this step
    {L:m                   ← Recipe parameters
      {L:2
        <A ParamName>
        <V ParamValue>
      }
    }
  }
}
```

---

## Control Job Commands

### Command Summary

| Command | Valid From | Description |
|---------|-----------|-------------|
| **CREATE** | N/A | Create new control job |
| **START** | WAITING FOR START | Begin execution |
| **PAUSE** | EXECUTING | Suspend execution |
| **RESUME** | PAUSED | Continue execution |
| **STOP** | EXECUTING, PAUSED | Graceful termination |
| **ABORT** | Any active | Emergency termination |
| **CANCEL** | QUEUED | Remove from queue |
| **DELETE** | COMPLETED, STOPPED, ABORTED | Remove finished job |
| **DESELECT** | SELECTED | Release resources |
| **PRIORITY_CHANGE** | Any active | Change job priority |

### Command Behavior

#### PAUSE Behavior
- Current substrate in process module continues to completion
- No new substrate enters process module
- Robot may complete current transfer
- Job enters PAUSED state when safe point reached

#### STOP Behavior
- All currently active process jobs finish normally
- No new process jobs are created
- Unprocessed substrates returned to carrier
- Job enters STOPPED state

#### ABORT Behavior
- Active process jobs are aborted immediately
- Substrates may be at unsafe processing point
- Equipment performs safe shutdown of processes
- All substrates returned to carrier (if possible)
- Job enters ABORTED state

---

## SECS-II Messages

### Primary Messages for CJM

Control Job operations use a combination of messages:

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S14F9/F10 | Create Object/Ack | H→E | Create control job |
| S14F11/F12 | Delete Object/Ack | H→E | Delete control job |
| S14F1/F2 | GetAttr/Data | H→E | Query control job attributes |
| S14F3/F4 | SetAttr/Data | H→E | Modify control job |
| S2F41/F42 | Host Command/Ack | H→E | Job commands (START/STOP/etc.) |
| S2F49/F50 | Enhanced Remote Cmd/Ack | H→E | Extended commands with OBJSPEC |
| S6F11/F12 | Event Report/Ack | E→H | Job state change events |

### Creating a Control Job (S14F9)

**Request:**
```
{L:3
  <A "ControlJob/CJ001">         ← OBJSPEC
  <A "ControlJob">               ← Object Type
  {L:7                            ← Initial attributes
    {L:2
      <A "CtrlJobID">
      <A "CJ001">
    }
    {L:2
      <A "MtlCarrierList">
      {L:1 <A "FOUP001">}
    }
    {L:2
      <A "ProcessingCtrlSpec">
      {L:1
        {L:3
          <A "Step1">
          <A "ETCH_01">
          {L:0}
        }
      }
    }
    {L:2
      <A "StartMethod">
      <U1 1>                      ← Auto-start
    }
    {L:2
      <A "ProcessStartPort">
      {L:1 <U1 1>}               ← LP1
    }
    {L:2
      <A "MtlProcessOrder">
      <U1 0>                      ← FIFO
    }
    {L:2
      <A "Priority">
      <U1 50>                     ← Normal priority
    }
  }
}
```

**Response (S14F10):**
```
{L:2
  <U1 OBJACK>                    ← 0 = success
  {L:n                            ← Errors (empty if success)
    {L:2
      <U1 ERRCODE>
      <A ERRTEXT>
    }
  }
}
```

### Sending Job Commands (S2F49)

**S2F49 — Enhanced Remote Command with OBJSPEC:**
```
{L:4
  <U4 DATAID>
  <A "ControlJob/CJ001">         ← OBJSPEC (target CJ)
  <A "START">                     ← Command
  {L:0}                           ← Parameters (none for START)
}
```

**S2F50 — Response:**
```
{L:2
  <U1 HCACK>                     ← 0 = success
  {L:n
    {L:2
      <A CPNAME>
      <U1 CPACK>                  ← Parameter-level acknowledge
    }
  }
}
```

---

## Material Routing

### Routing Concept

Control jobs define how material moves through equipment:

```
┌─────────────────────────────────────────────────────────────────┐
│                    EQUIPMENT ROUTING MAP                          │
│                                                                  │
│  INPUT              PROCESSING            OUTPUT                 │
│  ┌────┐            ┌────┐                ┌────┐                │
│  │LP1 │───────────→│PM1 │───────────────→│LP1 │  (Return)      │
│  │    │     ┌─────→│    │                │    │                │
│  └────┘     │      └────┘                └────┘                │
│             │      ┌────┐                ┌────┐                │
│  ┌────┐     │      │PM2 │───────────────→│LP2 │  (Output)      │
│  │LP2 │─────┘ ┌───→│    │                │    │                │
│  │    │────────┘    └────┘                └────┘                │
│  └────┘                                                          │
│                                                                  │
│  Route 1: LP1 → PM1 → LP1 (standard)                           │
│  Route 2: LP1 → PM2 → LP2 (cross-carrier)                      │
│  Route 3: LP2 → PM1 → LP1 (input/output split)                 │
└─────────────────────────────────────────────────────────────────┘
```

### Routing Types

| Type | Description | Example |
|------|-------------|---------|
| **Return to Source** | Substrate returns to same carrier/slot | LP1→PM1→LP1 |
| **Cross-Carrier** | Substrate moves to different carrier | LP1→PM1→LP2 |
| **Output Port** | Substrate goes to designated output | LP1→PM1→LP3(out) |
| **Multi-Step** | Multiple processing modules | LP1→PM1→PM2→LP1 |
| **Inspection Route** | Processing + measurement | LP1→PM1→MEAS→LP1 |

### MtlOutSpec (Material Output Specification)

```
{L:n
  {L:4
    <A SourceCarrierID>           ← From which carrier
    <A DestCarrierID>             ← To which carrier
    {L:m                           ← Slot mapping
      {L:2
        <U1 SourceSlot>
        <U1 DestSlot>
      }
    }
    <A ProcessResult>             ← Route based on result (optional)
  }
}
```

---

## Scenarios & Message Flows

### Scenario 1: Complete Control Job Lifecycle

```
Host                                        Equipment
 │                                              │
 │  ═══ PRE-CONDITIONS ═══                     │
 │  (Carrier FOUP001 already at LP1,           │
 │   ID verified, slot map confirmed)           │
 │                                              │
 │  ═══ CREATE CONTROL JOB ═══                 │
 │                                              │
 │─── S14F9 (Create ControlJob CJ001) ────────→│
 │     Carrier=FOUP001, Recipe=ETCH_01          │
 │     StartMethod=AUTO, Port=LP1              │
 │←── S14F10 (OBJACK=0, created) ─────────────│
 │                                              │
 │←── S6F11 (CJStateChanged: QUEUED) ─────────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ RESOURCE SELECTION ═══                 │
 │  (Carrier at port, module available)         │
 │                                              │
 │←── S6F11 (CJStateChanged: SELECTED) ───────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ AUTO-START EXECUTION ═══               │
 │  (StartMethod=AUTO)                          │
 │                                              │
 │←── S6F11 (CJStateChanged: EXECUTING) ──────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  (Equipment creates process jobs internally) │
 │                                              │
 │←── S6F11 (PRJobCreated: PJ001) ────────────│  Process Job for first batch
 │←── S6F11 (PRJobStateChange: PROCESSING) ───│
 │                                              │
 │         ... substrates being processed ...    │
 │                                              │
 │←── S6F11 (PRJobComplete: PJ001) ───────────│  First batch done
 │←── S6F11 (PRJobCreated: PJ002) ────────────│  Next batch
 │                                              │
 │         ... continues until all done ...      │
 │                                              │
 │←── S6F11 (CJStateChanged: COMPLETED) ──────│  All substrates processed
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ CLEANUP ═══                            │
 │                                              │
 │─── S14F11 (Delete CJ001) ──────────────────→│  Remove completed job
 │←── S14F12 (OBJACK=0) ──────────────────────│
```

### Scenario 2: Multiple Control Jobs (Priority)

```
Host                                        Equipment
 │                                              │
 │─── S14F9 (Create CJ001, Priority=50) ──────→│  Normal priority
 │←── S14F10 (OK) ────────────────────────────│
 │                                              │
 │─── S14F9 (Create CJ002, Priority=10) ──────→│  HIGH priority
 │←── S14F10 (OK) ────────────────────────────│
 │                                              │
 │  Equipment processes CJ002 first (Priority=10 is higher)
 │                                              │
 │←── S6F11 (CJ002: EXECUTING) ───────────────│  High priority starts first
 │                                              │
 │  After CJ002 completes, CJ001 starts        │
 │                                              │
 │←── S6F11 (CJ001: EXECUTING) ───────────────│  Normal priority follows
```

### Scenario 3: Control Job with Process Job Creation

```
Host                                        Equipment
 │                                              │
 │  CJ001 EXECUTING with 25 wafers             │
 │  Equipment creates PJs in batches of 5:      │
 │                                              │
 │←── S6F11 (PRJob PJ001 created, W01-W05) ───│  Batch 1
 │←── S6F11 (PJ001 PROCESSING) ───────────────│
 │                                              │
 │←── S6F11 (PRJob PJ002 created, W06-W10) ───│  Batch 2 (pipeline)
 │                                              │
 │←── S6F11 (PJ001 COMPLETED) ────────────────│  Batch 1 done
 │←── S6F11 (PJ002 PROCESSING) ───────────────│  Batch 2 starts
 │                                              │
 │←── S6F11 (PRJob PJ003 created, W11-W15) ───│  Batch 3
 │                                              │
 │  ... continues until all 25 wafers done ...  │
 │                                              │
 │←── S6F11 (CJ001 COMPLETED) ────────────────│  All batches done
```

### Scenario 4: Pause and Resume

```
Host                                        Equipment
 │                                              │
 │  CJ001 EXECUTING (PJ003 currently active)   │
 │                                              │
 │─── S2F49 (PAUSE, CJ001) ───────────────────→│
 │←── S2F50 (HCACK=0) ────────────────────────│
 │                                              │
 │  (PJ003 finishes current wafer, then pauses) │
 │                                              │
 │←── S6F11 (CJ001: PAUSED) ──────────────────│
 │                                              │
 │  ... operator resolves issue ...              │
 │                                              │
 │─── S2F49 (RESUME, CJ001) ──────────────────→│
 │←── S2F50 (HCACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CJ001: EXECUTING) ───────────────│  Resumed
```

---

## Collection Events for E94

| Event | Description | Data Variables |
|-------|-------------|----------------|
| CJCreated | Control job created | CtrlJobID, MtlCarrierList |
| CJStateChanged | State transition | CtrlJobID, PrevState, NewState |
| CJSelected | Resources reserved | CtrlJobID |
| CJStarted | Execution began | CtrlJobID, StartTime |
| CJCompleted | All material done | CtrlJobID, EndTime |
| CJPaused | Execution paused | CtrlJobID, Reason |
| CJResumed | Execution resumed | CtrlJobID |
| CJStopped | Gracefully stopped | CtrlJobID, Reason |
| CJAborted | Emergency stop | CtrlJobID, Reason |
| CJDeleted | Job removed | CtrlJobID |
| CJWaitingForStart | Ready for command | CtrlJobID |

---

## Relationship to Other Standards

### Integration Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Factory Host / MES                          │
└──────────────────────────────┬──────────────────────────────────┘
                               │ "Process LOT_A at TOOL_1"
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    E94 — CONTROL JOB                              │
│                                                                  │
│  Coordinates:                                                    │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │ E87 (Carrier)     │  │ E40 (Process Job) │                   │
│  │ - Carrier at port │  │ - Recipe execution │                   │
│  │ - Slot map done   │  │ - Material batching│                   │
│  │ - Access granted  │  │ - Module control   │                   │
│  └──────────────────┘  └──────────────────┘                    │
│                                                                  │
│  Monitors:                                                       │
│  ┌──────────────────┐                                           │
│  │ E90 (Substrate)   │                                          │
│  │ - Track progress  │                                           │
│  │ - Verify location │                                           │
│  └──────────────────┘                                           │
└─────────────────────────────────────────────────────────────────┘
```

### CJ Dependencies

| Dependency | E94 Needs From | For What |
|------------|---------------|----------|
| E87 | Carrier state, slot map | Know what material is available |
| E40 | Process job management | Execute actual processing |
| E90 | Substrate tracking | Monitor material progress |
| E39 | Object services | Object model framework |
| E30 | GEM base | Events, remote commands |

---

## Implementation Guidelines

### Minimum Requirements

1. Support at least 1 simultaneous control job
2. Implement full state model with all transitions
3. Coordinate with E87 for carrier access
4. Create process jobs (E40) for material processing
5. Report all state change events
6. Support START, STOP, ABORT commands
7. Support job DELETE after completion

### Capacity Planning

| Parameter | Typical Range | Description |
|-----------|---------------|-------------|
| Max simultaneous CJs | 1-6 | Limited by load ports |
| Max queued CJs | 3-20 | Limited by scheduling depth |
| Substrates per CJ | 1-25+ | Typically full carrier |
| PJs per CJ | 1-25 | Depends on batching |
| CJ execution time | Minutes to hours | Process dependent |

### Error Conditions

| Error | CJ Response | Host Action |
|-------|-------------|-------------|
| Carrier not at port | Stays QUEUED | Deliver carrier or cancel |
| Recipe not available | Reject creation | Upload recipe first |
| Module failure | PAUSE or ABORT | Resolve and RESUME or ABORT |
| Substrate processing fail | Continue or STOP | Depends on policy |
| Port suddenly unavailable | PAUSE | Resolve port issue |
| All Process Jobs failed | ABORTED | Investigate, re-run |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E94-0710 Standard*
