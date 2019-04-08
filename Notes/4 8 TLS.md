Transport Layer Security. Note that this doc is mostly about TLS 1.2, and the next notes will include information about the change to TLS 1.3, which is more recent. However, v1.3 is new enough that there are plenty of devices out there running servers with 1.2 and even older protocols.

TLS provides a secure channel for applications, usually between a web server and browser, as in HTTPS. TLS is more general though, for example, IMAP can run over TLS to allow for protected transfer of email.

* TLS, as a secure channel, provides message confidentiality, integrity, and replay protection:
  * Symmetric key encryption is used for message encryption and MAC computation
  * MAC covers a message sequence number for replay protection
* Mutual authentication of parties
  * Asymmetric key cryptography is used to authenticate parties to each other
  * Client authentication is optional
* Key exchange and key derivation
  * Multiple exchange methods are supported
  * Unique keys are generated for each connection
  * Different keys are used for the encryption and MAC

Only the server authenticates itself to the client to begin, then keys are exchanged, then the client, on the application layer, sends a password on to the now-authenticated server to verify its identity, but this last step is outside of the scope of TLS, but optionally supportable.

TLS also supports negotiation of cryptographic algorithms and parameters between the client and server, as multiple of each are supported. This happens in the ‘handshake.'

The master secret is a longer-term secret between the client and server.

TLS is flexible, therefore it is complicated, therefore it is difficult to secure.

SSL, the precurser to TLS, was designed by netscape in the mid 1990s. SSL v3.0 has been implemented in many browsers and servers and become a default standard without a standards body approval. SSL 1 and 2 were quickly broken.

TLS was adoped in 1999 by IETF. V1.0 was essentially SSL 3.0 with some design errors corrected. Further modification against later discovered attacks brought it all the way to v1.3, which was accepted as standard in 2018.

The point of an RFC specification is to be detailed enough that anyone can take it and implement it, and it will be interoperable with any other correct implementation.

TLS contains sub-protocols. The record protocol uses the algorithms and keys to actually make the encryption. The handshake protocol negotiates algorithms and keys. The alert protocol specifies errors. The change cipher spec protocol is a single-message protocol that serves as a trigger at the end of the handshake (once the key is established) to signal that the keys and algorithms are agreed upon, and it is now time to use those in the record protocol (trigger to change state). The change cipher spec protocol caused problems, so it was removed in v1.3 and its functionality was integrated into the handshake protocol

To read a protocol stack, understand that each layer provides services to the layers above it. For example, IP is responsible for routing and navigating packets, TCP takes the unreliable transmission of IP packets and handles misordered and missing frames, providing an abstraction of a reliable channel to protocols above it. Then, TLS provides an abstraction of a secure channel over this reliable connection to applications such as HTTP and IMAP, as well as providing the same encryption and integrity protection to the sub-protocols of TLS (the record, cipher spec, and alert protocols).

There are two layers of the security association: the connection and the session. Compromising a connection does not compromise the entire session or the other connections within the session, as they are independent.

A TLS session is a security association between a client and server. A session is a longer-term idea, a session can include multiple connection. It is stateful, meaning that the session maintains a state including the negotiated security algorithms and parameters, exchanged certificates, and the master shared secret. Connections in the same session share this session state but derive thier own connection keys from the master secret and establish connection specific random numbers.

Factoring our the master secret into the session state helps avoid expensive negotiation of new security parameters for each and every new connection. The protocols are optimized for fast new connections even if that means that the one-time session establishment is slower.

The alert protocol defines a format for error messages, which are then transported security across the established connection. A fatal error prevents the opening of new connections and terminated the connection, invalidating the session, though other existing connections do persist for the rest of their natural duration (as they are independent). A warning simply sends information to the other party.

Protocols are defined in terms of message formats, with rules for producing and consuming messages. The primatives of a TLS record protocol are familiar: A header with a type, version, length, all in clear text; IV/Nonce, in cleartext or encrypted; Application data (or handshake, alert, or change cypher spec), MAC, padding, and padding length, all encrypted with CBC mode or authenticated encyrption mode.

The padding is defined as:

* the last byte is the length n of the padding, not including the last cute
* the padding length is random, but such that the length of the padded message is a multiple of the block length of the block cypher used
* All padding bytes have a value n
* To verify: read that the last byte is n, then verify that the last n+1 bytes of the message are all n
* If verification is sucessful, remove the lsat n+1 bytes, then proceed with the MAC verification.

The randomization of the padding length also serves to partially hide the length of the message itself, which can add protection against side channel observation. 

