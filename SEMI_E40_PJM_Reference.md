# SEMI E40 — Processing Management Standard (PJM)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Process Job Concept](#process-job-concept)
4. [Process Job State Model](#process-job-state-model)
5. [Process Job Attributes](#process-job-attributes)
6. [Process Job Commands](#process-job-commands)
7. [SECS-II Messages](#secs-ii-messages)
8. [Material Handling in PJM](#material-handling-in-pjm)
9. [Multi-Recipe Processing](#multi-recipe-processing)
10. [Scenarios & Message Flows](#scenarios--message-flows)
11. [Relationship to Other Standards](#relationship-to-other-standards)
12. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E40** defines the **Processing Management Standard**, commonly referred to as **Process Job Management (PJM)**. It standardizes how a host system creates, controls, and monitors processing activities on equipment.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E40 |
| Full Name | Processing Management Standard |
| Abbreviation | PJM |
| Primary Purpose | Process-level job management on equipment |
| Dependencies | SEMI E5, E30, E39 |
| Related | E94 (Control Job), E87 (Carrier), E90 (Substrate) |
| Messages | S16F1-F28 (Stream 16) |

---

## Purpose & Scope

### What E40 Defines

1. **Process Job** — A unit of work that specifies WHAT to process, HOW to process, and WHERE
2. **State Model** — Lifecycle states of a process job
3. **Commands** — Operations to control process jobs
4. **Attributes** — Properties describing each process job
5. **Events** — Notifications of process job state changes
6. **Material Association** — Which substrates are assigned to which job

### Process Job vs. Control Job

| Aspect | Process Job (E40) | Control Job (E94) |
|--------|-------------------|-------------------|
| Level | Machine-level | Lot/batch-level |
| Scope | Specific processing module | Entire equipment |
| Material | Specific substrates | Carriers/lots |
| Recipe | Specific process recipe | May span multiple recipes |
| Controls | Start/stop/abort processing | Routing, carrier management |
| Analogy | "Run recipe X on these wafers" | "Process this lot through the tool" |

---

## Process Job Concept

### What is a Process Job?

A Process Job is an instruction to equipment that specifies:

```
┌─────────────────────────────────────────────────────────────┐
│                    PROCESS JOB                                │
├─────────────────────────────────────────────────────────────┤
│  Job ID:        "PJ001"                                      │
│  Recipe:        "ETCH_OXIDE_01"                              │
│  Material:      [W01, W02, W03, W04, W05]                   │
│  Start Method:  Auto-start when material arrives             │
│  Priority:      Normal                                       │
│  Parameters:    {Temp=350°C, Pressure=100mT}                │
└─────────────────────────────────────────────────────────────┘
```

### Process Job Scope

```
┌─────────────────────────────────────────────────────────────┐
│                    EQUIPMENT                                   │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Process Job  │  │  Process Job  │  │  Process Job  │     │
│  │    PJ001     │  │    PJ002     │  │    PJ003     │      │
│  │              │  │              │  │              │      │
│  │  Recipe: A   │  │  Recipe: B   │  │  Recipe: A   │      │
│  │  Module: PM1 │  │  Module: PM2 │  │  Module: PM1 │      │
│  │  Wafers:1-5 │  │  Wafers:6-10│  │  Wafers:11-15│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  Multiple process jobs can exist simultaneously              │
│  Multiple jobs can share the same module (queued)            │
└─────────────────────────────────────────────────────────────┘
```

---

## Process Job State Model

### State Diagram

```
                              ┌───────────────┐
                              │   QUEUED       │
              ┌──────────────→│ (Waiting for   │
              │               │  resources)    │
              │               └───────┬───────┘
              │                       │ Resources available
              │                       ▼
┌─────────────┴──┐           ┌───────────────┐
│ SETTING UP     │←──────────│  SELECTING     │
│ (Preparing     │  auto     │  MATERIAL     │
│  equipment)    │           └───────────────┘
└───────┬────────┘
        │ Setup complete
        ▼
┌───────────────┐           ┌───────────────┐
│  WAITING FOR   │──────────→│  PROCESSING   │
│  START         │  Start    │  (Active)     │
│               │  command   │              │
└───────────────┘           └───────┬───────┘
                                    │ Complete
                                    ▼
                            ┌───────────────┐
                            │ PROCESS       │
                            │ COMPLETE      │
                            └───────────────┘
```

### Complete State Model (All States)

```
                    ┌─────────────────────┐
          Create    │                     │
   ───────────────→│      QUEUED          │
                    │  (Job accepted,     │
                    │   waiting in queue) │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │  SETTING UP          │
                    │  (Recipe loading,    │
                    │   module preparing)  │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │  WAITING FOR START   │
                    │  (Ready, waiting    │
                    │   for START cmd)    │
                    └──────────┬──────────┘
                               │ START
                    ┌──────────┴──────────┐
                    │  PROCESSING          │◄────────────┐
                    │  (Active processing) │             │ RESUME
                    └──────────┬──────────┘             │
                               │                        │
              ┌────────────────┼────────────────┐       │
              │                │                │       │
              ▼                ▼                ▼       │
   ┌──────────────┐  ┌──────────────┐  ┌─────────────┐│
   │PROCESS       │  │  STOPPING    │  │  PAUSING    ││
   │COMPLETE      │  │(Graceful end)│  │             ││
   └──────────────┘  └──────┬───────┘  └──────┬──────┘│
                             │                 │       │
                             ▼                 ▼       │
                    ┌──────────────┐   ┌─────────────┐ │
                    │  STOPPED     │   │   PAUSED    │─┘
                    └──────────────┘   └─────────────┘
                    
   Any State ──ABORT──→ ┌──────────────┐
                        │  ABORTING    │
                        └──────┬───────┘
                               ▼
                        ┌──────────────┐
                        │  ABORTED     │
                        └──────────────┘
```

### State Definitions

| State | Description | Entry Condition |
|-------|-------------|-----------------|
| **QUEUED** | Job created, waiting for resources | S16F1 accepted |
| **SETTING UP** | Module preparing (recipe load, etc.) | Resources available |
| **WAITING FOR START** | Ready, awaiting start command | Setup complete |
| **PROCESSING** | Actively processing material | START command |
| **PAUSING** | Transitioning to paused | PAUSE command |
| **PAUSED** | Processing suspended | Pause complete |
| **PROCESS COMPLETE** | Normal completion | All material processed |
| **STOPPING** | Graceful termination in progress | STOP command |
| **STOPPED** | Gracefully terminated | Stop complete |
| **ABORTING** | Emergency termination in progress | ABORT command |
| **ABORTED** | Emergency termination complete | Abort complete |

### State Transition Table

| From State | Command/Event | To State |
|-----------|--------------|----------|
| — | Create (S16F1) | QUEUED |
| QUEUED | Resources available | SETTING UP |
| QUEUED | ABORT | ABORTING |
| SETTING UP | Setup complete | WAITING FOR START (or PROCESSING if auto-start) |
| SETTING UP | ABORT | ABORTING |
| WAITING FOR START | START | PROCESSING |
| WAITING FOR START | ABORT | ABORTING |
| PROCESSING | Complete | PROCESS COMPLETE |
| PROCESSING | PAUSE | PAUSING |
| PROCESSING | STOP | STOPPING |
| PROCESSING | ABORT | ABORTING |
| PAUSING | Pause complete | PAUSED |
| PAUSED | RESUME | PROCESSING |
| PAUSED | STOP | STOPPING |
| PAUSED | ABORT | ABORTING |
| STOPPING | Stop complete | STOPPED |
| ABORTING | Abort complete | ABORTED |

---

## Process Job Attributes

### Required Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| **PRJobID** | ASCII | Unique process job identifier |
| **PRMtlNameList** | List of ASCII | Material (substrate IDs) assigned to job |
| **PRRecipeMethod** | List | Recipe specification |
| **PRProcessStart** | Boolean | TRUE=auto-start, FALSE=wait for command |
| **PRJobState** | U1 | Current state of the job |

### Optional Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| PRPriority | U1 | Job priority (1=highest, 255=lowest) |
| PRPausedEvent | List | Events that auto-pause the job |
| PRJobCreateTime | ASCII | Timestamp of job creation |
| PRJobStartTime | ASCII | Timestamp processing started |
| PRJobEndTime | ASCII | Timestamp processing ended |
| PRModuleName | ASCII | Target module for processing |
| PRMtlType | U1 | Material type (wafer, panel, etc.) |

### PRRecipeMethod Format

The recipe specification for a process job:

```
{L:n
  {L:5
    <U1 RecipeType>        ← 0=recipe, 1=recipe with variables
    <A RecipeID>           ← Recipe/PP identifier
    {L:m                   ← Recipe parameters (overrides)
      {L:2
        <A ParamName>
        <V ParamValue>
      }
    }
    <A FixtureID>          ← Fixture/jig identifier (optional)
    <A ModuleSpec>         ← Target module specification
  }
}
```

### PRJobState Values

| Value | State |
|-------|-------|
| 1 | QUEUED |
| 2 | SETTING UP |
| 3 | WAITING FOR START |
| 4 | PROCESSING |
| 5 | PROCESS COMPLETE |
| 6 | STOPPING |
| 7 | PAUSING |
| 8 | PAUSED |
| 9 | STOPPED |
| 10 | ABORTING |
| 11 | ABORTED |

---

## Process Job Commands

### Command Summary

| Command | Description | Valid From States |
|---------|-------------|-----------------|
| **CREATE** | Create a new process job | N/A (no existing job) |
| **START** | Begin processing | WAITING FOR START |
| **STOP** | Graceful termination | PROCESSING, PAUSED |
| **ABORT** | Immediate termination | Any active state |
| **PAUSE** | Suspend processing | PROCESSING |
| **RESUME** | Continue from pause | PAUSED |
| **CANCEL** | Remove queued job | QUEUED |

### Command Parameters

#### CREATE Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| PRJobID | Yes | Unique job identifier |
| PRMtlNameList | Yes | List of substrate IDs |
| PRRecipeMethod | Yes | Recipe specification |
| PRProcessStart | Yes | Auto-start flag |
| PRPriority | No | Job priority |

#### START Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| PRJobID | Yes | Job to start |

---

## SECS-II Messages

### Stream 16 — Processing Management Messages

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S16F1/F2 | PRJob Create Request/Ack | H→E | Create process job |
| S16F3/F4 | PRJob Dequeue Request/Ack | H→E | Remove queued job |
| S16F5/F6 | PRJob Command Request/Ack | H→E | Control job (START/STOP/etc.) |
| S16F7/F8 | PRJob State Request/Data | H→E | Query job state |
| S16F9/F10 | PRJob Get State Query/Data | H→E | Query multiple jobs |
| S16F11/F12 | PRJobCreate (Multi) | H→E | Create with full specification |
| S16F15/F16 | PRJob Multi-Create/Ack | H→E | Create multiple jobs |
| S16F17/F18 | PRJob Get Space/Data | H→E | Query job capacity |
| S16F19/F20 | PRJob Info Request/Data | H→E | Detailed job information |
| S16F27/F28 | PRJob Command With Params/Ack | H→E | Extended command with parameters |

### S16F11 — PRJobCreate (Primary)

**Request:**
```
{L:5
  <A PRJobID>              ← Unique job identifier
  <B PRMtlType>           ← Material type
  {L:n                     ← Material list
    {L:3
      <A CarrierID>        ← Source carrier
      {L:m                 ← Slot list
        <U1 SlotID>        ← Slot numbers
      }
      <A MtlPosition>     ← Material position specification
    }
  }
  {L:p                     ← Recipe method
    {L:5
      <U1 RecipeType>
      <A RecipeID>
      {L:q
        {L:2
          <A RcpVarName>
          <V RcpVarValue>
        }
      }
      <A FixtureID>
      <A ModuleSpec>
    }
  }
  <BOOLEAN PRProcessStart> ← Auto-start flag
}
```

### S16F12 — PRJobCreate Acknowledge

**Response:**
```
{L:2
  <U1 PRJobAck>           ← 0=accepted, non-zero=error
  {L:n                     ← Error list
    {L:2
      <U1 ErrCode>
      <A ErrText>
    }
  }
}
```

**PRJobAck Codes:**

| Code | Meaning |
|------|---------|
| 0 | OK, accepted |
| 1 | Invalid PRJobID (duplicate or invalid format) |
| 2 | Invalid material |
| 3 | Invalid recipe |
| 4 | PRJob quantity exceeded |
| 5 | Not in correct state |
| 6 | Equipment does not support command |
| 7 | Material already in use by another job |

### S16F5 — PRJob Command Request

**Request:**
```
{L:2
  <A PRJobID>             ← Job to control
  <U1 PRCommand>          ← Command code
}
```

**PRCommand Values:**

| Code | Command |
|------|---------|
| 1 | STOP |
| 2 | ABORT |
| 3 | PAUSE |
| 4 | RESUME |
| 5 | START |
| 6 | CANCEL |

### S16F6 — PRJob Command Ack

**Response:**
```
{L:2
  <U1 PRAck>              ← 0=accepted
  {L:n
    {L:2
      <U1 ErrCode>
      <A ErrText>
    }
  }
}
```

---

## Material Handling in PJM

### Material Specification

Process jobs reference material (substrates) through various methods:

#### By Substrate ID

```
PRMtlNameList:
  {L:3
    <A "W001">    ← Substrate ID 1
    <A "W002">    ← Substrate ID 2
    <A "W003">    ← Substrate ID 3
  }
```

#### By Carrier and Slot

```
{L:1
  {L:3
    <A "FOUP001">          ← Carrier ID
    {L:5                    ← Slot numbers
      <U1 1>
      <U1 2>
      <U1 3>
      <U1 4>
      <U1 5>
    }
    <A "">                 ← Position (optional)
  }
}
```

### Material State During Processing

```
Material arrives at equipment
         │
         ▼
┌─────────────────┐
│ NEEDS PROCESSING│   Substrate assigned to PJ but not yet processed
└────────┬────────┘
         │ PJ enters PROCESSING
         ▼
┌─────────────────┐
│  IN PROCESS     │   Substrate actively being processed
└────────┬────────┘
         │ Processing complete
         ▼
┌─────────────────┐
│   PROCESSED     │   Processing complete for this substrate
└─────────────────┘

OR

┌─────────────────┐
│   ABORTED       │   Processing was aborted
└─────────────────┘

┌─────────────────┐
│   STOPPED       │   Processing was stopped prematurely
└─────────────────┘

┌─────────────────┐
│   REJECTED      │   Material rejected (did not process)
└─────────────────┘

┌─────────────────┐
│    LOST         │   Material location unknown
└─────────────────┘
```

---

## Multi-Recipe Processing

### Sequential Multi-Recipe

Process job specifies multiple recipes executed in order:

```
PRRecipeMethod:
  {L:3                    ← Three recipe steps
    {L:5                  ← Step 1
      <U1 0>             ← Type: recipe
      <A "CLEAN_01">     ← Recipe name
      {L:0}              ← No overrides
      <A "">             ← No fixture
      <A "CM1">          ← Clean module
    }
    {L:5                  ← Step 2
      <U1 0>
      <A "ETCH_OXIDE">
      {L:0}
      <A "">
      <A "PM1">          ← Process module 1
    }
    {L:5                  ← Step 3
      <U1 0>
      <A "MEASURE_01">
      {L:0}
      <A "">
      <A "MEAS1">        ← Measurement module
    }
  }
```

### Recipe with Parameter Overrides

```
{L:1
  {L:5
    <U1 1>                 ← Type: recipe with variables
    <A "ETCH_STD">         ← Base recipe
    {L:3                    ← Parameter overrides
      {L:2
        <A "Temperature">
        <F4 350.0>
      }
      {L:2
        <A "Pressure">
        <F4 100.0>
      }
      {L:2
        <A "Duration">
        <U4 180>
      }
    }
    <A "">
    <A "PM1">
  }
}
```

---

## Scenarios & Message Flows

### Scenario 1: Basic Process Job Lifecycle

```
Host                                        Equipment
 │                                              │
 │─── S16F11 (Create PRJob) ──────────────────→│
 │     PRJobID="PJ001"                          │
 │     Material=[W01,W02,W03]                   │
 │     Recipe="ETCH_01"                         │
 │     PRProcessStart=FALSE                     │
 │←── S16F12 (PRJobAck=0) ────────────────────│  Job QUEUED
 │                                              │
 │←── S6F11 (PRJobStateChange: QUEUED) ───────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │          ... equipment prepares module ...    │
 │                                              │
 │←── S6F11 (PRJobStateChange: SETTING UP) ───│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │←── S6F11 (PRJobStateChange: WAIT START) ───│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S16F5 (PRCommand: START, PJ001) ────────→│
 │←── S16F6 (PRAck=0) ────────────────────────│
 │                                              │
 │←── S6F11 (PRJobStateChange: PROCESSING) ───│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │          ... processing occurs ...            │
 │                                              │
 │←── S6F11 (PRJobStateChange: COMPLETE) ─────│
 │─── S6F12 ───────────────────────────────────→│
```

### Scenario 2: Auto-Start Process Job

```
Host                                        Equipment
 │                                              │
 │─── S16F11 (Create PRJob) ──────────────────→│
 │     PRProcessStart=TRUE (auto-start)         │
 │←── S16F12 (PRJobAck=0) ────────────────────│
 │                                              │
 │←── S6F11 (QUEUED) ─────────────────────────│
 │←── S6F11 (SETTING UP) ─────────────────────│
 │←── S6F11 (PROCESSING) ─────────────────────│  Auto-started!
 │                                              │
 │          ... no START needed ...              │
 │                                              │
 │←── S6F11 (PROCESS COMPLETE) ───────────────│
```

### Scenario 3: Pause and Resume

```
Host                                        Equipment
 │                                              │
 │   (Job PJ001 is PROCESSING)                 │
 │                                              │
 │─── S16F5 (PAUSE, PJ001) ───────────────────→│
 │←── S16F6 (PRAck=0) ────────────────────────│
 │                                              │
 │←── S6F11 (PAUSING) ────────────────────────│
 │←── S6F11 (PAUSED) ─────────────────────────│
 │                                              │
 │   ... resolve issue ...                      │
 │                                              │
 │─── S16F5 (RESUME, PJ001) ──────────────────→│
 │←── S16F6 (PRAck=0) ────────────────────────│
 │                                              │
 │←── S6F11 (PROCESSING) ─────────────────────│  Resumed
```

### Scenario 4: Abort

```
Host                                        Equipment
 │                                              │
 │   (Job PJ001 is PROCESSING — emergency!)    │
 │                                              │
 │─── S16F5 (ABORT, PJ001) ───────────────────→│
 │←── S16F6 (PRAck=0) ────────────────────────│
 │                                              │
 │←── S6F11 (ABORTING) ───────────────────────│  Emergency stop
 │                                              │
 │          ... equipment safely stops ...       │
 │                                              │
 │←── S6F11 (ABORTED) ────────────────────────│  Complete
```

---

## Relationship to Other Standards

### E40 + E94 (Control Job Management)

```
┌─────────────────────────────────────────────────────┐
│              Control Job (E94)                        │
│   CJ001: Lot "LOT_A", Carrier "FOUP001"            │
│                                                      │
│   ┌─────────────┐  ┌─────────────┐                 │
│   │ Process Job  │  │ Process Job  │                │
│   │   PJ001     │  │   PJ002     │                 │
│   │ Recipe: A   │  │ Recipe: B   │                 │
│   │ W01-W05     │  │ W06-W10    │                  │
│   └─────────────┘  └─────────────┘                 │
└─────────────────────────────────────────────────────┘
```

### E40 + E87 (Carrier Management)

- E87 manages carrier arrival at load port
- Carrier slot map provides material information
- E40 process job references material FROM carrier
- After processing, E87 manages carrier departure

### E40 + E90 (Substrate Tracking)

- E90 tracks WHERE each substrate is
- E40 specifies WHICH substrates to process
- E90 updates substrate state as E40 job progresses
- Combined: full material traceability

---

## Implementation Guidelines

### Process Job Capacity

Equipment must define:
- Maximum simultaneous process jobs
- Maximum queued jobs
- Maximum material per job
- Supported recipe types

### Collection Events for PJM

| Event | Description | Typical Data |
|-------|-------------|--------------|
| PRJobCreated | New job created | PRJobID, Recipe, Material |
| PRJobStateChanged | Job changed state | PRJobID, OldState, NewState |
| PRJobStarted | Processing began | PRJobID, StartTime |
| PRJobCompleted | Processing finished | PRJobID, EndTime, Result |
| PRJobAborted | Job aborted | PRJobID, Reason |
| PRJobStopped | Job stopped | PRJobID, Reason |
| PRMaterialProcessed | Single substrate done | PRJobID, SubstrateID |

### Status Variables for PJM

| SVID | Name | Type | Description |
|------|------|------|-------------|
| — | PRJobCount | U4 | Number of active process jobs |
| — | PRJobIDList | List | List of all job IDs |
| — | PRJobState_xx | U1 | State of job xx |
| — | PRCurrentRecipe | ASCII | Currently executing recipe |

### Error Handling

| Error Condition | Equipment Response |
|----------------|-------------------|
| Duplicate PRJobID | Reject with error code 1 |
| Invalid material reference | Reject with error code 2 |
| Recipe not found | Reject with error code 3 |
| No capacity for new job | Reject with error code 4 |
| Command in wrong state | Reject with error code 5 |
| Material already assigned | Reject with error code 7 |
| Processing failure | Auto-transition to ABORTING |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E40-0708 Standard*
