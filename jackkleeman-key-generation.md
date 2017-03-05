__OBIP__ obip-jackkleeman-key-generation  
__Title__ Standardise Seeded Key Generation  
__Author__ Jack Kleeman <jack@duo.money>  
__Discussions-To__: <jack@duo.money>  
__Status__ Draft  
__Type__ Standards Track  
__Created__ 03/03/2017  
__Post-History__ Slack, 03/03/2017 - 05/03/2017  
__Copyright__ Public Domain  

##Abstract
As more implementations of OpenBazaar arise, it is important that the key (and thus identity) generation process is standardised. OpenBazaar uses ed25519 elliptic identity keys; your peerID is the hash of the public key. OpenBazaar currently generates keys deterministically based on the mnemonic, an important and useful feature which allows users to remember or record a simple string of text which can be later used to derive their OpenBazaar and bitcoin identities. This will also come to be useful in other OpenBazaar implementations, as a mnemonic can be used to import your identity into a different client. However, for this to work, we must agree on a standard for key generation which takes note of cross-platform and -language requirements.

##Motivation
Currently the seed is passed through a `DeterministicReader` type. This turns the seed into a stream from which downstream functions can 'pull' bytes. Any number of bytes 'pulled' from this reader is in fact an scrypt-generated key, generating the required number of bytes from the entire seed, at a difficulty level of 512; a computationally intensive task, which will be particularly problematic in browser implementations. The original reason for this was to stretch seeds going into RSA key generation functions, which require long seeds and therefore a key-stretcher like scrypt to produce seeds from a 64 byte mnemonic-derived seed. OpenBazaar has since moved to elliptic keys, where a 32 byte seed is sufficient. It is still good practice to hash the seed, as seeds are partially leaked into ed25519 private keys, but there is no need to use an expensive key stretching function like scrypt.

The further problem with the current protocol is that it is quite Go-specific; producing a similar 'reader' which stretches every time bytes are requested is not trivial in other languages, and it would be far simpler to initially run the seed through a hash function, obtaining 32 bytes, and passing that to the generation function.

##Specification
I propose a simple change; given the seed, in ipfs/identity.go, simply run the mnemonic-derived seed through hmac-sha-256 once to obtain a 32 byte hashed seed, and then produce a `bytes.Reader` from this. This reader can then be passed to the ed25519 library, which will then pull the first (and only) 32 bytes for use in key generation.

In other implementations, where an ed25519 library might instead take a simple byte input instead of a go-style 'reader', one can simply hash the seed, and provide the result to the ed25519 library.

##Rationale
I have chosen hmac-sha-256 as it is the current industry standard for cryptographic hash functions, and given that the input seed is already random there is no need for stretching or artificial computational difficulty. 

Using hmac-sha-256 was discussed with lead developers Chris Pacia and Tyler Smith, and sha-256 was briefly considered instead, but we decided on using a hmac to maintain symmetry with Bitcoin BIP32 which derives seeds in a similar context.

##Backwards Incompatibility
After this change, the same mnemonic inputted to OpenBazaar will __not__ produce the same identity, which will affect the small group of people currently running nodes which need a persisting identity; for example bootstrap node runners. Given an existing identity, it is cryptographically impossible to generate a mnemonic that will produce the same identity after this obip; it would require not only computing the inverse of a hmac-sha-256 hash, but also the inverse of a pbkdf2 key-stretching operation, both of which are practically impossible. This cannot be mitigated, but fortunately the affected number of users is low; it is therefore important to make this change sooner rather than later.
