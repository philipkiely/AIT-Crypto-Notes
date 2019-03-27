Control Questions

Lecture 1 - Stream Ciphers

§ What are the three main properties of the XOR operation? § How does the one-time pad work?

X + 0 = 0 + X = X

X + X = 0

If A + B = C, then A = B + C and B = A + C

One-time pad

Represents plaintext as a sequence of bytes

Takes a password and repeats it many times to get a byte string as long as the plaintext

Obtain ciphertext by XOR-ing together the plaintext and password string

§ What does perfect secrecy mean intuitively?

Observing the encrypted message provides no new information about the original message

§ What are the disadvantages of the one-time pad?

Difficult to send key to recipient in secure manner, only practical way is to use an out-of-band channel before communication begins

This needs to be done with all potential communication partners

Key management becomes cumbersome

§ How stream ciphers work? (general internal structure)

Key stream is generated independently of the plaintext and ciphertext using a key K to initialize a state. After each iteration, a function of the state is outputted as z\_i and another function updates the state.

Each element of input stream m\_i is XORed with m\_i to get ciphertext output element c\_i

§ Why should the state space of a stream cipher be large? 

Key streams would start to repeat

if two plaintext characters mi and mj are encrypted with the same key character z, then the XOR sum of the ciphertext characters (m\_i + z) + (m\_j + z) = m\_i + m\_j

§ What are the advantages of stream ciphers?

Efficient, fast, requires small amount of memory 

§ What are the disadvantages of stream ciphers? 

Ciphertext is at least as long as plaintext

Synchronization needed

No integrity protection - attacker can make changes to ciphertext characters and know how these changes affect plaintext

Lecture 2 - Block Ciphers   

§ How block ciphers work? What does it mean intuitively that a block cipher behaves like a random permutation?

Stateless cipher that operates on blocks of bits to produce an output that cannot be distinguished from a random permutation

Intuitively, this means that if the key used in the block cipher encryption is unknown, then the output is unpredictable

§ How block ciphers are constructed? What do we mean by an avalanche effect?

Use multiple layers of simple operations (i.e. elementary arithmetic operations, logical operations, byte substitutions, and bit permutations) that make the resulting cipher more secure than the individual components

A small perturbation in the plaintext creates a very different output. Specifically in the context of Shannon’s SP network, we have that changing one bit in the input changes each bit in the output with probability about 1/2

§ What is the Kerkchoff principle? Why is it important?

Kerkchoff’s second principle says that it must be assumed that the encryption algorithm is known to the attacker

It is important because the security of the system cannot depend on the secrecy of the algorithm. In other words, even if the attacker knows the algorithm, it should still be secure

This is a good principle to follow because public, published designs receive more scrutiny and standards. Also, the secrecy of an algorithm can be broken by reverse engineering

§ What are the standard attacker models in case of encryption?

Ciphertext-only attack

Known-plaintext attack

Chosen-plaintext attack

Chosen-ciphertext attack

§ How does exhaustive key search work? What is its complexity?

Given a small number of plaintext-ciphertext pairs encrypted under a key K, K can be recovered by exhaustive search with 2^{k-1} processing complexity where k is the size of the key space

This brute force approach progresses through the entire key space and keeps trying candidate keys to encrypt plaintext and compares them to the ciphertext hoping for a match

§ For a cipher to be secure, is it sufficient to have a large key size?

No, it is necessary but sufficient to have a large key size. There may also be weaknesses in the algebraic structure of a block cipher that would allow for attacks that are more efficient than exhaustive key search.

§ What is double encryption and how does the meet-in-the- middle attack work on it?

Double encryption uses two keys, K1 and K2, to encrypt input text X using an encryption block cipher E twice, so that Y = E\_{K2}(E\_{K1}(X)).

To perform a meet-in-the-middle attack on double encryption, an attacker with access to a plaintext-ciphertext pair (X,Y) can compute and store E\_K(X) for all K and compute and store D\_K(Y) for all K. The attacker then looks for a match. Storing complexity ~ 2^k and processing complexity ~ 2 \* 2^k = 2^{k+1} \<\< 2^{2k}

§ How does 3DES work?

A key bundle (K1, K2, K3) is used such that Y = DES\_{K3}(DES^{-1}{K2}(DES{K1}(X)))

§ What are the main block cipher selection criteria? 

Design assumptions vs. application requirements

Efficiency

Speed

Memory

Code size

Security

Key size

Algebraic properties

Complexity of best known attacks

Openness of specification

Patent issues

