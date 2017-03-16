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
As more implementations of OpenBazaar arise, it is important that the key (and thus identity) generation process is standardised. OpenBazaar uses ed25519 elliptic identity keys; your peerID is the multihash encoded sha-256 hash of the public key. OpenBazaar currently generates keys deterministically based on the mnemonic, an important and useful feature which allows users to remember or record a simple string of text which can be later used to regenerate their OpenBazaar and bitcoin identities in case of database loss. This will also come to be useful in other OpenBazaar implementations, as a mnemonic can be used to import your identity into a different client. However, for this to work, we must agree on a standard for key generation which takes note of cross-platform and -language requirements.

##Motivation
Currently the seed is passed through a `DeterministicReader` type. This turns the seed into a stream from which downstream functions can 'pull' bytes. Any number of bytes 'pulled' from this reader is in fact an scrypt-generated key, generating the required number of bytes from the entire seed, at a difficulty level of 512; a computationally intensive task. The original reason for this was to stretch seeds going into RSA key generation functions, which require long seeds and therefore a function like scrypt. OpenBazaar has since moved to elliptic keys, where a 32 byte seed is sufficient. It is still good practice to hash the seed, as seeds are partially leaked into ed25519 private keys, but there is no need to use an expensive key stretching function like scrypt.

The further problem with the current protocol is that it is quite Go-specific; producing a similar 'reader' which stretches every time bytes are requested is not trivial in other languages, and it would be far simpler to initially run the seed through a hash function, obtaining 32 bytes, and passing that to the generation function.

##Specification
I propose a simple change; given the seed, in ipfs/identity.go, simply run the mnemonic-derived seed through hmac-sha-256 once to obtain a 32 byte hashed seed, and then produce a `bytes.Reader` from this. This reader can then be passed to the ed25519 library, which will then pull the first (and only) 32 bytes for use in key generation.

In other implementations, where an ed25519 library might instead take a simple byte input instead of a go-style 'reader', one can simply hash the seed, and provide the result to the ed25519 library.

For documentation, the full identity generation process is now:

`seed = bip39Derive(mnemonic)`  
`obSeed = HMAC-SHA256(seed, "OpenBazaar seed")`  
`identityPrivKey, identityPubkey = GenerateEd25519(obSeed)`  
`serializedPubkey = proto.Marshall(identityPubkey)`  
`hashedPubkey = sha256(serializedPubkey)`  
`identity = multihashEncode(hashedPubkey)`  

Where previously the second step used scrypt to produce an obSeed from a seed.

See the bip39 proposal [here](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) for a explanation of how a seed is generated from a mnemonic; put simply it is a pbkdf2 derivation from the mnemonic string with 2048 iterations and the salt 'mnemonic'.
##Rationale
I have chosen hmac-sha-256 as it is the current industry standard for cryptographic hash functions, and given that the input seed is already random and over 32 bytes there is no need for stretching or artificial computational difficulty. 

Using hmac-sha-256 was discussed with lead developers Chris Pacia and Tyler Smith, and sha-256 was briefly considered instead, but we decided on using a hmac to maintain symmetry with Bitcoin BIP32 which derives seeds in a similar context.

The salt choice, ie the second input to hmac-sha-256, is mostly irrelevant, it simply needs to be application specific: "OpenBazaar seed" was suggested by Chris Pacia.

