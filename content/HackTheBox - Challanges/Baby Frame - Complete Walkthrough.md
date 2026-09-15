
# #HTB 


![[Pasted image 20260915105906.png|700]]


# HTB Baby Frame Writeup

**Category:** Satellite Technology  
**Difficulty:** Very Easy  
**Platform:** HackTheBox

---

## Challenge Description

> A recently recovered experimental spacecraft broadcasting under spacecraft ID 12 has entered visibility range. Ground telemetry suggests that one onboard diagnostic application remains active on APID 42 over virtual channel 3. Mission operators believe the service is waiting for a single correctly formatted CCSDS space packet containing the user payload HEALTHCHECK.
> 
> Your task is to establish communication with the spacecraft and trigger the diagnostic response.

We are provided with a zip file containing a minimal `client.py` template with two incomplete functions:
- `generate_space_packet()`
- `generate_tc_frame()`

The objective is to implement these functions according to the official CCSDS specifications and successfully communicate with the remote spacecraft simulator.

---

## Initial Analysis

The provided client performs the following operations:
```python
space_packet = generate_space_packet(
    apid=42,
    packet_count=0,
    payload=b"TEST_PAYLOAD"
)

frame = generate_tc_frame(
    spacecraft_id=12,
    virtual_channel_id=3,
    tc_packet_count=0,
    payload=space_packet
)

payload = frame + space_packet
```

From the challenge description, we know the required values:

|Field|Value|
|---|---|
|Spacecraft ID|12|
|Virtual Channel|3|
|APID|42|
|Payload|`HEALTHCHECK`|

The two protocol references included with the challenge immediately point us toward the official CCSDS standards:
- **CCSDS Space Packet Protocol**
- **CCSDS Telecommand (TC) Space Data Link Protocol**    

This indicates that the challenge is primarily a **protocol implementation exercise** rather than a traditional exploitation challenge.

---

## Understanding the CCSDS Space Packet

A CCSDS Space Packet begins with a fixed **6-byte Primary Header**.
```
 0                   1                   2                   3
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |Ver|Type|SHF|               APID                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |Sequence Flags|      Packet Sequence Count                     |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |            Packet Length                                      |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The header consists of three 16-bit words.

### First Header Word

The first word contains:
- **Version** = 0
- **Packet Type** = 1 (Telecommand)
- **Secondary Header Flag** = 0
- **APID** = 42

This is packed as:
```python
word1 = (version << 13) | (packet_type << 12) | (secondary_header << 11) | apid
```

Which evaluates to **`0x102A`**.

### Second Header Word

The second word stores:
- **Sequence Flags** = `0b11` (Unsegmented Packet)
- **Packet Sequence Count** = 0

This becomes:
```python
word2 = (0b11 << 14) | packet_count
```

Which evaluates to **`0xC000`**.

### Packet Length

Unlike most protocols, CCSDS stores:
```python
Packet Length = Data Length - 1
```

The payload is `HEALTHCHECK`, which is **11 bytes**. Therefore:
```python
Length = 10 (0x000A)
```

### Final Space Packet

The packet layout becomes:
```
+----------------------+
| Primary Header (6B)  |
+----------------------+
| HEALTHCHECK          |
+----------------------+
```



---

## Understanding the Telecommand Frame

The Telecommand Transfer Frame acts as the **transport layer** for the Space Packet.

The primary header contains:
- **Spacecraft ID**
- **Virtual Channel ID**
- **Frame Length**
- **Sequence Number**

For this challenge:
```
Spacecraft ID = 12
VCID          = 3
Sequence      = 0
```

The interesting observation is that the provided template later performs:

```python
payload = frame + space_packet
```

instead of embedding the packet inside `generate_tc_frame()`.

Therefore `generate_tc_frame()` only returns the **5-byte TC Primary Header**, while the client appends the Space Packet afterwards.

The transmitted data therefore looks like:
```
+--------------------+
| TC Header (5B)     |
+--------------------+
| Space Packet       |
+--------------------+
```

This matches the expected packet layout for the challenge service.



---

## Packet Construction

### Space Packet

```python
def generate_space_packet(apid: int, packet_count: int, payload: bytes) -> bytes:
    word1 = (0 << 13) | (1 << 12) | (0 << 11) | (apid & 0x7FF)
    # Word 2: Sequence Flags(2b)=0b11 (Standalone) | Sequence Count(14b)
    word2 = (0b11 << 14) | (packet_count & 0x3FFF)
    # Word 3: Packet Data Length = (payload length) - 1
    word3 = len(payload) - 1
    # Pack the 6-byte primary header (big-endian) and append the payload
    packet = struct.pack(">HHH", word1, word2, word3) + payload
    return packet
```


### TC Frame Header

```python
def generate_tc_frame( spacecraft_id: int, virtual_channel_id: int, tc_packet_count: int, payload: bytes) -> bytes:
    # Word 1: Version(2b)=0 | Bypass(1b)=0 | Ctrl(1b)=0 | Spare(2b)=0 | SCID(10b)
    word1 = (0 << 14) | (0 << 13) | (0 << 12) | (0 << 10) | (spacecraft_id & 0x3FF)
    # Word 2: VCID(6b) | Frame Length(10b)
    # Frame Length = (Total frame length) - 1
    # Total = 5 (TC header) + len(payload) (space packet) + 2 (CRC)
    frame_length = 5 + len(payload) - 1
    word2 = ((virtual_channel_id & 0x3F) << 10) | (frame_length & 0x3FF)
    # Word 3: VCFC(8b) | Spare(8b)
    # Pack as 2 bytes: first byte = count, second byte = 0
    frame = struct.pack(">HHB", word1, word2, tc_packet_count & 0xFF)
    return frame
```

Where the application payload is:
```python
b"HEALTHCHECK"
```



---

## Communication Flow

The completed packet follows this structure:
```
TCP Connection
                      │
                      ▼
        ┌──────────────────────────┐
        │ TC Transfer Frame Header │
        └──────────────────────────┘
                      │
                      ▼
        ┌──────────────────────────┐
        │ CCSDS Space Packet       │
        ├──────────────────────────┤
        │ Primary Header (6 bytes) │
        ├──────────────────────────┤
        │ HEALTHCHECK              │
        └──────────────────────────┘
                      │
                      ▼
          Diagnostic Service (APID 42)
```

The onboard diagnostic application validates:
- Spacecraft ID = **12**
- Virtual Channel = **3**
- APID = **42**
- Payload = **`HEALTHCHECK`**

If all fields are correctly encoded according to the CCSDS specifications, the spacecraft responds with the challenge flag.

---

## Result

![[Pasted image 20260915110053.png]]


## Challange Solved

![[Pasted image 20260915110029.png|700]]


---

## Conclusion

Although classified as a "Very Easy" challenge, **HTB Baby Frame** provides an excellent introduction to real-world satellite communication protocols and demonstrates how spacecraft exchange telecommands using CCSDS standards. It reinforces the importance of reading protocol specifications carefully — especially the subtle off-by-one encoding rules that are unique to CCSDS.

This challenge is highly recommended for anyone interested in aerospace cybersecurity or embedded protocol analysis.

---
