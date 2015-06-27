So here is how a data field can be blinded by a Buyer in such a way as to be blackmail-resistant.
```
Let V, B, and M denote the PGP keys for the vendor, buyer, and moderator, respectively.
Let SECRETDATA be the data the buyer wants to blind.
Let ENC({Key1, Key2, ..., KeyN}, DATA) denote the encryption of DATA with public keys Key1, Key2, ..., KeyN.

Then the protocol works as follows:

1. The buyer creates a one-time-use key pair, B'.
2. The buyer chooses a nonce, R, uniformly at random from the set {0,1}^128
3. The buyer adds the following two values to the contract:
	- ENC( {V,B',M} , SECRETDATA ⊕ R )
	- SHA256( R )
4. The Buyer signs the contract as usual (that is using pgp key B), and passes the contract to the vendor.
5. Through secure, direct communications, the buyer passes the vendor SECRETDATA and R.
```

Now the vendor can decrypt the cyphertext ` ENC( {V,B',M}, SECRETDATA ⊕ R, ) ` to reveal ` SECRETDATA ⊕ R `, then use `R` to compute `SECRETDATA`. He can further verify that the buyer did in fact commit to these values. But the vendor cannot prove to a third party (other than the moderator... we'll get that next) that the buyer committed to SECRETDATA without the vendor revealing his own PGP private key.

The moderator, by himself, can (at best) decrypt and learn the value `SECRETDATA ⊕ R`. But without knowing `R` he learns exactly nothing.

To bring the moderator into the fold, either the vendor or the buyer simply reveal `R` to the moderator -- at which point the moderator can learn `SECRETDATA` and confirm that it's the value to which the buyer committed... but he can't convince any third party of that fact without revealing his own private pgp key.

Finally, during rating time, the buyer can choose to unblind the value by adding revealing and appending the private key to `B'` and `R` to the contract before signing it.

Any third party can then unblind and verify the value `SECRETDATA`.
