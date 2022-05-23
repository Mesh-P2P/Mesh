# Mesh Specification
## Table of Contents
0. Preliminaries
   1.  Terminology
   2.  Format
1. Base Data Envelope
2. Message Types
3. Common Data Definitions
4. 3rd Party Message Types
5. Spec Changes (ToDo)
## 0. Preliminaries
### 0.i Terminology
* (opt) - optional
* node - an instance of a Mesh implementation
* contact - a node that is explicitly allowed to connect to a node
* distance - smallest number of edges (direct connections) connecting two nodes
### 0.ii Format
"2. Message Types" is formated as follows:
#### 'name' (Type 'number')
##### Request (opt)
| Byte      | 0-a            | a-(a+b)        |...|
|-----------|----------------|----------------|---|
| Content   | Data A         | Data B         |...|
| Data type | Type of Data A | Type of Data B |...|
##### Response (opt)
| Byte      | 0-a            | a-(a+b)        |...|
|-----------|----------------|----------------|---|
| Content   | Data A         | Data B         |...|
| Data type | Type of Data A | Type of Data B |...|

(opt)


Description, Use Cases and Reaction

##### 'name' (Type 'number'.'number') (opt)
Same as any Message Type (opt)
## 1. Base Data Envelope
| Byte      | 0-1                 | 2     | 3     | 4-n       |
|-----------|---------------------|-------|-------|-----------|
| Content   | Length of Type Data | ID    | Type  | Type Data |
| Data type | UInt16              | UInt8 | UInt8 |           |
## 2. Message Types
### Response (Type 0)
| Byte      | 0-n     |
|-----------|---------|
| Content   | Message | 
| Data type |         |

A response to a message, its Message and reaction is specified in the Response part of the Message Type.
### Direct (Type 1)
| Byte      | 0-n     |
|-----------|---------|
| Content   | Message | 
| Data type |         |

A direct message to the connected node, its Message and reaction depends on the application.
### Config (Type 2)
| Byte      | 0       | 1-n                |
|-----------|---------|--------------------|
| Content   | SubType | connection options |
| Data type | UInt8   | data from subtype  |

A message concerning the configuration of the connection.
#### JSON config (Type 2.0)
| Byte      | 0-n                |
|-----------|--------------------|
| Content   | connection options |
| Data type | JSON               |
##### Options
 - maxBandwidth (excluding direct)
 - id (this node)
 - messageTypes (array)
 - standardVersions (array)
##### Initializing a connection
Both nodes send their settings as JSON (maxBandwidth; id (this node); messageTypes; standardVersions)

The node acting as the server sends all of their distances via messages type 2.3, the other node changes their distances accordingly, if its distance is more than 2 smaller,
then it sends its distance (type 2.1)
##### maxBandwidth (excluding direct)
sets how much data can be sent within a second (excluding direct messages)
 1. One node sends their preferred bandwidth
 2. The other node saves it
 3. The lower Max bandwidth of the contacts is used
##### messageTypes
Only the message types that both nodes support are used
##### standardVersions
The newest version that both nodes Support is used
#### Distance (Type 2.1)
| Byte      | 0-15 | 16                        |
|-----------|------|---------------------------|
| Content   | UUID | Distance (FF for removed) |
| Data type | UUID | UInt8                     |

Used to tell contacts its distance to a node (sent on change), react as follows:
 1. If any contact has a distance equal to or lower than your own
    1. Set own distance one higher than lowest of neighbors (if over limit remove)
 2. Else
    1. Send message type 2.2 to your contacts (excluding contact whose distance changed/was excluded)
    2. Set own distance one higher than the lowest response (if over limit remove)
#### Distance without self (Type 2.2)
##### Request:
| Byte      | 0-15 |
|-----------|------|
| Content   | UUID |
| Data type | UUID |
##### Response:
| Byte      | 0        |
|-----------|----------|
| Content   | Distance |
| Data type | UInt8    |

Used to calculate distance, react as follows:
 1. Get new distance (excluding the contact, that asked) (see Type 2.1)
 2. Respond with that distance (if over limit respond with FF)
#### Distance Sync (Type 2.3)
| Byte      | 0-15 | 16                        | 17-32 | 33                        |...|
|-----------|------|---------------------------|-------|---------------------------|---|
| Content   | UUID | Distance (FF for removed) | UUID  | Distance (FF for removed) |...|
| Data type | UUID | UInt8                     | UUID  | UInt8                     |...|

