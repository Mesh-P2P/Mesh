# Default
## All (except response)
| Byte      | 0-1                 | 2     | 3     | 4-n       |
|-----------|---------------------|-------|-------|-----------|
| Content   | Length of Type Data | ID    | Type  | Type Data |
| Data type | UInt16              | UInt8 | UInt8 |           |
## Response (Type 0)
| Byte      | 0-n     |
|-----------|---------|
| Content   | Message | 
| Data type |         |

if it responds to a forwarded message, the response is forwarded to the sender 
## Direct (Type 1)
| Byte      | 0-n     |
|-----------|---------|
| Content   | Message | 
| Data type |         |
## [Config](config) (Type 2)
| Byte      | 0       | 1-n                |
|-----------|---------|--------------------|
| Content   | SubType | connection options |
| Data type | UInt8   | data from subtype  |
### JSON config (Type 2.0)
| Byte      | 0-n                |
|-----------|--------------------|
| Content   | connection options |
| Data type | JSON               |
### Distance (Type 2.1)
| Byte      | 0-15 | 16                        |
|-----------|------|---------------------------|
| Content   | UUID | Distance (FF for removed) |
| Data type | UUID | UInt8                     |
### Distance without self (Type 2.2)
#### Request:
| Byte      | 0-15 |
|-----------|------|
| Content   | UUID |
| Data type | UUID |
#### Response:
| Byte      | 0        |
|-----------|----------|
| Content   | Distance |
| Data type | UInt8    |
## Broadcast (Type 3)
#### Request:
| Byte      | 0     | 1       | 2-n          |
|-----------|-------|---------|--------------|
| Content   | TTL   | SubType | SubType Data |
| Data type | UInt8 | UInt8   |              |
#### Response:
| Byte      | 0-1                  | 2-n        | n-(n+2)              | (n+2)-m    |...|
|-----------|----------------------|------------|----------------------|------------|---|
| Content   | Length of response a | Response a | Length of response b | Response b |...|
| Data type | UInt16               | Response   | UInt16               | Response   |...|

Broadcasts are forwarded to all contacts with a decremented TTL as long as TTL is bigger than 0.
A node responds with all unique response it got plus its own.
If the same message is received a short time after the first time getting it (has the same hash), it is ignored (empty response)
## Route (Type 4)
#### Request:
| Byte      | 0     | 1-16     | 17      | 18-n         |
|-----------|-------|----------|---------|--------------|
| Content   | TTL   | Receiver | SubType | SubType Data |
| Data type | UInt8 | UUID     | UInt8   |              |
#### Response:
| Byte      | 0-1                  | 2-n        | n-(n+2)              | (n+2)-m    |...|
|-----------|----------------------|------------|----------------------|------------|---|
| Content   | Length of response a | Response a | Length of response b | Response b |...|
| Data type | UInt16               | Response   | UInt16               | Response   |...|
##### Individual Response on failure
| Byte      | 0      |
|-----------|--------|
| Content   | Status |
| Data type | UInt8  |

If the receiver is a connected contact, this is forwarded to it, else if it has a referral entry this is forwarded to them, otherwise it's forwarded to all connected contacts with TTL decremented as long TTL is bigger than 0.
An empty response means that TTL reached 0 before getting to the Receiver, a node responds with all unique response it got.
If the same message is received a short time after the first time getting it (has the same hash), it is ignored (empty response)
### [Connect](connect) (Type 4.0)
**encrypted with RSA**
#### Request:
| Byte      | 0-15 | 16-17  | 18-n |
|-----------|------|--------|------|
| Content   | UUID | Port   | IP   |
| Data type | UUID | UInt16 | IP   |
#### Response:
| Byte      | 0-1    | 2-n |
|-----------|--------|-----|
| Content   | Port   | IP  |
| Data type | UInt16 | IP  |
### Contact Request (Type 4.1)
**encrypted with preshared key**
#### Request:
| Byte      | 0-15 | 16-n       |
|-----------|------|------------|
| Content   | UUID | Public Key |
| Data type | UUID | Public Key |
#### Response:
| Byte      | 0-n        |
|-----------|------------|
| Content   | Public Key | 
| Data type | Public Key |

# IP
| Byte      | 0       | 1-n  |
|-----------|---------|------|
| Content   | Type    | Data |
| Data type | UInt8   |      |
## IPv4
| Byte      | 0      | 1      | 2      | 3      |
|-----------|--------|--------|--------|--------|
| Content   | Part 1 | Part 2 | Part 3 | Part 4 |
| Data type | UInt8  | UInt8  | UInt8  | UInt8  |
## IPv6
| Byte      | 0-15    |
|-----------|---------|
| Content   | IP      | 
| Data type | IPv6    |
# Status
| Status | Meaning                             |
|--------|-------------------------------------|
| 0      | general failure                     |
| 1      | Not reachable (for Route/Broadcast) | 
| 2      | unsupported message type            |
| 3      | Unauthenticated                     |
# Officially recognized types of various Mesh-based programs/standards
## Tunnel (Type 5)
| Byte      | 0     | 1-16     | 2-3    | 4-n     |
|-----------|-------|----------|--------|---------|
| Content   | TTL   | Receiver | ID     | Packet  |
| Data type | UInt8 | UUID     | UInt16 |         |

Tunnels are like Route messages, but the Path persists until it is closed, all messages after the first are treated as responses.