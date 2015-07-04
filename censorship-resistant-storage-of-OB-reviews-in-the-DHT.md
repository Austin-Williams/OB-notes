## Censorship-Resistant Storage of Ratings in OpenBazaar's DHT

### Motivation

Buyers rate vendors and then store these ratings in OpenBazaar's DHT. Later, when a shopper wants to see a vendor's ratings, she pulls the vendor's ratings from the DHT. Such a shooper faces two problems when receiving rating data from the DHT:
1. How does the shopper know the ratings she receives correspond to actual OB trades?
2. How does the shopper know that she's received _all_ the vendor's ratings (that the bad ratings haven't been censored)?

In other words, how do we prevent (1) forgery and (2) censorship?

We've already addressed (1) with trade receipts. Because the ratings exist in a signed and completed Ricardian contract (aka 'trade receipt'), we know ratings cannot be forged without creating fake trades via sybil buyers. Forging attacks are thus made more costly by requiring the attacker to create OB sybils.

But how can a shopper know that she's seeing _all_ the ratings; that none of the ratings are being kept from her? How can she conclude, with high probability, that she's getting the whole picture? That's the problem we attempt to address here.

### Limitations

We'd like a solution that does _not_ involve using the bitcoin blockchain. We've decided the solution should be 'in house' -- storing the values in our own DHT. Now, generally speaking, the problem of reaching consensus on a value in an open p2p-network of anonymous (and potentially malicious) participants is extreamly hard. In fact, that was the core motivation behind the invention of the blockchain to begin with. However, the type of data over which we want to reach consensus (the set of all ratings for a particular vendor) has a nice property that makes our version of the consensus problem much easier to solve. We don't need the heavy lifting a blockchain provides.

### Key Observation

Fix a vendor and let `AllVendorRatings` denote the set of _all_ ratings of the vendor in question (this is the set to which we want shoppers to have access). Note that `AllVendorRatings` is an append-only list. Ratings can be added to it, but should never be removed from it. It's size is monotonicly increasing.

Now suppose we request the set `AllVendorRatings` from some node in the network, and that node tries to fool us by giving us some other set, `CensoredRatings`, instead. Since attackers cannot forge ratings (thanks to trade receipts), and since we always verify the trade receipts before trusting them, it must be the case that `CensoredRatings` is a subset of `AllVendorRatings` -- for otherwise `CensoredRatings` would contain a forgery, and we'd just toss the forgery out. 

The fact that `CensoredRatings ⊂ AllVendorRatings` (after we throw out forgeries) is the key to the following protocol.

### Preliminaries

We'll treat the set `AllVendorRatings` as a mutable value in the DHT that corresponds to the key `RIPEMD160(VendorPGPpubKey)`. We choose the key this way so that a shopper can compute the key given only the vendor's portable identity (PGP key) -- and thus can find where in the DHT to look for the vendor's ratings.

It also puts the key into an area of the network where the vendor is unlikely to have a significant influence -- that is, a vendor is unlikely to have a GUID in the neighborhood of this key (more on this later); and as we'll see, even if he does, his influence over the value mapped to the key is extreamly limited.

As with all key/value stores, the value will be hosted by several nodes in the network (at minimum the k closest nodes to the key) to allow for nodes to come and go without the network losing the value. See the 'locating resources' section of [this page](http://distrosys.wikia.com/wiki/DHT_-_Kademlia) for more info on key/value stores in Kademlia networks.

As per usual, nodes hosting the key/value pair periodicly check in with each other to update the value -- so they all have the most recent version. The important question to ask is "how do nodes update this value -- especially considering that some of the nodes making claims about this value may be malicious?" For example: if nodes, A & B check in with each other and find they disagree on the current value of `AllVendorRatings`, how can they reconcile that discrepency? After all, neither knows whether the other party is honest. The following section addresses this issue.

### Updating Values
First let's establish some notation so we can communicate the ideas that follow more clearly.

* Let `KEY = RIPEMD160(VendorPGPpubKey)` be the key which we want to point to the value `AllVendorRatings`.

* Suppose `X` is any set that contains some ratings for a vendor. Then we'll use the notation `SANITIZE(X)` to denote the set `X` upon removal of all ratings that cannot be verified by a trade receipt. In other words, `SANITIZE(X)` is what we get when we scan through `X` and pick out _only_ the good stuff. This is a just a compact way of denoting the process of 'checking the sigs on a trade reciept'.
When we ask a node for the value associated with the key `KEY`, a malicious node may respond with any manner of malformed set `X`. But we know for sure that `SANITIZE(X) ⊂ AllVendorRatings`. In some cases `SANITIZE(X)` will be the empty set. That's not a problem.

*Suppose node `A` and node `B` are two nodes hosting the key `KEY`.
Let `Node-A-Rating-Set` be the value that node `A` is currently assigning to the key `KEY`.
Let `Node-B-Rating-Set` be the value that node `B` is currently assigning to the key `KEY`.

Now if node `A` discovers that it's value is different from that of node `B`, it updates `Node-A-Ratings` against node `B` as follows.

```
Node A requests the value for KEY from node B.
Node B responds to node A with Node-B-Rating-Set.
Node A updates Node-A-Rating-Set by setting:
	Node-A-Rating-Set ← Node-A-Rating-Set ∪ SANITIZE(Node-B-Rating-Set)
```
The result is that `Node-A-Rating-Set` contains all the ratings of which node `A` was already aware, but now also contains all additional ratings of which node `B` was aware. The point is that updating against any node, malicious or otherwise, can never _decrease_ the number of ratings which you are storing, and can only _increase_ the number of ratings you are storing. And, of course, you are never storing forgeries because you'll have weeded those out during the `SANATIZE()` process.

Node `B` updates against node `A` in the same manner. In fact, all nodes in the neighborhood of `KEY` update against each other in this way. The end result is that all honest nodes will agree on the full set `AllVendorRatings`, and there is nothing a malicious node can do to trick an honest node into censoring a rating. In fact, as long as a given rating is stored with _any_ honest node, we can be sure that _all_ honest nodes will host it after the neighborhood updates.

### Storing New Ratings
A buyer (or vendor or moderator) can store new ratings in the DHT as they would any other value. If `T` is the trade receipt containing the review to be stored, they simply issue the `STORE(KEY,T)` command to nodes in the neighborhood of `KEY`. Nodes in the neighborhood of `KEY` will recognize that `T` is a trade reciept and update thier list of ratings accordingly; for example via:
```
Node-A-Rating-Set ← Node-A-Rating-Set ∪ SANITIZE(T)
```

### Security Considerations
** Who are the nodes hosting the ratings? **
	The `KEY` for the list of ratings is an output of `RIPEMD160`, which is indistinguishable from random. Thus the "neighborhood" hosting a given vendor's ratings is, essentially, being chosen at random. This is a good thing. The vendor cannot influence this choice of 'key placement' without changing PGP keys.
** Can a vendor "infiltrate" the neighborhood hosting his ratings? **
	** Yes. And this matters. ** The vendor needs only create a GUID that is closer to `KEY` than one of the k nodes currently nearest to `KEY`. He can do this by trial-and-error until he finds such a GUID. For every `N/k` GUIDs (where `N` is the total number of OB nodes) the attacker creates, one of them (on average) will be in the neighborhood of `KEY`.
	Thus an attaker can have _total control_ over a neighborhood of _any key_ in the DHT (not just a key for ratings) after creating, on average, just `N` GUIDs. This is a plausible attack, because GUIDs are (currently) very cheap to make. This attack can be prevented by making the initial creation of GUIDs more computationally expensive. See the 'Making GUIDs Costly' section below.
** How much damage can malicious nodes do? **
	The protocol is actually quite robust against malicious nodes. Suppose attackers control `m <= k` malicious nodes in the neighborhood of a `KEY` and suppose a shopper queries `q` nodes at random from the neighborhood asking for the list of vendor ratings. If `m < q` it is impossible for the attackers to censor any ratings at all. To say that another way, if a shopper queries more nodes than the attackers control, the shopper _will_ recieve the entire set `AllVendorRatings`.
	If `m >= q` then the probability that the attacker can censor ratings from the shopper is given by `(product ((m-n)/(k-n)), n=0 to (q-1)`. Let's plug some actual numbers into that to see how robust the protocol is. If `k=24` and a shopper queries just `q=4` nodes asking for the vendor's ratings, then in order for the attackers to have even a 5% chance of censoring the ratings the attackers would need to control `m=12` mailicious nodes -- that's half the nodes -- in the neighborhood of `KEY`.
** A rating's original STORE in the DHT must be seen by at least one honest node. **
	Otherwise it can be immediately censored by a malicious node -- that node simply won't relay or store the rating. This is less of a problem than it may first appear. We can simply directly request the new rating be stored in a few randomly selected nodes in the neiborhood of `KEY`. If a buyer (or vendor or moderator) directly connects to just 4 randomly selected nodes in the neighborhood of `KEY` and stored the review there, attackers would need to control _half_ of the nodes in the neighborhood of `KEY` in order to have even a 5% chance of censoring the review (assuming `k=24`).


### Making GUIDs Costly
Making initial GUID creation coslty is extraordinarily simple. One easy way to do it is to incorporate a proof-of-work into the original computation of the GUID.  For example, rather than computing the GUID as `GUID = RIPEMD160(SHA256(self_signed pubkey))` we simply compute the GUID as `GUID = RIPEMD160(SHA256(self_signed pubkey ) || nonce)` where `nonce` is a value such that `SHA256(self_signed pubkey || nonce))` -- interpreted as an integer -- is less than some global difficulty parameter `d`.
When a node checks the validity of a GUID, they not only expect proof that the node holds the privKey corresponding to the self_signed pubkey, but they also verify that `SHA256(self_signed pubkey || nonce) < d`.
If we decide on a minimum hardware spec for running OB, we could calculate the difficulty `d` so that clients running on minspec hardware could create a GUID in, say, 10 min (a one-time GUID setup cost of 10 min is a small price to pay for access to a worldwide decentralized marketplace with no censorship or fees). Users running faster hardware could compute a GUID in, say, 1 min.

### Final Thoughts

Preventing ratings from being censored is not easy in an anonymous p2p-network. Storing the ratings _solely_ in the DHT and still acheiving censorship-resistance is even harder. Any one node could have plenty of reason to censor a rating from a buyer: A moderator or third party could be bribed by the vendor to censor his negative ratings, for example. The protocol described here works by storing the ratings with several nodes. It provides reasonable assurances to the shopper even in the case where more than half of the storage nodes have been compromised. But we must keep in mind that, with GUIDs being so cheap to create, a determined attacker can quite easily take over the entire neighborhood responsible for storing ratings.

As a result, I can only recommend this method if we also make GUID creation more costly.

Using the bitcoin blockchain to store pointers to ratings would provide maximum assurances against censoriship -- and from a strictly security concious point-of-view I think that would be the ideal, but I understand that we've decided to go a different direction. If we are going to store the ratings solely in the OB DHT, I think we should require more than just the vendor and moderator nodes to store it.