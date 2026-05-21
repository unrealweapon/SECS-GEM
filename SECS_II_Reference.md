# SECS-II Message Reference Guide

## Overview

SECS-II (SEMI Equipment Communications Standard 2) defines the message content and format for communication between semiconductor equipment and a host computer. Messages are organized into **Streams** (categories) and **Functions** (specific operations within a stream).

**Naming Convention:** `SxFy` where `x` = Stream number, `y` = Function number.
- Odd-numbered functions are **Primary** (requests)
- Even-numbered functions are **Reply** messages
- `R` suffix indicates a reply is required
- `[R]` indicates reply is optional

---

## Stream Summary

| Stream | Category | Description |
|--------|----------|-------------|
| S1 | Tool Status | Equipment status and communication |
| S2 | Tool Control & Diagnostics | Equipment constants, remote commands, reports |
| S3 | Material Status | Material tracking, carrier management |
| S4 | Material Control | Material transfer and handoff |
| S5 | Exception Handling | Alarms and exceptions |
| S6 | Data Collection | Event reports, trace data |
| S7 | Process Program Management | Recipe/program transfer |
| S8 | Control Program Transfer | Boot/executive programs |
| S9 | Errors | Communication error messages |
| S10 | Terminal Services | Operator display messages |
| S11 | (Deleted/Obsolete) | File operations (obsolete) |
| S12 | Wafer Maps | Die mapping data |
| S13 | Dataset Transfers | Large data set transfers |
| S14 | Object Services | Object attribute management |
| S15 | Recipe Management | Recipe namespace and management |
| S16 | Processing Management | Process job control |
| S17 | Per Module Reports/Event Tracing | Module-level data collection |
| S18 | Subsystem Management | Subsystem read/write/command |
| S19 | Recipes & Parameters (PDE) | Process Definition Elements |
| S20 | E170 Recipe Management | SRO-based recipe operations |
| S21 | Item Transfer | Generic item transfer |

---


---

## Stream 1 - Tool Status

### `S1F1R` — Are You There?
**Direction:** Host ↔ Equip  

```
header only
```

### `S1F2` — On Line Data
**Direction:** Host ↔ Equip  

```
{L:0
}
or
{L:2
  MDLN
  SOFTREV
}
```

### `S1F3R` — Selected Equipment Status Request
**Direction:** Host → Equip  

```
{L:n
  SVID
}
```

### `S1F4` — Selected Equipment Status Data
**Direction:** Equip → Host  

```
{L:n
  SV
}
```

### `S1F5R` — Formatted Status Request
**Direction:** Host → Equip  

```
SFCD
```

### `S1F6` — Formatted Status Data
**Direction:** Equip → Host  

```
{L:n
  SV
}
```

### `S1F7` — Fixed Form Request
**Direction:** Host → Equip  

```
SFCD
```

### `S1F8` — Fixed Form Data
**Direction:** Equip → Host  

```
{L:n
  {L:2
    SVNAME
    SV0
  }
}
```

### `S1F9R` — Material Transfer Status Request
**Direction:** Host → Equip  

```
header only
```

### `S1F10` — Material Transfer Status Data
**Direction:** Equip → Host  

```
{L:2
  TSIP
  TSOP
}
```

### `S1F11R` — Status Variable Namelist Request
**Direction:** Host → Equip  

```
{L:n
  SVID
}
```

### `S1F12` — Status Variable Namelist Reply
**Direction:** Equip → Host  

```
{L:n
  {L:3
    SVID
    SVNAME
    UNITS
  }
}
```

### `S1F13R` — Establish Communications Request
**Direction:** Host ↔ Equip  

```
{L:0
}
or
{L:2
  MDLN
  SOFTREV
}
```

### `S1F14` — Establish Communications Ack
**Direction:** Host ↔ Equip  

```
{L:2
  COMMACK
  {L:2
    MDLN
    SOFTREV
  }
}
```

### `S1F15R` — Request OFF-LINE
**Direction:** Host → Equip  

```
header only
```

### `S1F16` — OFF-LINE Acknowledge
**Direction:** Equip → Host  

```
OFLACK
```

### `S1F17R` — Request ON-LINE
**Direction:** Host → Equip  

```
header only
```

### `S1F18` — ON-LINE Acknowledge
**Direction:** Equip → Host  

```
ONLACK
```

### `S1F19R` — Get Attribute
**Direction:** Host ↔ Equip  

```
{L:3
  OBJTYPE
  {L:m
    OBJID
  }
  {L:n
    ATTRID
  }
}
```

### `S1F20` — Attribute Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:m
    {L:n
      ATTRDATA
    }
  }
  {L:p
    ...
  }
}
```

### `S1F21R` — Data Variable Namelist Request
**Direction:** Host → Equip  

```
{L:n
  VID
}
```

### `S1F22` — Data Variable Namelist Reply
**Direction:** Equip → Host  

```
{L:n
  {L:3
    VID
    DVVALNAME
    UNITS
  }
}
```

### `S1F23R` — Collection Event Namelist Request
**Direction:** Host → Equip  

```
{L:n
  CEID
}
```

### `S1F24` — Collection Event Namelist Reply
**Direction:** Equip → Host  

```
{L:n
  {L:3
    CEID
    CENAME
    {L:a
      VID
    }
  }
}
```


---

## Stream 2 - Tool Control & Diagnostics

### `S2F1` — Service Program Load Inquire
**Direction:** Host ↔ Equip  

```
{L:2
  SPID
  LENGTH
}
```

### `S2F2` — Service Program Load Grant
**Direction:** Host ↔ Equip  

```
GRANT
```

### `S2F3` — Service Program Send
**Direction:** Host ↔ Equip  

```
SPD
```

### `S2F4` — Service Program Send Ack
**Direction:** Host ↔ Equip  

```
SPAACK
```

### `S2F5` — Service Program Load Request
**Direction:** Host ↔ Equip  

```
SPID
```

### `S2F6` — Service Program Load Data
**Direction:** Host ↔ Equip  

```
SPD
```

### `S2F7` — Service Program Run Send
**Direction:** Host ↔ Equip  

```
SPID
```

### `S2F8` — Service Program Run Ack
**Direction:** Host ↔ Equip  

```
CSAACK
```

### `S2F9` — Service Program Results Request
**Direction:** Host ↔ Equip  

```
SPID
```

### `S2F10` — Service Program Results Data
**Direction:** Host ↔ Equip  

```
SPR
```

### `S2F11` — Service Program Directory Request
**Direction:** Host ↔ Equip  

```
header only
```

### `S2F12` — Service Program Directory Data
**Direction:** Host ↔ Equip  

```
{L:n
  SPID
}
```

### `S2F13R` — Equipment Constant Request
**Direction:** Host → Equip  

```
{L:n
  ECID
}
```

### `S2F14` — Equipment Constant Data
**Direction:** Equip → Host  

```
{L:n
  ECV
}
```

### `S2F15R` — New Equipment Constant Send
**Direction:** Host → Equip  

```
{L:n
  {L:2
    ECID
    ECV
  }
}
```

### `S2F16` — New Equipment Constant Ack
**Direction:** Equip → Host  

```
EAC
```

### `S2F17R` — Date and Time Request
**Direction:** Host ↔ Equip  

```
header only
```

### `S2F18` — Date and Time Data
**Direction:** Host ↔ Equip  

```
TIME
```

### `S2F19R` — Reset/Initialize Send
**Direction:** Host → Equip  

```
RIC
```

### `S2F20` — Reset Acknowledge
**Direction:** Equip → Host  

```
RAC
```

### `S2F21[R]` — Remote Command Send
**Direction:** Host → Equip  

```
RCMD
```

### `S2F22` — Remote Command Ack
**Direction:** Equip → Host  

```
CMDA
```

### `S2F23R` — Trace Initialize Send
**Direction:** Host → Equip  

```
{L:5
  TRID
  DSPER
  TOTSMP
  REPGSZ
  {L:n
    SVID
  }
}
```

### `S2F24` — Trace Initialize Ack
**Direction:** Equip → Host  

```
TIAACK
```

### `S2F25R` — Loopback Diagnostic Request
**Direction:** Host ↔ Equip  

```
ABS
```

### `S2F26` — Loopback Diagnostic Data
**Direction:** Host ↔ Equip  

```
ABS
```

### `S2F27R` — Initiate Processing Request
**Direction:** Host → Equip  

```
{L:3
  LOC
  PPID
  {L:n
    MID
  }
}
```

### `S2F28` — Initiate Processing Ack
**Direction:** Equip → Host  

```
CMDA
```

### `S2F29R` — Equipment Constant Namelist Request
**Direction:** Host → Equip  

```
{L:n
  ECID
}
```

### `S2F30` — Equipment Constant Namelist
**Direction:** Equip → Host  

```
{L:n
  {L:6
    ECID
    ECNAME
    ECMIN
    ECMAX
    ECDEF
    UNITS
  }
}
```

### `S2F31R` — Date and Time Set Request
**Direction:** Host → Equip  

```
TIME
```

### `S2F32` — Date and Time Set Ack
**Direction:** Equip → Host  

```
TIACK
```

### `S2F33R` — Define Report
**Direction:** Host → Equip  

```
{L:2
  DATAID
  {L:n
    {L:2
      RPTID
      {L:m
        VID
      }
    }
  }
}
```

### `S2F34` — Define Report Ack
**Direction:** Equip → Host  

```
DRACK
```

### `S2F35R` — Link Event Report
**Direction:** Host → Equip  

```
{L:2
  DATAID
  {L:a
    {L:2
      CEID
      {L:b
        RPTID
      }
    }
  }
}
```

### `S2F36` — Link Event Report Ack
**Direction:** Equip → Host  

```
LRACK
```

### `S2F37R` — Enable/Disable Event Report
**Direction:** Host → Equip  

```
{L:2
  CEED
  {L:n
    CEID
  }
}
```

### `S2F38` — Enable/Disable Event Report Ack
**Direction:** Equip → Host  

```
ERACK
```

### `S2F39R` — Multi-block Inquire
**Direction:** Host → Equip  

```
{L:2
  DATAID
  DATALENGTH
}
```

### `S2F40` — Multi-block Grant
**Direction:** Equip → Host  

```
GRANT
```

### `S2F41R` — Host Command Send
**Direction:** Host → Equip  

```
{L:2
  RCMD
  {L:n
    {L:2
      CPNAME
      CPVAL
    }
  }
}
```

### `S2F42` — Host Command Ack
**Direction:** Equip → Host  

```
{L:2
  HCACK
  {L:n
    {L:2
      CPNAME
      CPACK
    }
  }
}
```

### `S2F43R` — Configure Spooling
**Direction:** Host → Equip  

```
{L:m
  {L:2
    STRID
    {L:n
      FCNID
    }
  }
}
```

### `S2F44` — Configure Spooling Ack
**Direction:** Equip → Host  

```
{L:2
  RSPACK
  {L:m
    {L:3
      STRID
      STRACK
      {L:n
        FCNID
      }
    }
  }
}
```

### `S2F45R` — Define Variable Limit Attributes
**Direction:** Host → Equip  

```
{L:2
  DATAID
  {L:n
    ...
  }
}
```

### `S2F46` — Define Variable Limit Attributes Ack
**Direction:** Equip → Host  

```
{L:2
  VLAACK
  {L:m
    ...
  }
}
```

### `S2F47R` — Variable Limit Attribute Request
**Direction:** Host → Equip  

```
{L:m
  VID
}
```

### `S2F48` — Variable Limit Attribute Send
**Direction:** Equip → Host  

```
{L:m
  {L:2
    VID
    {L:4*
      ...
    }
  }
}
```

### `S2F49R` — Enhanced Remote Command
**Direction:** Host → Equip  

```
{L:4
  DATAID
  OBJSPEC
  RCMD
  {L:n
    {L:2
      CPNAME
      CEPVAL
    }
  }
}
```

### `S2F50` — Enhanced Remote Command Ack
**Direction:** Equip → Host  

```
{L:2
  HCACK
  {L:n
    {L:2
      CPNAME
      CEPACK
    }
  }
}
```

### `S2F51R` — Request Report Identifiers
**Direction:** Host → Equip  

```
header only
```

### `S2F52` — Return Report Identifiers
**Direction:** Equip → Host  

```
{L:n
  RPTID
}
```

### `S2F53R` — Request Report Definitions
**Direction:** Host → Equip  

```
{L:n
  RPTID
}
```

### `S2F54` — Return Report Definitions
**Direction:** Equip → Host  

```
{L:n
  {L:2
    RPTID
    {L:a
      VID
    }
  }
}
```

### `S2F55R` — Request Event Report Links
**Direction:** Host → Equip  

```
{L:n
  CEID
}
```

### `S2F56` — Return Event Report Links
**Direction:** Equip → Host  

```
{L:n
  {L:3
    CEID
    CENAME
    {L:a
      RPTID
    }
  }
}
```

### `S2F57R` — Request Enabled Events
**Direction:** Host → Equip  

```
header only
```

### `S2F58` — Return Enabled Events
**Direction:** Equip → Host  

```
{L:n
  CEID
}
```

### `S2F59R` — Request Spool Streams/Functions
**Direction:** Host → Equip  

```
header only
```

### `S2F60` — Return Spool Streams and Functions
**Direction:** Equip → Host  

```
{L:n
  {L:2
    STRID
    {L:a
      FCNID
    }
  }
}
```

### `S2F61R` — Request Trace Identifiers
**Direction:** Host → Equip  

```
header only
```

### `S2F62` — Return Trace Identifiers
**Direction:** Equip → Host  

```
{L:n
  TRID
}
```

### `S2F63R` — Request Trace Definitions
**Direction:** Host → Equip  

```
{L:n
  TRID
}
```

### `S2F64` — Return Trace Definitions
**Direction:** Equip → Host  

```
{L:n
  {L:5
    TRID
    DSPER
    TOTSMP
    REPGSZ
    {L:a
      SVID
    }
  }
}
```


---

## Stream 3 - Material Status

### `S3F1R` — Material Status Request
**Direction:** Host → Equip  

```
header only
```

### `S3F2` — Material Status Data
**Direction:** Equip → Host  

```
{L:2
  MF
  {L:m
    {L:3
      LOC
      MID
      QUA
    }
  }
}
```

### `S3F3R` — Time to Completion Request
**Direction:** Host → Equip  

```
header only
```

### `S3F4` — Time to Completion Data
**Direction:** Equip → Host  

```
{L:2
  MF
  TTC
}
```

### `S3F5[R]` — Material Found Send
**Direction:** Equip → Host  

```
{L:2
  MF
  QUA
}
```

### `S3F6` — Material Found Ack
**Direction:** Host → Equip  

```
ACKC3
```

### `S3F7[R]` — Material Lost Send
**Direction:** Equip → Host  

```
{L:3
  MF
  QUA
  MID
}
```

### `S3F8` — Material Lost Ack
**Direction:** Host → Equip  

```
ACKC3
```

### `S3F9R` — Material ID Equate Send
**Direction:** Equip → Host  

```
{L:2
  MID
  EMID
}
```

### `S3F10` — Material ID Equate Ack
**Direction:** Host → Equip  

```
ACKC3
```

### `S3F11R` — Material ID Request
**Direction:** Equip → Host  

```
PTN
```

### `S3F12` — Material ID Request Ack
**Direction:** Host → Equip  

```
{L:3
  PTN
  MIDRA
  MID
}
```

### `S3F13R` — Material ID Send
**Direction:** Host → Equip  

```
{L:2
  PTN
  MID
}
```

### `S3F14` — Material ID Ack
**Direction:** Equip → Host  

```
MIDAC
```

### `S3F17R` — Carrier Action Request
**Direction:** Host → Equip  

```
{L:5
  DATAID
  CARRIERACTION
  CARRIERID
  PTN
  {L:n
    {L:2
      CATTRID
      CATTRDATA
    }
  }
}
```

### `S3F18` — Carrier Action Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S3F19R` — Cancel All Carrier Out Req
**Direction:** Host → Equip  

