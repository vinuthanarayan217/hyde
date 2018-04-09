---
layout: post
title: "Decentralizing authentication and authorization in embedded devices"
---

The proposed architecture uses the idea of decentralizing authentication and authorization by using local Certifying Authority(CA). It also off-loads the authentication and authorization processes from embedded devices to a local server.

Below, we define few terms that are used in this project.

* Entity: Every device participating in the proposed architecture.
* Entity Name: Identifies the entity uniquely.
* Group Name: Depending on its properties, each entity belongs to exactly one group, identified by “Group Name”.
* Server Entity: An entity that provides a service on request.
* Client Entity: An entity that initiates service requests.
* Local CA: A local certifying authority that signs every entity’s public key. This allows every entity to verify credentials of any other   entity in the network using CA’s public key.
* Certificate: Certificate of an entity is it’s public information (public key, name etc.) encrypted using CA’s private key. The process     of generating a certificate is called signing.  Auth: Special type of server entity that takes care of authentication and authorization   on behalf of embedded devices. 
* Distribution key(DK): Symmetric key used for communication between an entity and Auth.
* Session Key(SK): Symmetric key used for communication between any two entities in the network.
* Message Header: Header used in all communications between entities in the network. It is always sender entity’s name, terminated by       symbol colon(:). This allows the receiver entity to identify the sender entity. 
* Tag: Tag is a first byte of every message communicated, following message header. It is used to differentiate between various types of     messages. Here the tag names will include an underscore( ) symbol. 
* Nonce: Nonce is a random number generated to verify the identity of a certificate owner. The random number is encrypted using public key   signed in certificate and sent to the owner. In return, the same random number is expected, since only the owner possesses private key,   only it can decrypt and send it back. 