The mac function supported is HMAC, but the hashing function inside can be MD5, SHA1, or SHA256\. The MAC covers the whole payload, the header fields, and the sequence number, which is not a part of the message. TLS uses implicit sequence numbering, meaning that both sides maintain a counter and expect a certain number, but that number itself is not included in the message, only calculated into the MAC. 

The encryption in TLS can be:

* a stream cypher (RC4)
  * No IV and padding
  * no re-initialization of the cypher for individual messages (messages are treated as a continuous stream of bytes)
* A block cypher (CBC mode of 3DES or AES)
  * IV must be random
  * requires padding as described above
* Authenticated encryption modes (AES-GCM or AES-CCM)
  * Encryption and authentication are handled by one function computed from
    * client or server write key
    * nonce
    * cleartext payload
    * sequence number, type, version, length
  * The nonce is carried in the IV field
  * No padding

Attacks on TLS (until v1.2)
---------------------------

Chosen cypher text (padding oracle) attacks were fixed in version 1.1, though the new attack lucky 13 works against 1.1

A chosen plaintext attack, such as BEAST, CRIME, BREACH, and TIME, is possible against various versions.

A reminder on padding oracle attack: assume that the attacker can access a CBC decryption oracle. The oracle receives ciphertext messages, decrypts them in CBC mode, and checks the padding, reponding with a signle bit inidicating padding correctness. By carefully crafting messages to the oracle, we can evenually send messages only controlled by our value and the plaintext.

Against a TLS server, we send a random message and the server drops the message and issues an error. Error messages are protected by the record layer, they are encrypted, preventing the attacker from seeing if its a bad\_record \_mac or a decryption\_failed message. However, the response time of the server gives it away, as a bad\_record\_mac takes more time than decryption\_failed as it requires attempting MAC verification. This is an example of a side-channel attack.

These errors are both fatal errors, they close the connection. By including the same secret, but encrypted with different keys in each message, we can still break the secret in a multi-session attack.

An example is IMAP over TLS. The same email username and password is sent in every “check-for-new-mail” message encrypted with different keys every time. As the attacker knows the block in which the username and password lives, and can take that block and put it at the end with the value R that the attacker controls. Repeating this eventually reveals the password.

To protect against this attack, the updated protocol does a mac verification regardless of padding correctness to standardize the response time, checking the MAC as if there was no padding if there was a padding error. This still leaves some leakage of time, MAC calculation time is based on message length, and MAC as if there was no padding results in a slightly longer message, but likely not enough to matter, as in it won’t stand out among noise around calculation time. 

This was wrong. Lucky13 exploits this tiny timing variation. The mimum padding length is 9\. If message length \<= 55 bytes, then the entire message plus padding can be hashed in 1 55-byte block, resulting in 4 invocations of the compression function f during HMAC. If message length is longer than 55 bytes, there will be at least 5 invocations of f during HMAC. This reveals a timing channel.

Let C\* be a 16-byte encryption block encrypting the plaintext P\*, which the attacker wants to find. The attacker sends 4 blocks (64 bytes) with proper TLS to a decryption oracle, C1, C2, R, C\*. The oracle returns P1, P2, P3, P4, where P4 relies on C\* and R, which are both controlled by the attacker. There are three things that can happen from here:

1. P1 | P2 | P3 | P4 ends with invalid padding:
  1. last 20 bytes are interpreted as MAC
  2. remaining 44 bytes are interpreted as message
  3. 44+13 bytes (sqn and header) = 57 bytes go into MAC computation \> 55 bytes -\> 5 calls to function f
2. P4 ends with x00:
  1. x00 removed as valid padding
  2. next 20 bytes interpreted as MAC
  3. remaining 64-21=43 bytes interpreted as message
  4. 43+13 bytes (sqn and header) = 56 bytes go into MAC computation \> 55 bytes -\>5 calls to function f
3. P1|P2|P3|P4 ends with a valid padding of length at least 2:
  1. at least 2 bytes are removed as valid padding
  2. message length is at most 42 bytes
  3. \<= 42+13 bytes (sqn and header) = 55 bytes go into MAC computation = 55 bytes -\> 4 calls to function f

This relies on a tiny change in timing over a network, which have delay and jitter. Also, it requires a multi-session attack, meaning it can only break structured and standard secrets. However, by making a number of calls, it is possible to calculate the empirical distribution of times to figure out if 4 or 5+ calls are made (this does significantly slow down the attack).

If the response time is ‘short,’ there is a valid padding of length at least 2 at the end of the 4 blocks

if the response time is ‘long,’ change the last two bytes of r, requiring at most 2^16 trials.