```
header only
```

### `S3F20` — Cancel All Carrier Out Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S3F25R` — Port Action Req
**Direction:** Host → Equip  

```
{L:3
  PORTACTION
  PTN
  {L:m
    {L:2
      PARAMNAME
      PARAMVAL
    }
  }
}
```

### `S3F26` — Port Action Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S3F27R` — Change Access
**Direction:** Host → Equip  

```
{L:n
  {L:2
    PTN
    ACCESSMODE
  }
}
```

### `S3F28` — Change Access Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:3
      PTN
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S3F29R` — Carrier Tag Read Req
**Direction:** Host → Equip  

```
{L:4
  LOCID
  CARRIERSPEC
  DATASEG
  DATALENGTH
}
```

### `S3F30` — Carrier Tag Read Data
**Direction:** Equip → Host  

```
{L:2
  DATA
  {L:2
    CAACK
    {L:s
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

### `S3F31R` — Carrier Tag Write Data
**Direction:** Host → Equip  

```
{L:5
  LOCID
  CARRIERSPEC
  DATASEG
  DATALENGTH
  DATA
}
```


---

### `S3F15R` — SECS-I Matls Multi-block Inquire, not required for HSMS
**Direction:** Host → Equip  

```
{L:2
  DATAID
  DATALENGTH
}
```
### `S3F16` — Matls Multi-block Grant
**Direction:** Equip → Host  

```
GRANT
```
### `S3F21R` — Port Group Defn
**Direction:** Host → Equip  

```
{L:3
  PORTGRPNAME
  ACCESSMODE
  {L:n
    PTN
  }
}
```
### `S3F22` — Port Group Defn Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S3F23R` — Port Group Action Req
**Direction:** Host → Equip  

```
{L:3
  PGRPACTION
  PORTGRPNAME
  {L:m
    {L:2
      PARAMNAME
      PARAMVAL
    }
  }
}
```
### `S3F24` — Port Group Action Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S3F32` — Carrier Tag Write Ack
**Direction:** Equip → Host  

```
{L:2
  CAACK
  {L:s
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S3F33` — Cancel All Pod Out Req
**Direction:** Host ↔ Equip  

```
header only
```
### `S3F34` — Cancel All Pod Out Ack
**Direction:** Host ↔ Equip  

```
{L:2
  CAACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S3F35R` — eticle Transfer Job Req
**Direction:** Host ↔ Equip  

```
{L:7
  JOBACTION
  PODID
  INPTN
  OUTPTN
  {L:n
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:m
    {L:3
      RETICLEID
      RETREMOVEINSTR
      {L:r
        {L:2
          ATTRID
          ATTRDATA
        }
      }
    }
  }
  {L:k
    {L:2
      RETICLEID2
      RETPLACEINSTR
    }
  }
}
```
### `S3F36R` — eticle Transfer Job Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RPMACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
## Stream 4 - Material Control

### `S4F1R` — Ready to Send Materials
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F2` — Ready to Send Ack
**Direction:** Host ↔ Equip  

```
RSACK
```

### `S4F3` — Send Material
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F5` — Handshake Complete
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F7` — Not Ready to Send
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F9` — Stuck in Sender
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F11` — Stuck in Receiver
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F13` — Send Incomplete Timeout
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F15` — Material Received
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F17R` — Request to Receive
**Direction:** Host ↔ Equip  

```
{L:2
  PTN
  MID
}
```

### `S4F18` — Request to Receive Ack
**Direction:** Host ↔ Equip  

```
RRACK
```

### `S4F19R` — Transfer Job Create
**Direction:** Host → Equip  

```
{L:2
  DATAID
  {L:2
    TRJOBNAME
    {L:n
      {L:12
        ...
      }
    }
  }
}
```

### `S4F20` — Transfer Job Ack
**Direction:** Equip → Host  

```
{L:3
  TRJOBID
  {L:m
    TRATOMCID
  }
  {L:2
    TRACK
    {L:n
      ...
    }
  }
}
```

### `S4F21R` — Transfer Job Command
**Direction:** Host → Equip  

```
{L:4
  TRJOBID
  TRCMDNAME
  {L:n
    {L:2
      CPNAME
      CPVAL
    }
  }
}
```

### `S4F22` — Transfer Job Command Ack
**Direction:** Equip → Host  

```
{L:2
  TRACK
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S4F25R` — Multi-block Inquire
**Direction:** Host → Equip  

```
{L:2
  DATAID
  DATALENGTH
}
```

### `S4F26` — Multi-block Grant
**Direction:** Equip → Host  

```
GRANT
```


---

### `S4F23[R]` — Transfer Command Alert
**Direction:** Equip → Host  

```
{L:4
  TRJOBID
  TRJOBNAME
  TRJOBMS
  {L:2
    TRACK
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S4F24` — Transfer Alert Ack
**Direction:** Host → Equip  

```
header only
```
### `S4F27` — Handoff Ready
**Direction:** Host ↔ Equip  

```
{L:2
  EQNAME
  {L:11
    TRLINK
    TRPORT
    TROBJNAME
    TROBJTYPE
    TRROLE
    TRPTNR
    TRPTPORT
    TRDIR
    TRTYPE
    TRLOCATION
  }
}
```
### `S4F29` — Handoff Command
**Direction:** Host ↔ Equip  

```
{L:4
  TRLINK
  MCINDEX
  HOCMDNAME
  {L:n
    {L:2
      CPNAME
      CPVAL
    }
  }
}
```
### `S4F31` — Handoff Command Complete
**Direction:** Host ↔ Equip  

```
{L:3
  TRLINK
  MCINDEX
  {L:2
    HOACK
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S4F33` — Handoff Verified
**Direction:** Host ↔ Equip  

```
{L:2
  TRLINK
  {L:2
    HOACK
    {L:n
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S4F35` — Handoff Cancel Ready
**Direction:** Host ↔ Equip  

```
TRLINK
```
### `S4F37` — Handoff Cancel Ready Ack
**Direction:** Host ↔ Equip  

```
{L:2
  TRLINK
  HOCANCELACK
}
```
### `S4F39` — Handoff Halt
**Direction:** Host ↔ Equip  

```
TRLINK
```
### `S4F41` — Handoff Halt Ack
**Direction:** Host ↔ Equip  

```
{L:2
  TRLINK
  HOHALTACK
}
```
## Stream 5 - Exception Handling (Alarms)

### `S5F1[R]` — Alarm Report Send
**Direction:** Equip → Host  

```
{L:3
  ALCD
  ALID
  ALTX
}
```

### `S5F2` — Alarm Report Ack
**Direction:** Host → Equip  

```
ACKC5
```

### `S5F3[R]` — Enable/Disable Alarm Send
**Direction:** Host → Equip  

```
{L:2
  ALED
  ALID
}
```

### `S5F4` — Enable/Disable Alarm Ack
**Direction:** Equip → Host  

```
ACKC5
```

### `S5F5R` — List Alarms Request
**Direction:** Host → Equip  

```
ALIDVECTOR
```

### `S5F6` — List Alarm Data
**Direction:** Equip → Host  

```
{L:n
  {L:3
    ALCD
    ALID
    ALTX
  }
}
```

### `S5F7R` — List Enabled Alarm Request
**Direction:** Host → Equip  

```
header only
```

### `S5F8` — List Enabled Alarm Data
**Direction:** Equip → Host  

```
{L:n
  {L:3
    ALCD
    ALID
    ALTX
  }
}
```

### `S5F9[R]` — Exception Post Notify
**Direction:** Equip → Host  

```
{L:5
  TIMESTAMP
  EXID
  EXTYPE
  EXMESSAGE
  {L:n
    EXRECVRA
  }
}
```

### `S5F10` — Exception Post Confirm
**Direction:** Host → Equip  

```
header only
```

### `S5F11[R]` — Exception Clear Notify
**Direction:** Equip → Host  

```
{L:4
  TIMESTAMP
  EXID
  EXTYPE
  EXMESSAGE
}
```

### `S5F12` — Exception Clear Confirm
**Direction:** Host → Equip  

```
header only
```

### `S5F13R` — Exception Recover Request
**Direction:** Host → Equip  

```
{L:2
  EXID
  EXRECVRA
}
```

### `S5F14` — Exception Recover Ack
**Direction:** Equip → Host  

```
{L:2
  EXID
  {L:2
    ACKA
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```

### `S5F15[R]` — Exception Recovery Complete
**Direction:** Equip → Host  

```
{L:3
  ...
}
```

### `S5F16` — Exception Recovery Complete Confirm
**Direction:** Host → Equip  

```
header only
```

### `S5F17R` — Exception Recovery Abort Request
**Direction:** Host → Equip  

```
EXID
```

### `S5F18` — Exception Recovery Abort Ack
**Direction:** Equip → Host  

```
{L:2
  EXID
  {L:2
    ACKA
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```


---

## Stream 6 - Data Collection

### `S6F1[R]` — Trace Data Send
**Direction:** Equip → Host  

```
{L:4
  TRID
  SMPLN
  STIME
  {L:n
    SV
  }
}
```

### `S6F2` — Trace Data Ack
**Direction:** Host → Equip  

```
ACKC6
```

### `S6F3[R]` — Discrete Variable Data Send
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:n
    ...
  }
}
```

### `S6F4` — Discrete Variable Data Send Ack
**Direction:** Host → Equip  

```
ACKC6
```

### `S6F5R` — Multi-block Data Send Inquire
**Direction:** Equip → Host  

```
{L:2
  DATAID
  DATALENGTH
}
```

### `S6F6` — Multi-block Grant
**Direction:** Host → Equip  

```
GRANT6
```

### `S6F7R` — Data Transfer Request
**Direction:** Equip → Host  

```
DATAID
```

### `S6F8` — Data Transfer Data
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:n
    DSID
    {L:m
      {L:2
        DVNAME
        DVVAL
      }
    }
  }
}
```

### `S6F9[R]` — Formatted Variable Send
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:n
    ...
  }
}
```

### `S6F10` — Formatted Variable Ack
**Direction:** Host → Equip  

```
ACKC6
```

### `S6F13R` — Annotated Event Report Send
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:a
    {L:2
      RPTID
      {L:b
        {L:2
          VID
          V
        }
      }
    }
  }
}
```

