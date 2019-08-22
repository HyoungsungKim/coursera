# Week 4

## 1.1 Basic Security Concepts

### Security Threats

- Leakage
  - Unauthorized access to service or data
  - e.g., Someone knows your bank balance
- Tampering
  - Unauthorized modification of service or data
  - e.g. Someone modifies your bank balance
- Vandalism
  - interference with normal service, without direct gain to attackers
  - e.g. Denial of Service attack

### Common Attacks

- Eavesdropping(도청)
  - Attacker taps into network
  - Someone intercept all the messages in the network, then message are eavesdropped
- Masquerading(가장, 변장)
  - Attack pretends to be someone else, i.e, identity theft
- Message tampering(부당 변경)
  - Attack modifies messages
  - For instance, you can prove yourself to be someone else that you are not
- Replay attack
  - Attack replays old messages
- Denial-of-service : bombard a port

### Addressing The Challenges: CIA Properties

- Confidentiality
  - Protection against disclosure to unauthorized individuals
  - Addresses leakage threat
- Integrity
  - protection against unauthorized alteration or corruption
  - Addresses Tampering threat
- Availability
  - Service/data is always readable/writable
  - Addresses vandalism threat

### Policies vs Mechanisms

- Many scientists (e.g., Hansen) have argued for a separation of policy vs mechanism
- A security policy indicates ***what a secure system accomplishes***
- A security mechanism indicates ***how these goals are accomplished***
- e.g.
  - Policy : in a file system, only authorized individuals allowed to access file(i.e., CIA properties)
  - Mechanism : Encryption, capabilities, etc

### Mechanisms: Golden A's

- Authentication
  - Is a user (communicating over the network) claiming to be Alice, really Alice?
- Authorization
  - Yes, the user is Alice, but is she allowed to perform her requested operation on this object?
- Auditing
  - How did Eve manage to attack the system and breach defenses? Usually done by continuously logging all operations

### Designing Secure Systems

- Don't know how powerful attack is
- When designing a security protocol need to
  1. Specify Attack Model : Capabilities of attacker (Attacker model should be tied to reality)
  2. Design security mechanisms to satisfy policy under the attacker model
  3. Prove that mechanisms satisfy policy under attacker model
  4. Measure effect on overall performance (e.g., throughput) in the common case, i.e., no attacks

## 1.2. basic Cryptography Concepts

### Basic Security Terminology

- principals : processes that carry out actions on behalf of users
  - Alice
  - Bob
  - Carol
  - Dave
  - Eve (typically evil)
  - Mallory (typically malicious)
  - Sara (typically server)

### Keys

- Key = sequence of bytes assigned to a user
  - Can be used to "lock" a message, and only this key can be used to "unlock" that locked message

### Encryption

- Message (sequence of bytes) + key -> (encryption) -> Encoded message(sequence of bytes)
- Encoded Message (sequence of bytes) + key -> (Decryption) -> Original message (sequence of bytes)
- No one can decode an encoded message without the key

### Two Cryptography Systems

1. Symmetric Key systems :

- K_A = Alice's key; secret to Alice
- K_AB = Key shared only by Alice and Bob
- Same key used to both encrypt and decrypt a message

- e.g. DES(Data Encryption Standard) : 56 bit key operates on 64 bit blocks from the message

2. Public-Private Key systems:

- K_Apriv = Alice's private key; known only to Alice
- K_Apub = Alice's public key; known to everyone
- Anything encrypted with K_Apriv can be decrypted only with K_Apub
- Anything encrypted with K_Apub can be decryptd only with K_Apriv

- RSA and PGP fail into these category
- RSA = Rivest Shamir Adleman
- PGP = Pretty Good Privacy
- Keys are several 100s or 1000s of bit long
- Longer keys => harder for attackers to break
- Public keys maintained via PKI (Public Key Infrastructure)

- How does everyone know Alice's public key?
  - Typically, a public key infrastructure or known as a PKI is maintained

### Public-Private Key Cryptography

- If Alice wants to send a secret message M that can be read only by Bob
  - Alice encrypts it with Bob's public key
  - K_Bpub(M)
  - Bob only one able to decrypt it
  - K_Bpriv(K_Bpub(M)) =M
  - Symmetric too, i.e., K_Apub(k_Apriv(M)) = M

