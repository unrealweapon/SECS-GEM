# SEMI E37 — High-Speed SECS Message Services (HSMS)

## Table of Contents

1. [Overview](#overview)
2. [Purpose & Scope](#purpose--scope)
3. [Architecture](#architecture)
4. [Connection Procedures](#connection-procedures)
5. [Message Structure](#message-structure)
6. [Control Messages](#control-messages)
7. [State Machine](#state-machine)
8. [Timers](#timers)
9. [Session Modes](#session-modes)
10. [Error Handling](#error-handling)
11. [Implementation Guidelines](#implementation-guidelines)

---

## Overview

**SEMI E37** defines the **High-Speed SECS Message Services (HSMS)** protocol, which provides a TCP/IP-based transport mechanism for SECS-II messages. It replaces the older SEMI E4 (SECS-I) RS-232 serial protocol with a modern, high-bandwidth, reliable network communication layer.

| Attribute | Value |
|-----------|-------|
| Standard | SEMI E37 |
| Full Name | High-Speed SECS Message Services |
| Abbreviation | HSMS |
| Transport | TCP/IP |
| Port | Configurable (default varies, commonly 5000+) |
| Replaces | SEMI E4 (SECS-I RS-232) |
| Used With | SEMI E5 (SECS-II), SEMI E30 (GEM) |

---

## Purpose & Scope

### Why HSMS?

| Limitation of SECS-I (E4) | HSMS (E37) Solution |
|---------------------------|---------------------|
| RS-232 max ~19.2 kbps | TCP/IP — Gbps speeds |
| Point-to-point only | Network — multiple connections |
| Max cable length ~15m | Unlimited (LAN/WAN) |
| Single session per port | Multiple sessions possible |
| Complex block protocol | Stream-based, simpler |
| No standard discovery | IP-based addressing |

### What HSMS Defines

1. **TCP/IP connection management** — how to establish/terminate connections
2. **Message framing** — how SECS-II messages are encapsulated
3. **Session management** — how logical sessions are created
4. **Control messages** — keepalive, selection, rejection
5. **Timers** — timeouts for various protocol states
6. **Error recovery** — how to handle failures

### What HSMS Does NOT Define

- SECS-II message content (that's E5)
- Equipment behavior (that's E30/GEM)
- Application-level error handling
- Network infrastructure requirements

---

## Architecture

### Protocol Stack

```
┌────────────────────────────────────────┐
│          Application Layer              │
│     (GEM / SECS-II Messages)           │
├────────────────────────────────────────┤
│          HSMS Layer (E37)              │
│   (Framing, Session, Control)          │
├────────────────────────────────────────┤
│          TCP Layer                      │
│   (Reliable byte stream)               │
├────────────────────────────────────────┤
│          IP Layer                       │
│   (Addressing, routing)                │
├────────────────────────────────────────┤
│     Physical Network (Ethernet)        │
└────────────────────────────────────────┘
```

### Roles

HSMS defines two roles for TCP connection establishment:

| Role | Description | TCP Role |
|------|-------------|----------|
| **Passive** (Server) | Listens for incoming connections | TCP Server (listen/accept) |
| **Active** (Client) | Initiates outgoing connections | TCP Client (connect) |

**Typical Configuration:**
- **Equipment** = Passive mode (listens on configured port)
- **Host** = Active mode (connects to equipment IP:port)

> **Note:** The passive/active roles only determine WHO initiates the TCP connection. Either side can send SECS-II messages once the session is established.

### Connection vs. Session

```
┌─────────────────────────────────────────────────────────┐
│                TCP CONNECTION                             │
│  (Physical network link between two endpoints)           │
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │              HSMS SESSION                          │  │
│  │  (Logical communication channel)                   │  │
│  │  - Selected via Select.req/Select.rsp             │  │
│  │  - Carries SECS-II data messages                  │  │
│  │  - Identified by Session ID                       │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Connection Procedures

### Connection Establishment (Active/Passive)

```
Active Entity (Host)                    Passive Entity (Equipment)
        │                                        │
        │                                        │  Listening on port
        │                                        │
        │──── TCP SYN ──────────────────────────→│
        │←─── TCP SYN-ACK ─────────────────────│
        │──── TCP ACK ──────────────────────────→│  TCP Connected
        │                                        │
        │   (T5 timer starts — connect separation)
        │                                        │
        │──── Select.req ───────────────────────→│  Request session
        │←─── Select.rsp (SelectStatus=0) ──────│  Session accepted
        │                                        │
        │   HSMS SELECTED — Ready for SECS-II    │
        │                                        │
```

### Session Selection

After TCP connection, a **Select** exchange establishes the HSMS session:

**Select.req:**
```
┌────────────────────────────────────────────┐
│  Session ID (2 bytes)                       │
│  Header Byte 2: 0x00                        │
│  Header Byte 3: 0x00                        │
│  PType: 0x00                                │
│  SType: 0x01 (Select.req)                   │
│  System Bytes (4 bytes) — transaction ID    │
└────────────────────────────────────────────┘
```

**Select.rsp:**
```
┌────────────────────────────────────────────┐
│  Session ID (2 bytes)                       │
│  Header Byte 2: SelectStatus               │
│  Header Byte 3: 0x00                        │
│  PType: 0x00                                │
│  SType: 0x02 (Select.rsp)                   │
│  System Bytes (4 bytes) — same as request   │
└────────────────────────────────────────────┘
```

**SelectStatus Codes:**

| Code | Meaning |
|------|---------|
| 0 | Communication established (success) |
| 1 | Communication already active |
| 2 | Connection not ready |
| 3 | Connection already established (TCP busy) |
| 4-127 | Reserved |

### Session Deselection

Graceful session termination:

```
Initiator                               Responder
    │                                        │
    │──── Deselect.req ─────────────────────→│
    │←─── Deselect.rsp (Status=0) ─────────│
    │                                        │
    │   Session closed, TCP remains open      │
    │   (or TCP may be closed)               │
```

### Immediate Disconnection (Separate)

Forceful session termination (no response expected):

```
Initiator                               Responder
    │                                        │
    │──── Separate.req ─────────────────────→│  Immediate disconnect
    │                                        │  Close TCP connection
    │   (TCP connection terminated)          │
```

---

## Message Structure

### HSMS Message Format

Every HSMS message (both control and data) has this structure:

```
┌────────────────────────────────────────────────────────────────┐
│  Message Length (4 bytes, unsigned, big-endian)                  │
│  — Total bytes that follow (header + data)                      │
├────────────────────────────────────────────────────────────────┤
│  Message Header (10 bytes)                                      │
├────────────────────────────────────────────────────────────────┤
│  Message Body (0 to N bytes)                                    │
│  — SECS-II encoded data (for data messages)                    │
│  — Empty (for control messages)                                │
└────────────────────────────────────────────────────────────────┘
```

### Message Header (10 bytes)

```
Byte:  0    1    2    3    4    5    6    7    8    9
     ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
     │Session ID│ HB2│ HB3│PTyp│STyp│    System Bytes   │
     │ (W+Dev)  │    │    │    │    │   (Transaction)   │
     └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
```

| Field | Bytes | Description |
|-------|-------|-------------|
| Session ID | 0-1 | Device ID (bit 15 = R-bit for data messages) |
| Header Byte 2 | 2 | For data: W-bit(7) + Stream(0-6). For control: status |
| Header Byte 3 | 3 | For data: Function. For control: 0x00 |
| PType | 4 | Presentation Type (always 0x00 for SECS-II) |
| SType | 5 | Session Type — identifies message type |
| System Bytes | 6-9 | Transaction identifier (must be unique per open transaction) |

### SType Values

| SType | Message Type | Direction | Description |
|-------|-------------|-----------|-------------|
| 0 | **Data Message** | Bidirectional | SECS-II message payload |
| 1 | **Select.req** | Active→Passive | Establish session request |
| 2 | **Select.rsp** | Passive→Active | Establish session response |
| 3 | **Deselect.req** | Bidirectional | End session request |
| 4 | **Deselect.rsp** | Bidirectional | End session response |
| 5 | **Linktest.req** | Bidirectional | Keepalive/connectivity test |
| 6 | **Linktest.rsp** | Bidirectional | Keepalive response |
| 7 | **Reject.req** | Bidirectional | Reject received message |
| 9 | **Separate.req** | Bidirectional | Immediate disconnect (no response) |

### Data Message Example

Sending S1F1 (Are You There?) via HSMS:

```
Message Length: 0x00 0x00 0x00 0x0A  (10 bytes follow — header only)
Header:
  Byte 0-1: 0x00 0x01  (Session/Device ID = 1)
  Byte 2:   0x81        (W-bit=1, Stream=1)
  Byte 3:   0x01        (Function=1)
  Byte 4:   0x00        (PType=0, SECS-II)
  Byte 5:   0x00        (SType=0, Data Message)
  Byte 6-9: 0x00 0x00 0x00 0x01  (System Bytes = transaction 1)
Body:      (empty — S1F1 is header-only)
```

---

## Control Messages

### Select (SType 1/2)

Establishes a logical HSMS session over the TCP connection.

```
Active                                  Passive
  │                                        │
  │─── Select.req (SType=1) ──────────────→│
  │←── Select.rsp (SType=2, Status=0) ────│  Accepted
  │                                        │
  │   OR                                   │
  │                                        │
  │←── Select.rsp (SType=2, Status=1) ────│  Already active
```

### Deselect (SType 3/4)

Gracefully ends the HSMS session.

```
Initiator                               Responder
  │                                        │
  │─── Deselect.req (SType=3) ────────────→│
  │←── Deselect.rsp (SType=4, Status=0) ──│  Accepted
```

**Deselect Status:**

| Code | Meaning |
|------|---------|
| 0 | Deselect successful |
| 1 | Communication not yet established |
| 2 | Communication busy (pending transactions) |

### Linktest (SType 5/6)

Verifies the connection is still alive (keepalive mechanism).

```
Initiator                               Responder
  │                                        │
  │─── Linktest.req (SType=5) ────────────→│
  │←── Linktest.rsp (SType=6) ────────────│  Connection OK
```

**Usage:**
- Either side can initiate
- Sent periodically (configurable interval, e.g., every 30 seconds)
- If no Linktest.rsp within T6 timeout → connection failure
- Used when no other traffic exists to verify connectivity

### Reject (SType 7)

Rejects an unrecognized or inappropriate message.

```
Receiver                                Sender
  │                                        │
  │←── (Invalid message) ─────────────────│
  │─── Reject.req (SType=7) ──────────────→│  Message rejected
```

**Reject Reason Codes (Header Byte 3):**

| Code | Reason |
|------|--------|
| 1 | SType not supported |
| 2 | PType not supported |
| 3 | Transaction not open |
| 4 | Entity not selected |
| 5-255 | Reserved |

### Separate (SType 9)

Immediate disconnection — no response expected.

```
Initiator                               Responder
  │                                        │
  │─── Separate.req (SType=9) ────────────→│  Disconnect immediately
  │                                        │  TCP connection closed
  │   (No response — connection torn down)  │
```

---

## State Machine

### HSMS Connection State Model

```
                         ┌──────────────────────────┐
                         │     NOT CONNECTED         │
                         │  (No TCP connection)      │
                         └─────────────┬────────────┘
                                       │ TCP Connect
                                       ▼
                         ┌──────────────────────────┐
                         │       CONNECTED           │
                         │  (TCP established,        │
         ┌───────────────│   NOT yet selected)       │
         │               └─────────────┬────────────┘
         │                             │ Select.req/rsp success
         │ TCP disconnect              ▼
         │               ┌──────────────────────────┐
         │               │       SELECTED            │
         └───────────────│  (Session active,         │
                         │   SECS-II data flowing)   │
                         └──────────────────────────┘
```

### Detailed State Transitions

| Current State | Event | Action | New State |
|---------------|-------|--------|-----------|
| NOT CONNECTED | TCP connect succeeds | Start T7 | CONNECTED |
| NOT CONNECTED | TCP connect fails | Retry after T5 | NOT CONNECTED |
| CONNECTED | Select.req received | Send Select.rsp(0), cancel T7 | SELECTED |
| CONNECTED | Select.req sent, Select.rsp(0) received | Cancel T7 | SELECTED |
| CONNECTED | T7 timeout (no Select) | Close TCP | NOT CONNECTED |
| CONNECTED | TCP error | — | NOT CONNECTED |
| SELECTED | Data message received | Process SECS-II | SELECTED |
| SELECTED | Deselect.req received | Send Deselect.rsp | CONNECTED |
| SELECTED | Separate.req received | Close TCP | NOT CONNECTED |
| SELECTED | T6 timeout (Linktest) | Close TCP | NOT CONNECTED |
| SELECTED | TCP error | — | NOT CONNECTED |

### Active Mode State Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ACTIVE MODE                                    │
│                                                                      │
│  ┌──────────┐  TCP OK  ┌───────────┐  Select OK  ┌──────────────┐  │
│  │   NOT    │─────────→│ CONNECTED │────────────→│   SELECTED    │  │
│  │CONNECTED │←─────────│ (wait T7) │←────────────│(SECS-II data) │  │
│  └──────────┘  failure  └───────────┘  Deselect   └──────────────┘  │
│       ↑                       │                          │           │
│       │    T5 wait            │ T7 timeout               │ Separate  │
│       └───────────────────────┴──────────────────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
```

### Passive Mode State Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                       PASSIVE MODE                                    │
│                                                                      │
│  ┌──────────┐  Accept  ┌───────────┐  Select.req  ┌──────────────┐ │
│  │LISTENING │─────────→│ CONNECTED │─────────────→│   SELECTED    │ │
│  │(TCP svr) │          │ (wait T7) │←────────────│(SECS-II data) │ │
│  └──────────┘          └───────────┘  Deselect    └──────────────┘ │
│                              │                          │            │
│                              │ T7 timeout               │ Separate   │
│                              └──────────────────────────┘            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Timers

### HSMS Timer Definitions

| Timer | Name | Default | Range | Description |
|-------|------|---------|-------|-------------|
| **T3** | Reply Timeout | 45 sec | 1-120 sec | Time to wait for a SECS-II reply message |
| **T5** | Connect Separation | 10 sec | 1-240 sec | Minimum time between successive TCP connect attempts |
| **T6** | Control Transaction | 5 sec | 1-240 sec | Time to wait for HSMS control message response |
| **T7** | Not Selected | 10 sec | 1-240 sec | Time to wait for Select after TCP connection |
| **T8** | Network Intercharacter | 5 sec | 1-120 sec | Maximum time between successive bytes of a single message |

### Timer Usage Scenarios

#### T3 — Reply Timeout

```
Entity A                                Entity B
  │                                        │
  │─── S1F1 (Data, W=1) ─────────────────→│  Start T3
  │                                        │
  │   ... T3 expires with no reply ...     │
  │                                        │
  │   → Generate S9F9 (Transaction Timeout)│
  │   → Abort transaction                  │
```

#### T5 — Connect Separation

```
Active Entity
  │
  │─── TCP Connect attempt ────→ FAILED
  │
  │   (Wait T5 before retrying)
  │
  │─── TCP Connect attempt ────→ FAILED
  │
  │   (Wait T5 before retrying)
  │
  │─── TCP Connect attempt ────→ SUCCESS
```

#### T6 — Control Transaction Timeout

```
Entity A                                Entity B
  │                                        │
  │─── Linktest.req ──────────────────────→│  Start T6
  │                                        │
  │   ... T6 expires with no response ...  │
  │                                        │
  │   → Connection failure                 │
  │   → Close TCP, go to NOT CONNECTED     │
```

#### T7 — Not Selected Timeout

```
Passive Entity (Equip)                  Active Entity (Host)
  │                                        │
  │←── TCP Connection Accepted ────────────│  Start T7
  │                                        │
  │   ... T7 expires, no Select.req ...    │
  │                                        │
  │   → Close TCP connection               │
  │   → Return to LISTENING                │
```

#### T8 — Network Intercharacter Timeout

```
Receiver
  │
  │←── Bytes 1-4 of message header received
  │
  │   ... T8 timer between each byte ...
  │
  │   If T8 expires before complete message:
  │   → Discard partial message
  │   → Close TCP connection
```

---

## Session Modes

### Single-Session Mode (HSMS-SS)

The most common mode in semiconductor equipment:

| Feature | Description |
|---------|-------------|
| Sessions | Exactly ONE session per TCP connection |
| Session ID | Fixed, configured at setup time |
| Device ID | Maps to session ID |
| Use Case | Single host ↔ single equipment |

```
Host ←──── ONE TCP connection ────→ Equipment
           (ONE HSMS session)
           (ONE Device ID)
```

### General Session Mode (HSMS-GS)

Allows multiple sessions over a single TCP connection (rarely used):

| Feature | Description |
|---------|-------------|
| Sessions | Multiple sessions per TCP connection |
| Session ID | Different for each session |
| Device ID | Multiple devices multiplexed |
| Use Case | Gateway devices, multi-chamber tools |

```
Host ←──── ONE TCP connection ────→ Equipment
           Session 1 (Device 1)
           Session 2 (Device 2)
           Session 3 (Device 3)
```

---

## Error Handling

### Connection Failure Detection

| Method | Description |
|--------|-------------|
| TCP keepalive | OS-level (less reliable, long timeout) |
| Linktest | Application-level keepalive (preferred) |
| T6 timeout | Linktest response timeout → failure |
| T7 timeout | No Select after connect → close |
| T8 timeout | Incomplete message → close |
| TCP reset | Immediate connection loss |

### Error Recovery Procedures

#### Scenario 1: Network Interruption

```
Host                                    Equipment
  │                                        │
  │═══════ NETWORK FAILURE ═══════════════│
  │                                        │
  │   Linktest.req → no response           │   Linktest.req → no response
  │   T6 expires                           │   T6 expires
  │   Close TCP socket                     │   Close TCP socket
  │   → NOT CONNECTED                      │   → NOT CONNECTED (LISTENING)
  │                                        │
  │   Wait T5                              │
  │                                        │
  │═══════ NETWORK RESTORED ══════════════│
  │                                        │
  │──── TCP Connect ──────────────────────→│   Accept
  │──── Select.req ───────────────────────→│
  │←─── Select.rsp (0) ──────────────────│   Session restored
  │                                        │
  │──── S1F13 (Establish Comms) ──────────→│   Re-establish SECS-II
  │←─── S1F14 (COMMACK=0) ────────────────│
```

#### Scenario 2: Equipment Power Cycle

```
Host                                    Equipment
  │                                        │
  │   TCP error (RST/FIN)                  │   POWER OFF
  │   → NOT CONNECTED                      │
  │                                        │   POWER ON
  │   Wait T5, attempt connect             │   Start listening
  │                                        │
  │──── TCP Connect ──────────────────────→│
  │──── Select.req ───────────────────────→│
  │←─── Select.rsp (0) ──────────────────│
  │                                        │
  │←─── S1F13 (Equipment-initiated) ──────│   Equipment requests comms
  │──── S1F14 (COMMACK=0) ────────────────→│
```

#### Scenario 3: Duplicate Connection Attempt

```
Active (Host)                           Passive (Equipment)
  │                                        │
  │   (Already has active connection)      │
  │                                        │
  │──── TCP Connect (2nd attempt) ────────→│   
  │←─── TCP Accept ───────────────────────│   New TCP connection
  │──── Select.req ───────────────────────→│
  │←─── Select.rsp (Status=1: already active)│  Rejected
  │                                        │
  │   Close 2nd TCP connection             │
  │   Continue using existing connection   │
```

### System Bytes Management

- **System Bytes** must be unique for each open (pending reply) transaction
- Both sides maintain their own System Bytes counters
- Reply messages echo the System Bytes from the request
- Typically implemented as incrementing counter (wraps at 0xFFFFFFFF)

```
Host sends:   SystemBytes = 0x00000001 → S1F1
Equipment:    SystemBytes = 0x00000001 → S1F2 (echoed)

Host sends:   SystemBytes = 0x00000002 → S2F13
Equipment:    SystemBytes = 0x00000002 → S2F14 (echoed)
```

---

## Implementation Guidelines

### Configuration Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| IP Address | Equipment network address | Equipment-specific |
| Port Number | TCP port to listen/connect | 5000-65535 |
| Connect Mode | Active or Passive | Equipment=Passive |
| Session ID | Device identifier | 0-32767 |
| T3 | Reply timeout | 45 sec |
| T5 | Connect separation | 10 sec |
| T6 | Control timeout | 5 sec |
| T7 | Not-selected timeout | 10 sec |
| T8 | Intercharacter timeout | 5 sec |
| Linktest Interval | Keepalive frequency | 30-60 sec |

### Best Practices

1. **Always implement Linktest** — Don't rely on TCP keepalive alone
2. **Use unique System Bytes** — Avoid transaction confusion
3. **Handle partial reads** — TCP is a stream protocol, messages may be fragmented
4. **Implement T7** — Prevent abandoned connections from consuming resources
5. **Log all control messages** — Essential for debugging connection issues
6. **Support both roles** — Some installations swap Active/Passive
7. **Buffer management** — Handle large SECS-II messages efficiently
8. **Thread safety** — Multiple threads may send/receive simultaneously
9. **Graceful shutdown** — Use Deselect before closing connection
10. **Connection monitoring** — Track connection state for reporting to operators

### Message Reception Algorithm

```pseudocode
function receiveMessage():
    // Read 4-byte length prefix
    lengthBytes = readExactly(4, timeout=T8)
    messageLength = bigEndianToUint32(lengthBytes)
    
    if messageLength < 10:
        error("Invalid message length")
        closeConnection()
        return
    
    // Read header (10 bytes)
    header = readExactly(10, timeout=T8)
    
    // Read body (remaining bytes)
    bodyLength = messageLength - 10
    if bodyLength > 0:
        body = readExactly(bodyLength, timeout=T8)
    else:
        body = empty
    
    // Determine message type from SType
    stype = header[5]
    
    switch stype:
        case 0:  handleDataMessage(header, body)
        case 1:  handleSelectRequest(header)
        case 2:  handleSelectResponse(header)
        case 3:  handleDeselectRequest(header)
        case 4:  handleDeselectResponse(header)
        case 5:  handleLinktestRequest(header)
        case 6:  handleLinktestResponse(header)
        case 7:  handleReject(header)
        case 9:  handleSeparate(header)
        default: sendReject(header, reason=1)
```

### Typical Connection Sequence (Complete)

```
1. Equipment powers on
2. Equipment enters Passive mode, starts listening on configured port
3. Host detects equipment (or is configured with IP:port)
4. Host initiates TCP connection (Active mode)
5. TCP three-way handshake completes
6. Host sends Select.req
7. Equipment responds Select.rsp (Status=0)
8. HSMS session is now SELECTED
9. Equipment sends S1F13 (Establish Communications)
   OR Host sends S1F13
10. Other side responds S1F14 (COMMACK=0)
11. SECS-II/GEM communication begins
12. Periodic Linktest.req/rsp maintain connection awareness
```

### Wire Protocol Examples

#### Example: S1F13 (Establish Communications Request)

```
Hex dump:
00 00 00 0A        ← Length = 10 (header only)
00 01              ← Session ID = 1
81                 ← W-bit=1, Stream=1
0D                 ← Function=13
00                 ← PType=0 (SECS-II)
00                 ← SType=0 (Data Message)
00 00 00 05        ← System Bytes = 5
```

#### Example: Linktest.req

```
Hex dump:
00 00 00 0A        ← Length = 10 (header only)
FF FF              ← Session ID = 0xFFFF (control message)
00                 ← Header Byte 2 = 0
00                 ← Header Byte 3 = 0
00                 ← PType = 0
05                 ← SType = 5 (Linktest.req)
00 00 00 0A        ← System Bytes = 10
```

#### Example: Select.req

```
Hex dump:
00 00 00 0A        ← Length = 10 (header only)
00 01              ← Session ID = 1
00                 ← Header Byte 2 = 0
00                 ← Header Byte 3 = 0
00                 ← PType = 0
01                 ← SType = 1 (Select.req)
00 00 00 01        ← System Bytes = 1
```

---

## Appendix: Common Issues & Troubleshooting

| Issue | Possible Cause | Resolution |
|-------|---------------|------------|
| Cannot connect | Wrong IP/port, firewall | Verify network config, check firewall rules |
| Select rejected (status=1) | Session already exists | Close existing connection first |
| T7 timeout | Host too slow to Select | Increase T7 or fix host startup |
| Frequent disconnections | Linktest interval too short | Tune T6 and Linktest timing |
| Message corruption | T8 too short, network issues | Check network health, increase T8 |
| System Bytes mismatch | Bug in counter logic | Ensure unique per open transaction |
| Connection refused | Equipment not listening | Verify equipment state, port config |
| Half-open connections | No Linktest, stale TCP | Enable Linktest, reduce T6 |

---

*Document Version: 1.0*  
*Last Updated: 2026-05-21*  
*Based on: SEMI E37-0303 Standard*