### `S6F14` — Annotated Event Report Ack
**Direction:** Host → Equip  

```
ACKC6
```

### `S6F15R` — Event Report Request
**Direction:** Host → Equip  

```
CEID
```

### `S6F16` — Event Report Data
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:a
    {L:2
      RPTID
      {L:b
        V
      }
    }
  }
}
```

### `S6F19R` — Individual Report Request
**Direction:** Host → Equip  

```
RPTID
```

### `S6F20` — Individual Report Data
**Direction:** Equip → Host  

```
{L:n
  V
}
```

### `S6F23R` — Request Spooled Data
**Direction:** Host → Equip  

```
RSDC
```

### `S6F24` — Spooled Data Reply
**Direction:** Equip → Host  

```
RSDA
```

### `S6F25[R]` — Notification Report Send
**Direction:** Host ↔ Equip  

```
{L:7
  ...
}
```

### `S6F26` — Notification Report Send Ack
**Direction:** Host ↔ Equip  

```
ACKC6
```

### `S6F27[R]` — Trace Report Send
**Direction:** Equip → Host  

```
{L:3
  DATAID
  TRID
  {L:n
    {L:p
      {L:2
        RPTID
        {L:m
          V
        }
      }
    }
  }
}
```

### `S6F28` — Trace Report Send Ack
**Direction:** Host → Equip  

```
TRID
```


---

### `S6F11R` — Event Report Send
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:a
    {L:2
      RPTID
      {L:b
        V
      }
    }
  }
}
```
### `S6F12` — Event Report Ack
**Direction:** Host → Equip  

```
ACKC6
```
### `S6F17R` — Annotated Event Report Request
**Direction:** Host → Equip  

```
CEID
```
### `S6F18` — Annotated Event Report Data
**Direction:** Equip → Host  

```
{L:3
  DATAID
  CEID
  {L:a
    {L:2
      RPTID
      {L:b
        {L:2
          VID
          V
        }
      }
    }
  }
}
```
### `S6F21R` — Annotated Individual Report Request
**Direction:** Host → Equip  

```
RPTID
```
### `S6F22` — Annotated Individual Report Data
**Direction:** Equip → Host  

```
{L:n
  {L:2
    VID
    V
  }
}
```
### `S6F29R` — Trace Report Request
**Direction:** Host → Equip  

```
TRID
```
### `S6F30` — Trace Report Data
**Direction:** Equip → Host  

```
{L:3
  TRID
  {L:n
    {L:2
      RPTID
      {L:m
        V
      }
    }
  }
  ERRCODE
}
```
## Stream 7 - Process Program Management

### `S7F1R` — Process Program Load Inquire
**Direction:** Host ↔ Equip  

```
{L:2
  PPID
  LENGTH
}
```

### `S7F2` — Process Program Load Grant
**Direction:** Host ↔ Equip  

```
PPGNT
```

### `S7F3R` — Process Program Send
**Direction:** Host ↔ Equip  

```
{L:2
  PPID
  PPBODY
}
```

### `S7F4` — Process Program Send Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```

### `S7F5R` — Process Program Request
**Direction:** Host ↔ Equip  

```
PPID
```

### `S7F6` — Process Program Data
**Direction:** Host ↔ Equip  

```
{L:2
  PPID
  PPBODY
}
```

### `S7F17R` — Delete Process Program
**Direction:** Host → Equip  

```
{L:n
  PPID
}
```

### `S7F18` — Delete Process Program Ack
**Direction:** Equip → Host  

```
ACKC7
```

### `S7F19R` — Current Process Program Dir Request
**Direction:** Host → Equip  

```
header only
```

### `S7F20` — Current Process Program Data
**Direction:** Equip → Host  

```
{L:n
  PPID
}
```

### `S7F23R` — Formatted Process Program Send
**Direction:** Host ↔ Equip  

```
{L:4
  PPID
  MDLN
  SOFTREV
  {L:c
    {L:2
      CCODE
      {L:p
        PPARM
      }
    }
  }
}
```

### `S7F24` — Formatted Process Program Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```

### `S7F25R` — Formatted Process Program Request
**Direction:** Host ↔ Equip  

```
PPID
```

### `S7F26` — Formatted Process Program Data
**Direction:** Host ↔ Equip  

```
{L:4
  PPID
  MDLN
  SOFTREV
  {L:c
    {L:2
      CCODE
      {L:p
        PPARM
      }
    }
  }
}
```


---

### `S7F7R` — Process Program ID Request
**Direction:** Equip → Host  

```
MID
```
### `S7F8` — Process Program ID Data
**Direction:** Host → Equip  

```
{L:2
  PPID
  MID
}
```
### `S7F9R` — Matl/Process Matrix Request
**Direction:** Host ↔ Equip  

```
header only
```
### `S7F10` — Matl/Process Matrix Data
**Direction:** Host ↔ Equip  

```
{L:n
  {L:2
    PPID
    {L:a
      MID
    }
  }
}
```
### `S7F11[R]` — Matl/Process Matrix Update Send
**Direction:** Host → Equip  

```
{L:n
  {L:2
    PPID
    {L:a
      MID
    }
  }
}
```
### `S7F12` — Matl/Process Matrix Update Ack
**Direction:** Equip → Host  

```
ACKC7
```
### `S7F13[R]` — Matl/Process Matrix Delete Entry Send
**Direction:** Host → Equip  

```
{L:n
  {L:2
    PPID
    {L:a
      MID
    }
  }
}
```
### `S7F14` — Delete Matl/Process Matrix Entry Acknowledge
**Direction:** Equip → Host  

```
ACKC7
```
### `S7F15R` — Matrix Mode Select Send
**Direction:** Host → Equip  

```
MMODE
```
### `S7F16` — Matrix Mode Select Ack
**Direction:** Equip → Host  

```
ACKC7
```
### `S7F21` — Process Capabilities Request
**Direction:** Host → Equip  

```
header only
```
### `S7F22` — Process Capabilities Data
**Direction:** Equip → Host  

```
{L:5
  MDLN
  SOFTREV
  CMDMAX
  BYTMAX
  {L:c
    {L:11
      CCODE
      CNAME
      RQCMD
      BLKDEF
      BCDS
      IBCDS
      NBCDS
      ACDS
      IACDS
      NACDS
      {L:p
        L:x
      }
    }
  }
}
```
### `S7F27R` — Process Program Verification Send
**Direction:** Equip → Host  

```
{L:2
  PPID
  {L:n
    {L:3
      ACKC7A
      SEQNUM
      ERRW7
    }
  }
}
```
### `S7F28` — Process Program Verification Acknowledge
**Direction:** Host → Equip  

```
header only
```
### `S7F29R` — Process Program Verification Inquire
**Direction:** Equip → Host  

```
LENGTH
```
### `S7F30` — Process Program Verification Grant
**Direction:** Host → Equip  

```
PPGNT
```
### `S7F31R` — Verification Request Send
**Direction:** Host → Equip  

```
{L:4
  PPID
  MDLN
  SOFTREV
  {L:c
    {L:2
      CCODE
      {L:p
        PPARM
      }
    }
  }
}
```
### `S7F32` — Verification Request Acknowledge
**Direction:** Equip → Host  

```
ACKC7
```
### `S7F33R` — Process Program Available Request
**Direction:** Host ↔ Equip  

```
PPID
```
### `S7F34` — Process Program Availability Data
**Direction:** Host ↔ Equip  

```
{L:3
  PPID
  UNFLEN
  FRMLEN
}
```
### `S7F35R` — Process Program for MID Request
**Direction:** Host ↔ Equip  

```
MID
```
### `S7F36` — Process Program for MID Data
**Direction:** Host ↔ Equip  

```
{L:3
  MID
  PPID
  PPBODY
}
```
### `S7F37R` — Large PP Send
**Direction:** Host ↔ Equip  

```
DSNAME
```
### `S7F38` — Large PP Send Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```
### `S7F39R` — Large Formatted PP Send
**Direction:** Host ↔ Equip  

```
DSNAME
```
### `S7F40` — Large Formatted PP Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```
### `S7F41R` — Large PP Req
**Direction:** Host ↔ Equip  

```
DSNAME
```
### `S7F42` — Large PP Req Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```
### `S7F43R` — Large Formatted PP Req
**Direction:** Host ↔ Equip  

```
DSNAME
```
### `S7F44` — Large Formatted PP Req Ack
**Direction:** Host ↔ Equip  

```
ACKC7
```
## Stream 8 - Control Program Transfer

### `S8F1R` — Boot Program Request
**Direction:** Host ↔ Equip  

```
header only
```

### `S8F2` — Boot Program Data
**Direction:** Host ↔ Equip  

```
BPD
```

### `S8F3R` — Executive Program Request
**Direction:** Host ↔ Equip  

```
header only
```

### `S8F4` — Executive Program Data
**Direction:** Host ↔ Equip  

```
EPD
```


---

## Stream 9 - Communication Errors

### `S9F1` — Unknown Device ID
**Direction:** Equip → Host  

```
MHEAD
```

### `S9F3` — Unknown Stream
**Direction:** Equip → Host  

