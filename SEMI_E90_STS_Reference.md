# SEMI E90 — Substrate Tracking Standard (STS)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Substrate Concept](#substrate-concept)
4. [Substrate State Models](#substrate-state-models)
5. [Substrate Attributes](#substrate-attributes)
6. [Substrate Location Tracking](#substrate-location-tracking)
7. [Substrate ID Management](#substrate-id-management)
8. [SECS-II Messages & Commands](#secs-ii-messages--commands)
9. [Scenarios & Message Flows](#scenarios--message-flows)
10. [Relationship to Other Standards](#relationship-to-other-standards)
11. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E90** defines the **Substrate Tracking Standard (STS)**, which provides a comprehensive framework for tracking individual substrates (wafers, panels, etc.) as they move through equipment — from arrival to departure, including all intermediate locations and processing states.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E90 |
| Full Name | Substrate Tracking Standard |
| Abbreviation | STS |
| Primary Purpose | Individual substrate location and state tracking |
| Dependencies | SEMI E5, E30, E39, E87 |
| Related | E40 (PJM), E94 (CJM), E87 (CMS) |
| Key Object | Substrate (E39 object type) |

---

## Purpose & Scope

### What E90 Defines

1. **Substrate identity** — How substrates are identified and tracked
2. **Location tracking** — Where each substrate is at any moment
3. **Processing state** — What has been done to each substrate
4. **Transport state** — How substrate is being moved
5. **State models** — Lifecycle state machines for substrates
6. **Events** — Notifications when substrate state/location changes

### Why Substrate Tracking Matters

```
Without E90:                          With E90:
─────────────                         ─────────────
"Some wafers went in"                 "W001 entered PM1 at 10:23:01"
"Processing happened"                  "W001 recipe ETCH_01 started 10:23:05"
"Some wafers came out"                 "W001 processing complete 10:25:30"
"Which one is which?"                 "W001 returned to FOUP001 slot 1"
```

### Scope

```
┌─────────────────────────────────────────────────────────────────┐
│                         EQUIPMENT                                 │
│                                                                  │
│  Carrier (FOUP)        Internal Transport        Process Module  │
│  ┌──────────┐          ┌──────────┐             ┌──────────┐   │
│  │ Slot 1: W1│─────────→│ Robot:W1 │────────────→│ Chuck:W1 │   │
│  │ Slot 2: W2│          │          │             │          │   │
│  │ Slot 3: W3│          └──────────┘             └──────────┘   │
│  └──────────┘                                                    │
│                                                                  │
│  E90 tracks W1 through EVERY location transition                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Substrate Concept

### What is a Substrate?

A substrate is the individual unit being processed:

| Industry | Substrate Type | Description |
|----------|---------------|-------------|
| Semiconductor (300mm) | Wafer | 300mm silicon wafer |
| Semiconductor (200mm) | Wafer | 200mm silicon wafer |
| Display | Panel/Glass | LCD/OLED panel |
| Advanced Packaging | Strip | Lead frame strip |
| MEMS | Wafer | Specialized substrate |

### Substrate Identity

Each substrate has multiple identity levels:

| Identity Level | Description | Example |
|----------------|-------------|---------|
| **SubstID** | Equipment-assigned ID | "S001" (internal tracking) |
| **AcquiredID** | ID read from substrate (OCR/barcode) | "T7N12345.01" |
| **HostSubstID** | Host-assigned identifier | "LOT_A_W01" |
| **SourceLocation** | Where it came from | "FOUP001_SLOT01" |

### Substrate in Context

```
┌──────────────────────────────────────────────────────────────┐
│ Substrate Object (E39)                                        │
├──────────────────────────────────────────────────────────────┤
│  SubstID:          "W001"                                     │
│  AcquiredID:       "T7N12345.01" (from OCR reader)           │
│  LotID:            "LOT_A"                                    │
│  SubstSource:      CarrierID="FOUP001", SlotID=1             │
│  SubstDestination: CarrierID="FOUP001", SlotID=1             │
│  SubstLocID:       "PM1_CHUCK" (current location)            │
│  SubstProcState:   IN_PROCESS                                │
│  SubstTransState:  AT_DESTINATION                            │
│  SubstState:       AT_EQUIPMENT                              │
│  SubstUsage:       PRODUCT                                    │
│  SubstType:        WAFER                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Substrate State Models

### 1. Substrate Transport State Model

Tracks HOW the substrate is being moved:

```
┌───────────────────────────────────────────────────────────────┐
│              SUBSTRATE TRANSPORT STATES                         │
│                                                                │
│  ┌────────────────────┐                                       │
│  │  AT SOURCE          │  Substrate at its origin location     │
│  │  (In carrier slot)  │                                       │
│  └─────────┬──────────┘                                       │
│            │ Pick up / Move command                             │
│            ▼                                                    │
│  ┌────────────────────┐                                       │
│  │  IN TRANSIT         │  Substrate being moved (on robot)     │
│  │  (Being moved)     │                                        │
│  └─────────┬──────────┘                                       │
│            │ Place / Arrive at destination                      │
│            ▼                                                    │
│  ┌────────────────────┐                                       │
│  │  AT WORK            │  Substrate at processing location     │
│  │  (At process module)│                                       │
│  └─────────┬──────────┘                                       │
│            │ Pick up for return                                 │
│            ▼                                                    │
│  ┌────────────────────┐                                       │
│  │  IN TRANSIT         │  Substrate being returned             │
│  │  (Returning)       │                                        │
│  └─────────┬──────────┘                                       │
│            │ Placed back in carrier                             │
│            ▼                                                    │
│  ┌────────────────────┐                                       │
│  │  AT DESTINATION     │  Substrate at final location          │
│  │  (Back in carrier)  │                                       │
│  └────────────────────┘                                       │
└───────────────────────────────────────────────────────────────┘
```

### Transport State Values

| State | Value | Description |
|-------|-------|-------------|
| AT_SOURCE | 0 | At original location (carrier slot) |
| AT_WORK | 1 | At processing/measurement location |
| AT_DESTINATION | 2 | At final location (return slot) |
| IN_TRANSIT | 3 | Being moved between locations |

---

### 2. Substrate Processing State Model

Tracks WHAT has been done to the substrate:

```
┌───────────────────────────────────────────────────────────────┐
│            SUBSTRATE PROCESSING STATES                          │
│                                                                │
│  ┌────────────────────┐                                       │
│  │ NEEDS PROCESSING    │  Not yet processed by this equipment  │
│  └─────────┬──────────┘                                       │
│            │ Processing starts                                  │
│            ▼                                                    │
│  ┌────────────────────┐                                       │
│  │ IN PROCESS          │  Currently being processed            │
│  └─────────┬──────────┘                                       │
│            │                                                    │
│     ┌──────┼──────┬──────────┬──────────┐                    │
│     ▼      ▼      ▼          ▼          ▼                    │
│  ┌──────┐┌──────┐┌────────┐┌────────┐┌────────┐             │
│  │PROCE-││STOP- ││ABORT-  ││REJECT- ││LOST    │             │
│  │SSED  ││PED   ││ED      ││ED      ││        │             │
│  │(Done)││(Halt)││(Fail)  ││(Skip)  ││(Gone)  │             │
│  └──────┘└──────┘└────────┘└────────┘└────────┘             │
└───────────────────────────────────────────────────────────────┘
```

### Processing State Values

| State | Value | Description |
|-------|-------|-------------|
| **NEEDS PROCESSING** | 0 | Awaiting processing at this equipment |
| **IN PROCESS** | 1 | Processing is active |
| **PROCESSED** | 2 | Successfully processed |
| **ABORTED** | 3 | Processing aborted (abnormal) |
| **STOPPED** | 4 | Processing stopped (by command) |
| **REJECTED** | 5 | Substrate rejected (not processed) |
| **LOST** | 6 | Substrate location unknown |
| **SKIPPED** | 7 | Intentionally not processed |

---

### 3. Substrate Overall State Model

High-level view of substrate at equipment:

```
┌──────────────────────────────────────────────────────────────┐
│              SUBSTRATE OVERALL STATES                          │
│                                                               │
│  ┌─────────────────┐                                         │
│  │ NOT AT EQUIPMENT │  Substrate not tracked by this tool    │
│  └────────┬────────┘                                         │
│           │ Arrives (carrier opened, substrate instantiated)   │
│           ▼                                                    │
│  ┌─────────────────┐                                         │
│  │  AT EQUIPMENT    │  Substrate tracked by this equipment   │
│  │                  │                                         │
│  │  (SubstProcState │  Processing state tracked              │
│  │   SubstTransState│  Transport state tracked               │
│  │   SubstLocID)    │  Location tracked                      │
│  └────────┬────────┘                                         │
│           │ Departs (returned to carrier, carrier leaves)      │
│           ▼                                                    │
│  ┌─────────────────┐                                         │
│  │ NOT AT EQUIPMENT │  Substrate no longer tracked           │
│  └─────────────────┘                                         │
└──────────────────────────────────────────────────────────────┘
```

---

## Substrate Attributes

### Required Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| **SubstID** | ASCII | Unique substrate identifier (within equipment) |
| **SubstLocID** | ASCII | Current physical location ID |
| **SubstProcState** | U1 | Processing state value |
| **SubstTransState** | U1 | Transport state value |
| **SubstState** | U1 | Overall state (at equipment / not) |
| **SubstSource** | List | Origin (CarrierID + SlotID) |
| **SubstDestination** | List | Destination (CarrierID + SlotID) |

### Optional Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| AcquiredID | ASCII | ID read from substrate (OCR/barcode) |
| LotID | ASCII | Lot/batch identifier |
| SubstType | U1 | Wafer/panel/strip type |
| SubstUsage | U1 | Product/test/dummy/monitor |
| SubstMtrlStatus | U1 | Material quality status |
| BatchID | ASCII | Batch processing group |
| SubstHistory | List | Processing history log |
| SubstProcStateTime | ASCII | Timestamp of last proc state change |
| SubstLocStateTime | ASCII | Timestamp of last location change |

### Substrate Usage Types

| Value | Usage | Description |
|-------|-------|-------------|
| 0 | PRODUCT | Production substrate |
| 1 | TEST | Test/qualification substrate |
| 2 | DUMMY | Filler substrate (thermal mass) |
| 3 | MONITOR | Process monitor substrate |
| 4 | ENGINEERING | Engineering evaluation |
| 5 | EMPTY | Placeholder (no physical substrate) |

### Substrate Material Status

| Value | Status | Description |
|-------|--------|-------------|
| 0 | OK | Normal, no issues |
| 1 | SCRAPPED | Substrate is scrap |
| 2 | RETURNED | Returned unprocessed |
| 3 | SKIP | Skip this substrate |
| 4 | HOLD | Held for review |

---

## Substrate Location Tracking

### Location ID System

Every position where a substrate can exist has a unique Location ID:

```
Equipment Location Map:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Load Port 1 (LP1):
  LP1_SLOT01, LP1_SLOT02, ... LP1_SLOT25

Load Port 2 (LP2):
  LP2_SLOT01, LP2_SLOT02, ... LP2_SLOT25

Aligner:
  AL1_CHUCK

Pre-Aligner:
  PA1_CHUCK

Robot 1 (Transfer):
  ROBOT1_ARM1, ROBOT1_ARM2

Robot 2 (Process):
  ROBOT2_ARM1, ROBOT2_ARM2

Buffer:
  BUF_POS01, BUF_POS02, BUF_POS03

Process Module 1:
  PM1_CHUCK, PM1_LIFT

Process Module 2:
  PM2_CHUCK, PM2_LIFT

Measurement:
  MEAS1_STAGE
```

### Location Tracking Example

Complete substrate journey through equipment:

```
Time    Location          Transport State    Proc State
─────   ────────────────  ───────────────   ──────────────
10:00   LP1_SLOT01        AT_SOURCE          NEEDS_PROCESSING
10:01   ROBOT1_ARM1       IN_TRANSIT         NEEDS_PROCESSING
10:02   AL1_CHUCK         AT_WORK            NEEDS_PROCESSING
10:03   ROBOT1_ARM1       IN_TRANSIT         NEEDS_PROCESSING
10:04   PM1_CHUCK         AT_WORK            IN_PROCESS
10:15   PM1_CHUCK         AT_WORK            PROCESSED
10:16   ROBOT1_ARM2       IN_TRANSIT         PROCESSED
10:17   LP1_SLOT01        AT_DESTINATION     PROCESSED
```

### Location Change Events

Every location change generates an event:

| Event | Data Variables | Description |
|-------|---------------|-------------|
| SubstLocChanged | SubstID, FromLoc, ToLoc, Timestamp | Substrate moved |
| SubstEnteredModule | SubstID, ModuleID | Entered process module |
| SubstLeftModule | SubstID, ModuleID | Left process module |

---

## Substrate ID Management

### ID Acquisition Methods

| Method | Description | Typical Use |
|--------|-------------|-------------|
| **Host-Assigned** | Host provides ID via carrier/slot mapping | Most common |
| **OCR** | Optical Character Recognition of wafer markings | Wafer fab |
| **Barcode** | Read barcode on substrate | Assembly |
| **RFID** | Radio-frequency tag reading | Specialized |
| **Positional** | ID inferred from carrier + slot position | When no physical mark |

### ID Acquisition Flow

```
Host                                        Equipment
 │                                              │
 │  (Carrier FOUP001 opened at LP1)            │
 │                                              │
 │←── S6F11 (SubstInstantiated)               │  Equipment creates substrate objects
 │     SubstID="S001", Source=LP1_SLOT01        │  from slot map
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │   (Substrate moved to aligner/reader)        │
 │                                              │
 │←── S6F11 (SubstIDRead)                     │  OCR reads "T7N12345.01"
 │     SubstID="S001", AcquiredID="T7N12345"   │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │   Host verifies: "T7N12345" matches expected │
 │                                              │
 │─── S14F3 (SetAttr: HostSubstID) ───────────→│  Host assigns logical ID
 │←── S14F4 (OK) ─────────────────────────────│
```

### ID Verification States

```
┌────────────────┐
│ ID NOT READ     │  No ID acquisition attempted
└───────┬────────┘
        │ Read attempted
        ▼
┌────────────────┐     Success     ┌────────────────┐
│ ID READING     │────────────────→│ ID READ OK      │
└───────┬────────┘                 └────────────────┘
        │ Failure
        ▼
┌────────────────┐
│ ID READ FAILED  │  Cannot read / mismatch
└────────────────┘
```

---

## SECS-II Messages & Commands

### Primary Messages

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S14F1/F2 | GetAttr/Data | H→E | Read substrate attributes |
| S14F3/F4 | SetAttr/Data | H→E | Write substrate attributes |
| S6F11/F12 | Event Report/Ack | E→H | Substrate state/location events |
| S1F3/F4 | Status Request/Data | H→E | Query substrate status variables |
| S3F17/F18 | Carrier Action/Ack | H→E | Substrate-related carrier commands |

### Remote Commands for Substrate Operations

Via S2F41 or S2F49:

| Command | Parameters | Description |
|---------|-----------|-------------|
| SUBSTRATE_READ_ID | SubstID, LocationID | Request ID acquisition |
| SUBSTRATE_VERIFY_ID | SubstID, ExpectedID | Verify substrate identity |
| SUBSTRATE_SET_USAGE | SubstID, Usage | Change substrate usage type |
| SUBSTRATE_SET_STATE | SubstID, ProcState | Override processing state |
| SUBSTRATE_SET_DESTINATION | SubstID, DestCarrier, DestSlot | Set return location |

### Querying Substrate Information

**S14F1 — Get Substrate Attributes:**
```
{L:3
  <A "Substrate/S001">    ← OBJSPEC
  <A "Substrate">         ← OBJTYPE
  {L:5                     ← Attributes to read
    <A "SubstLocID">
    <A "SubstProcState">
    <A "SubstTransState">
    <A "AcquiredID">
    <A "LotID">
  }
}
```

**S14F2 — Response:**
```
{L:2
  {L:1
    {L:2
      <A "S001">
      {L:5
        {L:2 <A "SubstLocID"> <A "PM1_CHUCK">}
        {L:2 <A "SubstProcState"> <U1 1>}        ← IN_PROCESS
        {L:2 <A "SubstTransState"> <U1 1>}       ← AT_WORK
        {L:2 <A "AcquiredID"> <A "T7N12345">}
        {L:2 <A "LotID"> <A "LOT_A">}
      }
    }
  }
  {L:0}                   ← No errors
}
```

---

## Scenarios & Message Flows

### Scenario 1: Complete Substrate Lifecycle

```
Host                                        Equipment
 │                                              │
 │  ═══ SUBSTRATE INSTANTIATION ═══            │
 │                                              │
 │  (Carrier opened, slot map says slot 1 occupied)
 │                                              │
 │←── S6F11 (SubstInstantiated)               │
 │     SubstID="S001"                           │
 │     Source=FOUP001/Slot1                     │
 │     ProcState=NEEDS_PROCESSING              │
 │     TransState=AT_SOURCE                    │
 │     LocID=LP1_SLOT01                        │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ SUBSTRATE MOVEMENT TO PROCESS ═══      │
 │                                              │
 │←── S6F11 (SubstLocChanged)                 │
 │     SubstID="S001"                           │
 │     LocID=ROBOT1_ARM1                       │
 │     TransState=IN_TRANSIT                   │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │←── S6F11 (SubstLocChanged)                 │
 │     SubstID="S001"                           │
 │     LocID=PM1_CHUCK                         │
 │     TransState=AT_WORK                      │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ PROCESSING ═══                         │
 │                                              │
 │←── S6F11 (SubstProcStateChanged)            │
 │     SubstID="S001"                           │
 │     ProcState=IN_PROCESS                    │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │         ... processing occurs ...             │
 │                                              │
 │←── S6F11 (SubstProcStateChanged)            │
 │     SubstID="S001"                           │
 │     ProcState=PROCESSED                     │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ RETURN TO CARRIER ═══                  │
 │                                              │
 │←── S6F11 (SubstLocChanged)                 │
 │     SubstID="S001"                           │
 │     LocID=ROBOT1_ARM2                       │
 │     TransState=IN_TRANSIT                   │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │←── S6F11 (SubstLocChanged)                 │
 │     SubstID="S001"                           │
 │     LocID=LP1_SLOT01                        │
 │     TransState=AT_DESTINATION               │
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ═══ SUBSTRATE DE-INSTANTIATION ═══         │
 │                                              │
 │  (Carrier closed and departed)               │
 │                                              │
 │←── S6F11 (SubstDeinstantiated)             │
 │     SubstID="S001"                           │
 │─── S6F12 ───────────────────────────────────→│
```

### Scenario 2: Substrate Processing Abort

```
Host                                        Equipment
 │                                              │
 │  (S001 at PM1, processing)                  │
 │                                              │
 │←── S6F11 (SubstProcStateChanged)            │
 │     SubstID="S001"                           │
 │     ProcState=ABORTED                       │  Equipment error!
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │←── S5F1 (Alarm: Process Failure)            │  Alarm raised
 │─── S5F2 ───────────────────────────────────→│
 │                                              │
 │  (Equipment returns substrate to carrier)    │
 │                                              │
 │←── S6F11 (SubstLocChanged: PM1→Robot)       │
 │←── S6F11 (SubstLocChanged: Robot→LP1_SLOT01)│
 │     TransState=AT_DESTINATION               │
```

### Scenario 3: Substrate Re-routing

```
Host                                        Equipment
 │                                              │
 │  (S001 processed at PM1, needs measurement) │
 │                                              │
 │─── S14F3 (SetAttr: SubstDestination         │
 │           = MEAS1) ────────────────────────→│  Re-route to measurement
 │←── S14F4 (OK) ─────────────────────────────│
 │                                              │
 │  (Equipment routes to measurement instead    │
 │   of returning to carrier)                   │
 │                                              │
 │←── S6F11 (SubstLocChanged: MEAS1_STAGE)    │
```

---

## Collection Events for E90

| Event | Description | Key Data |
|-------|-------------|----------|
| SubstInstantiated | Substrate object created | SubstID, Source, ProcState |
| SubstDeinstantiated | Substrate object removed | SubstID, FinalState |
| SubstLocChanged | Location changed | SubstID, FromLoc, ToLoc |
| SubstProcStateChanged | Processing state changed | SubstID, OldState, NewState |
| SubstTransStateChanged | Transport state changed | SubstID, OldState, NewState |
| SubstIDRead | ID acquisition complete | SubstID, AcquiredID |
| SubstIDReadFailed | ID read failure | SubstID, ErrorInfo |
| SubstUsageChanged | Usage type changed | SubstID, OldUsage, NewUsage |
| SubstLost | Substrate location unknown | SubstID, LastKnownLoc |
| SubstFound | Lost substrate located | SubstID, FoundLoc |

---

## Relationship to Other Standards

### E90 Integration Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  E87 (CMS): Carrier arrives → Slot map → E90 instantiates      │
│                                              substrates          │
│                                                │                 │
│  E94 (CJM): Control Job says WHICH lot        │                 │
│              to process                        ▼                 │
│                                     E90 tracks EACH substrate    │
│                                                │                 │
│  E40 (PJM): Process Job processes    ←─────────┘                │
│              specific substrates                                 │
│                                                                  │
│  E116 (EPT): Tracks throughput using E90 events                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Substrate Count Tracking

| Source Standard | What it Tracks | Granularity |
|----------------|---------------|-------------|
| E87 | ContentMap (slots occupied) | Carrier level |
| E90 | Individual substrate identity and location | Substrate level |
| E40 | Which substrates assigned to process job | Job level |
| E94 | Which carriers/lots to process | Lot level |

---

## Implementation Guidelines

### Minimum Compliance

1. Track all substrates from instantiation to de-instantiation
2. Report location changes (SubstLocChanged events)
3. Report processing state changes
4. Support GetAttr on substrate objects (S14F1/F2)
5. Maintain accurate SubstLocID at all times
6. Report SubstInstantiated/SubstDeinstantiated events

### Equipment Location Map

Every equipment must define its complete location map:

```
Location Map Definition:
┌──────────────┬─────────────────────────────────────┐
│ Location ID   │ Description                          │
├──────────────┼─────────────────────────────────────┤
│ LP1_SLOT01   │ Load Port 1, Slot 1                  │
│ LP1_SLOT02   │ Load Port 1, Slot 2                  │
│ ...          │                                       │
│ ROBOT1_ARM1  │ Transfer Robot, Upper Arm            │
│ ROBOT1_ARM2  │ Transfer Robot, Lower Arm            │
│ ALIGN1       │ Pre-Aligner Station                  │
│ PM1_CHUCK    │ Process Module 1, Wafer Chuck        │
│ PM1_LIFT     │ Process Module 1, Lift Pins          │
│ PM2_CHUCK    │ Process Module 2, Wafer Chuck        │
│ BUF_POS01    │ Buffer Position 1                    │
│ MEAS_STAGE   │ Measurement Stage                    │
└──────────────┴─────────────────────────────────────┘
```

### Performance Considerations

| Aspect | Guideline |
|--------|-----------|
| Event generation rate | One event per location change (can be high-frequency) |
| Maximum substrates | Must handle all slots across all carriers simultaneously |
| Location update latency | Report within 100ms of physical move |
| History depth | Configurable — at minimum current state |
| Concurrent tracking | Support all processing modules simultaneously |

### Error Handling

| Condition | Response |
|-----------|----------|
| Substrate not found at expected location | Set ProcState=LOST, generate SubstLost event |
| Substrate found at unexpected location | Generate SubstFound event, update location |
| Substrate dropped during transfer | Alarm + LOST state |
| ID read failure | Report SubstIDReadFailed, proceed per host instruction |
| Carrier removed with substrates still inside equipment | Alarm + track remaining substrates |

### Status Variables for E90

| SVID | Name | Description |
|------|------|-------------|
| — | SubstCount | Total substrates tracked |
| — | SubstInProcessCount | Substrates currently processing |
| — | SubstLocList | List of all substrate locations |
| — | SubstProcStateList | List of all processing states |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E90-0710 Standard*
