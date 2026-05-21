# SEMI E170/E171 — SRO Recipe Management

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [SRO Concepts](#sro-concepts)
4. [Recipe Lifecycle](#recipe-lifecycle)
5. [Recipe Object Model](#recipe-object-model)
6. [Recipe Versioning](#recipe-versioning)
7. [Recipe Approval & CMS](#recipe-approval--cms)
8. [SECS-II Messages (Stream 20)](#secs-ii-messages-stream-20)
9. [Recipe Distribution](#recipe-distribution)
10. [Scenarios & Message Flows](#scenarios--message-flows)
11. [E170 vs E171 (Differences)](#e170-vs-e171-differences)
12. [Relationship to Other Standards](#relationship-to-other-standards)
13. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E170** and **SEMI E171** define the **Specification for Recipe Object (SRO)** and recipe management services. Together they provide enterprise-level recipe management including versioning, approval workflows, distribution, and change control — building upon E139's structured recipe format.

| Attribute | E170 | E171 |
|-----------|------|------|
| Full Name | Specification for Recipe and Parameter Management using Recipe Object | Recipe Management for High-Volume Manufacturing |
| Focus | Recipe Object model & operations at equipment | Factory-wide recipe distribution & control |
| Scope | Equipment-level | Factory/CMS-level |
| Messages | Stream 20 (S20Fx) | Stream 20 (S20Fx) — extended |
| Key Concept | Recipe as managed object with lifecycle | Recipe body management, distribution, approval |

---

## Purpose & Scope

### Evolution of Recipe Management

```
Generation 1 (E30/GEM):
  S7F3 — Upload binary blob recipe
  S7F5 — Download binary blob recipe
  ✗ Cannot read contents
  ✗ No versioning
  ✗ No approval workflow

Generation 2 (E139/PDE):
  S19Fx — Structured parameters
  ✓ Host can read/modify parameters
  ✗ No versioning
  ✗ No approval workflow
  ✗ No distribution management

Generation 3 (E170/E171/SRO):
  S20Fx — Full recipe lifecycle
  ✓ Structured parameters (from E139)
  ✓ Versioning (revision control)
  ✓ Approval workflow
  ✓ Factory-wide distribution
  ✓ Change tracking/audit
  ✓ Recipe validation before use
```

### What E170/E171 Add Over E139

| Capability | E139 | E170/E171 |
|-----------|------|-----------|
| Read recipe parameters | ✓ | ✓ |
| Modify recipe parameters | ✓ | ✓ |
| Version control | ✗ | ✓ |
| Approval states | ✗ | ✓ |
| Change history | ✗ | ✓ |
| Factory distribution | ✗ | ✓ |
| Recipe locking | Basic | ✓ (multi-level) |
| Recipe body vs. header | Combined | Separated |
| Multi-equipment sync | ✗ | ✓ |
| Audit trail | ✗ | ✓ |

---

## SRO Concepts

### Core Terminology

| Term | Description |
|------|-------------|
| **SRO** | Specification for Recipe Object — the standard framework |
| **Recipe Object** | A managed recipe entity with identity, version, and body |
| **Recipe Body** | The actual recipe content (parameters, steps) |
| **Recipe Header** | Metadata about the recipe (version, status, author) |
| **Recipe Namespace** | Organizational hierarchy for recipes |
| **Recipe Version** | A specific revision of a recipe |
| **Recipe State** | Approval/lifecycle state (Draft, Approved, etc.) |
| **CMS** | Change Management System — factory recipe authority |
| **Recipe Agent** | Software that manages recipes (host-side or equipment-side) |

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 FACTORY RECIPE MANAGEMENT                         │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │            CMS (Change Management System)                  │  │
│  │  - Master recipe repository                                │  │
│  │  - Version control                                         │  │
│  │  - Approval workflow                                       │  │
│  │  - Distribution management                                 │  │
│  │  - Audit trail                                            │  │
│  └────────────────────────────┬──────────────────────────────┘  │
│                               │                                  │
│              ┌────────────────┼────────────────┐                │
│              │                │                │                │
│              ▼                ▼                ▼                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Equipment 1  │  │  Equipment 2  │  │  Equipment 3  │        │
│  │  (E170 SRO)  │  │  (E170 SRO)  │  │  (E170 SRO)  │         │
│  │              │  │              │  │              │          │
│  │  Local copy  │  │  Local copy  │  │  Local copy  │          │
│  │  of approved │  │  of approved │  │  of approved │          │
│  │  recipes     │  │  recipes     │  │  recipes     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Recipe Object Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    RECIPE OBJECT (SRO)                            │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ RECIPE HEADER (Metadata)                                  │   │
│  │  RecipeID:       "ETCH_OXIDE"                             │   │
│  │  Version:        "3.2"                                    │   │
│  │  State:          APPROVED                                 │   │
│  │  Author:         "JSmith"                                 │   │
│  │  CreateDate:     "20260115"                               │   │
│  │  ApproveDate:    "20260120"                               │   │
│  │  ApprovedBy:     "TJones"                                 │   │
│  │  Description:    "Standard oxide etch — production"       │   │
│  │  EquipType:      "ETCHER_01"                              │   │
│  │  ChangeHistory:  [v1.0→v2.0→v3.0→v3.1→v3.2]            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ RECIPE BODY (Content — E139 PDE format)                   │   │
│  │  Global Parameters: {...}                                 │   │
│  │  Step 1: "Stabilize" — {Temp=350, Press=100, ...}        │   │
│  │  Step 2: "Main Etch" — {Power=500, CF4=50, ...}          │   │
│  │  Step 3: "Over Etch" — {Power=300, CF4=30, ...}          │   │
│  │  Step 4: "Purge" — {Power=0, N2=200, ...}                │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Recipe Lifecycle

### Recipe State Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    RECIPE LIFECYCLE STATES                        │
│                                                                  │
│  ┌────────────┐    Edit     ┌────────────┐                     │
│  │   DRAFT    │◄───────────│  EDITING    │                     │
│  │(New/WIP)   │────────────→│(Being mod'd)│                     │
│  └─────┬──────┘   Submit    └────────────┘                     │
│        │                                                         │
│        │ Submit for approval                                     │
│        ▼                                                         │
│  ┌────────────┐                                                 │
│  │ PENDING    │  Waiting for approver review                    │
│  │ APPROVAL   │                                                 │
│  └─────┬──────┘                                                 │
│        │                                                         │
│   ┌────┴────┐                                                   │
│   │         │                                                    │
│   ▼         ▼                                                    │
│ ┌──────┐ ┌──────┐                                              │
│ │APPRO-│ │REJEC-│                                              │
│ │VED   │ │TED   │──→ Back to DRAFT for rework                  │
│ └──┬───┘ └──────┘                                              │
│    │                                                             │
│    │ Deploy to equipment                                         │
│    ▼                                                             │
│ ┌────────────┐                                                  │
│ │ DEPLOYED   │  Active on equipment, ready for use             │
│ └─────┬──────┘                                                  │
│       │                                                          │
│       │ Superseded by new version                                │
│       ▼                                                          │
│ ┌────────────┐                                                  │
│ │ OBSOLETE   │  No longer valid for production                  │
│ └────────────┘                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Recipe State Definitions

| State | Value | Description |
|-------|-------|-------------|
| **DRAFT** | 1 | Recipe under development, not validated |
| **EDITING** | 2 | Currently being modified by a user |
| **PENDING APPROVAL** | 3 | Submitted, awaiting review |
| **APPROVED** | 4 | Reviewed and approved for use |
| **REJECTED** | 5 | Review failed, needs rework |
| **DEPLOYED** | 6 | Active on equipment, available for processing |
| **LOCKED** | 7 | Cannot be modified (active in production) |
| **OBSOLETE** | 8 | Superseded, should not be used |
| **ARCHIVED** | 9 | Preserved for reference only |

### State Transitions

| From | To | Trigger | Authorization |
|------|-----|---------|---------------|
| — | DRAFT | Create | Recipe author |
| DRAFT | EDITING | Open for edit | Recipe author |
| EDITING | DRAFT | Save/close | Recipe author |
| DRAFT | PENDING APPROVAL | Submit | Recipe author |
| PENDING APPROVAL | APPROVED | Approve | Approver |
| PENDING APPROVAL | REJECTED | Reject | Approver |
| REJECTED | DRAFT | Rework | Recipe author |
| APPROVED | DEPLOYED | Deploy to equipment | Deploy authority |
| DEPLOYED | LOCKED | Start production use | Automatic |
| LOCKED | DEPLOYED | Production complete | Automatic |
| DEPLOYED | OBSOLETE | New version deployed | Deploy authority |
| Any | ARCHIVED | Archive | Administrator |

---

## Recipe Object Model

### Recipe Object Attributes (E39)

| Attribute | Type | Description |
|-----------|------|-------------|
| **RecipeID** | ASCII | Unique recipe identifier |
| **RecipeVersion** | ASCII | Version string (e.g., "3.2") |
| **RecipeState** | U1 | Lifecycle state |
| **RecipeType** | ASCII | Process/Clean/Setup/etc. |
| **Author** | ASCII | Creator username |
| **CreateDate** | ASCII | Creation timestamp |
| **ModifyDate** | ASCII | Last modification timestamp |
| **Approver** | ASCII | Approver username |
| **ApproveDate** | ASCII | Approval timestamp |
| **Description** | ASCII | Human-readable description |
| **EquipmentType** | ASCII | Target equipment model |
| **BodyChecksum** | ASCII | Integrity hash of body |
| **ChangeDescription** | ASCII | What changed from previous version |
| **ParentVersion** | ASCII | Version this was derived from |
| **LockStatus** | U1 | Lock state (0=unlocked, 1=locked) |
| **DeployTarget** | List | Equipment(s) where deployed |
| **ExpirationDate** | ASCII | Optional expiry date |

### Recipe Body Object

The body is separate from the header for efficient transfer:

| Attribute | Type | Description |
|-----------|------|-------------|
| **BodyID** | ASCII | Body identifier (links to header) |
| **BodyFormat** | U1 | Format type (PDE/binary/XML) |
| **BodySize** | U4 | Size in bytes |
| **BodyContent** | List/Binary | Actual recipe content |
| **ParameterDefs** | List | Parameter definitions (E139) |
| **StepCount** | U2 | Number of recipe steps |
| **Checksum** | ASCII | Integrity verification |

---

## Recipe Versioning

### Version Numbering

```
Version format: Major.Minor

Major version: Significant process change (requalification needed)
Minor version: Tuning/optimization (no requalification)

Example history:
  1.0  Initial recipe (qualified)
  1.1  Tuned temperature +5°C
  1.2  Adjusted RF power
  2.0  New gas chemistry (re-qualified)
  2.1  Flow rate optimization
  3.0  Endpoint detection added (re-qualified)
  3.1  Endpoint threshold tuned
  3.2  Duration fallback adjusted
```

### Version Tree

```
1.0 ─── 1.1 ─── 1.2
 │
 └── 2.0 ─── 2.1
      │
      └── 3.0 ─── 3.1 ─── 3.2 (current)
```

### Version Comparison

Equipment/CMS can compare recipe versions:

```
Comparison: Version 3.1 vs 3.2
─────────────────────────────────────────────────
Parameter          v3.1        v3.2        Change
────────────────   ─────────   ─────────   ──────
Temperature        350°C       350°C       (same)
RF_Power           500W        500W        (same)
Duration           180s        195s        +15s
EndpointFallback   200s        210s        +10s
CF4_Flow           50 sccm     50 sccm     (same)
─────────────────────────────────────────────────
Summary: 2 parameters changed (Duration, EndpointFallback)
```

---

## Recipe Approval & CMS

### Approval Workflow

```
┌────────────────────────────────────────────────────────────────────────┐
│                    RECIPE APPROVAL WORKFLOW                              │
│                                                                         │
│  Process Engineer                    Process Owner                      │
│  ┌───────────────┐                 ┌───────────────┐                   │
│  │ 1. Create/Edit│                 │               │                   │
│  │    recipe     │                 │               │                   │
│  └───────┬───────┘                 │               │                   │
│          │                          │               │                   │
│  ┌───────┴───────┐                 │               │                   │
│  │ 2. Validate   │                 │               │                   │
│  │    parameters │                 │               │                   │
│  └───────┬───────┘                 │               │                   │
│          │                          │               │                   │
│  ┌───────┴───────┐                 │               │                   │
│  │ 3. Submit for │                 │               │                   │
│  │    approval   │─────────────────→│ 4. Review    │                   │
│  └───────────────┘                 │    recipe     │                   │
│                                    └───────┬───────┘                   │
│                                            │                            │
│                                    ┌───────┴───────┐                   │
│                                    │ 5. Approve or │                   │
│                                    │    Reject     │                   │
│                                    └───────┬───────┘                   │
│                                            │                            │
│  ┌───────────────┐                ┌───────┴───────┐                   │
│  │ 6. Deploy to  │←───────────────│   APPROVED    │                   │
│  │    equipment  │                └───────────────┘                   │
│  └───────────────┘                                                     │
└────────────────────────────────────────────────────────────────────────┘
```

### CMS Integration

The Change Management System (CMS) is the central authority:

| CMS Function | Description |
|-------------|-------------|
| Master Repository | Single source of truth for all recipes |
| Version Control | Maintains all versions with history |
| Access Control | Role-based permissions |
| Approval Engine | Manages approval workflow |
| Distribution | Pushes recipes to target equipment |
| Audit Trail | Records all changes with timestamps |
| Validation | Ensures recipes meet quality standards |
| Synchronization | Keeps equipment copies in sync |

### Recipe Distribution Model

```
CMS (Master)
│
├── Equipment Group "ETCH_01" (3 tools)
│   ├── ETCH_01_A: Recipe v3.2 deployed ✓
│   ├── ETCH_01_B: Recipe v3.2 deployed ✓
│   └── ETCH_01_C: Recipe v3.1 deployed ⚠ (update pending)
│
├── Equipment Group "ETCH_02" (2 tools)
│   ├── ETCH_02_A: Recipe v2.1 deployed ✓ (different recipe)
│   └── ETCH_02_B: Recipe v2.1 deployed ✓
│
└── Deployment Status: 4/5 synchronized ✓
```

---

## SECS-II Messages (Stream 20)

### Stream 20 Message Summary

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S20F1/F2 | Recipe List Request/Data | H→E | List recipe objects |
| S20F3/F4 | Recipe Create/Ack | H→E | Create new recipe object |
| S20F5/F6 | Recipe Delete/Ack | H→E | Delete recipe object |
| S20F7/F8 | Recipe Header Get/Data | H→E | Get recipe metadata |
| S20F9/F10 | Recipe Body Download/Ack | H→E | Download recipe body to equipment |
| S20F11/F12 | Recipe Body Upload Request/Data | H→E | Upload recipe body from equipment |
| S20F13/F14 | Recipe State Change/Ack | H→E | Change recipe lifecycle state |
| S20F15/F16 | Recipe Verify Request/Result | H↔E | Verify recipe integrity |
| S20F17/F18 | Recipe Compare Request/Data | H→E | Compare two recipe versions |
| S20F19/F20 | Recipe Lock/Ack | H→E | Lock recipe for production |
| S20F21/F22 | Recipe Unlock/Ack | H→E | Unlock recipe |
| S20F23/F24 | Recipe Approve/Ack | H→E | Approve recipe (if done at equipment) |
| S20F25/F26 | Recipe Deploy/Ack | H→E | Deploy recipe to equipment |
| S20F27/F28 | Recipe Obsolete/Ack | H→E | Mark recipe as obsolete |

### S20F3/F4 — Create Recipe Object

**S20F3 Request:**
```
{L:5
  <A RecipeID>             ← Recipe identifier
  <A RecipeVersion>        ← Version string
  <A RecipeType>           ← Type (Process/Clean/etc.)
  <A Description>          ← Human description
  {L:n                     ← Additional attributes
    {L:2
      <A AttrName>
      <V AttrValue>
    }
  }
}
```

**S20F4 Response:**
```
{L:2
  <U1 SRACK>              ← 0=success
  {L:n                     ← Errors
    {L:2
      <U1 ErrCode>
      <A ErrText>
    }
  }
}
```

### S20F9/F10 — Recipe Body Download (Host → Equipment)

**S20F9 Request:**
```
{L:4
  <A RecipeID>             ← Target recipe
  <A RecipeVersion>        ← Version to download
  <U4 BodySize>           ← Size of body content
  {L body}                 ← Recipe body (PDE format from E139)
}
```

**S20F10 Response:**
```
{L:2
  <U1 SRACK>              ← 0=success
  {L:n
    {L:2
      <U1 ErrCode>
      <A ErrText>          ← "Checksum mismatch", "No space", etc.
    }
  }
}
```

### S20F13/F14 — Recipe State Change

**S20F13 Request:**
```
{L:4
  <A RecipeID>
  <A RecipeVersion>
  <U1 NewState>            ← Target state code
  <A ChangeReason>         ← Why state is changing
}
```

### S20F15/F16 — Recipe Verify

**S20F15 Request:**
```
{L:3
  <A RecipeID>
  <A RecipeVersion>
  <A ExpectedChecksum>     ← Expected hash value
}
```

**S20F16 Response:**
```
{L:3
  <U1 VerifyResult>        ← 0=match, 1=mismatch, 2=not found
  <A ActualChecksum>       ← Calculated hash
  <A VerifyDetail>         ← Additional info
}
```

### S20F17/F18 — Recipe Compare

**S20F17 Request:**
```
{L:4
  <A RecipeID>
  <A Version1>             ← First version to compare
  <A Version2>             ← Second version to compare
  <U1 CompareScope>        ← 0=all, 1=params only, 2=header only
}
```

**S20F18 Response:**
```
{L:3
  <U1 CompareResult>       ← 0=identical, 1=different
  <U4 DifferenceCount>    ← Number of differences
  {L:n                     ← Difference details
    {L:4
      <A ParamName>
      <A StepID>
      <V Value1>           ← Value in version 1
      <V Value2>           ← Value in version 2
    }
  }
}
```

### SRACK Codes (Stream 20 Acknowledge)

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Recipe not found |
| 2 | Version not found |
| 3 | Invalid state transition |
| 4 | Permission denied |
| 5 | Recipe locked |
| 6 | Validation failed |
| 7 | Storage full |
| 8 | Checksum mismatch |
| 9 | Recipe in use |
| 10 | Duplicate (already exists) |

---

## Recipe Distribution

### Distribution Process

```
CMS (Factory Level)                     Equipment
 │                                          │
 │  1. Recipe ETCH_V3.2 approved           │
 │                                          │
 │─── S20F25 (Deploy recipe request) ─────→│  Notify equipment
 │←── S20F26 (SRACK=0, ready) ────────────│
 │                                          │
 │─── S20F9 (Download recipe body) ────────→│  Send recipe content
 │←── S20F10 (SRACK=0) ───────────────────│  Stored successfully
 │                                          │
 │─── S20F15 (Verify checksum) ────────────→│  Verify integrity
 │←── S20F16 (Match=0, verified) ─────────│  ✓ Integrity OK
 │                                          │
 │─── S20F13 (State→DEPLOYED) ────────────→│  Activate recipe
 │←── S20F14 (SRACK=0) ───────────────────│  Recipe active
 │                                          │
 │  Recipe now available for process jobs   │
```

### Rollback Procedure

```
CMS                                     Equipment
 │                                          │
 │  Problem detected with v3.2!            │
 │                                          │
 │─── S20F19 (Lock v3.2 — prevent use) ───→│  Lock current version
 │←── S20F20 (SRACK=0) ───────────────────│
 │                                          │
 │─── S20F13 (v3.2 → OBSOLETE) ───────────→│  Mark obsolete
 │←── S20F14 (SRACK=0) ───────────────────│
 │                                          │
 │─── S20F13 (v3.1 → DEPLOYED) ───────────→│  Re-activate v3.1
 │←── S20F14 (SRACK=0) ───────────────────│  Rollback complete
 │                                          │
 │  v3.1 now active, v3.2 obsolete         │
```

---

## Scenarios & Message Flows

### Scenario 1: New Recipe Development & Deployment

```
Engineer                CMS                     Equipment
 │                       │                          │
 │─── Create draft ─────→│                          │
 │←── DRAFT created ─────│                          │
 │                       │                          │
 │─── Edit parameters ──→│                          │
 │←── Saved ─────────────│                          │
 │                       │                          │
 │─── Submit for         │                          │
 │    approval ─────────→│                          │
 │                       │                          │
 │                       │ (Notification to approver)
 │                       │                          │
Approver                 │                          │
 │─── Review recipe ────→│                          │
 │←── Recipe details ────│                          │
 │                       │                          │
 │─── APPROVE ──────────→│                          │
 │                       │                          │
 │                       │─── S20F25 (Deploy) ─────→│
 │                       │←── S20F26 (OK) ─────────│
 │                       │                          │
 │                       │─── S20F9 (Body) ────────→│
 │                       │←── S20F10 (OK) ─────────│
 │                       │                          │
 │                       │─── S20F15 (Verify) ─────→│
 │                       │←── S20F16 (OK) ─────────│
 │                       │                          │
 │                       │─── S20F13 (DEPLOYED) ───→│
 │                       │←── S20F14 (OK) ─────────│
 │                       │                          │
 │                       │  Recipe ready for use    │
```

### Scenario 2: Recipe Version Update

```
CMS                                     Equipment
 │                                          │
 │  v3.1 currently DEPLOYED                │
 │  v3.2 just APPROVED                     │
 │                                          │
 │─── S20F13 (v3.1 → OBSOLETE) ───────────→│  Retire old version
 │←── S20F14 (OK) ────────────────────────│
 │                                          │
 │─── S20F9 (Download v3.2 body) ─────────→│  Install new version
 │←── S20F10 (OK) ────────────────────────│
 │                                          │
 │─── S20F13 (v3.2 → DEPLOYED) ───────────→│  Activate new version
 │←── S20F14 (OK) ────────────────────────│
 │                                          │
 │  Next process job will use v3.2          │
 │  Jobs already running with v3.1 continue │
```

### Scenario 3: Recipe Integrity Check

```
Host/CMS                                Equipment
 │                                          │
 │  Periodic verification                   │
 │                                          │
 │─── S20F15 (Verify ETCH_V3.2,           │
 │     Checksum="a1b2c3d4e5f6") ───────────→│  Check integrity
 │                                          │
 │  Equipment calculates hash of stored body │
 │                                          │
 │←── S20F16 (Result=0, Match) ────────────│  ✓ OK
 │                                          │
 │  OR                                      │
 │                                          │
 │←── S20F16 (Result=1, Mismatch,          │  ✗ CORRUPTION!
 │     Actual="x9y8z7...") ────────────────│
 │                                          │
 │  → Trigger re-download of recipe         │
 │  → Alarm: recipe corruption detected     │
```

---

## E170 vs E171 (Differences)

### Scope Comparison

| Aspect | E170 | E171 |
|--------|------|------|
| **Focus** | Equipment-level recipe management | Factory-wide recipe system |
| **Repository** | Equipment-local storage | Centralized CMS repository |
| **Operations** | CRUD on local recipes | Distribution, sync, workflow |
| **Versioning** | Supports version attribute | Full version tree management |
| **Approval** | Simple state change | Full workflow engine |
| **Distribution** | Receives from host | Manages multi-tool deployment |
| **Audit** | Local event log | Enterprise audit trail |

### When to Use Each

| Scenario | Use |
|----------|-----|
| Equipment needs to store recipes locally | E170 |
| Host needs to push recipe to equipment | E170 (S20F9) |
| Factory needs to manage recipe across 50 tools | E171 |
| Need approval workflow with multiple reviewers | E171 |
| Need to compare recipe versions | E170 (S20F17) or E171 |
| Need to roll back to previous version | E171 (distribution level) |
| Equipment needs to validate recipe before use | E170 (S20F15) |
| Need audit trail for FDA/regulatory compliance | E171 |

### Implementation Approaches

```
Simple (E170 only):
  Host ──S20Fx──→ Equipment
  (Host manages versions, equipment stores/uses)

Full (E170 + E171):
  CMS ─── E171 ───→ Host ─── E170(S20Fx) ───→ Equipment
  (CMS is master, Host relays, Equipment is consumer)
```

---

## Relationship to Other Standards

### Recipe Standards Evolution

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  E30 (GEM) S7Fx        ← Basic recipe upload/download (binary)     │
│       │                                                              │
│       ▼                                                              │
│  E139 (PDE) S19Fx      ← Structured parameters, host-readable     │
│       │                                                              │
│       ▼                                                              │
│  E170/E171 (SRO) S20Fx ← Versioning, approval, distribution       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Integration Map

| Standard | Role in Recipe Management |
|----------|---------------------------|
| E30 (GEM) | Legacy S7 may coexist (backward compatibility) |
| E39 (OSS) | Recipe Object model framework |
| E40 (PJM) | Process Job references recipe by ID + version |
| E94 (CJM) | Control Job includes recipe in ProcessingCtrlSpec |
| E139 (PDE) | Recipe body FORMAT (E170/E171 uses this internally) |
| E170 | Equipment-level recipe object management |
| E171 | Factory-level recipe lifecycle management |

---

## Implementation Guidelines

### Minimum E170 Compliance

1. Support recipe objects with ID, version, state
2. Implement S20F1/F2 (list recipes)
3. Implement S20F7/F8 (get header)
4. Implement S20F9/F10 (download body)
5. Implement S20F13/F14 (state change)
6. Implement S20F15/F16 (verify)
7. Report recipe events via S6F11
8. Maintain recipe integrity (checksum)

### Storage Requirements

| Requirement | Description |
|------------|-------------|
| Multiple versions | Store at least 2 versions per recipe |
| State persistence | Survive power cycles |
| Integrity | Checksum validation on read |
| Capacity | Document maximum recipe count |
| Access control | Prevent unauthorized modification |
| Concurrent access | Handle simultaneous read requests |

### Collection Events for E170/E171

| Event | Description |
|-------|-------------|
| RecipeCreated | New recipe object created |
| RecipeDeleted | Recipe object removed |
| RecipeStateChanged | Lifecycle state transition |
| RecipeBodyChanged | Recipe content modified |
| RecipeDeployed | Recipe activated on equipment |
| RecipeObsolete | Recipe marked obsolete |
| RecipeLocked | Recipe locked for production |
| RecipeUnlocked | Recipe unlocked |
| RecipeVerifyFailed | Integrity check failed |
| RecipeDownloaded | Recipe body received from host |

### Security Considerations

| Concern | Mitigation |
|---------|------------|
| Unauthorized modification | Role-based access control |
| Recipe corruption | Checksum/hash verification |
| Version confusion | Strict version naming + validation |
| Unauthorized deployment | Approval workflow required |
| Loss of recipe | Redundant storage + CMS backup |
| Replay attack | Timestamp + sequence validation |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E170-0709 and SEMI E171-0709 Standards*