```
MHEAD
```

### `S9F5` — Unknown Function
**Direction:** Equip → Host  

```
MHEAD
```

### `S9F7` — Illegal Data
**Direction:** Equip → Host  

```
MHEAD
```

### `S9F9` — Transaction Timeout
**Direction:** Equip → Host  

```
SHEAD
```

### `S9F11` — Data Too Long
**Direction:** Equip → Host  

```
MHEAD
```

### `S9F13` — Conversation Timeout
**Direction:** Equip → Host  

```
{L:2
  MEXP
  EDID
}
```


---

## Stream 10 - Terminal Services

### `S10F1[R]` — Terminal Request
**Direction:** Equip → Host  

```
{L:2
  TID
  TEXT
}
```

### `S10F2` — Terminal Request Ack
**Direction:** Host → Equip  

```
ACKC10
```

### `S10F3[R]` — Terminal Display, Single
**Direction:** Host → Equip  

```
{L:2
  TID
  TEXT
}
```

### `S10F4` — Terminal Display, Single Ack
**Direction:** Equip → Host  

```
ACKC10
```

### `S10F5[R]` — Terminal Display, Multi-Block
**Direction:** Host → Equip  

```
{L:2
  TID
  {L:n
    TEXT
  }
}
```

### `S10F6` — Terminal Display, Multi-Block Ack
**Direction:** Equip → Host  

```
ACKC10
```

### `S10F7` — Multi-block Not Allowed
**Direction:** Equip → Host  

```
TID
```

### `S10F9` — Broadcast
**Direction:** Host → Equip  

```
TEXT
```

### `S10F10` — Broadcast Acknowledge
**Direction:** Equip → Host  

```
ACKC10
```


---

## Stream 12 - Wafer Maps

### `S12F1R` — Map Setup Data Send
**Direction:** Equip → Host  

```
{L:15
  ...
}
```

### `S12F2` — Map Setup Data Ack
**Direction:** Host → Equip  

```
SDACK
```

### `S12F3R` — Map Setup Data Request
**Direction:** Equip → Host  

```
{L:9
  ...
}
```

### `S12F4` — Map Setup Data
**Direction:** Host → Equip  

```
{L:15
  ...
}
```

### `S12F5R` — Map Transmit Inquire
**Direction:** Equip → Host  

```
{L:4
  MID
  IDTYP
  MAPFT
  MLCL
}
```

### `S12F6` — Map Transmit Grant
**Direction:** Host → Equip  

```
GRNT1
```

### `S12F7R` — Map Data Send Type 1
**Direction:** Equip → Host  

```
{L:3
  MID
  IDTYP
  {L:n
    {L:2
      RSINF
      BINLT
    }
  }
}
```

### `S12F9R` — Map Data Send Type 2
**Direction:** Equip → Host  

```
{L:4
  MID
  IDTYP
  STRP
  BINLT
}
```

### `S12F11R` — Map Data Send Type 3
**Direction:** Equip → Host  

```
{L:3
  MID
  IDTYP
  {L:n
    {L:2
      XYPOS
      BINLT
    }
  }
}
```


---

### `S12F8` — Map Data Ack Type 1
**Direction:** Host → Equip  

```
MDACK
```
### `S12F10` — Map Data Ack Type 2
**Direction:** Host → Equip  

```
MDACK
```
### `S12F12` — Map Data Ack Type 3
**Direction:** Host → Equip  

```
MDACK
```
### `S12F13R` — Map Data Request Type 1
**Direction:** Equip → Host  

```
{L:2
  MID
  IDTYP
}
```
### `S12F14` — Map Data Type 1
**Direction:** Host → Equip  

```
{L:3
  MID
  IDTYP
  {L:n
    {L:2
      RSINF
      BINLT
    }
  }
}
```
### `S12F15R` — Map Data Request Type 2
**Direction:** Equip → Host  

```
{L:2
  MID
  IDTYP
}
```
### `S12F16` — Map Data Type 2
**Direction:** Host → Equip  

```
{L:4
  MID
  IDTYP
  STRP
  BINLT
}
```
### `S12F17R` — Map Data Request Type 3
**Direction:** Equip → Host  

```
{L:3
  MID
  IDTYP
  SDBIN
}
```
### `S12F18` — Map Data Type 3
**Direction:** Host → Equip  

```
{L:3
  MID
  IDTYP
  {L:n
    {L:2
      XYPOS
      BINLT
    }
  }
}
```
### `S12F19` — Map Error Report Send
**Direction:** Host ↔ Equip  

```
{L:2
  MAPER
  DATLC
}
```
## Stream 13 - Data Set Transfers

### `S13F1R` — Send Data Set Send
**Direction:** Host ↔ Equip  

```
{L:1
  DSNAME
}
```
### `S13F2` — Send Data Set Ack
**Direction:** Host ↔ Equip  

```
{L:2
  DSNAME
  ACKC13
}
```
### `S13F3R` — Open Data Set Request
**Direction:** Host ↔ Equip  

```
{L:3
  HANDLE
  DSNAME
  CKPNT
}
```
### `S13F4` — Open Data Set Data
**Direction:** Host ↔ Equip  

```
{L:5
  HANDLE
  DSNAME
  ACKC13
  RTYPE
  RECLEN
}
```
### `S13F5R` — Read Data Set Request
**Direction:** Host ↔ Equip  

```
{L:2
  HANDLE
  READLN
}
```
### `S13F6R` — ead Data Set Data
**Direction:** Host ↔ Equip  

```
{L:4
  HANDLE
  ACKC13
  CKPNT
  {L:n
    FILDAT
  }
}
```
### `S13F7R` — Close Data Set Send
**Direction:** Host ↔ Equip  

```
{L:1
  HANDLE
}
```
### `S13F8` — Close Data Set Ack
**Direction:** Host ↔ Equip  

```
{L:2
  HANDLE
  ACKC13
}
```
### `S13F9R` — Reset Data Set Send
**Direction:** Host ↔ Equip  

```
header only
```
### `S13F10R` — eset Data Set Ack
**Direction:** Host ↔ Equip  

```
header only
```
### `S13F11R` — Data Set Obj Multi-Block Inquire
**Direction:** Host ↔ Equip  

```
{L:3
  DATAID
  OBJSPEC
  DATALENGTH
}
```
### `S13F12` — Data Set Obj Multi-Block Grant
**Direction:** Host ↔ Equip  

```
GRANT
```
### `S13F13R` — Table Data Send
**Direction:** Host ↔ Equip  

```
{L:8
  DATAID
  OBJSPEC
  TBLTYP
  TBLID
  TBLCMD
  {L:n
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:c
    COLHDR
  }
  {L:r
    {L:m
      TBLELT
    }
  }
}
```
### `S13F14` — Table Data Ack
**Direction:** Host ↔ Equip  

```
{L:2
  TBLACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S13F15R` — Table Data Request
**Direction:** Host ↔ Equip  

```
{L:7
  DATAID
  OBJSPEC
  TBLTYP
  TBLID
  TBLCMD
  {L:p
    COLHDR
  }
  {L:q
    TBLELT
  }
}
```
### `S13F16` — Table Data
**Direction:** Host ↔ Equip  

```
{L:6
  TBLTYP
  TBLID
  {L:n
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:c
    COLHDR
  }
  {L:r
    {L:c
      TBLELT
    }
  }
  {L:2
    TBLACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

## Stream 14 - Object Services

### `S14F1R` — Get Attributes Request
**Direction:** Host ↔ Equip  

```
{L:5
  OBJSPEC
  OBJTYPE
  {L:i
    OBJID
  }
  {L:q
    {L:3
      ATTRID
      ATTRDATA
      ATTRRELN
    }
  }
  {L:a
    ATTRID
  }
}
```
### `S14F2` — Attribute Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:2
      OBJID
      {L:a
        {L:2
          ATTRID
          ATTRDATA
        }
      }
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F3R` — Set Attributes
**Direction:** Host ↔ Equip  

```
{L:4
  OBJSPEC
  OBJTYPE
  {L:i
    OBJID
  }
  {L:n
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F4` — Set Attributes Reply
**Direction:** Host ↔ Equip  

```
{L:2
  {L:i
    {L:2
      OBJID
      {L:n
        {L:2
          ATTRID
          ATTRDATA
        }
      }
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F5R` — Get Type Data
**Direction:** Host ↔ Equip  

```
OBJSPEC
```
### `S14F6` — Type Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    OBJTYPE
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F7R` — Get Attribute Names for the types
**Direction:** Host ↔ Equip  

```
{L:2
  OBJSPEC
  {L:n
    OBJTYPE
  }
}
```
### `S14F8` — Attribute Names of the object types
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:2
      OBJTYPE
      {L:a
        ATTRID
      }
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F9R` — Create Obj Request
**Direction:** Host ↔ Equip  

```
{L:3
  OBJSPEC
  OBJTYPE
  {L:a
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F10` — Create Obj Ack
**Direction:** Host ↔ Equip  

```
{L:3
  OBJSPEC
  {L:b
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F11R` — Delete Obj Request
**Direction:** Host ↔ Equip  

```
{L:2
  OBJSPEC
  {L:a
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F12` — Delete Obj Ack
**Direction:** Host ↔ Equip  

```
{L:2
  {L:b
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F13R` — Object Attach Request
**Direction:** Host ↔ Equip  

```
{L:2
  OBJSPEC
  {L:a
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F14` — Object Attach Ack
**Direction:** Host ↔ Equip  

```
{L:3
  OBJTOKEN
  {L:b
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F15R` — Attached Obj Action Req.
**Direction:** Host ↔ Equip  

```
{L:4
  OBJSPEC
  OBJCMD
  OBJTOKEN
  {L:a
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F16` — Attached Obj Action Ack
**Direction:** Host ↔ Equip  

```
{L:2
  {L:b
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F17R` — Supervised Obj Action Req
**Direction:** Host ↔ Equip  

```
{L:4
  OBJSPEC
  OBJCMD
  TARGETSPEC
  {L:a
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S14F18` — Supervised Obj Action Ack
**Direction:** Host ↔ Equip  

```
{L:2
  {L:b
    {L:2
      ATTRID
      ATTRDATA
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F19R` — Generic Service Req
**Direction:** Host → Equip  

```
{L:5
  DATAID
  OPID
  OBJSPEC
  SVCNAME
  {L:m
    {L:2
      SPNAME
      SPVAL
    }
  }
}
```
### `S14F20` — Generic Service Ack
**Direction:** Host ↔ Equip  

```
{L:4
  SVCACK
  LINKID
  {L:n
    {L:2
      SPNAME
      SPVAL
    }
  }
  {L:2
    SVCACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F21R` — Generic Service Completion
**Direction:** Host ↔ Equip  

```
{L:5
  DATAID
  OPID
  LINKID
  {L:n
    {L:2
      SPNAME
      SPVAL
    }
  }
  {L:2
    SVCACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F22` — Generic Service Comp Ack
**Direction:** Host ↔ Equip  

```
DATAACK
```
### `S14F23R` — Multi-block Generic Service Inquire
**Direction:** Host ↔ Equip  

```
{L:2
  DATAID
  DATALENGTH
}
```
### `S14F24` — Multi-block Generic Service Grant
**Direction:** Host ↔ Equip  

```
GRANT
```
### `S14F25R` — Service Name Request
**Direction:** Host ↔ Equip  

