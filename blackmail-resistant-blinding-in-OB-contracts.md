## Blackmail-Resistant Value Blinding for Ricardian Contracts

### Important Update
This proposed approach is NOT safe. The seller can still blackmail the buyer by using zero-knowledge proofs to prove to a third party verifier that the buyer committed to some data. The seller can do this _without_ revealing her private key to the verifier.

DO NOT use this approach. I'm keeping up only for historical purposes.

### Motivation

A vendor or moderator may try to blackmail a buyer by threatening to reveal information that's been blinded and signed by the buyer. We would like a way for the buyer to be protected against blackmail that acheives the following:

1. The vendor should be able to verify the information blinded in the contract and signed by the buyer.
2. The vendor should not be capable of proving to a third party (other than the moderator) the values to which the buyer has commited.
3. The moderator should not be able to see or verify the information blinded in the contract except in the case of a dispute.
4. When the moderator can see/verify the blinded data, he should be bound by the same restrictions as the verifier in 2.
5. At a later time, the buyer ought to be able to unblind the chosen datafeilds in such a way that any third party can verify them.

We will call such a method for data blinding _blackmail-reistant_.

### The Protocol

Here is how a data field can be blinded by a buyer in such a way as to be blackmail-resistant.


Let `V`, `B`, and `M` denote the PGP keys for the vendor, buyer, and moderator, respectively.
Let `SECRETDATA` be the data the buyer wants to blind.
We'll use the nottion `ENC({Key1, Key2, ..., KeyN}, DATA)` to denote the encryption of `DATA` with public keys `Key1, Key2, ..., KeyN`. Let `L` be the length of `SECRETDATA`. We will assume that `L >= 128` (If not, then pad `SECRETDATA` until it is).

Then the protocol works as follows:
```
1. The buyer creates a one-time-use key pair, B'.
2. The buyer chooses a nonce, R, uniformly at random from the set {0,1}^L
3. The buyer adds the following two values to the contract:
	- ENC( {V,B',M} , SECRETDATA ⊕ R )
	- SHA256( R )
4. The Buyer signs the contract as usual (that is using pgp key B), and passes the contract to the vendor.
5. Through secure, direct communications, the buyer passes the vendor SECRETDATA and R.
```

Now the vendor can decrypt the cyphertext ` ENC( {V,B',M}, SECRETDATA ⊕ R, ) ` to reveal ` SECRETDATA ⊕ R `, then use `R` to compute `SECRETDATA`. He can further verify that the buyer did in fact commit to these values by checking that `SHA256( R )` matches the hash in the contract. But the vendor cannot prove to a third party (other than the moderator... we'll get that next) that the buyer committed to `SECRETDATA` without the vendor revealing his own PGP private key.

The moderator, by himself, can (at best) decrypt and learn the value `SECRETDATA ⊕ R`. But without knowing `R` he learns exactly nothing about `SECRETDATA`.

To bring the moderator into the fold, either the vendor or the buyer simply reveals `R` to the moderator -- at which point the moderator can learn `SECRETDATA` and confirm that it's the value to which the buyer committed... but he can't convince any third party of that fact without revealing his own private pgp key.

Finally, during rating time, the buyer can choose to unblind the value by appending the private key to `B'` and the value `R` to the contract before signing it.

Any third party can then unblind and verify the value `SECRETDATA`.

### A Note on the Direct Communications
It's important that the buyer not send the vendor or moderator signed `SECRETDATA` plaintexts, as those could be used by the vendor or moderator to prove the buyer's commitment to that data. Instead, OTR chat should be used for communications between buyer/vendor/moderator.

### Key Concept
The heart of the idea is this: the vendor/moderator can share any information they want with anyone. The only pieces of information we can count on them to NOT give away are their private keys. _So we should require their private keys be necessary to verify the data in our blinds._