Sent at the start of a connection. Sent repeatedly as needed.
### Broadcast (Type 3)
##### Request:
| Byte      | 0     | 1       | 2-n          |
|-----------|-------|---------|--------------|
| Content   | TTL   | SubType | SubType Data |
| Data type | UInt8 | UInt8   |              |
##### Response:
| Byte      | 0-1                  | 2-n        | n-(n+2)              | (n+2)-m    |...|
|-----------|----------------------|------------|----------------------|------------|---|
| Content   | Length of response a | Response a | Length of response b | Response b |...|
| Data type | UInt16               | Response   | UInt16               | Response   |...|

Broadcasts are forwarded to all contacts with a decremented TTL as long as TTL is bigger than 0.
A node responds with all unique response it got plus its own.
If the same message is received a short time after the first time getting it (has the same hash), it is ignored (empty response).
A TTL limit (>3) may be set to reduce traffic, if a message has a higher TTL, it is forwarded, with the TTL set one lower than the limit.
### Route (Type 4)
##### Request:
| Byte      | 0     | 1-16     | 17      | 18-n         |
|-----------|-------|----------|---------|--------------|
| Content   | TTL   | Receiver | SubType | SubType Data |
| Data type | UInt8 | UUID     | UInt8   |              |
##### Response:
| Byte      | 0-1                  | 2-n        | n-(n+2)              | (n+2)-m    |...|
|-----------|----------------------|------------|----------------------|------------|---|
| Content   | Length of response a | Response a | Length of response b | Response b |...|
| Data type | UInt16               | Response   | UInt16               | Response   |...|
###### Individual Response on failure
| Byte      | 0      |
|-----------|--------|
| Content   | Status |
| Data type | UInt8  |

If the receiver is a connected contact, this is forwarded to it, else if it has a distance this is forwarded to n amount of contacts (<4) whith a lower distance at random, otherwise it's forwarded to all connected contacts with TTL decremented as long TTL is bigger than 0.
An empty response means that TTL reached 0 before getting to the Receiver, a node responds with all unique response it got.
If the same message is received a short time after the first time getting it (has the same hash), it is ignored (empty response).
A TTL limit (>3) may be set to reduce traffic, if a message has a higher TTL, it is forwarded, with the TTL set one lower than the limit.
#### [Connect](connect) (Type 4.0)
**encrypted with RSA**
##### Request:
| Byte      | 0-15 | 16-17  | 18-n |
|-----------|------|--------|------|
| Content   | UUID | Port   | IP   |
| Data type | UUID | UInt16 | IP   |
##### Response:
| Byte      | 0-1    | 2-n |
|-----------|--------|-----|
| Content   | Port   | IP  |
| Data type | UInt16 | IP  |

Used for hole punching as follows:
1. The A sends a connect message for B
2. (optional) if unauthorized B responds with Err Unauthorized
3. if B can't connect via the specified type
   1. B responds with Err Unsupported Type and the supported type
   2. A resends the message with a supported type
4. B sends its Port and IP back
5. B tries to connect to A and A tries to connect to B
#### Contact Request (Type 4.1)
**encrypted with preshared key**
##### Request:
| Byte      | 0-15 | 16-n       |
|-----------|------|------------|
| Content   | UUID | Public Key |
| Data type | UUID | Public Key |
##### Response:
| Byte      | 0-n        |
|-----------|------------|
| Content   | Public Key | 
| Data type | Public Key |

Used to exchange contact information between nodes, with only the id and a preshared key of one node.
## 3. Common Data Definitions
### IP
| Byte      | 0       | 1-n  |
|-----------|---------|------|
| Content   | Type    | Data |
| Data type | UInt8   |      |
#### IPv4
| Byte      | 0      | 1      | 2      | 3      |
|-----------|--------|--------|--------|--------|
| Content   | Part 1 | Part 2 | Part 3 | Part 4 |
| Data type | UInt8  | UInt8  | UInt8  | UInt8  |
#### IPv6
| Byte      | 0-15    |
|-----------|---------|
| Content   | IP      | 
| Data type | IPv6    |
### Status
| Status | Meaning                             |
|--------|-------------------------------------|
| 0      | general failure                     |
| 1      | Not reachable (for Route/Broadcast) | 
| 2      | unsupported message type            |
| 3      | Unauthenticated                     |
## 4. 3rd Party Message Types
### Tunnel (Type 5)
| Byte      | 0     | 1-16     | 2-3    | 4-n     |
|-----------|-------|----------|--------|---------|
| Content   | TTL   | Receiver | ID     | Packet  |
| Data type | UInt8 | UUID     | UInt16 |         |

Tunnels are like Route messages, but the Path persists until it is closed, all messages after the first are treated as responses.
