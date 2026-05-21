# SEMI E116 — Equipment Performance Tracking Standard (EPT)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [EPT Concepts](#ept-concepts)
4. [EPT State Model](#ept-state-model)
5. [EPT Time Categories](#ept-time-categories)
6. [EPT Events & Transitions](#ept-events--transitions)
7. [EPT Metrics & Calculations](#ept-metrics--calculations)
8. [SECS-II Integration](#secs-ii-integration)
9. [Multi-Module Equipment](#multi-module-equipment)
10. [Scenarios & Examples](#scenarios--examples)
11. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E116** defines the **Equipment Performance Tracking (EPT)** standard, which provides a standardized method for categorizing equipment time into meaningful performance states. This enables consistent OEE (Overall Equipment Effectiveness) calculations across all equipment types.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E116 |
| Full Name | Equipment Performance Tracking |
| Abbreviation | EPT |
| Primary Purpose | Standardized equipment utilization and performance measurement |
| Dependencies | SEMI E5, E30, E10, E58, E79 |
| Related | E10 (Equipment Reliability), E79 (Statistical Process Control) |
| Key Output | Time categorization for OEE calculation |

---

## Purpose & Scope

### What E116 Defines

1. **EPT States** — Standardized time categories for equipment tracking
2. **State Model** — State machine for equipment performance states
3. **Transitions** — When and why state changes occur
4. **Metrics** — How to calculate OEE and related KPIs
5. **Events** — What triggers state transitions
6. **Reporting** — How to report EPT data to host

### Why EPT Matters

```
WITHOUT EPT:                              WITH EPT:
──────────────                            ──────────────
"Tool was running today"                  "Tool was PRODUCTIVE 72% of time"
"There was some downtime"                 "UNSCHEDULED DOWN: 45 min (pump fail)"
"We processed some wafers"                "STANDBY: 2.5 hrs (waiting for lots)"
"OEE? About 80%... maybe?"               "OEE = 72.3% (A=95%, P=82%, Q=93%)"
```

### E116 vs. E10

| Aspect | E10 (Reliability) | E116 (Performance) |
|--------|-------------------|---------------------|
| Focus | Equipment reliability states | Equipment performance/utilization |
| Granularity | 6 broad states | Detailed sub-states |
| Time basis | Calendar time | Operations time |
| Primary metric | MTBF, MTTR, Availability | OEE, Utilization |
| Users | Maintenance engineering | Production/manufacturing |

---

## EPT Concepts

### Core Principles

| Concept | Description |
|---------|-------------|
| **Equipment Time** | 100% of calendar time is accounted for |
| **Mutual Exclusivity** | Equipment is in exactly ONE EPT state at any moment |
| **Completeness** | All time must be categorized (no gaps) |
| **Module-Level** | Tracked per processing module (not just tool level) |
| **Event-Driven** | State changes triggered by defined events |
| **Host-Reportable** | EPT state available via standard SECS-II |

### Time Accounting

All time is accounted for — no gaps:

```
Total Calendar Time (24 hours)
├── NON-SCHEDULED TIME (factory not running)
│   └── Weekend, holiday, planned non-production
└── SCHEDULED TIME (factory operations)
    ├── UNSCHEDULED DOWN TIME (unplanned failures)
    │   └── Equipment failures, unexpected repairs
    ├── SCHEDULED DOWN TIME (planned maintenance)
    │   └── PM, calibration, planned upgrades
    ├── ENGINEERING TIME (engineering use)
    │   └── Qualifications, tests, recipe development
    ├── STANDBY TIME (ready but idle)
    │   └── Waiting for material, waiting for operator
    └── PRODUCTIVE TIME (processing or supporting processing)
        ├── Regular production run time
        ├── Setup/transition time between lots
        └── Measurement/inspection during production
```

---

## EPT State Model

### High-Level State Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         EQUIPMENT TIME                                    │
│                                                                          │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────────────┐  │
│  │ NON-SCHEDULED   │  │ UNSCHEDULED DOWN │  │  SCHEDULED DOWN       │  │
│  │                 │  │                  │  │                       │  │
│  │ Factory closed  │  │ Breakdown        │  │  Planned maintenance  │  │
│  │ No production   │  │ Unexpected fail  │  │  Calibration          │  │
│  └─────────────────┘  └──────────────────┘  └───────────────────────┘  │
│                                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────────────────────┐  │
│  │ ENGINEERING     │  │              OPERATIONS TIME                   │  │
│  │                 │  │                                               │  │
│  │ Qualification   │  │  ┌─────────────┐     ┌─────────────────────┐ │  │
│  │ Recipe dev     │  │  │  STANDBY    │     │    PRODUCTIVE        │ │  │
│  │ Testing        │  │  │  (Idle)     │     │    (Working)         │ │  │
│  │                │  │  │             │     │                      │ │  │
│  └─────────────────┘  │  │ No material │     │ Processing material │ │  │
│                        │  │ Waiting     │     │ Setup/transition    │ │  │
│                        │  └─────────────┘     └─────────────────────┘ │  │
│                        └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Detailed State Definitions

#### 1. NON-SCHEDULED (NS)

Equipment not expected to produce:

| Sub-State | Description |
|-----------|-------------|
| NS_WeekendHoliday | Weekend/holiday — no production |
| NS_NonProductionShift | Non-production shift |
| NS_PlannedShutdown | Planned factory shutdown |
| NS_Install | Initial equipment installation |

#### 2. UNSCHEDULED DOWN (UD)

Equipment failed unexpectedly:

| Sub-State | Description |
|-----------|-------------|
| UD_Breakdown | Equipment hardware failure |
| UD_Facility | Facility problem (power, gas, water) |
| UD_OutOfSpec | Process out of specification |
| UD_AssistRequired | Needs intervention (error recovery) |
| UD_Software | Software crash or hang |
| UD_Material_Jam | Material stuck/jammed |

#### 3. SCHEDULED DOWN (SD)

Planned non-production time:

| Sub-State | Description |
|-----------|-------------|
| SD_PreventiveMaint | Scheduled PM |
| SD_Calibration | Sensor/system calibration |
| SD_PlannedUpgrade | Hardware/software upgrade |
| SD_PartChange | Scheduled consumable change |
| SD_Setup | Major setup/changeover |
| SD_Clean | Scheduled chamber clean |

#### 4. ENGINEERING (ENG)

Equipment used for engineering purposes:

| Sub-State | Description |
|-----------|-------------|
| ENG_Qualification | Equipment qualification runs |
| ENG_RecipeDev | Recipe development/tuning |
| ENG_Experiment | Engineering experiments |
| ENG_Test | Equipment testing |
| ENG_ProcessDev | Process development |

#### 5. STANDBY (SB)

Equipment ready but not processing:

| Sub-State | Description |
|-----------|-------------|
| SB_NoMaterial | Idle — waiting for material (carrier) |
| SB_NoOperator | Waiting for operator action |
| SB_NoHost | Waiting for host instruction |
| SB_NoSupport | Waiting for support equipment |
| SB_ScheduledIdle | Intentionally idle (schedule gap) |
| SB_Warming | Warming up (ready soon) |
| SB_MtlRemoval | Waiting for output carrier removal |

#### 6. PRODUCTIVE (PR)

Equipment actively supporting production:

| Sub-State | Description |
|-----------|-------------|
| PR_Regular | Normal production processing |
| PR_Setup | Lot-to-lot setup/transition |
| PR_Rework | Reprocessing material |
| PR_Test | Production test wafers |
| PR_Engineering | Eng wafers during production |
| PR_Loading | Material being loaded |
| PR_Unloading | Material being unloaded |

---

## EPT Time Categories

### Time Category Hierarchy

```
Total Time (100%)
│
├── Non-Scheduled Time (NS)
│
└── Scheduled Time
    │
    ├── Down Time
    │   ├── Unscheduled Down (UD)
    │   └── Scheduled Down (SD)
    │
    └── Uptime
        ├── Engineering Time (ENG)
        │
        └── Operations Time
            ├── Standby Time (SB)
            └── Productive Time (PR)
```

### Time Category Calculations

```
Scheduled Time = Total Time - Non-Scheduled Time
Down Time = Unscheduled Down + Scheduled Down
Uptime = Scheduled Time - Down Time
Operations Time = Uptime - Engineering Time
Productive Time = Operations Time - Standby Time
```

---

## EPT Events & Transitions

### Transition Triggers

| From State | To State | Trigger Event |
|-----------|----------|---------------|
| Any → | NS | Factory schedule (shift end) |
| NS → | SB | Factory schedule (shift start) |
| SB → | PR | Material arrives, processing starts |
| PR → | SB | Processing done, no more material |
| Any → | UD | Equipment alarm (failure) |
| UD → | SB | Repair complete, equipment ready |
| SB → | SD | PM schedule begins |
| SD → | SB | PM complete |
| SB → | ENG | Engineering qualification starts |
| ENG → | SB | Engineering activity ends |
| PR → | UD | Failure during processing |
| PR → | SB | Between lots (no next lot ready) |
| SB → | PR | Next lot available, processing begins |

### Transition Event Mapping to GEM Events

| EPT Transition | GEM Event (S6F11) | Description |
|---------------|-------------------|-------------|
| → PR_Regular | ProcessingStarted | Processing begins |
| PR → SB | ProcessingCompleted + No pending | Done, going idle |
| → UD | AlarmSet (equipment alarm) | Failure detected |
| UD → SB | AlarmCleared + repair done | Equipment recovered |
| → SD | MaintenanceStarted | PM begins |
| SD → SB | MaintenanceCompleted | PM finished |
| → PR_Setup | LotStartSetup | Lot transition |
| PR_Setup → PR_Regular | SetupComplete | Ready to process |

### State Transition Diagram (Operations Focus)

```
                         Alarm/Failure
                    ┌───────────────────────────────────────┐
                    │                                        │
                    ▼                                        │
┌──────────────┐  Repair    ┌──────────────┐  Material    ┌─┴────────────┐
│ UNSCHEDULED  │──────────→│   STANDBY    │─────────────→│  PRODUCTIVE   │
│    DOWN      │           │   (Idle)     │             │  (Working)    │
│              │←──────────│              │←────────────│              │
└──────────────┘  Failure   └──────┬───────┘  Complete   └──────────────┘
                                   │    ↑
                              PM   │    │ PM Done
                              Start│    │
                                   ▼    │
                            ┌──────────────┐
                            │  SCHEDULED   │
                            │    DOWN      │
                            └──────────────┘
```

---

## EPT Metrics & Calculations

### OEE (Overall Equipment Effectiveness)

The primary metric derived from EPT data:

```
OEE = Availability × Performance × Quality

Where:
  Availability = (Scheduled Time - Down Time) / Scheduled Time
  Performance  = (Actual Output × Ideal Cycle Time) / Operations Time
  Quality      = Good Units / Total Units Produced
```

### Key Performance Indicators (KPIs)

| KPI | Formula | Target |
|-----|---------|--------|
| **OEE** | A × P × Q | >85% (world class) |
| **Availability** | Uptime / Scheduled Time | >95% |
| **Utilization** | Productive Time / Operations Time | >80% |
| **MTBF** | Operations Time / Number of Failures | Maximize |
| **MTTR** | Total Repair Time / Number of Repairs | Minimize |
| **Throughput** | Units Processed / Productive Time | Maximize |
| **Standby Ratio** | Standby Time / Operations Time | <20% |

### Example Calculation

```
24-hour period:
─────────────────────────────────────────────────────────────
Non-Scheduled:           0 hours  (24/7 factory)
Scheduled Down (PM):     1.5 hours
Unscheduled Down:        0.5 hours (pump alarm, 30 min repair)
Engineering:             0 hours
Standby:                 4 hours (waiting for lots)
Productive:              18 hours

Calculations:
─────────────────────────────────────────────────────────────
Scheduled Time      = 24 - 0 = 24 hours
Down Time           = 1.5 + 0.5 = 2 hours
Uptime              = 24 - 2 = 22 hours
Operations Time     = 22 - 0 = 22 hours
Productive Time     = 18 hours

Availability = (24 - 2) / 24 = 91.7%
Utilization  = 18 / 22 = 81.8%

If ideal cycle = 2 min/wafer, actual output = 480 wafers:
Performance  = (480 × 2min) / (18hrs × 60min) = 960 / 1080 = 88.9%

If 470 good wafers out of 480:
Quality      = 470 / 480 = 97.9%

OEE = 91.7% × 88.9% × 97.9% = 79.8%
```

### EPT Time Tracking Data Structure

```
EPT Record:
{
  ModuleID: "PM1",
  StateCode: "PR_Regular",
  StartTime: "20260521100000",
  EndTime:   "20260521101500",
  Duration:  900 seconds,
  SubstCount: 5,
  ContextData: {
    LotID: "LOT_A",
    RecipeID: "ETCH_01",
    CarrierID: "FOUP001"
  }
}
```

---

## SECS-II Integration

### EPT Status Variables

| SVID | Name | Type | Description |
|------|------|------|-------------|
| — | EPTState | U1/A | Current EPT state code |
| — | EPTSubState | A | Current sub-state |
| — | EPTStateStartTime | A | When current state began |
| — | EPTStateDuration | U4 | Seconds in current state |
| — | EPTModuleID | A | Module being tracked |

### EPT Collection Events

| Event | Description | DVALs |
|-------|-------------|-------|
| EPTStateChanged | EPT state transition | ModuleID, OldState, NewState, Timestamp |
| EPTSubStateChanged | Sub-state transition | ModuleID, OldSubState, NewSubState |
| EPTPeriodSummary | Periodic summary report | ModuleID, Period, StateTotals |

### EPT Data in Event Reports

Configure via standard GEM event reporting (S2F33/35/37):

```
Report Definition:
  RPTID: 1001
  VIDs: [EPTModuleID, EPTState, EPTSubState, EPTStateStartTime]

Link to Event:
  CEID: EPTStateChanged → RPTID: 1001

Equipment sends S6F11 on each EPT state change:
  {L:3
    <U4 DATAID>
    <U4 CEID_EPTStateChanged>
    {L:1
      {L:2
        <U4 1001>        ← RPTID
        {L:4
          <A "PM1">      ← ModuleID
          <A "PR">       ← New State
          <A "PR_Regular">← Sub-state
          <A "20260521143000">← Start time
        }
      }
    }
  }
```

---

## Multi-Module Equipment

### Per-Module Tracking

Each processing module has its own EPT state:

```
Equipment: CLUSTER_TOOL_01
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Module: PM1          Module: PM2          Module: PM3          │
│  State: PRODUCTIVE    State: STANDBY       State: SCHED_DOWN    │
│  Sub: PR_Regular      Sub: SB_NoMaterial   Sub: SD_Clean        │
│  Since: 10:30:00      Since: 10:45:00      Since: 09:00:00     │
│                                                                  │
│  Module: TM1 (Transfer)                                         │
│  State: PRODUCTIVE                                               │
│  Sub: PR_Regular                                                 │
│  Since: 10:30:00                                                │
│                                                                  │
│  Equipment Level:                                                │
│  State: PRODUCTIVE (at least one module productive)              │
└─────────────────────────────────────────────────────────────────┘
```

### Equipment-Level vs. Module-Level

| Level | Rule | Example |
|-------|------|---------|
| Equipment | PRODUCTIVE if ANY module is productive | PM1 working = Tool productive |
| Equipment | DOWN if ALL modules are down | All PMs failed = Tool down |
| Equipment | STANDBY if no module is productive AND no module is down | All idle = Tool standby |
| Module | Independent state per module | PM1 can be down while PM2 processes |

### Bottleneck Identification

```
Time Period: 10:00 - 18:00 (8 hours)

Module    Productive  Standby  Down   Utilization
────────  ──────────  ───────  ────   ───────────
PM1       6.5h        1.0h     0.5h   81.3%
PM2       5.0h        2.5h     0.5h   62.5%      ← Bottleneck (low util)
PM3       7.0h        0.5h     0.5h   87.5%
Transfer  7.5h        0.5h     0.0h   93.8%

Analysis: PM2 has excessive standby → investigate material flow
```

---

## Scenarios & Examples

### Scenario 1: Normal Production Day

```
Time    Event                           EPT State          Sub-State
─────   ─────────────────────────       ──────────────    ─────────────
06:00   Shift start                     STANDBY           SB_NoMaterial
06:15   Carrier arrives LP1             PRODUCTIVE        PR_Loading
06:17   Processing starts               PRODUCTIVE        PR_Regular
08:00   Lot complete, no next lot       STANDBY           SB_NoMaterial
08:05   New carrier arrives             PRODUCTIVE        PR_Setup
08:10   Processing starts               PRODUCTIVE        PR_Regular
10:30   Pump alarm!                     UNSCHEDULED DOWN  UD_Breakdown
11:00   Repair complete                 STANDBY           SB_NoMaterial
11:02   Resume processing               PRODUCTIVE        PR_Regular
14:00   Scheduled PM begins             SCHEDULED DOWN    SD_PreventiveMaint
15:30   PM complete                     STANDBY           SB_NoMaterial
15:35   Material available              PRODUCTIVE        PR_Regular
22:00   Last lot done                   STANDBY           SB_NoMaterial
```

### Scenario 2: State Accounting

```
Summary for 06:00-22:00 (16 hours):

┌─────────────────────────────────────────────────────────────────┐
│  PRODUCTIVE:        12h 20min (77.1%)                            │
│  ├── PR_Regular:    11h 45min                                    │
│  ├── PR_Setup:      15min                                        │
│  └── PR_Loading:    20min                                        │
│                                                                  │
│  STANDBY:           1h 50min (11.5%)                             │
│  └── SB_NoMaterial: 1h 50min                                     │
│                                                                  │
│  UNSCHEDULED DOWN:  30min (3.1%)                                 │
│  └── UD_Breakdown:  30min                                        │
│                                                                  │
│  SCHEDULED DOWN:    1h 30min (9.4%)                              │
│  └── SD_PM:         1h 30min                                     │
│                                                                  │
│  Total:             16h 00min (100%)                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Guidelines

### Data Collection Requirements

| Requirement | Description |
|------------|-------------|
| Time resolution | Minimum 1-second granularity |
| State persistence | Survive power cycle (log to disk) |
| History depth | Minimum 7 days online, archive longer |
| Reporting interval | Real-time events + periodic summaries |
| Clock accuracy | Synchronized with factory time (E30 clock) |

### State Determination Logic

```pseudocode
function determineEPTState(equipment):
    if not isScheduledProduction():
        return NON_SCHEDULED
    
    if hasActiveAlarm() and isEquipmentDown():
        return UNSCHEDULED_DOWN
    
    if isInPlannedMaintenance():
        return SCHEDULED_DOWN
    
    if isEngineeringMode():
        return ENGINEERING
    
    if isProcessingMaterial():
        return PRODUCTIVE
    
    // None of the above — must be STANDBY
    return STANDBY
```

### Integration with Existing GEM

EPT is built on existing GEM infrastructure:

| GEM Feature | EPT Usage |
|-------------|-----------|
| Collection Events | Report EPT state changes |
| Status Variables | Expose current EPT state |
| Alarms | Trigger UD transitions |
| Equipment Constants | EPT configuration parameters |
| Remote Commands | Override EPT state (engineering) |
| Clock (S2F31) | Timestamp EPT transitions |

### Best Practices

1. **No gaps** — Every second must be in exactly one state
2. **Conservative classification** — When in doubt, use STANDBY (not PRODUCTIVE)
3. **Root cause sub-states** — Always specify meaningful sub-state for diagnostics
4. **Multi-module independence** — Don't average across modules; track each separately
5. **Automatic transitions** — Minimize manual state overrides
6. **Validate data** — Ensure state transitions are logical (no impossible sequences)
7. **Correlate with production** — Link EPT data to lots/carriers for context

### Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| EPT_ENABLED | Enable/disable EPT tracking | TRUE |
| EPT_SUMMARY_PERIOD | Periodic summary interval | 3600 sec (1 hour) |
| EPT_STANDBY_THRESHOLD | Min idle time to count as standby | 60 sec |
| EPT_MODULES_TRACKED | Which modules to track | All process modules |
| EPT_HISTORY_DEPTH | Days of online history | 7 days |
| EPT_AUTO_NS | Auto-detect non-scheduled time | TRUE |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E116-0709 Standard*
