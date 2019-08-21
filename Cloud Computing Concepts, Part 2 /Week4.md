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