# SEMI E39 — Object Services Standard (OSS)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Object Model Concepts](#object-model-concepts)
4. [Object Types](#object-types)
5. [Attributes](#attributes)
6. [Services (Operations)](#services-operations)
7. [SECS-II Messages Used](#secs-ii-messages-used)
8. [State Models](#state-models)
9. [Relationships & Containment](#relationships--containment)
10. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E39** defines the **Object Services Standard (OSS)**, which provides a generalized framework for managing objects within semiconductor equipment. It establishes an object-oriented approach to representing physical and logical entities in equipment.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E39 |
| Full Name | Object Services Standard |
| Abbreviation | OSS |
| Primary Purpose | Object management framework for equipment entities |
| Dependencies | SEMI E5 (SECS-II), SEMI E30 (GEM) |
| Used By | E87 (CMS), E90 (STS), E40 (PJM), E94 (CJM) |

---

## Purpose & Scope

### What E39 Provides

1. **Object Model** — Framework to represent equipment entities as objects with attributes
2. **Services** — Standard operations on objects (create, delete, get/set attributes)
3. **Object Types** — Predefined categories of objects
4. **Attribute System** — Typed, named properties of objects
5. **Relationships** — Parent/child containment hierarchies
6. **Event Notification** — Report object state changes

### Why E39 Exists

Before E39, each SEMI standard defined its own way to represent entities. E39 provides:
- **Consistency** — Common object representation across all GEM300 standards
- **Extensibility** — New object types can be added
- **Interoperability** — Host systems use uniform access patterns
- **Abstraction** — Physical differences hidden behind standard interface

### Relationship to Other Standards

```
┌───────────────────────────────────────────────────────────────┐
│                    E39 Object Services                          │
│              (Base Object Model & Services)                     │
├───────────┬────────────┬────────────┬────────────┬────────────┤
│   E87     │    E90     │    E40     │    E94     │   E116     │
│  Carrier  │ Substrate  │  Process   │  Control   │ Equipment  │
│Management │ Tracking   │Job Mgmt    │Job Mgmt    │Performance │
└───────────┴────────────┴────────────┴────────────┴────────────┘
```

---

## Object Model Concepts

### Core Principles

| Concept | Description |
|---------|-------------|
| **Object** | An instance of an object type with a unique ID and attributes |
| **Object Type** | A class/category defining what attributes an object has |
| **Attribute** | A named, typed property of an object |
| **Object ID** | Unique identifier for an object instance (ObjID) |
| **Object Specifier** | Full path to identify an object: `Type:ID` or nested |
| **Containment** | Objects can contain other objects (parent/child) |

### Object Specifier Format (OBJSPEC)

The OBJSPEC string uniquely identifies an object in the equipment:

```
Format: ObjectType/ObjectID
        or
        ParentType/ParentID/ChildType/ChildID

Examples:
  "EquipmentModule/PM1"
  "LoadPort/LP1/Carrier/FOUP001"
  "ProcessJob/PJ001"
  "Substrate/W01"
```

### Object Lifecycle

```
┌──────────┐    Create    ┌──────────┐    Delete    ┌──────────┐
│  Does    │─────────────→│  Exists  │─────────────→│  Does    │
│Not Exist │              │(Active)  │              │Not Exist │
└──────────┘              └────┬─────┘              └──────────┘
                               │
                    ┌──────────┼──────────┐
                    │          │          │
                    ▼          ▼          ▼
              GetAttr     SetAttr    State Changes
              (Read)      (Write)    (Events)
```

---

## Object Types

### Standard Object Types Defined by E39

| Object Type | Description | Typical Instances |
|-------------|-------------|-------------------|
| **Equipment** | The entire equipment | 1 per tool |
| **Module** | A functional unit of equipment | Chambers, robots |
| **ProcessingResource** | Entity that processes material | Process modules |
| **Port** | Material entry/exit point | Load ports |
| **Carrier** | Material container | FOUPs, cassettes |
| **Substrate** | Individual processing unit | Wafers, panels |
| **ProcessJob** | A processing task | Active jobs |
| **ControlJob** | A control-level job | Lot-level control |
| **Service** | An equipment service | Calibration, cleaning |

### Object Type Hierarchy

```
Equipment
├── Module
│   ├── ProcessingResource
│   │   └── (contains substrates during processing)
│   ├── TransferResource
│   │   └── (robot arms, transfer mechanisms)
│   └── StorageResource
│       └── (buffer positions)
├── Port (LoadPort)
│   └── Carrier
│       └── Substrate (slot positions)
├── ProcessJob
├── ControlJob
└── Service
```

### Equipment Object

The root object representing the entire tool:

| Attribute | Type | Description |
|-----------|------|-------------|
| ObjID | ASCII | Equipment identifier |
| ObjType | ASCII | "Equipment" |
| MDLN | ASCII | Model name |
| SOFTREV | ASCII | Software revision |
| EquipmentState | U1 | Current processing state |
| ClockSetting | ASCII | Current time |

### Module Object

A functional subunit of equipment:

| Attribute | Type | Description |
|-----------|------|-------------|
| ObjID | ASCII | Module identifier (e.g., "PM1") |
| ObjType | ASCII | "Module" or specific subtype |
| ModuleState | U1 | Module operational state |
| ModuleMode | U1 | Auto/Manual/Maintenance |
| SubstrateCount | U4 | Number of substrates in module |

### Carrier Object

A material container (used heavily by E87):

| Attribute | Type | Description |
|-----------|------|-------------|
| ObjID | ASCII | Carrier ID (e.g., "FOUP001") |
| ObjType | ASCII | "Carrier" |
| CarrierIDStatus | U1 | ID verification status |
| SlotMapStatus | U1 | Slot map verification status |
| ContentMap | List | Substrate positions |
| AccessMode | U1 | Manual/Auto access |
| Capacity | U1 | Number of slots |
| SubstrateCount | U4 | Populated slots |
| LocationID | ASCII | Current location |

### Substrate Object

An individual unit being processed (used heavily by E90):

| Attribute | Type | Description |
|-----------|------|-------------|
| ObjID | ASCII | Substrate ID (wafer ID) |
| ObjType | ASCII | "Substrate" |
| SubstLocID | ASCII | Current location |
| SubstState | U1 | Processing state |
| SubstSource | ASCII | Origin carrier/slot |
| SubstDestination | ASCII | Destination carrier/slot |
| LotID | ASCII | Lot identifier |
| SubstProcState | U1 | NeedsProcessing/InProcess/Processed/etc. |
| AcquiredID | ASCII | ID read from substrate |

---

## Attributes

### Attribute Properties

Each attribute has the following meta-properties:

| Meta-Property | Description |
|---------------|-------------|
| **AttrName** | Unique name within the object type |
| **AttrID** | Numeric identifier |
| **DataType** | SECS-II format (A, U4, Boolean, etc.) |
| **Access** | Read-Only, Read-Write, or Write-Only |
| **Persistence** | Whether value survives power cycle |
| **DefaultValue** | Value at object creation |
| **Range** | Valid value range or enumeration |

### Standard Attribute Categories

| Category | Examples | Description |
|----------|----------|-------------|
| Identity | ObjID, ObjType | What the object is |
| State | CurrentState, SubState | Object's current condition |
| Configuration | Capacity, Mode | How the object is set up |
| Status | Count, Utilization | Real-time measurements |
| Relationship | ParentObjID, ContainedObj | Links to other objects |
| Temporal | CreateTime, LastModified | Time-related attributes |

### Attribute Data Types

| Type | Format | Description |
|------|--------|-------------|
| ASCII | A | Text string |
| Boolean | Boolean | True/False |
| Unsigned Integer | U1/U2/U4/U8 | Positive whole numbers |
| Signed Integer | I1/I2/I4/I8 | Positive/negative whole numbers |
| Float | F4/F8 | Decimal numbers |
| Binary | B | Raw bytes |
| List | L | Ordered collection |

---

## Services (Operations)

### Core Object Services

E39 defines these standard operations on objects:

| Service | Description | Primary Direction |
|---------|-------------|-------------------|
| **GetAttr** | Read one or more attributes | H→E (request), E→H (data) |
| **SetAttr** | Write one or more attributes | H→E |
| **Create** | Instantiate a new object | H→E or E (internal) |
| **Delete** | Remove an object instance | H→E or E (internal) |
| **GetType** | List available object types | H→E |
| **GetList** | List instances of a type | H→E |

### GetAttr Service

Read attributes from a specified object:

**Request:**
```
{L:3
  <A OBJSPEC>           ← Object specifier (Type/ID)
  <A OBJTYPE>           ← Object type
  {L:n
    <A ATTRID_1>        ← Attribute names to read
    <A ATTRID_2>
    ...
    <A ATTRID_n>
  }
}
```

**Response:**
```
{L:2
  {L:n
    {L:2
      <A ATTRID>        ← Attribute name
      <V ATTRDATA>      ← Attribute value (type varies)
    }
  }
  {L:p                  ← Error list (empty if all OK)
    {L:2
      <A ATTRID>        ← Failed attribute
      <U1 ERRCODE>      ← Error code
    }
  }
}
```

### SetAttr Service

Write attributes to a specified object:

**Request:**
```
{L:3
  <A OBJSPEC>           ← Object specifier
  <A OBJTYPE>           ← Object type
  {L:n
    {L:2
      <A ATTRID>        ← Attribute name
      <V ATTRDATA>      ← New value
    }
  }
}
```

**Response:**
```
{L:2
  <U1 OBJACK>          ← Overall acknowledge (0=success)
  {L:p                  ← Error list
    {L:2
      <A ATTRID>        ← Failed attribute
      <U1 ERRCODE>      ← Error code
      <A ERRTEXT>       ← Error description
    }
  }
}
```

### Object Acknowledge Codes (OBJACK)

| Code | Meaning |
|------|---------|
| 0 | Successful |
| 1 | Error (see error list for details) |

### Attribute Error Codes

| Code | Meaning |
|------|---------|
| 1 | Unknown object in OBJSPEC |
| 2 | Unknown attribute name |
| 3 | Read-only attribute (cannot set) |
| 4 | Invalid value for attribute |
| 5 | Object not in correct state |
| 6 | Attribute not accessible in current state |

---

## SECS-II Messages Used

E39 primarily uses **Stream 14** for object services:

| Message | Name | Direction | Description |
|---------|------|-----------|-------------|
| S14F1/F2 | GetAttr Request/Data | H↔E | Read object attributes |
| S14F3/F4 | SetAttr Request/Data | H↔E | Write object attributes |
| S14F5/F6 | GetType Request/Data | H→E | List object types |
| S14F7/F8 | GetList Request/Data | H→E | List object instances |
| S14F9/F10 | Create Object Request/Ack | H↔E | Create new object |
| S14F11/F12 | Delete Object Request/Ack | H↔E | Delete object |
| S14F13/F14 | Object Attach/Ack | H↔E | Attach object to parent |
| S14F15/F16 | Object Detach/Ack | H↔E | Detach object from parent |
| S14F17/F18 | GetAttrList Request/Data | H→E | Get attribute definitions |
| S14F19/F20 | SetAttrList Request/Data | H→E | Batch attribute update |

### S14F1/F2 — GetAttr (Detailed)

**S14F1 Request:**
```
{L:3
  <A OBJSPEC>       ← "Equipment/" or "LoadPort/LP1" etc.
  <A OBJTYPE>       ← "Equipment", "Carrier", "Substrate"
  {L:n              ← Attributes to read (empty list = all)
    <A ATTRID>
  }
}
```

**S14F2 Response:**
```
{L:2
  {L:m              ← Matching objects
    {L:2
      <A OBJID>     ← Object instance ID
      {L:n          ← Attribute values
        {L:2
          <A ATTRID>
          <V ATTRDATA>
        }
      }
    }
  }
  {L:p              ← Errors (empty if none)
    {L:2
      <A ERRATTR>
      <U1 ERRCODE>
    }
  }
}
```

### S14F9/F10 — Create Object

**S14F9 Request:**
```
{L:3
  <A OBJSPEC>       ← Object specifier for new object
  <A OBJTYPE>       ← Type of object to create
  {L:n              ← Initial attribute values
    {L:2
      <A ATTRID>
      <V ATTRDATA>
    }
  }
}
```

**S14F10 Response:**
```
{L:2
  <U1 OBJACK>      ← 0=created, 1=error
  {L:p              ← Error details
    {L:2
      <U1 ERRCODE>
      <A ERRTEXT>
    }
  }
}
```

---

## State Models

### Generic Object State Model

E39 defines a general object state lifecycle:

```
┌─────────────┐
│  UNDEFINED  │   Object does not exist
└──────┬──────┘
       │ Create
       ▼
┌─────────────┐
│   ACTIVE    │   Object exists and is operational
└──────┬──────┘
       │ Delete
       ▼
┌─────────────┐
│  UNDEFINED  │   Object removed
└─────────────┘
```

### Extended Object States (used by derived standards)

```
┌─────────────┐
│  UNDEFINED  │
└──────┬──────┘
       │ Create
       ▼
┌─────────────┐    Disable    ┌─────────────┐
│   ACTIVE    │──────────────→│  DISABLED    │
│             │←──────────────│             │
└──────┬──────┘    Enable     └─────────────┘
       │
       │ Delete
       ▼
┌─────────────┐
│  UNDEFINED  │
└─────────────┘
```

### Object Event Notification

When object state changes, equipment reports via S6F11:

| Event | Description |
|-------|-------------|
| ObjectCreated | New object instance created |
| ObjectDeleted | Object instance removed |
| ObjectStateChanged | Object attribute/state changed |
| ObjectAttached | Object attached to parent |
| ObjectDetached | Object detached from parent |

---

## Relationships & Containment

### Containment Model

Objects are organized in a tree hierarchy:

```
Equipment (Root)
├── ProcessingResource: PM1
│   ├── Substrate: W01 (during processing)
│   └── Substrate: W02 (during processing)
├── LoadPort: LP1
│   └── Carrier: FOUP001
│       ├── Substrate: W01 (slot 1)
│       ├── Substrate: W02 (slot 2)
│       └── Substrate: W25 (slot 25)
├── LoadPort: LP2
│   └── Carrier: FOUP002
│       └── ...
├── TransferModule: TM1
│   └── Substrate: W03 (in transit)
└── BufferStation: BUF1
    └── Substrate: W04 (buffered)
```

### Containment Rules

| Rule | Description |
|------|-------------|
| Single Parent | An object can only be in one container at a time |
| Type Constraint | Not all types can contain all other types |
| Move = Detach + Attach | Moving requires detaching from old parent |
| Cascading Delete | Deleting parent may delete children |
| Root | Equipment object is always the root |

### Object Relationships

| Relationship | Description | Example |
|-------------|-------------|---------|
| Contains | Parent holds child | LoadPort contains Carrier |
| IsContainedBy | Child is inside parent | Carrier is in LoadPort |
| Uses | Object uses another for function | ProcessJob uses Recipe |
| AssociatedWith | Logical link without containment | Substrate associated with Lot |

---

## Implementation Guidelines

### Object Registry

Equipment must maintain an internal registry:

```
┌─────────────────────────────────────────────────────────────┐
│                    Object Registry                            │
├─────────────┬───────────┬──────────┬────────────────────────┤
│ ObjType     │ ObjID     │ State    │ Attributes             │
├─────────────┼───────────┼──────────┼────────────────────────┤
│ Equipment   │ EQ1       │ Active   │ MDLN="TOOL-X"          │
│ LoadPort    │ LP1       │ Active   │ PortState=InService     │
│ LoadPort    │ LP2       │ Active   │ PortState=InService     │
│ Carrier     │ FOUP001   │ Active   │ SlotMap=[1,1,0,0...]   │
│ Substrate   │ W001      │ Active   │ State=NeedsProcessing  │
│ ProcessJob  │ PJ001     │ Active   │ State=Executing        │
└─────────────┴───────────┴──────────┴────────────────────────┘
```

### Thread Safety

- Object access must be thread-safe (concurrent GetAttr/SetAttr)
- State changes must be atomic
- Events must be generated AFTER state change completes
- Containment changes must be serialized

### GEM Integration

E39 objects integrate with GEM data collection:

| GEM Feature | E39 Integration |
|-------------|-----------------|
| Status Variables | Map to object attributes |
| Collection Events | Generated on object state changes |
| Remote Commands | Trigger object operations |
| Equipment Constants | May configure object behavior |

### Compliance Requirements

To comply with E39, equipment must:

1. Support at minimum the Equipment object type
2. Implement GetAttr (S14F1/F2) for all objects
3. Report object-related events through S6F11
4. Provide OBJSPEC format for object addressing
5. Document all object types and attributes in compliance statement

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E39-0703 Standard*
