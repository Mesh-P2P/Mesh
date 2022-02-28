## Hole Punching
1. The A sends a connect message for B
2. (optional) if unauthorized B responds with Err Unauthorized
3. if B can't connect via the specified type
   1. B responds with Err Unsupported Type and the supported type
   2. A resends the message with a supported type
4. B sends its Port and IP back
5. B tries to connect to A and A tries to connect to B
## SSL
A takes the role of the client, B the role of the server
1. TLS Handshake without any certificates but signed by the server
## Config
refer to [Initializing a connection](Config#initializing-a-connection)

## Nodes accessible without Hole Punching
The Node uses port 7575.
Connecting nodes will not do Hole Punching
## Nodes on a local network
Nodes advertise themselves (without UUID) via udp broadcasts **Needs to be more specific** 
Nodes listen on port 7575. 
Any node on the network connects with that node but doesn't send its UUID. 
If there are multiple nodes on a device, only one will listen on port 7575 (the one, that started first).