```
{L:2
  OBJSPEC
  {L:n
    OBJTYPE
  }
}
```
### `S14F26` — Service Name Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:2
      OBJTYPE
      {L:a
        SVCNAME
      }
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S14F27R` — Service Parameter Name Req
**Direction:** Host ↔ Equip  

```
{L:3
  OBJSPEC
  OBJTYPE
  {L:n
    SVCNAME
  }
}
```
### `S14F28` — Service Parameter Name Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:2
      SVCNAME
      {L:a
        SPNAME
      }
    }
  }
  {L:2
    OBJACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

## Stream 15 - Recipe Management

### `S15F1R` — Recipe Management Multi-Block Inquire
**Direction:** Host ↔ Equip  

```
{L:3
  DATAID
  RCPSPEC
  RMDATASIZE
}
```
### `S15F2R` — ecipe Management Multi-block Grant
**Direction:** Host ↔ Equip  

```
RMGRNT
```
### `S15F3R` — Recipe Namespace Action Req
**Direction:** Host ↔ Equip  

```
{L:2
  RMNSSPEC
  RMNSCMD
}
```
### `S15F4R` — ecipe Namespace Action
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F5R` — Recipe Namespace Rename Req
**Direction:** Host ↔ Equip  

```
{L:2
  RMNSSPEC
  RMNEWNS
}
```
### `S15F6R` — ecipe Namespace Rename Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F7R` — Recipe Space Req
**Direction:** Host ↔ Equip  

```
OBJSPEC
```
### `S15F8R` — ecipe Space Data
**Direction:** Host ↔ Equip  

```
{L:2
  RMSPACE
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F9R` — Recipe Status Request
**Direction:** Host ↔ Equip  

```
RCPSPEC
```
### `S15F10R` — ecipe Status Data
**Direction:** Host ↔ Equip  

```
{L:3
  RCPSTAT
  RCPVERS
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F11R` — Recipe Version Request
**Direction:** Host ↔ Equip  

```
{L:4
  RMNSSPEC
  RCPCLASS
  RCPNAME
  AGENT
}
```
### `S15F12R` — ecipe Version Data
**Direction:** Host ↔ Equip  

```
{L:3
  AGENT
  RCPVERS
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F13R` — Recipe Create Req
**Direction:** Host ↔ Equip  

```
{L:5
  DATAID
  RCPUPDT
  RCPSPEC
  {L:m
    {L:2
      RCPATTRID
      RCPATTRDATA
    }
  }
  RCPBODY
}
```
### `S15F14R` — ecipe Create Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F15R` — Recipe Store Req
**Direction:** Host ↔ Equip  

```
{L:4
  DATAID
  RCPSPEC
  RCPSECCODE
  {L:3
    {L:2*
      RCPSECNM
      {L:g
        {L:2
          RCPATTRID
          RCPATTRDATA
        }
      }
    }
    RCPBODY
    {L:m
      {L:2
        RCPSECNM
        {L:a
          {L:2
            RCPATTRID
            RCPATTRDATA
          }
        }
      }
    }
  }
}
```
### `S15F16R` — ecipe Store Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RCPSECCODE
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F17R` — Recipe Retrieve Req
**Direction:** Host ↔ Equip  

```
{L:2
  RCPSPEC
  RCPSECCODE
}
```
### `S15F18R` — ecipe Retrieve Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:q
    {L:r
      RCPSECNM
      {L:g
        {L:2
          RCPATTRID
          RCPATTRDATA
        }
      }
    }
    RCPBODY
    {L:m
      {L:2
        RCPSECNM
        {L:a
          {L:2
            RCPATTRID
            RCPATTRDATA
          }
        }
      }
    }
  }
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F19R` — Recipe Rename Req
**Direction:** Host ↔ Equip  

```
{L:3
  RCPSPEC
  RCPRENAME
  RCPNEWID
}
```
### `S15F20R` — ecipe Rename Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F21R` — Recipe Action Req
**Direction:** Host ↔ Equip  

```
{L:6
  DATAID
  RCPCMD
  RMNSSPEC
  OPID
  AGENT
  {L:n
    RCPID
  }
}
```
### `S15F22R` — ecipe Action Ack
**Direction:** Host ↔ Equip  

```
{L:4
  AGENT
  LINKID
  RCPCMD
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F23R` — Recipe Descriptor Req
**Direction:** Host ↔ Equip  

```
{L:3
  DATAID
  OBJSPEC
  {L:n
    RCPID
  }
}
```
### `S15F24R` — ecipe Descriptor Data
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:a
      {L:3*
        RCPDESCNM
        RCPDESCTIME
        RCPDESCLTH
      }
    }
  }
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F25R` — Recipe Parameter Update Req
**Direction:** Host ↔ Equip  

```
{L:4
  DATAID
  RMNSSPEC
  AGENT
  {L:n
    {L:3
      RCPPARNM
      RCPPARVAL
      RCPPARRULE
    }
  }
}
```
### `S15F26R` — ecipe Parameter Update Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F27R` — Recipe Download Req
**Direction:** Host → Equip  

```
{L:5
  DATAID
  RCPOWCODE
  RCPSPEC
  {L:m
    {L:2
      RCPATTRID
      RCPATTRDATA
    }
  }
  RCPBODY
}
```
### `S15F28R` — ecipe Download Ack
**Direction:** Equip → Host  

```
{L:3
  RCPID
  {L:n
    {L:2
      RCPATTRID
      RCPATTRDATA
    }
  }
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F29R` — Recipe Verify Req
**Direction:** Host → Equip  

```
{L:4
  DATAID
  OPID
  RESPEC
  {L:m
    RCPID
  }
}
```
### `S15F30R` — ecipe Verify Ack
**Direction:** Equip → Host  

```
{L:5
  OPID
  LINKID
  RCPID
  {L:n
    {L:2
      RCPATTRID
      RCPATTRDATA
    }
  }
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F31R` — Recipe Unload Req
**Direction:** Host → Equip  

```
RCPSPEC
```
### `S15F32R` — ecipe Unload Data
**Direction:** Equip → Host  

```
{L:4
  RCPSPEC
  {L:m
    {L:2
      RCPATTRID
      RCPATTRDATA
    }
  }
  RCPBODY
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F33R` — Recipe Select Req
**Direction:** Host → Equip  

```
{L:3
  DATAID
  RESPEC
  {L:r
    {L:2
      RCPID
      {L:p
        {L:2
          RCPPARNM
          RCPPARVAL
        }
      }
    }
  }
}
```
### `S15F34R` — ecipe Select Ack
**Direction:** Equip → Host  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F35R` — Recipe Delete Req
**Direction:** Host → Equip  

```
{L:4
  DATAID
  RESPEC
  RCPDEL
  {L:n
    RCPID
  }
}
```
### `S15F36R` — ecipe Delete Ack
**Direction:** Equip → Host  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F37R` — DRNS Segment Approve Action Req
**Direction:** Host ↔ Equip  

```
{L:6
  RMSEGSPEC
  OBJTOKEN
  RMGRNT
  OPID
  RCPID
  RMCHGTYPE
}
```
### `S15F38` — DRNS Segment Approve Action Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F39R` — DRNS Recorder Seg Req
**Direction:** Host ↔ Equip  

```
{L:5
  DATAID
  RMNSCMD
  RMRECSPEC
  RMSEGSPEC
  OBJTOKEN
}
```
### `S15F40` — DRNS Recorder Seg Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F41R` — DRNS Recorder Mod Req
**Direction:** Host ↔ Equip  

```
{L:5
  DATAID
  RMRECSPEC
  OBJTOKEN
  RMNSCMD
  {L:c
    RCPID
    RCPNEWID
    RMSEGSPEC
    RMCHGTYPE
    OPID
    TIMESTAMP
    RMREQUESTOR
  }
}
```
### `S15F42` — DRNS Recorder Mod Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F43R` — DRNS Get Change Req
**Direction:** Host ↔ Equip  

```
{L:3
  DATAID
  OBJSPEC
  TARGETSPEC
}
```
### `S15F44` — DRNS Get Change Ack
**Direction:** Host ↔ Equip  

```
{L:2
  {L:n
    {L:7
      RCPID
      RCPNEWID
      RMSEGSPEC
      RMCHGTYPE
      OPID
      TIMESTAMP
      RMREQUESTOR
    }
  }
  {L:2
    RMACK
    {L:p
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F45R` — DRNS Mgr Seg Aprvl Req
**Direction:** Host ↔ Equip  

```
{L:4
  DATAID
  RCPSPEC
  RCPNEWID
  RMCHGTYPE
}
```
### `S15F46` — DRNS Mgr Seg Aprvl Ack
**Direction:** Host ↔ Equip  

```
{L:3
  RMCHGTYPE
  RMGRNT
  OPID
}
```
### `S15F47R` — DRNS Mgr Rebuild Req
**Direction:** Host ↔ Equip  

```
{L:5
  DATAID
  OBJSPEC
  RMNSSPEC
  RMRECSPEC
  {L:n
    RMSEGSPEC
  }
}
```
### `S15F48` — DRNS Mgr Rebuild Ack
**Direction:** Host ↔ Equip  

```
{L:2
  RMACK
  {L:p
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S15F49R` — Large Recipe Download Req
**Direction:** Host → Equip  

```
{L:2
  DSNAME
  RCPOWCODE
}
```
### `S15F50` — Large Recipe Download Ack
**Direction:** Equip → Host  

```
ACKC15
```
### `S15F51R` — Large Recipe Upload Req
**Direction:** Host → Equip  

```
DSNAME
```
### `S15F52` — Large Recipe Upload Ack
**Direction:** Equip → Host  

```
ACKC15
```
### `S15F53R` — Recipe Verification Send
**Direction:** Equip → Host  

```
{L:3
  RCPSPEC
  RCPID
  {L:2
    RMACK
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S15F54R` — ecipe Verification Ack
**Direction:** Host → Equip  

```
header only
```

## Stream 16 - Processing Management

### `S16F1R` — Process Job Data MBI
**Direction:** Equip → Host  

```
{L:2
  DATAID
  DATALENGTH
}
```

### `S16F2` — PJD MBI Grant
**Direction:** Host → Equip  

```
GRANT
```

### `S16F3R` — Process Job Create Req
**Direction:** Host → Equip  

```
{L:5
  DATAID
  MF
  {L:n
    MID
  }
  {L:3
    ...
  }
  ...
}
```

### `S16F4` — Process Job Create Ack
**Direction:** Equip → Host  

