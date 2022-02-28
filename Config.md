## Options
 - [Max bandwidth (excluding direct)](#max-bandwidth-excluding-direct)
 - [Distances](#distances) (array)
 - UUIDs
 - [Supported Message types](#supported-message-types) (array)
 - [Supported Standard versions](#supported-standard-versions) (array)
## Initializing a connection
Both nodes send their settings as JSON (Max bandwidth; UUID; Supported Message types;Supported Standard versions)
## Max bandwidth (excluding direct)
sets how much data can be sent within a second (excluding direct messages)
 1. One node sends their preferred bandwidth
 2. The other node saves it
 3. The lower Max bandwidth of the contacts is used
## Supported Message types
Only the message types that both nodes support are used
## Supported Standard versions
The newest version that both nodes Support is used
## Distances
### Distance change
Getting new distance:
 1. If any contact has a distance equal or lower than your own
    1. Set own distance one higher than lowest of neighbors (if over limit remove)
 2. Else
    1. Send message [type 1.2](Messages#distance-without-self-type-12) to your contacts (excluding contact whose distance changed/was excluded)
    2. Set own distance one higher than the lowest response (if over limit remove)

When getting asked for your distance excluding contact ([message type 1.2](Messages#distance-without-self-type-12)):
 1. Get new distance (excluding the contact, that asked)
 2. Respond with that distance (if over limit respond with FF)