Lecture 3 - Block Encryption Modes

§ Why do we need block encryption modes?

We usually need to encrypt messages that are longer (or shorter) than the block size of the block cipher

§ What is the main weakness of the ECB mode?

Encrypting the same plaintext with the same key results in the same ciphertext (no randomization). This does not properly hide patterns in the plaintext.

§ How do the CBC mode work and what is its main properties?

CBC encrypts plaintext using the same key, but ciphertext block Y\_{i} depends on X\_{i} as well as all preceding plaintext blocks.

CTR 

– strength and weaknesses?

CBC strength is that different IVs result in different ciphertext. CBC weakness is that chosen ciphertext attacks may be possible because proper decryption of a ciphertext block only needs a correct proceeding ciphertext block.

– issues with IVs(CBC)?

IVs must be the same size as the block size, must be transferred to the receiver, and must be unpredictable but not necessarily secret. 

– padding?

Extra bytes added to the short end block such that it reaches the correct size

Receive must unambiguously be able to recognize and remove padding so padding is always used even if the length of the original message is a multiple of length of the block size

Different padding schemes

– self-synchronization?

Automatically recovers from loss of a ciphertext block

– random access, parallel computation, pre-computation?

Yes random access, parallel computation only for decryption, and no pre-computation

§ How do the CBC mode work and what is its main properties?

Stream cipher where the internal state is (K, ctr\_{i}) and after each ciphertext character c\_i is outputted by XORing z\_i and m\_i, the update function increments the counter. The z\_i value is the output of the block cipher with the internal state.

– strength and weaknesses?

Strength is that it does not need to increase message length

No integrity protection (changes in ciphertext can cause controlled modification of ciphertext)

– issues with counters(CTR)?

Counter values should not repeat within a given message

Initial counter must be chosen to ensure counters are unique across all message generated under the given key

– padding?

No padding needed

– self-synchronization?

No

– random access, parallel computation, pre-computation?

Yes, yes, and yes

§ How to generate unpredictable IVs for CBC?

IV computed with a nonce N, IV = E\_{K}(N), or a cryptographic RNG

§ How to generate non-repeating counters for CTR? 

Divide the counter into two sub-blocks, ctr = ctr’|ctr’’

Increment ctr’ with every new message 

Increment ctr’’ with every new block within the message

Lecture 4 - Attacks on CBC

§ What is the basic idea of the content leak attack?

If two encrypted blocks happen to be equal, Y\_{i} = Y\_{j}, then D\_k(Y\_{i}) = D\_k(Y\_{j}) =\> X\_i + X\_j = Y\_{i-1} + Y\_{j-1}

Attacker learns the difference X\_i + X\_j

§ When do we expect to have at least two identical ciphertext blocks in a CBC encrypted message? (length of message as a function of the block length)

k = sqrt(N) = 2^{n/2} where k is the number of output blocks and N = 2^n is the size of the output space

§ What attacker model does the padding oracle attack belong to?

Chosen ciphertext attack model

§ What is the main idea of the padding oracle attack?

Padding errors leak information, so an attacker can decrypt any CBC encrypted message efficiently by repeatedly sending crafted ciphertexts to the server and observing its response

An attacker can recover the last byte and iteratively recover all previous bytes

§ How we can prevent padding oracle attacks?

Encrypt-then-MAC construction is one way

§ Why are predictable IVs in CBC mode dangerous?

Subject to be exploited by chosen plaintext attack

§ What could be the problem with repeated guessing of a plaintext block in practice?

The block length of the cipher is large and guessing X\_i may be infeasible

§ When can the guessing attack that exploits predictable IVs still work? 

The attacker can prepend arbitrary plaintext to the plaintext before it is encrypted so she does not need to guess the entire block because a large part is already known

Lecture 5 - Message Authentication

§ What is a cryptographic hash function?

Maps arbitrarily long messages into a fixed length output

§ What are the 3 main security requirements on crypto hash functions?

Weak collision resistance: given an input x, it is hard to find an x’ such that H(x) = H(x’)

Strong collision resistance: computationally infeasible to find an x and x’ such that H(x) = H(x’)

Preimage resistance: given a hash value y, it is computationally infeasible to find any input x such that H(x) = y

§ What is the Birthday Paradox and how is it related to hash functions?

When drawing elements randomly from a set of N elements, a repeated element will be encountered with high probability after ~sqrt(N) selections

After ~sqrt(N) hash functions on different inputs, it is likely that there is a collision

§ How iterative hash functions work? (scheme)