### Shared/Symmetric vs Public/Private

When do you use the shard or symmetric key system vs the public key system?

- Shared keys reveal too much information

  - Hard to revoke permissions from principals

  - e.g. group of principals shared on key

    -> want to remove one principal from group

    -> need everyone in group to change key

- Public/private keys involve costly encryption or decryption

  - At least one of these 2 operations is costly
  - For instance, encryption is fast, but decryption become a slow operation

- Many systems use public/private key system to generate shared key, and use latter on messages

## 1.3 Implementing Mechanism using Cryptography

### 1. Authentication

- Two principals verify each others' identities
- Two flavors
  - Direct authentication : directly between two parties
  - Indirect authentication : uses a trusted third-party server
    - Called authentication server
    - e.g., a Version server

> Need 5 message
>
> 1. Alice request
> 2. Bob send nonce to ask "Are you Alice?"
> 3. Alice send back a encrypted message and Bob decrypt message and ensure it is Alice
> 4. Alice ask "Are you really Bob?" and send nonce
> 5. Bob send encrypted message and Alice decrypt and ensure it is bob

### Why Not Optimize Number of Message?

1. Alice send request with nonce
2. Bob send back nonce and encrypted Alice's nonce
3. Alice recognize Bob and send back encrypted Bob's nonce
   and Bob recognize it is Alice using decryption

### Unfortunately, This Subject to replay Attack

Scenario : Eve(Malicious) attack Bob

1. (Eve start 1st session) Eve send request with nonce, pretending Eve is Alice
2. Bob send back nonce and encrypted Eve's nonce
3. (Eve start 2nd session) Eve send request with Bob's nonce
4. Bob send back nonce and ***encrypted Bob's nonce***
5. (Eve finish 1st session) Eve send back encrypted Bob's nonce and Bob misunderstand Eve is Alice

### Indirect Authentication Using Authentication Server And Shared Keys

AS(Authentication Server is used) : trusted server, trusted by everyone.

Every principle in the system shares a key directly with authentication server.

1. Alice send message "Hey I would like to start to session with Bob" to AS
2. AS sends back a ticket which consist of the AS actually generate a new shared key for this particular session. Shared key is K_(A,B), But it is not send back as plain text.
   - A Ticket : K\_(A,AS)(K\_(A,B)), K\_(B,AS)(K\_(A,B))
   - K\_(A,B) is encrypted by K\_(A,AS) and K_(B, AS)
3. Alice sends A, K\_(B,AS)(K\_(A,B)) to Bob
   - Alice and Bob only ones who can decrypt portions of the ticket and obtain K_(A,B)

## 2. Digital Signature

- Just like "real" signatures
  - Authentic, Unforgeable
  - Verifiable, Non-repudiable(repudiable : 거부할 수 있는)
- To sign a message M, Alice encrypts message with her own private key
  - Signed message: [M, K_Apriv(M)]
  - Anyone can verify, with Alice's public key, that Alice signed it
- To make it more efficient, use a one-way hash function, e.g., SHA-1, MD-5, etc
  - Signed message:[M, K_Apriv(Hash(M))]
  - Efficient since hash is fast and small; don't need to encrypt decrypt full message

## 3. Digital Certificates

- Just like "real" certificates
- Implemented using digital signatures
- Digital Certificates have
  - Standard format
  - Transitivity property, i.e., chains of certificates
  - Tracing chain backwards must end at trusted authority (at root)

## 4. Authorization

- Access Control Matrix
  - For every combination of (principal, object) say what mode of access is allowed
  - May be very large (1000s of principals, millions of objects)
  - may be sparse (most entries are "no access")
- Access Control Lists(ACLs) = per object, list of allowed principals and access allowed to each
- Capability Lists = per principal, list of files allowed to access and type of access allowed
  - Could split it up into capabilities, each for a different(principal, file)

### Security: Summary

- Security Challenges Abound
  - Lots of threats and attacks
- CIA Properties are desirable policies
- Encryption and decryption
- Shard key vs Public/private key systems
- Implementing authentication, signatures, certificates
- Authorization