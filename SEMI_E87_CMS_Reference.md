# SEMI E87 — Carrier Management Standard (CMS)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Carrier Concept](#carrier-concept)
4. [Load Port State Model](#load-port-state-model)
5. [Carrier State Model](#carrier-state-model)
6. [Carrier ID & Slot Map Verification](#carrier-id--slot-map-verification)
7. [Access Modes](#access-modes)
8. [Carrier Attributes](#carrier-attributes)
9. [Load Port Attributes](#load-port-attributes)
10. [SECS-II Messages & Commands](#secs-ii-messages--commands)
11. [Scenarios & Message Flows](#scenarios--message-flows)
12. [Relationship to Other Standards](#relationship-to-other-standards)
13. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E87** defines the **Carrier Management Standard (CMS)**, which standardizes how equipment manages carriers (FOUPs, cassettes, pods) at load ports and tracks their lifecycle from arrival to departure.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E87 |
| Full Name | Carrier Management Standard |
| Abbreviation | CMS |
| Primary Purpose | Carrier lifecycle management at equipment |
| Dependencies | SEMI E5, E30, E39 |
| Related | E84 (AMHS), E90 (Substrate), E94 (Control Job) |
| Key Objects | Carrier, LoadPort |

---

## Purpose & Scope

### What E87 Defines

1. **Load Port management** — State model for physical carrier access points
2. **Carrier tracking** — Lifecycle from arrival through departure
3. **ID verification** — Reading and verifying carrier identity
4. **Slot mapping** — Detecting substrate presence in carrier slots
5. **Access control** — Who can access carrier content (host vs. operator)
6. **Content management** — Tracking what's in each carrier slot

### What E87 Does NOT Define

- Physical carrier transport (that's E84/AMHS)
- Substrate processing (that's E40)
- Substrate identity and tracking (that's E90)
- Lot-level control (that's E94)

### Physical Context

```
                    AMHS (Automated Material Handling System)
                              │
                              │ Carrier delivered
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         EQUIPMENT                                 │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐                │
│  │  Load Port  │  │  Load Port  │  │  Load Port  │               │
│  │    LP1      │  │    LP2      │  │    LP3      │               │
│  │            │  │            │  │            │               │
│  │ ┌────────┐ │  │ ┌────────┐ │  │            │               │
│  │ │ FOUP   │ │  │ │ FOUP   │ │  │  (empty)   │               │
│  │ │ 001    │ │  │ │ 002    │ │  │            │               │
│  │ │ W1-W25 │ │  │ │ W1-W20 │ │  │            │               │
│  │ └────────┘ │  │ └────────┘ │  │            │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │            Processing Modules             │                   │
│  └──────────────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Carrier Concept

### Carrier Types

| Type | Description | Capacity | Industry |
|------|-------------|----------|----------|
| **FOUP** | Front Opening Unified Pod | 25 wafers | 300mm fab |
| **FOSB** | Front Opening Shipping Box | 25 wafers | 300mm transport |
| **Cassette** | Open cassette | 13-25 wafers | 200mm fab |
| **Magazine** | Strip/panel magazine | Variable | Assembly |
| **SMIF Pod** | Standard Mechanical Interface | 25 wafers | 200mm cleanroom |

### Carrier Properties

```
┌─────────────────────────────────────────────────────┐
│                  CARRIER (FOUP)                       │
├─────────────────────────────────────────────────────┤
│  Carrier ID:      "FOUP001"                          │
│  Capacity:        25 slots                           │
│  Content Map:     [1,1,1,0,0,...,0]  (3 wafers)     │
│  ID Status:       Verified                           │
│  SlotMap Status:  Verified                           │
│  Access Mode:     Auto                               │
│  Location:        LP1                                │
│  Association:     Control Job CJ001                  │
├─────────────────────────────────────────────────────┤
│  Slot 1:  Wafer W001 [Present]                      │
│  Slot 2:  Wafer W002 [Present]                      │
│  Slot 3:  Wafer W003 [Present]                      │
│  Slot 4:  [Empty]                                    │
│  ...                                                 │
│  Slot 25: [Empty]                                    │
└─────────────────────────────────────────────────────┘
```

---

## Load Port State Model

### Load Port Transfer State Model

Tracks the physical state of the load port mechanism:

```
┌───────────────────────────────────────────────────────────────┐
│                   LOAD PORT TRANSFER STATES                     │
│                                                                │
│  ┌───────────────┐                                            │
│  │ OUT OF SERVICE │  Load port disabled/maintenance            │
│  └───────┬───────┘                                            │
│          │ InService                                           │
│          ▼                                                     │
│  ┌───────────────┐                                            │
│  │ IN SERVICE     │  Load port available                       │
│  │               │                                            │
│  │  ┌─────────────────────────────────────────────────┐      │
│  │  │ TRANSFER BLOCKED                                │       │
│  │  │ (Carrier access not ready)                      │       │
│  │  └──────────────────────────┬──────────────────────┘      │
│  │                             │ Ready                         │
│  │  ┌──────────────────────────▼──────────────────────┐      │
│  │  │ READY TO LOAD                                   │       │
│  │  │ (Empty port, ready for carrier)                  │       │
│  │  └──────────────────────────┬──────────────────────┘      │
│  │                             │ Carrier arrives               │
│  │  ┌──────────────────────────▼──────────────────────┐      │
│  │  │ READY TO UNLOAD                                 │       │
│  │  │ (Carrier present, ready for removal)             │       │
│  │  └─────────────────────────────────────────────────┘      │
│  └───────────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────────┘
```

### Load Port State Definitions

| State | Description |
|-------|-------------|
| **OUT OF SERVICE** | Port disabled, no operations possible |
| **IN SERVICE / TRANSFER BLOCKED** | Port enabled but not ready for transfer |
| **IN SERVICE / READY TO LOAD** | Port empty and ready to receive carrier |
| **IN SERVICE / READY TO UNLOAD** | Carrier present, can be removed |

### Load Port Reservation States

```
┌───────────────────┐
│  NOT RESERVED      │  No carrier expected
└────────┬──────────┘
         │ Reserve command
         ▼
┌───────────────────┐
│  RESERVED          │  Carrier expected (space held)
└────────┬──────────┘
         │ Carrier arrives or Cancel
         ▼
┌───────────────────┐
│  NOT RESERVED      │  Back to available
└───────────────────┘
```

### Load Port Association States

```
┌───────────────────┐
│  NOT ASSOCIATED    │  No carrier linked to port
└────────┬──────────┘
         │ Carrier placed or Bind command
         ▼
┌───────────────────┐
│  ASSOCIATED        │  Carrier linked to this port
└────────┬──────────┘
         │ Carrier removed or Unbind
         ▼
┌───────────────────┐
│  NOT ASSOCIATED    │  Carrier link removed
└───────────────────┘
```

---

## Carrier State Model

### Carrier Lifecycle States

```
                    ┌─────────────────────┐
                    │   CARRIER NOT        │
                    │   INSTANTIATED       │  Carrier not known to equipment
                    └──────────┬──────────┘
                               │ Carrier arrives or is created
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CARRIER INSTANTIATED                           │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │   ID NOT          ID              WAITING          SLOT  │    │
│  │   VERIFIED ──→ VERIFICATION ──→ FOR HOST ──→    MAP     │    │
│  │                  IN PROGRESS      (ID done)     VERIFY   │    │
│  │                                                          │    │
│  └──────────────────────────────────┬───────────────────────┘    │
│                                     │                            │
│  ┌──────────────────────────────────▼───────────────────────┐    │
│  │                  CARRIER ACCESSED                          │    │
│  │  (Door open, substrates can be moved in/out)              │    │
│  └──────────────────────────────────┬───────────────────────┘    │
│                                     │                            │
│  ┌──────────────────────────────────▼───────────────────────┐    │
│  │                  CARRIER COMPLETE                          │    │
│  │  (Processing done, ready for departure)                   │    │
│  └──────────────────────────────────┬───────────────────────┘    │
│                                     │                            │
└─────────────────────────────────────┼────────────────────────────┘
                                      │ Carrier departs
                                      ▼
                    ┌─────────────────────┐
                    │   CARRIER NOT        │
                    │   INSTANTIATED       │
                    └─────────────────────┘
```

### Detailed Carrier States

| State | Value | Description |
|-------|-------|-------------|
| **ID NOT READ** | 0 | Carrier arrived, ID not yet read |
| **WAITING FOR HOST** | 1 | ID read, waiting for host decision |
| **ID VERIFICATION OK** | 2 | Host confirmed carrier ID |
| **ID VERIFICATION FAILED** | 3 | ID mismatch or read failure |
| **SLOT MAP NOT READ** | 4 | Slot map not yet performed |
| **WAITING FOR HOST (SlotMap)** | 5 | Slot map done, awaiting host |
| **SLOT MAP VERIFICATION OK** | 6 | Host confirmed slot map |
| **SLOT MAP VERIFICATION FAILED** | 7 | Slot map mismatch |
| **CARRIER ACCESSING** | 8 | Carrier door opening |
| **CARRIER ACCESSED** | 9 | Carrier open, substrates accessible |
| **CARRIER CLOSING** | 10 | Carrier door closing |
| **CARRIER CLOSED** | 11 | Carrier sealed |
| **CARRIER COMPLETE** | 12 | All processing done |
| **CARRIER STOPPED** | 13 | Carrier processing stopped |

---

## Carrier ID & Slot Map Verification

### Carrier ID Verification Flow

```
Host                                        Equipment
 │                                              │
 │   Carrier placed on LP1                      │
 │                                              │
 │←── S6F11 (CarrierArrived) ─────────────────│  Carrier detected
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │          ... equipment reads ID barcode ...   │
 │                                              │
 │←── S6F11 (IDRead, CarrierID="FOUP001") ────│  ID read complete
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S3F17 (CarrierAction: ProceedWithCarrier)→│  Host accepts carrier
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (IDVerificationOK) ───────────────│  ID confirmed
 │─── S6F12 ───────────────────────────────────→│
```

### Carrier ID Status Values

| Status | Description |
|--------|-------------|
| 0 | ID Not Read |
| 1 | ID Read, awaiting host verification |
| 2 | ID Verified OK by host |
| 3 | ID Verification Failed |

### Slot Map Verification Flow

```
Host                                        Equipment
 │                                              │
 │   (After ID verification)                    │
 │                                              │
 │          ... equipment scans slots ...        │
 │                                              │
 │←── S6F11 (SlotMapRead) ────────────────────│  Slot map data:
 │     ContentMap = [1,1,1,0,0,...,0]           │  [occupied, occupied, occupied, empty...]
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S3F17 (SlotMapVerify, ContentMap) ───────→│  Host confirms map
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (SlotMapVerificationOK) ──────────│  Map confirmed
 │─── S6F12 ───────────────────────────────────→│
```

### Slot Map Status Values

| Status | Description |
|--------|-------------|
| 0 | Slot Map Not Read |
| 1 | Slot Map Read, awaiting verification |
| 2 | Slot Map Verified OK |
| 3 | Slot Map Verification Failed |

### Content Map Values (per slot)

| Value | Meaning |
|-------|---------|
| 0 | Empty — no substrate in slot |
| 1 | Occupied — substrate correctly seated |
| 2 | Double-slotted — two substrates detected |
| 3 | Cross-slotted — substrate misaligned |
| 4 | Unknown — cannot determine |
| 5 | Undefined — not scanned yet |

---

## Access Modes

### Access Mode Definition

Access mode determines how substrates in a carrier can be accessed:

| Mode | Value | Description |
|------|-------|-------------|
| **MANUAL** | 0 | Operator manually loads/unloads substrates |
| **AUTO** | 1 | Equipment automatically accesses substrates |

### Access Mode Transitions

```
┌─────────────────────────────────────────────────────┐
│              ACCESS MODE CONTROL                      │
│                                                      │
│  ┌──────────┐                     ┌──────────────┐  │
│  │  MANUAL   │ ←────────────────→ │    AUTO       │  │
│  │  Access   │  Host or Operator  │    Access    │  │
│  │           │  command            │              │  │
│  └──────────┘                     └──────────────┘  │
│                                                      │
│  MANUAL: Operator opens FOUP door manually          │
│  AUTO: Equipment robot opens and accesses FOUP      │
└─────────────────────────────────────────────────────┘
```

---

## Carrier Attributes

### Core Carrier Attributes (E39 Object)

| Attribute | Type | Access | Description |
|-----------|------|--------|-------------|
| CarrierID | ASCII | R/W | Carrier identifier |
| ObjType | ASCII | RO | "Carrier" |
| LocationID | ASCII | RO | Current location (port ID) |
| CarrierIDStatus | U1 | RO | ID verification state |
| SlotMapStatus | U1 | RO | Slot map verification state |
| AccessMode | U1 | R/W | Manual(0) or Auto(1) |
| ContentMap | List[U1] | RO | Slot occupancy array |
| Capacity | U1 | RO | Number of slots |
| SubstrateCount | U4 | RO | Number of occupied slots |
| CarrierState | U1 | RO | Current lifecycle state |
| Usage | ASCII | R/W | Purpose (PRODUCT, TEST, DUMMY, etc.) |
| CarrierType | ASCII | RO | Physical carrier type |
| LotID | ASCII | R/W | Associated lot identifier |

### Carrier Usage Types

| Usage | Description |
|-------|-------------|
| PRODUCT | Production wafers |
| TEST | Test/qualification wafers |
| DUMMY | Dummy/filler wafers |
| MONITOR | Monitor wafers |
| ENGINEERING | Engineering test |
| EMPTY | Empty carrier |

---

## Load Port Attributes

| Attribute | Type | Access | Description |
|-----------|------|--------|-------------|
| PortID | ASCII/U1 | RO | Load port identifier |
| PortTransferState | U1 | RO | Transfer state |
| PortReservationState | U1 | RO | Reservation state |
| PortAssociationState | U1 | RO | Carrier association |
| PortAccessMode | U1 | R/W | Manual/Auto |
| CarrierID | ASCII | RO | Associated carrier (if any) |
| PortType | U1 | RO | Input/Output/Both |

### Port Types

| Type | Value | Description |
|------|-------|-------------|
| Input | 1 | Material input only |
| Output | 2 | Material output only |
| Input/Output | 3 | Bidirectional |

---

## SECS-II Messages & Commands

### Primary Messages for E87

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S3F17/F18 | Carrier Action Request/Ack | H↔E | Carrier operations |
| S1F3/F4 | Status Request/Data | H→E | Query port/carrier status |
| S2F41/F42 | Host Command/Ack | H→E | Port/carrier commands |
| S6F11/F12 | Event Report/Ack | E→H | Carrier events |
| S14F1/F2 | GetAttr/Data | H→E | Read carrier/port attributes |

### S3F17 — Carrier Action Request

**Request:**
```
{L:5
  <U4 DATAID>
  <A CARRIERACTION>       ← Action to perform
  <A CARRIERID>           ← Target carrier ID
  <U1 PORTID>            ← Target port (or 0 for any)
  {L:n                    ← Additional parameters
    {L:2
      <A PARAMNAME>
      <V PARAMVALUE>
    }
  }
}
```

### Carrier Actions (S3F17)

| Action | Description |
|--------|-------------|
| **BIND** | Associate carrier ID with port |
| **CANCEL_BIND** | Cancel pending bind |
| **PROCEED_WITH_CARRIER** | Accept carrier after ID read |
| **CANCEL_CARRIER** | Reject carrier |
| **CARRIER_NOTIFICATION** | Inform equipment of incoming carrier |
| **CANCEL_CARRIER_NOTIFICATION** | Cancel expected carrier |
| **SLOT_MAP_READ** | Request slot map verification |
| **CARRIER_RE_OPENED** | Re-access carrier |
| **CARRIER_CLOSED** | Carrier processing complete |
| **CANCEL_CARRIER_AT_PORT** | Remove carrier association |

### S3F18 — Carrier Action Acknowledge

**Response:**
```
{L:2
  <U1 CAACK>             ← Acknowledge code
  {L:n                    ← Error details
    {L:2
      <U1 ERRCODE>
      <A ERRTEXT>
    }
  }
}
```

**CAACK Codes:**

| Code | Meaning |
|------|---------|
| 0 | Acknowledged, command will be performed |
| 1 | Invalid command |
| 2 | Cannot perform now |
| 3 | Invalid data/parameter |
| 4 | Acknowledge, completion signaled by event |
| 5 | Rejected, already in requested state |
| 6 | No such object |

### Remote Commands for E87

Via S2F41:

| Command | Parameters | Description |
|---------|-----------|-------------|
| PORT_IN_SERVICE | PortID | Enable load port |
| PORT_OUT_OF_SERVICE | PortID | Disable load port |
| CARRIER_OUT | PortID | Request carrier removal |
| RESERVE_AT_PORT | PortID, CarrierID | Reserve port for carrier |
| CANCEL_RESERVATION | PortID | Cancel reservation |
| ACCESS | PortID | Open carrier for access |
| RELEASE | PortID | Close carrier after access |

---

## Scenarios & Message Flows

### Scenario 1: Complete Carrier Arrival and Processing

```
Host                                        Equipment
 │                                              │
 │  ══════ CARRIER ARRIVES ══════               │
 │                                              │
 │←── S6F11 (CarrierArrivedAtPort, LP1) ──────│  Carrier detected on LP1
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ══════ ID VERIFICATION ══════               │
 │                                              │
 │←── S6F11 (CarrierIDRead="FOUP001") ────────│  Barcode/RFID read
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S3F17 (PROCEED_WITH_CARRIER,             │
 │           CarrierID="FOUP001", Port=1) ─────→│  Host accepts
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CarrierIDVerified) ──────────────│  ID confirmed
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ══════ SLOT MAP ══════                      │
 │                                              │
 │←── S6F11 (SlotMapAvailable,                 │  
 │     ContentMap=[1,1,1,0,...,0]) ────────────│  3 wafers detected
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S3F17 (SLOT_MAP_READ, confirm map) ─────→│  Host confirms
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (SlotMapVerified) ────────────────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ══════ ACCESS CARRIER ══════                │
 │                                              │
 │─── S2F41 (ACCESS, PortID=1) ───────────────→│  Open FOUP door
 │←── S2F42 (HCACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CarrierAccessed) ────────────────│  Door open
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │  ══════ PROCESSING ══════                    │
 │  (Process jobs move substrates in/out)       │
 │                                              │
 │  ══════ COMPLETE ══════                      │
 │                                              │
 │─── S3F17 (CARRIER_CLOSED) ─────────────────→│  Done with carrier
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CarrierClosed) ──────────────────│  Door closed
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S2F41 (CARRIER_OUT, PortID=1) ──────────→│  Request removal
 │←── S2F42 (HCACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CarrierDeparted, LP1) ───────────│  Carrier removed
 │─── S6F12 ───────────────────────────────────→│
```

### Scenario 2: Carrier ID Verification Failure

```
Host                                        Equipment
 │                                              │
 │←── S6F11 (CarrierIDRead="UNKNOWN") ────────│  Cannot read ID
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │─── S3F17 (CANCEL_CARRIER, Port=1) ─────────→│  Host rejects carrier
 │←── S3F18 (CAACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 (CarrierIDVerifyFailed) ──────────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │   (Carrier must be removed by operator/AMHS) │
```

### Scenario 3: Carrier Pre-Notification (AMHS)

```
Host                                        Equipment
 │                                              │
 │─── S3F17 (CARRIER_NOTIFICATION,             │
 │     CarrierID="FOUP001", Port=1) ───────────→│  Carrier coming to LP1
 │←── S3F18 (CAACK=0) ────────────────────────│  Port prepared
 │                                              │
 │   ... AMHS delivers carrier ...              │
 │                                              │
 │←── S6F11 (CarrierArrivedAtPort) ───────────│  Physical arrival
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │   (ID verification skipped — already known)  │
 │                                              │
 │←── S6F11 (CarrierIDVerified) ──────────────│  Auto-verified
```

---

## Collection Events for E87

| Event | Description |
|-------|-------------|
| CarrierArrivedAtPort | Carrier physically detected at port |
| CarrierDeparted | Carrier removed from port |
| CarrierIDRead | Carrier ID barcode/RFID read |
| CarrierIDNotRead | Failed to read carrier ID |
| CarrierIDVerificationOK | Host confirmed carrier ID |
| CarrierIDVerificationFailed | ID mismatch or rejection |
| SlotMapAvailable | Slot map scan complete |
| SlotMapVerificationOK | Host confirmed slot map |
| SlotMapVerificationFailed | Slot map mismatch |
| CarrierAccessing | Carrier door opening |
| CarrierAccessed | Carrier door open, accessible |
| CarrierClosing | Carrier door closing |
| CarrierClosed | Carrier sealed |
| CarrierComplete | All processing done |
| CarrierStopped | Carrier processing stopped |
| PortStateChanged | Load port state changed |
| PortReservationChanged | Port reservation changed |

---

## Relationship to Other Standards

### Integration Map

```
┌────────────────────────────────────────────────────────────────┐
│                                                                 │
│  E84 (AMHS) ──→ Delivers carrier ──→ E87 (CMS) at Load Port  │
│                                              │                  │
│                                              ▼                  │
│                                    E90 (STS) tracks substrates │
│                                              │                  │
│                                              ▼                  │
│  E94 (CJM) ──→ Creates process jobs ──→ E40 (PJM) processes  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### E87 + E94 (Control Job)

- E94 Control Job references carriers
- CJ specifies which carrier(s) to process
- E87 manages the carrier arrival/departure
- CJ monitors carrier state for routing decisions

### E87 + E90 (Substrate Tracking)

- E87 provides ContentMap (which slots are occupied)
- E90 tracks individual substrate identity and location
- When carrier is accessed, E90 tracks substrate movement
- E87 updates ContentMap as substrates move

---

## Implementation Guidelines

### Minimum Compliance

Equipment must support:
1. At least one load port with state model
2. Carrier state tracking (arrival through departure)
3. Carrier ID reporting
4. Content map maintenance
5. Required collection events
6. S3F17/F18 for carrier actions
7. Carrier attributes accessible via S14F1/F2

### Configuration Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| Number of load ports | Physical ports | 1-6 |
| Default access mode | Manual or Auto | Auto |
| ID reader type | Barcode/RFID/OCR | Equipment-specific |
| Slot map sensor | Beam-break/camera | Equipment-specific |
| Carrier capacity | Slots per carrier | 13, 25 |
| Auto-verify | Skip host verification | Configurable |

### Error Handling

| Condition | Equipment Response |
|-----------|-------------------|
| ID read failure | Report IDNotRead event, wait for host |
| Slot map error (cross-slot) | Report error, request host decision |
| Carrier removed unexpectedly | Report CarrierDeparted, alert alarm |
| Port hardware failure | Port OUT OF SERVICE, alarm |
| Duplicate carrier ID | Report to host for resolution |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E87-0708 Standard*