Input is divided into fixed length blocks

Last block is padded if necessary

Each input block is processed according to the following scheme:

§ Describe the so called ”sponge construction”! 

Message blocks are XORed into a subset of the internal state, which is then permuted as a whole by a block permutation

§ How are MAC functions used to ensure message authentication? (basic operating principle)

Maps an arbitrarily long message and a key into a fixed length output

After successful verification of the MAC value, the receiver is assured that the message has been generated by the sender and not altered in transit

§ What attacker models do you know for MAC functions?

Forge valid MAC values on a set of messages

Recover the MAC key

§ What are the desired security properties of MAC functions?

Key non-recovery: should be hard to recover the secret key K given one or more message-MAC pairs for that K

Computation resistance: given zero or more message-MAC pairs, it is hard to find any valid message-MAC pairs for a new message

§ How do brute force attacks against MAC functions work?

Given a number of message-MAC pairs (m\_i, M\_i), for every possible key, compute the MAC value of m\_i and compare it to M\_i. If there is a match for every pair, then it is very likely we have found the right key.

Complexity is 2^k

§ What is the meet-in-the-middle attack in the context of MAC functions?

If an attacker knows a message m\* to be sent ahead of time in a communication session, he precompute MAC values of m\* with 2^{k/2} randomly chosen keys and store the (MAC\_{k}(m\*), K) pairs

When the party starts the session, the attacker checks the actual MAC value against his precomputed values. If a match is found, it is highly likely the attacker has obtained the MAC key

Using the MAC key, the attacker can forge and modify further messages in the session

§ How do HMAC and CBC-MAC work? (scheme)

HMAC\_{K}(m) = H((K+ XOR opad) | H((K+ XOR ipad) | m))      

H is an iterative hash function with input block size b and output size n

K+ is K padded with 0s to obtain length of b bits

ipad and opad are constants

§ What is the potential problem with CBC-MAC? How to avoid it? 

Susceptible to forgery

Countermeasures:

Optional final encryption

Prepend to the message a block containing the length of the message before MAC computation

Use K to encrypt the message length and use the result as the MAC key 

Lecture 6 - Secure Channels

§ What is authenticated encryption? What are the 2 main approaches to achieve authenticated encryption?

Simultaneously protects the confidentiality and the integrity of a message

Approaches

Different combinations of an encryption function and a MAC function

Specialized authenticated encryption schemes

§ Describe the generic combinations of encryption and MAC! Which one is the most secure?

ENC\_{Ke}(m | MAC\_{km}(m))

The MAC is verified only after decrypting the message and checking the padding

If information on correctness of padding is leaked out, then a padding attack is possible

ENC\_{Ke}(m) | MAC\_{Km}(m)

The MAC is verified only after decrypting the message and checking padding

Padding oracle attack may be possible

ENC\_{Ke}(m) | MAC\_{Km}(ENC\_{Ke}(m))

No decryption and verification of padding before MAC verification

Chosen ciphertext attacks (including padding oracle) will fail!

MOST SECURE

§ How does the CCM authenticated encryption scheme work?

CTR mode with CBC-MAC

Input: message X, key K, associated data A, and nonce N

Output: encrypted message Y, authentication tag T (encrypted CBC-MAC value)

§ What are the advantages of special authenticated encryption schemes? 

Use a single key

Prevent chosen-ciphertext attacks (e.g. padding oracle)

Can be more efficient than generic combinations of MAC and encryption

Some schemes process the message in a single pass

§ How can message sequence numbers be used for replay detection?

Requires that sequence numbers of received messages are monotonically increasing

§ What is the difference between an explicit and an implicit message sequence numbering scheme?

In explicit sequence numbering, C\_{snd} is sent explicitly and if C\_{snd} \> C\_{rcv} the receive accepts the message, sets C\_{rcv} = C\_{snd}, else drops the message

In implicit sequence numbering, C\_{snd} is encrypted in the MAC function and the receiver accepts the message only if MAC is verified assuming C\_{rcv} = C\_{snd} + 1, then increments C\_{rcv}, else drops the message

Explicit sequence numbering can work in an unreliable network but implicit sequence numbering only works in a reliable network

§ How can replay protection be solved in an unreliable network?

Receiving window keeps track of received sequence numbers within a window of a given size

Drops sequence numbers smaller than left edge of window

If received message has sequence number larger than right edge of window and MAC is correct, receiver accepts message and the window is advanced

If received message falls within the window, receiver accepts the message only if its MAC is correct and the number has not been seen before

