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
* Symmetrically Encrypted Message: It is a concatenation of encrypted message tag, cipher text, and its hash value. (The hash value must also contain the authentication details, for which we use distribution key in generation of hash)

IoT devices are prone to attacks. **Table I** shows frequently observed attacks in a network to which, IoT systems are susceptible. Therefore, there is a need for lightweight, robust security mechanisms that provide authentication, authorization of the participants along with confidentiality and integrity of the data. To meet the need of a security system for constrained nodes, the IETF working group 'DTLS In Constrained Environments (DICE)' is making great efforts to make a DTLS profile for the IoT environment.

![attacks](https://user-images.githubusercontent.com/25291535/38536610-c3d23e58-3ca7-11e8-8bc2-db9ab800e117.png)

## Local CA

In a PKI architecture CA is a third party that provides certificates to servers, in general. In the proposed architecture, we use PKIs initial trust-building process between two parties. So every entity needs to have a certificate signed by CA. But it is expensive to get a certificate from globally well-known CAs and it is limited only to global IP addresses. In addition, it works on Trust Chains that are often not affordable for an embedded device. Therefore, in the proposed architecture, we use a public-private asymmetric key pair to certify/verifypublic modules of all the entities. Henceforth, this publicprivate asymmetric key pair is referred to as local CA.

## B. Auth

Auth is the central body that controls the communication amongst all entities in the network. Being an entity, Auth also needs to have a certificate signed by the local CA. To facilitate communication between entities, Auth requires to maintain four tables as given below:

* **Group Table:** Contains information about the validity period for a given group name. 
* **Entity Table:** Contains information of all the registered entities, which includes Entity Name, Group Name, Public Key, Distribution key, validity, IP address and Destination Port address.
* **Access Table:** Contains information regarding authorization of an entity, i.e. whether an entity is allowed to access a particular entity/group.
* **Session Table:** Contains information regarding active sessions between all the entities in network, such as session keys and their validities. Information is erased if validity of the session is over. 

Group Table and Access Table are used as inputs by Auth. Therefore, the information required in these tables has to be entered initially by the network administrator.

Auth is a master entity that allows or rejects communication between two entities. It is also responsible for authenticating and authorizing an entity, generating and distributing DKs and SKs. Being a central command center, Auth by default becomes a prime target for attackers, therefore an Auth has to be a non resource-constrained device, withstanding all kinds of attacks on it. 

## Stages for Secure Communication

For all the entities, secure communication takes place in three stages as shown in Figure 1. Consider an example of a client entity E1 wants to communicate with a server entity E2. Auth server entity must be running before the communication process starts.

**1) Registration:** Auth verifies the identity and information of the entities and registers them. The entities become part of the network. Communication between each entity and Auth is secured using DKs.

**2) Session Key Distribution:** E1 requests Auth to grant communication session with E2. If allowed, Auth creates a secure link between E1 and E2 by distributing a SK. 

**3) Communication:** Secure communication between E1 and E2 happens using SK.

1) Registration Phase: Every entity has to register itself with Auth to become part of the communication network. The registration process is a two-step request-response sequence as shown in Figure 2. To register itself with Auth, an entity sends a message with registration request tag REQREG, its public information (i.e. entity name, group name, public key) and certificate



The proposed communication architecture is implemented using Python and C languages. The python implementation targets non resource-constrained entities that have availability of Python 3.5 interpreter. The C implementation focuses on resource constraints and portability. The implementations use AES-CBC-128 and RSA-1024 as symmetric and asymmetric encryption algorithms, respectively. They use XORed double encryption of data for hash generation.

The python code implementation provides an entire secure communication architecture that includes Auth, server and client entities. Further, a software stack that provides solutions for local CA is also implemented. Using this software stack, files required by all the entities for registration are generated. It uses OpenSSL1 for asymmetric key generation. Auth implementation uses SQLite2 to efficiently manage tables as databases. The server/client entity provides APIs to send/receive messages through proposed secure communication architecture.

The C implementation comprises of server and client entities. It provides portability files, that can be used to port the client/sever entity to any device and/or OS. Additionally, it is highly configurable and so provides freedom to an application developer to adapt the implementation and make it suitable for the corresponding platform. These properties are essential as IoT devices and their OSes are diverse. Because of these properties the foot-print of C implementation depends on its configuration. The minimum requirements are 2.5kB of ROM, 76B of DATA RAM and packet buffer size plus 20B of Stack RAM.

**GitHub Repo:** [Link](https://github.com/sunithan29/embedded_security)


