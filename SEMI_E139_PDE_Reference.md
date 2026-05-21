# SEMI E139 — Recipe and Parameter Management (PDE)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [PDE Concepts](#pde-concepts)
4. [Recipe Structure](#recipe-structure)
5. [Parameter Definition Elements](#parameter-definition-elements)
6. [Recipe Types & Hierarchy](#recipe-types--hierarchy)
7. [Recipe Namespace](#recipe-namespace)
8. [SECS-II Messages (Stream 19)](#secs-ii-messages-stream-19)
9. [Parameter Validation](#parameter-validation)
10. [Scenarios & Message Flows](#scenarios--message-flows)
11. [Relationship to Other Standards](#relationship-to-other-standards)
12. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E139** defines the **Recipe and Parameter Management** standard using **Process Definition Elements (PDE)**. It provides a structured, host-readable format for recipes and their parameters, enabling the host to understand, validate, modify, and create recipes programmatically.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E139 |
| Full Name | Specification for Recipe and Parameter Management |
| Abbreviation | PDE (Process Definition Elements) |
| Primary Purpose | Structured recipe/parameter representation |
| Dependencies | SEMI E5, E30, E39 |
| Related | E40 (PJM), E170/E171 (SRO) |
| Messages | Stream 19 (S19Fx) |

---

## Purpose & Scope

### Problem E139 Solves

Traditional recipe management (E30/S7) treats recipes as opaque blobs:

```
BEFORE E139 (S7F3 — Unformatted PP):
┌──────────────────────────────────────────────────┐
│  PPID: "ETCH_01"                                  │
│  PPBODY: [binary blob — host cannot read/modify] │
│                                                   │
│  Host: "What parameters are in this recipe?"     │
│  Equipment: "¯\_(ツ)_/¯ It's just bytes"         │
└──────────────────────────────────────────────────┘

AFTER E139 (Structured PDE):
┌──────────────────────────────────────────────────┐
│  Recipe: "ETCH_01"                                │
│  Parameters:                                      │
│    Temperature: 350°C (range: 200-500°C)         │
│    Pressure: 100 mTorr (range: 10-500 mTorr)    │
│    RF_Power: 500W (range: 0-2000W)               │
│    Duration: 180 sec (range: 10-600 sec)         │
│    Gas_Flow_CF4: 50 sccm (range: 0-200 sccm)    │
│                                                   │
│  Host: "I'll change Temperature to 375°C"        │
│  Equipment: "Validated ✓ — within range"          │
└──────────────────────────────────────────────────┘
```

### What E139 Defines

1. **Recipe structure** — How recipes are organized internally
2. **Parameter definitions** — Metadata about each parameter (type, range, units)
3. **Parameter values** — Current settings within a recipe
4. **Namespace** — How recipes are organized and named
5. **Validation rules** — How to validate parameter values
6. **Host access** — Standard way to read/write recipe parameters
7. **Recipe creation** — How host can create new recipes

### What E139 Does NOT Define

- Recipe execution logic (that's the equipment's internal concern)
- Process results (that's data collection via E30)
- Recipe versioning/history (that's E170/E171)
- Recipe approval workflow (that's MES/host)

---

## PDE Concepts

### Core Terminology

| Term | Description |
|------|-------------|
| **Recipe** | A complete set of process instructions |
| **Parameter** | A single configurable value within a recipe |
| **PDE** | Process Definition Element — a building block |
| **Step** | A recipe segment executed in sequence |
| **Parameter Definition** | Metadata describing a parameter's constraints |
| **Parameter Value** | The actual setting of a parameter |
| **Recipe Body** | The complete structured content |
| **Recipe Namespace** | Hierarchical organization of recipes |

### PDE Element Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    RECIPE (Collection of PDEs)                    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ PARAMETER DEFINITION ELEMENTS                             │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │   │
│  │  │ParamDef │  │ParamDef │  │ParamDef │  │ParamDef │    │   │
│  │  │Temp     │  │Pressure │  │Power    │  │Time     │    │   │
│  │  │F4       │  │F4       │  │U4       │  │U4       │    │   │
│  │  │200-500  │  │10-500   │  │0-2000   │  │10-600   │    │   │
│  │  │°C       │  │mTorr    │  │Watts    │  │seconds  │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ PARAMETER VALUE ELEMENTS (Current Recipe Settings)        │   │
│  │  Temp=350°C, Pressure=100mTorr, Power=500W, Time=180sec │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ STEP STRUCTURE                                            │   │
│  │  Step 1: Stabilize (Temp=350, Pressure=100)              │   │
│  │  Step 2: Etch (Power=500, Gas=50, Time=180)              │   │
│  │  Step 3: Purge (Gas=0, Time=30)                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Recipe Structure

### Recipe Organization

A recipe in E139 is organized hierarchically:

```
Recipe: "ETCH_OXIDE_V2"
│
├── Header
│   ├── RecipeID: "ETCH_OXIDE_V2"
│   ├── Description: "Oxide etch — standard"
│   ├── Version: "2.0"
│   ├── Author: "Process Engineer"
│   ├── CreateDate: "20260401"
│   ├── ModifyDate: "20260515"
│   └── RecipeType: "Process"
│
├── Global Parameters (apply to entire recipe)
│   ├── WaferSize: 300mm
│   ├── ChuckTemp: 20°C
│   └── BasePressure: 1 mTorr
│
├── Step 1: "Stabilize"
│   ├── Duration: 30 sec
│   ├── ChuckTemp: 350°C
│   ├── ChamberPressure: 100 mTorr
│   └── Ar_Flow: 100 sccm
│
├── Step 2: "Main Etch"
│   ├── Duration: 180 sec
│   ├── RF_Power: 500 W
│   ├── CF4_Flow: 50 sccm
│   ├── O2_Flow: 10 sccm
│   └── EndpointMode: "Optical"
│
├── Step 3: "Over Etch"
│   ├── Duration: 45 sec
│   ├── RF_Power: 300 W
│   └── CF4_Flow: 30 sccm
│
└── Step 4: "Purge"
    ├── Duration: 15 sec
    ├── RF_Power: 0 W
    └── N2_Flow: 200 sccm
```

### Recipe Body Format (SECS-II)

```
{L:n                              ← Recipe body
  {L:2
    <A "RecipeID">
    <A "ETCH_OXIDE_V2">
  }
  {L:2
    <A "Description">
    <A "Oxide etch standard">
  }
  {L:2
    <A "StepCount">
    <U2 4>
  }
  {L:2                            ← Steps array
    <A "Steps">
    {L:4                          ← 4 steps
      {L:step1_params}
      {L:step2_params}
      {L:step3_params}
      {L:step4_params}
    }
  }
}
```

---

## Parameter Definition Elements

### Parameter Definition Structure

Each parameter has a definition that describes its constraints:

| Property | Type | Description |
|----------|------|-------------|
| **ParamName** | ASCII | Unique parameter name |
| **ParamID** | U4 | Numeric identifier |
| **DataType** | U1 | SECS-II format code (F4, U4, A, etc.) |
| **Units** | ASCII | Engineering units |
| **MinValue** | (varies) | Minimum allowed value |
| **MaxValue** | (varies) | Maximum allowed value |
| **DefaultValue** | (varies) | Default/initial value |
| **Resolution** | (varies) | Smallest increment |
| **Access** | U1 | Read-only, Read-write, Tunable |
| **Description** | ASCII | Human-readable description |
| **StepScope** | U1 | Global, Per-step, or Both |
| **Enumeration** | List | Allowed discrete values (if applicable) |

### Parameter Access Levels

| Level | Value | Description |
|-------|-------|-------------|
| Read-Only | 0 | Cannot be modified (fixed by recipe author) |
| Read-Write | 1 | Can be modified by host |
| Tunable | 2 | Can be modified at runtime (recipe override) |
| Hidden | 3 | Not visible to host |

### Parameter Step Scope

| Scope | Value | Description |
|-------|-------|-------------|
| Global | 0 | Applies to entire recipe (all steps) |
| Per-Step | 1 | Has independent value per step |
| Both | 2 | Has global default + per-step override |

### Example Parameter Definitions

```
Parameter Definition List:
┌────────────────────────────────────────────────────────────────────────────┐
│ Name          │ Type │ Units  │ Min    │ Max    │ Default │ Scope │ Access │
├────────────────┼──────┼────────┼────────┼────────┼─────────┼───────┼────────┤
│ Temperature   │ F4   │ °C     │ 20.0   │ 500.0  │ 350.0   │ Step  │ RW     │
│ Pressure      │ F4   │ mTorr  │ 0.1    │ 1000.0 │ 100.0   │ Step  │ RW     │
│ RF_Power      │ U4   │ W      │ 0      │ 2000   │ 500     │ Step  │ RW     │
│ Duration      │ U4   │ sec    │ 1      │ 3600   │ 180     │ Step  │ RW     │
│ CF4_Flow      │ F4   │ sccm   │ 0.0    │ 200.0  │ 50.0    │ Step  │ RW     │
│ O2_Flow       │ F4   │ sccm   │ 0.0    │ 100.0  │ 10.0    │ Step  │ RW     │
│ EndpointMode  │ A    │ —      │ —      │ —      │ "Time"  │ Step  │ RW     │
│ WaferSize     │ U2   │ mm     │ 150    │ 450    │ 300     │ Global│ RO     │
│ ChuckType     │ A    │ —      │ —      │ —      │ "ESC"   │ Global│ RO     │
└────────────────┴──────┴────────┴────────┴────────┴─────────┴───────┴────────┘

Enumerated parameter:
  EndpointMode: ["Time", "Optical", "Interferometry", "Mass_Spec"]
```

### SECS-II Format for Parameter Definition

```
{L:11
  <A "Temperature">           ← ParamName
  <U4 1001>                   ← ParamID
  <U1 0x24>                   ← DataType (F4)
  <A "°C">                    ← Units
  <F4 20.0>                   ← MinValue
  <F4 500.0>                  ← MaxValue
  <F4 350.0>                  ← DefaultValue
  <F4 0.1>                    ← Resolution
  <U1 1>                      ← Access (RW)
  <U1 1>                      ← StepScope (Per-Step)
  <A "Chuck temperature setpoint">  ← Description
}
```

---

## Recipe Types & Hierarchy

### Recipe Types

| Type | Description | Example |
|------|-------------|---------|
| **Process Recipe** | Complete processing instructions | "ETCH_OXIDE_V2" |
| **Setup Recipe** | Equipment preparation | "CHAMBER_CONDITION_01" |
| **Clean Recipe** | Chamber cleaning | "WAFERLESS_CLEAN" |
| **Calibration Recipe** | Sensor calibration | "MFC_CAL_CF4" |
| **Template Recipe** | Base for creating new recipes | "ETCH_TEMPLATE" |
| **Golden Recipe** | Approved reference (locked) | "ETCH_OXIDE_GOLDEN" |

### Recipe Hierarchy

```
Template Recipe: "ETCH_TEMPLATE"
│   (Defines structure — all parameters)
│
├── Golden Recipe: "ETCH_OXIDE_GOLDEN"
│   (Qualified — locked, reference)
│
├── Production Recipe: "ETCH_OXIDE_V2"
│   (Active production recipe)
│
├── Tuned Recipe: "ETCH_OXIDE_V2_LOT_A"
│   (Per-lot adjustments from production recipe)
│
└── Development Recipe: "ETCH_OXIDE_DEV_03"
    (Engineering — not for production)
```

### Recipe Inheritance

```
┌──────────────────────────────────────────────┐
│ Template: "ETCH_TEMPLATE"                     │
│   Temperature: [defined, range 20-500]       │
│   Pressure: [defined, range 0-1000]          │
│   RF_Power: [defined, range 0-2000]          │
│   Duration: [defined, range 1-3600]          │
└──────────────────────┬───────────────────────┘
                       │ Inherit
                       ▼
┌──────────────────────────────────────────────┐
│ Recipe: "ETCH_OXIDE_V2"                       │
│   Temperature: 350°C                          │
│   Pressure: 100 mTorr                        │
│   RF_Power: 500 W                            │
│   Duration: 180 sec                          │
└──────────────────────┬───────────────────────┘
                       │ Tune (per-lot override)
                       ▼
┌──────────────────────────────────────────────┐
│ Tuned: "ETCH_OXIDE_V2" + overrides           │
│   Temperature: 360°C   ← Changed for lot    │
│   Pressure: 100 mTorr  ← Same               │
│   RF_Power: 520 W      ← Changed            │
│   Duration: 180 sec    ← Same               │
└──────────────────────────────────────────────┘
```

---

## Recipe Namespace

### Namespace Organization

Recipes are organized in a hierarchical namespace (like a filesystem):

```
/
├── Process/
│   ├── Etch/
│   │   ├── ETCH_OXIDE_V1
│   │   ├── ETCH_OXIDE_V2
│   │   ├── ETCH_NITRIDE_V1
│   │   └── ETCH_POLY_V3
│   ├── Deposition/
│   │   ├── CVD_OXIDE_01
│   │   └── CVD_NITRIDE_02
│   └── Clean/
│       ├── WET_CLEAN_STD
│       └── DRY_CLEAN_01
├── Setup/
│   ├── CHAMBER_INIT
│   └── ROBOT_HOME
├── Calibration/
│   ├── MFC_CAL_CF4
│   └── PRESSURE_CAL
└── Template/
    ├── ETCH_TEMPLATE
    └── DEP_TEMPLATE
```

### Namespace Operations

| Operation | Description | Message |
|-----------|-------------|---------|
| List | List recipes in a folder | S19F1/F2 |
| Create | Create new recipe | S19F3/F4 |
| Delete | Remove recipe | S19F5/F6 |
| Read | Get recipe content | S19F7/F8 |
| Write | Update recipe content | S19F9/F10 |
| Copy | Copy recipe | S19F11/F12 |
| Rename | Rename/move recipe | S19F13/F14 |
| GetDefs | Get parameter definitions | S19F15/F16 |

---

## SECS-II Messages (Stream 19)

### Stream 19 Message Summary

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S19F1/F2 | Recipe Directory Request/Data | H→E | List recipes in namespace |
| S19F3/F4 | Recipe Create Request/Ack | H→E | Create new recipe |
| S19F5/F6 | Recipe Delete Request/Ack | H→E | Delete recipe |
| S19F7/F8 | Recipe Retrieve Request/Data | H→E | Download recipe content |
| S19F9/F10 | Recipe Store Request/Ack | H→E | Upload recipe content |
| S19F11/F12 | Recipe Copy Request/Ack | H→E | Copy recipe |
| S19F13/F14 | Recipe Rename Request/Ack | H→E | Rename recipe |
| S19F15/F16 | Get Definitions Request/Data | H→E | Get parameter metadata |
| S19F17/F18 | Get Values Request/Data | H→E | Get parameter values only |
| S19F19/F20 | Set Values Request/Ack | H→E | Set parameter values only |

### S19F1/F2 — Recipe Directory

**S19F1 Request:**
```
{L:2
  <A RecipeNamespace>     ← Folder path (e.g., "/Process/Etch/")
  <U1 RecipeFilter>      ← Filter type (0=all, 1=by type)
}
```

**S19F2 Response:**
```
{L:n
  {L:4
    <A RecipeID>           ← Recipe name
    <A RecipeType>         ← Type (Process, Clean, etc.)
    <A LastModified>       ← Timestamp
    <A Description>        ← Brief description
  }
}
```

### S19F7/F8 — Recipe Retrieve

**S19F7 Request:**
```
{L:2
  <A RecipeID>            ← Full recipe path/name
  <U1 RetrieveScope>     ← 0=full, 1=header, 2=params only
}
```

**S19F8 Response:**
```
{L:4
  <A RecipeID>
  <A RecipeType>
  {L:m                    ← Parameter definitions
    {L:11
      <A ParamName>
      <U4 ParamID>
      <U1 DataType>
      <A Units>
      <V MinValue>
      <V MaxValue>
      <V DefaultValue>
      <V Resolution>
      <U1 Access>
      <U1 StepScope>
      <A Description>
    }
  }
  {L:s                    ← Steps with values
    {L:2
      <A StepName>
      {L:p               ← Parameter values for this step
        {L:2
          <A ParamName>
          <V ParamValue>
        }
      }
    }
  }
}
```

### S19F19/F20 — Set Parameter Values

**S19F19 Request:**
```
{L:3
  <A RecipeID>            ← Target recipe
  <U2 StepNumber>        ← Step (0=global)
  {L:n                    ← Parameters to change
    {L:2
      <A ParamName>
      <V NewValue>
    }
  }
}
```

**S19F20 Response:**
```
{L:2
  <U1 RCPACK>            ← 0=success
  {L:n                    ← Errors per parameter
    {L:3
      <A ParamName>
      <U1 ErrCode>
      <A ErrText>
    }
  }
}
```

**RCPACK Codes:**

| Code | Meaning |
|------|---------|
| 0 | Success — all parameters updated |
| 1 | Recipe not found |
| 2 | Recipe locked (read-only) |
| 3 | Parameter validation failed (see error list) |
| 4 | Step number invalid |
| 5 | Recipe in use (cannot modify while running) |
| 6 | Permission denied |

---

## Parameter Validation

### Validation Rules

When host sends parameter values, equipment validates:

| Check | Description | Error |
|-------|-------------|-------|
| Type match | Value type matches parameter definition | "Type mismatch" |
| Range check | Value within Min-Max bounds | "Out of range" |
| Resolution | Value aligns with resolution step | "Resolution error" |
| Enumeration | Value in allowed discrete set | "Invalid selection" |
| Dependency | Cross-parameter constraints satisfied | "Dependency violated" |
| Access | Parameter is writable | "Read-only" |

### Cross-Parameter Dependencies

Some parameters have relationships:

```
Dependency Rules (Example):
─────────────────────────────────────────────────────────
Rule 1: If RF_Power > 1000W THEN Temperature must be < 400°C
Rule 2: CF4_Flow + O2_Flow must be <= MaxTotalFlow (200 sccm)
Rule 3: If EndpointMode = "Optical" THEN WindowState must be "Clean"
Rule 4: Duration must be > 0 when EndpointMode = "Time"
```

### Validation Response Example

```
Host requests: Set Temperature=600°C, RF_Power=500W

Response:
{L:2
  <U1 3>                        ← Validation failed
  {L:1
    {L:3
      <A "Temperature">          ← Failed parameter
      <U1 2>                    ← Error: out of range
      <A "Value 600.0 exceeds maximum 500.0">
    }
  }
}
```

---

## Scenarios & Message Flows

### Scenario 1: Read Recipe and Parameters

```
Host                                        Equipment
 │                                              │
 │─── S19F1 (List recipes in /Process/Etch/) ──→│
 │←── S19F2 (ETCH_OXIDE_V1, V2, NITRIDE_V1) ──│
 │                                              │
 │─── S19F15 (Get parameter defs for ETCH_V2) ─→│
 │←── S19F16 (Temp: F4, 20-500°C; ...)        │  Parameter metadata
 │                                              │
 │─── S19F17 (Get values for ETCH_V2, Step 2) ─→│
 │←── S19F18 (Temp=350, Power=500, ...) ───────│  Current values
```

### Scenario 2: Modify Recipe Parameters

```
Host                                        Equipment
 │                                              │
 │─── S19F19 (Set ETCH_V2, Step 2:            │
 │     Temperature=375, RF_Power=550) ─────────→│
 │                                              │  Validates:
 │                                              │  375 in [20,500] ✓
 │                                              │  550 in [0,2000] ✓
 │←── S19F20 (RCPACK=0, success) ─────────────│
 │                                              │
 │←── S6F11 (RecipeChanged: ETCH_V2) ─────────│  Event notification
 │─── S6F12 ───────────────────────────────────→│
```

### Scenario 3: Create New Recipe from Template

```
Host                                        Equipment
 │                                              │
 │─── S19F7 (Retrieve ETCH_TEMPLATE) ─────────→│  Get template
 │←── S19F8 (Full template content) ───────────│
 │                                              │
 │─── S19F3 (Create "ETCH_NEW_01"             │
 │     from template, namespace=/Process/Etch/) →│  Create new recipe
 │←── S19F4 (RCPACK=0) ───────────────────────│
 │                                              │
 │─── S19F19 (Set ETCH_NEW_01 parameters) ────→│  Set values
 │←── S19F20 (RCPACK=0) ──────────────────────│
```

### Scenario 4: Runtime Parameter Override (Tuning)

```
Host                                        Equipment
 │                                              │
 │  (Process job PJ001 using recipe ETCH_V2)   │
 │  (Tunable parameters can be modified live)   │
 │                                              │
 │─── S2F41 (RCMD="RECIPE_TUNE",              │
 │     ParamName="Temperature",                 │
 │     ParamValue=360) ────────────────────────→│  Runtime override
 │←── S2F42 (HCACK=0) ────────────────────────│  Applied immediately
 │                                              │
 │←── S6F11 (RecipeParamTuned) ───────────────│
 │─── S6F12 ───────────────────────────────────→│
```

---

## Relationship to Other Standards

### Recipe Management Evolution

```
E30 (GEM):     S7F3/F5 — Opaque binary recipes (upload/download blob)
     │
     │ Evolution: need to READ recipe content
     ▼
E139 (PDE):    S19Fx — Structured, parameter-level access
     │
     │ Evolution: need versioning, approval, CMS
     ▼
E170/171(SRO): Recipe lifecycle management, versioning, CMS integration
```

### Integration Points

| Standard | Integration with E139 |
|----------|----------------------|
| E40 (PJM) | Process Job references recipe by ID |
| E94 (CJM) | Control Job specifies recipe in ProcessingCtrlSpec |
| E30 (GEM) | Legacy S7 coexists with S19 |
| E170/E171 | Adds versioning on top of E139 structure |
| E39 (OSS) | Recipe is an E39 object with attributes |

---

## Implementation Guidelines

### Minimum Requirements

1. Support S19F1/F2 (recipe listing)
2. Support S19F7/F8 (recipe retrieve — at least parameters)
3. Support S19F15/F16 (parameter definitions)
4. Implement parameter validation on S19F19
5. Report RecipeChanged events
6. Document all parameters in compliance statement

### Recipe Storage

| Consideration | Guideline |
|--------------|-----------|
| Persistence | Recipes survive power cycles |
| Capacity | Document maximum recipe count |
| Protection | Prevent corruption during writes |
| Backup | Support recipe export/import |
| Versioning | Optional — if not, use E170 |
| Locking | Prevent modification during execution |

### Performance

| Operation | Expected Latency |
|-----------|-----------------|
| List recipes | < 1 sec |
| Retrieve recipe | < 2 sec |
| Get parameters | < 500 ms |
| Set parameters | < 1 sec (with validation) |
| Create recipe | < 2 sec |
| Delete recipe | < 1 sec |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E139-0709 Standard*