```
{L:2
  PRJOBID
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

### `S16F5R` — Process Job Cmd Req
**Direction:** Host → Equip  

```
{L:3
  PRJOBID
  PRCMDNAME
  {L:n
    {L:2
      CPNAME
      CPVAL
    }
  }
}
```

### `S16F6` — Process Job Cmd Ack
**Direction:** Equip → Host  

```
{L:2
  PRJOBID
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

### `S16F11R` — PRJobCreateEnh
**Direction:** Host → Equip  

```
{L:7
  ...
}
```

### `S16F12` — PRJobCreateEnh Ack
**Direction:** Equip → Host  

```
{L:2
  PRJOBID
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```

### `S16F15R` — PRJobMultiCreate
**Direction:** Host → Equip  

```
{L:2
  DATAID
  {L:m
    ...
  }
}
```

### `S16F16` — PRJobMultiCreate Ack
**Direction:** Equip → Host  

```
{L:2
  {L:m
    PRJOBID
  }
  {L:2
    ACKA
    ...
  }
}
```
## Stream 17 - Event Tracing / Per Module

### `S17F1R` — Data Report Create Req
**Direction:** Host → Equip  

```
{L:4
  DATAID
  RPTID
  DATASRC
  {L:n
    VID
  }
}
```
### `S17F2` — Data Report Create Ack
**Direction:** Equip → Host  

```
{L:2
  RPTID
  ERRCODE
}
```
### `S17F3R` — Data Report Delete Req
**Direction:** Host → Equip  

```
{L:n
  RPTID
}
```
### `S17F4` — Data Report Del Ack
**Direction:** Equip → Host  

```
{L:2
  ACKA
  {L:m
    {L:3
      RPTID
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S17F5R` — Trace Create Req
**Direction:** Host → Equip  

```
{L:6
  DATAID
  TRID
  CEED
  {L:n
    RPTID
  }
  {L:8*
    TOTSMP
    REPGSZ
    EVNTSRC
    CEIDSTART
    EVNTSRC2
    CEIDSTOP
    TRAUTOD
    RPTOC
  }
}
```
### `S17F6` — Trace Create Ack
**Direction:** Equip → Host  

```
{L:2
  TRID
  ERRCODE
}
```
### `S17F7R` — Trace Delete Req
**Direction:** Host → Equip  

```
{L:n
  TRID
}
```
### `S17F8` — Trace Delete Ack
**Direction:** Equip → Host  

```
{L:2
  ACKA
  {L:m
    {L:3
      TRID
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S17F9R` — Collection Event Link Req
**Direction:** Host → Equip  

```
{L:4
  DATAID
  EVNTSRC
  CEID
  {L:n
    RPTID
  }
}
```
### `S17F10` — Collection Event Link Ack
**Direction:** Equip → Host  

```
{L:3
  EVNTSRC
  CEID
  ERRCODE
}
```
### `S17F11R` — Collection Event Unlink
**Direction:** Host → Equip  

```
{L:3
  EVNTSRC
  CEID
  RPTID
}
```
### `S17F12` — Collection Event Unlink Ack
**Direction:** Equip → Host  

```
{L:4
  EVNTSRC
  CEID
  RPTID
  ERRCODE
}
```
### `S17F13R` — Trace Reset Req
**Direction:** Host → Equip  

```
{L:n
  TRID
}
```
### `S17F14` — Trace Reset Ack
**Direction:** Equip → Host  

```
{L:2
  ACKA
  {L:m
    {L:3
      TRID
      ERRCODE
      ERRTEXT
    }
  }
}
```

## Stream 18 - Subsystem Management

### `S18F1R` — Read Attribute Req
**Direction:** Host → Equip  

```
{L:2
  TARGETID
  {L:n
    ATTRID
  }
}
```
### `S18F2R` — ead Attribute Data
**Direction:** Equip → Host  

```
{L:4
  TARGETID
  SSACK
  {L:n
    ATTRDATA
  }
  {L:s
    STATUS
  }
}
```
### `S18F3R` — Write Attribute Req
**Direction:** Host → Equip  

```
{L:2
  TARGETID
  {L:n
    {L:2
      ATTRID
      ATTRDATA
    }
  }
}
```
### `S18F4` — Write Attribute Ack
**Direction:** Equip → Host  

```
{L:3
  TARGETID
  SSACK
  {L:s
    STATUS
  }
}
```
### `S18F5R` — Read Request
**Direction:** Host → Equip  

```
{L:3
  TARGETID
  DATASEG
  DATALENGTH
}
```
### `S18F6R` — ead Data
**Direction:** Equip → Host  

```
{L:4
  TARGETID
  SSACK
  DATA
  {L:s
    STATUS
  }
}
```
### `S18F7R` — Write Data Request
**Direction:** Host → Equip  

```
{L:4
  TARGETID
  DATASEG
  DATALENGTH
  DATA
}
```
### `S18F8` — Write Data Ack
**Direction:** Equip → Host  

```
{L:3
  TARGETID
  SSACK
  {L:s
    STATUS
  }
}
```
### `S18F9R` — Read ID Req
**Direction:** Host → Equip  

```
TARGETID
```
### `S18F10R` — ead ID Data
**Direction:** Equip → Host  

```
{L:4
  TARGETID
  SSACK
  MID
  {L:s
    STATUS
  }
}
```
### `S18F11R` — Write ID Req
**Direction:** Host → Equip  

```
{L:2
  TARGETID
  MID
}
```
### `S18F12` — Write ID Ack
**Direction:** Equip → Host  

```
{L:3
  TARGETID
  SSACK
  {L:s
    STATUS
  }
}
```
### `S18F13R` — Subsystem Command
**Direction:** Host → Equip  

```
{L:3
  TARGETID
  SSCMD
  {L:n
    CPVAL
  }
}
```
### `S18F14` — Subsystem Command Ack
**Direction:** Equip → Host  

```
{L:3
  TARGETID
  SSACK
  {L:s
    STATUS
  }
}
```
### `S18F15R` — Read 2D Code Cond Req
**Direction:** Host → Equip  

```
TARGETID
```
### `S18F16R` — ead 2D Code Cond Data
**Direction:** Equip → Host  

```
{L:5
  TARGETID
  SSACK
  MID
  {L:s
    STATUS
  }
  {L:c
    CONDITION
  }
}
```

## Stream 19 - Recipe and Parameter Management (E139)

### `S19F1R` — Request Process Definition Element (PDE) Directory
**Direction:** Host ↔ Equip  

```
{L:2
  {L:m
    {L:3
      PDEATTRIBUTENAME
      COMPARISONOPERATOR
      PDEATTRIBUTEVALUE
    }
  }
  {L:n
    PDEATTRIBUTE
  }
}
```
### `S19F2` — PDE Directory Data
**Direction:** Host ↔ Equip  

```
{L:3
  DIRRSPSTAT
  STATUSTXT
  {L:m
    {L:2
      UID
      {L:n
        {L:2
          PDEATTRIBUTE
          PDEATTRIBUTEVALUE
        }
      }
    }
  }
}
```
### `S19F3R` — PDE Delete Request
**Direction:** Host → Equip  

```
{L:n
  UID
}
```
### `S19F4` — PDE Delete Acknowledge
**Direction:** Equip → Host  

```
{L:n
  {L:3
    UID
    DELRSPSTAT
    STATUSTXT
  }
}
```
### `S19F5R` — PDE Header Data Request
**Direction:** Host ↔ Equip  

```
{L:n
  UID
}
```
### `S19F6` — PDE Header Data Reply
**Direction:** Host ↔ Equip  

```
{L:2
  TCID
  {L:n
    {L:3
      UID
      GETRSPSTAT
      STATUSTXT
    }
  }
}
```
### `S19F7R` — request the transfer of PDEs via Stream 13
**Direction:** Host ↔ Equip  

```
{L:n
  UID
}
```
### `S19F8` — PDE Transfer Reply
**Direction:** Host ↔ Equip  

```
{L:2
  TCID
  {L:n
    {L:3
      UID
      GETRSPSTAT
      STATUSTXT
    }
  }
}
```
### `S19F9R` — Request to Send PDE
**Direction:** Host ↔ Equip  

```
{L:2
  TCID
  TRANSFERSIZE
}
```
### `S19F10` — Initiate PDE transfer Reply
**Direction:** Host ↔ Equip  

```
{L:3
  TCID
  RTSRSPSTAT
  STATUSTXT
}
```
### `S19F11R` — Send PDE
**Direction:** Host ↔ Equip  

```
TCID
```
### `S19F12` — Send PDE Acknowledge
**Direction:** Host ↔ Equip  

```
header only
```
### `S19F13R` — TransferContainer Report
**Direction:** Host ↔ Equip  

```
{L:n
  {L:4
    UID
    SENDRSPSTAT
    VERIFYRSPSTAT
    STATUSTXT
  }
}
```
### `S19F14` — TransferContainer Report Ack
**Direction:** Host ↔ Equip  

```
header only
```
### `S19F15R` — Request PDE Resolution
**Direction:** Host → Equip  

```
{L:2
  TARGETPDE
  {L:n
    {L:2
      PDEREF
      RESOLUTION
    }
  }
}
```
### `S19F16` — PDE Resolution Data
**Direction:** Equip → Host  

```
{L:2
  {L:m
    {L:2
      PDEREF
      RESOLUTION
    }
  }
  {L:n
    {L:3
      UID
      RESPDESTAT
      STATUSTXT
    }
  }
}
```
### `S19F17R` — Verify PDE Request
**Direction:** Host → Equip  

```
{L:4
  TARGETPDE
  {L:n
    {L:2
      PDEREF
      RESOLUTION
    }
  }
  VERIFYTYPE
  VERIFYDEPTH
}
```
### `S19F18` — PDE Verification Result
**Direction:** Equip → Host  

```
{L:2
  VERIFYSUCCESS
  {L:n
    {L:3
      UID
      VERIFYRSPSTAT
      STATUSTXT
    }
  }
}
```
### `S19F19R` — S19 Multi-block Inquire
**Direction:** Host ↔ Equip  

```
DATALENGTH
```
### `S19F20` — S19 Multi-block Grant
**Direction:** Host ↔ Equip  

```
GRANT
```

## Stream 20 - Recipe Management (E170 SRO)

### `S20F1R` — SetSRO Attributes Request
**Direction:** Host → Equip  

```
{L:6
  OBJID
  OBJTYPE
  AUTOPOST_DISABLE
  AUTOCLEAR_DISABLE
  RETAINRECIPE_DISABLE
  AUTOCLOSE
}
```
### `S20F2` — SetSRO Attributes Acknowledge
**Direction:** Equip → Host  

```
SSAACK
```
### `S20F3R` — GetOperationIDList Request
**Direction:** Host → Equip  

```
{L:3
  OBJID
  OBJTYPE
  OPETYPE
}
```
### `S20F4` — GetOperationIDList Acknowledge
**Direction:** Equip → Host  

```
{L:2
  {L:n
    OPEID
  }
  GOILACK
}
```
### `S20F5R` — OpenConnectionEvent Send
**Direction:** Equip → Host  

```
{L:7
  OBJID
  OBJTYPE
  OPETYPE
  RMSUSERID
  RMSPWD
  EQUSERID
  OPEID
}
```
### `S20F6` — OpenConnectionEvent Acknowledge
**Direction:** Host → Equip  

```
{L:2
  OPEID
  OCEACK
}
```
### `S20F7R` — CloseConnectionEvent Send
**Direction:** Equip → Host  

```
{L:4
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
}
```
### `S20F8` — CloseConnectionEvent Acknowledge
**Direction:** Host → Equip  

```
{L:2
  OPEID
  CCEACK
}
```
### `S20F9R` — ClearOperation Request
**Direction:** Host → Equip  

```
{L:4
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
}
```
### `S20F10` — ClearOperation Acknowledge
**Direction:** Equip → Host  

```
COACK
```
### `S20F11R` — GetRecipeXIDList Request
**Direction:** Host → Equip  

```
{L:4
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
}
```
### `S20F12` — GetRecipeXIDList Acknowledge
**Direction:** Equip → Host  

```
{L:2
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
  GRXLACK
}
```
### `S20F13R` — DeleteRecipe Request
**Direction:** Host → Equip  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  {L:9
    TIMESTAMP
    OPEID
    ASSGNID
    COPYID
    REVID
    RecID
    VERID
    TYPEID
    EQID
  }
}
```
### `S20F14` — DeleteRecipe Acknowledge
**Direction:** Equip → Host  

```
DRRACK
```
### `S20F15R` — WriteRecipe Request
**Direction:** Host → Equip  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  {L:n
    {L:10
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
      RCPBODYA
    }
  }
}
```
### `S20F16` — WriteRecipe Acknowledge
**Direction:** Equip → Host  

```
WRACK
```
### `S20F17R` — ReadRecipe Request
**Direction:** Host → Equip  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
}
```
### `S20F18R` — eadRecipe Acknowledge
**Direction:** Equip → Host  

```
{L:2
  {L:n
    {L:10
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
      RCPBODYA
    }
  }
  RRACK_S20
}
```
### `S20F19R` — QueryRecipeXIDList Event Send
**Direction:** Equip → Host  

```
{L:4
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
}
```
### `S20F20` — QueryRecipeXIDList Event Acknowledge
**Direction:** Host → Equip  

```
{L:3
  OPEID
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
  QRXLEACK
}
```
### `S20F21R` — QueryRecipe Event Send
**Direction:** Equip → Host  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
}
```
### `S20F22` — QueryRecipe Event Acknowledge
**Direction:** Host → Equip  