§ How should keys be derived from a long-term shared secret between the communicating parties?

Keys should be derived from the shared secret with a psuedo-random one-way function

Derived key cannot be predicted without shared secret

Shared secret cannot be deduced from the derived key

§ What approaches do you know for defining message formats? 

Message with a fixed structure (header, payload, footer)

Message as a series of type-length-value (TLV) triplets

Type defines how to interpret what follows

Length defines the length of the value

Value is some data of a given type

Lecture 7 - Public Key Cryptography

§ What is the basic idea of public-key cryptography?

A public-key private-key pair (K+, K-) is generated in which an encryption function uses K+, E(K+, X) = Y, and a decryption function uses K-, D(K-, Y) = X

§ What is a digital envelope? (hybrid approach)

Public-key encryption is used to encrypt a symmetric key and the coded data and coded symmetric key are outputted

§ What hard problems is the security of public-key crypto schemes related?

Factoring

Computing the discrete logarithm

Some schemes can be reduced to NP-hard problems

§ What is semantic security? How to achieve it?

An attacker should not be able to choose two plaintexts X and X’ and later distinguish between the encryptions E\_{k}(X) and E\_{k}(X’) of these messages

Probabilistic encryption uses some random input so that encrypting the same message twice will result in different outputs

§ How does the RSA algorithm work?

Choose two large primes p and q

n = pg, phi(n) = (p-1)(q-1)

Choose e such that 1 \< e \< phi(n) and gcd(e, phi(n)) = 1

Compute the inverse d of e (mod phi(n))

Output public key: (e, n)

Output private key: d (and p, q)

Encryption algorithm: c = m^e mod n

Decryption algorithm: m = c^d mod n

§ Which hard problems RSA is related to?

Factoring

§ What are practical issues to consider in case of RSA? 

– unconcealed messages

 A message is unconcealed if it decrypts to itself, m^{e} mod n = m

– small messages

 If m \< n^{1/e}, then m^e \< n , and hence c = m^e

 This is bad because m can be computed by taking eth root of c

– small encryption exponent

Attacker can use Chinese Remainder Theorem to determine m if the sender sent  m to at least e recipients and e is small

– homomorphic property

If m1 and m2 are two plaintext messages and c1 and c2 are the corresponding ciphertexts, then the encryption of m1m2 mod n is c1c2 mod n. Leads to adaptive chosen-ciphertext attack

§ What is PKCS \#1 ? 

Formatting to impose structural constraints on plaintext so that RSA is no longer malleable

An encryption algorithm is "malleable" if it is possible to transform a ciphertext into another ciphertext which decrypts to a related plaintext

§ How does the ElGamal encryption work?

Parameters: p is a large prime, q is a prime divisor of p-1, g in [1,p-1] is an element of order q

Private key: uniformly randomly selected from [1,q-1]

Public key: y = g^{x} mod p

Encrypt: Chose uniformly random k from [1, q-1] and compute c1 = g^{k} mod p and c2 = my^{k} mod p. Output (c1, c2)

Decrypt: Output c2(c1^{-x}) mod p = m

§ What is Elliptic Curve Cryptography (ECC) in a nutshell?

Cyclic subgroup of an elliptic curve defined over a finite field generated by a point within the field can be used to implement discrete logarithm systems. This difficult problem is used in the encryption of the plaintext 

§ What are the advantages of ECC?

Smaller parameters

Faster operations 

§ How does the ElGamal algorithm work over elliptic curves?

Cyclic subgroup of prime order n of E(F\_p) for some prime p and elliptic curve E over F\_p

Group operation is point addition, point/scalar multiplication

Computations:

Public key: Q = dP

Encryption: C1 = kP and C2 = M + kQ

Decryption: M = C2 - dC1

\*See slide deck for more details

§ What is a digital signature scheme?

a digital code (generated and authenticated by public key encryption) which is attached to a message to verify its contents and the sender's identity

§ What is the key difference between MAC functions and digital signatures? What additional security function do signatures provide?

Unlike a MAC function, digital signatures are unforgeable by the receiver and verifiable by a third party

Additional security includes message authentication and integrity protection, as well as non-repudiation of origin

§ What attacker models do exist for digital signature schemes?

Key-only attack (attacker knows only the signature verification key)

Known-message attack (attacker has message-signature pairs)

(Adaptive) chosen-message attack (attacker can choose a message and obtain its signature from an oracle)

§ What is the hash-and-sign paradigm?

