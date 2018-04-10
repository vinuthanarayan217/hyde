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

1) Registration Phase: Every entity has to register itself with Auth to become part of the communication network. The registration process is a two-step request-response sequence as shown in Figure 2. To register itself with Auth, an entity sends a message with registration request tag REQREG, its public information (i.e. entity name, group name, public key) and certificate. 

![reg](https://user-images.githubusercontent.com/25291535/38537149-a04e8830-3caa-11e8-8753-35787cd31a1f.png)

Figure 3(a). Auth validates the public information of an entity using local CA’s public key. Auth generates a nonce N1, encrypts it using the entity’s public key and sends nonce verification request message along with its own public information and certificate (Figure 3(b)). On arrival of this message, an entity validates Auth’s public information in a similar manner. It decrypts N1 using its private key, generates another nonce N2, encrypts both N1 and N2 using Auth’s public key and sends it back to Auth. This message is also tagged as nonce verification request message (Figure 3(c)).

![3](https://user-images.githubusercontent.com/25291535/38536797-b3d18ac6-3ca8-11e8-9f12-2cb41d03bf22.png)

After receiving this message, Auth decrypts it and verifies N1. Auth generates distribution key, gets its validity period from Group Table and registers the entity in Entity Table. The registration process is completed at this point. Auth then encrypts N2 along with distribution key and sends it to the entity with registration accepted tag ACPTREG (Figure 3(d)). The entity decrypts the message with registration accepted tag and verifies N2. It acknowledges itself as registered and uses the given symmetric distribution key for all further communication with Auth.

a) Exceptions::
* If either the entity’s public information or N1 is not verified by Auth, it sends back a message with registration reject tag to the entity. 
* If either Auth’s public information or N2 is not verified by an entity, it drops the packet. 

By this procedure, E1 and E2 get registered and obtain DK1 and DK2 as distribution key, respectively.

**2) Session Key Distribution Phase:**

Every entity has to get a SK from Auth to communicate with any other entity in the network. The process of session key distribution is shown in Figure 4 and the formats of transmitted packets are shown in Figure 5.

![4](https://user-images.githubusercontent.com/25291535/38537161-b199a6ec-3caa-11e8-970c-1b69de25c4b0.png)

 To get a session key, E1 symmetrically encrypts access request tag REQACC along with the name of E2 and the duration for which it intends to communicate, using DK1. The duration is specified in minutes. The message header is attached with the symmetrically encrypted message and sent to Auth (Figure 5(a)).
 
 ![5](https://user-images.githubusercontent.com/25291535/38537251-32ef393c-3cab-11e8-89a8-8581763d4726.png)


On getting an encrypted message from E1, Auth generates hash value from cipher text and compares it with the received hash value to verify that integrity and authenticity of the cipher text is not violated. Auth then decrypts cipher text using DK1. It checks in Access Table whether E1 is authorized by the network administrator to communicate with E2. It grants E1 a session with E2 for up to the maximum duration specified in the Access Table. Auth then generates a session key (SK1) and updates the Session Table with related information. Auth encrypts access granted tag ACPTACC, E2’s information (name, IP and Port), SK1 and allowed duration using DK1, and sends a symmetrically encrypted message along with message header to E1(Figure 5(b)). After that, it encrypts access acknowledgment tag ACKACC, E1’s information, SK1 and allowed duration using DK2, and sends a symmetrically encrypted message along with message header to E2 (Figure 5(c)). On receiving an encrypted message from Auth, both E1 and E2 verify the integrity and authenticity of the received message by generating the hash value from cipher text and comparing it with the received hash value. Both entities decrypt message using their respective distribution keys. E1 updates its To Access Table with E2’s information, SK1 and granted duration. E2 updates its From Access Table with E1’s information, SK1 and granted duration. E2 then sends a symmetrically encrypted message with message header auth acknowledgment tag(ACKAUTH) to Auth. 


To Access Table and From Access Table are tables managed by the client and the server entities, respectively, for session related information.

a) Exceptions:
*  If the hash value generated from cipher text is not equal to received hash value, the packet is dropped. 
* If E1 is not authorized to communicate with E2 in Access Table, Auth sends a symmetrically encrypted message with message header and access reject tagRJCTACC to E1.

**3) Communication Phase:**

The process during communication phase is shown in Figure 6 and the message formats are shown in Figure 7. 

![6](https://user-images.githubusercontent.com/25291535/38537364-de4b1a12-3cab-11e8-8e4d-88efabd30731.png)


Being a client entity, E1 has to initiate the communication. To communicate to E2, E1 checks access session availability in its To Access Table and gets E2’s information. E1 encrypts its data to be sent to E2 with request comm tag REQCOMM using SK1. It then sends a symmetrically encrypted message along with message header to E2 (Figure 7(a)).

![7](https://user-images.githubusercontent.com/25291535/38537373-ec90b316-3cab-11e8-95e6-1c27207a2bcc.png)

On receiving an encrypted message from E1, E2 first checks for the availability of access session in its From Access Table. It verifies the integrity and authenticity of the received message as explained above. It decrypts cipher text using SK1 and passes the data to the application layer. If some data is to be sent as a response, the same process is followed, but from E2 to E1 and with response comm tag RESPCOMM (Figure 7(b)).

a) Exceptions:

*  If an access session is not available in To Access Table of E1, it requests Auth to grant access session to communicate with E2 as explained in the previous subsection.

* If a session is not available in From Access Table of E2, then it ignores the message.

* If the hash value generated from cipher text is not equal to received hash value, the packet is dropped.






The proposed communication architecture is implemented using Python and C languages. The python implementation targets non resource-constrained entities that have availability of Python 3.5 interpreter. The C implementation focuses on resource constraints and portability. The implementations use AES-CBC-128 and RSA-1024 as symmetric and asymmetric encryption algorithms, respectively. They use XORed double encryption of data for hash generation.

The python code implementation provides an entire secure communication architecture that includes Auth, server and client entities. Further, a software stack that provides solutions for local CA is also implemented. Using this software stack, files required by all the entities for registration are generated. It uses OpenSSL1 for asymmetric key generation. Auth implementation uses SQLite2 to efficiently manage tables as databases. The server/client entity provides APIs to send/receive messages through proposed secure communication architecture.

The C implementation comprises of server and client entities. It provides portability files, that can be used to port the client/sever entity to any device and/or OS. Additionally, it is highly configurable and so provides freedom to an application developer to adapt the implementation and make it suitable for the corresponding platform. These properties are essential as IoT devices and their OSes are diverse. Because of these properties the foot-print of C implementation depends on its configuration. The minimum requirements are 2.5kB of ROM, 76B of DATA RAM and packet buffer size plus 20B of Stack RAM.

**GitHub Repo:** [Link](https://github.com/sunithan29/embedded_security)


