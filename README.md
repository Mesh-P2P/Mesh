# Mesh

A P2P standard based on the fact that your contacts likely share at least some of yours (In development)

![Diagram](charts/main.svg)

Messages are to be sent via https (?AJAX) POST on port 7575 their data structure is as follows:
  * IP:
    ```
    header:
      {
      from: (?unique per contact) ID of sender of request
      auth: authentication data
      }
    body:
      {
        type: "IP"
        to: ID of the receipient
        from: (?unique per contact) ID of sender
        auth: authentication data
        body: encrypted IP of sender
      }
    ```
  * referral (?better name):
    ```
    header:
      {
      from: (?unique per contact) ID of sender of request
      auth: authentication data
      }
    body:
      {
        type: "referral"
        referent: ID of the Peer refferred to
        gen: distance to referent in hops
      }
    ```
  * connect:
    ```
    header:
      {
      from: (?unique per contact) ID of sender of request
      auth: authentication data
      }
    body:
      {
        type: "connect"
      }
    ```
  * message:
    ```
    header:
      {
      from: (?unique per contact) ID of sender of request
      auth: authentication data
      }
    body:
      {
        type: "message"
        body: encrypted message
      }
    ```
They are to be answered either with an error code when an error occurs or with status 200 and authentication.

Peers have to store:
  1. contacts with ID, IP, public key (encrypt)
  2. refferals with ID of the Peer refferred to, distance to referent in hops, IDs of the Peers to relay to

Peers have to process incoming messages as follows:
![onmessage.svg](charts/onmessage.svg)