The message is hashed and the hash is encrypted with the sender’s private key to generate a signature. The verification is performed by decrypting the signature with the public key and seeing if it matches the hash of the message.

– Why is it used in practice?

Increase efficiency by signing the hash of the message instead of the message

– What are the requirements on the hash function used? 

It is essential that the hash function is collision resistant

Lecture 8 - Public Key Certificates 

§ Why do we need public key certificates?

Public key certificates provide a scalable approach for public key authentication

§ What essential elements does a public key certificate contain?

Subject identification information

Subject public key

Certification Authority’s (CA) name

CA’s digital signature

§ Why should certificates have an expiration date?

Key-pairs should not be valid forever

§ What could be reasons for revoking a certificate before it expires?

Private-key is (suspected to be) compromised

Change of data contained by the certificate

§ What is a CA? What are its functions? Why do we trust CAs?

CA is a collection of hardware, software, and staff

Functions:

Issues certificates for users or other CAs

Maintains certificate revocation information

Publishes currently valid certificates and certificate revocation list

Maintains archives

Must comply with strict security requirements related to the protection and usage of its private keys

Tamper resistant hardware security modules

Defines and publishes its certificate issuing policies

Complies with laws and regulations

Is subject to regular control

§ What are the steps of the certificate life cycle?

Subscriber registers with the CA and requests a certificate

CA validates information provided by the subscriber

Key-pair generation

Issuance of the certificate

Certificate may be revoked if needed

Certificate expires

§ How are private keys stored and protected?

CA’s private key stored on tamper resistant hardware module 

The private key of a subscriber is stored on a tamper resistant hardware token (e.g. smart card) or on regular data storage media (e.g. USB key)

§ How a hierarchical PKI operates?

CAs are typically organized into a hierarchy where the key of a subordinate CA is certified by another, higher level CA

Modelled as a directed tree where nodes are CAs and edges are certificates

§ What is a certificate chain? How is it verified?

A certificate chain is a sequence of certificates that starts with root CA’s self-signed certificate, ends with end user certificate, and contains certificates of intermediary CAs on the path from the root to the user

Signatures in the chain are verified by previous certificate’s public key. Root certificate verifies itself and every end-user must obtain an authentic copy of the public key of the root CA (obtained out-of-band)

§ How are certificates revoked? 

Revoked certificates are usually put on a Certificate Revocation List (CRL) published by the CA

Revocation status must be checked for every certificate in the chain to verify the entire chain

Lecture 9 - Random Number Generation

§ Why do we need random numbers in cryptographic applications?

Random numbers (bits) are needed for various purposes, including for forming cryptographic keys and other cryptographic parameters

§ What is the difference between a true random number generator and a pseudo-random number generator (PRNG)?

The output of a true random number generator cannot be predicted by an observer before it is generated, while a pseudo-random number generator processes somewhat unpredictable inputs and generates pseudo-random outputs

True random numbers may be difficult to obtain in large quantity, whereas pseudo-random numbers can be generated quickly in large quantity

§ What practical problems do PRNGs solve?

Gives fast access to a large quantity of numbers, where if implemented properly, cannot be distinguished from a real random sequence

§ What is the general structure of PRNG?

State is updated after every output and recomputed when sufficient amount of entropy is collected from the inputs

Generation is usually a one-way function like a hash or block cipher

§ What do we assume about the attacker in case of PRNGs? (goals and capabilities)

Goals:

Predict future outputs

Compute previous (not yet observed) outputs

Recover the PRNG internal state

Capabilities:

Weak: attacker can only observe some outputs

Medium: attacker can observe or manipulate collected input samples

Strong: attacker can occasionally compromise internal state

§ How does the Fortuna PRNG work?

– internal state (what makes up the internal state?)

C - 128 bit counter

K - 256 bit key

– generation algorithm (how outputs are generated?)

– entropy accumulator (how true random samples are collected?)

 – reseed control (when reseeds happen?)

– reseed algorithm (how reseeding works?) 

Lecture 10 - Key Exchange Protocols

§ What is the purpose of key exchange protocols?

§ What are the design objectives of key exchange protocols?

Secrecy of the key

Key authentication

Key freshness

§ What are the two main types of key exchange protocols?

§ What do we assume about the attacker?

§ What kind of attacks do you know against key exchange protocols?

§ How do key transport protocols ensure the secrecy of the established key?

§ How is key authenticity achieved?

§ What methods do you know for providing key freshness? Explain how they work, and what their advantages and disadvantages are!

§ What is perfect forward secrecy?