```
QREACK
```
### `S20F23R` — PostRecipe Event Send
**Direction:** Equip → Host  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  {L:n
    {L:10
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
      RCPBODYA
    }
  }
}
```
### `S20F24` — PostRecipe Event Acknowledge
**Direction:** Host → Equip  

```
PREACK
```
### `S20F25R` — SetPRC Attributes Request
**Direction:** Host → Equip  

```
{L:5
  OBJID
  OBJTYPE
  {L:n
    MAXNUMBER
  }
  MAXTIME
  PRCPREEXECHK
}
```
### `S20F26` — SetPRC Attributes Acknowledge
**Direction:** Equip → Host  

```
SPAACK
```
### `S20F27R` — PreSpecifyRecipe Request
**Direction:** Host → Equip  

```
{L:6
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  PRJOBID
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
}
```
### `S20F28` — PreSpecifyRecipe Acknowledge
**Direction:** Equip → Host  

```
PSRACK
```
### `S20F29R` — QueryPJRecipeXIDList Event Send
**Direction:** Equip → Host  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  PRJOBID
}
```
### `S20F30` — QueryPJRecipeXIDList Event Acknowledge
**Direction:** Host → Equip  

```
{L:2
  {L:n
    {L:9
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
    }
  }
  QPRKEACK
}
```
### `S20F31R` — Pre-Exe Check Event Send
**Direction:** Equip → Host  

```
{L:6
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  PRJOBID
  CHKINFO
}
```
### `S20F32` — Pre-Exe Check Event Acknowledge
**Direction:** Host → Equip  

```
{L:3
  PECRSLT
  {L:n
    {L:10
      TIMESTAMP
      OPEID
      ASSGNID
      COPYID
      REVID
      RecID
      VERID
      TYPEID
      EQID
      RCPBODYA
    }
  }
  PECEACK
}
```
### `S20F33R` — PreSpecifyRecipe Event Send
**Direction:** Equip → Host  

```
{L:5
  OBJID
  OBJTYPE
  OPETYPE
  OPEID
  PRJOBID
}
```
### `S20F34` — PreSpecifyRecipe Event Acknowledge
**Direction:** Host → Equip  

```
PSREACK
```

## Stream 21 - Item Transfer

### `S21F1R` — Item Load Inquire
**Direction:** Host ↔ Equip  

