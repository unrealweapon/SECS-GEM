# SEMI E5 & E30 Standards — Comprehensive Reference

## Table of Contents

1. [Introduction](#introduction)
2. [SEMI E5 — SECS-II Standard](#semi-e5--secs-ii-standard)
   - [Overview & Purpose](#overview--purpose)
   - [Message Structure](#message-structure)
   - [Data Item Formats](#data-item-formats)
   - [Stream & Function Organization](#stream--function-organization)
   - [Complete Stream Reference](#complete-stream-reference)
3. [SEMI E30 — GEM Standard](#semi-e30--gem-standard)
   - [Overview & Purpose](#gem-overview--purpose)
   - [GEM Fundamental Requirements](#gem-fundamental-requirements)
   - [GEM State Models](#gem-state-models)
   - [GEM Capabilities](#gem-capabilities)
   - [GEM Scenarios & Message Flows](#gem-scenarios--message-flows)
4. [Relationship Between E5 and E30](#relationship-between-e5-and-e30)
5. [Implementation Guidelines](#implementation-guidelines)

---

## Introduction

The **SEMI** (Semiconductor Equipment and Materials International) standards define the communication protocols between semiconductor manufacturing equipment and factory host systems. Two of the most critical standards are:

| Standard | Full Name | Purpose |
|----------|-----------|---------|
| **SEMI E5** | SEMI Equipment Communications Standard 2 (SECS-II) | Defines the **message content and format** |
| **SEMI E30** | Generic Equipment Model (GEM) | Defines the **behavior and capabilities** required of equipment |

Together, these standards form the backbone of semiconductor factory automation (CIM — Computer Integrated Manufacturing).

### Standards Ecosystem

```
┌─────────────────────────────────────────────────────────────────┐
│                    Factory Host (MES/CIM)                        │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │  SEMI E30 (GEM Behavior)   │  ← Defines WHAT to do
                    ├───────────────────────────-┤
                    │  SEMI E5 (SECS-II Messages)│  ← Defines HOW to say it
                    ├────────────────────────────┤
                    │  SEMI E37 (HSMS Transport) │  ← Defines HOW to send it
                    │  or SEMI E4 (SECS-I Serial)│
                    └─────────────┬─────────────┘
                                  │
┌─────────────────────────────────┴───────────────────────────────┐
│                    Equipment Controller                           │
└──────────────────────────────────────────────────────────────────┘
```

---

## SEMI E5 — SECS-II Standard

### Overview & Purpose

**SEMI E5** (commonly called **SECS-II**) specifies:

- The **structure** of messages exchanged between host and equipment
- The **encoding** of data items within messages
- The **organization** of messages into Streams and Functions
- The **semantics** of each message (what it means and when to use it)

SECS-II is transport-independent — it defines message content regardless of whether SECS-I (RS-232) or HSMS (TCP/IP) is used for physical delivery.

### Message Structure

#### Message Header (10 bytes)

Every SECS-II message has a 10-byte header:

| Byte(s) | Field | Description |
|---------|-------|-------------|
| 0-1 | Device ID | Equipment identifier (15 bits + R-bit) |
| 2 | Stream | Message category (bits 0-6), W-bit (bit 7) |
| 3 | Function | Specific operation within the stream |
| 4-5 | PType/SType | Protocol type (always 0 for SECS-II) |
| 6-9 | System Bytes | Transaction ID for matching request/reply |

**Key bits:**
- **W-bit (Wait bit):** Set to 1 when a reply is expected
- **R-bit (Reply/Direction bit):** Indicates message direction (0 = to equipment, 1 = from equipment)

#### Message Body

The body consists of zero or more **data items**, each encoded as:

```
┌──────────────────────────────────────────────┐
│  Format Code (6 bits) │ Length Bytes (2 bits) │  ← Item Header
├──────────────────────────────────────────────┤
│  Length (1-3 bytes)                           │
├──────────────────────────────────────────────┤
│  Data (variable length)                      │
└──────────────────────────────────────────────┘
```

### Data Item Formats

SECS-II defines the following data formats:

| Format Code | Binary | Type | Description | Size |
|-------------|--------|------|-------------|------|
| 000000 | 0x00 | L (List) | Ordered list of items | Variable |
| 001000 | 0x08 | B (Binary) | Raw binary data | 1 byte/item |
| 001001 | 0x09 | Boolean | True/False | 1 byte |
| 010000 | 0x10 | A (ASCII) | ASCII string | 1 byte/char |
| 010001 | 0x11 | JIS-8 | Japanese string | 1 byte/char |
| 010010 | 0x12 | 2-byte Char | Unicode-like | 2 bytes/char |
| 011000 | 0x18 | I8 | 8-byte signed integer | 8 bytes |
| 011001 | 0x19 | I1 | 1-byte signed integer | 1 byte |
| 011010 | 0x1A | I2 | 2-byte signed integer | 2 bytes |
| 011100 | 0x1C | I4 | 4-byte signed integer | 4 bytes |
| 100000 | 0x20 | F8 | 8-byte floating point (IEEE 754) | 8 bytes |
| 100100 | 0x24 | F4 | 4-byte floating point (IEEE 754) | 4 bytes |
| 101000 | 0x28 | U8 | 8-byte unsigned integer | 8 bytes |
| 101001 | 0x29 | U1 | 1-byte unsigned integer | 1 byte |
| 101010 | 0x2A | U2 | 2-byte unsigned integer | 2 bytes |
| 101100 | 0x2C | U4 | 4-byte unsigned integer | 4 bytes |

#### Data Item Notation Convention

In SECS-II documentation, messages are described using this notation:

```
{L:n           ← List with n items
  <A "text">   ← ASCII string with value "text"
  <U4 100>     ← Unsigned 4-byte integer with value 100
  <B 0x01>     ← Binary byte with value 0x01
  <BOOLEAN T>  ← Boolean TRUE
}
```

### Stream & Function Organization

Messages are organized into **Streams** (categories) and **Functions** (operations):

- **Stream:** A logical grouping of related messages (0-127)
- **Function:** A specific operation within a stream (0-255)
- **Naming:** `SxFy` where x = stream, y = function

#### Conventions

| Rule | Description |
|------|-------------|
| Odd functions | Primary messages (requests/commands) |
| Even functions | Secondary messages (replies/acknowledgments) |
| Function 0 | Abort transaction (for any stream) |
| `R` suffix | Reply is **required** |
| `[R]` suffix | Reply is **optional** |
| `W-bit = 1` | Sender expects a reply |

### Complete Stream Reference

---

#### Stream 1 — Equipment Status

Provides the ability to determine equipment status and identity.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S1F0 | Abort | H↔E | Abort S1 transaction |
| S1F1/F2 | Are You There / On Line Data | H↔E | Communication liveness check |
| S1F3/F4 | Selected Equipment Status Request/Data | H→E | Request specific status variables |
| S1F5/F6 | Formatted Status Request/Data | H→E | Request status in formatted form |
| S1F11/F12 | Status Variable Namelist Request/Reply | H→E | Get names/units of SVIDs |
| S1F13/F14 | Establish Communications Request/Ack | H↔E | Initialize communication |
| S1F15/F16 | Request OFF-LINE / Ack | H→E | Take equipment offline |
| S1F17/F18 | Request ON-LINE / Ack | H→E | Bring equipment online |
| S1F21/F22 | Data Variable Namelist Request/Reply | H→E | Get names/units of DVALs |
| S1F23/F24 | Collection Event Namelist Request/Reply | H→E | Get event names and linked VIDs |

**Key Data Items:**
- **SVID** — Status Variable ID
- **SV** — Status Variable value
- **MDLN** — Equipment Model Type
- **SOFTREV** — Software Revision
- **COMMACK** — Establish Communications Acknowledge (0=accepted, 1=denied)
- **OFLACK** — Off-Line Acknowledge (0=accepted)
- **ONLACK** — On-Line Acknowledge (0=accepted, 1=refused, 2=already online)

---

#### Stream 2 — Equipment Control and Diagnostics

Provides remote control, configuration, and diagnostic capabilities.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S2F13/F14 | Equipment Constant Request/Data | H→E | Read equipment constants |
| S2F15/F16 | New Equipment Constant Send/Ack | H→E | Write equipment constants |
| S2F17/F18 | Date and Time Request/Data | H↔E | Synchronize time |
| S2F21/F22 | Remote Command Send/Ack | H→E | Execute remote command |
| S2F23/F24 | Trace Initialize Send/Ack | H→E | Set up data tracing |
| S2F25/F26 | Loopback Diagnostic Request/Data | H↔E | Communication test |
| S2F29/F30 | Equipment Constant Namelist Request/Reply | H→E | Get EC metadata |
| S2F31/F32 | Date and Time Set Request/Ack | H→E | Set equipment clock |
| S2F33/F34 | Define Report / Ack | H→E | Define report structure |
| S2F35/F36 | Link Event Report / Ack | H→E | Link reports to events |
| S2F37/F38 | Enable/Disable Event Report / Ack | H→E | Control event reporting |
| S2F39/F40 | Multi-block Inquire/Grant | H→E | Request permission for multi-block |
| S2F41/F42 | Host Command Send/Ack | H→E | Enhanced remote command (with params) |
| S2F43/F44 | Reset Spooling / Ack | H→E | Configure spooling |
| S2F45/F46 | Define Variable Limit Attributes / Ack | H→E | Set variable limits |
| S2F47/F48 | Variable Limit Attribute Request/Data | H→E | Query variable limits |
| S2F49/F50 | Enhanced Remote Command / Ack | H→E | Remote command with OBJSPEC |

**Key Data Items:**
- **ECID** — Equipment Constant ID
- **ECV** — Equipment Constant Value
- **RCMD** — Remote Command name
- **CPSNAME/CPVAL** — Command Parameter Name/Value
- **DATAID** — Data ID for reports
- **RPTID** — Report ID
- **VID** — Variable ID
- **CEID** — Collection Event ID
- **HCACK** — Host Command Acknowledge code
- **DRACK** — Define Report Acknowledge
- **LRACK** — Link Report Acknowledge
- **ERACK** — Enable Report Acknowledge

**S2F41 HCACK Codes:**

| Code | Meaning |
|------|---------|
| 0 | Acknowledge, command has been performed |
| 1 | Command does not exist |
| 2 | Cannot perform now |
| 3 | At least one parameter is invalid |
| 4 | Acknowledge, command will be performed with completion signaled later |
| 5 | Rejected, already in desired condition |
| 6 | No such object exists |

---

#### Stream 3 — Material Status

Provides material (lot, carrier, substrate) identification and tracking.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S3F1/F2 | Material Status Request/Data | H→E | Query material at equipment |
| S3F3/F4 | Material ID Send/Ack | E→H | Report material arrival |
| S3F5/F6 | Material Move Request/Ack | H→E | Request material movement |
| S3F17/F18 | Carrier Action Request/Ack | H↔E | Carrier-level operations |

---

#### Stream 4 — Material Control

Controls material transfer between equipment and transport systems.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S4F1/F2 | Ready To Transfer Request/Ack | E→H | Equipment ready for transfer |
| S4F19/F20 | Transfer Job Create/Ack | H→E | Create transfer job |
| S4F21/F22 | Transfer Job Command/Ack | H→E | Control transfer job |
| S4F25/F26 | Multi-Carrier Transfer Request/Reply | H→E | Transfer multiple carriers |

---

#### Stream 5 — Exception Handling (Alarms)

Manages alarm notification and recovery.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S5F1/F2 | Alarm Report Send/Ack | E→H | Report alarm occurrence |
| S5F3/F4 | Enable/Disable Alarm Send/Ack | H→E | Control alarm reporting |
| S5F5/F6 | List Alarms Request/Data | H→E | Query defined alarms |
| S5F7/F8 | List Enabled Alarms Request/Data | H→E | Query enabled alarms |

**Alarm Structure (S5F1):**
```
{L:3
  <B ALCD>       ← Alarm Code (bit 7: set=alarm set, clear=alarm cleared)
  <U4 ALID>      ← Alarm ID
  <A ALTX>       ← Alarm Text description
}
```

**ALCD Bit Definitions:**

| Bit | Meaning |
|-----|---------|
| 7 | 1=Alarm Set, 0=Alarm Cleared |
| 0-2 | Alarm Category (0=not used, 1=personal safety, 2=equipment safety, 3-7=categories) |

---

#### Stream 6 — Data Collection

Manages event-driven and on-demand data collection.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S6F1/F2 | Trace Data Send/Ack | E→H | Send traced data |
| S6F5/F6 | Multi-block Data Send Inquire/Grant | E→H | Request multi-block send |
| S6F11/F12 | Event Report Send/Ack | E→H | Send event report data |
| S6F15/F16 | Event Report Request/Data | H→E | On-demand event data |
| S6F19/F20 | Individual Report Request/Data | H→E | Request single report |

**S6F11 Event Report Structure:**
```
{L:3
  <U4 DATAID>
  <U4 CEID>
  {L:n                    ← Reports linked to this event
    {L:2
      <U4 RPTID>
      {L:m                ← Variables in this report
        <V>               ← Variable values (SV, DVVAL, etc.)
      }
    }
  }
}
```

---

#### Stream 7 — Process Program Management

Manages recipe/process program transfer between host and equipment.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S7F1/F2 | Process Program Load Inquire/Grant | H→E | Request to upload recipe |
| S7F3/F4 | Process Program Send/Ack | H→E | Send recipe to equipment |
| S7F5/F6 | Process Program Request/Data | H→E | Download recipe from equipment |
| S7F17/F18 | Delete Process Program Send/Ack | H→E | Delete recipe |
| S7F19/F20 | Current EPPD Request/Data | H→E | List recipes on equipment |
| S7F23/F24 | Formatted Process Program Send/Ack | H↔E | Formatted recipe transfer |
| S7F25/F26 | Formatted Process Program Request/Data | H↔E | Request formatted recipe |

**Key Data Items:**
- **PPID** — Process Program ID (recipe name)
- **PPBODY** — Process Program Body (recipe content)
- **PPGNT** — Process Program Grant (0=OK, 1=already have, 2=no space, 3=invalid PPID)
- **ACKC7** — Stream 7 Acknowledge (0=accepted, 1-63=various errors)

---

#### Stream 8 — Control Program Transfer

Manages equipment boot and executive program transfer (less commonly used in modern equipment).

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S8F1/F2 | Control Program Load Inquire/Grant | H→E | Request executive load |
| S8F3/F4 | Control Program Send/Ack | H→E | Send control program |

---

#### Stream 9 — System Errors

Equipment reports communication and protocol errors to the host.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S9F1 | Unrecognized Device ID | E→H | Unknown device in header |
| S9F3 | Unrecognized Stream Type | E→H | Unknown stream number |
| S9F5 | Unrecognized Function Type | E→H | Unknown function number |
| S9F7 | Illegal Data | E→H | Bad data in message body |
| S9F9 | Transaction Timer Timeout | E→H | T3 timeout |
| S9F11 | Data Too Long | E→H | Message exceeds max size |
| S9F13 | Conversation Timeout | E→H | Multi-block conversation timeout |

---

#### Stream 10 — Terminal Services

Provides text display capabilities on the equipment terminal.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S10F1/F2 | Terminal Request/Ack | H→E | Display text on equipment |
| S10F3/F4 | Terminal Display, Single/Ack | H→E | Display on specific terminal |
| S10F5/F6 | Terminal Display, Multi-block/Ack | H→E | Multi-block terminal display |
| S10F9/F10 | Broadcast/Ack | H→E | Broadcast message to all terminals |

**S10F1 Structure:**
```
{L:2
  <B TID>       ← Terminal ID
  <A TEXT>      ← Text to display
}
```

---

#### Stream 12 — Wafer Mapping

Supports wafer die map data exchange (used in wafer probing/sorting).

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S12F1/F2 | Map Setup Data Send/Ack | E→H | Send map coordinate setup |
| S12F3/F4 | Map Setup Data Request/Data | H→E | Request map setup |
| S12F5/F6 | Map Transmit Inquire/Grant | E→H | Request to send map |
| S12F7/F8 | Map Data Send Type 1/Ack | E→H | Send coordinate map |
| S12F9/F10 | Map Data Send Type 2/Ack | E→H | Send row/column map |
| S12F14/F15 | Map Data Request Type 1/Data | H→E | Request coordinate map |
| S12F16/F17 | Map Data Request Type 2/Data | H→E | Request row/column map |

---

#### Stream 14 — Object Services

Provides generalized object attribute management.

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S14F1/F2 | GetAttr Request/Data | H↔E | Get object attributes |
| S14F3/F4 | SetAttr Request/Data | H↔E | Set object attributes |
| S14F9/F10 | Create Object Request/Ack | H↔E | Create new object |
| S14F11/F12 | Delete Object Request/Ack | H↔E | Delete object |

---

#### Stream 16 — Processing Management

Controls process jobs on equipment (used heavily with GEM300).

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S16F1/F2 | Process Job Create Request/Ack | H→E | Create process job |
| S16F5/F6 | Process Job Command Request/Ack | H→E | Control process job (START, STOP, ABORT) |
| S16F9/F10 | Process Job State Query/Data | H→E | Query process job status |
| S16F11/F12 | PRJobCreate | H→E | Multi-recipe process job |
| S16F15/F16 | PRJob Multi-Create/Ack | H→E | Create multiple jobs |

---

## SEMI E30 — GEM Standard

### GEM Overview & Purpose

**SEMI E30** (Generic Equipment Model — **GEM**) defines the **behavior** that semiconductor equipment must implement to communicate with a factory host system. While SECS-II (E5) defines the message formats, GEM specifies:

- **What capabilities** equipment must support
- **When and how** to use SECS-II messages
- **State machine models** for equipment behavior
- **Required vs. optional** functionality
- **Compliance criteria** for equipment certification

GEM is built on top of SECS-II and uses SECS-II messages as its communication mechanism.

### GEM Fundamental Requirements

GEM defines capabilities in two categories:

#### Fundamental GEM Requirements (MUST implement)

| # | Capability | Description |
|---|-----------|-------------|
| 1 | State Models | Communication and Control state models |
| 2 | Equipment Processing States | Define processing states visible to host |
| 3 | Host-Initiated S1F13/F14 Scenario | Respond to establishment requests |
| 4 | Event Notification | Report events to host |
| 5 | On-Line Identification | Provide MDLN and SOFTREV |
| 6 | Error Messages | Report communication errors (S9) |
| 7 | Documentation | Provide GEM compliance statement |
| 8 | Control (Operator Initiated) | Support offline/online from operator |

#### Additional GEM Capabilities (Conditional/Optional)

| # | Capability | Description |
|---|-----------|-------------|
| 9 | Establish Communications | Equipment-initiated S1F13 |
| 10 | Dynamic Event Report Configuration | S2F33/35/37 support |
| 11 | Variable Data Collection | Status Variables and Data Variables |
| 12 | Trace Data Collection | S2F23 trace setup |
| 13 | Status Data Collection | S1F3/4 status requests |
| 14 | Alarm Management | S5F1-8 alarm support |
| 15 | Remote Control | S2F21/41/49 remote commands |
| 16 | Equipment Constants | S2F13-16 EC management |
| 17 | Process Program Management | S7 recipe management |
| 18 | Material Movement | S3/S4 material handling |
| 19 | Equipment Terminal Services | S10 terminal display |
| 20 | Clock | S2F17/18 and S2F31/32 time sync |
| 21 | Limits Monitoring | Variable limit checking |
| 22 | Spooling | S2F43/44 message spooling |
| 23 | Control (Host Initiated) | S1F15-18 online/offline from host |

---

### GEM State Models

GEM defines several mandatory state machines:

#### 1. Communication State Model

```
┌─────────────────────────────────────────────────────────────┐
│                    COMMUNICATING                              │
│                                                              │
│  ┌────────────────┐         ┌────────────────────────────┐  │
│  │ WAIT CRA       │ ──────→ │ WAIT DELAY                 │  │
│  │ (Wait Commun.  │ timeout │ (Wait before retry)        │  │
│  │  Request Ack)  │ ←────── │                            │  │
│  └────────────────┘         └────────────────────────────┘  │
│          │                                                   │
│          │ S1F13/F14 success                                 │
│          ▼                                                   │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              COMMUNICATING                               │  │
│  │              (Normal Operation)                          │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

States:
  ┌──────────────────────┐       ┌────────────────────┐
  │  DISABLED            │ ────→ │  ENABLED            │
  │  (Not Communicating) │ ←──── │  (Communicating)    │
  └──────────────────────┘       └────────────────────┘
```

**Communication State Transitions:**

| Current State | Trigger | New State |
|---------------|---------|-----------|
| DISABLED | Enable | ENABLED — NOT COMMUNICATING |
| NOT COMMUNICATING | S1F13 received or sent successfully | COMMUNICATING |
| COMMUNICATING | Communication failure | NOT COMMUNICATING |
| ENABLED | Disable | DISABLED |

---

#### 2. Control State Model

This is the **most critical** GEM state model. It determines what level of control the host has over the equipment.

```
                    ┌─────────────────────────┐
                    │     EQUIPMENT OFF-LINE   │
                    │                          │
                    │  ┌───────────────────┐   │
                    │  │ EQ OFF-LINE       │   │
                    │  │ (attempt online)  │   │
                    │  └────────┬──────────┘   │
                    │           │              │
                    │  ┌────────┴──────────┐   │
                    │  │ HOST OFF-LINE     │   │
                    │  │ (host forced)     │   │
                    │  └───────────────────┘   │
                    └────────────┬──────────────┘
                                 │ Transition to ON-LINE
                                 ▼
                    ┌─────────────────────────┐
                    │     EQUIPMENT ON-LINE    │
                    │                          │
                    │  ┌───────────────────┐   │
                    │  │    ON-LINE LOCAL  │   │  ← Operator has control
                    │  └────────┬──────────┘   │
                    │           │              │
                    │  ┌────────┴──────────┐   │
                    │  │   ON-LINE REMOTE  │   │  ← Host has control
                    │  └───────────────────┘   │
                    └──────────────────────────┘
```

**Control States Detail:**

| State | Description | Host Commands | Operator Commands |
|-------|-------------|---------------|-------------------|
| **Equipment Offline** | Equipment refuses host communication | None | Full control |
| **Attempt Online** | Transitioning to online | None | Can abort |
| **Host Offline** | Host explicitly set equipment offline | S1F17 to go online | Limited |
| **Online Local** | Online but operator has priority | Limited | Full control |
| **Online Remote** | Host has full control | Full control | Emergency only |

**Control State Transition Triggers:**

| Transition | Trigger Message | Description |
|-----------|----------------|-------------|
| Offline → Online | S1F17 (Host) or Operator | Request to go online |
| Online → Offline | S1F15 (Host) or Operator | Request to go offline |
| Local → Remote | Operator panel switch | Give host control |
| Remote → Local | Operator panel switch | Take back control |

---

#### 3. Processing State Model (Equipment-Specific)

GEM requires equipment to expose its processing states. While the specific states vary by equipment type, GEM defines a typical model:

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│   IDLE   │────→│  SETUP   │────→│EXECUTING │────→│  IDLE    │
│          │     │          │     │          │     │          │
└──────────┘     └──────────┘     └────┬─────┘     └──────────┘
                                       │
                                       ▼
                                  ┌──────────┐
                                  │  PAUSE   │
                                  └──────────┘
```

Common processing states:

| State | Description |
|-------|-------------|
| IDLE | No processing, waiting for job |
| SETUP | Preparing for processing (recipe load, etc.) |
| READY | Setup complete, waiting to start |
| EXECUTING | Actively processing material |
| PAUSE | Processing temporarily suspended |
| ABORTING | Abnormal termination in progress |
| STOPPING | Normal termination in progress |

---

### GEM Capabilities — Detailed

#### 1. State Models (Fundamental)

Equipment **MUST** implement:
- Communication State Model
- Control State Model
- Processing State Model (equipment-specific)

**Required Messages:**
- S1F1/F2, S1F13/F14, S1F15/F16, S1F17/F18
- S6F11/F12 (for state change events)

---

#### 2. Event Notification (Fundamental)

Equipment must report significant events to the host when requested.

**Mechanism:**
1. Host defines reports: `S2F33` — which variables to include in a report
2. Host links reports to events: `S2F35` — which reports fire on which events
3. Host enables events: `S2F37` — turn on/off specific events
4. Equipment sends events: `S6F11` — when enabled event occurs

**Standard Collection Events (CEIDs):**

| Event | Description | Typical CEID |
|-------|-------------|--------------|
| EquipmentOffline | Equipment went offline | Equipment-specific |
| ControlStateChange | Control state changed | Equipment-specific |
| ProcessingStarted | Processing began | Equipment-specific |
| ProcessingCompleted | Processing finished | Equipment-specific |
| ProcessingStopped | Processing stopped abnormally | Equipment-specific |
| AlarmSet | Alarm activated | Equipment-specific |
| AlarmCleared | Alarm deactivated | Equipment-specific |
| PPChange | Process program changed | Equipment-specific |
| MaterialReceived | Material arrived at port | Equipment-specific |
| MaterialRemoved | Material left port | Equipment-specific |
| JobStarted | Process job started | Equipment-specific |
| JobCompleted | Process job finished | Equipment-specific |

**Event Report Configuration Flow:**

```
Host                                        Equipment
 │                                              │
 │─── S2F33 (Define Report) ──────────────────→│  Define RPTID with VIDs
 │←── S2F34 (DRACK=0) ────────────────────────│  Acknowledged
 │                                              │
 │─── S2F35 (Link Event Report) ──────────────→│  Link RPTID to CEID
 │←── S2F36 (LRACK=0) ────────────────────────│  Acknowledged
 │                                              │
 │─── S2F37 (Enable Event) ───────────────────→│  Enable CEID
 │←── S2F38 (ERACK=0) ────────────────────────│  Acknowledged
 │                                              │
 │          ... time passes, event occurs ...    │
 │                                              │
 │←── S6F11 (Event Report) ───────────────────│  Event data sent
 │─── S6F12 (Ack) ────────────────────────────→│  Acknowledged
```

---

#### 3. Alarm Management

Equipment reports alarm conditions (set and cleared) to the host.

**Alarm Categories:**

| Category | Value | Description |
|----------|-------|-------------|
| Not Used | 0 | Category not applicable |
| Personal Safety | 1 | Risk to personnel |
| Equipment Safety | 2 | Risk to equipment |
| Parameter Control Warning | 3 | Process parameter out of range |
| Parameter Control Error | 4 | Parameter exceeded error limit |
| Irrecoverable Error | 5 | Equipment needs maintenance |
| Equipment Status Warning | 6 | Equipment approaching limit |
| Attention Flags | 7 | Informational |
| Data Integrity | 8 | Data corruption concern |

**Alarm Flow:**

```
Host                                        Equipment
 │                                              │
 │─── S5F3 (Enable Alarm ALID=xx) ───────────→│  Enable specific alarm
 │←── S5F4 (ACKC5=0) ─────────────────────────│  Acknowledged
 │                                              │
 │          ... alarm condition occurs ...       │
 │                                              │
 │←── S5F1 (Alarm Set, ALCD bit7=1) ─────────│  Alarm activated
 │─── S5F2 (Ack) ─────────────────────────────→│  Acknowledged
 │                                              │
 │          ... alarm condition clears ...       │
 │                                              │
 │←── S5F1 (Alarm Clear, ALCD bit7=0) ────────│  Alarm deactivated
 │─── S5F2 (Ack) ─────────────────────────────→│  Acknowledged
```

---

#### 4. Remote Control

Allows the host to send commands to the equipment when in **Online Remote** state.

**Three levels of remote commands:**

| Message | Level | Description |
|---------|-------|-------------|
| S2F21/F22 | Basic | Simple command name, no parameters |
| S2F41/F42 | Enhanced | Command with named parameters |
| S2F49/F50 | Extended | Command with object specification |

**S2F41 Structure (Enhanced Remote Command):**
```
{L:2
  <A RCMD>              ← Command name (e.g., "START", "STOP", "PP-SELECT")
  {L:n                  ← Parameters
    {L:2
      <A CPNAME>        ← Parameter name
      <A/U4/... CPVAL>  ← Parameter value
    }
  }
}
```

**Common GEM Remote Commands:**

| Command | Parameters | Description |
|---------|-----------|-------------|
| START | None or LOT_ID | Start processing |
| STOP | None | Stop processing gracefully |
| ABORT | None | Abort processing immediately |
| CANCEL | None | Cancel pending job |
| PAUSE | None | Pause processing |
| RESUME | None | Resume from pause |
| PP-SELECT | PPID, LOT_ID | Select recipe for next run |
| LOCAL | None | Switch to local mode |
| REMOTE | None | Switch to remote mode |

---

#### 5. Equipment Constants (EC)

Equipment constants are configurable parameters that affect equipment operation.

**Characteristics:**
- Persist across power cycles
- Changed by host (S2F15) or operator
- Have min/max/default values
- Change takes effect immediately or on next cycle

**EC Management Flow:**

```
Host                                        Equipment
 │                                              │
 │─── S2F29 (EC Namelist Request) ────────────→│  Get all EC definitions
 │←── S2F30 (EC Namelist Data) ────────────────│  ECID, name, min, max, default, units
 │                                              │
 │─── S2F13 (EC Request [ECID list]) ─────────→│  Read specific constants
 │←── S2F14 (EC Data [values]) ────────────────│  Current values
 │                                              │
 │─── S2F15 (New EC Send) ────────────────────→│  Write new values
 │←── S2F16 (EAC=0) ──────────────────────────│  Accepted
```

**EAC (Equipment Acknowledge Code) Values:**

| EAC | Meaning |
|-----|---------|
| 0 | Acknowledged, values accepted |
| 1 | Denied, at least one constant does not exist |
| 2 | Denied, busy |
| 3 | Denied, at least one value out of range |

---

#### 6. Process Program Management

Manages recipes/process programs stored on equipment.

**Program Types:**
- **Unformatted** (S7F3/F5): Opaque binary blob
- **Formatted** (S7F23/F25): Structured, host-readable

**PP Management Flow:**

```
Host                                        Equipment
 │                                              │
 │─── S7F19 (List PP Request) ────────────────→│  Get recipe list
 │←── S7F20 (PP List Data) ───────────────────│  List of PPIDs
 │                                              │
 │─── S7F1 (PP Load Inquire) ─────────────────→│  Request to send recipe
 │←── S7F2 (PPGNT=0, permission granted) ─────│  Space available
 │                                              │
 │─── S7F3 (PP Send: PPID + PPBODY) ──────────→│  Upload recipe
 │←── S7F4 (ACKC7=0) ─────────────────────────│  Accepted
 │                                              │
 │─── S7F5 (PP Request: PPID) ────────────────→│  Download recipe
 │←── S7F6 (PP Data: PPID + PPBODY) ──────────│  Recipe content
 │                                              │
 │─── S7F17 (PP Delete: PPID) ────────────────→│  Delete recipe
 │←── S7F18 (ACKC7=0) ────────────────────────│  Deleted
```

---

#### 7. Spooling

When communication is lost, equipment stores (spools) messages for later delivery.

**Spooling Mechanism:**
1. Host configures which messages to spool: `S2F43`
2. Communication is lost
3. Equipment stores configured messages in spool buffer
4. Communication is restored
5. Equipment sends spooled messages to host
6. Normal operation resumes

**Spool Configuration (S2F43):**
```
{L:n
  {L:2
    <U1 STRID>          ← Stream to spool
    {L:m
      <U1 FCNID>       ← Functions to spool within stream
    }
  }
}
```

**Typical Spooled Messages:**
- S5F1 (Alarms)
- S6F11 (Event Reports)

---

#### 8. Clock Synchronization

Equipment time must be synchronized with the factory host.

**Time Format (per E5):**
```
YYYYMMDDHHMMSScc
```
Where:
- YYYY = 4-digit year
- MM = month (01-12)
- DD = day (01-31)
- HH = hour (00-23)
- MM = minute (00-59)
- SS = second (00-59)
- cc = centiseconds (00-99)

**Clock Sync Flow:**
```
Host                                        Equipment
 │                                              │
 │─── S2F31 (Set Time) ───────────────────────→│  Set equipment clock
 │←── S2F32 (TIACK=0) ────────────────────────│  Accepted
 │                                              │
 │─── S2F17 (Time Request) ───────────────────→│  Query equipment time
 │←── S2F18 (TIME="20260521143000") ──────────│  Current time
```

---

#### 9. Variable Limits Monitoring

Equipment monitors variables against defined limits and generates events when thresholds are crossed.

**Limit Zones:**

```
      ┌─────────────────── UPPER DEADBAND ───────────────────┐
      │                                                       │
   ───┼── UPPER LIMIT ──────────────────────────────────────┼───
      │                                                       │
      │              NORMAL OPERATING ZONE                    │
      │                                                       │
   ───┼── LOWER LIMIT ──────────────────────────────────────┼───
      │                                                       │
      └─────────────────── LOWER DEADBAND ───────────────────┘
```

**Limit Configuration (S2F45):**
```
{L:n
  {L:3
    <U4 VID>              ← Variable to monitor
    {L:p                  ← Limit sets
      {L:5
        <U1 LIMITID>      ← Limit identifier
        <F4 UPPERDB>      ← Upper deadband
        <F4 LOWERDB>      ← Lower deadband
        <F4 UPPERLIMIT>   ← Upper limit value
        <F4 LOWERLIMIT>   ← Lower limit value
      }
    }
  }
}
```

---

#### 10. Equipment Terminal Services

Allows host to display messages on equipment operator terminal.

**Use Cases:**
- Display lot instructions to operator
- Show status messages
- Display alarm descriptions
- Broadcast factory-wide announcements

---

### GEM Scenarios & Message Flows

#### Scenario 1: Full Communication Establishment

```
Power On
   │
   ▼
Equipment: Enter DISABLED state
   │
   │ (Communication enabled)
   ▼
Equipment: Enter NOT COMMUNICATING state
   │
   │ Send S1F13 (Establish Communications Request)
   ▼
Host: Receive S1F13
   │
   │ Send S1F14 (COMMACK=0, accepted)
   ▼
Equipment: Enter COMMUNICATING state
   │
   │ (Equipment is now ONLINE — LOCAL or REMOTE per configuration)
   ▼
Host: Send S2F17 (Date/Time Request)
   │
Equipment: Reply S2F18 (current time)
   │
Host: Send S2F31 (Set Time) if needed
   │
Host: Send S1F3 (Status Request) — verify equipment state
   │
Host: Send S2F33/S2F35/S2F37 (Configure event reports)
   │
   ▼
Normal Operation Begins
```

---

#### Scenario 2: Process Job Execution

```
Host                                        Equipment
 │                                              │
 │─── S7F1/F3 (Upload Recipe if needed) ──────→│
 │                                              │
 │─── S2F41 RCMD="PP-SELECT"                   │
 │         CPNAME="PPID", CPVAL="recipe1" ────→│  Select recipe
 │←── S2F42 (HCACK=0) ────────────────────────│
 │                                              │
 │─── S2F41 RCMD="START" ─────────────────────→│  Start processing
 │←── S2F42 (HCACK=0) ────────────────────────│
 │                                              │
 │←── S6F11 CEID=ProcessingStarted ───────────│  Processing started event
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │          ... processing occurs ...            │
 │                                              │
 │←── S6F11 CEID=ProcessingCompleted ─────────│  Processing done event
 │─── S6F12 ───────────────────────────────────→│
```

---

#### Scenario 3: Alarm Handling

```
Host                                        Equipment
 │                                              │
 │          ... abnormal condition ...           │
 │                                              │
 │←── S5F1 (ALCD=0x80|cat, ALID, ALTX) ──────│  Alarm SET
 │─── S5F2 ───────────────────────────────────→│
 │                                              │
 │←── S6F11 (AlarmSet event with details) ────│  Event with alarm data
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │          ... condition corrected ...          │
 │                                              │
 │←── S5F1 (ALCD=0x00|cat, ALID, ALTX) ──────│  Alarm CLEARED
 │─── S5F2 ───────────────────────────────────→│
 │                                              │
 │←── S6F11 (AlarmCleared event) ─────────────│  Event with clear data
 │─── S6F12 ───────────────────────────────────→│
```

---

#### Scenario 4: Recovery from Communication Loss

```
Host                                        Equipment
 │                                              │
 │          ═══ COMMUNICATION LOST ═══          │
 │                                              │
 │     (Host unreachable)                       │  Detect T6/T7 timeout
 │                                              │  Enter NOT COMMUNICATING
 │                                              │  Begin SPOOLING messages
 │                                              │
 │          ... time passes, events occur ...    │  S5F1, S6F11 → spool buffer
 │                                              │
 │          ═══ COMMUNICATION RESTORED ═══      │
 │                                              │
 │←── S1F13 (Establish Comms) ────────────────│  Request reconnection
 │─── S1F14 (COMMACK=0) ──────────────────────→│  Accepted
 │                                              │
 │←── S5F1 (Spooled alarm 1) ─────────────────│  Deliver spooled messages
 │─── S5F2 ───────────────────────────────────→│
 │←── S6F11 (Spooled event 1) ────────────────│
 │─── S6F12 ───────────────────────────────────→│
 │←── S6F11 (Spooled event 2) ────────────────│
 │─── S6F12 ───────────────────────────────────→│
 │                                              │
 │     (Spool empty, normal operation)          │
```

---

## Relationship Between E5 and E30

| Aspect | SEMI E5 (SECS-II) | SEMI E30 (GEM) |
|--------|-------------------|----------------|
| **Defines** | Message format & content | Equipment behavior |
| **Level** | Protocol/Syntax | Application/Semantics |
| **Scope** | All possible messages | Required subset + behavior |
| **Analogy** | Dictionary of words | Grammar rules for sentences |
| **Without other** | Messages have no context | Behavior has no communication means |
| **Focus** | Data encoding, streams, functions | State models, scenarios, requirements |
| **Versioning** | Rarely changes (mature) | Updated for new capabilities |

### How They Work Together

1. **GEM** says: "Equipment must report alarms to the host"
2. **SECS-II** says: "Use S5F1 with this specific data format"
3. **GEM** says: "Equipment must implement the Control State Model"  
4. **SECS-II** says: "Use S1F15/F16 for offline, S1F17/F18 for online"

---

## Implementation Guidelines

### GEM Compliance Checklist

#### Minimum (Fundamental) Requirements

- [ ] Communication State Model implemented
- [ ] Control State Model implemented (all states reachable)
- [ ] Equipment Processing States defined and reported
- [ ] S1F13/F14 host-initiated supported
- [ ] At least one collection event implemented (state change)
- [ ] S6F11 sent for enabled events
- [ ] Equipment provides MDLN and SOFTREV
- [ ] S9Fx error messages sent for protocol violations
- [ ] GEM Compliance Statement document created
- [ ] Operator can switch online/offline

#### Common Additional Requirements

- [ ] Equipment-initiated S1F13 on power-up
- [ ] Dynamic event report configuration (S2F33/35/37)
- [ ] Status Variables readable (S1F3/F4)
- [ ] Data Variables available in reports
- [ ] Alarms reported (S5F1/F2) with enable/disable
- [ ] Remote commands supported (S2F41/F42)
- [ ] Equipment constants manageable (S2F13-16)
- [ ] Process programs uploadable/downloadable (S7)
- [ ] Clock sync supported (S2F17/18, S2F31/32)
- [ ] Spooling configured and functional

### Common Data Items Reference

| Data Item | Format | Description |
|-----------|--------|-------------|
| ALCD | Binary/U1 | Alarm code (set/clear + category) |
| ALID | U4 | Alarm ID |
| ALTX | ASCII | Alarm text description |
| CEED | Boolean | Collection event enable/disable |
| CEID | U4 | Collection Event ID |
| CENAME | ASCII | Collection Event Name |
| COMMACK | Binary | Communication acknowledge (0=OK, 1=denied) |
| CPNAME | ASCII | Command parameter name |
| CPVAL | Varies | Command parameter value |
| DATAID | U4 | Data ID |
| DRACK | Binary | Define Report acknowledge |
| EAC | Binary | Equipment acknowledge code |
| ECID | U4 | Equipment Constant ID |
| ECNAME | ASCII | Equipment Constant name |
| ECV | Varies | Equipment Constant value |
| ERACK | Binary | Enable/disable event acknowledge |
| HCACK | Binary | Host command acknowledge |
| LRACK | Binary | Link event report acknowledge |
| MDLN | ASCII | Equipment model type |
| OFLACK | Binary | Offline acknowledge |
| ONLACK | Binary | Online acknowledge |
| PPID | ASCII | Process Program ID |
| PPBODY | Binary/List | Process Program body |
| RCMD | ASCII | Remote command name |
| RPTID | U4 | Report ID |
| SOFTREV | ASCII | Software revision |
| SV | Varies | Status Variable value |
| SVID | U4 | Status Variable ID |
| SVNAME | ASCII | Status Variable name |
| TIME | ASCII[16] | Timestamp (YYYYMMDDHHMMSScc) |
| UNITS | ASCII | Engineering units |
| VID | U4 | Variable ID |

### Timer Reference

GEM/SECS defines several communication timers:

| Timer | Default | Description |
|-------|---------|-------------|
| T1 | 1 sec | Inter-character timeout (SECS-I only) |
| T2 | 10 sec | Protocol timeout (SECS-I only) |
| T3 | 45 sec | Reply timeout (time to wait for reply) |
| T4 | 45 sec | Inter-block timeout (SECS-I multi-block) |
| T5 | 10 sec | Connect separation timeout (HSMS) |
| T6 | 5 sec | Control transaction timeout (HSMS) |
| T7 | 10 sec | Not Selected timeout (HSMS) |
| T8 | 5 sec | Network intercharacter timeout (HSMS) |

### GEM Variable Types

| Type | Abbreviation | Description | Access |
|------|-------------|-------------|--------|
| Status Variable | SV | Real-time equipment state | Read-only (S1F3/F4) |
| Data Variable | DVAL/DV | Process data collected during events | Read via event reports |
| Equipment Constant | EC | Configuration parameters | Read/Write (S2F13-16) |
| Collection Event | CE | Significant equipment occurrence | Triggers reports |

### Error Handling Best Practices

1. **Always respond** to primary messages (even if rejecting)
2. **Use appropriate error codes** — don't just return 0 for everything
3. **Send S9Fx** messages for protocol-level errors
4. **Log all communication** for debugging
5. **Implement T3 timeout** — abort waiting transactions
6. **Handle multi-block** messages properly (S2F39/F40 for large data)
7. **Validate data types** — reject messages with wrong format codes
8. **Report alarms** for equipment failures, don't just ignore them

---

## Appendix A: GEM Compliance Statement Template

A GEM Compliance Statement must document:

1. **Equipment identification** (MDLN, SOFTREV)
2. **Supported streams and functions** (complete list)
3. **State models** with transitions
4. **Collection events** with descriptions and linked variables
5. **Status variables** with IDs, names, units, and ranges
6. **Equipment constants** with IDs, names, min/max/default/units
7. **Alarms** with IDs, text, categories, and linked events
8. **Remote commands** with parameters and descriptions
9. **Process programs** format and management
10. **Data variables** available in reports
11. **Variable limits** if supported
12. **Spooling configuration** if supported

---

## Appendix B: SECS-II Message Quick Reference (All Streams)

### Stream 1 — Equipment Status
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | Are You There | H↔E | R |
| 2 | On Line Data | H↔E | - |
| 3 | Selected Equipment Status Request | H→E | R |
| 4 | Selected Equipment Status Data | E→H | - |
| 11 | SV Namelist Request | H→E | R |
| 12 | SV Namelist Reply | E→H | - |
| 13 | Establish Communications Request | H↔E | R |
| 14 | Establish Communications Ack | H↔E | - |
| 15 | Request OFF-LINE | H→E | R |
| 16 | OFF-LINE Ack | E→H | - |
| 17 | Request ON-LINE | H→E | R |
| 18 | ON-LINE Ack | E→H | - |
| 21 | Data Variable Namelist Request | H→E | R |
| 22 | Data Variable Namelist Reply | E→H | - |
| 23 | Collection Event Namelist Request | H→E | R |
| 24 | Collection Event Namelist Reply | E→H | - |

### Stream 2 — Equipment Control
| Fn | Name | Dir | W |
|----|------|-----|---|
| 13 | Equipment Constant Request | H→E | R |
| 14 | Equipment Constant Data | E→H | - |
| 15 | New Equipment Constant Send | H→E | R |
| 16 | New Equipment Constant Ack | E→H | - |
| 17 | Date and Time Request | H↔E | R |
| 18 | Date and Time Data | H↔E | - |
| 21 | Remote Command Send | H→E | [R] |
| 22 | Remote Command Ack | E→H | - |
| 23 | Trace Initialize Send | H→E | R |
| 24 | Trace Initialize Ack | E→H | - |
| 29 | EC Namelist Request | H→E | R |
| 30 | EC Namelist Reply | E→H | - |
| 31 | Date and Time Set Request | H→E | R |
| 32 | Date and Time Set Ack | E→H | - |
| 33 | Define Report | H→E | R |
| 34 | Define Report Ack | E→H | - |
| 35 | Link Event Report | H→E | R |
| 36 | Link Event Report Ack | E→H | - |
| 37 | Enable/Disable Event Report | H→E | R |
| 38 | Enable/Disable Event Report Ack | E→H | - |
| 41 | Host Command Send | H→E | R |
| 42 | Host Command Ack | E→H | - |
| 43 | Reset Spooling | H→E | R |
| 44 | Reset Spooling Ack | E→H | - |
| 45 | Define Variable Limit Attributes | H→E | R |
| 46 | Define Variable Limit Ack | E→H | - |
| 49 | Enhanced Remote Command | H→E | R |
| 50 | Enhanced Remote Command Ack | E→H | - |

### Stream 5 — Exception Handling
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | Alarm Report Send | E→H | R |
| 2 | Alarm Report Ack | H→E | - |
| 3 | Enable/Disable Alarm Send | H→E | R |
| 4 | Enable/Disable Alarm Ack | E→H | - |
| 5 | List Alarms Request | H→E | R |
| 6 | List Alarms Data | E→H | - |
| 7 | List Enabled Alarms Request | H→E | R |
| 8 | List Enabled Alarms Data | E→H | - |

### Stream 6 — Data Collection
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | Trace Data Send | E→H | - |
| 2 | Trace Data Ack | H→E | - |
| 5 | Multi-block Data Send Inquire | E→H | R |
| 6 | Multi-block Grant | H→E | - |
| 11 | Event Report Send | E→H | R |
| 12 | Event Report Ack | H→E | - |
| 15 | Event Report Request | H→E | R |
| 16 | Event Report Data | E→H | - |
| 19 | Individual Report Request | H→E | R |
| 20 | Individual Report Data | E→H | - |

### Stream 7 — Process Program Management
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | PP Load Inquire | H→E | R |
| 2 | PP Load Grant | E→H | - |
| 3 | PP Send | H→E | R |
| 4 | PP Send Ack | E→H | - |
| 5 | PP Request | H→E | R |
| 6 | PP Data | E→H | - |
| 17 | PP Delete Send | H→E | R |
| 18 | PP Delete Ack | E→H | - |
| 19 | Current EPPD Request | H→E | R |
| 20 | Current EPPD Data | E→H | - |
| 23 | Formatted PP Send | H↔E | R |
| 24 | Formatted PP Ack | H↔E | - |
| 25 | Formatted PP Request | H↔E | R |
| 26 | Formatted PP Data | H↔E | - |

### Stream 9 — System Errors
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | Unrecognized Device ID | E→H | - |
| 3 | Unrecognized Stream | E→H | - |
| 5 | Unrecognized Function | E→H | - |
| 7 | Illegal Data | E→H | - |
| 9 | Transaction Timer Timeout | E→H | - |
| 11 | Data Too Long | E→H | - |
| 13 | Conversation Timeout | E→H | - |

### Stream 10 — Terminal Services
| Fn | Name | Dir | W |
|----|------|-----|---|
| 1 | Terminal Request | H→E | R |
| 2 | Terminal Ack | E→H | - |
| 3 | Terminal Display Single | H→E | R |
| 4 | Terminal Display Ack | E→H | - |
| 5 | Terminal Display Multi-block | H→E | R |
| 6 | Terminal Display Multi Ack | E→H | - |
| 9 | Broadcast | H→E | R |
| 10 | Broadcast Ack | E→H | - |

---

## Appendix C: HSMS (E37) Quick Reference

While not part of E5/E30 directly, HSMS is the transport layer used by most modern equipment.

### HSMS Message Types

| SType | Name | Description |
|-------|------|-------------|
| 0 | Data Message | SECS-II message payload |
| 1 | Select.req | Request to establish session |
| 2 | Select.rsp | Response to select request |
| 3 | Deselect.req | Request to end session |
| 4 | Deselect.rsp | Response to deselect |
| 5 | Linktest.req | Connection keepalive request |
| 6 | Linktest.rsp | Connection keepalive response |
| 7 | Reject.req | Reject received message |
| 9 | Separate.req | Immediate disconnect |

### HSMS Connection States

```
┌──────────────┐    TCP Connect    ┌──────────────┐    Select.req/rsp   ┌──────────────┐
│ NOT CONNECTED│───────────────────→│  CONNECTED   │────────────────────→│   SELECTED   │
│              │←───────────────────│  (TCP only)  │←────────────────────│  (SECS-II    │
└──────────────┘    TCP Disconnect  └──────────────┘    Deselect/Separate│   active)    │
                                                                          └──────────────┘
```

---

## Appendix D: Glossary

| Term | Definition |
|------|-----------|
| **CIM** | Computer Integrated Manufacturing — factory automation system |
| **CEID** | Collection Event ID — unique identifier for an event |
| **DVVAL** | Data Variable Value — process data collected during events |
| **EAP** | Equipment Automation Program — software layer between MES and equipment |
| **EC** | Equipment Constant — configurable equipment parameter |
| **GEM** | Generic Equipment Model — SEMI E30 behavioral standard |
| **GEM300** | Extended GEM for 300mm fabs (adds E39, E40, E87, E90, E94, E116) |
| **HSMS** | High-Speed SECS Message Services — TCP/IP transport (SEMI E37) |
| **MES** | Manufacturing Execution System — factory control software |
| **OBJSPEC** | Object Specifier — identifies a specific object instance |
| **PP** | Process Program — recipe or process instructions |
| **PPID** | Process Program Identifier — recipe name |
| **RPTID** | Report ID — identifier for a defined report structure |
| **SECS** | SEMI Equipment Communications Standard |
| **SECS-I** | SEMI E4 — RS-232 serial transport layer |
| **SECS-II** | SEMI E5 — message content standard |
| **SV** | Status Variable — real-time equipment value |
| **SVID** | Status Variable ID — unique identifier for a status variable |
| **VID** | Variable ID — generic identifier for any variable |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E5-0712, SEMI E30-1103 standards*
