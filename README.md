# Mesh

A P2P standard made for realtime communication based on the fact that your contacts likely share at least some of yours (In development)

![Diagram](charts/main.svg)

Messages are to be sent via https (?AJAX) POST to port 7575 their data structure is as follows (header and body are encrypted at a local level):
  * IP:
    ```
    header:
      {
        from: (?unique per contact) UUID of sender of request
        secret: secret
        type: "IP"
	(optional) pub_key: public key of sender
      }
    request body:
      {
        to: UUID of the receipient
        encrypted: {
	  from: (?unique per contact) UUID of sender
          secret: secret
          IP: IP:PORT of sender
	}
        hops: distance to sender in hops
      }
    response body:
    	{
	  encrypted: {
	    IP: IP:PORT of receipient
	    secret: secret
	  }
	}
    ```
  * referral (?better name):
    ```
    header:
      {
        from: (?unique per contact) UUID of sender of request
        secret: secret
        type: "referral"
      }
    body:
      {
        referent: UUID of the Peer refferred to
        hops: distance to referent in hops
      }
  * message:
    ```
    header:
      {
        from: (?unique per contact) UUID of sender of request
        secret: secret
        type: "message"
      }
    body: message
  * contact request:
    ```
    header:
      {
        from: (?unique per contact) UUID of sender of request
        secret: secret
        type: "contact_req"
	(optional) pub_key: public key of sender
      }
    request body:
      {
      	pub_key: public key of the sender
	from: UUID of sender
      	to: UUID of Peer of which contact is requested
      }
    response body:
      encrypted {
      	pub_key: public key of Peer of which the public key is requested
      	auth_phrase: auth phrase
      	secret: secret
      } 
    ```
Clarifications:
	1. headers are for message type and direct connection information.
	2. the optional pub_key part of the header is for first contact (peers with domains) with the mesh
They are to be answered either with an error code when an error occurs or with status 200, the encrypted secret and the IP:PORT they received the message from.

Peers store:
  1. contacts: UUID, IP, public key (encrypt), secret
  2. refferals: UUID of the Peer refferred to, distance to referent in hops, UUIDs of the Peers to relay to
  3. information about self: private key (decrypt), public key, UUID, IP:PORT

Peers process incoming messages as follows:

![onmessage.svg](charts/onmessage.svg)

Peers do the following when sending messages:

![sendmessage.svg](charts/sendmessage.svg)

Peers add contacts as follows:

![addcontact.svg](charts/addcontact.svg)

The direct authentication happens as follows:

![auth.svg](charts/auth.svg)