```
{L:4
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
}
```
### `S21F2` — Item Load Grant
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMACK
  ITEMERROR
}
```
### `S21F3R` — Item Send
**Direction:** Host ↔ Equip  

```
{L:5
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
  {L:n
    ITEMPART
  }
}
```
### `S21F4` — Item Send Acknowledge
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMACK
  ITEMERROR
}
```
### `S21F5R` — Item Request
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMTYPE
  ITEMID
}
```
### `S21F6` — Item Data
**Direction:** Host ↔ Equip  

```
{L:7
  ITEMACK
  ITEMERROR
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
  {L:n
    ITEMPART
  }
}
```
### `S21F7R` — Item Type List Request
**Direction:** Host ↔ Equip  

```
ITEMTYPE
```
### `S21F8` — Item Type List Results
**Direction:** Host ↔ Equip  

```
{L:4
  ITEMACK
  ITEMERROR
  ITEMTYPE
  {L:n
    {L:3
      ITEMID
      ITEMLENGTH
      ITEMVERSION
    }
  }
}
```
### `S21F9R` — Supported Item Type List Request
**Direction:** Host ↔ Equip  

```
header only
```
### `S21F10` — Supported Item Type List Result
**Direction:** Host ↔ Equip  

```
{L:3
  ITEMACK
  ITEMERROR
  {L:n
    ITEMTYPE
  }
}
```
### `S21F11` — Item Delete
**Direction:** Host → Equip  

```
{L:2
  ITEMTYPE
  {L:n
    ITEMID
  }
}
```
### `S21F12` — Item Delete Acknowledge
**Direction:** Equip → Host  

```
{L:3
  ITEMACK
  ITEMTYPE
  {L:n
    {L:3
      ITEMID
      ITEMACK
      ITEMERROR
    }
  }
}
```
### `S21F13R` — Request Permission To Send Item
**Direction:** Host ↔ Equip  

```
{L:5
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
  ITEMPARTCOUNT
}
```
### `S21F14` — Grant Permission To Send Item
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMACK
  ITEMERROR
}
```
### `S21F15R` — Item Request
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMTYPE
  ITEMID
}
```
### `S21F16` — Item Request Grant
**Direction:** Host ↔ Equip  

```
{L:7
  ITEMACK
  ITEMERROR
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
  ITEMPARTCOUNT
}
```
### `S21F17R` — Send Item Part
**Direction:** Host ↔ Equip  

```
{L:8
  ITEMTYPE
  ITEMID
  ITEMLENGTH
  ITEMVERSION
  ITEMINDEX
  ITEMPARTCOUNT
  ITEMPARTLENGTH
  ITEMPART
}
```
### `S21F18` — Send Item Part Acknowledge
**Direction:** Host ↔ Equip  

```
{L:2
  ITEMACK
  ITEMERROR
}
```
### `S21F19R` — Item Type Feature Support
**Direction:** Host ↔ Equip  

```
{L:n
  ITEMTYPE
}
```
### `S21F20` — Item Type Feature Support Results
**Direction:** Host ↔ Equip  

```
{L:n
  {L:4
    ITEMACK
    ITEMERROR
    ITEMTYPE
    ITEMTYPESUPPORT
  }
}
```

# Data Items Reference

### `S16F7[R]` — Process Job Alert Notify
**Direction:** Equip → Host  

```
{L:4
  TIMESTAMP
  PRJOBID
  PRJOBMILESTONE
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S16F8` — Process Job Alert Ack
**Direction:** Host → Equip  

```
header only
```
### `S16F9[R]` — Process Job Event Notify
**Direction:** Equip → Host  

```
{L:4
  PREVENTID
  TIMESTAMP
  PRJOBID
  {L:n
    {L:2
      VID
      V
    }
  }
}
```
### `S16F10` — Process Job Event Ack
**Direction:** Host → Equip  

```
header only
```
### `S16F17R` — PRJobDequeue
**Direction:** Host → Equip  

```
{L:m
  PRJOBID
}
```
### `S16F18` — PRJobDequeue Ack
**Direction:** Equip → Host  

```
{L:2
  {L:m
    PRJOBID
  }
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S16F19R` — PRJob List Req
**Direction:** Host → Equip  

```
header only
```
### `S16F20` — PRJob List Data
**Direction:** Equip → Host  

```
{L:m
  {L:2
    PRJOBID
    PRSTATE
  }
}
```
### `S16F21R` — PRJob Create Limit Req
**Direction:** Host → Equip  

```
header only
```
### `S16F22` — PRJob Create Limit Data
**Direction:** Equip → Host  

```
PRJOBSPACE
```
### `S16F23R` — PRJob Recipe Variable Set
**Direction:** Host → Equip  

```
{L:2
  PRJOBID
  {L:m
    {L:2
      RCPPARNM
      RCPPARVAL
    }
  }
}
```
### `S16F24` — PRJob Recipe Variable Ack
**Direction:** Host → Equip  

```
{L:2
  ACKA
  {L:n
    {L:2
      ERRCODE
      ERRTEXT
    }
  }
}
```
### `S16F25R` — PRJob Start Method Set
**Direction:** Host → Equip  

```
{L:2
  {L:m
    PRJOBID
  }
  PRPROCESSSTART
}
```
### `S16F26` — PRJob Start Method Ack
**Direction:** Equip → Host  

```
{L:2
  {L:m
    PRJOBID
  }
  {L:2
    ACKA
    {L:n
      {L:2
        ERRCODE
        ERRTEXT
      }
    }
  }
}
```
### `S16F27R` — Control Job Command
**Direction:** Host → Equip  

```
{L:3
  CTLJOBID
  CTLJOBCMD
  {L:2
    CPNAME
    CPVAL
  }
}
```
### `S16F28` — Control Job Command Ack
**Direction:** Equip → Host  

```
{L:2
  ACKA
  {L:2
    ERRCODE
    ERRTEXT
  }
}
```
### `S16F29` — PRSetMtrlOrder
**Direction:** Host → Equip  

```
PRMTRLORDER
```
### `S16F30` — PRSetMtrlOrder Ack
**Direction:** Equip → Host  

```
ACKA
```
## Acknowledgement Codes

| Item | Format | Description | Values |
|------|--------|-------------|--------|
| **ACKC3** | B:1 | Stream 3 acknowledge | 0=OK |
| **ACKC5** | B:1 | Stream 5 acknowledge | 0=OK |
| **ACKC6** | B:1 | Stream 6 acknowledge | 0=OK |
| **ACKC7** | B:1 | Stream 7 acknowledge | 0=Accepted, 1=Permission not granted, 2=Length error, 3=Matrix overflow, 4=PPID not found, 5=Unsupported mode, 6=Async completion, 7=Storage limit error |
| **ACKC10** | B:1 | Stream 10 acknowledge | 0=Accepted for display, 1=Will not be displayed, 2=Terminal not available |
| **ACKC13** | B:1 | Stream 13 acknowledge | 0=OK, 1=Retryable error, 2=Unknown dataset, 3=Illegal checkpoint, 4=Too many open datasets, 5=Opened too many times, 6=No open dataset, 7=Cannot continue, 8=End of data, 9=Handle in use, 10=Pending transaction |
| **ACKC15** | B:1 | Stream 15 acknowledge | 0=OK, 1=Async completion, 2=DSNAME not found, 3=Permission not granted, 4=Non-specified error |
| **ACKA** | TF:1 | Generic acknowledge | True=OK, False=Failure |
| **CAACK** | U1:1 | Carrier action acknowledge | 0=OK, 1=Invalid command, 2=Cannot perform now, 3=Invalid data/argument, 4=Async completion, 5=Invalid state, 6=Command with errors |
| **CMDA** | B:1 | Command acknowledge | 0=OK, 1=Command does not exist, 2=Not now |
| **COMMACK** | B:1 | Establish comms ack | 0=OK, 1=Denied |
| **CPACK** | B:1 | Remote command param ack | 1=Unknown CPNAME, 2=Illegal value, 3=Illegal format |
| **CSAACK** | U1:1 | Service program run ack | 0=Success, 1=Busy, 2=Invalid SPID, 3=Invalid data |
| **DRACK** | B:1 | Define report acknowledge | 0=OK, 1=Out of space, 2=Invalid format, 3=RPTID already defined, 4=Invalid VID |
| **EAC** | B:1 | Equipment constant ack | 0=OK, 1=Constant not exist, 2=Busy, 3=Value out of range |
| **ERACK** | B:1 | Enable/disable event ack | 0=OK, 1=Denied |
| **GRANT** | B:1 | Multiblock grant | 0=OK, 1=Busy, 2=Insufficient space, 3=Duplicate DATAID |
| **GRANT6** | B:1 | Stream 6 multiblock grant | 0=OK, 1=Busy, 2=Not interested |
| **HCACK** | B:1 | Host command acknowledge | 0=OK completed, 1=Invalid command, 2=Cannot do now, 3=Parameter error, 4=Async completion, 5=Already in condition, 6=Invalid object |
| **LRACK** | B:1 | Link report acknowledge | 0=OK, 1=Out of space, 2=Invalid format, 3=CEID link defined, 4=Invalid CEID, 5=Invalid RPTID |
| **OFLACK** | B:1 | Offline acknowledge | 0=OK |
| **ONLACK** | B:1 | Online acknowledge | 0=OK, 1=Refused, 2=Already online |
| **PPGNT** | B:1 | Process program grant | 0=OK, 1=Already have, 2=No space, 3=Invalid PPID, 4=Busy, 5=Will not accept, 6=Other error |
| **RAC** | U1:1 | Reset acknowledge | 0=OK, 1=Denied |
| **RMACK** | U1:1 | Recipe management ack | 0=OK, 1=Cannot perform, 2=Completed with errors, 3=Async completion, 4=No request exists |
| **RSACK** | B:1 | Ready to send ack | 0=OK, 1=Invalid port, 2=Port occupied, 3=Busy, 4=Lacks permission |
| **RRACK** | B:1 | Request to receive ack | 0=OK, 1=Invalid port, 2=Material not at port, 3=Busy, 4=Lacks permission |
| **SPAACK** | U1:1 | Service program send ack | 0=Done, 1=Error |
| **TIAACK** | B:1 | Trace initialize ack | 0=OK, 1=Too many SVIDs, 2=No more traces, 3=Invalid period, 4=Unknown SVID, 5=Bad REPGSZ |
| **TIACK** | B:1 | Time set acknowledge | 0=OK, 1=Not done |

---

## Common Data Items

| Item | Format | Description | Used By |
|------|--------|-------------|---------|
| **ALCD** | B:1 | Alarm code (>=128 = alarm set) | S5F1, S5F6, S5F8 |
| **ALED** | B:1 | Enable/disable alarm (128=enable, 0=disable) | S5F3 |
| **ALID** | U4:1 | Alarm type ID | S5F1, S5F3, S5F6, S5F8 |
| **ALTX** | A:120 | Alarm text | S5F1, S5F6, S5F8 |
| **CEED** | TF:1 | Collection event enable (true=enabled) | S2F37, S17F5 |
| **CEID** | U4:1 | Collection event ID | S1F23, S2F35, S2F37, S6F11, S6F15 |
| **CENAME** | A:n | Collection event name | S1F24, S2F56 |
| **CPNAME** | A:n | Command parameter name | S2F41, S2F42, S2F49 |
| **CPVAL** | A:n | Command parameter value (any scalar) | S2F41, S4F21 |
| **DATAID** | U4:1 | Data identifier | S2F33, S2F35, S6F11, S6F27 |
| **DATALENGTH** | U4:1 | Data length in bytes | S2F39, S6F5 |
| **DSPER** | A:6 | Data sample period (hhmmss) | S2F23 |
| **DVVALNAME** | A:n | Data value variable name | S1F22 |
| **ECV** | A:n | Equipment constant value (any scalar) | S2F14, S2F15 |
| **ECID** | U4:1 | Equipment constant ID | S2F13, S2F15, S2F29, S2F30 |
| **ECNAME** | A:n | Equipment constant name | S2F30 |
| **ERRCODE** | U4:1 | Error code | S3F18, S4F22 (see values below) |
| **ERRTEXT** | A:80 | Error description text | S3F18, S4F22 |
| **FCNID** | U1:1 | Message type function value | S2F43, S2F44, S2F60 |
| **LENGTH** | U4:1 | Program length in bytes | S2F1, S7F1 |
| **MDLN** | A:20 | Equipment model type | S1F2, S1F13, S1F14 |
| **MEXP** | A:6 | Expected message (SxxFyy) | S9F13 |
| **MHEAD** | B:10 | Message header of received block | S9F1, S9F3, S9F5, S9F7, S9F11 |
| **MID** | A:16 | Material ID | S3F2, S3F7, S4F1, S7F7, S7F8 |
| **OBJID** | A:80 | Object identifier | S1F19, S14F1, S14F2 |
| **OBJSPEC** | A:n | Object specifier | S2F49, S14F1 |
| **OBJTYPE** | A:40 | Object type | S1F19, S14F1 |
| **PPBODY** | B:n | Process program data (any non-list) | S7F3, S7F6 |
| **PPID** | A:80 | Process program ID | S7F1, S7F3, S7F5, S7F6, S7F17 |
| **PTN** | U1:1 | Port number | S3F11-S3F17, S4F1-S4F17 |
| **RCMD** | A:n | Remote command (max 20 chars) | S2F21, S2F41, S2F49 |
| **REPGSZ** | U4:1 | Report group size | S2F23, S17F5 |
| **RPTID** | U4:1 | Report ID | S2F33, S2F35, S6F11, S6F13 |
| **SFCD** | B:1 | Status form code | S1F5, S1F7 |
| **SHEAD** | B:10 | Message header of sent block | S9F9 |
| **SOFTREV** | A:20 | Software revision | S1F2, S1F13, S1F14 |
| **SPID** | A:6 | Service program identifier | S2F1, S2F5, S2F7 |
| **STRID** | U1:1 | Stream value | S2F43, S2F44, S2F60 |
| **SV** | A:n | Status variable value (any type) | S1F4, S1F6, S6F1 |
| **SVID** | U4:1 | Status variable ID | S1F3, S1F11, S1F12, S2F23 |
| **SVNAME** | A:n | Status variable name | S1F8, S1F12 |
| **TEXT** | A:120 | Display text line | S10F1, S10F3, S10F5, S10F9 |
| **TID** | B:1 | Terminal ID | S10F1, S10F3, S10F5, S10F7 |
| **TIME** | A:32 | Date/time value | S2F18, S2F31 |
| **TIMESTAMP** | A:32 | Timestamp | S5F9, S5F11, S16F7, S16F9 |
| **TOTSMP** | U4:1 | Total samples (multiple of REPGSZ) | S2F23, S17F5 |
| **TRID** | A:n | Trace ID | S2F23, S6F1, S6F27, S17F5 |
| **UNITS** | A:n | Units identifier | S1F12, S1F22, S2F30 |
| **V** | A:n | Variable value (any type incl. list) | S6F11, S6F16, S6F20, S6F27 |
| **VID** | A:n | Variable ID | S1F21, S2F33, S2F47, S6F13 |

---

## ERRCODE Values (Common)

| Code | Description |
|------|-------------|
| 0 | OK |
| 1 | Unknown object |
| 2 | Unknown class |
| 3 | Unknown object instance |
| 4 | Unknown attribute type |
| 5 | Read-only attribute |
| 6 | Unknown class |
| 7 | Invalid attribute value |
| 8 | Syntax error |
| 9 | Verification error |
| 10 | Validation error |
| 11 | Object ID in use |
| 12 | Improper parameters |
| 13 | Missing parameters |
| 14 | Unsupported option |
| 15 | Busy |
| 16 | Unavailable |
| 17 | Command not valid in current state |
| 18 | No material altered |
| 19 | Partially processed |
| 20 | All material processed |
| 21 | Recipe specification error |
| 22 | Failure when processing |
| 23 | Failure when not processing |
| 24 | Lack of material |
| 25 | Job aborted |
| 26 | Job stopped |
| 27 | Job cancelled |
| 28 | Cannot change selected recipe |
| 29 | Unknown event |
| 30 | Duplicate report ID |
| 31 | Unknown data report |
| 32 | Data report not linked |
| 33 | Unknown trace report |
| 34 | Duplicate trace ID |
| 35 | Too many reports |
| 36 | Invalid sample period |
| 37 | Group size too large |
| 38 | Recovery action invalid |
| 39 | Busy with previous recovery |
| 40 | No active recovery |
| 41 | Recovery failed |
| 42 | Recovery aborted |
| 43 | Invalid table element |
| 44 | Unknown table element |
| 45 | Cannot delete predefined |
| 46 | Invalid token |
| 47 | Invalid parameter |
| 48 | Load port does not exist |

---

## Material Format Codes (MF)

| Code | Description |
|------|-------------|
| 1 | Wafers |
| 2 | Cassettes |
| 3 | Die |
| 4 | Boats |
| 5 | Ingots |
| 6 | Leadframes |
| 7 | Lots |
| 8 | Magazines |
| 9 | Packages |
| 10 | Plates |
| 11 | Tubes |
| 12 | Waterframes |
| 13 | Carrier (FOUP, SMIF pod, cassette) |
| 14 | Substrate |

---

## Data Type Encoding

| Code | Type | Description |
|------|------|-------------|
| L | List | Ordered list of items |
| B | Binary | Binary data (byte array) |
| TF | Boolean | True/False |
| A | ASCII | ASCII text string |
| I1 | Integer | 1-byte signed integer |
| I2 | Integer | 2-byte signed integer |
| I4 | Integer | 4-byte signed integer |
| I8 | Integer | 8-byte signed integer |
| U1 | Unsigned | 1-byte unsigned integer |
| U2 | Unsigned | 2-byte unsigned integer |
| U4 | Unsigned | 4-byte unsigned integer |
| U8 | Unsigned | 8-byte unsigned integer |
| F4 | Float | 4-byte IEEE float |
| F8 | Float | 8-byte IEEE double |

---

## GEM (E30) Key Messages

The following messages form the core of GEM compliance:

### Communication
- **S1F1/S1F2** - Are You There / On Line Data
- **S1F13/S1F14** - Establish Communications

### State Management
- **S1F15/S1F16** - Request OFF-LINE
- **S1F17/S1F18** - Request ON-LINE

### Status Variables
- **S1F3/S1F4** - Selected Equipment Status Request/Data
- **S1F11/S1F12** - Status Variable Namelist

### Equipment Constants
- **S2F13/S2F14** - Equipment Constant Request/Data
- **S2F15/S2F16** - New Equipment Constant Send/Ack
- **S2F29/S2F30** - EC Namelist Request/Data

### Dynamic Event Reporting
- **S2F33/S2F34** - Define Report
- **S2F35/S2F36** - Link Event Report
- **S2F37/S2F38** - Enable/Disable Event Report
- **S6F11/S6F12** - Event Report Send/Ack
- **S6F15/S6F16** - Event Report Request/Data

### Alarms
- **S5F1/S5F2** - Alarm Report Send/Ack
- **S5F3/S5F4** - Enable/Disable Alarm
- **S5F5/S5F6** - List Alarms Request/Data

### Remote Commands
- **S2F41/S2F42** - Host Command Send/Ack
- **S2F49/S2F50** - Enhanced Remote Command/Ack

### Process Programs
- **S7F1/S7F2** - PP Load Inquire/Grant
- **S7F3/S7F4** - PP Send/Ack
- **S7F5/S7F6** - PP Request/Data
- **S7F17/S7F18** - Delete PP
- **S7F19/S7F20** - PP Directory Request/Data

### Date/Time
- **S2F17/S2F18** - Date and Time Request/Data

### Spooling
- **S2F43/S2F44** - Configure Spooling
- **S6F23/S6F24** - Request Spooled Data

### Terminal Services
- **S10F1/S10F2** - Terminal Request/Ack
- **S10F3/S10F4** - Terminal Display Single/Ack

---

## Time Format

The TIME/TIMESTAMP format is controlled by the `TimeFormat` equipment constant (ECV):

| Value | Format | Example |
|-------|--------|---------|
| 0 | A:12 YYMMDDHHMMSS | `050413093452` |
| 1 | A:16 YYYYMMDDHHMMSScc | `2005041309345240` |
| 2 | ISO 8601 | `2005-04-13T09:34:52.40Z` |

---

## References

- SEMI E5 (SECS-II) - Message Content Standard
- SEMI E30 (GEM) - Generic Equipment Model
- SEMI E37 (HSMS) - High-Speed Message Services
- SEMI E39 (OSS) - Object Services Standard
- SEMI E40 (PJM) - Processing Management
- SEMI E87 (CMS) - Carrier Management Standard
- SEMI E90 (STS) - Substrate Tracking Standard
- SEMI E94 (CJM) - Control Job Management
- SEMI E116 (EPT) - Equipment Performance Tracking
- SEMI E139 - Recipe & Parameter Management (PDE)
- SEMI E170/E171 - SRO Recipe Management

*Source: [hume.com/secs](http://www.hume.com/secs/) - SECS-II Reference*
